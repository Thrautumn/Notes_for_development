## 相关依赖 
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.19</version>
</dependency>
```

## 基本信息

### `config.xml` 
  
  该文件用于配置 MyBatis 。包括事务管理方式（ transaction ）、数据源（ datasource ）、驱动、数据库URL 以及 Mapper 映射文件等信息。

### `Mapper.xml`

该文件用于存放具体的 SQL 语句，并定义其与 Java 方法/实体类的映射关系。
> 对于简单的 SQL 语句，可以直接利用 接口 + 注解开发。但是对于多表嵌套查询的 SQL 语句，仍然建议用 xml 文件编写。

### SQL 相关类

#### SqlSessionFactoryBuilder & SqlSessionFactory

* **`SqlSessionFactoryBuilder`**：专门用于构建 SqlSessionFactory 实例，用完即弃。

* **`SqlSessionFactory`**： MyBatis 的核心工厂类，专门用于构建 SqlSession 实例，单例模式，用完即弃。

#### SqlSession

* MyBatis 与数据库交互的核心会话类。每个 SqlSession 对应一次独立的数据库连接。用完必须关闭，否则会导致数据库连接泄漏。

* 一般结合 Mapper 接口执行 SQL 语句。******

#### 使用模版

```Java
@UtilityClass
public class MybatisUtils {
    // 全局单例
    private static SqlSessionFactory sqlSessionFactory;

    // 代码块初始化 SqlSessionFactory
    // 利用 SqlSessionFactory 的 build 方法以及 Resources.getResourceAsStream 方法传入配置文件
    static {
        try {
            sqlSessionFactory = new SqlSessionFactoryBuilder()
                    .build(Resources.getResourceAsStream("mybatis_config.xml"));
        }
        catch(IOException e) {
            e.printStackTrace();
        }
    }

    // 设置 SqlSessionFactory 生成的 SqlSession 是自动 commit 的
    private static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession(true);
    }
}

```

## 编写 Mapper.xml 文件

### 基本格式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.test.mapper.Mapper">
    ......
</mapper>
```

下面将介绍常用的编写 SQL 语句所用的标签。按层级来介绍。*每个标签后面会带上 Rx 的标记。 x 为层权。权数越小，表示在越外层。*

### `<mapper>` R1

#### 属性 

* **`namespace`**： 作为该 mapper 文件的唯一标识名。该标签在 mapper.xml 文件中唯一存在。如若填入**某接口的完整限定名**时，用作接口绑定。

### `<select>` R2

#### 属性 

* **`id`**： 查询语句的标识
  
* **`parameterType`**：指定入参类型。因为 DQL 语句中有 `where id = xxx` 这样的子句，xxx 就是入参。**但这个属性可以不写，由 MyBatis 自动推断**。

* **`resultType`**： 指定查询结果的类型

* **`resultMap`**： 关联对应的 **[`<resultMap>`](#resultmap-r2)** 的 id 值。

### `<select>` & `<mapper>` 例子
```xml
<mapper namespace = "test">
    <select id = "selectUserById" parameterType = "int" resultType = "User">
        select * 
        from user 
        where id = #{id}
    </select>
</mapper>
```
注意 where 子句的写法，使用了参数占位符，能有效地避免 SQL 注入。

### `<resultMap>` R2

需要注意的是，如果 DB 中的字段名与 Java 中实体类中定义的字段名不一致的话，MyBatis在自动处理时会映射出错。所以引入了 `<resultMap>` 用以指导 `<select>` 如何将查询结果转换为实体类。**需配套 [`<result>` 或 `<id>`](#result--id-r3) 使用**。
>比如 DB 中定义一张表，对应着 Java 中的 Student 实体类。表中某字段名为 Sage ，但到了 class Student 中却变为了 age 。这就是不匹配，要出问题的。

#### 属性

* **`id`**：必填。不然 `<select>` 无法关联。

* **`type`**： 用于替换 `<select>` 中的 `resultType` 属性。指示查询的返回类型。

### `<result>` & `<id>` R3

写在 `<resultMap>` 内，配套使用。用于关联 DB 中字段与 Java 实体类中的字段。当字段为 主键时，用 `<id>` ，反之用 `<result>` 。

#### 属性

* **`column`**：DB 中字段名

* **`property`**： Java 中实体类字段名

### `<select>` & `<resultMap>` & `<result>` & `<id>` 例子

```xml
<mapper namespace = "test">
    <resultMap id = "rm1" type = "User">
        <id column = "id" property = "id" />
        <result column = "Sage" property = "age" />
    </resultMap>
    <select id = "selectUserById" resultMap = "rm1">
        select * 
        from user 
        where id = #{id}
    </select>
</mapper>
```

### `<association>` R3

**嵌套在 `<resultMap>` 内**，用于 MyBatis 处理**一对一 多表查询**。**内部可以嵌套 `<id>` `<result>` 标签**。

#### 属性

* **`column`**： 表A --> 表B，用 A 中哪个字段去联系 B 

* **`property`**： 表A --> 表B，A对应实体类的关联字段名（这个字段对应的变量去接收 B 的结果）

* **`javaType`**： 表A --> 表B，B 的实体类类型。其实就是 Property 这个属性的实体类类型。

#### 举个例子

现在有三张经典表 **Student(Sid, Sname, Sage)** 、 **Course(Cid, Cname, Ccredit)** 、 **SC(id, Sid, Cid, Grade)** 。现在希望查询 某个学生选的某一门课的完整信息（学生信息 + 选课记录 + 课程信息） 

</br>

```mermaid
flowchart LR

A[Student]
B[SC]
C[Course]

A --> |Association| B 
B --> |Association| C
```

</br>

```Java
// 1. 课程实体类（Course，对应 course 表）
public class Course {
    private Long id;         
    private String name;     
    private float credit;  
}

// 2. 选课记录实体类（SC，对应 sc 表）
public class SC {
    private Long id;         
    private Long studentId;  
    private Long courseId;   
    private Integer score;   
    
    private Course course;      // 关联属性：当前选课记录对应的课程（Course对象）
}

// 3. 学生实体类（Student，主表，对应 student 表）
public class Student {
    private Long id;         
    private String name;     
    private Integer age;     
   
    private SC sc;  // 关联属性：当前学生的某一条选课记录（SC对象）
}
```

```xml
<mapper namespace="com.example.mapper.StudentMapper">
    <resultMap id="StudentWithSCAndCourseMap" type="Student">
        <!-- ========== 第一层：映射主表 Student 的字段 ========== -->
        <id column="Sid" property="id"/>          
        <result column="Sname" property="name"/> 
        <result column="Sage" property="age"/>    

        <!-- ========== 第二层：association 映射关联表 SC（选课记录） ========== -->
        <association column="Sid" property="sc" javaType="SC" >                
            <result column="Grade" property="score"/>    
          
            <!-- ========== 第三层：association 嵌套映射关联表 Course（课程） ========== -->
            <association column="Cid" property="course" javaType="Course" > 
                <id column="Cid" property="id"/>        
                <result column="Cname" property="name"/>
                <result column="Ccredit" property="credit"/>
            </association>
        </association>
    </resultMap>

    <select id="selectStudentWithSCAndCourse" resultMap="StudentWithSCAndCourseMap">
        select s.Sid, Sname, Sage, c.Cid, c.Cname, c.Ccredit, Grade
        from Student s, Course c, SC
        where s.Sid = SC.Sid and c.Cid = SC.Cid and s.Sid = #{Sid} and c.Cid = #{Cid};
    </select>
</mapper>
```

**！！！ 永远都是先写 SQL 语句，根据 SQL 语句来写 resultMap ！！！**
上面这个例子，由于 SC 表就查了一个 Grade ，所以其他字段根本不用写 result 。

### `<collection>` R3

**嵌套在 `<resultMap>` 内**，用于 MyBatis 处理**一对多 多表查询**。**内部可以嵌套 `<id>` `<result>` 标签**。

#### 属性

* **`column`**： 表A --> 表B，用 A 中哪个字段去联系 B 。这个字段必须是集合类型

* **`property`**： 表A --> 表B，A对应实体类的关联字段名（这个字段对应的变量去接收 B 的结果）

* **`ofType`**： 表A --> 表B，B 的实体类类型。其实就是 Property 这个属性的实体类类型。

#### 举个例子

还是三张经典表 **Student(Sid, Sname, Sage)** 、 **Course(Cid, Cname, Ccredit)** 、 **SC(id, Sid, Cid, Grade)** 。现在希望查询 查询 某个学生的所有选课记录 + 每门课的课程信息（ 选课记录 + 课程信息） 

</br>

```mermaid
flowchart LR

A[Student]
B[SC]
C[Course]

A --> |Collection| B 
B --> |Association| C
```

</br>

```Java
public class Course {
    private Long id;         
    private String name;     
    private float credit;  
}

public class SC {
    private Long id;         
    private Long studentId;  
    private Long courseId;   
    private Integer score;   
    private Course course;  
}
public class Student {
    private Long id;         
    private String name;     
    private Integer age;     
    // 一对多：一个学生对应多个选课记录 → 集合类型
    private List<SC> scList;  // 对应property="scList"
}
```

```xml
<mapper namespace="com.example.mapper.StudentMapper">
    <resultMap id="StudentWithSCListMap" type="Student">
        <!-- 第一层：映射Student主表（表A） -->
        <id column="Sid" property="id"/>          
        <result column="Sname" property="name"/> 

        <!-- 第二层：<collection> 映射一对多的SC（表B） -->
        <collection column="Sid" property="scList" ofType="SC">          
            <result column="Grade" property="score"/>    

            <!-- 第三层：嵌套<association>映射Course（SC→Course，一对一） -->
            <association column="Cid" property="course" javaType="Course"> 
                <id column="Cid" property="id"/>        
                <result column="Cname" property="name"/>
                <result column="Ccredit" property="credit"/>
            </association>
        </collection>
    </resultMap>

    <!-- 一对多查询SQL：查某个学生的所有选课记录+课程 -->
    <select id="selectStudentWithSCList" resultMap="StudentWithSCListMap">
       select s.Sid, Sname, c.Cid, Cname, Ccredit, Grade
       from Student s, Course c, SC
       where s.Sid = SC.Sid and c.Cid = SC.Cid and Sid = #{Sid}
    </select>
</mapper>
```
</br>

### `<insert>` R2

#### 属性

* **`id`**

* **`parameterType`**： 待操作的数据类型

* **`useGenerratedKeys`**： 当为 true 时，可以获取 DB 中自动递增主键的值。插入后 MyBatis 会把数据库生成的自增主键值赋值到指定属性。**这个属性必须和 keyColumn / keyProperty 配合使用**，单独用无意义。

* **`keyColumn`**： 指定表中自动递增的主键列名

* **`keyProperty`**： 指定主键在实体类中对应的字段名。这样插入后，可以将自动递增的主键值返回给实例对象

#### 举个例子
```Java
@Data
@Accessors(chain = true)
public class Student {
    
    private Long id;         // 自增、主键
    private String name;     
    private Integer age;     
}
```

```Java
public interface StudentMapper {
    int insertStudent(Student student);
}
```

```xml
<mapper namespace="com.example.mapper.StudentMapper">
    <insert id="insertStudent" parameterType="Student" useGeneratedKeys="true" keyColumn="Sid" keyProperty="id">               
        insert into student (Sname, Sage)
        values (#{name}, #{age})    <!-- name age 与实体类中相应字段名一致 -->
    </insert>
</mapper>
```

```Java
public class InsertTest {
    public static void main(String[] args) 
        try (SqlSession session = sqlSessionFactory.openSession(true)) {
            StudentMapper mapper = session.getMapper(StudentMapper.class);

            Student student = new Student().setName("小红").setAge(19);
          
            mapper.insertStudent(student);
            
            // 这样依赖，虽然我们没有为 student 实例对象赋值 id 字段。但是 mapper 中的 keyProperty 自动将递增的值赋给了 id 字段。
            System.out.println("数据库生成的自增主键Sid = " + student.getId());
        }
    }
}
```
### `<update>` R3

#### 属性

* **`id`**

* **`parameterType`**

### `<delete>` R2

#### 属性

* **`id`**

* **`parameterType`**

**DML 语句的传参不建议逐一传递。建议直接传入实体类对象。根据实体类字段名与 `#{variableName}` 中的参数名注意配对实现。所以必须一致。例子见上。**

## 接口绑定

分为两种。一种纯接口 + [注解开发](#注解开发)，适合简单的 SQL 语句。另一种是 mapper + 接口，适合复杂的 SQL 语句。下面介绍后者。

### 大致流程

\* 建议在同一个 package 下创建 mapper.xml 文件和绑定的 Mapper 接口。

1. 将 `<mapper>` 的 `namespace` 属性改为绑定的 Mapper 接口的**完整限定名**。
> 完整限定名：一级包名.二级包名.…….最后一级包名.类名/接口名

2. 在 Mapper 接口中，以 DQL 或 DML 等语句的 id 属性值作为方法名，写出方法的定义。

3. 使用时，利用 `SqlSession` 的 `T getMapper(Class<T> class)` 方法获取待使用的 Mapper ，即可使用其中的方法。
 
### **参数匹配**

* **接口方法参数为单参**：
mapper.xml 中直接用 #{参数名}（或 #{任意名}，MyBatis 自动匹配）

* **接口方法参数为多参**：
必须用 @Param 注解命名，比如 `selectStudent(@Param("name") String name, @Param("age") int age);` 其中 `@Param()` 的值为 xml 中 `#{}` 中的名称。

### 举个例子
```xml
<mapper namespace="com.example.mapper.StudentMapper">
    <!-- select的id = 接口方法名 → 核心绑定规则 -->
    <select id="selectStudentById" parameterType="Long" resultType="Student">
        select Sid as id, Sname as name, Sage as age
        from Student 
        where Sid = #{id}
    </select>

    <select id="selectTest" resultType="Student">
        select Sid as id, Sname as name, Sage as age 
        from Student
        where Sname = #{name} and Sage = #{age}
    </select>

    <insert id="insertStudent" parameterType="Student" useGeneratedKeys="true" keyColumn="Sid" keyProperty="id">
        insert into Student (Sname, Sage) values (#{name}, #{age})
    </insert>
</mapper>
```

```Java
public interface StudentMapper {
    Student selectStudentById(Long id);
    
    List<Student> selectTest(@Param("name") String name, @Param("age") int age);

    int insertStudent(Student student);
}
```

```Java
public class Test {
    public static void main(String[] args) {
        StudentMapper mapper = session.getMapper(StudentMapper.class);
            
        Student student = mapper.selectStudentById(1L);

        List<Student> student1 = mapper.selectTest("w", 20);
    }
}
```
## 注解开发

扔掉 xml 文件，仅保留 Mapper 接口，使用方法同 xml 。以下注解全部打在方法上面，相当于 xml 文件中的标签。

### @Select

#### 注解属性

* **`String value()`**： 仅需知道这个。填入 DQL 语句。

### @Results

相当于 `<resultMap>` 标签。

#### 注解属性

* **`Result[] value()`**； 接收 @Result 注解。

### @Result

相当于 `<result>` 标签。

#### 注解属性

* **`boolean id()`**： 标识当前字段是否为主键
  
* **`String column()`**： 表字段名

* **`String property()`**： 映射的实体类字段名

