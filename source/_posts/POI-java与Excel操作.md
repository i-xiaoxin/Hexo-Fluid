---
title: POI-java与Excel操作
date: 2022-09-23 18:12:02
tags: 
- java
- POI
- Excel
index_img: /img/article8.jpg
banner_img: /img/post_banner.jpg
categories:
- utils
---

### 前言

<p class="note note-success">
    Apache POI是用Java编写的免费开源的跨平台的Java API，Apache POI提供API给Java程 序对Microsoft Office格式档案读和写的功能，其中使用最多的就是使用POI操作Excel文 件。
</p>

### 导入maven坐标

```xml
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
</dependency>
<dependency>
   <groupId>org.apache.poi</groupId>
   <artifactId>poi-ooxml</artifactId>
</dependency>
```

### POI结构说明

- HSSF提供读写Microsoft Excel XLS格式档案的功能。
- XSSF提供读写Microsoft Excel OOXML XLSX格式档案的功能。
- HWPF提供读写Microsoft Word DOC格式档案的功能。
- HSLF提供读写Microsoft PowerPoint格式档案的功能。
- HDGF提供读Microsoft Visio格式档案的功能。
- HPBF提供读Microsoft Publisher格式档案的功能。
- HSMF提供读Microsoft Outlook格式档案的功能。

### POI-Excel写入（Writer）

```java
/**
 * java-poi-excel-使用
 */
public class ExcelWriterTest {
    public static void main(String[] args) throws IOException{
        // 1。 创造工作簿 workbook
        HSSFWorkbook workbook = new HSSFWorkbook();
        // 2. 创造工作表 sheet
        HSSFSheet sheet1 = workbook.createSheet("sheet1");
        // 3. 工作行 row    表头0行
        HSSFRow row = sheet1.createRow(0);
        // 4. 单元格 cell   表头字段
        row.createCell(0).setCellValue("姓名");
        row.createCell(1).setCellValue("年龄");
        // 从1行开始创建表row 并创建cell赋值
        for (int i = 1; i < 5; i++) {
            HSSFRow rowi = sheet1.createRow(i);
            rowi.createCell(0).setCellValue("第"+i+"姓名");
            rowi.createCell(1).setCellValue("第"+i+"年龄");
        }

        // IO流输出文件
        FileOutputStream fileOutputStream = new FileOutputStream("C:\\Users\\0\\Desktop\\POI.xls");
        workbook.write(fileOutputStream);
        fileOutputStream.flush();
        fileOutputStream.close();
        System.out.println("Excel创建成功");
    }
}
```

### POI-Excel读取（Reader）

```java
/**
 * java-poi-excel-xls-Reader使用
 */
public class ExcelReaderTest {
    public static void main(String[] args) throws IOException{
        HSSFWorkbook workbook = new HSSFWorkbook(new FileInputStream("C:\\Users\\0\\Desktop\\POI.xls"));
        HSSFSheet sheet1 = workbook.getSheet("sheet1");
        for (int i = 0; i <sheet1.getLastRowNum(); i++) {
            HSSFRow rowi = sheet1.getRow(i);
            String row1 = rowi.getCell(0).getStringCellValue();
            String row2 = rowi.getCell(1).getStringCellValue();
            System.out.println(row1+"|"+row2);
        }
    }
}
```

### POI-Excel操作工具类

```java
/**
 * excel 操作工具类
 */
public class POIUtils {
    private static final String xls = "xls";

    private static final String xlsx = "xlsx";

    private static final String DATE_FORMAT = "yyyy/MM/dd";

    /**
     * 读excel文件并返回
     */
    public static List<String[]> readExcel(MultipartFile file) throws IOException{
        // 检查文件
        checkFile(file);
        // 获得Workbook工作簿对象
        Workbook workBook = getWorkBook(file);
        // 创建返回对象，把每行中的值作为一个数组，所有行作为一个集合返回
        List<String[]> list = new ArrayList<>();
        if (workBook != null) {
            for (int sheetNum = 0; sheetNum < workBook.getNumberOfSheets(); sheetNum++) {
                // 获得当前sheet工作表
                Sheet sheet = workBook.getSheetAt(sheetNum);
                if (sheet == null) {
                    continue;
                }
                // 获得当前sheet的开始行
                int firstRowNum = sheet.getFirstRowNum();
                // 获得当前sheet的结束行
                int lastRowNum = sheet.getLastRowNum();
                // 循环除了第一行的所有行
                for (int rowNum = firstRowNum + 1; rowNum <= lastRowNum; rowNum++) {
                    // 获得当前行
                    Row row = sheet.getRow(rowNum);
                    if (row == null) {
                        continue;
                    }
                    // 获得当前行的开始列
                    int firstCellNum = row.getFirstCellNum();
                    // 获得当前行的列数
                    int lastCellNum = row.getPhysicalNumberOfCells();
                    String[] cells = new String[row.getPhysicalNumberOfCells()];
                    // 循环当前行
                    for (int cellNum = firstCellNum; cellNum < lastCellNum; cellNum++) {
                        Cell cell = row.getCell(cellNum);
                        cells[cellNum] = getCellValue(cell);
                    }
                    list.add(cells);
                }
            }
            workBook.close();
        }
        return list;
    }

    // 校验文件是否合法
    public static void checkFile(MultipartFile file) throws IOException{
        // 判断文件是否存在
        if (null == file) {
            throw new FileNotFoundException("文件不存在！");
        }
        // 获得文件名
        String fileName = file.getOriginalFilename();
        // 判断文件是否是excel文件
        if (!fileName.endsWith(xls) && !fileName.endsWith(xlsx)) {
            throw new IOException(fileName + "不是excel文件");
        }
    }

    public static Workbook getWorkBook(MultipartFile file){
        // 获得文件名
        String fileName = file.getOriginalFilename();
        // 创建Workbook工作薄对象，表示整个excel
        Workbook workbook = null;
        try {
            // 获取excel文件的io流
            InputStream is = file.getInputStream();
            // 根据文件后缀名不同(xls和xlsx)获得不同的Workbook实现类对象
            if (fileName.endsWith(xls)) {
                // 2003
                workbook = new HSSFWorkbook(is);
            } else if (fileName.endsWith(xlsx)) {
                // 2007
                workbook = new XSSFWorkbook(is);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return workbook;
    }

    public static String getCellValue(Cell cell){
        String cellValue = "";
        if (cell == null) {
            return cellValue;
        }
        // 如果当前单元格内容为日期类型，需要特殊处理
        String dataFormatString = cell.getCellStyle().getDataFormatString();
        if (dataFormatString.equals("m/d/yy")) {
            cellValue = new SimpleDateFormat(DATE_FORMAT).format(cell.getDateCellValue());
            return cellValue;
        }
        // 把数字当成String来读，避免出现1读成1.0的情况
        if (cell.getCellType() == Cell.CELL_TYPE_NUMERIC) {
            cell.setCellType(Cell.CELL_TYPE_STRING);
        }
        // 判断数据的类型
        switch (cell.getCellType()) {
            case Cell.CELL_TYPE_NUMERIC: // 数字
                cellValue = String.valueOf(cell.getNumericCellValue());
                break;
            case Cell.CELL_TYPE_STRING: // 字符串
                cellValue = String.valueOf(cell.getStringCellValue());
                break;
            case Cell.CELL_TYPE_BOOLEAN: // Boolean
                cellValue = String.valueOf(cell.getBooleanCellValue());
                break;
            case Cell.CELL_TYPE_FORMULA: // 公式
                cellValue = String.valueOf(cell.getCellFormula());
                break;
            case Cell.CELL_TYPE_BLANK: // 空值
                cellValue = "";
                break;
            case Cell.CELL_TYPE_ERROR: // 故障
                cellValue = "非法字符";
                break;
            default:
                cellValue = "未知类型";
                break;
        }
        return cellValue;
    }
}
```

[1.POI官网](https://poi.apache.org/)

[2.POI常用API中文查询](https://www.yiibai.com/apache_poi/apache_poi_core_classes.html)
