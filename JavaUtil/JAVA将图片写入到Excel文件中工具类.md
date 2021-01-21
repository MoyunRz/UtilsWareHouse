**依赖包**
```xml
  <dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
  </dependency>
```

```java
import org.apache.poi.xssf.usermodel.XSSFClientAnchor;
import org.apache.poi.xssf.usermodel.XSSFDrawing;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.*;
import java.util.HashMap;
import java.util.Map;

public class WriteImgUtil {

    /**
     * 写入图片到Excel指定的位置
     *
     * @param patriarch 画图的顶级管理器，一个sheet只能获取一次，多次插入图片请使用同一个patriarch对象
     * @param wb        HSSFWorkbook对象
     * @param filepath      图片文件路径
     * @param cellPoint 自定义的对象，指定要插入图片的坐标(x, y)
     * @return cellPoint 自定义的对象，返回下一个要插入图片的坐标(x, y)
     * @throws IOException
     */
    public static void whiteImg(XSSFDrawing patriarch, XSSFWorkbook wb, String filepath, CellPoint cellPoint) throws IOException {

        ByteArrayOutputStream byteArrayOut = new ByteArrayOutputStream();
        BufferedImage bufferImg = ImageIO.read(new File(filepath));
        ImageIO.write(bufferImg, filepath.substring(filepath.lastIndexOf(".") + 1), byteArrayOut);

        // 起点坐标
        int x1 = cellPoint.getRow1();
        int y1 = cellPoint.getCol1();
        // 终点坐标
        int x2 = cellPoint.getRow2();
        int y2 = cellPoint.getCol2();
        // anchor主要用于设置图片的属性
        XSSFClientAnchor anchor1 = new XSSFClientAnchor(0, 0, 0, 0, (short) x1, y1, (short) x2, y2);
        // 插入图片
        int index = wb.addPicture(byteArrayOut.toByteArray(), XSSFWorkbook.PICTURE_TYPE_PICT);
        patriarch.createPicture(anchor1, index);
    }


    public static class CellPoint {
        // 起点横坐标
        private int row1;
        // 起点纵坐标
        private int col1;
        // 终点横坐标
        private int row2;
        // 终点纵坐标
        private int col2;

        public CellPoint(int row1, int col1, int row2, int col2) {
            this.row1 = row1;
            this.col1 = col1;
            this.row2 = row2;
            this.col2 = col2;
        }

        public int getRow1() {
            return row1;
        }

        public void setRow1(int row1) {
            this.row1 = row1;
        }

        public int getCol1() {
            return col1;
        }

        public void setCol1(int col1) {
            this.col1 = col1;
        }

        public int getRow2() {
            return row2;
        }

        public void setRow2(int row2) {
            this.row2 = row2;
        }

        public int getCol2() {
            return col2;
        }

        public void setCol2(int col2) {
            this.col2 = col2;
        }
    }

    /**
     * * 插入图片到指定excel
     *
     * @param file       待插入excel文件
     * @param sheetIndex 待插入excel的Sheet序号(从0开始)
     * @param map        key: 待插入图片路径  value: 图片在excel中的坐标
     * @return
     * @Date: 2020/11/28 10:09
     */
    public static void addImageToExcel(File file, Integer sheetIndex, Map<String, CellPoint> map) {
        FileOutputStream fileOut = null;
        try {
            FileInputStream inputstream = new FileInputStream(file);
            XSSFWorkbook wb = new XSSFWorkbook(inputstream);
            XSSFSheet sheet1 = wb.getSheetAt(sheetIndex);
            // 画图的顶级管理器，一个sheet只能获取一个（一定要注意这点）
            XSSFDrawing patriarch = sheet1.createDrawingPatriarch();
            for (Map.Entry<String, CellPoint> entry : map.entrySet()) {
                String key = entry.getKey(); // 图片路径
                CellPoint point = entry.getValue(); // 图片插入坐标
                whiteImg(patriarch, wb, key, point);
            }
            fileOut = new FileOutputStream(file);
            // 写入excel文件
            wb.write(fileOut);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (fileOut != null) {
                try {
                    fileOut.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws IOException {
        Map<String, CellPoint> map = new HashMap<>();
        map.put("D:\\1.png",new CellPoint(5,13,7,14));
        addImageToExcel(new File("D:\\2.xlsx"),0,map);
        System.out.println("ok");
    }

}

```
