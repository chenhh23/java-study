### Reference
- [Spring AOP and Cglib](https://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib/index.html)
- [JDK Proxy官方文档](https://docs.oracle.com/javase/7/docs/api/java/lang/reflect/Proxy.html)

---
### Cglib
CGLIB是一个功能强大，高性能的代码生成包。它为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。通常可以使用Java的动态代理创建代理，但当要代理的类没有实现接口或者为了更好的性能，CGLIB是一个好的选择。 <br/>

Spring AOP 框架对 AOP 代理类的处理原则是：如果目标对象的实现类实现了接口，Spring AOP 将会采用 JDK 动态代理来生成 AOP 代理类；如果目标对象的实现类没有实现接口，Spring AOP 将会采用 CGLIB 来生成 AOP 代理类——不过这个选择过程对开发者完全透明、开发者也无需关心。<br/>

JDK中的动态代理是通过反射类Proxy以及InvocationHandler回调接口实现的，
但是，JDK中所要进行动态代理的类必须要实现一个接口，也就是说只能对该类所实现接口中定义的方法进行代理，这在实际编程中具有一定的局限性，而且使用反射的效率也并不是很高。

---
#### Callback
定义一个拦截器。在调用目标方法时，CGLib会回调MethodInterceptor接口方法拦截，来实现你自己的代理逻辑，类似于JDK中的InvocationHandler接口。
参数：Object为由CGLib动态生成的代理类实例，Method为上文中实体类所调用的被代理的方法引用，Object[]为参数值列表，MethodProxy为生成的代理类对方法的代理引用。
```
public class TargetInterceptor implements MethodInterceptor{
 
	/**
	 * 重写方法拦截在方法前和方法后加入业务
	 * Object obj为目标对象
	 * Method method为目标方法
	 * Object[] params 为参数，
	 * MethodProxy proxy CGlib方法代理对象
	 */
	@Override
	public Object intercept(Object obj, Method method, Object[] params,
			MethodProxy proxy) throws Throwable {
		System.out.println("调用前");
		Object result = proxy.invokeSuper(obj, params);
		System.out.println(" 调用后"+result);
		return result;
	}
 
 
}
```

---
#### CallbackFilter
在CGLib回调时可以设置对不同被代理方法执行不同的回调逻辑，或者根本不执行回调。
```
public class TargetMethodCallbackFilter implements CallbackFilter {
 
	/**
	 * 过滤方法
	 * 返回的值为数字，代表了Callback数组中的索引位置，要到用的Callback
	 */
	@Override
	public int accept(Method method) {
		if(method.getName().equals("method1")){
			System.out.println("filter method1 ==0");
			return 0;
		}
		if(method.getName().equals("method2")){
			System.out.println("filter method2 ==1");
			return 1;
		}
		if(method.getName().equals("method3")){
			System.out.println("filter method3 ==2");
			return 2;
		}
		return 0;
	}
 
}
```

---
#### 延迟加载对象
- LazyLoader 只在第一次访问延迟加载属性时触发代理类回调方法
- Dispatcher 在每次访问延迟加载属性时都会触发代理类回调方法

```
public class LazyBean {
	private PropertyBean propertyBean;
	private PropertyBean propertyBeanDispatcher;
 
	public LazyBean(String name, int age) {
		this.propertyBean = createPropertyBean();
		this.propertyBeanDispatcher = createPropertyBeanDispatcher();
	}
 
	
 
	/**
	 * 只第一次懒加载
	 * @return
	 */
	private PropertyBean createPropertyBean() {
		/**
		 * 使用cglib进行懒加载 对需要延迟加载的对象添加代理，在获取该对象属性时先通过代理类回调方法进行对象初始化。
		 * 在不需要加载该对象时，只要不去获取该对象内属性，该对象就不会被初始化了（在CGLib的实现中只要去访问该对象内属性的getter方法，
		 * 就会自动触发代理类回调）。
		 */
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(PropertyBean.class);
		PropertyBean pb = (PropertyBean) enhancer.create(PropertyBean.class,
				new ConcreteClassLazyLoader());
		return pb;
	}
	/**
	 * 每次都懒加载
	 * @return
	 */
	private PropertyBean createPropertyBeanDispatcher() {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(PropertyBean.class);
		PropertyBean pb = (PropertyBean) enhancer.create(PropertyBean.class,
				new ConcreteClassDispatcher());
		return pb;
	}
}

public class ConcreteClassLazyLoader implements LazyLoader {
	/**
	 * 对需要延迟加载的对象添加代理，在获取该对象属性时先通过代理类回调方法进行对象初始化。
	 * 在不需要加载该对象时，只要不去获取该对象内属性，该对象就不会被初始化了（在CGLib的实现中只要去访问该对象内属性的getter方法，
	 * 就会自动触发代理类回调）。
	 */
	@Override
	public Object loadObject() throws Exception {
		System.out.println("before lazyLoader...");
		PropertyBean propertyBean = new PropertyBean();
		System.out.println("after lazyLoader...");
		return propertyBean;
	}
 
}
  
public class ConcreteClassDispatcher implements Dispatcher{
        /**
	 * 对需要延迟加载的对象添加代理，在获取该对象属性时先通过代理类回调方法进行对象初始化。
	 * 在不需要加载该对象时，只要不去获取该对象内属性，该对象就不会被初始化了（在CGLib的实现中只要去 
         * 访问该对象内属性的getter方法，就会自动触发代理类回调，每次获取getter方法时，都会去加载）。
	 *
	 */
	@Override
	public Object loadObject() throws Exception {
		System.out.println("before Dispatcher...");
		PropertyBean propertyBean = new PropertyBean();
		System.out.println("after Dispatcher...");
		return propertyBean;
	}
 
}
```

---
#### 接口生成器
InterfaceMaker会动态生成一个接口，该接口包含指定类定义的所有方法。

```
public class TestInterfaceMaker {
 
	public static void main(String[] args) throws Exception {
		InterfaceMaker interfaceMaker =new InterfaceMaker();
		//抽取某个类的方法生成接口方法
		interfaceMaker.add(TargetObject.class);
		Class<?> targetInterface=interfaceMaker.create();
		for(Method method : targetInterface.getMethods()){
			System.out.println(method.getName());
		}
		//接口代理并设置代理接口方法拦截
		Object object = Enhancer.create(Object.class, new Class[]{targetInterface}, new MethodInterceptor(){
			@Override
			public Object intercept(Object obj, Method method, Object[] args,
					MethodProxy methodProxy) throws Throwable {
				if(method.getName().equals("method1")){
					System.out.println("filter method1 ");
					return "mmmmmmmmm";
				}
				if(method.getName().equals("method2")){
					System.out.println("filter method2 ");
					return 1111111;
				}
				if(method.getName().equals("method3")){
					System.out.println("filter method3 ");
					return 3333;
				}
				return "default";
			}});
	}
}
```

---
### JDK Dynamic Proxy
通过`InvocationHandler`和`Proxy`来实现针对于实现了接口的类进行动态代理
具体做法如下:
- 首先实现一个InvocationHandler，方法调用会被转发到该类的invoke()方法。
- 然后在需要使用Hello的时候，通过JDK动态代理获取Hello的代理对象。
![image](https://github.com/chenhh23/java-study/blob/master/picture/jdk-dynamic-proxy.png)
```
class MyInvocatioHandler implements InvocationHandler {
    private Object target;

    public MyInvocatioHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("-----before-----");
        Object result = method.invoke(target, args);
        System.out.println("-----end-----");
        return result;
    }
    // 生成代理对象
    public Object getProxy() {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        Class<?>[] interfaces = target.getClass().getInterfaces();
        return Proxy.newProxyInstance(loader, interfaces, this);
    }
}
```

代理对象的生成过程由Proxy类的newProxyInstance方法实现，分为3个步骤：
1. ProxyGenerator.generateProxyClass方法负责生成代理类的字节码。
2. native方法Proxy.defineClass0负责字节码加载的实现，并返回对应的Class对象。
3. 利用clazz.newInstance反射机制生成代理类的对象。

--- 
JDK动态代理局限性
通过反射类Proxy和InvocationHandler回调接口实现的jdk动态代理，要求委托类必须实现一个接口，但事实上并不是所有类都有接口，对于没有实现接口的类，便无法使用该方方式实现动态代理。
