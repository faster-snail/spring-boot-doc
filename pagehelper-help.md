## spring boot 配置添加
application.yml
```yaml
pagehelper:
  helperDialect: mysql
  reasonable: true
  supportMethodsArguments: true
  pageSizeZero: true
  params: count=countSql
```
## spring boot pom.xml 添加
```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.12</version>
</dependency>
```

## 分页查询操作
```java
PageHelper.startPage(1,2);
List<UserEntity> users = userService.selectUsers(25);
PageInfo<UserEntity> pageInfo = new PageInfo<>(users);
```
>说明:  原来查询的是是user列表，现在将user列表缓存到内存中，并使用pageInfo 作为返回列表

## 效果：
```json
{
    "code": 200,
    "message": "ok",
    "data": {
        "total": 6,
        "list": [
            {
                "id": 1,
                "username": "a",
                "age": 25,
                "phone": "11111111111"
            },
            {
                "id": 2,
                "username": "b",
                "age": 25,
                "phone": "2222222222"
            }
        ],
        "pageNum": 1,
        "pageSize": 2,
        "size": 2,
        "startRow": 1,
        "endRow": 2,
        "pages": 3,
        "prePage": 0,
        "nextPage": 2,
        "isFirstPage": true,
        "isLastPage": false,
        "hasPreviousPage": false,
        "hasNextPage": true,
        "navigatePages": 8,
        "navigatepageNums": [
            1,
            2,
            3
        ],
        "navigateFirstPage": 1,
        "navigateLastPage": 3
    }
}
```
