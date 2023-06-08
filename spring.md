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
deduceMainApplicationClass的实现原理比较巧妙，新建了一个运行时异常对象，通过这个对象获取当前的调用函数堆栈数组StackTrace，之后遍历这个堆栈数组，找到方法名为main的类，返回这个类。

SpringBoot将deduceMainApplicationClass方法推断出来的类赋值给了this.mainApplicationClass。通过idea的Find Usages查看属性的使用情况，发现只是在banner打印或者log上有涉及，好像没发现有很大的重要性。
虽然没有发现this.mainApplicationClass比较重要的使用价值，但是推断这个类的实现方法deduceMainApplicationClass的实现原理还是挺巧妙的，值得学习一下。
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
