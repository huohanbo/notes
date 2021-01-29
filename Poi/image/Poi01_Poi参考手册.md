# 参考资料
> [Apache POI - the Java API for Microsoft Documents](http://poi.apache.org/index.html)
> [Apache POI - Component APIs](http://poi.apache.org/components/index.html)

--------------------------------------------------
Component APIs

# 组件总览
## POI项目组件
Apache POI项目是用于基于Microsoft的OLE2复合文档格式开发文件格式的纯Java端口的主项目。Microsoft Office文档以及使用MFC属性集的程序将OLE 2复合文档格式用于序列化其文档对象。

Apache POI还是用于基于Office Open XML（ooxml）开发文件格式的纯Java端口的主项目。OOXML是ECMA / ISO标准化工作的一部分。

POI结构说明：
1. HSSF提供读写Microsoft Excel XLS格式档案的功能。
2. XSSF提供读写Microsoft Excel OOXML XLSX格式档案的功能。
3. HWPF提供读写Microsoft Word DOC格式档案的功能。
4. HSLF提供读写Microsoft PowerPoint格式档案的功能。
5. HDGF提供读Microsoft Visio格式档案的功能。
6. HPBF提供读Microsoft Publisher格式档案的功能。
7. HSMF提供读Microsoft Outlook格式档案的功能。

### POIFS 用于OLE2文档
POIFS是POI中最古老，最稳定的部分。这是OLE2复合文档格式到纯Java的移植。它同时支持读写功能。
根据定义，我们用于二进制（非XML）Microsoft Office格式的所有组件最终都依赖于它。

### HSSF和XSSF 用于Excel文档
HSSF是我们将Microsoft Excel 97（-2003）文件格式（BIFF8）移植到纯Java的端口。
XSSF是Microsoft Excel XML（2007+）文件格式（OOXML）到纯Java的移植。
SS是一个使用通用API为两种格式提供通用支持的软件包。
它们都支持读写功能。

### HWPF和XWPF 用于Word文档
HWPF是我们将Microsoft Word 97（-2003）文件格式移植到纯Java的端口。
它支持读取和有限的写入功能。它还为较旧的Word 6和Word 95格式提供了简单的文本提取支持。

我们还在根据OOXML规范为WordprocessingML（2007+）格式开发XWPF。
这提供了对较简单文件的读写支持，以及文本提取功能。

### HSLF和XSLF 用于PowerPoint文档
HSLF是我们将Microsoft PowerPoint 97（-2003）文件格式移植到纯Java的端口。
它支持读写功能。

我们还在根据OOXML规范为PresentationML（2007+）格式开发XSLF。

### HPSF 用于OLE2文档
HPSF是OLE 2属性集格式到纯Java的移植。
属性集通常用于存储文档的属性（标题，作者，最后修改日期等），但是它们也可以用于特定于应用程序的目的。
HPSF支持读取和写入属性。

### HDGF和XDGF 用于Visio文档
HDGF是我们将Microsoft Visio 97（-2003）文件格式移植到纯Java的端口。
它目前仅支持非常低的阅读水平，并且支持简单的文本提取。

XDGF是Microsoft Visio XML（.vsdx）文件格式到纯Java的移植。
它比HDGF支持更多。

### HPBF 用于Publisher文档
HPBF是我们将Microsoft Publisher 98（-2007）文件格式移植到纯Java的端口。
目前，它仅支持低水平读取大约一半的文件部分，并支持简单的文本提取。

### HMEF 用于Outlook附件
HMEF是Microsoft TNEF（传输中性编码格式）文件格式到纯Java的移植。
Outlook有时会使用TNEF对消息进行编码，通常会以winmail.dat的形式出现。
HMEF当前仅支持较低级别的阅读，但我们希望添加文本和附件提取。

### HSMF 用于Outlook信息
HSMF是Microsoft Outlook消息文件格式到纯Java的移植。
目前，它仅包含MSG文件的某些文本内容以及一些附件。进一步的支持和文档进展缓慢。

目前，建议用户参考单元测试以供使用。

## POI组件图
略

# Excel (HSSF/XSSF)
## 总览
HSSF是POI项目对Excel '97（-2007）文件格式的纯Java实现。
XSSF是POI项目对Excel 2007 OOXML（.xlsx）文件格式的纯Java实现。

HSSF和XSSF提供了读取电子表格，创建，修改，读取和写入XLS电子表格的方法。他们提供：
+ low level API，用于特殊需要实现
+ eventmodel API，用于实现高效的只读访问XLS文件
+ usermodel api，用于创建，读取和修改XLS文件

生成电子表格的另一种方法是通过Cocoon序列化器（但是仍将间接使用HSSF）。使用Cocoon，可以通过简单地应用样式表并指定序列化程序来序列化任何XML数据源（例如，可能是在SQL中输出的ESQL页面）。

如果只是在读取电子表格数据，则根据文件格式，使用 org.apache.poi.hssf.eventusermodel包 或 org.apache.poi.xssf.eventusermodel包 中的eventmodel api。

如果要修改电子表格数据，请使用usermodel api。也可以通过这种方式生成电子表格。

注意，与低级别的eventusermodel相比，usermodel具有更高的内存占用量，但是其主要优点是使用起来更加简单。
注意，由于新的XSSF支持的Excel 2007 OOXML（.xlsx）文件是基于XML的，因此处理它们的内存占用量比旧的HSSF支持的（.xls）二进制文件要高。

### SXSSF
从**3.8-beta3**开始，POI提供了基于XSSF的低内存占用的SXSSF API。

SXSSF是XSSF的API兼容流扩展，可用于必须生成非常大的电子表格且堆空间有限的情况。
SXSSF通过限制对滑动窗口内的行的访问来实现其低内存占用，而XSSF允许对文档中的所有行进行访问。
不再存在于窗口中的较旧的行由于被写入磁盘而变得不可访问。

在自动刷新模式下，可以指定访问窗口的大小，以在内存中保留一定数量的行。
当达到该值时，创建额外的一行会导致索引最低的行从访问窗口中删除并写入磁盘。
或者，可以将窗口大小设置为动态增长。
可以根据需要通过显式调用flushRows（int keepRows）定期对其进行修剪。

由于实现的流性质，与XSSF相比存在以下限制：
+ 在某个时间点只能访问有限数量的行。
+ 不支持Sheet.clone（）。
+ 不支持公式评估。

### 基本API
```
// 创建工作簿
Workbook wb = new HSSFWorkbook();
Workbook wb = new XSSFWorkbook();

// 创建表（通过工作簿）
Sheet sheet1 = wb.createSheet("sheet1");

// 创建单元格格式（通过工作簿，单元格格式可复用）
CellStyle cellStyle = wb.createCellStyle();
cellStyle.setDataFormat(createHelper.createDataFormat().getFormat("m/d/yy h:mm"));

// 创建行（通过表，行从0开始）
Row row = sheet.createRow(0);

// 创建单元格（通过行）
Cell cell = row.createCell(0);
// 设置单元格的值
cell.setCellValue(1);
// 设置单元格的格式
cell.setCellStyle(cellStyle);

// 保存文件（通过工作簿）
try (OutputStream fileOut = new FileOutputStream("workbook.xlsx")) {
    wb.write(fileOut);
}

```

## 快速入门
### !!!打开工作薄
当打开一个工作簿（.xls HSSFWorkbook 或.xlsx XSSFWorkbook）时，可以从File 或 InputStream加载工作簿。
使用File对象可以减少内存消耗，而InputStream需要更多内存，因为它必须缓冲整个文件。

如果使用WorkbookFactory，会使打开变得容易：
```
// Use a file
Workbook wb = WorkbookFactory.create(new File("MyExcel.xls"));

// Use an InputStream, needs more memory
Workbook wb = WorkbookFactory.create(new FileInputStream("MyExcel.xlsx"));
```

如果直接使用HSSFWorkbook或XSSFWorkbook，通常应遍历POIFSFileSystem 或 OPCPackage，以完全控制生命周期（包括完成后关闭文件）：
```
// HSSFWorkbook, File
POIFSFileSystem fs = new POIFSFileSystem(new File("file.xls"));
HSSFWorkbook wb = new HSSFWorkbook(fs.getRoot(), true);
....
fs.close();

// HSSFWorkbook, InputStream, needs more memory
POIFSFileSystem fs = new POIFSFileSystem(myInputStream);
HSSFWorkbook wb = new HSSFWorkbook(fs.getRoot(), true);

// XSSFWorkbook, File
OPCPackage pkg = OPCPackage.open(new File("file.xlsx"));
XSSFWorkbook wb = new XSSFWorkbook(pkg);
....
pkg.close();

// XSSFWorkbook, InputStream, needs more memory
OPCPackage pkg = OPCPackage.open(myInputStream);
XSSFWorkbook wb = new XSSFWorkbook(pkg);
....
pkg.close();
```

### 遍历行和单元格
遍历工作簿中的所有工作表，工作表中的所有行，行中的所有单元格，可以通过简单的for循环来实现。
通过调用workbook.sheetIterator（）， sheet.rowIterator（）和row.cellIterator（）或隐式使用for-each循环，可以使用这些迭代器。
请注意，rowIterator和cellIterator遍历已创建的行或单元格，跳过空的行和单元格。
```
for (Sheet sheet : wb ) {
    for (Row row : sheet) {
        for (Cell cell : row) {
            // Do something here
        }
    }
}
```

#### 控制空的单元格
在某些情况下，需要完全控制如何处理丢失或空白的行和单元格，并且需要确保访问每个单元格，而不仅仅是访问文件中定义的那些单元格。
在这种情况下，应获取一行的第一列和最后一列信息，然后调用getCell（int，MissingCellPolicy） 来获取单元格。
使用 MissingCellPolicy 控制空白或空单元格的处理方式：
```
// 决定要处理的行
int rowStart = Math.min(15, sheet.getFirstRowNum());
int rowEnd = Math.max(1400, sheet.getLastRowNum());
for (int rowNum = rowStart; rowNum < rowEnd; rowNum++) {
   Row r = sheet.getRow(rowNum);
   if (r == null) {
      // 整行都是空的，根据需要处理
      continue;
   }
   int lastColumn = Math.max(r.getLastCellNum(), MY_MINIMUM_COLUMN_COUNT);
   for (int cn = 0; cn < lastColumn; cn++) {
      Cell c = r.getCell(cn, Row.RETURN_BLANK_AS_NULL);
      if (c == null) {
          // 单元格内容为空
      } else {
         // 单元格内容不为空，对单元格内容进行一些有用的操作
      }
   }
}
```

### 获取单元格内容
要获取单元格的内容，您首先需要知道它是哪种单元格（例如，将字符串单元格作为其数字内容将获得NumberFormatException）。
因此，您将需要打开单元格的类型，然后为该单元格调用适当的getter方法。

在下面的代码中，我们遍历一张表格的每个单元格，打印出该单元格的引用，然后打印出该单元格的内容。
```
DataFormatter formatter = new DataFormatter();
Sheet sheet1 = wb.getSheetAt(0);
for (Row row : sheet1) {
    for (Cell cell : row) {
        CellReference cellRef = new CellReference(row.getRowNum(), cell.getColumnIndex());
        System.out.print(cellRef.formatAsString());
        System.out.print(" - ");
        
		// 通过获取单元格值并应用任何数据格式（日期，0.00、1.23e9，$ 1.23等）来获取显示在单元格中的文本
        String text = formatter.formatCellValue(cell);
        System.out.println(text);
        
		// 或者，获取值后并自行格式化
        switch (cell.getCellType()) {
            case CellType.STRING:
                System.out.println(cell.getRichStringCellValue().getString());
                break;
            case CellType.NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    System.out.println(cell.getDateCellValue());
                } else {
                    System.out.println(cell.getNumericCellValue());
                }
                break;
            case CellType.BOOLEAN:
                System.out.println(cell.getBooleanCellValue());
                break;
            case CellType.FORMULA:
                System.out.println(cell.getCellFormula());
                break;
            case CellType.BLANK:
                System.out.println();
                break;
            default:
                System.out.println();
        }
    }
}
```

### 获取文本内容
对于大多数文本提取要求，标准ExcelExtractor类应提供您所需要的全部。
```
try (InputStream inp = new FileInputStream("workbook.xls")) {
    HSSFWorkbook wb = new HSSFWorkbook(new POIFSFileSystem(inp));
    ExcelExtractor extractor = new ExcelExtractor(wb);
    extractor.setFormulasNotResults(true);
    extractor.setIncludeSheetNames(false);
    String text = extractor.getText();
    wb.close();
}
```


### 创建工作簿
```
Workbook wb = new HSSFWorkbook();
...
try  (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}

Workbook wb = new XSSFWorkbook();
...
try (OutputStream fileOut = new FileOutputStream("workbook.xlsx")) {
    wb.write(fileOut);
}
```

### 创建工作表
```
Workbook wb = new HSSFWorkbook();  // or new XSSFWorkbook();
Sheet sheet1 = wb.createSheet("new sheet");

// 请注意，工作表名称为不能超过31个字符，且不得包含以下任何字符：
// 0x0000，0x0003，冒号（:)，反斜杠（\），星号（*），问号（？），正斜杠（/），打开方括号（[），右方括号（]）
Sheet sheet2 = wb.createSheet("second sheet");

// 可以使用org.apache.poi.ss.util.WorkbookUtil＃createSafeSheetName（String nameProposal）}
// 安全地创建有效名称，此程序将无效字符替换为空格（' '）
String safeName = WorkbookUtil.createSafeSheetName("[O'Brien's sales*?]"); // returns " O'Brien's sales   "
Sheet sheet3 = wb.createSheet(safeName);
try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 创建单元格
```
Workbook wb = new HSSFWorkbook();

CreationHelper createHelper = wb.getCreationHelper();

Sheet sheet = wb.createSheet("new sheet");

// 创建一个行并在其中放入一些单元格。行从0开始。
Row row = sheet.createRow(0);

// 创建一个单元格并在其中放置一个值
Cell cell = row.createCell(0);
cell.setCellValue(1);

// 或一行执行
row.createCell(1).setCellValue(1.2);
row.createCell(2).setCellValue(createHelper.createRichTextString("This is a string"));
row.createCell(3).setCellValue(true);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 创建日期单元格
```
Workbook wb = new HSSFWorkbook();

CreationHelper createHelper = wb.getCreationHelper();

Sheet sheet = wb.createSheet("new sheet");

Row row = sheet.createRow(0);

// 创建一个单元格并在其中放置一个日期值。第一个单元格未设置样式
Cell cell = row.createCell(0);
cell.setCellValue(new Date());

// 我们将第二个单元格设置为日期（和时间）。
// 从工作簿中创建新的单元格样式很重要，否则最终可能会修改内置样式并不仅影响此单元格，而且影响其他单元格。
CellStyle cellStyle = wb.createCellStyle();
cellStyle.setDataFormat(createHelper.createDataFormat().getFormat("m/d/yy h:mm"));

cell = row.createCell(1);
cell.setCellValue(new Date());
cell.setCellStyle(cellStyle);

// 您也可以将日期设置为java.util.Calendar
cell = row.createCell(2);
cell.setCellValue(Calendar.getInstance());
cell.setCellStyle(cellStyle);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 创建不同类型的单元格
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("new sheet");
Row row = sheet.createRow(2);
row.createCell(0).setCellValue(1.1);
row.createCell(1).setCellValue(new Date());
row.createCell(2).setCellValue(Calendar.getInstance());
row.createCell(3).setCellValue("a string");
row.createCell(4).setCellValue(true);
row.createCell(5).setCellType(CellType.ERROR);
// Write the output to a file
try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 设置单元格对齐方式
```
public static void main(String[] args) throws Exception {
    Workbook wb = new XSSFWorkbook();
	
    Sheet sheet = wb.createSheet();
	
    Row row = sheet.createRow(2);
    row.setHeightInPoints(30);
	
    createCell(wb, row, 0, HorizontalAlignment.CENTER, VerticalAlignment.BOTTOM);
    createCell(wb, row, 1, HorizontalAlignment.CENTER_SELECTION, VerticalAlignment.BOTTOM);
    createCell(wb, row, 2, HorizontalAlignment.FILL, VerticalAlignment.CENTER);
    createCell(wb, row, 3, HorizontalAlignment.GENERAL, VerticalAlignment.CENTER);
    createCell(wb, row, 4, HorizontalAlignment.JUSTIFY, VerticalAlignment.JUSTIFY);
    createCell(wb, row, 5, HorizontalAlignment.LEFT, VerticalAlignment.TOP);
    createCell(wb, row, 6, HorizontalAlignment.RIGHT, VerticalAlignment.TOP);
    
    try (OutputStream fileOut = new FileOutputStream("xssf-align.xlsx")) {
        wb.write(fileOut);
    }
    wb.close();
}

/**
 * 创建一个单元格并以某种方式对齐它
 *
 * @param wb     the workbook
 * @param row    the row to create the cell in
 * @param column the column number to create the cell in
 * @param halign the horizontal alignment for the cell.
 * @param valign the vertical alignment for the cell.
 */
private static void createCell(Workbook wb, Row row, int column, HorizontalAlignment halign, VerticalAlignment valign) {
    
	Cell cell = row.createCell(column);
    cell.setCellValue("Align It");
	
    CellStyle cellStyle = wb.createCellStyle();
    cellStyle.setAlignment(halign);
    cellStyle.setVerticalAlignment(valign);
    cell.setCellStyle(cellStyle);
}

```

### 设置单元格边框
```
Workbook wb = new HSSFWorkbook();

Sheet sheet = wb.createSheet("new sheet");

Row row = sheet.createRow(1);

Cell cell = row.createCell(1);
cell.setCellValue(4);

// Style the cell with borders all around.
CellStyle style = wb.createCellStyle();
style.setBorderBottom(BorderStyle.THIN);
style.setBottomBorderColor(IndexedColors.BLACK.getIndex());
style.setBorderLeft(BorderStyle.THIN);
style.setLeftBorderColor(IndexedColors.GREEN.getIndex());
style.setBorderRight(BorderStyle.THIN);
style.setRightBorderColor(IndexedColors.BLUE.getIndex());
style.setBorderTop(BorderStyle.MEDIUM_DASHED);
style.setTopBorderColor(IndexedColors.BLACK.getIndex());
cell.setCellStyle(style);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}

wb.close();
```

### 设置单元格填充和颜色
```
Workbook wb = new XSSFWorkbook();

Sheet sheet = wb.createSheet("new sheet");

Row row = sheet.createRow(1);

// BackgroundColor 填充背景
CellStyle style = wb.createCellStyle();
style.setFillBackgroundColor(IndexedColors.AQUA.getIndex());
style.setFillPattern(FillPatternType.BIG_SPOTS);
Cell cell = row.createCell(1);
cell.setCellValue("X");
cell.setCellStyle(style);

// ForegroundColor 填充前景，而不是字体颜色。
style = wb.createCellStyle();
style.setFillForegroundColor(IndexedColors.ORANGE.getIndex());
style.setFillPattern(FillPatternType.SOLID_FOREGROUND);
cell = row.createCell(2);
cell.setCellValue("X");
cell.setCellStyle(style);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 设置字体
```
Workbook wb = new HSSFWorkbook();

Sheet sheet = wb.createSheet("new sheet");

Row row = sheet.createRow(1);

// 创建新字体
Font font = wb.createFont();
font.setFontHeightInPoints((short)24);
font.setFontName("Courier New");
font.setItalic(true);
font.setStrikeout(true);

// 字体会设置到样式中，所以需要创建一个新的样式
CellStyle style = wb.createCellStyle();
style.setFont(font);

Cell cell = row.createCell(1);
cell.setCellValue("This is a test of fonts");
cell.setCellStyle(style);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();

```

请注意，工作簿中唯一字体的最大数量限制为32767。您应该在应用程序中重新使用字体，而不是为每个单元格创建字体。例子：

错误：
```
for (int i = 0; i < 10000; i++) {
    Row row = sheet.createRow(i);
	
    Cell cell = row.createCell(0);
	
    CellStyle style = workbook.createCellStyle();
	
    Font font = workbook.createFont();
    font.setBoldweight(Font.BOLDWEIGHT_BOLD);
	
    style.setFont(font);
	
    cell.setCellStyle(style);
}
```

正确：
```
CellStyle style = workbook.createCellStyle();

Font font = workbook.createFont();
font.setBoldweight(Font.BOLDWEIGHT_BOLD);

style.setFont(font);

for (int i = 0; i < 10000; i++) {
    Row row = sheet.createRow(i);
	
    Cell cell = row.createCell(0);
    cell.setCellStyle(style);
}
```

### 设置颜色
HSSF：
```
HSSFWorkbook wb = new HSSFWorkbook();

HSSFSheet sheet = wb.createSheet();

HSSFRow row = sheet.createRow(0);

HSSFCell cell = row.createCell(0);

cell.setCellValue("Default Palette");

// 使用标准调色板中的某些颜色。
HSSFCellStyle style = wb.createCellStyle();
style.setFillForegroundColor(HSSFColor.LIME.index);
style.setFillPattern(FillPatternType.SOLID_FOREGROUND);

HSSFFont font = wb.createFont();
font.setColor(HSSFColor.RED.index);
style.setFont(font);

cell.setCellStyle(style);

// 使用默认调色板保存
try (OutputStream out = new FileOutputStream("default_palette.xls")) {
    wb.write(out);
}


// 现在，让我们替换调色板中的RED和LIME，具有更有吸引力的组合
cell.setCellValue("Modified Palette");

// 为工作簿创建自定义调色板
HSSFPalette palette = wb.getCustomPalette();

// 用freebsd.org red替换标准red
palette.setColorAtIndex(HSSFColor.RED.index,
        (byte) 153,  //RGB red (0-255)
        (byte) 0,    //RGB green
        (byte) 0     //RGB blue
);

// 用freebsd.org gold代替石灰
palette.setColorAtIndex(HSSFColor.LIME.index, (byte) 255, (byte) 204, (byte) 102);

// 保存修改后的调色板。请注意，无论我们使用以前的RED还是LIME，新颜色神奇地出现。
try (out = new FileOutputStream("modified_palette.xls")) {
    wb.write(out);
}
```

XSSF：
```
XSSFWorkbook wb = new XSSFWorkbook();

XSSFSheet sheet = wb.createSheet();

XSSFRow row = sheet.createRow(0);

XSSFCell cell = row.createCell( 0);

cell.setCellValue("custom XSSF colors");

XSSFCellStyle style1 = wb.createCellStyle();
style1.setFillForegroundColor(new XSSFColor(new java.awt.Color(128, 0, 128), new DefaultIndexedColorMap()));
style1.setFillPattern(FillPatternType.SOLID_FOREGROUND);
```

### 合并单元格
```
Workbook wb = new HSSFWorkbook();

Sheet sheet = wb.createSheet("new sheet");

Row row = sheet.createRow(1);

Cell cell = row.createCell(1);
cell.setCellValue("This is a test of merging");

// 合并单元格（上下左右）
sheet.addMergedRegion(new CellRangeAddress(
        1, //first row (0-based)
        1, //last row  (0-based)
        1, //first column (0-based)
        2  //last column  (0-based)
));

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 读取并重写工作簿
```
try (InputStream inp = new FileInputStream("workbook.xls")) {
	
    Workbook wb = WorkbookFactory.create(inp);
	
    Sheet sheet = wb.getSheetAt(0);
	
    Row row = sheet.getRow(2);
	
    Cell cell = row.getCell(3);
    if (cell == null)｛
        cell = row.createCell(3);
	｝
    cell.setCellType(CellType.STRING);
    cell.setCellValue("a test");
	
    try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
        wb.write(fileOut);
    }
}
```

### 在单元格中使用换行符
```
Workbook wb = new XSSFWorkbook();

Sheet sheet = wb.createSheet();

Row row = sheet.createRow(2);

Cell cell = row.createCell(2);
cell.setCellValue("Use \n with word wrap on to create a new line");

// 要启用换行符，您需要使用wrap = true设置单元格样式
CellStyle cs = wb.createCellStyle();
cs.setWrapText(true);
cell.setCellStyle(cs);

// 增加行高以容纳两行文本
row.setHeightInPoints((2*sheet.getDefaultRowHeightInPoints()));
// 调整列宽以适合内容
sheet.autoSizeColumn(2);

try (OutputStream fileOut = new FileOutputStream("ooxml-newlines.xlsx")) {
    wb.write(fileOut);
}
wb.close();
```

### 格式化数据
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("format sheet");

CellStyle style;
DataFormat format = wb.createDataFormat();

Row row;
Cell cell;
int rowNum = 0;
int colNum = 0;

row = sheet.createRow(rowNum++);
cell = row.createCell(colNum);
cell.setCellValue(11111.25);
style = wb.createCellStyle();
style.setDataFormat(format.getFormat("0.0"));
cell.setCellStyle(style);

row = sheet.createRow(rowNum++);
cell = row.createCell(colNum);
cell.setCellValue(11111.25);
style = wb.createCellStyle();
style.setDataFormat(format.getFormat("#,##0.0000"));
cell.setCellStyle(style);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 设置工作表为一页
```
Workbook wb = new HSSFWorkbook();

Sheet sheet = wb.createSheet("format sheet");
PrintSetup ps = sheet.getPrintSetup();
sheet.setAutobreaks(true);
ps.setFitHeight((short)1);
ps.setFitWidth((short)1);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 设置打印区域
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("Sheet1");

// 设置第一张纸的打印区域
wb.setPrintArea(0, "$A$1:$C$2");
// 或者
wb.setPrintArea(
        0, //sheet index
        0, //start column
        1, //end column
        0, //start row
        0  //end row
);

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 设置页码
```
Workbook wb = new HSSFWorkbook(); // or new XSSFWorkbook();

Sheet sheet = wb.createSheet("format sheet");

Footer footer = sheet.getFooter();
footer.setRight( "Page " + HeaderFooter.page() + " of " + HeaderFooter.numPages() );

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
wb.close();
```

### 使用快捷方法
便利功能提供了实用功能，例如在合并区域周围设置边框以及更改样式属性而无需显式创建新样式。
```
Workbook wb = new HSSFWorkbook();  // or new XSSFWorkbook()
Sheet sheet1 = wb.createSheet( "new sheet" );

// 创建合并区域
Row row = sheet1.createRow( 1 );
Row row2 = sheet1.createRow( 2 );
Cell cell = row.createCell( 1 );
cell.setCellValue( "This is a test of merging" );
CellRangeAddress region = CellRangeAddress.valueOf("B2:E5");
sheet1.addMergedRegion( region );

// 设置边框和边框颜色
RegionUtil.setBorderBottom( BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderTop(    BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderLeft(   BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderRight(  BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBottomBorderColor(IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setTopBorderColor(   IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setLeftBorderColor(  IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setRightBorderColor( IndexedColors.AQUA.getIndex(), region, sheet1, wb);

// 显示HSSFCellUtil的一些用法
CellStyle style = wb.createCellStyle();
style.setIndention((short)4);
CellUtil.createCell(row, 8, "This is the value of the cell", style);
Cell cell2 = CellUtil.createCell( row2, 8, "This is the value of the cell");
CellUtil.setAlignment(cell2, HorizontalAlignment.CENTER);

try (OutputStream fileOut = new FileOutputStream( "workbook.xls" )) {
    wb.write( fileOut );
}
wb.close();
```

# PowerPoint (HSLF/XSLF)
# Word (HWPF/XWPF)
# Outlook (HSMF)
# Visio (HDGF+XDGF)
# Publisher (HPBF)
# OLE2 Filesystem (POIFS)
# OLE2 Document Props (HPSF)
# TNEF (HMEF) for winmail.dat
# OpenXML4J (OOXML)
# Logging framework




