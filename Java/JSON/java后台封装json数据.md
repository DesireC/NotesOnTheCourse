[<img src="../../index.jpg" width = "80" height = "80"  />](../../index.md#index)

<h1 id="javajson">java 后台封装json数据学习总结</h1>

 

一、数据封装

1. List集合转换成json代码

   ```java
   List list = new ArrayList();
   list.add( "first" );
   list.add( "second" );
   JSONArray jsonArray2 = JSONArray.fromObject( list );
   ```

2. Map集合转换成json代码

   ```java
   Map map = new HashMap();
   map.put("name", "json");
   map.put("bool", Boolean.TRUE);
   map.put("int", new Integer(1));
   map.put("arr", new String[] { "a", "b" });
   map.put("func", "function(i){ return this.arr[i]; }");
   JSONObject json = JSONObject.fromObject(map);
   ```
   
3. Bean转换成json代码

   ```java
   JSONObject jsonObject = JSONObject.fromObject(new JsonBean());
   ```

   

4. 数组转换成json代码

   ```java
   boolean[] boolArray = new boolean[] { true, false, true };
   JSONArray jsonArray1 = JSONArray.fromObject(boolArray);
   ```


5. 一般数据转换成json代码

   ```java
   JSONArray jsonArray3 = JSONArray.fromObject("['json','is','easy']" );
   ```

二、JAR包简介

​      在你的应用中加入引入JSON-lib包，JSON-lib包同时依赖于以下的JAR包：

下载地址:[http://json-lib.sourceforge.net/](http://json-lib.sourceforge.net/)

​      1.commons-lang.jar

​      2.commons-beanutils.jar

​      3.commons-collections.jar

​      4.commons-logging.jar 

​      5.ezmorph.jar

​      6.json-lib-2.2.2-jdk15.jar

用法同上

```java
JSONObject jsonObject = JSONObject.fromObject(message);
getResponse().getWriter().write(jsonObject.toString());
```

当把数据转为json后，用如上的方法发送到客户端。前端就可以取得json数据了。

也可以用

```java
List  list1 = new ArrayList<ListDate>()
    
ListDate ListDate2 = new ListDate();
ListDate2.setId(examSubject.getId());
ListDate2.setValue(examSubject.getSubjectName());

list1.add(ListDate2);

JSONArray jsonArray1 = JSONArray.fromObject(list1);
```

前台循环取

```js
$.each(date, function(i, obj) {
   $("#examName").append("<option value='" + obj.id + "'>"+ obj.value+ "</option>");
});
```

三、JSONObject对象使用

​     JSON-lib包是一个beans,collections,maps,java arrays 和XML和JSON互相转换的包。在本例中，我们将使用JSONObject类创建JSONObject对象，然后我们打印这些对象的值。为了使用JSONObject对象，我们要引入"net.sf.json"包。为了给对象添加元素，我们要使用put()方法。

样例：

```java
import net.sf.json.JSONArray;
import net.sf.json.JSONObject;
 
public class JSONObjectSample {
    //创建JSONObject对象   
    private static JSONObject createJSONObject(){   
        JSONObject jsonObject = new JSONObject();   
        jsonObject.put("username","huangwuyi");   
        jsonObject.put("sex", "男");   
        jsonObject.put("QQ", "999999999");   
        jsonObject.put("Min.score", new Integer(99));   
        jsonObject.put("nickname", "梦中心境");   
        return jsonObject;   
    }   
    public static void main(String[] args) {   
        JSONObject jsonObject = JSONObjectSample.createJSONObject();   
        //输出jsonobject对象   
        System.out.println("添加属性前的对象jsonObject==>\n"+jsonObject);   
        //判读输出对象的类型   
        boolean isArray = jsonObject.isArray();   
        boolean isEmpty = jsonObject.isEmpty();   
        boolean isNullObject = jsonObject.isNullObject();   
        System.out.println("isArray:"+isArray+" isEmpty:"+isEmpty+" isNullObject:"+isNullObject);  
        //添加属性   
        jsonObject.element("address", "福建省厦门市");   
        System.out.println("添加属性后的对象jsonObject==>\n"+jsonObject);   
 
        //返回一个JSONArray对象   
        JSONArray jsonArray = new JSONArray();   
        jsonArray.add(0, "this is a jsonArray value");   
        jsonArray.add(1,"another jsonArray value");   
        jsonObject.element("jsonArray", jsonArray);   
        JSONArray array = jsonObject.getJSONArray("jsonArray");   
        System.out.println("返回一个JSONArray对象==>\n"+array);   
        //添加JSONArray后的值   
        System.out.println("结果==>\n"+jsonObject);   
            
        //根据key返回一个字符串   
        String username = jsonObject.getString("username");   
        System.out.println("username==>"+username);  
         
        //把字符转换为 JSONObject
        String temp=jsonObject.toString();
        JSONObject object = JSONObject.fromObject(temp);
        //转换后根据Key返回值
        System.out.println("qq==>"+object.get("QQ"));
    }   
}
```
运行结果：
```json
添加属性前的对象jsonObject==>
{"username":"huangwuyi",
 "sex":"男",
 "QQ":"999999999",
 "Min.score":99,
 "nickname":"梦中心境"}
isArray:false 
isEmpty:false 
isNullObject:false
添加属性后的对象jsonObject==>
{"username":"huangwuyi",
 "sex":"男",
 "QQ":"999999999",
 "Min.score":99,
 "nickname":"梦中心境",
 "address":"福建省厦门市"}
返回一个JSONArray对象==>
["this is a jsonArray value", "another jsonArray value"]
结果==>
{"username":"huangwuyi",
 "sex":"男",
 "QQ":"999999999",
 "Min.score":99,
 "nickname":"梦中心境",
 "address":"福建省厦门市",
 "jsonArray":["this is a jsonArray value","another jsonArray value"]}
username==>huangwuyi
qq==>999999999
```