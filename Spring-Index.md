### Reference
- [Spring官方文档](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/)
- [Validation](https://www.ibm.com/developerworks/cn/java/j-lo-beanvalid/index.html)

---
### Core
- **IoC**: The Spring Framework implementation of the Inversion of Control (IoC) which is also known as dependency injection (DI).The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework’s IoC container.
- **BeanFactory**: The BeanFactory interface provides an advanced configuration mechanism capable of managing any type of object.
- **ApplicationContext**: ApplicationContext is a sub-interface of BeanFactory. It adds easier integration with Spring’s AOP features; message resource handling (for use in internationalization), event publication; and application-layer specific contexts such as the WebApplicationContext for use in web applications.<br/>
![image](/uploads/21331b14be9411de379cb03dff0d6ae8/image.png)
- **Resource**: Spring’s Resource interface is meant to be a more capable interface for abstracting access to low-level resources.<br/>
![image](/uploads/4462428cb7301f1bb9ed96d55fed00ce/image.png)
- **BeanPostProcessor**: The BeanPostProcessor interface defines callback methods that you can implement to provide your own (or override the container’s default) instantiation logic, dependency-resolution logic, and so forth. If you want to implement some custom logic after the Spring container finishes instantiating, configuring, and initializing a bean, you can plug in one or more BeanPostProcessor implementations. That is say **`BeanPostProcessors` operate on bean (or object) instances**. A bean post-processor typically checks for callback interfaces or may wrap a bean with a proxy. Some Spring AOP infrastructure classes are implemented as bean post-processors in order to provide proxy-wrapping logic.<br/> 
![image](/uploads/74fe3cea2ab62876750106bc533b6d00/image.png)
<br/>
Using callback interfaces or annotations in conjunction with a custom BeanPostProcessor implementation is a common means of extending the Spring IoC container. For example, `RequiredAnnotationBeanPostProcessor`,`AutowiredAnnotationBeanPostProcessor`.
- **BeanFactoryPostProcessor**: To change the actual bean definition. The semantics of this interface are similar to those of the BeanPostProcessor, with one major difference: BeanFactoryPostProcessor operates on the bean configuration metadata; that is, the Spring IoC container allows a BeanFactoryPostProcessor to read the configuration metadata and potentially change it before the container instantiates any beans other than BeanFactoryPostProcessors. Spring includes a number of predefined bean factory post-processors, such as PropertyOverrideConfigurer and PropertyPlaceholderConfigurer. A custom BeanFactoryPostProcessor can also be used, for example, to register custom property editors.
- **FactoryBean**: Customizing instantiation logic with a FactoryBean. The FactoryBean interface is a point of pluggability into the Spring IoC container’s instantiation logic. If you have complex initialization code that is better expressed in Java as opposed to a (potentially) verbose amount of XML, you can create your own FactoryBean, write the complex initialization inside that class, and then plug your custom FactoryBean into the container.When you need to ask a container for an actual FactoryBean instance itself instead of the bean it produces, preface the bean’s id with the ampersand symbol ( &) when calling the getBean() method of the ApplicationContext. <br/>
![image](/uploads/c65403d703e52a644f97021b39dbede3/image.png)
-  

---
### Configuration
Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control

***Annotation injection is performed before XML injection, thus the latter configuration will override the former for properties wired through both approaches.***

---
#### XML files
<bean/>
#### Java-based configuration
Define beans external to your application classes by using Java rather than XML files.

---
### Resource

#### UrlResource
The UrlResource wraps a java.net.URL, and may be used to access any object that is normally accessible via a URL, such as files, an HTTP target, an FTP target, etc. All URLs have a standardized String representation, such that appropriate standardized prefixes are used to indicate one URL type from another. This includes file: for accessing filesystem paths, http: for accessing resources via the HTTP protocol, ftp: for accessing resources via FTP, etc.

--- 
#### ClassPathResource
This class represents a resource which should be obtained from the classpath. This uses either the thread context class loader, a given class loader, or a given class for loading resources.

--- 
#### FileSystemResource
This is a Resource implementation for java.io.File handles. It obviously supports resolution as a File, and as a URL.

---
#### ServletContextResource
This is a Resource implementation for ServletContext resources, interpreting relative paths within the relevant web application’s root directory.

---
#### InputStreamResource
A Resource implementation for a given InputStream. This should only be used if no specific Resource implementation is applicable. In particular, prefer ByteArrayResource or any of the file-based Resource implementations where possible.

---
#### ByteArrayResource
This is a Resource implementation for a given byte array. It creates a ByteArrayInputStream for the given byte array.

---
#### ResourceLoader
The ResourceLoader interface is meant to be implemented by objects that can return (i.e. load) Resource instances. All application contexts implement the ResourceLoader interface, and therefore all application contexts may be used to obtain Resource instances.
![image](/uploads/4e4740e9d1f660409d69ae57e93044d0/image.png)

---
#### ResourceLoaderAware
The ResourceLoaderAware interface is a special marker interface, identifying objects that expect to be provided with a ResourceLoader reference.When a class implements ResourceLoaderAware and is deployed into an application context (as a Spring-managed bean), it is recognized as ResourceLoaderAware by the application context. The application context will then invoke the setResourceLoader(ResourceLoader), supplying itself as the argument (remember, all application contexts in Spring implement the ResourceLoader interface).
![image](/uploads/5cedfb266d36ecfa4a67f1f39f1f17fe/image.png)

### BeanDefinition
- [class](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-class)
- [name](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-beanname)
- [scope](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-scopes)
- [constructor arguments](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-collaborators)
- [properties](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-collaborators)
- [autowiring mode](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-autowire)
- [lazy-initialization mode](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-lazy-init)
- [initialization method](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean)
- [destruction method](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean)
