```java
import org.jdom2.Document;
import org.jdom2.Element;
import org.jdom2.input.SAXBuilder;
import org.jdom2.output.Format;
import org.jdom2.output.XMLOutputter;
import org.w3c.dom.Node;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.StringWriter;
import java.lang.reflect.Field;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.UUID;

/**
 * 应用模块名称:
 * 代码描述:
 * Copyright: Copyright (C) 2020, Inc. All rights reserved.
 * Company:
 *
 * @author
 * @since 2020/12/10 17:19
 */
public class XmlInterfaceUtils {

    private XmlInterfaceUtils() {

    }

    /**
     * 对象 转 xml
     *
     * @param object
     * @return 返回一个生成xml的位置
     */
    public synchronized static String convertToXml(Object object) {
        if (object == null) {
            return null;
        }
        Element root = null;
        String pth ="";
        try {
            Class<?> aClass = object.getClass();
            // 根节点
            root = new Element("root");
            for (Field declaredField : aClass.getDeclaredFields()) {
                declaredField.setAccessible(true);
                // 这个属性为 List 时
                if (declaredField.getGenericType() instanceof ParameterizedType) {
                    Element child1 = new Element(declaredField.getName());
                    List<?> objList = (List<?>) declaredField.get(object);
                    if (objList != null) {
                        ListObjecToXml(objList, child1);
                        root.addContent(child1);
                    }
                } else {
                    String fieldName = declaredField.getName();
                    Element child = new Element(fieldName);
                    child.setText(String.valueOf(declaredField.get(object)));
                    root.addContent(child);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("对象转换成XML出现异常：" + e.getMessage());
        }
        try {
            // 将 根节点 加入 文档中
            Document doc = new Document(root);
            // 创建xml输出流操作类
            XMLOutputter xmlOutput = new XMLOutputter();
            // 格式化xml内容
            xmlOutput.setFormat(Format.getPrettyFormat());
            String fpath = System.getProperty("user.dir") + "\\parser\\src\\main\\resources\\history";
//            File directory = new File(fpath);
//            String courseFile = fpath + "\\history";
            // 把xml输出到指定位置
            File f = new File(fpath);
            if (!f.exists()) {
                f.mkdirs();
            }
            SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHssmm");
            File file = new File(fpath + "\\" + UUID.randomUUID().toString().replaceAll("-", "") + ".xml");
            pth = file.getAbsolutePath();
            xmlOutput.output(doc, new FileOutputStream(file));
            Object obj = dataXmltoEntityMsg(object.getClass(),file.getAbsolutePath());
            if(obj==null){
                convertToXmlMsg(object,pth);
            }else{
                return file.getAbsolutePath();
            }
        } catch (Exception e) {
            e.printStackTrace();
            convertToXmlMsg(object,pth);
            System.out.println("生成xml文件出现异常：" + e.getMessage());
            return null;
        }
        return null;
    }

    /**
     * 对象 转 xml
     *
     * @param object
     * @return 返回一个生成xml的位置
     */
    public synchronized static String convertToXmlMsg(Object object,String paths) {
        DeleteFileUtil.deleteFile(paths);
        if (object == null) {
            return null;
        }
        Element root = null;
        try {
            Class<?> aClass = object.getClass();
            // 根节点
            root = new Element("root");
            for (Field declaredField : aClass.getDeclaredFields()) {
                declaredField.setAccessible(true);
                // 这个属性为 List 时
                if (declaredField.getGenericType() instanceof ParameterizedType) {
                    Element child1 = new Element(declaredField.getName());
                    List<?> objList = (List<?>) declaredField.get(object);
                    if (objList != null) {
                        ListObjecToXml(objList, child1);
                        root.addContent(child1);
                    }
                } else {
                    String fieldName = declaredField.getName();
                    Element child = new Element(fieldName);
                    child.setText(String.valueOf(declaredField.get(object)));
                    root.addContent(child);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("对象转换成XML出现异常：" + e.getMessage());
        }
        try {
            // 将 根节点 加入 文档中
            Document doc = new Document(root);
            // 创建xml输出流操作类
            XMLOutputter xmlOutput = new XMLOutputter();
            // 格式化xml内容
            xmlOutput.setFormat(Format.getPrettyFormat());
            String fpath = System.getProperty("user.dir") + "\\parser\\src\\main\\resources\\history";
//            File directory = new File(fpath);
//            String courseFile = fpath + "\\history";
            // 把xml输出到指定位置
            File f = new File(fpath);
            if (!f.exists()) {
                f.mkdirs();
            }
            SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHssmm");
            File file = new File(fpath + "\\Data" + format.format(new Date()) + ".xml");
            xmlOutput.output(doc, new FileOutputStream(file));
            return file.getAbsolutePath();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("生成xml文件出现异常：" + e.getMessage());
        }
        return null;
    }


    private synchronized static void ListObjecToXml(List<?> objData, Element element) throws Exception {
        if (objData != null) {
            for (int i = 0; i < objData.size(); i++) {
                Field[] declaredFields = objData.get(i).getClass().getDeclaredFields();
                Object obj = objData.get(i);
                Element root = new Element(obj.getClass().getSimpleName());
                for (Field declaredField : declaredFields) {
                    declaredField.setAccessible(true);
                    Element child = new Element(declaredField.getName());
                    if (declaredField.getGenericType() instanceof ParameterizedType) {
                        List<?> objList = (List<?>) declaredField.get(obj);
                        if (objList != null) {
                            ListObjecToXml(objList, child);
                            root.addContent(child);
                        }
                    } else {
                        child.setText(String.valueOf(declaredField.get(obj)));
                        root.addContent(child);
                    }

                }
                element.addContent(root);
            }
        }
    }

    /**
     * xml 转 对象
     *
     * @param clazz
     * @param xmlStr
     * @return
     * @throws Exception
     */
    public synchronized static Object dataXmltoEntityMsg(Class<?> clazz, String xmlStr) {
        if (clazz == null) {
            System.out.println("未设置对象的类型");
            return null;
        }
        File file = new File(xmlStr);
        if (!file.exists()) {
            System.out.println("解析失败，找不到文件");
            return null;
        }
        //创建Jdom2的解析器对象
        SAXBuilder builder = new SAXBuilder();
        String nodeName = "root";
        Document document = null;
        Element root = null;
        Object obj = null;
        try {
            document = builder.build(file.getAbsoluteFile());
            root = document.getRootElement();
            if (!nodeName.equals(root.getName())) {
                System.out.println("xml内容无法转成 " + clazz + "对象，请检查！");
                return null;
            }
            // new出 当前最大的对象
            obj = clazz.getConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("无法进行xml解析，请检查！");
            return null;
        }
        try {
            List<Element> children = root.getChildren();
            for (Element child : children) {
                // 第二层的xml数据
                List<Element> children1 = child.getChildren();
                if (children1.isEmpty()) { // 处理第一层的xml数据
                    Field field = clazz.getDeclaredField(child.getName());
                    field.setAccessible(true);
                    if ("int".equals(field.getGenericType().getTypeName())) {
                        field.set(obj, Integer.parseInt(child.getValue()));
                    } else {
                        field.set(obj, child.getValue());
                    }
                } else { // 处理第二层的 xml 数据
                    mm(clazz, obj, child, children1);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("解析xml数据转换成实体出现异常：" + e.getMessage());
            return null;
        }
        return obj;
    }

    /**
     * xml 转 对象
     *
     * @param clazz
     * @param xmlStr
     * @return
     * @throws Exception
     */
    public synchronized static Object dataXmltoEntity(Class<?> clazz, String xmlStr) {
        if (clazz == null) {
            System.out.println("未设置对象的类型");
            return null;
        }
        File file = new File(xmlStr);
        if (!file.exists()) {
            System.out.println("解析失败，找不到文件");
            return null;
        }
        //创建Jdom2的解析器对象
        SAXBuilder builder = new SAXBuilder();
        String nodeName = "root";
        Document document = null;
        Element root = null;
        Object obj = null;
        try {
            document = builder.build(file.getAbsoluteFile());
            root = document.getRootElement();
            if (!nodeName.equals(root.getName())) {
                System.out.println("xml内容无法转成 " + clazz + "对象，请检查！");
                return null;
            }
            // new出 当前最大的对象
            obj = clazz.getConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("无法进行xml解析，请检查！");
        }
        try {
            List<Element> children = root.getChildren();
            for (Element child : children) {
                // 第二层的xml数据
                List<Element> children1 = child.getChildren();
                if (children1.isEmpty()) { // 处理第一层的xml数据
                    Field field = clazz.getDeclaredField(child.getName());
                    field.setAccessible(true);
                    if ("int".equals(field.getGenericType().getTypeName())) {
                        field.set(obj, Integer.parseInt(child.getValue()));
                    } else {
                        field.set(obj, child.getValue());
                    }
                } else { // 处理第二层的 xml 数据
                    mm(clazz, obj, child, children1);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("解析xml数据转换成实体出现异常：" + e.getMessage());
        }
        return obj;
    }

    private synchronized static void mm(Class<?> clazz, Object obj, Element child, List<Element> children1) throws Exception {
        // 取到当前的 list 属性
        Field field = clazz.getDeclaredField(child.getName());
        field.setAccessible(true);
        Class<?> genericClazz = null;
        if (field.getType() == List.class) {
            // 如果是List类型，得到其Generic的类型
            Type genericType = field.getGenericType();
            if (genericType != null) {
                // 如果是泛型参数的类型
                if (genericType instanceof ParameterizedType) {
                    ParameterizedType pt = (ParameterizedType) genericType;
                    //得到泛型里的class类型对象
                    genericClazz = (Class<?>) pt.getActualTypeArguments()[0];
                }
            }
        }
        if (genericClazz != null) {
            List list = new ArrayList();
            // list 中 包含的对象
            for (Element element : children1) {
                // 取出当前类的属性与值
                List<Element> children2 = element.getChildren();
                // new 出List中包含的对象
                Object o = genericClazz.getConstructor().newInstance();
                // 当前对象进行赋值
                for (Element element1 : children2) {
                    if (element1.getChildren().isEmpty()) {
                        Field field1 = genericClazz.getDeclaredField(element1.getName());
                        field1.setAccessible(true);
                        field1.set(o, element1.getValue());
                    } else {
                        // 递归处理
                        mm(genericClazz, o, element1, element1.getChildren());
                    }
                }
                list.add(o);
            }
            field.set(obj, list);
        }
    }

    public synchronized static String turnDocumentToString(String paths) {
        try {
            // 读取 xml 文件
            File fileinput = new File(paths);
            DocumentBuilderFactory dbFactory = DocumentBuilderFactory
                    .newInstance();
            DocumentBuilder dBuilder = dbFactory.newDocumentBuilder();
            org.w3c.dom.Document doc = dBuilder.parse(fileinput);

            // 方法1：将xml文件转化为String
             StringWriter sw = new StringWriter();
             TransformerFactory tf = TransformerFactory.newInstance();
             Transformer transformer = tf.newTransformer();
             transformer.setOutputProperty(OutputKeys.OMIT_XML_DECLARATION,
             "no");
             transformer.setOutputProperty(OutputKeys.METHOD, "xml");
             transformer.setOutputProperty(OutputKeys.INDENT, "yes");
             transformer.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
             transformer.transform(new DOMSource(doc), new StreamResult(sw));

            //方法2：和方法1类似
//            DOMSource domSource = new DOMSource((Node) doc);
//            StringWriter writer = new StringWriter();
//            StreamResult result = new StreamResult(writer);
//            TransformerFactory tf = TransformerFactory.newInstance();
//            Transformer transformer = tf.newTransformer();
//            transformer.transform(domSource, result);

            // 将转换过的xml的String 样式打印到控制台
//            System.out.println(writer.toString());
            return sw.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    public synchronized static boolean createFile(String destFileName) {

        // 加上这个指定模块
        File file = new File(destFileName);
        //要创建的单个文件已存在
        if (file.exists()) {
            return true;
        }
        //如果输入的文件是以分隔符结尾的，则说明File对象是目录而不是文件
        if (destFileName.endsWith(File.separator)) {
            return false;
        }
        //判断目标文件所在目录是否存在
        if (!file.getParentFile().exists()) {
            //判断父文件夹是否存在，如果存在则表示创建成功，否则失败
            if (!file.getParentFile().mkdirs()) {
                return false;
            }
        }
        //创建目标文件
        try {
            if (file.createNewFile()) {
                return true;
            } else {
                return false;
            }
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }

    }
}


```
