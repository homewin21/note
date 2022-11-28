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

### 3.获取工厂实例
```java
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<String>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```
从“META-INF/spring.factories”文件加载和实例化给定类型的工厂，中间还涉及到一些类加载器classLoader对文件中类定义资源的读取操作，同时在spring5.0版本SpringFactoriesLoader也会用一个Map<ClassLoader, MultiValueMap<String, String>> cache = new ConcurrentReferenceHashMap();作为缓存来减少数据读取操作
参考：https://blog.csdn.net/BlackReimu/article/details/123982778
