Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

spring data jpa让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现

---
### Spring Data JPA

Spring Data repository抽象的中央接口是Repository。它需要一个域类来管理也需要这个域类的id类型作为类型参数。这个接口主要扮演一个标记者接口来捕获需要处理的类型并且帮助你发现拓展的接口。CrudRepository接口为它管理的实体类提供了复杂的CRUD功能。<br/>

---
CrudRepository 接口
```
public interface CrudRepository<T, ID extends Serializable>
    extends Repository<T, ID> {

    // 保存实体
    <S extends T> S save(S entity); 

    // 根据id查询实体
    T findOne(ID primaryKey);       

    // 返回所有实体类
    Iterable<T> findAll();          

    // 返回所有实体类的数量
    Long count();                   

    // 删除指定的实体类
    void delete(T entity);          

    // 根据指定的id判断某个实体是否存在
    boolean exists(ID primaryKey);  

    // … 更多的方法省略
}
```

---
分页接口PagingAndSortingRepository
``` 
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {

  Iterable<T> findAll(Sort sort);

  Page<T> findAll(Pageable pageable);
}
```

---
config
``` 
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@Configuration
@EnableJpaRepositories
class Config {}
```

典型的，你的接口一般会继承Repository, CrudRepository 或者 PagingAndSortingRepository。二者选一，如果你不想继承Spring Data的接口，你可以在你的repository接口增加@RepositoryDefinition注解。继承CrudRepository曝露了一套完整的方法来操作你的实体类。如果你想选择性的曝露一些方法，只需要将你想要曝露的方法从CrudRepository里复制到你的域repository。<br/>

其实Spring data 觉大部分的SQL都可以根据方法名定义的方式来实现，但是由于某些原因我们想使用自定义的SQL来查询，spring data也是完美支持的；在SQL的查询方法上面使用@Query注解，如涉及到删除和修改在需要加上@Modifying.也可以根据需要添加 @Transactional 对事物的支持，查询超时的设置等
```
@Modifying
@Query("update User u set u.userName = ?1 where c.id = ?2")
int modifyByIdAndUserId(String  userName, Long id);
	
@Transactional
@Modifying
@Query("delete from User where id = ?1")
void deleteByUserId(Long id);
  
@Transactional(timeout = 10)
@Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress(String emailAddress);
```

---
@Entity用于标识关系型数据库实体，@Document用于标识nosql类型数据库实体


#### 查询创建策略
下面的策略对于repository底层解析查询是可用的。你可以配置这些策略，在XML配置中使用query-lookup-strategy属性或者使用Enable${store}Repositories注解的queryLookupStrategy属性在java 配置中。一些策略可能不支持某些特别的数据存储。

- **CREATE** 尝试从查询方法的名称构建一个特定存储技术的查询。一般的方法是从查询方法名称中移除一系列约定好的前缀并且解析剩下的字符。

- **USE_DECLARED_QUERY** 尝试去找到一个已声明的查询并且在找不到时抛出一个异常。这个查询可以使用一个注解定义或者通过其他方式来声明。查询特定存储技术的文档来获得更多可用的选项。如果repository底层找不到某个方法对应的已声明的查询，启动时将会失败。

- **CREATE_IF_NOT_FOUND**（默认）组合了CREATE和USE_DECLARED_QUERY策略。它首先查找是否有已经声明的查询语句，如果没有它将会根据方法名称创建一个查询。

---
### Spring Data Elasticsearch
Spring Data Elasticsearch模块包含一个自定义命名空间允许用户定义repository bean和ElasticsearchServer实例。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/data/elasticsearch
http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch-1.0.xsd">

    <elasticsearch:repositories base-package="com.acme.repositories" />
    <!-- 使用Transport Client注册一个Elasticsearch Server实例-->
    <elasticsearch:transport-client id="client" cluster-nodes="localhost:9300,someip:9300" />

</beans>
```

---
基于注解的配置方式
```
@Configuration
@EnableElasticsearchRepositories(basePackages = "com/acme/repositories")
static class Config {

    @Bean
    public ElasticsearchOperations elasticsearchTemplate() {
        return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
    }
}
```
		
| 关键字 | 示例 | Elasticsearch json查询 |
|----|----|----|
| And |	findByNameAndPrice |	{"bool" : {"must" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}} |
| Or |	findByNameOrPrice |	{"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"price" : "?"}} ]}} |
| Is |	findByName |	{"bool" : {"must" : {"field" : {"name" : "?"}}}} |
| Not |	findByNameNot |	{"bool" : {"must_not" : {"field" : {"name" : "?"}}}} |
| Between |	findByPriceBetween |	{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| LessThanEqual |	findByPriceLessThan |	{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| GreaterThanEqual |	findByPriceGreaterThan |	{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}} |
| Before |	findByPriceBefore |	{"bool" : {"must" : {"range" : {"price" : {"from" : null,"to" : ?,"include_lower" : true,"include_upper" : true}}}}} |
| After |	findByPriceAfter |	{"bool" : {"must" : {"range" : {"price" : {"from" : ?,"to" : null,"include_lower" : true,"include_upper" : true}}}}} |
| Like |	findByNameLike |	{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}} |
| StartingWith |	findByNameStartingWith |	{"bool" : {"must" : {"field" : {"name" : {"query" : "?*","analyze_wildcard" : true}}}}} |
| EndingWith |	findByNameEndingWith |	{"bool" : {"must" : {"field" : {"name" : {"query" : "*?","analyze_wildcard" : true}}}}} |
| Contains/Containing |	findByNameContaining |	{"bool" : {"must" : {"field" : {"name" : {"query" : "?","analyze_wildcard" : true}}}}} |
| In |	findByNameIn(Collection<String>names) |	{"bool" : {"must" : {"bool" : {"should" : [ {"field" : {"name" : "?"}}, {"field" : {"name" : "?"}} ]}}}} |
| NotIn	 | findByNameNotIn(Collection<String>names) |	{"bool" : {"must_not" : {"bool" : {"should" : {"field" : {"name" : "?"}}}}}} |
| Near |	findByStoreNear |	暂不支持 |
| True |	findByAvailableTrue |	{"bool" : {"must" : {"field" : {"available" : true}}}} |
| False |	findByAvailableFalse |	{"bool" : {"must" : {"field" : {"available" : false}}}} |
| OrderBy |	findByAvailableTrueOrderByNameDesc |	{"sort" : [{ "name" : {"order" : "desc"} }],"bool" : {"must" : {"field" : {"available" : true}}}} |

#### 条件构造器

```
@NoRepositoryBean
public interface ElasticsearchRepository<T, ID extends Serializable> extends ElasticsearchCrudRepository<T, ID> {
  <S extends T> S index(S var1);

  Iterable<T> search(QueryBuilder var1);

  Page<T> search(QueryBuilder var1, Pageable var2);
  /**
    SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(matchAllQuery())
    .withFilter(boolFilter().must(termFilter("id", documentId)))
    .build();
   */
  Page<T> search(SearchQuery var1);

  Page<T> searchSimilar(T var1, String[] var2, Pageable var3);

  void refresh();

  Class<T> getEntityClass();
}
```
