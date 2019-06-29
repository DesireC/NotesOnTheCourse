[<img src="../../index.jpg" width = "80" height = "80"  />](../../index.md#index)


<h1 id="zidong">Mybatis获取自动增长Id</h1>

<h4>MyBatis成功插入后获取自动增长的id</h4>

<h5>1、向xxMapping.xml配置中加上两个配置。</h5>

```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id" parameterType="UserEntity">
		INSERT INTO USER VALUES(null,#{userName},#{password},#{realName})
	</insert>
```

其中keyProperty的值就是数据库中自增长字段名。

<h5>2、在Controller插入方法中，插入成功后，直接通过model的get的方法就能获得自增长的id值。</h5>

```java
@RequestMapping("addUser")
public String addUser(@ModelAttribute UserEntity userEntity) {
    int i = userService.insertUser(userEntity);//插入记录到数据库，userEntity中没有设置id的值
    String result = "";
    if (i > 0) {
        result = "inster User SUCCESS!!! ID: " + userEntity.getId();//插入成功后，将自增长的id存入到原来的model中，通过get方法就能拿到自增长的id了
    } else {
        result = "inster User FAIL！！！";
    }
    return result;
}
```

