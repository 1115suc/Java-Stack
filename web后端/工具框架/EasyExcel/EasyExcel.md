# 📊 EasyExcel 数据处理工具

## 🔧 项目集成 EasyExcel

### 🚀 启动项目并使用

- 在 Spring Boot 项目中引入 EasyExcel 依赖后即可开始使用：
    - 📦 官方文档：https://easyexcel.opensource.alibaba.com/

### 📦 引入依赖

```xml
<!-- EasyExcel 核心依赖 -->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>easyexcel</artifactId>
  <version>3.4.4</version>
</dependency>
```


### ⚙️ 基础配置与使用

#### ✅ 创建实体类（用于读写 Excel）

```java
@Data
@HeadRowHeight(value = 35) // 表头行高
@ContentRowHeight(value = 25) // 内容行高
@ColumnWidth(value = 50) // 列宽
public class EasyExcelExcel {
    @ExcelProperty(value = {"学生名称"}, index = 1)
    @ColumnWidth(30)
    private String name;
    
    @ExcelProperty(value = {"学号"}, index = 2)
    @ColumnWidth(20)
    private String studentNum;

    @ExcelProperty(value = "学生性别", index = 3)
    @ColumnWidth(20)
    private String sex;

    @ExcelProperty(value = {"学生年龄"}, index = 4)
    @ExcelIgnore // 忽略该字段不参与导出
    @ColumnWidth(20)
    private Integer age;

    @ExcelProperty(value = {"出生日期"}, index = 5)
    @DateTimeFormat("yyyy-MM-dd")
    @ColumnWidth(30)
    private Date birthday;
}
```

#### 📥 读取 Excel 文件

```java
public class ExcelReader {
    public static void readExcel(String fileName) {
        EasyExcel.read(fileName, User.class, new UserDataListener())
                .sheet()
                .doRead();
    }
}

// 自定义监听器处理每一行数据
public class UserDataListener extends AnalysisEventListener<User> {
    @Override
    public void invoke(User user, AnalysisContext context) {
        System.out.println("解析到一条数据：" + user);
        // 可以在这里保存到数据库等操作
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        System.out.println("所有数据解析完成！");
    }
}
```


#### 📤 写入 Excel 文件

```java
public class ExcelWriter {
    public static void writeExcel(String fileName, List<User> dataList) {
        EasyExcel.write(fileName, User.class)
                .sheet("用户信息")
                .doWrite(dataList);
    }
}
```

#### 📝 使用场景

- 导出业务数据

```java
@Override
    public void stockExport(HttpServletResponse response, Integer page, Integer pageSize) {
        try {
            ...
            if (CollectionUtils.isEmpty(infos)) {
                //响应提示信息
                RequestInfoUtil.setUtf8(response);
                R<Object> r = R.error(ResponseCode.NO_RESPONSE_DATA);
              	response.setContentType("application/json");
                response.setCharacterEncoding("utf-8");
                response.getWriter().write(new ObjectMapper().writeValueAsString(r));
                return;
            }
            //设置响应excel文件格式类型
            response.setContentType("application/vnd.ms-excel");
            //2.设置响应数据的编码格式
            response.setCharacterEncoding("utf-8");
            //3.设置默认的文件名称
            // 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
            String fileName = URLEncoder.encode("stockRt", "UTF-8");
            //设置默认文件名称：兼容一些特殊浏览器
            response.setHeader("content-disposition", "attachment;filename=" + fileName + ".xlsx");
            //4.响应excel流
            EasyExcel.write(response.getOutputStream(),StockUpdownDomain.class)
                     .sheet("股票信息")
                     .doWrite(infos);
        } catch (IOException e) {
            ...
        }
    }
```

---

## 🏷️ 注解使用指南

### 📌 常用注解说明

| 注解 | 使用位置 | 功能 |
|------|----------|------|
| `@ExcelProperty` | 实体类字段 | 指定列名及顺序 |
| `@ExcelIgnore` | 实体类字段 | 忽略该字段不参与导入导出 |
| `@ContentRowHeight` | 类上 | 设置内容行高 |
| `@HeadRowHeight` | 类上 | 设置表头行高 |
| `@ColumnWidth` | 类上 / 字段上 | 设置列宽 |

### 📝 示例代码

```java
@Data
@ContentRowHeight(20)
@HeadRowHeight(25)
@ColumnWidth(20)
public class Employee {

    @ExcelProperty(value = "员工编号", index = 0)
    private String empNo;

    @ExcelProperty(value = "员工姓名", index = 1)
    private String name;

    @ExcelProperty(value = "部门", index = 2)
    private String department;

    @ExcelIgnore
    private String password; // 密码不需要导出
}
```


---

## 📖 EasyExcel 高级特性

### 🔄 多sheet读写

```java
// 写多个Sheet
public class MultiSheetWriter {
    public static void writeMultiSheet(String fileName, Map<String, List<?>> sheetDataMap) {
        ExcelWriter excelWriter = EasyExcel.write(fileName).build();
        int sheetIndex = 0;
        for (Map.Entry<String, List<?>> entry : sheetDataMap.entrySet()) {
            WriteSheet writeSheet = EasyExcel.writerSheet(sheetIndex++, entry.getKey()).head(entry.getValue().get(0).getClass()).build();
            excelWriter.write(entry.getValue(), writeSheet);
        }
        excelWriter.finish();
    }
}
```


### 🛠 自适应列宽

```java
public class AutoWidthWriter {
    public static void autoWidthWrite(String fileName, List<User> dataList) {
        EasyExcel.write(fileName, User.class)
                .registerWriteHandler(new LongestMatchColumnWidthStyleStrategy())
                .sheet("自适应宽度")
                .doWrite(dataList);
    }
}
```


### 🧪 异常处理机制

```java
public class ExceptionHandlingListener extends AnalysisEventListener<User> {
    @Override
    public void onException(Exception exception, AnalysisContext context) throws Exception {
        if (exception instanceof ExcelDataConvertException) {
            ExcelDataConvertException excelDataConvertException = (ExcelDataConvertException) exception;
            System.err.println("第" + excelDataConvertException.getRowIndex() +
                    "行，第" + excelDataConvertException.getColumnIndex() + "列解析异常");
        } else {
            throw exception;
        }
    }

    @Override
    public void invoke(User user, AnalysisContext context) {
        // 正常处理逻辑...
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 解析完成后执行的操作...
    }
}
```


---

## 🎯 应用场景建议

- **批量导入导出业务数据**
- **报表统计分析**
- **定时任务中的数据同步**
- **后台管理系统中的数据维护**

> 💡 提示：
> - 使用 `@ExcelProperty` 的时候可以通过设置 `converter` 属性来自定义转换器
> - 对于大数据量读写推荐使用监听器模式避免内存溢出