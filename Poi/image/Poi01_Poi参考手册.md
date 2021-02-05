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

### 创建行与单元格
```
Workbook wb = new HSSFWorkbook();

CreationHelper createHelper = wb.getCreationHelper();

Sheet sheet = wb.createSheet("new sheet");

// 创建一个行并在其中放入一些单元格。行从0开始。
Row row = sheet.createRow(0);

// 创建一个单元格并在其中放置一个值
Cell cell = row.createCell(0);
cell.setCellValue(1);

// 或在同一行执行
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

### 使用便捷函数
便捷函数提供许多实用功能，例如设置合并区域的边框样式，在不显式创建新样式的情况下改变样式属性。
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

// 设置合并区域的边框和边框颜色
RegionUtil.setBorderTop(    BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderBottom( BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderLeft(   BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setBorderRight(  BorderStyle.MEDIUM_DASHED, region, sheet1, wb );
RegionUtil.setTopBorderColor(   IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setBottomBorderColor(IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setLeftBorderColor(  IndexedColors.AQUA.getIndex(), region, sheet1, wb);
RegionUtil.setRightBorderColor( IndexedColors.AQUA.getIndex(), region, sheet1, wb);

// HSSFCellUtil的一些使用示列
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

### 在工作表中上移或下移行
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("row sheet");
// 创建一些行和单元格备用

// 将表的第6-11行上移至第0-5行
sheet.shiftRows(5, 10, -5);
```

### 调整列宽以适合内容
```
Sheet sheet = workbook.getSheetAt(0);
// 调整第一列的宽度
sheet.autoSizeColumn(0);

// 调整第二列的宽度
sheet.autoSizeColumn(1);
```

### 设置工作表为选中状态
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("sheet");
sheet.setSelected(true);
```

### 设置缩放倍率。
缩放以分数表示。例如，要表示75%的缩放，分子使用3，分母使用4。
```
Workbook wb = new HSSFWorkbook();
Sheet sheet1 = wb.createSheet("new sheet");
sheet1.setZoom(75);   // 75 percent magnification
```

### 拆分和冻结窗格
可以创建两种类型的窗格：冻结窗格和拆分窗格。

**冻结窗格**按列和行拆分，可以使用以下机制创建冻结窗格：
```
Sheet1.createFreezePane(3，2，3，2)；
```
其中，前两个参数是要拆分的列和行，后两个参数表示右下角象限中可见的单元格。

**拆分窗格**以不同的方式显示。拆分区域被分成四个独立的工作区，拆分发生在像素级别，用户可以通过将拆分拖动到新位置来调整拆分。
使用以下调用创建拆分窗格：
```
Sheet2.createSplitPane(2000,2000，0，0，Sheet.PANE_LOWER_LEFT)；
```
第一个参数是拆分的x位置。这是一个点的1/20。在这种情况下，一个点似乎等同于一个像素。第二个参数是拆分的y位置。同样是在1/20的时候。
最后一个参数指示当前具有焦点的窗格，这是Sheet.PANE_LOWER_LEFT、PANE_LOWER_RIGHT、PANE_UPPER_RIGHT或PANE_UPPER_LEFT之一。


```
Workbook wb = new HSSFWorkbook();
Sheet sheet1 = wb.createSheet("new sheet");
Sheet sheet2 = wb.createSheet("second sheet");
Sheet sheet3 = wb.createSheet("third sheet");
Sheet sheet4 = wb.createSheet("fourth sheet");

// 只冻结一行
sheet1.createFreezePane( 0, 1, 0, 1 );

// 只冻结一列
sheet2.createFreezePane( 1, 0, 1, 0 );

// 冻结列和行
sheet3.createFreezePane( 2, 2 );

// 创建左下角为活动象限的拆分
sheet4.createSplitPane( 2000, 2000, 0, 0, Sheet.PANE_LOWER_LEFT );
try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 设置重复行和列
可以使用Sheet类中的setRepeatingRows()和setRepeatingColumns()方法在打印输出中设置重复的行和列。
这些方法需要一个CellRangeAddress参数，该参数指定要重复的行或列的范围。
对于setRepeatingRows()，它应该指定要重复的行的范围，其中列部分跨越所有列。
对于setRepeatingColum()，它应该指定要重复的列的范围，其中行部分跨越所有行。
如果该参数为NULL，则将删除重复的行或列。
```
Workbook wb = new HSSFWorkbook();
Sheet sheet1 = wb.createSheet("Sheet1");
Sheet sheet2 = wb.createSheet("Sheet2");

// 设置行从第4行重复到第5行
sheet1.setRepeatingRows(CellRangeAddress.valueOf("4:5"));

// 设置列从A列重复到C列
sheet2.setRepeatingColumns(CellRangeAddress.valueOf("A:C"));

try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 页眉和页脚
```
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet("new sheet");
Header header = sheet.getHeader();
header.setCenter("Center Header");
header.setLeft("Left Header");
header.setRight(HSSFHeader.font("Stencil-Normal", "Italic") +
                HSSFHeader.fontSize((short) 16) + "Right w/ Stencil-Normal Italic font and size 16");
try (OutputStream fileOut = new FileOutputStream("workbook.xls")) {
    wb.write(fileOut);
}
```

### 页眉和页脚（XSSF增强）
```
Workbook wb = new XSSFWorkbook();
XSSFSheet sheet = (XSSFSheet) wb.createSheet("new sheet");
// 创建第一个页眉
Header header = sheet.getFirstHeader();
header.setCenter("Center First Page Header");
header.setLeft("Left First Page Header");
header.setRight("Right First Page Header");

// 创建偶数页眉
Header header2 = sheet.getEvenHeader();
der2.setCenter("Center Even Page Header");
header2.setLeft("Left Even Page Header");
header2.setRight("Right Even Page Header");

// 创建奇数页眉
Header header3 = sheet.getOddHeader();
der3.setCenter("Center Odd Page Header");
header3.setLeft("Left Odd Page Header");
header3.setRight("Right Odd Page Header");

// 设置/删除标题属性
XSSFHeaderProperties prop = sheet.getHeaderFooterProperties();
prop.setAlignWithMargins();
prop.scaleWithDoc();
prop.removeDifferentFirstPage(); // 这不会删除首页页眉或页脚
prop.removeDifferentEvenOdd(); // 这不会删除偶数页眉或页脚
try (OutputStream fileOut = new FileOutputStream("workbook.xlsx")) {
    wb.write(fileOut);
}
```

### 图形 Shapes
POI支持使用Microsoft Office绘图工具绘制图形。
工作表上的图形按组和图形的层次结构进行组织。最上面的图形是```patriarch```，这在工作表上看不到。
要开始绘制，您需要在```HSSFSheet```类上调用```createPatriarch```。
这具有擦除存储在该薄板中的任何其他图形信息的效果。
默认情况下，除非调用此方法，否则POI会将图形记录单独留在工作表中。

#### 创建图形
要创建图形，您必须完成以下步骤：
+ 创造```patriarch```。
+ 创建锚点```anchor```来在图纸上定位图形。
+ 使用```patriarch```来创造这个图形。
+ 设置图形类型(直线、椭圆形、矩形等)。
+ 设置有关该图形的任何其他样式详细信息。(例如：线条粗细等)

```
HSSFPatriarch patriarch = sheet.createDrawingPatriarch();
a1 = new HSSFClientAnchor( 0, 0, 1023, 255, (short) 1, 0, (short) 1, 0 );
HSSFSimpleShape shape1 = patriarch.createSimpleShape(a1);
shape1.setShapeType(HSSFSimpleShape.OBJECT_TYPE_LINE);
```

文本框是使用不同的调用创建的：
```
HSSFTextbox textbox1 = patriarch.createTextbox(new HSSFClientAnchor(0,0,0,0,(short)1,1,(short)2,2));
textbox1.setString(new HSSFRichTextString("This is a test") );
```

可以使用不同的字体设置文本框中文本部分的样式：
```
HSSFFont font = wb.createFont();
font.setItalic(true);
font.setUnderline(HSSFFont.U_DOUBLE);
HSSFRichTextString string = new HSSFRichTextString("Woo!!!");
string.applyFont(2,5,font);
textbox.setString(string );
```

#### 创建图形组
可以将图形分组在一起。这是通过调用createGroup()然后使用这些组创建图形来实现的。也可以在组内创建组。

创建图形组：
```
// 创建一个图形组
HSSFShapeGroup group = patriarch.createGroup(new HSSFClientAnchor(0,0,900,200,(short)2,2,(short)2,2));

// 在组中创建几条线
HSSFSimpleShape shape1 = group.createShape(new HSSFChildAnchor(3,3,500,500));
shape1.setShapeType(HSSFSimpleShape.OBJECT_TYPE_LINE);
( (HSSFChildAnchor) shape1.getAnchor() ).setAnchor(3,3,500,500);
HSSFSimpleShape shape2 = group.createShape(new HSSFChildAnchor(1,200,400,600));
shape2.setShapeType(HSSFSimpleShape.OBJECT_TYPE_LINE);
```

所创建的组具有自己的坐标空间，用于放置在其中的图形。POI默认为(0，0,1023,255)，但您可以根据需要进行更改。以下是方法：
```
myGroup.setCoordinates(10,10,20,20); // top-left, bottom-right
```
如果你在一个组中创建一个组，它也会有自己的坐标空间。

#### 设置图形样式
默认情况下，图形可能看起来有点朴素。但是，可以对图形应用不同的样式。目前可以做的事情有：
+ 更改填充颜色。
+ 创建一个不带填充颜色的图形。
+ 更改线条的粗细。
+ 更改线条的样式。例：虚线，点线。
+ 更改线条颜色。

以下是一个示例：
```
HSSFSimpleShape s = patriarch.createSimpleShape(a);
s.setShapeType(HSSFSimpleShape.OBJECT_TYPE_OVAL);
s.setLineStyleColor(10,10,10);
s.setFillColor(90,10,200);
s.setLineWidth(HSSFShape.LINEWIDTH_ONE_PT * 3);
s.setLineStyle(HSSFShape.LINESTYLE_DOTSYS);
```

#### Graphics2d
虽然本地POI形状绘制命令是在形状中绘制形状的推荐方法，但有时也需要使用标准API来与外部库兼容。
考虑到这一点，我们为Graphics和Graphics2d创建了一些包装器。

所有图形命令都发布到```HSSFShapeGroup```中。它是这样做的：
```
a = new HSSFClientAnchor( 0, 0, 1023, 255, (short) 1, 0, (short) 1, 0 );
group = patriarch.createGroup( a );
group.setCoordinates( 0, 0, 80 * 4 , 12 * 23  );
float verticalPointsPerPixel = a.getAnchorHeightInPoints(sheet) / (float)Math.abs(group.getY2() - group.getY1());
g = new EscherGraphics( group, wb, Color.black, verticalPointsPerPixel );
g2d = new EscherGraphics2d( g );
drawChemicalStructure( g2d );
```

### 图像 Images
图像是```drawing```支持的一部分。添加图像，只需在```drawing patriarch```上调用createPicture()即可。
图片支持以下类型：
+ PNG
+ JPG
+ DIB

#### 创建图像
应该注意的是，将图像添加到工作表后，任何现有图形都可能被删除。
```
// 创建一个工作簿
Workbook wb = new XSSFWorkbook(); //or new HSSFWorkbook();

// 将图片数据添加到此工作簿
InputStream is = new FileInputStream("image1.jpeg");

byte[] bytes = IOUtils.toByteArray(is);
int pictureIdx = wb.addPicture(bytes, Workbook.PICTURE_TYPE_JPEG);
is.close();
CreationHelper helper = wb.getCreationHelper();

// 创建工作表
Sheet sheet = wb.createSheet();

// 创建drawing patriarch。这是所有图形的顶级容器。
Drawing drawing = sheet.createDrawingPatriarch();

// 添加图片图形
ClientAnchor anchor = helper.createClientAnchor();

// 设置图片的左上角
anchor.setCol1(3);
anchor.setRow1(2);
Picture pict = drawing.createPicture(anchor, pictureIdx);

// 自动调整图片相对于其左上角的大小
pict.resize();

// 保存工作薄
String file = "picture.xls";
if(wb instanceof XSSFWorkbook) file += "x";
try (OutputStream fileOut = new FileOutputStream(file)) {
    wb.write(fileOut);
}
```

#### 从工作簿读取图像
```
ist lst = workbook.getAllPictures();
or (Iterator it = lst.iterator(); it.hasNext(); ) {
   PictureData pict = (PictureData)it.next();
   String ext = pict.suggestFileExtension();
   byte[] data = pict.getData();
   if (ext.equals("jpeg")){
     try (OutputStream out = new FileOutputStream("pict.jpg")) {
       out.write(data);
     }
   }
```

### 命名区域和命名单元格
命名区域是按名称引用一组单元格的一种方式。命名单元格是命名区域的退化情况，因为“单元格组”只包含一个单元格。
您可以按命名区域创建和引用工作簿中的单元格。

当使用命名区域时，使用org.apache.poi.ss.util.CellReference和org.apache.poi.ss.util.AreaReference类。

注意：使用‘A1：B1’这样的相对值可能会导致在Microsoft Excel中使用工作簿时名称指向的单元格意外移动，通常使用‘$A$1：$B$1’这样的绝对引用可以避免这种情况。

#### 创建命名区域/命名单元格
```
String sname = "TestSheet", cname = "TestName", cvalue = "TestVal";
Workbook wb = new HSSFWorkbook();
Sheet sheet = wb.createSheet(sname);
sheet.createRow(0).createCell(0).setCellValue(cvalue);

// 使用areareference为单个单元格创建命名区域
Name namedCell = wb.createName();
namedCell.setNameName(cname + "1");
String reference = sname+"!$A$1:$A$1"; // area reference
namedCell.setRefersToFormula(reference);

// 使用cellreference为单个单元格创建命名区域
Name namedCel2 = wb.createName();
namedCel2.setNameName(cname + "2");
reference = sname+"!$A$1"; // cell reference
namedCel2.setRefersToFormula(reference);

// 使用AreaReference为区域创建命名范围
Name namedCel3 = wb.createName();
namedCel3.setNameName(cname + "3");
reference = sname+"!$A$1:$C$5"; // area reference
namedCel3.setRefersToFormula(reference);

// 创建命名格式
Name namedCel4 = wb.createName();
namedCel4.setNameName("my_sum");
namedCel4.setRefersToFormula("SUM(" + sname + "!$I$2:$I$6)");
```

#### 从命名区域/命名单元格读取
```
String cname = "TestName";
Workbook wb = getMyWorkbook();

// 检索命名范围
int namedCellIdx = wb.getNameIndex(cellName);
Name aNamedCell = wb.getNameAt(namedCellIdx);

// 检索命名范围内的单元格并测试其内容
AreaReference aref = new AreaReference(aNamedCell.getRefersToFormula());
CellReference[] crefs = aref.getAllReferencedCells();
for (int i=0; i<crefs.length; i++) {
    Sheet s = wb.getSheet(crefs[i].getSheetName());
    Row r = sheet.getRow(crefs[i].getRow());
    Cell c = r.getCell(crefs[i].getCol());
    // extract the cell contents based on cell type etc.
}
```

#### 从非连续命名范围读取
```
// Setup code
String cname = "TestName";
Workbook wb = getMyWorkbook();

// 检索命名范围
// 将类似于“$C$10，$D$12：$D$14
int namedCellIdx = wb.getNameIndex(cellName);
Name aNamedCell = wb.getNameAt(namedCellIdx);

// 检索命名范围内的单元格并测试其内容。
// 将获得C10的一个AreaReference和D12至D14的一个AreaReference
AreaReference[] arefs = AreaReference.generateContiguous(aNamedCell.getRefersToFormula());
for (int i=0; i<arefs.length; i++) {
    // Only get the corners of the Area
    // (use arefs[i].getAllReferencedCells() to get all cells)
    CellReference[] crefs = arefs[i].getCells();
    for (int j=0; j<crefs.length; j++) {
        // Check it turns into real stuff
        Sheet s = wb.getSheet(crefs[j].getSheetName());
        Row r = s.getRow(crefs[j].getRow());
        Cell c = r.getCell(crefs[j].getCol());
        // Do something with this corner cell
    }
}
```

注意，删除单元格时，Excel不会删除附加的命名区域。因此，工作簿可以包含指向不再存在的单元格的命名区域。
在构造AreaReference之前，应该检查引用的有效性：
```
if(name.isDeleted()){
  // 命名区域指向已删除的单元格
} else {
  AreaReference ref = new AreaReference(name.getRefersToFormula());
}
```

### 单元格注释
注释是附加到单元格或与单元格关联的富文本注释，独立于其他单元格内容。
注释内容与单元格分开存储，并显示在与单元格分开但与单元格关联的绘图对象中。

#### 创建单元格注释
```
Workbook wb = new XSSFWorkbook();
CreationHelper factory = wb.getCreationHelper();
Sheet sheet = wb.createSheet();
Row row   = sheet.createRow(3);
Cell cell = row.createCell(5);
cell.setCellValue("F4");
Drawing drawing = sheet.createDrawingPatriarch();

// 当注释框可见时，将其显示在1x3的空间中
ClientAnchor anchor = factory.createClientAnchor();
anchor.setCol1(cell.getColumnIndex());
anchor.setCol2(cell.getColumnIndex()+1);
anchor.setRow1(row.getRowNum());
anchor.setRow2(row.getRowNum()+3);

// 创建评论并设置文本+作者
Comment comment = drawing.createCellComment(anchor);
RichTextString str = factory.createRichTextString("Hello, World!");
comment.setString(str);
comment.setAuthor("Apache POI");

// 将注释分配给单元格
cell.setCellComment(comment);
String fname = "comment-xssf.xls";
if(wb instanceof XSSFWorkbook) fname += "x";
try (OutputStream out = new FileOutputStream(fname)) {
    wb.write(out);
}
wb.close();
```

#### 读取单元格注释
```
Cell cell = sheet.get(3).getColumn(1);
Comment comment = cell.getCellComment();
if (comment != null) {
  RichTextString str = comment.getString();
  String author = comment.getAuthor();
}
// 或者，您可以按(行、列)检索单元格注释
comment = sheet.getCellComment(3, 1);
```

要获取所有注释，请执行以下操作：
```
Map<CellAddress, Comment> comments = sheet.getCellComments();
Comment commentA1 = comments.get(new CellAddress(0, 0));
Comment commentB1 = comments.get(new CellAddress(0, 1));
for (Entry<CellAddress, ? extends Comment> e : comments.entrySet()) {
  CellAddress loc = e.getKey();
  Comment comment = e.getValue();
  System.out.println("Comment at " + loc + ": " +
      "[" + comment.getAuthor() + "] " + comment.getString().getString());
```

### 超链接
#### 创建超链接
```
Workbook wb = new XSSFWorkbook(); //or new HSSFWorkbook();
CreationHelper createHelper = wb.getCreationHelper();

// 超链接的单元格样式。默认情况下，超链接为蓝色并带下划线。
CellStyle hlink_style = wb.createCellStyle();
Font hlink_font = wb.createFont();
hlink_font.setUnderline(Font.U_SINGLE);
hlink_font.setColor(IndexedColors.BLUE.getIndex());
hlink_style.setFont(hlink_font);
Cell cell;
Sheet sheet = wb.createSheet("Hyperlinks");

// URL
cell = sheet.createRow(0).createCell(0);
cell.setCellValue("URL Link");
Hyperlink link = createHelper.createHyperlink(HyperlinkType.URL);
link.setAddress("https://poi.apache.org/");
cell.setHyperlink(link);
cell.setCellStyle(hlink_style);

// 链接到当前目录中的文件
cell = sheet.createRow(1).createCell(0);
cell.setCellValue("File Link");
link = createHelper.createHyperlink(HyperlinkType.FILE);
link.setAddress("link1.xls");
cell.setHyperlink(link);
cell.setCellStyle(hlink_style);

// 电子邮件链接
cell = sheet.createRow(2).createCell(0);
cell.setCellValue("Email Link");
link = createHelper.createHyperlink(HyperlinkType.EMAIL);
//note, if subject contains white spaces, make sure they are url-encoded
link.setAddress("mailto:poi@apache.org?subject=Hyperlinks");
cell.setHyperlink(link);
cell.setCellStyle(hlink_style);

// 链接到此工作簿中的某个位置
//create a target sheet and cell
Sheet sheet2 = wb.createSheet("Target Sheet");
sheet2.createRow(0).createCell(0).setCellValue("Target Cell");
cell = sheet.createRow(3).createCell(0);
cell.setCellValue("Worksheet Link");
Hyperlink link2 = createHelper.createHyperlink(HyperlinkType.DOCUMENT);
link2.setAddress("'Target Sheet'!A1");
cell.setHyperlink(link2);
cell.setCellStyle(hlink_style);

try (OutputStream out = new FileOutputStream("hyperinks.xlsx")) {
    wb.write(out);
}
wb.close();
```

#### 读取超链接
```
Sheet sheet = workbook.getSheetAt(0);
Cell cell = sheet.getRow(0).getCell(0);
Hyperlink link = cell.getHyperlink();
if(link != null){
    System.out.println(link.getAddress());
}
```

### 数据验证

#### 预定义值
下面的代码将把用户可以输入到单元格A1中的值限制为三个整数值之一，即10、20或30。
```
HSSFWorkbook workbook = new HSSFWorkbook();
HSSFSheet sheet = workbook.createSheet("Data Validation");

CellRangeAddressList addressList = new CellRangeAddressList(0, 0, 0, 0);
DVConstraint dvConstraint = DVConstraint.createExplicitListConstraint(new String[]{"10", "20", "30"});
DataValidation dataValidation = new HSSFDataValidation(addressList, dvConstraint);
dataValidation.setSuppressDropDownArrow(true);
sheet.addValidationData(dataValidation);
```

#### 预定义值-下拉列表
此代码将执行相同的操作，但为用户提供了一个可从中选择值的下拉列表。
```
HSSFWorkbook workbook = new HSSFWorkbook();
HSSFSheet sheet = workbook.createSheet("Data Validation");

CellRangeAddressList addressList = new CellRangeAddressList(0, 0, 0, 0);
DVConstraint dvConstraint = DVConstraint.createExplicitListConstraint(new String[]{"10", "20", "30"});
DataValidation dataValidation = new HSSFDataValidation(addressList, dvConstraint);
dataValidation.setSuppressDropDownArrow(false);
sheet.addValidationData(dataValidation);
```

#### 错误信息
创建一个消息框，如果用户输入的值无效，该消息框将显示给用户。
```
dataValidation.setErrorStyle(DataValidation.ErrorStyle.STOP);
dataValidation.createErrorBox("Box Title", "Message Text");
```

#### 提示信息
创建当包含数据验证的单元格获得焦点时用户将看到的提示
```
dataValidation.createPromptBox("Title", "Message Text");
dataValidation.setShowPromptBox(true);
```

#### 进阶数据验证
##### 10到100之间的整数的验证
```
dvConstraint = DVConstraint.createNumericConstraint(
  DVConstraint.ValidationType.INTEGER,
  DVConstraint.OperatorType.BETWEEN, "10", "100");
```

#### 为单元格创建数据验证
特定单元格的内容可用于提供数据验证的值，DVConstraint.createFormulaListConstraint(String)方法支持这一点。

要指定值来自连续的单元格范围，请执行以下任一操作：
```
dvConstraint = DVConstraint.createFormulaListConstraint("$A$1:$A$3");
```

### 嵌入对象
可以对嵌入的Excel、Word或PowerPoint文档执行更详细的处理，或处理任何其他类型的嵌入对象。

HSSF:
```
POIFSFileSystem fs = new POIFSFileSystem(new File("excel_with_embeded.xls"));
HSSFWorkbook workbook = new HSSFWorkbook(fs);
for (HSSFObjectData obj : workbook.getAllEmbeddedObjects()) {
    //the OLE2 Class Name of the object
    String oleName = obj.getOLE2ClassName();
    if (oleName.equals("Worksheet")) {
        DirectoryNode dn = (DirectoryNode) obj.getDirectory();
        HSSFWorkbook embeddedWorkbook = new HSSFWorkbook(dn, false);
        //System.out.println(entry.getName() + ": " + embeddedWorkbook.getNumberOfSheets());
    } else if (oleName.equals("Document")) {
        DirectoryNode dn = (DirectoryNode) obj.getDirectory();
        HWPFDocument embeddedWordDocument = new HWPFDocument(dn);
        //System.out.println(entry.getName() + ": " + embeddedWordDocument.getRange().text());
    }  else if (oleName.equals("Presentation")) {
        DirectoryNode dn = (DirectoryNode) obj.getDirectory();
        SlideShow<?,?> embeddedPowerPointDocument = new HSLFSlideShow(dn);
        //System.out.println(entry.getName() + ": " + embeddedPowerPointDocument.getSlides().length);
    } else {
        if(obj.hasDirectoryEntry()){
            // The DirectoryEntry is a DocumentNode. Examine its entries to find out what it is
            DirectoryNode dn = (DirectoryNode) obj.getDirectory();
            for (Entry entry : dn) {
                //System.out.println(oleName + "." + entry.getName());
            }
        } else {
            // There is no DirectoryEntry
            // Recover the object's data from the HSSFObjectData instance.
            byte[] objectData = obj.getObjectData();
        }
    }
}
```

XSSF:
```
XSSFWorkbook workbook = new XSSFWorkbook("excel_with_embeded.xlsx");
for (PackagePart pPart : workbook.getAllEmbeddedParts()) {
    String contentType = pPart.getContentType();
    // Excel Workbook - either binary or OpenXML
    if (contentType.equals("application/vnd.ms-excel")) {
        HSSFWorkbook embeddedWorkbook = new HSSFWorkbook(pPart.getInputStream());
    }
    // Excel Workbook - OpenXML file format
    else if (contentType.equals("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet")) {
        OPCPackage docPackage = OPCPackage.open(pPart.getInputStream());
        XSSFWorkbook embeddedWorkbook = new XSSFWorkbook(docPackage);
    }
    // Word Document - binary (OLE2CDF) file format
    else if (contentType.equals("application/msword")) {
        HWPFDocument document = new HWPFDocument(pPart.getInputStream());
    }
    // Word Document - OpenXML file format
    else if (contentType.equals("application/vnd.openxmlformats-officedocument.wordprocessingml.document")) {
        OPCPackage docPackage = OPCPackage.open(pPart.getInputStream());
        XWPFDocument document = new XWPFDocument(docPackage);
    }
    // PowerPoint Document - binary file format
    else if (contentType.equals("application/vnd.ms-powerpoint")) {
        HSLFSlideShow slideShow = new HSLFSlideShow(pPart.getInputStream());
    }
    // PowerPoint Document - OpenXML file format
    else if (contentType.equals("application/vnd.openxmlformats-officedocument.presentationml.presentation")) {
        OPCPackage docPackage = OPCPackage.open(pPart.getInputStream());
        XSLFSlideShow slideShow = new XSLFSlideShow(docPackage);
    }
    // Any other type of embedded object.
    else {
        System.out.println("Unknown Embedded Document: " + contentType);
        InputStream inputStream = pPart.getInputStream();
    }
}
```

### 自动筛选
```
Workbook WB=new HSSFWorkbook()；//或new XSSFWorkbook()；
Sheet Sheet=wb.createSheet()；
Sheet.setAutoFilter(CellRangeAddress.valueOf(“C5:F200”))；
```

### 条件格式设置
```
Workbook workbook = new HSSFWorkbook();
Sheet sheet = workbook.createSheet();
SheetConditionalFormatting sheetCF = sheet.getSheetConditionalFormatting();
ConditionalFormattingRule rule1 = sheetCF.createConditionalFormattingRule(ComparisonOperator.EQUAL, "0");
FontFormatting fontFmt = rule1.createFontFormatting();
fontFmt.setFontStyle(true, false);
fontFmt.setFontColorIndex(IndexedColors.DARK_RED.index);
BorderFormatting bordFmt = rule1.createBorderFormatting();
bordFmt.setBorderBottom(BorderStyle.THIN);
bordFmt.setBorderTop(BorderStyle.THICK);
bordFmt.setBorderLeft(BorderStyle.DASHED);
bordFmt.setBorderRight(BorderStyle.DOTTED);
PatternFormatting patternFmt = rule1.createPatternFormatting();
patternFmt.setFillBackgroundColor(IndexedColors.YELLOW.index);
ConditionalFormattingRule rule2 = sheetCF.createConditionalFormattingRule(ComparisonOperator.BETWEEN, "-10", "10");

ConditionalFormattingRule [] cfRules =
{
    rule1, rule2
};

CellRangeAddress[] regions = {
    CellRangeAddress.valueOf("A3:A5")
};

sheetCF.addConditionalFormatting(regions, cfRules);
```

### 隐藏和取消隐藏行
使用Excel，可以隐藏工作表上的行，方法是选择该行(或多行)，右键单击鼠标右键一次，然后从出现的弹出菜单中选择“隐藏”。

要使用POI进行模拟，只需在XSSFRow或HSSFRow的实例上调用setZeroHeight()方法(该方法在两个类都实现的ss.usermodel.Row接口上定义)，如下所示：
```
Workbook workbook = new XSSFWorkbook();  // OR new HSSFWorkbook()
Sheet sheet = workbook.createSheet(0);
Row row = workbook.createRow(0);
row.setZeroHeight();
```
如果现在将文件保存到光盘，则第一页上的第一行将不可见。

使用Excel，可以取消隐藏以前隐藏的行，方法是选择隐藏的行上方和下方的行，然后按住Ctrl键、Shift键和数字9，然后将它们全部释放。
要使用POI模拟此行为，请执行以下操作：
```
Workbook workbook = WorkbookFactory.create(new File(.......));
Sheet = workbook.getSheetAt(0);
Iterator<Row> row Iter = sheet.iterator();
while(rowIter.hasNext()) {
  Row row = rowIter.next();
  if(row.getZeroHeight()) {
    row.setZeroHeight(false);
  }
}
```
如果现在将文件保存到光盘，则工作簿第一页上以前隐藏的任何行现在都将可见。

该示例说明了两个功能：
首先，只需调用setZeroHeight()方法并传递布尔值‘false’，就可以取消隐藏一行。
其次，它说明了如何测试行是否隐藏。只需调用getZeroHeight()方法，如果该行被隐藏，它将返回‘true’，否则返回‘false’。

### 设置单元格属性
有时，使用基本样式创建电子表格，然后将特殊样式应用于某些单元格(如在单元格区域周围绘制边框或设置区域填充)会更容易或更高效。
SetCellProperties使您无需在电子表格中创建一堆不必要的中间样式就可以做到这一点。

属性被创建为Map，并以以下方式应用于单元格。
```
Workbook workbook = new XSSFWorkbook();
Sheet sheet = workbook.createSheet("Sheet1");
Map<String, Object> properties = new HashMap<String, Object>();

// 设置单元格边框
properties.put(CellUtil.BORDER_TOP, BorderStyle.MEDIUM);
properties.put(CellUtil.BORDER_BOTTOM, BorderStyle.MEDIUM);
properties.put(CellUtil.BORDER_LEFT, BorderStyle.MEDIUM);
properties.put(CellUtil.BORDER_RIGHT, BorderStyle.MEDIUM);

// 给它加一种颜色(红色)
properties.put(CellUtil.TOP_BORDER_COLOR, IndexedColors.RED.getIndex());
properties.put(CellUtil.BOTTOM_BORDER_COLOR, IndexedColors.RED.getIndex());
properties.put(CellUtil.LEFT_BORDER_COLOR, IndexedColors.RED.getIndex());
properties.put(CellUtil.RIGHT_BORDER_COLOR, IndexedColors.RED.getIndex());

// 将边框应用于B2单元格
Row row = sheet.createRow(1);
Cell cell = row.createCell(1);
CellUtil.setCellStyleProperties(cell, properties);

// 将边框应用于从D4开始的3x3区域
for (int ix=3; ix <= 5; ix++) {
  row = sheet.createRow(ix);
  for (int iy = 3; iy <= 5; iy++) {
    cell = row.createCell(iy);
    CellUtil.setCellStyleProperties(cell, properties);
  }
}
```

注意：
这不会替换单元格的属性，它会将您放入Map中的属性与单元格的现有样式属性合并。
如果属性已存在，则将其替换为新属性。如果属性不存在，则会添加该属性。
此方法不会删除CellStyle属性。

### 绘制边框
在Excel中，只需按一下按钮，就可以在整个工作簿区域上应用一组边框。
PropertyTemplate对象使用定义为允许在单元格范围周围绘制上、下、左、右、水平、垂直、内、外或所有边框的方法和常量来模拟此情况。
其他方法允许将颜色应用于边框。

它的工作原理是这样的：
创建一个PropertyTemplate对象，该对象是要应用于工作表的边框的容器。
然后向PropertyTemplate添加边框和颜色，最后将其应用于需要该边框集的任何工作表。
可以创建多个PropertyTemplate对象并将其应用于单个工作表，也可以将同一PropertyTemplate对象应用于多个工作表。
它就像一张预印的表格。

```
// draw borders (three 3x3 grids)
PropertyTemplate pt = new PropertyTemplate();
// #1) these borders will all be medium in default color
pt.drawBorders(new CellRangeAddress(1, 3, 1, 3),
        BorderStyle.MEDIUM, BorderExtent.ALL);
// #2) these cells will have medium outside borders and thin inside borders
pt.drawBorders(new CellRangeAddress(5, 7, 1, 3),
        BorderStyle.MEDIUM, BorderExtent.OUTSIDE);
pt.drawBorders(new CellRangeAddress(5, 7, 1, 3), BorderStyle.THIN,
        BorderExtent.INSIDE);
// #3) these cells will all be medium weight with different colors for the
//     outside, inside horizontal, and inside vertical borders. The center
//     cell will have no borders.
pt.drawBorders(new CellRangeAddress(9, 11, 1, 3),
        BorderStyle.MEDIUM, IndexedColors.RED.getIndex(),
        BorderExtent.OUTSIDE);
pt.drawBorders(new CellRangeAddress(9, 11, 1, 3),
        BorderStyle.MEDIUM, IndexedColors.BLUE.getIndex(),
        BorderExtent.INSIDE_VERTICAL);
pt.drawBorders(new CellRangeAddress(9, 11, 1, 3),
        BorderStyle.MEDIUM, IndexedColors.GREEN.getIndex(),
        BorderExtent.INSIDE_HORIZONTAL);
pt.drawBorders(new CellRangeAddress(10, 10, 2, 2),
        BorderStyle.NONE,
        BorderExtent.ALL);
// apply borders to sheet
Workbook wb = new XSSFWorkbook();
Sheet sh = wb.createSheet("Sheet1");
pt.applyBorders(sh);
```

注意：
最后一个pt.drawBorders()调用使用BorderStyle.NONE从区域中删除边框。
与setCellStyleProperties一样，applyBorders方法合并单元格样式的属性，因此现有边框只有在被其他内容替换时才会更改，或者只有在被BorderStyle.NONE替换时才会被删除。
若要从边框中删除颜色，请使用IndexedColor.AUTOMATIC.getIndex()。

此外，若要从PropertyTemplate对象中删除边框或颜色，请使用BorderExtent.NONE。

这还不适用于对角线边界。

### 创建透视表
透视表是电子表格文件的强大功能。您可以使用以下代码创建数据透视表。
```
XSSFWorkbook wb = new XSSFWorkbook();
XSSFSheet sheet = wb.createSheet();

// 创建一些用于构建数据透视表的数据
setCellData(sheet);
XSSFPivotTable pivotTable = sheet.createPivotTable(new AreaReference("A1:D4"), new CellReference("H5"));

// 配置透视表。
// 使用第一列作为行标签
pivotTable.addRowLabel(0);

// 总结第二列
pivotTable.addColumnLabel(DataConsolidateFunction.SUM, 1);

// 将第三列设置为筛选器
pivotTable.addColumnLabel(DataConsolidateFunction.AVERAGE, 2);

// 在第四列添加筛选器
pivotTable.addReportFilter(3);
```

#### 具有多种样式的单元格(富文本字符串)
要将一组文本格式(颜色、样式、字体等)应用于单元格，应为工作簿创建单元格样式，然后将其应用于单元格。
HSSF Example：
```
HSSFCell hssfCell = row.createCell(idx);
//rich text consists of two runs
HSSFRichTextString richString = new HSSFRichTextString( "Hello, World!" );
richString.applyFont( 0, 6, font1 );
richString.applyFont( 6, 13, font2 );
hssfCell.setCellValue( richString );
```

XSSF Example：
```
XSSFCell cell = row.createCell(1);
XSSFRichTextString rt = new XSSFRichTextString("The quick brown fox");
XSSFFont font1 = wb.createFont();
font1.setBold(true);
font1.setColor(new XSSFColor(new java.awt.Color(255, 0, 0)));
rt.applyFont(0, 10, font1);
XSSFFont font2 = wb.createFont();
font2.setItalic(true);
font2.setUnderline(XSSFFont.U_DOUBLE);
font2.setColor(new XSSFColor(new java.awt.Color(0, 255, 0)));
rt.applyFont(10, 19, font2);
XSSFFont font3 = wb.createFont();
font3.setColor(new XSSFColor(new java.awt.Color(0, 0, 255)));
rt.append(" Jumped over the lazy dog", font3);
cell.setCellValue(rt);
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




