## Java后端开发规范

　　一、技术栈规约

　　二、命名规范

　　三、Java代码规范（注释规范、异常与日志、代码逻辑规范）

　　四、Mybatis与SQL规范

　　五、结果检查（单元测试及代码扫描）

　　六、安全规范

### 一、技术栈规约 

![img](https://img2018.cnblogs.com/blog/1244163/201906/1244163-20190605135955964-771152631.png)

![img](https://img2018.cnblogs.com/blog/1244163/201906/1244163-20190605140119591-1760329980.png)

### 二、命名规范

- 命名使用英文词组合，严禁使用中文拼音或拼音首字母组合命名（专有名词例外） - OrganizationTreeNode, OrganizationVO ; 不推荐使用PSTree , Tlogs
- groupId，package包名前缀统一为: com.wiwj
- 包名第三位为产品分类名，如com.wiwj.cbs
- 常量命名全大写，单词间下划线分隔。如: DEFAULT_PAGE_SIZE
- 其他命名遵循驼峰式命名法:
  - 类名：首字母大写的UpperCamelCase，如: Organization
  - 方法名、变量名：首字母小写的lowerCamelCase，如: orgName
- 特定标识命名:
  - 领域模型增加类型后缀标识，如xxVO, yyDAO
  - 基类/抽象类使用Base/Abstract等前缀标识
  - 设计模式类添加Factory，Builder，Proxy等标识
  - Controller, Service, Mapper统一添加到对应分层目录
  - 接口实现类添加Impl后缀标识
  - 枚举类添加Enum后缀标识
  - CRUD接口采用统一前缀: get, count, create, delete, update, batchCreate …

### 三、Java代码规范

#### 　　注释规范

- Java文件统一添加固定Header，通过IDE统一配置（code templates）　

```
*/***

** <Description> <br>*

** @author mazhicheng@5i5j.com<br>*

** @version 1.0<br>*

** @date ${YEAR}/${MONTH}/${DAY} <br>*

**/*
```



- 接口和方法统一添加Java Doc标准注释

　

```
　　　/**

　　　　　　*缓存key-value并设定过期时间

　　　　　　* @param key 缓存对象的key

　　　　　　 * @param valueList 缓存对象

　　　　　　* @return 缓存是否成功

　　　　　　*/

　　　　　 <T> boolean addList(String key, List<T> valueList);
```



- 需暂留的弃用类/方法添加 @Deprecated 废弃标记 和 @see 链接指向新接口

　　

```
　　* @see com.wiwj.common.cache.redis.JedisSentinelPoolUtil

　　　　*/

　　　　@Deprecated

　　　　public class JedisUtils {…}
```



#### 　　异常与日志

- 调用外部服务等可能异常的代码块，用 try/catch 代码块捕获并在catch中记录异常跟踪日志及业务逻辑处理
- 禁止吞掉异常信息

　　　　禁止catch里不做任何记录和处理，吞掉异常及其堆栈信息

　　　　禁止: logger.error(“XXX操作异常”) 或 logger.error(“XXX操作异常”+e) 或 e.printStackTrace()

　　　　正确: logger.error("XXX操作异常", e)

- 对于非预期的条件，尽量增加else记录跟踪日志
- 禁止通过System.*.out()打印日志（单元测试例外）
- 日志记录logger需使用Slf4J代理声明，禁止绑死具体日志系统的API，避免后期更换日志组件导致代码的大量改动

　　　　private static final Logger log = LoggerFactory.getLogger(OrganizationServiceImpl.class);

　　　　 如采用了lombok，可用 @Slf4j 注解替代以上声明。

- 对 trace/debug/info 级别的日志输出，必须使用占位符形式，避免直接String拼接异常信息（即使日志级别不匹配也会执行拼接操作空耗资源）。

　　　　正确写法如:

　　　　　　 log.debug("当前用户id: {} ，操作对象: {}=>{} ", userId, objectType, objectId);

　　　　　　或条件输出形式如:

　　　　　　　　if(log.isDebugEnabled()){

　　　　　　　　　　 log.debug("当前用户id: “+id+” ，操作对象: “+ objectType +”=> “+ objectId);

　　　　　　　　 }

#### 　　逻辑代码规范

- 废弃的/无用的代码一律直接删除，禁止以注释等方式保留。如需查看历史代码，通过SVN/Git的history找回

　　　　(无用的代码会干扰团队成员的阅读/或被误调，越积越多会导致代码维护成本增高)

- 接口类中的方法不需添加 public 修饰符
- 需要序列化的Bean类统一实现Serializable接口并用IDE生成serialVersionUID

　　

```
　　public class MyEntity implements Serializable {   

　　　　　　 private static final long serialVersionUID = 123456L;   

　　　　　　 ...

　　　　}
```



- 常用字符串统一定义在常量类里，如: “utf-8”, “yyyyMMdd”
- 避免数字类型比较的坑: 统一采用equals进行比较其值，不用==进行比较，避免踩坑。
- if/else/for/while语句后必须使用大括号，即使只有一行代码。(需求总是变化的，一行是暂时的)
- 嵌套层次过多的代码块利用反向思维缩减层次
- 方法单一职责: 单个方法代码行数控制在100行以内，超长的需要拆分（拆分成多个方法或类）
- 避免NPE(NullPointException)的一些建议:
  - equals比较将非空对象前置:如 "true".equals(request.getParameter("isXx"))，即使后者为空也不会导致NPE。
  - 数据库字段可空的映射属性使用包装类型定义:如基本数据类型的int映射到数据库的null值将产生NPE，而用吧包装类型 Integer 则不会。
  - 可能为空的变量进行必要判空，并在非预期条件下打印必要的跟踪日志，不但避免NPE，还非常便于跟踪调试。如:
  - 级联调用 obj.getA().getB().getC() 易产生 NPE，先进行判空或使用 JDK8 的 Optional 类包装。
  - 调用Dubbo接口拿到返回值时，进行判空。
  - 封装统一的判空类用于常用类型的判空，代码需要判空时统一调用即可。如 XX.isEmpty(), XX.isNotEmpty()

- 遵循: Don’t Repeat Yourself，即 DRY 原则。避免进行简单的复制粘贴修改，当出现重复代码时思考是否封装

　　　　当代码中存在大量重复代码时，一旦代码逻辑变动将很容易导致顾此失彼，产生bug，非常不利于维护。

- Bean属性拷贝推荐用Spring BeanCopier或者Mapstruct,避免Apache BeanUtils或调用setter
- 禁止在循环中执行耗时的操作，如在循环中执行SQL语句/调用外部服务等

　　　　// 错误的示例：

　　

```
　　for(Long id : idList){   

　　　　　　// 循环执行SQL查询或调用外部系统接口，产生性能问题   

　　　　　　Entity entity = xxService.getEntityById(id);   

　　　　　　 ...

　　　　}
```

　　　　// 此案例的更优方案是 通过idList一次性查询获取到Entity集合，然后转换为Map<Id, Entity>供后续获取。

- 需要多次使用的可复用对象将对象单独定义，禁止多次调用取不同属性。如：

　　　　String name = userService.getUser(id).getName();

　　　　Long deptId = userService.getUser(id).getDepeId();

　　　　替换为:

　　　　User user = userService.getUser(id); String name = user.getName(), ….

- 可异步执行的耗时操作采用异步处理：使用Spring @Async 或 MQ，或夜间Timer定时
- 常用数据考虑缓存，存入Redis，设置缓存过期时间
- 需要保证写一致性的逻辑，在外层方法上添加事务 @Transactional(rollbackFor = Exception.class)

### 四、Mybatis与SQL规范

- 表名、字段名、索引等数据结构定义大小写: Oracle大写, MySQL小写。名称使用英文+下划线，并控制总长度，如 user_name。
- 表名建议采用“模块标识_”前缀，如 bas_user（如果模块库独立可省略模块名标识）
- 禁止程序中的SQL使用并行计算 /*+parallel(t,n)*/
- SQL使用标准SQL，避免出现数据库特定的语法
- 未经评审不可直接使用视图、触发器、存储过程 SQL JOIN表数量不超过3张，超过3张表需要经过评审 (拆分成多次单表查询、主表冗余、程序绑定id-name映射、根据条件动态JOIN等)。
- 合理创建索引，并尽量避免不走索引的情况: 如
  - LIKE右/任意匹配(‘%xx’, ‘%xx%’)不走索引, 换为“精确匹配=”或固定前缀的左匹配’张%’
  - 不等条件（!=、<>、NOT）不走索引，应尽量避免（转换成IN/BETWEEN等）
  - IS (NOT) NULL 不走索引，应尽量避免（如字段给定默认值，避免NULL）
  - 索引列使用函数或隐式转换都将导致索引失效，如 to_char(create_date,'yyyymmdd') = '20190102'
- 禁止手动拼接SQL语句，利用Mybatis等ORM框架的动态SQL实现。 参数使用#{} (避免${}产生SQL注入问题)。
- 禁止使用数据库处理函数 decode()，改为Java枚举或Map定义，通过id进行绑定 decode(client.TYPE, 1, '私客', 2, '店组公客', 3, '组团公客‘)
- 禁止动态拼接时强加 1=1 之类的写法，如WHERE 1=1。使用Mybatis动态SQL标签实现，如<where>，<set>，<trim>
- SQL中的参数类型确保与列定义一致，避免数据库隐式转换开销且无法使用索引，如:
  - 列定义为日期类型，参数要转换为Date日期类型进行比较:

　　　　　　CREATE_TIME <= '2019-04-14 23:59:59’

　　　　　　CREATE_TIME <= to_date('2019-04-14 00:00:00','yyyy-MM-dd HH24:mi:ss’)

- - 列定义为数字类型，参数不用String DEPT_ID = '123’
- ID主键自增的情况下，按create_time排序改为按ID排序，效果一样效率更高

### 五、检查结果

- 后端服务及其他需要自测的代码，编写对应的单元测试类，统一采用Junit，禁止直接在原Java类中写main()方法自测。

　　　　单元测试会在打包前统一运行，可及时发现受影响的代码问题（比如新代码导致了之前的代码逻辑产生问题，如果有单元测试可在打包时及时发现）

　　　　Junit单元测试类示例:

　　　

```
　public class TestApollo {

　　　　　　@Test // 标记为单元测试方法

　　　　　　public void testApolloConfig(){

　　　　　　　　String appId = Foundation.app().getAppId();

　　　　　　　　// 预期结果断言

　　　　　　　　Assert.assertNotNull(appId);

　　　　　　}

　　　　}
```



- IDE中安装代码质量检查插件: FindBugs 及 Alibaba Java Coding Guidelines

　　![img](https://img2018.cnblogs.com/blog/1244163/201906/1244163-20190605143735926-123266219.png)

### 六、安全规约

 　![img](https://img2018.cnblogs.com/blog/1244163/201906/1244163-20190605143813370-1969215247.png)