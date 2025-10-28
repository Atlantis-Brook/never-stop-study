#Java #MybatisPlus

- **Problem**: Github中最近的贡献记录丢失问题，导致在贡献图标中看不到自己的历史提交记录。
- **Solution**:
	 - 最佳实践写法（防 NPE），写 Lambda 条件时，推荐这样写：
```java
Long systemId = systemInfo != null ? systemInfo.getId() : null;

LambdaQueryWrapper<SlowPageStatisticsPO> query = Wrappers.lambdaQuery();
query
    .between(startTime != null && endTime != null, SlowPageStatisticsPO::getCreateTime, startTime, endTime)
    .eq(systemId != null, SlowPageStatisticsPO::getSystemId, systemId);

```
- **Note**: ⚠️如果传入参数表达式包含对象访问（如 `systemInfo.getId()`），可能 NPE

