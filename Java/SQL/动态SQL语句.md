[<img src="../../index.jpg" width = "80" height = "80"  />](../../index.md#index)

<h1 id="dtsql">动态SQL语句</h1>

有些时候，sql语句 where条件中，需要一些安全判断，例如按某一条件查询时如果
传入的参数是空，此时查询出的结果很可能是空的，也许我们需要参数为空时，是
查出全部的信息。使用 Oracle的序列、mysql的函数生成 Id。这时我们可以使用动态
sql。
下文均采用 mysql语法和函数（例如字符串链接函数 CONCAT）。

<h3>1、selectKey标签</h3>

在 insert语句中，在 Oracle经常使用序列、在 MySQL中使用函数来自动生成插入表
的主键，而且需要方法能返回这个生成主键。使用 myBatis的 selectKey标签可以实现
这个效果。
下面例子，使用 mysql数据库自定义函数 nextval('student')，用来生成一个 key，并把他
设置到传入的实体类中的 studentId属性上。所以在执行完此方法后，边可以通过
这个实体类获取生成的 key。

*xml代码*

```xml
<!--插入学生 自动主键--> 
<insert id="createStudentAutokey" parameterType="liming.student.data.model.StudentEntity" keyProperty="studentId">
    <selectkey keyProperty="studentId" resultType="String" order="BEEORE">
    	select nextval('student')
    </selectkey>
    INSERT INTO STUDENT_TBL(STUDENT_ID,
    STUDENT_NAME,
    STUDENT_SEX,
    STUDENT_BIRTHDAY,
    STUDENT_PHOTO,
    CLASSS_ID,
    PLACE_ID)
    VALUES(#{studentId},
    #{studentName},
    #{studentSex},
    #{studentBirthday},
    #{studentPhoto, javaType=byte[], jdbc=BLOB, typeHandler=org.apache.ibatis.type.BlobTypeHandler},
    #{classId},
    #{placeId})
</insert>
```

调用接口方法和获取自动生成key

*java代码*

```java
StudentEntity entity = new StudentEntity();
entity.setStudentName("你好");
entity.setStudentSex(1);
entity.setStudentBirthday(DateUtil.parse("1985-05-28"));
entity.setClassId("20001");
entity.setPlaceId("70001");
this.dynamicSqlMapper.createStudentAutoKey(entity);
System.out.println("新增学生ID：" + entity.getStudentId());
```

selectKey语句属性配置细节：

| 属性          | 描述                                                         | 取值                                |
| ------------- | ------------------------------------------------------------ | ----------------------------------- |
| keyProperty   | selectKey语句生成结果需要设置的属性。                        |                                     |
| resultType    | 生成结果类型，MyBatis允许使用基本的数据类型，包括String、int类型。 |                                     |
| order         | 1：BEFORE，会先选择主键，然后设置 keyProperty，再执行insert语句；<br/>2：AFTER，就先运行 insert语句再运行 selectKey语句。 | BEFORE<br/>AFTER                    |
| statementType | MyBatis支持 STATEMENT，PREPARED和 CALLABLE的语句形式，对应 Statement，PreparedStatement和 CallableStatement响应 | STATEMENT<br/>PREPARED<br/>CALLABLE |

<h3>2、if标签</h3>

if标签可用在许多类型的 sql语句中，我们以查询为例。首先看一个很普通的查询：

*xml代码*

```xml
<!--查询学生 list，like姓名 --> 
<select id="getStudentListLikeName" parameterType="StudentEntity" resultMap="studentResultMap">
    SELECT * FROM STUDENT_TBL stu
    WHERE
    st.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName}), '%')
</select>
```

但是此时如果 studentName或 studentSex为 null，此语句很可能报错或查询结果为空。此时我们使用 if动态 sql语句先进行判断，如果值为 null或等于空字符串，我们就不进行此条件的判断，增加灵活性。
参数为实体类 StudentEntity。将实体类中所有的属性均进行判断，如果不为空则执行判断条件。

*xml代码*

```xml
<!-- if(判断参数)-将实体类不为空的属性作为 where条件 -->
	<select id="getStudentList_if" resultMap="resultMap_studentEntity" parameterType="liming.student.m
anager.data.model.StudentEntity">
		SELECTST.STUDENT_ID, 
		ST.STUDENT_NAME,
		ST.STUDENT_SEX,
		ST.STUDENT_BIRTHDAY,
		ST.STUDENT_PHOTO,
		ST.CLASS_ID,
		ST.PLACE_ID
		FROM STUDENT_TBL ST
		WHERE
		<if test="studentName!=null">
			ST.STUDENT_NAME LIKE CONCAT(CONCAT('%',#{studentName,jdbcType=VARCHAR}),'%')
		</if>
		<if test="studentSex!=null and studentSex!=''">
			AND ST.STUDENT_SEX=#{studentSex,jdbcType=INTEGER}
		</if>
		<if test="studentBirthday!=null">
			AND ST.STUDENT_BIRTHDAY=#{studentBirthday,jdbcType=DATE}
		</if>
		<if test="classId!=null and classId!=''">
			AND ST.CLASS_ID=#{classId,jdbcType=VARCHAR}
		</if>
		<if test="classEntity!=null and classEntity.classId!=null and classEntity.classId!=''">
			AND ST.CLASS_ID=#{classEntity.classId,jdbcType=VARCHAR}
		</if>
		<if test="placeId!=null and placeId!=''">
			AND ST.PLACE_ID=#{placeId,jdbcType=VARCHAR}
		</if>
		<if test="placeEntity!=null and placeEntity.placeId!=null and placeEntity.placeId!=''">
			AND ST.PLACE_ID=#{placeEntity.placeId,jdbcType=VARCHAR}
		</if>
		<if test="studentId!=null and studentId!=''">
			AND ST.STUDENT_ID=#{studentId,jdbcType=VARCHAR}
		</if>
	</select>
```

*java代码*

```java 
public Void select(){
	StudentEntity entity = new StudentEntity();
    entity.setStudentName("");
    entity.setStudentSex(1);
    entity.setStudentBirthday(DateUtil.parse("1996-11-06"));
    entity.sereClassId("20001");
    //entity.setPlaceId("700001");
    List<StudentEntity> list = this.dynamicSqlMapper.getStudentList_if(entity);
    for(StudentEntity e : list){
        System.out.println(e.toString());
    }
}
```

<h3>3、 if + where 的条件判断</h3>

当where中的条件使用的if标签较多时，这样的组合可能会导致错误。我们以在1中的查询语句为例子，当java代码按如下方法调用时：

*java代码*

```java
@Test 
public void select_test_2_1() { 
    StudentEntity entity = new StudentEntity(); 
    entity.setStudentName(null); 
    entity.setStudentSex(1); 
    List<StudentEntity> list = this.dynamicSqlMapper.getStudentList_if(entity); 
    for (StudentEntity e : list) { 
   	 System.out.println(e.toString()); 
	} 
} 
```

如果上面例子，参数studentName为null，将不会进行STUDENT_NAME列的判断，则会直接导“WHERE AND”关键字多余的错误SQL。

这时我们可以使用where动态语句来解决。这个“where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。

上面例子修改为：

*xml代码*

```xml
<!-- 3 select - where/if(判断参数) - 将实体类不为空的属性作为where条件 --> 
<select id="getStudentList_whereIf" resultMap="resultMap_studentEntity" parameterType="liming.student.manager.data.model.StudentEntity"> 
    SELECT ST.STUDENT_ID, 
    ST.STUDENT_NAME, 
    ST.STUDENT_SEX, 
    ST.STUDENT_BIRTHDAY, 
    ST.STUDENT_PHOTO, 
    ST.CLASS_ID, 
    ST.PLACE_ID 
    FROM STUDENT_TBL ST 
    <where> 
        <if test="studentName !=null "> 
        	ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName, jdbcType=VARCHAR}),'%') 
        </if> 
        <if test="studentSex != null and studentSex != '' "> 
        	AND ST.STUDENT_SEX = #{studentSex, jdbcType=INTEGER} 
        </if> 
        <if test="studentBirthday != null "> 
        	AND ST.STUDENT_BIRTHDAY = #{studentBirthday, jdbcType=DATE} 
        </if> 
        <if test="classId != null and classId!= '' "> 
        	AND ST.CLASS_ID = #{classId, jdbcType=VARCHAR} 
        </if> 
        <if test="classEntity != null and classEntity.classId !=null and classEntity.classId !=' ' "> 
        	AND ST.CLASS_ID = #{classEntity.classId, jdbcType=VARCHAR} 
        </if> 
        <if test="placeId != null and placeId != '' "> 
        	AND ST.PLACE_ID = #{placeId, jdbcType=VARCHAR} 
        </if> 
        <if test="placeEntity != null and placeEntity.placeId != null and placeEntity.placeId != '' "> 
        	AND ST.PLACE_ID = #{placeEntity.placeId, jdbcType=VARCHAR} 
        </if> 
        <if test="studentId != null and studentId != '' "> 
        	AND ST.STUDENT_ID = #{studentId, jdbcType=VARCHAR} 
        </if> 
    </where> 
</select> 
```

<h3>4、 if + set 的更新语句</h3>

当update语句中没有使用if标签时，如果有一个参数为null，都会导致错误。

当在update语句中使用if标签时，如果前面的if没有执行，则或导致逗号多余错误。使用set标签可以将动态的配置SET 关键字，和剔除追加到条件末尾的任何不相关的逗号。

使用if+set标签修改后，如果某项为null则不进行更新，而是保持数据库原值。如下示例：

*xml代码*

```xml
<!-- 4 if/set(判断参数) - 将实体类不为空的属性更新 --> 
<update id="updateStudent_if_set" parameterType="liming.student.manager.data.model.StudentEntity"> 
    UPDATE STUDENT_TBL 
    <set> 
        <if test="studentName != null and studentName != '' "> 
       		STUDENT_TBL.STUDENT_NAME = #{studentName}, 
        </if> 
        <if test="studentSex != null and studentSex != '' "> 
        	STUDENT_TBL.STUDENT_SEX = #{studentSex}, 
        </if> 
        <if test="studentBirthday != null "> 
        	STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday}, 
        </if> 
        <if test="studentPhoto != null "> 
        	STUDENT_TBL.STUDENT_PHOTO = #{studentPhoto, javaType=byte[], jdbcType=BLOB, typeHandler=org.apache.ibatis.type.BlobTypeHandler}, 
        </if> 
        <if test="classId != '' "> 
        	STUDENT_TBL.CLASS_ID = #{classId} 
        </if> 
        <if test="placeId != '' "> 
        	STUDENT_TBL.PLACE_ID = #{placeId} 
        </if> 
    </set> 
    WHERE 
    STUDENT_TBL.STUDENT_ID = #{studentId}; 
</update> 
```

<h3>5、if + trim代替where/set标签</h3>

trim是更灵活的去处多余关键字的标签，他可以实践where和set的效果。

**5.1trim代替where**

*xml代码*

```xml
<!-- 5.1 if/trim代替where(判断参数) - 将实体类不为空的属性作为where条件 --> 
<select id="getStudentList_if_trim" resultMap="resultMap_studentEntity"> 
    SELECT ST.STUDENT_ID, 
    ST.STUDENT_NAME, 
    ST.STUDENT_SEX, 
    ST.STUDENT_BIRTHDAY, 
    ST.STUDENT_PHOTO, 
    ST.CLASS_ID, 
    ST.PLACE_ID 
    FROM STUDENT_TBL ST 
    <trim prefix="WHERE" prefixOverrides="AND|OR"> 
        <if test="studentName !=null "> 
        	ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName, jdbcType=VARCHAR}),'%') 
        </if> 
        <if test="studentSex != null and studentSex != '' "> 
        	AND ST.STUDENT_SEX = #{studentSex, jdbcType=INTEGER} 
        </if> 
        <if test="studentBirthday != null "> 
        	AND ST.STUDENT_BIRTHDAY = #{studentBirthday, jdbcType=DATE} 
        </if> 
        <if test="classId != null and classId!= '' "> 
        	AND ST.CLASS_ID = #{classId, jdbcType=VARCHAR} 
        </if> 
        <if test="classEntity != null and classEntity.classId !=null and classEntity.classId !=' ' "> 
        	AND ST.CLASS_ID = #{classEntity.classId, jdbcType=VARCHAR} 
        </if> 
        <if test="placeId != null and placeId != '' "> 
        	AND ST.PLACE_ID = #{placeId, jdbcType=VARCHAR} 
        </if> 
        <if test="placeEntity != null and placeEntity.placeId != null and placeEntity.placeId != '' "> 
        	AND ST.PLACE_ID = #{placeEntity.placeId, jdbcType=VARCHAR} 
        </if> 
        <if test="studentId != null and studentId != '' "> 
        	AND ST.STUDENT_ID = #{studentId, jdbcType=VARCHAR} 
        </if> 
    </trim> 
</select> 
```

**5.2 trim代替set**

*xml代码*

```xml
<!-- 5.2 if/trim代替set(判断参数) - 将实体类不为空的属性更新 --> 
<update id="updateStudent_if_trim" parameterType="liming.student.manager.data.model.StudentEntity"> 
    UPDATE STUDENT_TBL 
    <trim prefix="SET" suffixOverrides=","> 
        <if test="studentName != null and studentName != '' "> 
        	STUDENT_TBL.STUDENT_NAME = #{studentName}, 
        </if> 
        <if test="studentSex != null and studentSex != '' "> 
        	STUDENT_TBL.STUDENT_SEX = #{studentSex}, 
        </if> 
        <if test="studentBirthday != null "> 
        	STUDENT_TBL.STUDENT_BIRTHDAY = #{studentBirthday}, 
        </if> 
        <if test="studentPhoto != null "> 
        	STUDENT_TBL.STUDENT_PHOTO = #{studentPhoto, javaType=byte[], jdbcType=BLOB, typeHandler=org.apache.ibatis.type.BlobTypeHandler}, 
        </if> 
        <if test="classId != '' "> 
        	STUDENT_TBL.CLASS_ID = #{classId}, 
        </if> 
        <if test="placeId != '' "> 
        	STUDENT_TBL.PLACE_ID = #{placeId} 
        </if> 
    </trim> 
    WHERE 
    STUDENT_TBL.STUDENT_ID = #{studentId} 
</update> 
```

<h3>6、 choose (when, otherwise)</h3>

有时候我们并不想应用所有的条件，而只是想从多个选项中选择一个。而使用if标签时，只要test中的表达式为true，就会执行if标签中的条件。MyBatis提供了choose 元素。if标签是与(and)的关系，而choose比傲天是或（or）的关系。

choose标签是按顺序判断其内部when标签中的test条件出否成立，如果有一个成立，则choose结束。当choose中所有when的条件都不满则时，则执行otherwise中的sql。类似于Java 的switch 语句，choose为switch，when为case，otherwise则为default。

例如下面例子，同样把所有可以限制的条件都写上，方面使用。choose会从上到下选择一个when标签的test为true的sql执行。安全考虑，我们使用where将choose包起来，放置关键字多于错误。

*xml代码*

```xml
<!-- 6 choose(判断参数) - 按顺序将实体类第一个不为空的属性作为where条件 --> 
<select id="getStudentList_choose" resultMap="resultMap_studentEntity" parameterType="liming.student.manager.data.model.StudentEntity"> 
    SELECT ST.STUDENT_ID, 
    ST.STUDENT_NAME, 
    ST.STUDENT_SEX, 
    ST.STUDENT_BIRTHDAY, 
    ST.STUDENT_PHOTO, 
    ST.CLASS_ID, 
    ST.PLACE_ID 
    FROM STUDENT_TBL ST 
    <where> 
        <choose> 
            <when test="studentName !=null "> 
            	ST.STUDENT_NAME LIKE CONCAT(CONCAT('%', #{studentName, jdbcType=VARCHAR}),'%') 
            </when > 
            <when test="studentSex != null and studentSex != '' "> 
            	AND ST.STUDENT_SEX = #{studentSex, jdbcType=INTEGER} 
            </when > 
            <when test="studentBirthday != null "> 
            	AND ST.STUDENT_BIRTHDAY = #{studentBirthday, jdbcType=DATE} 
            </when > 
            <when test="classId != null and classId!= '' "> 
            	AND ST.CLASS_ID = #{classId, jdbcType=VARCHAR} 
            </when > 
            <when test="classEntity != null and classEntity.classId !=null and classEntity.classId !=' ' "> 
            	AND ST.CLASS_ID = #{classEntity.classId, jdbcType=VARCHAR} 
            </when > 
            <when test="placeId != null and placeId != '' "> 
            	AND ST.PLACE_ID = #{placeId, jdbcType=VARCHAR} 
            </when > 
            <when test="placeEntity != null and placeEntity.placeId != null and placeEntity.placeId != '' "> 
            	AND ST.PLACE_ID = #{placeEntity.placeId, jdbcType=VARCHAR} 
            </when > 
            <when test="studentId != null and studentId != '' "> 
            	AND ST.STUDENT_ID = #{studentId, jdbcType=VARCHAR} 
            </when > 
            <otherwise> 
            </otherwise> 
        </choose> 
    </where> 
</select> 
```

<h3>7、 foreach</h3>

对于动态SQL 非常必须的，主是要迭代一个集合，通常是用于IN 条件。List 实例将使用“list”做为键，数组实例以“array” 做为键。

foreach元素是非常强大的，它允许你指定一个集合，声明集合项和索引变量，它们可以用在元素体内。它也允许你指定开放和关闭的字符串，在迭代之间放置分隔符。这个元素是很智能的，它不会偶然地附加多余的分隔符。

注意：你可以传递一个List实例或者数组作为参数对象传给MyBatis。当你这么做的时候，MyBatis会自动将它包装在一个Map中，用名称在作为键。List实例将会以“list”作为键，而数组实例将会以“array”作为键。

这个部分是对关于XML配置文件和XML映射文件的而讨论的。下一部分将详细讨论Java API，所以你可以得到你已经创建的最有效的映射。

**7.1参数为array示例的写法**

接口的方法声明：

*java代码*

```java
public List<StudentEntity> getStudentListByClassIds_foreach_array(String[] classIds); 
```

动态SQL语句：

*xml代码*

```xml
<!— 7.1 foreach(循环array参数) - 作为where中in的条件 --> 
<select id="getStudentListByClassIds_foreach_array" resultMap="resultMap_studentEntity"> 
    SELECT ST.STUDENT_ID, 
    ST.STUDENT_NAME, 
    ST.STUDENT_SEX, 
    ST.STUDENT_BIRTHDAY, 
    ST.STUDENT_PHOTO, 
    ST.CLASS_ID, 
    ST.PLACE_ID 
    FROM STUDENT_TBL ST 
    WHERE ST.CLASS_ID IN 
    <foreach collection="array" item="classIds" open="(" separator="," close=")"> 
    	#{classIds} 
    </foreach> 
</select> 
```

测试代码，查询学生中，在20000001、20000002这两个班级的学生：

*java代码*

```java
@Test 
public void test7_foreach() { 
    String[] classIds = { "20000001", "20000002" }; 
    List<StudentEntity> list = this.dynamicSqlMapper.getStudentListByClassIds_foreach_array(classIds); 
    for (StudentEntity e : list) { 
    	System.out.println(e.toString()); 
    }
} 
<p>
    <span style="font-size: 14px; font-weight: bold; white-space: normal;">
    </span>
</p> 
```

**7.2参数为list示例的写法**

接口的方法声明：

*java代码*

```java
public List<StudentEntity> getStudentListByClassIds_foreach_list(List<String> classIdList); 
```

动态SQL语句：

*Xml代码*

```xml
<!-- 7.2 foreach(循环List<String>参数) - 作为where中in的条件 --> 2.<select id="getStudentListByClassIds_foreach_list" resultMap="resultMap_studentEntity"> 
    SELECT ST.STUDENT_ID, 
    ST.STUDENT_NAME, 
    ST.STUDENT_SEX, 
    ST.STUDENT_BIRTHDAY, 
    ST.STUDENT_PHOTO, 
    ST.CLASS_ID, 
    ST.PLACE_ID 
    FROM STUDENT_TBL ST 
    WHERE ST.CLASS_ID IN 
    <foreach collection="list" item="classIdList" open="(" separator="," close=")"> 
    	#{classIdList} 
    </foreach> 
</select> 
```

测试代码，查询学生中，在20000001、20000002这两个班级的学生：

*Java代码*

```java
@Test 
public void test7_2_foreach() { 
    ArrayList<String> classIdList = new ArrayList<String>(); 
    classIdList.add("20000001"); 
    classIdList.add("20000002"); 
    List<StudentEntity> list = this.dynamicSqlMapper.getStudentListByClassIds_foreach_list(classIdList); 
    for (StudentEntity e : list) { 
    	System.out.println(e.toString()); 
    } 
} 
```

