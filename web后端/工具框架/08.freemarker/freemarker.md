# 📝 FreeMarker 学习笔记

## 🔍 FreeMarker 介绍

- FreeMarker 是一款 **模板引擎**：即一种基于模板和要改变的数据，并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。
- 模板编写为FreeMarker Template Language (FTL)。它是简单的，专用的语言，*不是* 像PHP那样成熟的编程语言。那意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算，之后模板显示已经准备好的数据。在模板中，你可以专注于如何展现数据，而在模板之外可以专注于要展示什么数据。

![1528820943975](img\1528820943975.png)

## 🧠 FreeMarker 语法

### 📚 基础语法种类

1. 注释，即 `<#--  -->`，介于其之间的内容会被freemarker忽略

```velocity
<#--我是一个freemarker注释-->
```

2. 插值（Interpolation）：即 **`${..}`** 部分，freemarker会用真实的值代替**`${..}`**

```velocity
Hello ${name}
```


3. FTL指令：和HTML标记类似，名字前加#予以区分，Freemarker会解析标签中的表达式或逻辑。

```velocity
<# >FTL指令</#> 
```


4. 文本，仅文本信息，这些不是freemarker的注释、插值、FTL指令的内容会被freemarker忽略解析，直接输出内容。

```velocity
<#--freemarker中的普通文本-->
我是一个普通的文本
```


### 📋 集合指令(List、Map)

#### List集合指令:
```html
<#list stus as stu>
<tr>
    <td>${stu_index+1}</td>
    <td>${stu.name}</td>
    <td>${stu.age}</td>
    <td>${stu.money}</td>
</tr>
</#list>
```


#### Map集合指令:
```html
<a href="###">方式一：通过map['keyname'].property</a><br/>
输出stu1的学生信息：<br/>
姓名：${stuMap['stu1'].name}<br/>
年龄：${stuMap['stu1'].age}<br/>
<br/>

<a href="###">方式二：通过map.keyname.property</a><br/>
输出stu2的学生信息：<br/>
姓名：${stuMap.stu2.name}<br/>
年龄：${stuMap.stu2.age}<br/>
<br/>

<#list stuMap?keys as key >
<tr>
    <td>${key_index}</td>
    <td>${stuMap[key].name}</td>
    <td>${stuMap[key].age}</td>
    <td>${stuMap[key].money}</td>
</tr>
</#list>
```


👆 上面代码解释：

- `${k_index}`：index：得到循环的下标，使用方法是在stu后边加"_index"，它的值是从0开始

### ❓ if 指令

- if 指令即判断指令，是常用的FTL指令，freemarker在解析时遇到if会进行判断，条件为真则输出if中间的内容，否则跳过内容不再输出。
- 语法：`<#if ></if>`

```html
    <#list stus as stu >
        <#if stu.name='小红'>
            <tr style="color: red">
                <td>${stu_index}</td>
                <td>${stu.name}</td>
                <td>${stu.age}</td>
                <td>${stu.money}</td>
            </tr>
            <#else >
            <tr>
                <td>${stu_index}</td>
                <td>${stu.name}</td>
                <td>${stu.age}</td>
                <td>${stu.money}</td>
            </tr>
        </#if>
    </#list>
```

![1539947776259](img\1539947776259.png)

### ➕ 运算符

#### 1️⃣ 算数运算符

FreeMarker表达式中完全支持算术运算，FreeMarker支持的算术运算符包括：

- 加法： `+`
- 减法： `-`
- 乘法： `*`
- 除法： [/](file://E:\Users\32147\Documents\Obsidian%20Vault\Java-Stack\LICENSE)
- 求模 (求余)： `%`

```html
<b>算数运算符</b>
<br/>
    100+5 运算：  ${100 + 5 }<br/>
    100 - 5 * 5运算：${100 - 5 * 5}<br/>
    5 / 2运算：${5 / 2}<br/>
    12 % 10运算：${12 % 10}<br/>
<hr>
```


#### 2️⃣ 比较运算符

- **`=`**或者**`==`**: 判断两个值是否相等
- **`!=`**: 判断两个值是否不等
- **`>`**或者**`gt`**: 判断左边值是否大于右边值
- **`>=`**或者**`gte`**: 判断左边值是否大于等于右边值
- **`<`**或者**`lt`**: 判断左边值是否小于右边值
- **`<=`**或者**`lte`**: 判断左边值是否小于等于右边值

```html
<dd>
    <#-- 日期的比较需要通过?date将属性转为data类型才能进行比较 -->
    <#if (date1?date >= date2?date)>
        形式二：使用括号形式比较时间 date1?date >= date2?date
    </#if>
</dd>
```


> ⚠️ 注意：
> - **`=`**和**`!=`**可以用于字符串、数值和日期来比较是否相等 <br>
    > **`=`**和**`!=`**两边必须是相同类型的值，否则会产生错误 <br>
    > 字符串 **`"x"`** 、**`"x "`** 、**`"X"`**比较是不等的，因为FreeMarker是精确比较 <br>
    > 其它的运行符可以作用于数字和日期，但不能作用于字符串 <br>
    > 使用**`gt`**等字母运算符代替**`>`**会有更好的效果，因为 FreeMarker会把**`>`**解释成FTL标签的结束字符 <br>
    > 可以使用括号来避免这种情况，如：**`<#if (x>y)>`** <br>

#### 3️⃣ 逻辑运算符

- 逻辑与： &&
- 逻辑或： ||
- 逻辑非： !

```html
<b>逻辑运算符</b>
    <br/>
    <#if (10 lt 12 )&&( 10  gt  5 )  >
        (10 lt 12 )&&( 10  gt  5 )  显示为 true
    </#if>
    <br/>
    <br/>
    <#if !false>
        false 取反为true
    </#if>
<hr>
```


### 🚫 空值处理

#### 1️⃣ 判断某变量是否存在使用 "??"

```html
    <#if stus??>
    <#list stus as stu>
    	......
    </#list>
    </#if>
```


#### 2️⃣ 缺失变量默认值使用 "!"

- 使用!要以指定一个默认值，当变量为空时显示默认值
    - `${name!''}`表示如果name为空显示空字符串。
- 如果是嵌套对象则建议使用（）括起来
    - `${(stu.bestFriend.name)!''}`表示，如果stu或bestFriend或name为空默认显示空字符串。

### 🔧 内建函数

#### 1️⃣ 获取某个集合的大小

**`${集合名?size}`**

#### 2️⃣ 日期格式化

- 显示年月日：**`${today?date}`**
- 显示时分秒：**`${today?time}`**
- 显示日期+时间：**`${today?datetime}`**
- 自定义格式化：**`${today?string("yyyy年MM月")}`**

#### 3️⃣ 内建函数 `c`

```java
model.addAttribute("point", 102920122);
```


point是数字型，使用${point}会显示这个数字的值，每三位使用逗号分隔。

如果不想显示为每三位分隔的数字，可以使用c函数将数字型转成字符串输出

**`${point?c}`**

#### 4️⃣ 将json字符串转成对象

一个例子：

其中用到了 `assign`标签，assign的作用是定义一个变量。

```velocity
<#assign text="{'bank':'工商银行','account':'10101920201920212'}" />
<#assign data=text?eval />
开户行：${data.bank}  账号：${data.account}
```


## 🚀 FreeMarker 快速入门

### 1️⃣ 引入依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- freemarker -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <!-- lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <!-- apache 对 java io 的封装工具库 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-io</artifactId>
        <version>1.3.2</version>
    </dependency>
</dependencies>
```


### 2️⃣ 配置文件
```yaml
server:
  port: 8881 #服务端口
spring:
  application:
    name: freemarker-demo #指定服务名
  freemarker:
    cache: false  #关闭模板缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
    suffix: .ftl               #指定Freemarker模板文件的后缀名
    template-loader-path: classpath:/templates   #模板存放位置
```


### 3️⃣ 创建模板类
```java
@Data
public class Student {
    private String name;//姓名
    private int age;//年龄
    private Date birthday;//生日
    private Float money;//钱包
}
```


### 4️⃣ 创建模板文件
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
<b>普通文本 String 展示：</b><br><br>
Hello ${name} <br>
<hr>
<b>对象Student中的数据展示：</b><br/>
姓名：${stu.name}<br/>
年龄：${stu.age}
<hr>
</body>
</html>
```


### 5️⃣ 创建测试类
```java
@Controller
public class HelloController {

    @GetMapping("/basic")
    public String test(Model model) {
        //1.纯文本形式的参数
        model.addAttribute("name", "freemarker");
        //2.实体类相关的参数
        Student student = new Student();
        student.setName("小明");
        student.setAge(18);
        model.addAttribute("stu", student);

        return "01-basic";
    }
}
```
```java
@SpringBootTest(classes = FreemarkerDemoApplication.class)
@RunWith(SpringRunner.class)
public class FreemarkerTest {

    @Autowired
    private Configuration configuration;

    @Test
    public void test() throws IOException, TemplateException {
        //freemarker的模板对象，获取模板
        Template template = configuration.getTemplate("02-list.ftl");
        Map params = getData();
        //合成
        //第一个参数 数据模型
        //第二个参数  输出流
        template.process(params, new FileWriter("d:/list.html"));
    }

    private Map getData() {
        Map<String, Object> map = new HashMap<>();

        //小强对象模型数据
        Student stu1 = new Student();
        stu1.setName("小强");
        stu1.setAge(18);
        stu1.setMoney(1000.86f);
        stu1.setBirthday(new Date());

        //小红对象模型数据
        Student stu2 = new Student();
        stu2.setName("小红");
        stu2.setMoney(200.1f);
        stu2.setAge(19);

        //将两个对象模型数据存放到List集合中
        List<Student> stus = new ArrayList<>();
        stus.add(stu1);
        stus.add(stu2);

        //向map中存放List集合数据
        map.put("stus", stus);

        //创建Map数据
        HashMap<String, Student> stuMap = new HashMap<>();
        stuMap.put("stu1", stu1);
        stuMap.put("stu2", stu2);
        //向map中存放Map数据
        map.put("stuMap", stuMap);

        //返回Map
        return map;
    }
}
```