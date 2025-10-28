#Java #MybatisPlus

- **Problem**: 在使用mp中的`.eq(condition, column, val)`时
```java
	List<Long> deleteIdList = slowPageOverviewService.lambdaQuery()  
		.between(startTime != null && endTime != null, SlowPageOverviewPO::getCreateTime, startTime, endTime)  
		.eq(systemInfo != null, SlowPageOverviewPO::getSystemId, systemInfo.getId())  
		.list()  
		.stream()  
		.map(SlowPageOverviewPO::getId)  
		.collect(Collectors.toList());
	```
- **Solution**:
	 - 问题根因：
	   在 MyBatis-Plus 的源码中，无论是 `QueryWrapper` 还是 `LambdaQueryWrapper`，它们都继承自 `AbstractWrapper`。  
	   核心方法定义在 `AbstractWrapper` 中：
  ```java
	public Children eq(boolean condition, R column, Object val) {
    return addCondition(condition, column, EQ, val);
}
	```
	-   最佳实践写法（防 NPE），写 Lambda 条件时，推荐这样写：
```java
Long systemId = systemInfo != null ? systemInfo.getId() : null;
	
	LambdaQueryWrapper<SlowPageStatisticsPO> query = Wrappers.lambdaQuery();
	query
	    .between(startTime != null && endTime != null, SlowPageStatisticsPO::getCreateTime, startTime, endTime)
	    .eq(systemId != null, SlowPageStatisticsPO::getSystemId, systemId);
```
- **Note**: ⚠️如果传入参数表达式包含对象访问（如 `systemInfo.getId()`），可能 NPE

