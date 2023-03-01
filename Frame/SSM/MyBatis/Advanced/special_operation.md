# MyBatis处理特殊操作

**提示**：下列各项操作均使用[student表](../Table/student.md)来操作。

## 处理字段和属性的映射关系

数据表中字段通常使用**下划线命名法**，而Java命名变量使用**驼峰命名法**，两者命名不同，MyBatis自动映射关联不到，字段将无法赋值到实体类中。

处理字段和属性的映射关系三种方式：

- 通过设置字段别名处理映射关系
- 通过resultMap处理映射关系
- 通过配置文件处理映射关系

### 通过设置字段别名处理映射关系

只需要在SQL语句中设置同属性名相同的别名即可解决：

- mapper接口中定义一个`selectAllByAlias()`方法：

  > StudentMapper.java

  ```java
  /**
  * 通过Sql起别名来解决映射问题
  * @return 所有数据
  */
  List<Student> selectAllByAlias();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="selectAllByAlias" resultType="com.hjc.demo.pojo.Student">
  	select id,s_name sName,age,sex from student;
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testSelectByAlias() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //执行查询
     List<Student> students = mapper.selectAllByAlias();
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select id,s_name sName,age,sex from student; 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3
  > Student{id=1, sName='小明', age=15, sex='男'}
  > Student{id=2, sName='小红', age=19, sex='女'}
  > Student{id=3, sName='小寸', age=25, sex='男'}

### 通过resultMap处理映射关系

resultMap 是 MyBatis 中最复杂的元素，主要用于解决实体类属性名与数据库表中字段名不一致的情况，可以将查询结果映射成实体对象。

通过sql语句查询出来的数据就会通过resultMap设置的自定义映射的关系进行映射

**拓展**：[resultMap用法]()

- mapper接口中定义一个`selectAllByResultMap()`方法：

  > StudentMapper.java

  ```java
  /**
  * 通过resultMap来解决映射问题
  * @return 所有数据
  */
  List<Student> selectAllByResultMap();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <!-- 设置自定义映射关系：id：唯一标识，不能重复、type：设置映射关系中的实体类类型-->
  <resultMap id="stuResultMap" type="com.hjc.demo.pojo.Student">
  	<!-- 设置主键字段:property属性名 column表中字段名 -->
      <id property="id" column="id"/>
      <!-- 设置普通字段:property属性名 column表中字段名-->
      <result property="sName" column="s_name"/>
      <!-- 设置普通字段:property属性名 column表中字段名-->
      <result property="age" column="age"/>
      <!-- 设置普通字段:property属性名 column表中字段名-->
      <result property="sex" column="sex"/>
  </resultMap>
  <select id="selectAllByResultMap" resultMap="stuResultMap">
  	select id,s_name,age,sex from student;
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testSelectByResultMap() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //执行查询
     List<Student> students = mapper.selectAllByResultMap();
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select id,s_name,age,sex from student; 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3
  > Student{id=1, sName='小明', age=15, sex='男'}
  > Student{id=2, sName='小红', age=19, sex='女'}
  > Student{id=3, sName='小寸', age=25, sex='男'}

### 通过配置文件处理映射关系

MyBatis为解决此类问题，内置了将字段名中的下划线转换为Java的驼峰命名。

**拓展**：[settings用法]()

- MyBatis核心配置文件：

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>  
  <!DOCTYPE configuration  
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-config.dtd">  
  <configuration>  
       <!-- ...-->
       <settings>  
           <setting name="mapUnderscoreToCamelCase" value="true" />  
       </settings>  
       <!-- ...-->
  </configuration>
  ```

- mapper接口中定义一个`selectAllBySettings()`方法：

  > StudentMapper.java

  ```java
  /**
  * 通过resultMap来解决映射问题
  * @return 所有数据
  */
  List<Student> selectAllBySettings();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="selectAllBySettings" resultType="com.hjc.demo.pojo.Student">
  	select id,s_name,age,sex from student;
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testSelectByResultMap() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //执行查询
     List<Student> students = mapper.selectAllBySettings();
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select id,s_name,age,sex from student; 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3
  > Student{id=1, sName='小明', age=15, sex='男'}
  > Student{id=2, sName='小红', age=19, sex='女'}
  > Student{id=3, sName='小寸', age=25, sex='男'}

## 模糊查询

当使用模糊查询时，由于`#{}`底层为占位符，获取参数值并且执行过程中用`？`代替`#{}` 不会解析成占位符，会被认为是字符串。

![image-20230202193346092](./assets/image-20230202193346092.png)

解决办法：

1. 使用`${}`解决问题(不推荐，会存在注入问题)：

   ```xml
   <select id="selectStuByLike" resultType="com.hjc.demo.pojo.Student">
   	select id,s_name,age,sex from student where s_name like '%${name}%';
   </select>
   ```

2. 通过concat解决问题：

   ```xml
   <select id="selectStuByLike" resultType="com.hjc.demo.pojo.Student">
   	select id,s_name,age,sex from student where s_name like concat('%',#{name},'%');
   </select>
   ```

3. 通过拼接方式解决问题：

   ```xml
   <select id="selectStuByLike" resultType="com.hjc.demo.pojo.Student">
   	select id,s_name,age,sex from student where s_name like "%"#{name}"%";
   </select>
   ```

## 批量删除

由于`#{}`本身解析之后就会带一个`'` `'`，导致结果出现问题，所以只能使用`${}`。

- mapper接口中定义一个`deleteMore()`方法：

  > StudentMapper.java

  ```java
  /**
  * 批量删除
  * @param ids 需要删除的id
  * @return 删除的条数
  */
  int deleteMore(@Param("ids") String ids);
  ```

- mapper映射文件中增加删除语句：

  > StudentMapper.xml

  ```xml
  <delete id="deleteMore">
  	delete from student where id in (${ids})
  </delete>
  ```

- 测试代码：

  ```java
  @Test
  public void testDeleteMore() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      //执行批量删除
      int result = mapper.deleteMore("1,2,3");
      System.out.println("result = " + result);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: delete from student where id in (1,2,3)
  > DEBUG  ==> Parameters: 
  > DEBUG  <==    Updates: 3 
  > result = 3

## 动态设置表名

由于`#{}`本身解析之后就会带一个`'` `'`，导致结果出现问题，所以只能使用`${}`。

- mapper接口中定义一个`getAllStu()`方法：

  > StudentMapper.java

  ```java
  /**
  * 动态设置表名，查询所有的用户信息
  * @param tableName 需要查询的表
  * @return 表中的所有数据
  */
  List<Student> getAllStu(@Param("tableName") String tableName);
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="getAllStu" resultType="com.hjc.demo.pojo.Student">
  	select * from ${tableName}
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testSelectByTable() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询指定表中的数据
     List<Student> students = mapper.getAllStu("student");
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3 
  > Student{id=4, sName='小明', age=15, sex='男'}
  > Student{id=5, sName='小红', age=19, sex='女'}
  > Student{id=6, sName='小寸', age=25, sex='男'}

## 获取自增的主键

- [主键（自动递增）回填](../Basis/basic_operation.md#主键(自动递增)回填)

## 各类查询功能

**注意**：查询功能必须设置resultType(结果类型)或resultMap(结果映射)

- 若查询出的数据只有一条：
  1. 可以通过实体类对象接收
  2. 可以通过list集合接收

- 若查询出的数据有多条：
  1. 可以通过list集合接收。一定不能通过实体类对象接收，此时会抛异常TooManyResultsException
  2. 可以通过map类型的list集合接收
  3. 可以在mapper接口的方法上添加[`@MapKey`注解](../Advanced/mybatis_annotation.md#@MapKey:设置当前map的键)，此时可以将每条数据转换的map集合作为值，以某个字段的值作为键，放在同一个map集合中

### 查询单个数据

#### 使用实体类

- mapper接口中定义一个`selectById()`方法：

  > StudentMapper.java

  ```java
  /**
  * 根据id查询用户信息(查询出的数据只有一条)
  * @param id 根据id查询
  * @return 返回一个表中信息
  */
  Student selectById(@Param("id") Integer id);
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="selectById" resultType="com.hjc.demo.pojo.Student">
      select * from student where id = #{id}
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testOneQueryToBean() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      //查询数据
      Student student = mapper.selectById(1);
      System.out.println("student = " + student);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student where id = ? 
  > DEBUG  ==> Parameters: 1(Integer) 
  > DEBUG  <==      Total: 1 
  > student = Student{id=1, sName='小明', age=15, sex='男'}

#### 使用List集合

- mapper接口中定义一个`selectByIdToList()`方法：

  > StudentMapper.java

  ```java
  /**
  * 根据id查询用户信息(查询出的数据只有一条，通过list接收)
  * @param id 根据id查询
  * @return 返回一个表中信息
  */
  List<Student> selectByIdToList(@Param("id") Integer id);
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="selectByIdToList" resultType="com.hjc.demo.pojo.Student">
      select * from student where id = #{id}
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testOneQueryToList() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询数据
     List<Student> students = mapper.selectByIdToList(1);
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student where id = ? 
  > DEBUG  ==> Parameters: 1(Integer) 
  > DEBUG  <==      Total: 1 
  > Student{id=1, sName='小明', age=15, sex='男'}

#### 使用Map集合

- mapper接口中定义一个`getStuByIdToMap()`方法：

  > StudentMapper.java

  ```java
  /**
  * 根据id查询用户信息为一个map集合
  * @param id 需要查询数据的id
  * @return 所匹配的数据
  */
  @MapKey("id")
  Map<String,Object> getStuByIdToMap(@Param("id") Integer id);
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="getStuByIdToMap" resultType="java.util.Map">
          select * from student where id = #{id}
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testOneQueryToMap() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询数据
     Map<String, Object> map = mapper.getStuByIdToMap(1);
     System.out.println("map = " + map);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student where id = ? 
  > DEBUG  ==> Parameters: 1(Integer) 
  > DEBUG  <==      Total: 1 
  > map = {1={sex=男, id=1, s_name=小明, age=15}}

#### 查询总记录数

查询总记录数：

```sql
 select count(*) from 表名
```

**说明**：设置为count(1)、count(*)、count(字段)都可以。

从查询结果来说：count(1)、 count(*)、 count(字段) 不一样的，count字段如果某一字段的值为null，这条记录是不会被放到总记录数中的。

COUNT(column_name) 函数返回指定列的值的数目(NULL 不计入)。

- mapper接口中定义一个`getCount()`方法：

  > StudentMapper.java

  ```java
  /**
  * 查询总记录数
  * @return 总记录条数
  */
  Integer getCount();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="getCount" resultType="java.lang.Integer">
          select count(*) from student
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testCount() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询总记录数
     Integer count = mapper.getCount();
     System.out.println("count = " + count);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select count(*) from student 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 1
  > count = 3

### 查询多个数据

#### 使用List集合

- mapper接口中定义一个`selectAllToList()`方法：

  > StudentMapper.java

  ```java
  /**
  * 通过list接收所有表中信息
  * @return 返回所有表中信息
  */
  List<Student> selectAllToList();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="selectAllToList" resultType="com.hjc.demo.pojo.Student">
  	select * from student
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testAllQueryToList() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询数据
     List<Student> students = mapper.selectAllToList();
     students.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3 
  > Student{id=1, sName='小明', age=15, sex='男'}
  > Student{id=2, sName='小红', age=19, sex='女'}
  > Student{id=3, sName='小寸', age=25, sex='男'}

#### 使用Map集合

- mapper接口中定义一个`getStuAllToMap()`方法：

  > StudentMapper.java

  ```java
  /**
  * 查询用户信息为一个map集合
  * @return 所匹配的数据
  */
  @MapKey("id")
  Map<Integer,Object> getStuAllToMap();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="getStuAllToMap" resultType="java.util.Map">
          select * from student
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testAllQueryToMap() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
     //查询数据
     Map<Integer, Object> map = mapper.getStuAllToMap();
     System.out.println("map = " + map);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3 
  > map = {1={sex=男, id=1, s_name=小明, age=15}, 2={sex=女, id=2, s_name=小红, age=19}, 3={sex=男, id=3, s_name=小寸, age=25}}

#### 使用List+Map结合

- mapper接口中定义一个`getAllStuToMap()`方法：

  > StudentMapper.java

  ```java
  /**
  * 将表中所有的数据以map的方式存入到list集合中
  * @return 返回以id为key 把转换成的map集合作为值
  */
  @MapKey("id")
  List<Map<String,Object>> getAllStuToMap();
  ```

- mapper映射文件中增加查询语句：

  > StudentMapper.xml

  ```xml
  <select id="getAllStuToMap" resultType="java.util.Map">
  	select * from student
  </select>
  ```

- 测试代码：

  ```java
  @Test
  public void testAllQueryToListMap() throws IOException {
      //读取MyBatis的核心配置文件
      InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
      //通过核心配置文件所对应的字节输入流创建工厂类SqlSessionFactory，生产SqlSession对象
      SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resource);
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都必须手动提交或回滚事务
      //SqlSession sqlSession = sqlSessionFactory.openSession();
      //创建SqlSession对象，此时通过SqlSession对象所操作的sql都会自动提交
      SqlSession sqlSession = build.openSession(true);
      //通过代理模式创建UserMapper接口的代理实现类对象
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      //查询数据
      List<Map<String, Object>> stu = mapper.getAllStuToMap();
      stu.forEach(System.out::println);
  }
  ```

- 执行测试代码，控制台输出：

  > DEBUG  ==>  Preparing: select * from student 
  > DEBUG  ==> Parameters: 
  > DEBUG  <==      Total: 3 
  > {sex=男, id=1, s_name=小明, age=15}
  > {sex=女, id=2, s_name=小红, age=19}
  > {sex=男, id=3, s_name=小寸, age=25}