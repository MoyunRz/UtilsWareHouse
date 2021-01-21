```java
import org.dom4j.Document;
import org.dom4j.DocumentHelper;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.XMLWriter;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileWriter;
import java.io.IOException;
import java.util.List;
import java.util.Map;


public class WriteXmlDemo {

    /**
     * 将实体类写入xml
     *
     * @param path    文件路径
     * @param xmlId   标签id
     * @param xmlName 标签名
     * @param emplist 数据
     * @throws IOException
     */
    public void setWriteXmlDemo(String path, String xmlName, String xmlChild, String xmlId, Map<String, Object> emplist) {
//        String utf ="UTF-8";
//        String vsion ="1.0";
//        String XML = "<?xml version="+vsion+" encoding="+utf+"?>10019807752138114538203547650510236870>";
        // 使用DocumentHelper创建一个Document对象

        Document doc = DocumentHelper.createDocument();
        // 添加并得到根元素
        Element root = doc.addElement(xmlName);
        // 为根元素添加子元素
        Element empEle = root.addElement(xmlChild);
        // 为子元素添加属性
        empEle.addAttribute("id", xmlId);
        // 遍历Map里的元素
        for (String key : emplist.keySet()) {
            empEle.addElement(key).addText(String.valueOf(emplist.get(key)));
        }
        XMLWriter writer = null;
        //格式良好地输出
        OutputFormat format = OutputFormat.createPrettyPrint();
        try {
            writer = new XMLWriter(new FileWriter(new File(path)), format);
            writer.write(doc);
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * @param path     文件存储路径
     * @param xmlName  标签名
     * @param xmlChild 子类标签
     * @param xmlId    listMap里的ID
     * @param listMap  需要存储的数据
     * @throws IOException
     */
    public void setWriteXmlDemo(String path, String xmlName, String xmlChild, String xmlId, List<Map<String, Object>> listMap) throws IOException {
        //        String utf ="UTF-8";
        //        String vsion ="1.0";
        //        String XML = "<?xml version="+vsion+" encoding="+utf+"?>10019807752138114538203547650510236870>";
        // 使用DocumentHelper创建一个Document对象

        Document doc = DocumentHelper.createDocument();
        // 添加并得到根元素
        Element root = doc.addElement(xmlName);

        for (Map<String, Object> emplist : listMap) {
            // 为根元素添加子元素
            Element empEle = root.addElement(xmlChild);
            // 为子元素添加属性
            empEle.addAttribute("id", String.valueOf(emplist.get(xmlId)));
            // 遍历Map里的元素
            for (String key : emplist.keySet()) {
                empEle.addElement(key).addText(String.valueOf(emplist.get(key)));
            }
        }

        XMLWriter writer = null;
        try {
            //格式良好地输出
            OutputFormat format = OutputFormat.createPrettyPrint();
            // false覆盖之前的文件
            writer = new XMLWriter(new FileWriter(new File(path), false), format);
            writer.write(doc);
            writer.close();
        } catch (FileNotFoundException e) {
//            writer.close();
            e.printStackTrace();
        }
    }

    /**
     * @param path   xml文件路径
     * @param elName xml里同名某标签
     * @return
     */
    public List<Element> getParseXmlDemo(String path, String elName) {

        try {

            SAXReader reader = new SAXReader();
            Document doc = reader.read(new FileInputStream(path));
            Element root = doc.getRootElement();
            // 获取根标签<list>下面的所有子标签<emp>
            List<Element> elements = root.elements(elName);
            return elements;

//            /**
//                 * 遍历所有<Emp>标签并解析出相关信息并以一个实例
//                 * 保存然后将其存入集合
//            **/
//            for(Element empEle: elements) {
//                String dbName =empEle.elementText("dbName");
//                Long offSet = Long.parseLong(empEle.attribute("offSet").getValue());
//                String F_MeterParamID = empEle.elementText("F_MeterParamID");
//                String F_BuildID = empEle.elementText("F_BuildID");
//                RecordData emp = new RecordData(dbName,offSet, F_MeterParamID, F_BuildID);
//                emplist.add(emp);
//            }
//
//            System.out.println("解析完毕");
//
//            for(RecordData e:emplist) {
//                System.out.println(e);
//            }
//            System.out.println(emplist);


        } catch (Exception e) {
//            e.printStackTrace();
            return null;
        }
    }


}


```
