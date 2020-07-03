        深夜，在东莞，7天酒店，打开电脑，访问国内最大的同性交友网站。

​        日常开发中，导出导入场景非常多，尤其是对于后台管理更是一个列表一个导出，如果从导出的业务中抽离出复用代码，专注于逻辑开发，对于开发者而言非常重要。前有使用POI，但作者还是更喜EasyExcel的简洁高效不拖沓，所以特意写篇文章记录下。

# 准备工作

​        准备工作是看文档了解EasyExcel吗？不，我们直接上手吧！我发现最近的业务里面，最简单的例子已经应付下来了！所以准备工作自然只需导入EasyExcel的jar包，这里我们由于是springboot项目，所以直接使用maven。直接上最新的版本了！pom.xml给它加上：

```java
<dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>2.2.5</version>
</dependency>
```

# 导出 

​        准备工作已经完成，导出开始，首先需要一个Bean类，导出的字段和Excel文件的字段一样即可。@Data是用了lombok，@ExcelProperty则包含了Excel首行的名称和字段所在位置，从0开始，不能重复。

```java
@Data
public class ExportVo {
  
  @ExcelProperty(value = "名称", index = 0)
  private String name;

  @ExcelProperty(value = "时间", index = 1)
  private Date time;
  
  @NumberFormat("#.##%")
  @ExcelProperty(value = "完成率", index = 2)
  private Float rate;
  
}
```

​        接下来是逻辑实现：

```java
  @PostMapping("/export")
  public void export(@RequestBody ExportDto dto, HttpServletResponse response)
          throws IOException {
    String fileName = "统计表";
    ExcelUtil.download(response, fileName, ExportVo.class,
            getExportVoList());
  }

  private List getExportVoList() {
    //  这里主要是获取List<ExportVo>内容，不提供实现了，需要说下注意点。
      1、导出列表应该有时间之类的限制（例如最近一个月），避免导出Excel过大，过大Excel一般采用分sheet导出（尤其xls文件，有65535条数限制）、分文件打包导出，上传文件服务器异步导出，不可突破的最大导出（限死5w条）。
      2、一来考虑数据库查询和内存压力，二来分页插件可能存在最大页数限制，所以较多条数时必须做分页查询，再组合List对象。
      3、一般从数据库直接查出数据不满足导出字段需要，有必要时需要做字段转换。
  }
```

​      导出的处理类：

```java
public class ExcelUtil {

  public static void download(HttpServletResponse response, String fileName,
                              Class cls, List dataList)
          throws IOException {
    response.setContentType("application/vnd.ms-excel");
    response.setCharacterEncoding("utf-8");
    String fname = URLEncoder.encode(fileName, "utf-8");
    response.setHeader("Content-disposition",
            "attachment;filename=" + fname + ExcelTypeEnum.XLSX.getValue());

    LongestMatchColumnWidthStyleStrategy longestMatchColumnWidthStyleStrategy =
            new LongestMatchColumnWidthStyleStrategy();
    EasyExcel.write(response.getOutputStream(), cls)
            .sheet("sheet1")
            .registerWriteHandler(longestMatchColumnWidthStyleStrategy)

            .doWrite(dataList);
    response.flushBuffer();
  }
```

​        启动程序，使用postman试下，直接点Send的话会返回一堆乱码，选择Send and Download则可导出Excel，不过文件名是URL编码过的，这个文件名编码问题在浏览器则不会存在。

![image-20200702002721411](C:\Users\lin\AppData\Roaming\Typora\typora-user-images\image-20200702002721411.png)

# 导入

​        导入是为了减少人工录入大量数据的烦恼，挺好的。

```java
  @Autowired
  private ImportService importService;

  @PostMapping("/import")
  public void import(
          @RequestParam(value = "file") MultipartFile file,
          @Min(1) @RequestParam("type") int type
  ) throws IOException {
    ImportQueryDto dto = new ImportQueryDto();
    dto.setType(type);
    // ImportDto是导入对应类，UploadDataListener是处理类，逻辑处理服务需要以参数形式传入
    EasyExcel.read(file.getInputStream(), ImportDto.class,
            new UploadDataListener(importService)).sheet().doRead();
  }

```

​     导入Excel对应类，ExcelProperty对应导入字段的首部。

```java
@Data
public class ImportDto {
  @ExcelProperty("名称")
  private String name;
  @ExcelProperty("数量")
  private Integer num;
}
```

​       导入处理：

```java
public class UploadDataListener
        extends AnalysisEventListener<ImportDto> {
  //  一次导入多少便入库，避免大量入库
  private static final int BATCH_COUNT = 50;
  //  存放导入列表
  List<ImportDto> list = new ArrayList<>();

  private ImportService importService;

  public UploadDataListener(ImportService importService) {
    this.importService = importService;
  }

  @Override
  public void invoke(ImportDto importDto, AnalysisContext analysisContext) {
    list.add(importDto);
    if (list.size() >= BATCH_COUNT) {
      saveData();
      list.clear();
    }
  }

  @Override
  public void doAfterAllAnalysed(AnalysisContext analysisContext) {
    saveData();
  }

  private void saveData() {
    List<ImportModel> importModelList = new ArrayList<>();
    for (ImportDto dto : list) {
      ImportModel model = new ImportModel();
      model.setName(dto.getName());
      model.setNum(dto.getNum())
      importModelList.add(model);
    }
    importService.saveBatch(importModelList);
  }

}

```

​        实际上，导入文件的内容是需要校验的，数据格式校验等，需要询问产品是否忽略错误的数据，只导入正常的数据，然后怎么友好提示错误的内容，通过返回一个包含错误信息Excel或提示框，更进一步的提示，则是提示到哪一个格子有问题。

​       睡觉。