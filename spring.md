	记录spring源码相关内容
### 1. org.springframework.boot.SpringApplication#deduceMainApplicationClass
```java
	private Class<?> deduceMainApplicationClass() {
		try {
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			for (StackTraceElement stackTraceElement : stackTrace) {
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// Swallow and continue
		}
		return null;
	}
```
参考:http://qclog.cn/1133

### 2.异常处理机制FailureAnalyzer
参考:https://juejin.cn/post/6991263997915824141
