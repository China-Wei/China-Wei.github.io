### 什么是ES？

Elasticsearch（简称为ES）是一个开源**的分布式搜索引擎，用于全文搜索、实时分析和可视化。** 它建立在Apache Lucene搜索引擎库的基础上，提供了RESTful API，支持分布式架构和水平扩展，特别适用于处理大规模的非结构化或半结构化数据。

#### Elasticsearch与传统数据库查询的区别：

  * **搜索引擎特性：** Elasticsearch是一个搜索引擎，其主要设计目标是支持高效的**全文搜索和实时分析。** 它专注于处理大量文本数据，支持复杂的全文搜索查询，例如模糊搜索、词组匹配、范围查询等。传统数据库主要面向结构化数据，更**适用于关系型查询** 。

  * **分布式和水平扩展：** Elasticsearch是分布式的，它可以在多个节点上运行，并能够水平扩展以处理大规模数据。传统数据库通常在单个节点上运行，并且通常需要垂直扩展（增加服务器性能）来处理更大的负载。

  * **Schema-less（无模式）：** Elasticsearch是无模式的，意味着你不需要在索引中预定义数据结构。你可以直接插入数据，Elasticsearch会根据数据自动推断其类型。传统数据库需要在创建表时定义模式。

  * **实时性：** Elasticsearch提供实时索引和搜索功能，数据几乎立即可用于搜索。传统数据库的实时性可能受到数据复制和索引的影响。

  * **多语言支持：** Elasticsearch支持多种查询语言，例如查询字符串、DSL（Domain Specific Language）等，使其更易于使用和集成。传统数据库通常使用SQL。

要在Spring Boot项目中集成Elasticsearch，首先需要添加相应的依赖，然后配置Elasticsearch的连接信息，最后通过Spring Data Elasticsearch或者Elasticsearch的Java REST客户端来与Elasticsearch进行交互。

### SpringBoot整合ElasticSearch的步骤：

#### 0.代码层级结构

<img src='https://img-blog.csdnimg.cn/direct/619e5e4fc6804cf7b49b4d7b9e860c6b.png' width='436' height='691' />

#### 1\. 添加依赖

在pom.xml中添加Spring Data Elasticsearch依赖：

    
    
     <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
     </dependency>

#### 2\. 配置Elasticsearch连接信息

在application.properties或(application.yml)中配置Elasticsearch连接信息，如下：

    
    
    spring.elasticsearch.uris=http://localhost:9200
    
    
    spring:
      elasticsearch:
        uris: http://localhost:9200
    
    

如果Elasticsearch没有设置用户名和密码，这可能就足够了。如果需要认证，可以使用以下配置：

    
    
    spring.elasticsearch.username=your_username
    spring.elasticsearch.password=your_password
    
    
    spring:
      elasticsearch:
        uris: http://localhost:9200
        username:your_username
        password:your_password

#### 3\. 创建实体类

创建一个简单的实体类，使用@Document注解指定索引和类型：

    
    
    import lombok.Data;
    import org.springframework.data.annotation.Id;
    import org.springframework.data.elasticsearch.annotations.Document;
    
    /**
     * @Document(indexName = "user-demo") 索引名称为user-demo
     * */
    @Data
    @Document(indexName = "user-demo")
    public class User {
        @Id
        private String id;
        private String username;
        private String address;
        private Integer age;
        private String gender;
    }
    

#### 4\. 使用Spring Data Elasticsearch进行操作

创建一个UserMapper接口，继承自ElasticsearchRepository：

    
    
    import com.example.demoes.entity.User;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
    import org.springframework.stereotype.Repository;
    
    import java.util.List;
    /**
     * 这些方法的名称遵循一种命名约定，这种约定将方法名中的关键词（如 findBy、Like、IgnoreCase 等）与实体类（User）的属性名相关联。
     * 这样，Spring Data Elasticsearch 可以根据方法名自动生成相应的查询。
     *
     * 在Spring Data Elasticsearch中，查询方法的命名约定依赖于一些关键词和命名规范，这些关键词用于描述要执行的查询操作。以下是一些常见的关键词和它们的含义：
     *
     *     findBy：指定查询的条件的前缀，后面通常跟随实体类的属性名。例如，findByUsername 表示要根据用户名进行查询。
     *
     *     Like：用于模糊查询，通常与属性名一起使用。例如，findByUsernameLike 表示要进行用户名的模糊查询。
     *
     *     IgnoreCase：用于不区分大小写的查询，通常与属性名一起使用。例如，findByUsernameIgnoreCase 表示不区分大小写地查询用户名。
     *
     *     Containing：用于包含查询，通常与属性名一起使用。例如，findByUsernameContaining 表示包含指定内容的查询。
     *
     *     And 和 Or：用于构建复合查询条件。例如，findByUsernameAndAge 表示根据用户名和年龄进行查询。
     *
     *     OrderBy：用于指定结果的排序方式。例如，findByAgeOrderByUsernameDesc 表示根据年龄查询并按用户名降序排序。
     *
     *     Page 和 Iterable：用于指定查询结果的返回类型。例如，Page<User> 表示返回分页结果，Iterable<User> 表示返回一个列表。
     *
     *     其他属性名：您可以根据实体类的属性名来定义查询条件。例如，如果实体类有一个属性为 email，您可以编写 findByEmail 方法来根据邮箱进行查询。
     * */
    @Repository
    public interface UserMapper extends ElasticsearchRepository<User,String> {
    
        /**
         *自定义查询
         * */
        List<User> findByAddressLikeIgnoreCaseOrderByAgeDesc(String address);
    
        Page<User> findByUsernameContaining(String username, Pageable page);
    
        User findByAge(Integer age);
    
        List<User> findByGender(String gender);
    
        long count();
    
        Boolean existsById();
    
    }
    

#### 5.测试

在测试中使用Spring Boot的@SpringBootTest注解，来验证Elasticsearch是否正常工作。

    
    
    import com.example.demoes.entity.User;
    import com.example.demoes.mapper.UserMapper;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate;
    
    
    import java.util.*;
    
    
    @SpringBootTest
    class DemoEsApplicationTests {
    
        //    @Resource
        @Autowired
        private UserMapper userMapper;
    
    
        @Autowired
        private ElasticsearchRestTemplate elasticsearchRestTemplate;
    
    
        @Test
        void add() {
            for (int i = 2; i < 11; i++) {
                User user = new User();
                user.setId(i + "");
                user.setUsername("zhangsan" + i);
                user.setGender("男");
                user.setAge(i + i);
                user.setAddress("河北省" + i);
                //直接将数据保存到es中，索引名称为user-demo，在user对象中设置的
                //通过kibana进行查询GET /user-demo/_search
                User save = userMapper.save(user);
                System.out.println("save = " + save);
            }
        }
    
    
        @Test
        void delete() {
            String id = "9";
            userMapper.deleteById(id);
        }
    
        @Test
        void update() {
    
            User updateUser = new User();
            updateUser.setId("9");
            updateUser.setUsername("Lisi");
            updateUser.setGender("女");
            updateUser.setAddress("北京市");
            updateUser.setAge(39);
    
            User save = userMapper.save(updateUser);
            System.out.println("save = " + save);
        }
    
        @Test
        void find() {
            //模糊查询。精确查询
            Iterable<User> all = userMapper.findAll();
            List<User> list = new ArrayList<>();
            for (User user : all) {
                list.add(user);
            }
            // 使用Collections.sort方法按照用户的ID进行排序
    //        Collections.sort(list, Comparator.comparing(User::getId));
    
            // 使用自定义比较器按照用户的ID进行排序
            Collections.sort(list, new Comparator<User>() {
                @Override
                public int compare(User user1, User user2) {
                    // 假设ID是字符串类型
                    return user1.getUsername().compareTo(user2.getUsername());
                }
            });
    
            // 使用流（Stream）按照用户的ID进行排序
            /*List<User> sortedList = list.stream()
                    .sorted(Comparator.comparing(User::getId))
                    .collect(Collectors.toList());*/
            // 现在list中的用户已按照ID升序排列
            for (User user : list) {
                System.out.println("user = " + user);
            }
    
        }
    
        @Test
        void findById() {
            //注意这里的id是字符串类型，然后userMapper中继承的id也必须是String类型
            Optional<User> byId = userMapper.findById("4");
            System.out.println("byId = " + byId);
        }
    
        @Test
        void findCount() {
            long count = userMapper.count();
            System.out.println("count 数 " + count);
        }
    
        @Test
        void existsById() {
            System.out.println("id是否存在： " + userMapper.existsById("1"));
        }
    
        @Test
        void findByIdAll() {
    
            ArrayList<String> ids = new ArrayList<>();
            ids.add("3");
            ids.add("5");
            ids.add("4");
            ids.add("66");
    
            //通过ids（集合）进行查询对应的user集合对象
            Iterable<User> allById = userMapper.findAllById(ids);
            for (User user : allById) {
                System.out.println("user = " + user);
            }
        }
    
    
        /**
         * 自定义查询
         */
        @Test
        void findByUsernameLike() {
            //模糊查询，会通过命名规范进行一个查询
            List<User> addr = userMapper.findByAddressLikeIgnoreCaseOrderByAgeDesc("河北");
            System.out.println("addr = " + addr);
    
            Page<User> page = userMapper.findByUsernameContaining("4", PageRequest.of(0, 100));
            System.out.println("page = " + page.getContent());
    
            User zhangList = userMapper.findByAge(16);
            System.out.println("user = " + zhangList);
    
            List<User> gender = userMapper.findByGender("女");
            System.out.println("gender = " + gender);
    
        }
    
    

以上步骤是一个简单的Spring Boot集成Elasticsearch的示例。你可以根据你的实际需求进行更复杂的配置和操作。

