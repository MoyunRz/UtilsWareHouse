```java

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;
 
import org.apache.poi.hssf.usermodel.HSSFDateUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.CollectionUtils;
 
/**
 * @Description: Excel读写工具类
 */
public class ExcelUtils {
 
    private static final Logger logger = LoggerFactory.getLogger(ExcelUtils.class);
 
    /**
     * 读取Excel内容
     * @param file  需要被读的文件对象
     * @param startRow  从哪一行开始读 (rowIndex从0开始的)
     * @param isExcel2003   是否是excel2003还是更高的版本
     * @param sheetIndex    读取哪一个sheet (sheetIndex也是从0开始)
     * @return  List<List<String>>
     * @throws Exception
     */
    public static List<List<String>> readExcel(File file, int startRow, boolean isExcel2003, int sheetIndex) throws Exception {
        List<List<String>> dataLst;
        InputStream is = null;
        try {
            /** 创建读取文件的输入流  */
            is = new FileInputStream(file);
            /** 根据版本选择创建Workbook的方式 */
            Workbook wb;
            if (isExcel2003) {
                wb = new HSSFWorkbook(is);
            } else {
                wb = new XSSFWorkbook(is);
            }
            /** 调用本类的读取方法读取excel数据  */
            dataLst = read(wb, startRow, sheetIndex);
        } catch (Exception ex) {
        	logger.error("读取excel文件异常!", ex);
            ex.printStackTrace();
            throw  ex;
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        /** 返回最后读取的结果 */
        return dataLst;
    }
 
    private static List<List<String>> read(Workbook wb, int startRow, int sheetIndex) {
        /** 总列数 */
        int totalCells = 0;
    	/** 创建集合存储读取的数据  */
        List<List<String>> dataLst = new ArrayList<List<String>>();
        /** 得到第一个shell */
        Sheet sheet = wb.getSheetAt(sheetIndex);
        /** 得到Excel的行数  */
        int totalRows = sheet.getPhysicalNumberOfRows();
        /** 得到Excel的列数  */
        if (totalRows >= 1 && sheet.getRow(0) != null) {
            totalCells = sheet.getRow(0).getPhysicalNumberOfCells();
        }
        /** 循环Excel的行  */
        for (int r = startRow; ; r++) {
            Row row = sheet.getRow(r);
            if (row == null) {
                break;
            }
            List<String> rowLst = new ArrayList<String>();
            /** 循环Excel的列 */
            for (int c = 0; c < totalCells; c++) {
                Cell cell = row.getCell(c);
                String cellValue = "";
                if (null != cell) {
                    // 以下是判断数据的类型
                    switch (cell.getCellTypeEnum()) {
                    case NUMERIC: // 数字
                    	// 判断是不是日期格式
                    	if (HSSFDateUtil.isCellDateFormatted(cell)) {
                    		cellValue = cell.getDateCellValue() + "";
						}else {
							cellValue = cell.getNumericCellValue() + "";
						}
                        break;
                    case STRING: // 字符串
                        cellValue = cell.getStringCellValue();
                        break;
                    case BOOLEAN: // Boolean
                        cellValue = cell.getBooleanCellValue() + "";
                        break;
                    case FORMULA: // 公式
                        cellValue = cell.getCellFormula() + "";
                        break;
                    case BLANK: // 空值
                        cellValue = "";
                        break;
                    case ERROR: // 故障
                        cellValue = "非法字符";
                        break;
                    default:
                        cellValue = "未知类型";
                        break;
                    }
                }
                rowLst.add(cellValue);
            }
            /** 保存第r行的第c列 */
            boolean isEmptyRow = true;
        	
            if (rowLst != null) {	
            	for (String s : rowLst) {
            		if (s != null && !s.isEmpty()) {
            			isEmptyRow = false;
            		}
            	}
            }
            if (!isEmptyRow) {
            	dataLst.add(rowLst);
            }
        }
        return dataLst;
    }
 
    /**
     * 读取Excel内容
     * @param filePath 被读取文件的绝对路径
     * @param startRow
     * @param isExcel2003
     * @param sheetIndex
     * @return  List<List<String>>
     * @throws Exception
     */
    public static List<List<String>> readExcel(String filePath, int startRow, boolean isExcel2003, int sheetIndex) throws Exception {
    	
    	return readExcel(new File(filePath) , startRow, isExcel2003, sheetIndex);
    }
 
    /**
     * 将数据写入Excel工作簿
     * @param header    表格的标题
     * @param dataList  所需写入的数据 List<List<String>>
     * @param isExcel2003   是否是excel2003还是更高的版本
     * @param sheetName     生成的excel中sheet的名字
     * @return  Workbook    之后直接写出即可，如workbook.write(new FileOutputStream("E://test/20190410_test.xlsx"));
     */
    public static Workbook getWorkbookFromList(List<String> header, List<List<String>> dataList, boolean isExcel2003,
                                             String sheetName) {
        Workbook wb;
        // 创建Workbook对象(excel的文档对象)
        if (isExcel2003) {
            wb = new HSSFWorkbook();
        } else {
            wb = new XSSFWorkbook();
        }
        // 建立新的sheet对象（excel的表单）
        Sheet sheet = wb.createSheet(sheetName);
        // 在sheet里创建第一行，参数为行索引(excel的行)，可以是0～65535之间的任何一个
        int rowNum = 0;
        Row row0 = sheet.createRow(rowNum);
        if (!CollectionUtils.isEmpty(header)) {
            // 设置表头
            for (int i = 0; i < header.size(); i++) {
                Cell cell = row0.createCell(i);
                // 设置单元格样式
                cell.setCellStyle(POIUtils.getCellStyle(wb, "Calibri", (short) 12));
                // 设置列宽
                sheet.setColumnWidth(i, 256 * 20);
                cell.setCellValue(header.get(i));
            }
            rowNum++;
        }
        if (!CollectionUtils.isEmpty(dataList)) {
            // 填充数据
            for (List<String> cellList : dataList) {
                Row row = sheet.createRow(rowNum);
                for (int i = 0; i < cellList.size(); i++) {
                    Cell cell = row.createCell(i);
                    cell.setCellStyle(POIUtils.getCellStyle(wb, "Calibri", (short) 12));
                    if (CollectionUtils.isEmpty(header)) {
                        sheet.setColumnWidth(i, 256 * 20);
                    }
                    cell.setCellValue(cellList.get(i));
                }
                rowNum++;
            }
        }
        return wb;
    }
 
    /**
     * 将数据写入Excel工作簿
     * @param header    表格的标题
     * @param dataList  所需写入的数据 List<Object>
     * @param isExcel2003   是否是excel2003还是更高的版本
     * @param sheetName     生成的excel中sheet的名字
     * @return  Workbook对象，之后直接写出即可，如workbook.write(new FileOutputStream("E://test/20190410_test.xlsx"));
     * @throws Exception
     */
    public static Workbook getWorkbookFromObj(List<String> header, List<?> dataList, boolean isExcel2003,
                                            String sheetName) throws Exception {
        Workbook wb;
        // 创建Workbook对象(excel的文档对象)
        if (isExcel2003) {
            wb = new HSSFWorkbook();
        } else {
            wb = new XSSFWorkbook();
        }
        // 建立新的sheet对象（excel的表单）
        Sheet sheet = wb.createSheet(sheetName);
        // 在sheet里创建第一行，参数为行索引(excel的行)，可以是0～65535之间的任何一个
        int rowNum = 0;
        Row row0 = sheet.createRow(rowNum);
        if (!CollectionUtils.isEmpty(header)) {
            // 设置表头
            for (int i = 0; i < header.size(); i++) {
                Cell cell = row0.createCell(i);
                // 设置单元格样式
                cell.setCellStyle(POIUtils.getCellStyle(wb, "Calibri", (short) 12));
                sheet.setColumnWidth(i, 256 * 20);
                cell.setCellValue(header.get(i));
            }
            rowNum++;
        }
        if (!CollectionUtils.isEmpty(dataList)) {
            // 填充数据
            Class<? extends Object> objClass = dataList.get(0).getClass();
            Field[] fields = objClass.getDeclaredFields();
            for (int i = 0; i < dataList.size(); i++) {
                // 创建row对象
                Row row = sheet.createRow(rowNum);
                // 遍历获取每一个字段的值
                for (int j = 0; j < fields.length; j++) {
                    String fieldVal = "";
                    Method[] methods = objClass.getDeclaredMethods();
                    for (Method method : methods) {
                        if (method.getName().equalsIgnoreCase("get" + fields[j].getName())) {
                            String property = (String) method.invoke(dataList.get(i), null);
                            fieldVal = property == null ? "" : property;
                            break;
                        }
                    }
                    Cell cell = row.createCell(j);
                    cell.setCellStyle(POIUtils.getCellStyle(wb, "Calibri", (short) 12));
                    if (CollectionUtils.isEmpty(header)) {
                        sheet.setColumnWidth(j, 256 * 20);
                    }
                    cell.setCellValue(fieldVal);
                }
                rowNum++;
            }
        }
        return wb;
    }
 
    public static boolean validateExcel(String filePath) {
        /** 检查文件名是否为空或者是否是Excel格式的文件 */
        if (filePath == null
                || !(isExcel2003(filePath) || isExcel2007(filePath))) {
            // "文件名不是excel格式";
            return false;
        }
        /** 检查文件是否存在 */
        File file = new File(filePath);
        if (file == null || !file.exists()) {
            // "文件不存在";
            return false;
        }
        return true;
    }
    
    public static boolean isExcel2003(String filePath) {
        return filePath.matches("^.+\\.(?i)(xls)$");
    }
    
    public static boolean isExcel2007(String filePath) {
        return filePath.matches("^.+\\.(?i)(xlsx)$");
    }
}

```

```java

 
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.ss.usermodel.Font;
import org.apache.poi.xssf.usermodel.XSSFCellStyle;
import org.apache.poi.xssf.usermodel.XSSFColor;
import org.apache.poi.xssf.usermodel.XSSFFont;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
 
import java.awt.*;
import java.awt.Color;
 
/**
 * Excel的单元格样式
 */
public class POIUtils {
	
	/**
	 * 设置单元格的边框（细）且为黑色，字体水平垂直居中，自动换行
	 * @param workbook
	 * @param fontName
	 * @param fontSize
	 * @return
	 */
    public static CellStyle getCellStyle(Workbook workbook, String fontName, short fontSize){

        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
          // 设置上下左右四个边框宽度
          style.setBorderTop(BorderStyle.THIN);
          style.setBorderBottom(BorderStyle.THIN);
          style.setBorderLeft(BorderStyle.THIN);
          style.setBorderRight(BorderStyle.THIN);
          // 设置上下左右四个边框颜色
          style.setTopBorderColor(IndexedColors.BLACK.getIndex());
          style.setBottomBorderColor(IndexedColors.BLACK.getIndex());
          style.setLeftBorderColor(IndexedColors.BLACK.getIndex());
          style.setRightBorderColor(IndexedColors.BLACK.getIndex());
          // 水平居中，垂直居中，自动换行
          style.setAlignment(HorizontalAlignment.CENTER);
          style.setVerticalAlignment(VerticalAlignment.CENTER);
          style.setWrapText(false);
          // 设置字体样式及大小
          font.setFontName(fontName);
          font.setFontHeightInPoints(fontSize);
          
          style.setFont(font);
          
          return style;
    }
}

```
