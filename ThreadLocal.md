### Reference
- [CSDN详解](https://blog.csdn.net/shmilychan/article/details/69193624)
- [简书详解](https://www.jianshu.com/p/4ce4b56134fb)
- [iteye详解](http://www.iteye.com/topic/103804)

### 使用场景
ThreadLocal, 中文翻译是线程局部变量。 目的是对于某个成员变量，每个线程都持有该成员变量的一个初始化副本，各个线程之间对变量的修改相互之间不影响。ThreadLocal 不是用来解决共享对象的多线程访问问题的，一般情况下，通过ThreadLocal.set() 到线程中的对象是该线程自己使用的对象，其他线程是不需要访问的，也访问不到的。

---
#### ThreadLocal
通过查看ThreadLocal类源码，该类中提供了两个主要的方法 get() 和set()，还有一个用于回收本地变量中的方法remove()也是常用的如下：
```
  /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }


    /**
     * Removes the current thread's value for this thread-local
     * variable.  If this thread-local variable is subsequently
     * {@linkplain #get read} by the current thread, its value will be
     * reinitialized by invoking its {@link #initialValue} method,
     * unless its value is {@linkplain #set set} by the current thread
     * in the interim.  This may result in multiple invocations of the
     * {@code initialValue} method in the current thread.
     *
     * @since 1.5
     */
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

#### ThreadLocalMap 
ThreadLocalMap是ThreadLocal类的一个静态内部类，它实现了键值对的设置和获取（类似于 Map<K,V> 存储的key-value），每个线程中都有一个独立的ThreadLocalMap，它所存储的值，只能被当前线程读取和修改。ThreadLocal类通过操作每一个线程特有的 ThreadLocalMap 副本，从而实现了变量访问在不同线程中隔离。因为每个线程的变量都是自己特有的，完全不会有并发错误。ThreadLocalMap的Key是ThreadLocal，而Value是被初始化的副本。

#### How to use
通过声明一个静态的ThreadLocal。<br/>
`private static ThreadLocal<Connection> threadLocal = new ThreadLocal<Connection>()` <br/>

1. 每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。 
2. 将一个共用的ThreadLocal静态实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静态ThreadLocal实例的get()方法取得自己线程保存的那个对象，避免了将这个对象作为参数传递的麻烦。 

当用户需要调用连接数据库的时候，只需要通过 DatabaseConnection.getConnection()连接数据库，在实现业务逻辑的时候，每次调用都会为创建一个连接数据库的线程副本，每次所作的修改互不干扰，在多用户并发访问使用的时候，很好的避免了使用同步带来的性能问题。
```
package cn.czl.util.dbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * 为了更加方便的传输关键的引用对象（Connection），所以在本类之中将利用ThreadLocal进行数据库连接的保存
 * 这个类中的ThreadLocal应该作为一个公共的数据，公共的数据使用static声明
 */
public class DatabaseConnection  {
    private static final String DBDRIVER = "oracle.jdbc.driver.OracleDriver" ;
    private static final String DBURL = "jdbc:oracle:thin:@localhost:1521:orcl" ;
    private static final String USER = "scott" ;
    private static final String PASSWORD = "tiger" ;
    private static ThreadLocal<Connection> threadLocal = new ThreadLocal<Connection>() ;
    /**
     * 取得数据库的连接对象，如果在取得的时候没有进行连接则自动创建新的连接
     * 但是需要防止同一个线程可能重复调用此操作的问题
     * @return
     */
    public static Connection getConnection() {
        Connection conn = threadLocal.get() ;   // 首先判断一下在ThreadLocal里面是否保存有当前连接对象
        if (conn == null) { // 如果此时的连接对象是空，那么就表示没有连接过，则创建一个新的连接
            conn = rebuildConnection() ;    // 创建新的连接
            threadLocal.set(conn);  // 保存到ThreadLocal之中，以便一个线程执行多次数据库的时候使用
        }
        return conn ;   // 返回连接对象
    }
    /**
     * 重新建立新的数据库连接
     * @return Connection接口对象
     */
    private static Connection rebuildConnection() { // 创建新的连接对象
        try {
            Class.forName(DBDRIVER) ;
            return DriverManager.getConnection(DBURL,USER,PASSWORD); 
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null ;
    }
    /**
     * 关闭数据库连接
     */
    public static void close() {
        System.out.println(Thread.currentThread().getName() + " 关闭数据库连接。");
        Connection conn = threadLocal.get() ;   // 取得数据库连接
        if (conn != null) {
            try {
                conn.close();
                threadLocal.remove();   // 从ThreadLocal中删除掉保存的数据
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```
