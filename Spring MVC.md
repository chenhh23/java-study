##### SpringMvc工作流程
![springmvc流程](https://github.com/chenhh23/java-study/blob/master/picture/jdk-dynamic-proxy.png)

SpringMVC运行原理 
1. 客户端请求提交到DispatcherServlet 
2. 由DispatcherServlet控制器查询一个或多个HandlerMapping，找到处理请求的Controller 
3. DispatcherServlet将请求提交到Controller 
4. Controller调用业务逻辑处理后，返回ModelAndView 
5. DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图 
6. 视图负责将结果显示到客户端 

DispatcherServlet是整个Spring MVC的核心。它负责接收HTTP请求组织协调Spring MVC的各个组成部分。其主要工作有以下三项：
1. 截获符合特定格式的URL请求。 
2. 初始化DispatcherServlet上下文对应的WebApplicationContext，并将其与业务层、持久化层的WebApplicationContext建立关联。 
3. 初始化Spring MVC的各个组成组件，并装配到DispatcherServlet中。
