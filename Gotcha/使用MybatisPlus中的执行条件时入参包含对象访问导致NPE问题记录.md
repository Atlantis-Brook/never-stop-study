#Java #MybatisPlus

- **Problem**: 在使用mp中的`{java icon} .eq(condition, column, val)`时，即使`{java icon} condition`中增加了条件判断，依然会执行`val`中的`{java icon} systemInfo.getId()`导致**NPE**问题
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
	 - 条件查询：
	   在 MyBatis-Plus 的源码中，无论是 `QueryWrapper` 还是 `LambdaQueryWrapper`，它们都继承自 `AbstractWrapper`。  
	   核心方法定义在 `AbstractWrapper` 中：
```java
	public Children eq(boolean condition, R column, Object val) {
    return addCondition(condition, column, EQ, val);
}
```
	- 问题根因：Java 求值顺序，区别主要在 Java 自身的 **求值机制**
	```java
	.eq(Objects.nonNull(systemInfo), SlowPageStatisticsPO::getSystemId, systemInfo.getId())
	```
	⚠️ 在调用 `.eq()` 前，Java 会 **先求出所有参数的值**：
	1. 计算 `Objects.nonNull(systemInfo)`；
	2. 计算 `SlowPageStatisticsPO::getSystemId`；
	3. 计算 `systemInfo.getId()`（这里直接空指针了）。
	然后才执行 `.eq(...)`。 
	- 最佳实践写法（防 NPE），写 Lambda 条件时，推荐这样写：
```java
Long systemId = systemInfo != null ? systemInfo.getId() : null;
	
	LambdaQueryWrapper<SlowPageStatisticsPO> query = Wrappers.lambdaQuery();
	query
	    .between(startTime != null && endTime != null, SlowPageStatisticsPO::getCreateTime, startTime, endTime)
	    .eq(systemId != null, SlowPageStatisticsPO::getSystemId, systemId);
```
- **Note**: ⚠️如果传入参数表达式包含对象访问（如 `systemInfo.getId()`），可能 NPE

