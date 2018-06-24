
### 源码阅读

### 客户端
````
客户端入口AutoJsonRpcClientProxyCreator
实现了BeanFactoryPostProcessor, ApplicationContextAware.初始化完成后会默认执行postProcessBeanFactory方法和setApplicationContext方法
````
#### postProcessBeanFactory 方法分析
根据客户端配置获取到扫描路径
String resolvedPath = resolvePackageToScan();
获取包下所有的Resource资源applicationContext.getResources(resolvedPath)
遍历每个Resource
获取Resource的详细信息
````
SimpleMetadataReaderFactory metadataReaderFactory = new SimpleMetadataReaderFactory(applicationContext);
MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(resource);
ClassMetadata classMetadata = metadataReader.getClassMetadata();
AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
String jsonRpcPathAnnotation = JsonRpcService.class.getName();
主要是判断有没有JsonRpcService这个注解
String className = classMetadata.getClassName();
//String path = (String) annotationMetadata.getAnnotationAttributes(jsonRpcPathAnnotation).get("value");获取注解的值
String path = className;可以修改直接获取类名称
````
registerJsonProxyBean  ：Registers a new proxy bean with the bean factory. 动态注册一个代理类到spring中
````
	private void registerJsonProxyBean(DefaultListableBeanFactory defaultListableBeanFactory, String className, String path) {
		BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder
				.rootBeanDefinition(JsonProxyFactoryBean.class)
				.addPropertyValue("serviceUrl", appendBasePath(path))
				.addPropertyValue("serviceInterface", className);
		
		if (objectMapper != null) {
			beanDefinitionBuilder.addPropertyValue("objectMapper", objectMapper);
		}
		
		if (contentType != null) {
			beanDefinitionBuilder.addPropertyValue("contentType", contentType);
		}
		
		defaultListableBeanFactory.registerBeanDefinition(className + "-clientProxy", beanDefinitionBuilder.getBeanDefinition());
	}
	
代理类会通过 jsonRpcHttpClient.invoke  发送请求	
````server url http://localhost:8081/manager/com.fayayo.api.ProductRpc

### 服务端
````
AutoJsonRpcServiceImplExporter实现了BeanFactoryPostProcessor  初始化完成会执行postProcessBeanFactory
````
#### postProcessBeanFactory代码分析
寻找具体的实现类Map<String, String> servicePathToBeanName = findServiceBeanDefinitions(defaultListableBeanFactory);
获取注解AutoJsonRpcServiceImpl，JsonRpcService  根据注解获取所需要的path(访问路径，默认是JsonRpcService的value值，自己修改为了className)
注册代理类
registerServiceProxy(defaultListableBeanFactory, makeUrlPath(entry.getKey()), entry.getValue());
JsonServiceExporter 实现HtttpHandler,最终容器注册是这个Bean的实例.



















