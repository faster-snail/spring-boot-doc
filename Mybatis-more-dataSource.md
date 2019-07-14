## Mybatis 多数据源配置

### 第一步：application.yml 添加mysql配置
```yaml
spring:
  datasource:
    db1:
      jdbc-url: jdbc:mysql://localhost:3306/a?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
    db2:
      jdbc-url: jdbc:mysql://localhost:3306/b?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
```
### 第二步：创建java配置

Db1DataSourceConfig.java
```java
@Configuration
@MapperScan(basePackages = "com.ops.moreds.db1.dao", sqlSessionTemplateRef = "db1SqlSessionTemplate")
public class Db1DataSourceConfig {

    /**
     * 生成数据源.  @Primary 注解声明为默认数据源
     */
    @Bean(name = "db1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    /**
     * 创建 SqlSessionFactory
     */
    @Bean(name = "db1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("db1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        //  bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/db1/*.xml"));
        return bean.getObject();
    }

    /**
     * 配置事务管理
     */
    @Bean(name = "db1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("db1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "db1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("db1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```
Db2DataSourceConfig.java
```java

@Configuration
@MapperScan(basePackages = "com.ops.moreds.db2.dao", sqlSessionTemplateRef = "db2SqlSessionTemplate")
public class Db2DataSourceConfig {

    /**
     * 生成数据源.  @Primary 注解声明为默认数据源
     */
    @Bean(name = "db2DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    /**
     * 创建 SqlSessionFactory
     */
    @Bean(name = "db2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("db2DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        //  bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/db1/*.xml"));
        return bean.getObject();
    }

    /**
     * 配置事务管理
     */
    @Bean(name = "db2TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("db2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "db2SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("db2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

缺少的包自己引入即可！

### 第三步：根据上面的配置地址，创建包和mapper
```shell
mkdir com.ops.moreds.db1.dao -p
mkdir com.ops.moreds.db1.entity -p
mkdir com.ops.moreds.db1.service -p
mkdir com.ops.moreds.db2.dao -p
mkdir com.ops.moreds.db2.entity -p
mkdir com.ops.moreds.db2.service -p
```
### 第四步：配置dao，实例使用Mybatis基于注解的方式
#### 实例db1操作
com.ops.moreds.db1.dao.UserDao.java
```java
@Service
public interface UserDao {
    // Insert
    @Insert("INSERT INTO user(username, age, phone) VALUES (#{username},#{age},#{phone})")
    public void insertOneUser(@Param("username") String username,
                              @Param("age") int age,
                              @Param("phone") String phone);

    @Select("SELECT id,username,age,phone FROM user WHERE age = #{age}")
    public List<UserEntity> getUsers(@Param("age") Integer age);
}
```
com.ops.moreds.db1.dao.UserEntity.java 
```java
@Data
public class UserEntity {
    private Integer id;
    private String username;
    private Integer age;
    private String phone;
}
```
> 使用了Lombok插件

com.ops.moreds.db1.dao.UserService.java
```java
@Service
public class UserServiceImpl {
    @Autowired
    UserDao userDao;
    public void saveUser(String username,int age,String phone){
        userDao.insertOneUser(username,age,phone);
    }
    public List<UserEntity> selectUsers(Integer age){
        List<UserEntity> users = userDao.getUsers(age);
        return users;
    }
}
```
> 比较简单的service
#### db2 照葫芦画瓢
...

> 此过程适用于作者个人使用，由于本人java技术非常基础，因此，目录规划可能有错误，但是能跑起来

