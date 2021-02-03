# xml解析

## xml解析

[Edit](http://maxiang.io/#/?provider=evernote&amp;guid=4b5f464b-a23d-4d25-8d2d-8afc325270ed&amp;notebook=%E6%8A%80%E6%9C%AF%E7%9F%A5%E8%AF%86)

## xml解析

#### sax的解析

**观察者模式** _SAX的全称是Simple APIs for XML，即XML简单应用程序接口。_

**优/缺点**

1. 解析简单，对内存要求低
2. 分析器只做简单的分析工作，提供顺序访问机制，已访问过的不可回退查看

[参考文档：csdnXML解析之 SAX解析](http://www.cnblogs.com/mengdd/archive/2013/06/02/3114177.html)

```text
<bookstore><book category="children">      <title lang="en">Harry Potter</title>       <author>J K. Rowling</author>       <year>2005</year>       <price>29.99</price> </book>
```

**分析器的构造**

```text
// step 1: 获得SAX解析器工厂实例    SAXParserFactory factory = SAXParserFactory.newInstance();    // step 2: 获得SAX解析器实例    SAXParser parser = factory.newSAXParser();    // step 3: 开始进行解析    // 传入待解析的文档的处理器，转为javabean    parser.parse(new File("books.xml"), javabean);
```

#### StAX解析

**pull模式** _Stream API for XML_

[Java学习：使用StAX解析XML](https://my.oschina.net/Tsybius2014/blog/539543) [使用 StAX 部分解析 XML 文档](https://www.ibm.com/developerworks/cn/xml/x-tipstx2/) [使用Stax解析XML](http://blog.csdn.net/jadyer/article/details/8691820)

demo

```text
InputStream in = new FileInputStream(filePath);XMLInputFactory xif = XMLInputFactory.newInstance();//创建StAX分析工厂XMLStreamReader reader = xif.createXMLStreamReader(in);//创建分析器while(reader.hasNext())//迭代   {       int event = reader.next();//读取下一个事件       if(event == XMLStreamReader.START_ELEMENT)//如果这个事件是元素开始       {          if("student_id".equals(reader.getLocalName()))//判断元素是不是student_id          {//如果是student_id则输出元素的文本内容              System.out.print(reader.getLocalName()+" : ");              System.out.println(reader.getElementText());          }                          }   }}
```

}

出现的问题：

```text
1.使用reader.getAttributeCount()没有获取到任何信息2.导致reader.getAttributeName(index)  reader.getAttributeValue(index)没有获取到任何信息。
```

### 最终解析流程

![](https://github.com/yangbao93/docs/tree/d23f2b2cbc4eb06e62d38114d6a7f5410080c7b5/技术知识/Java/xml解析/会计档案公共类XML解析.png) 1.获取xml和xmlInputFactory，XmlStreamReader

```text
// 1.构建XMLStreamReader  // File xmlFile = new File(getXmlPath().toString());  File xmlFile = new File("filePath");  XMLInputFactory factory = XMLInputFactory.newInstance();  XMLStreamReader reader = factory.createXMLStreamReader(new FileInputStream(xmlFile));  while (reader.hasNext()) {    int type = reader.next();    if (type == XMLStreamReader.START_ELEMENT) {      String localName = reader.getLocalName();      //选择属于那张表      choseConcreteParser(reader, localName);    }  }
```

2.选择相应的操作 choseConcreteParser

```text
//table1 table2 均继承table表switch(localName){    case table1:    parseAndInsertDB4PublicXml(reader,new Table1());    break;    case table2:    parseAndInsertDB4PublicXml(reader,new Table2());    break;}
```

3.解析相关的数据parseAndInsertDB4PublicXml

```text
//方法名：parseAndInsertDB4PublicXml(reader , table)List<Map<String, String>> fieldList = null;MetaObject applyMeta = null;if (table instanceof table1) {  Table1 table1 = (Table1 ) table;  applyMeta = SystemMetaObject.forObject(table1);  fieldList = PropertyUtil.initAnnoPropertyDic(Table1.class);  elementName = Table1.name;} else if (table instanceof table2) {  Table2 table2 = (Table2) table;  applyMeta = SystemMetaObject.forObject(table2 );  fieldList = PropertyUtil.initAnnoPropertyDic(Table2.class);  elementName = Table2.name;} createConcreteObject(fieldList, reader, applyMeta, elementName);insertData2DB(auditAdminData);return reader;
```

4.PropertyUtil类

```text
public class PropertyUtil {  /**   * 根据实体类名获取字段名称和中文名称   *    * @param entityName 实体类名   * @return List<Map<String,String>> 为了后面采用反射机制，这里返回的得map的key是注解(本来只是注释，但是   *         java反射没办法获取注释，所以把注释放到注解里面，所以这里要求，注解需要和 审计署总账xml里面的element的name一样， 不能有丝毫差异，否则会出问题)   *         map的value是属性property 通过上面的描述，也可以发现要求审计署每一类xml里面里面不能有一样的子element， 否则key就一样了   *    */  public static List<Map<String, String>> initAnnoPropertyDic(Class clzz) {    // 用于存储字段和中文值的集合    List<Map<String, String>> fieldList = new ArrayList<Map<String, String>>();    // 用于存储实体类字段(key)和中文名(value)    Map<String, String> valueMap = new Hashtable<String, String>();    // 获取对象中所有的Field    Field[] fields = clzz.getDeclaredFields();    // 循环实体类字段集合,获取标注@PropertyAnnotation的字段    for (Field field : fields) {      if (field.isAnnotationPresent(PropertyAnnotation.class)) {        // 获取字段名        String fieldNames = field.getName();        // 获取字段注解        PropertyAnnotation columnConfig = field.getAnnotation(PropertyAnnotation.class);        // 判断是否已经获取过该code的字典数据 避免重复获取        if (valueMap.get(columnConfig.description()) == null) {          valueMap.put(columnConfig.description(), fieldNames);        } else {          System.out.println("出错了");        }      }    }    fieldList.add(valueMap);// 将LinkedHashMap放入List集合中    return fieldList;  }}
```

5.createConcreteObject方法

```text
private XMLStreamReader createConcreteObject(List<Map<String, String>> fieldList,  XMLStreamReader reader, MetaObject applyMeta, String elementName) throws Exception {Map<String, String> map = fieldList.get(0);while (reader.hasNext()) {  int type = reader.next();  if (type == XMLStreamReader.END_ELEMENT && reader.getLocalName().equals(elementName)) {    break;  }  if (type == XMLStreamReader.START_ELEMENT) {    String localName = reader.getLocalName();    String elementText = reader.getElementText();    for (String applyBodySetName : applyMeta.getSetterNames()) {      if (applyBodySetName.toLowerCase().equals(map.get(localName))) {        Class<?> classType = applyMeta.getGetterType(applyBodySetName);        String typeName = classType.getName();        if (typeName.equals(BigDecimal.class.getName())) {          if (!StringUtils.isEmpty(elementText)) {            applyMeta.setValue(applyBodySetName, new BigDecimal(elementText));          }        } else if (typeName.equals(Integer.class.getName())            && !StringUtils.isEmpty(elementText)) {          if (!StringUtils.isEmpty(elementText)) {            applyMeta.setValue(applyBodySetName, Integer.valueOf(elementText));          }        } else if (typeName.equals("int")) {          if (!StringUtils.isEmpty(elementText)) {            applyMeta.setValue(applyBodySetName, Integer.valueOf(elementText));          }        } else if (typeName.equals("java.util.Date")) {          DateFormat fmt = new SimpleDateFormat("yyyyMMdd");          Date date = fmt.parse(elementText);          applyMeta.setValue(applyBodySetName, date);        } else if (typeName.equals("boolean")) {          if (elementText.equals("0")) {            applyMeta.setValue(applyBodySetName, false);          } else {            applyMeta.setValue(applyBodySetName, true);          }        } else {          applyMeta.setValue(applyBodySetName, elementText);        }        break;      }    }  }}return reader;}
```

6.持久化

```text
private void insertData2DB(AuditAdminData auditAdminData) {if (auditAdminData == null) {  return;}if (table instanceof table1) {  table1Repo.add((Table1) table);} else if (table2 instanceof table2) {  table2Repo.add((Table) table);}}
```

**多线程时的线程安全问题**

springMVC默认controller，dao，service都是单例的。 每个请求过来都会调用原有instance去处理：多线程出现时，里面的数据就不会是线程安全的了。

多数情况下不需要考虑多线程安全，但是如果声明了实例变量，则会出现问题：

```text
public class Controller extends AbstractCommandController {    protected Company company;  protected ModelAndView handle(HttpServletRequest request,HttpServletResponse response,Object command,BindException errors) throws Exception {      company = ................;            }    }
```

解决方式： – controller中使用Threadlocal变量 – 在controller中声明scope=prototype（每次都会创建新的controller）

```text
Controller  RequestMapping("/fui")  public class FuiController extends SpringController {  //这么定义的话就是单例  Controller  @Scope("prototype")  @RequestMapping("/fui")  public class FuiController extends SpringController {  //每次都创建
```

_**如果一个类如果没有成员变量，那这个类肯定是thread-safe的，所以UserDaoHibernate的thread-safe取决于其父类。而 UserManagerImpl 的安全性又取决于UserDaoHibernate，最后是HibernateTemplate。可以看出，这里的bean都小心翼翼的维护其成员变量， 或者基本没有成员变量，而将thread-safe转嫁给Spring的API。如果开发者按照约定的或者用自动产生的工具（appgen不错）来编写数 据访问层，是没有线程安全性的问题的。Spring本身不提供这方面的保证。**_

%23xml%u89E3%u6790%0A%0A%23%23%23sax%u7684%u89E3%u6790%0A**%u89C2%u5BDF%u8005%u6A21%u5F0F**%0A_SAX%u7684%u5168%u79F0%u662FSimple%20APIs%20for%20XML%uFF0C%u5373XML%u7B80%u5355%u5E94%u7528%u7A0B%u5E8F%u63A5%u53E3%u3002_%0A%0A%0A%23%23%23%23%u4F18/%u7F3A%u70B9%0A1.%20%u89E3%u6790%u7B80%u5355%uFF0C%u5BF9%u5185%u5B58%u8981%u6C42%u4F4E%0A2.%20%u5206%u6790%u5668%u53EA%u505A%u7B80%u5355%u7684%u5206%u6790%u5DE5%u4F5C%uFF0C%u63D0%u4F9B%u987A%u5E8F%u8BBF%u95EE%u673A%u5236%uFF0C%u5DF2%u8BBF%u95EE%u8FC7%u7684%u4E0D%u53EF%u56DE%u9000%u67E5%u770B%0A%0A%5B%u53C2%u8003%u6587%u6863%uFF1AcsdnXML%u89E3%u6790%u4E4B%20SAX%u89E3%u6790%5D%28http%3A//www.cnblogs.com/mengdd/archive/2013/06/02/3114177.html%29%0A%0A%20%20%20%20%3Cbookstore%3E%0A%20%20%20%20%3Cbook%20category%3D%22children%22%3E%0A%20%20%20%20%20%20%20%20%20%20%3Ctitle%20lang%3D%22en%22%3EHarry%20Potter%3C/title%3E%20%0A%20%20%20%20%20%20%20%20%20%20%3Cauthor%3EJ%20K.%20Rowling%3C/author%3E%20%0A%20%20%20%20%20%20%20%20%20%20%3Cyear%3E2005%3C/year%3E%20%0A%20%20%20%20%20%20%20%20%20%20%3Cprice%3E29.99%3C/price%3E%20%0A%20%20%20%20%3C/book%3E%0A%3C/bookstore%3E%0A%0A%23%23%23%23%u5206%u6790%u5668%u7684%u6784%u9020%0A%0A%09%20%20%20%20//%20step%201%3A%20%u83B7%u5F97SAX%u89E3%u6790%u5668%u5DE5%u5382%u5B9E%u4F8B%0A%20%20%20%20%20%20%20%20SAXParserFactory%20factory%20%3D%20SAXParserFactory.newInstance%28%29%3B%0A%0A%20%20%20%20%20%20%20%20//%20step%202%3A%20%u83B7%u5F97SAX%u89E3%u6790%u5668%u5B9E%u4F8B%0A%20%20%20%20%20%20%20%20SAXParser%20parser%20%3D%20factory.newSAXParser%28%29%3B%0A%0A%20%20%20%20%20%20%20%20//%20step%203%3A%20%u5F00%u59CB%u8FDB%u884C%u89E3%u6790%0A%20%20%20%20%20%20%20%20//%20%u4F20%u5165%u5F85%u89E3%u6790%u7684%u6587%u6863%u7684%u5904%u7406%u5668%uFF0C%u8F6C%u4E3Ajavabean%0A%20%20%20%20%20%20%20%20parser.parse%28new%20File%28%22books.xml%22%29%2C%20javabean%29%3B%0A%0A%23%23%23StAX%u89E3%u6790%0A**pull%u6A21%u5F0F**%0A_Stream%20API%20for%20XML_%0A%0A%5BJava%u5B66%u4E60%uFF1A%u4F7F%u7528StAX%u89E3%u6790XML%5D%28https%3A//my.oschina.net/Tsybius2014/blog/539543%29%0A%5B%u4F7F%u7528%20StAX%20%u90E8%u5206%u89E3%u6790%20XML%20%u6587%u6863%5D%28https%3A//www.ibm.com/developerworks/cn/xml/x-tipstx2/%29%0A%5B%u4F7F%u7528Stax%u89E3%u6790XML%5D%28http%3A//blog.csdn.net/jadyer/article/details/8691820%29%0A%0Ademo%0A%0A%0A%0A%20%20%20%20InputStream%20in%20%3D%20new%20FileInputStream%28filePath%29%3B%0A%20%20%20%20XMLInputFactory%20xif%20%3D%20XMLInputFactory.newInstance%28%29%3B//%u521B%u5EFAStAX%u5206%u6790%u5DE5%u5382%0A%20%20%20%20XMLStreamReader%20reader%20%3D%20xif.createXMLStreamReader%28in%29%3B//%u521B%u5EFA%u5206%u6790%u5668%0A%20%20%20%20while%28reader.hasNext%28%29%29//%u8FED%u4EE3%0A%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20int%20event%20%3D%20reader.next%28%29%3B//%u8BFB%u53D6%u4E0B%u4E00%u4E2A%u4E8B%u4EF6%0A%20%20%20%20%20%20%20%20%20%20%20if%28event%20%3D%3D%20XMLStreamReader.START\_ELEMENT%29//%u5982%u679C%u8FD9%u4E2A%u4E8B%u4EF6%u662F%u5143%u7D20%u5F00%u59CB%0A%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%28%22student\_id%22.equals%28reader.getLocalName%28%29%29%29//%u5224%u65AD%u5143%u7D20%u662F%u4E0D%u662Fstudent\_id%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B//%u5982%u679C%u662Fstudent\_id%u5219%u8F93%u51FA%u5143%u7D20%u7684%u6587%u672C%u5185%u5BB9%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20System.out.print%28reader.getLocalName%28%29+%22%20%3A%20%22%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20System.out.println%28reader.getElementText%28%29%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%0A%20%20%20%20%7D%0A%7D%0A%0A%u51FA%u73B0%u7684%u95EE%u9898%uFF1A%0A%0A%20%20%20%201.%u4F7F%u7528reader.getAttributeCount%28%29%u6CA1%u6709%u83B7%u53D6%u5230%u4EFB%u4F55%u4FE1%u606F%0A%20%20%20%202.%u5BFC%u81F4reader.getAttributeName%28index%29%0A%09%20%20reader.getAttributeValue%28index%29%u6CA1%u6709%u83B7%u53D6%u5230%u4EFB%u4F55%u4FE1%u606F%u3002%0A%0A%0A%0A%23%23%u6700%u7EC8%u89E3%u6790%u6D41%u7A0B%0A%21%5B%u89E3%u6790xml%u8111%u56FE%5D%28./%u4F1A%u8BA1%u6863%u6848%u516C%u5171%u7C7BXML%u89E3%u6790.png%29%0A%0A%0A1.%u83B7%u53D6xml%u548CxmlInputFactory%uFF0CXmlStreamReader%0A%0A%20%20%20%20%0A%20%20%20%20%20%20//%201.%u6784%u5EFAXMLStreamReader%0A%20%20%20%20%20%20//%20File%20xmlFile%20%3D%20new%20File%28getXmlPath%28%29.toString%28%29%29%3B%0A%20%20%20%20%20%20File%20xmlFile%20%3D%20new%20File%28%22filePath%22%29%3B%0A%20%20%20%20%20%20XMLInputFactory%20factory%20%3D%20XMLInputFactory.newInstance%28%29%3B%0A%20%20%20%20%20%20XMLStreamReader%20reader%20%3D%20factory.createXMLStreamReader%28new%20FileInputStream%28xmlFile%29%29%3B%0A%20%20%20%20%20%20while%20%28reader.hasNext%28%29%29%20%7B%0A%20%20%20%20%20%20%20%20int%20type%20%3D%20reader.next%28%29%3B%0A%20%20%20%20%20%20%20%20if%20%28type%20%3D%3D%20XMLStreamReader.START\_ELEMENT%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20String%20localName%20%3D%20reader.getLocalName%28%29%3B%0A%20%20%20%20%20%20%20%20%20%20//%u9009%u62E9%u5C5E%u4E8E%u90A3%u5F20%u8868%0A%20%20%20%20%20%20%20%20%20%20choseConcreteParser%28reader%2C%20localName%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A2.%u9009%u62E9%u76F8%u5E94%u7684%u64CD%u4F5C%20%20%20%20choseConcreteParser%0A%0A%09//table1%20table2%20%u5747%u7EE7%u627Ftable%u8868%0A%20%20%20%20switch%28localName%29%7B%0A%09%20%20%20%20case%20table1%3A%0A%09%20%20%20%20parseAndInsertDB4PublicXml%28reader%2Cnew%20Table1%28%29%29%3B%0A%09%09break%3B%0A%09%09case%20table2%3A%0A%09%09parseAndInsertDB4PublicXml%28reader%2Cnew%20Table2%28%29%29%3B%0A%09%09break%3B%0A%20%20%20%20%7D%0A3.%u89E3%u6790%u76F8%u5173%u7684%u6570%u636EparseAndInsertDB4PublicXml%0A%0A%20%20%20%20//%u65B9%u6CD5%u540D%uFF1AparseAndInsertDB4PublicXml%28reader%20%2C%20table%29%0A%20%20%20%20List%3CMap%3CString%2C%20String%3E%3E%20fieldList%20%3D%20null%3B%0A%20%20%20%20MetaObject%20applyMeta%20%3D%20null%3B%0A%20%20%20%20if%20%28table%20instanceof%20table1%29%20%7B%0A%20%20%20%20%20%20Table1%20table1%20%3D%20%28Table1%20%29%20table%3B%0A%20%20%20%20%20%20applyMeta%20%3D%20SystemMetaObject.forObject%28table1%29%3B%0A%20%20%20%20%20%20fieldList%20%3D%20PropertyUtil.initAnnoPropertyDic%28Table1.class%29%3B%0A%20%20%20%20%20%20elementName%20%3D%20Table1.name%3B%0A%20%20%20%20%7D%20else%20if%20%28table%20instanceof%20table2%29%20%7B%0A%20%20%20%20%20%20Table2%20table2%20%3D%20%28Table2%29%20table%3B%0A%20%20%20%20%20%20applyMeta%20%3D%20SystemMetaObject.forObject%28table2%20%29%3B%0A%20%20%20%20%20%20fieldList%20%3D%20PropertyUtil.initAnnoPropertyDic%28Table2.class%29%3B%0A%20%20%20%20%20%20elementName%20%3D%20Table2.name%3B%0A%20%20%20%20%7D%20%0A%20%20%20%20createConcreteObject%28fieldList%2C%20reader%2C%20applyMeta%2C%20elementName%29%3B%0A%20%20%20%20insertData2DB%28auditAdminData%29%3B%0A%20%20%20%20return%20reader%3B%0A%0A4.PropertyUtil%u7C7B%0A%0A%20%20%0A%0A%60%60%60%0A%20%20public%20class%20PropertyUtil%20%7B%0A%20%20/**%0A%20%20%20**_**%20%u6839%u636E%u5B9E%u4F53%u7C7B%u540D%u83B7%u53D6%u5B57%u6BB5%u540D%u79F0%u548C%u4E2D%u6587%u540D%u79F0%0A%20%20%20**_**%20%0A%20%20%20**_**%20@param%20entityName%20%u5B9E%u4F53%u7C7B%u540D%0A%20%20%20**_**%20@return%20List%3CMap%3CString%2CString%3E%3E%20%u4E3A%u4E86%u540E%u9762%u91C7%u7528%u53CD%u5C04%u673A%u5236%uFF0C%u8FD9%u91CC%u8FD4%u56DE%u7684%u5F97map%u7684key%u662F%u6CE8%u89E3%28%u672C%u6765%u53EA%u662F%u6CE8%u91CA%uFF0C%u4F46%u662F%0A%20%20%20**_**%20%20%20%20%20%20%20%20%20java%u53CD%u5C04%u6CA1%u529E%u6CD5%u83B7%u53D6%u6CE8%u91CA%uFF0C%u6240%u4EE5%u628A%u6CE8%u91CA%u653E%u5230%u6CE8%u89E3%u91CC%u9762%uFF0C%u6240%u4EE5%u8FD9%u91CC%u8981%u6C42%uFF0C%u6CE8%u89E3%u9700%u8981%u548C%20%u5BA1%u8BA1%u7F72%u603B%u8D26xml%u91CC%u9762%u7684element%u7684name%u4E00%u6837%uFF0C%20%u4E0D%u80FD%u6709%u4E1D%u6BEB%u5DEE%u5F02%uFF0C%u5426%u5219%u4F1A%u51FA%u95EE%u9898%29%0A%20%20%20**_**%20%20%20%20%20%20%20%20%20map%u7684value%u662F%u5C5E%u6027property%20%u901A%u8FC7%u4E0A%u9762%u7684%u63CF%u8FF0%uFF0C%u4E5F%u53EF%u4EE5%u53D1%u73B0%u8981%u6C42%u5BA1%u8BA1%u7F72%u6BCF%u4E00%u7C7Bxml%u91CC%u9762%u91CC%u9762%u4E0D%u80FD%u6709%u4E00%u6837%u7684%u5B50element%uFF0C%20%u5426%u5219key%u5C31%u4E00%u6837%u4E86%0A%20%20%20**_**%20%0A%20%20%20**_**/%0A%20%20public%20static%20List%3CMap%3CString%2C%20String%3E%3E%20initAnnoPropertyDic%28Class%20clzz%29%20%7B%0A%20%20%20%20//%20%u7528%u4E8E%u5B58%u50A8%u5B57%u6BB5%u548C%u4E2D%u6587%u503C%u7684%u96C6%u5408%0A%20%20%20%20List%3CMap%3CString%2C%20String%3E%3E%20fieldList%20%3D%20new%20ArrayList%3CMap%3CString%2C%20String%3E%3E%28%29%3B%0A%20%20%20%20//%20%u7528%u4E8E%u5B58%u50A8%u5B9E%u4F53%u7C7B%u5B57%u6BB5%28key%29%u548C%u4E2D%u6587%u540D%28value%29%0A%20%20%20%20Map%3CString%2C%20String%3E%20valueMap%20%3D%20new%20Hashtable%3CString%2C%20String%3E%28%29%3B%0A%20%20%20%20//%20%u83B7%u53D6%u5BF9%u8C61%u4E2D%u6240%u6709%u7684Field%0A%20%20%20%20Field%5B%5D%20fields%20%3D%20clzz.getDeclaredFields%28%29%3B%0A%20%20%20%20//%20%u5FAA%u73AF%u5B9E%u4F53%u7C7B%u5B57%u6BB5%u96C6%u5408%2C%u83B7%u53D6%u6807%u6CE8@PropertyAnnotation%u7684%u5B57%u6BB5%0A%20%20%20%20for%20%28Field%20field%20%3A%20fields%29%20%7B%0A%20%20%20%20%20%20if%20%28field.isAnnotationPresent%28PropertyAnnotation.class%29%29%20%7B%0A%20%20%20%20%20%20%20%20//%20%u83B7%u53D6%u5B57%u6BB5%u540D%0A%20%20%20%20%20%20%20%20String%20fieldNames%20%3D%20field.getName%28%29%3B%0A%20%20%20%20%20%20%20%20//%20%u83B7%u53D6%u5B57%u6BB5%u6CE8%u89E3%0A%20%20%20%20%20%20%20%20PropertyAnnotation%20columnConfig%20%3D%20field.getAnnotation%28PropertyAnnotation.class%29%3B%0A%20%20%20%20%20%20%20%20//%20%u5224%u65AD%u662F%u5426%u5DF2%u7ECF%u83B7%u53D6%u8FC7%u8BE5code%u7684%u5B57%u5178%u6570%u636E%20%u907F%u514D%u91CD%u590D%u83B7%u53D6%0A%20%20%20%20%20%20%20%20if%20%28valueMap.get%28columnConfig.description%28%29%29%20%3D%3D%20null%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20valueMap.put%28columnConfig.description%28%29%2C%20fieldNames%29%3B%0A%20%20%20%20%20%20%20%20%7D%20else%20%7B%0A%20%20%20%20%20%20%20%20%20%20System.out.println%28%22%u51FA%u9519%u4E86%22%29%3B%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20fieldList.add%28valueMap%29%3B//%20%u5C06LinkedHashMap%u653E%u5165List%u96C6%u5408%u4E2D%0A%20%20%20%20return%20fieldList%3B%0A%20%20%7D%0A%7D%0A%60%60%60%0A5.createConcreteObject%u65B9%u6CD5%0A%0A%20%20%20%20%20%20private%20XMLStreamReader%20createConcreteObject%28List%3CMap%3CString%2C%20String%3E%3E%20fieldList%2C%0A%20%20%20%20%20%20XMLStreamReader%20reader%2C%20MetaObject%20applyMeta%2C%20String%20elementName%29%20throws%20Exception%20%7B%0A%0A%20%20%20%20Map%3CString%2C%20String%3E%20map%20%3D%20fieldList.get%280%29%3B%0A%20%20%20%20while%20%28reader.hasNext%28%29%29%20%7B%0A%20%20%20%20%20%20int%20type%20%3D%20reader.next%28%29%3B%0A%20%20%20%20%20%20if%20%28type%20%3D%3D%20XMLStreamReader.END\_ELEMENT%20%26%26%20reader.getLocalName%28%29.equals%28elementName%29%29%20%7B%0A%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20if%20%28type%20%3D%3D%20XMLStreamReader.START\_ELEMENT%29%20%7B%0A%20%20%20%20%20%20%20%20String%20localName%20%3D%20reader.getLocalName%28%29%3B%0A%20%20%20%20%20%20%20%20String%20elementText%20%3D%20reader.getElementText%28%29%3B%0A%20%20%20%20%20%20%20%20for%20%28String%20applyBodySetName%20%3A%20applyMeta.getSetterNames%28%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20if%20%28applyBodySetName.toLowerCase%28%29.equals%28map.get%28localName%29%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20Class%3C%3F%3E%20classType%20%3D%20applyMeta.getGetterType%28applyBodySetName%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20String%20typeName%20%3D%20classType.getName%28%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20if%20%28typeName.equals%28BigDecimal.class.getName%28%29%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%20%28%21StringUtils.isEmpty%28elementText%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20new%20BigDecimal%28elementText%29%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20if%20%28typeName.equals%28Integer.class.getName%28%29%29%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%26%26%20%21StringUtils.isEmpty%28elementText%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%20%28%21StringUtils.isEmpty%28elementText%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20Integer.valueOf%28elementText%29%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20if%20%28typeName.equals%28%22int%22%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%20%28%21StringUtils.isEmpty%28elementText%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20Integer.valueOf%28elementText%29%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20if%20%28typeName.equals%28%22java.util.Date%22%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20DateFormat%20fmt%20%3D%20new%20SimpleDateFormat%28%22yyyyMMdd%22%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20Date%20date%20%3D%20fmt.parse%28elementText%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20date%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20if%20%28typeName.equals%28%22boolean%22%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%20%28elementText.equals%28%220%22%29%29%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20false%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20true%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20applyMeta.setValue%28applyBodySetName%2C%20elementText%29%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20break%3B%0A%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%0A%20%20%20%20%7D%0A%20%20%20%20return%20reader%3B%0A%20%20%20%20%7D%0A%20%0A6.%u6301%u4E45%u5316%0A%0A%20%20%20%20private%20void%20insertData2DB%28AuditAdminData%20auditAdminData%29%20%7B%0A%0A%20%20%20%20if%20%28auditAdminData%20%3D%3D%20null%29%20%7B%0A%20%20%20%20%20%20return%3B%0A%20%20%20%20%7D%0A%0A%20%20%20%20if%20%28table%20instanceof%20table1%29%20%7B%0A%20%20%20%20%20%20table1Repo.add%28%28Table1%29%20table%29%3B%0A%20%20%20%20%7D%20else%20if%20%28table2%20instanceof%20table2%29%20%7B%0A%20%20%20%20%20%20table2Repo.add%28%28Table%29%20table%29%3B%0A%20%20%20%20%7D%0A%20%20%20%20%7D%0A%0A%20%20%0A%20%20%20%20%0A%0A----------%0A%0A%0A%0A%23%23%23%23%23%u591A%u7EBF%u7A0B%u65F6%u7684%u7EBF%u7A0B%u5B89%u5168%u95EE%u9898%0A%0AspringMVC%u9ED8%u8BA4controller%uFF0Cdao%uFF0Cservice%u90FD%u662F%u5355%u4F8B%u7684%u3002%0A%09%u6BCF%u4E2A%u8BF7%u6C42%u8FC7%u6765%u90FD%u4F1A%u8C03%u7528%u539F%u6709instance%u53BB%u5904%u7406%uFF1A%u591A%u7EBF%u7A0B%u51FA%u73B0%u65F6%uFF0C%u91CC%u9762%u7684%u6570%u636E%u5C31%u4E0D%u4F1A%u662F%u7EBF%u7A0B%u5B89%u5168%u7684%u4E86%u3002%0A%0A%u591A%u6570%u60C5%u51B5%u4E0B%u4E0D%u9700%u8981%u8003%u8651%u591A%u7EBF%u7A0B%u5B89%u5168%uFF0C%u4F46%u662F%u5982%u679C%u58F0%u660E%u4E86%u5B9E%u4F8B%u53D8%u91CF%uFF0C%u5219%u4F1A%u51FA%u73B0%u95EE%u9898%uFF1A%0A%0A%20%20%20%20public%20class%20Controller%20extends%20AbstractCommandController%20%7B%20%20%20%20%0A%20%20%20%20protected%20Company%20company%3B%20%20%0A%20%20%20%20protected%20ModelAndView%20handle%28HttpServletRequest%20request%2CHttpServletResponse%20response%2CObject%20command%2CBindException%20errors%29%20throws%20Exception%20%7B%20%20%0A%20%20%20%20%20%20%20%20company%20%3D%20................%3B%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%20%20%7D%20%20%20%0A%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%0A%20%20%20%0A%20%u89E3%u51B3%u65B9%u5F0F%uFF1A%0A%09%20--%20controller%u4E2D%u4F7F%u7528Threadlocal%u53D8%u91CF%0A%09%20--%20%u5728controller%u4E2D%u58F0%u660Escope%3Dprototype%uFF08%u6BCF%u6B21%u90FD%u4F1A%u521B%u5EFA%u65B0%u7684controller%uFF09%0A%09%20%20%20%20%20%20%20%0A%0A%60%60%60%0AController%20%20%0ARequestMapping%28%22/fui%22%29%20%20%0Apublic%20class%20FuiController%20extends%20SpringController%20%7B%20%20%0A//%u8FD9%u4E48%u5B9A%u4E49%u7684%u8BDD%u5C31%u662F%u5355%u4F8B%20%20%0A%20%20%0AController%20%20%0A@Scope%28%22prototype%22%29%20%20%0A@RequestMapping%28%22/fui%22%29%20%20%0Apublic%20class%20FuiController%20extends%20SpringController%20%7B%20%20%0A//%u6BCF%u6B21%u90FD%u521B%u5EFA%0A%60%60%60%0A\***%u5982%u679C%u4E00%u4E2A%u7C7B%u5982%u679C%u6CA1%u6709%u6210%u5458%u53D8%u91CF%uFF0C%u90A3%u8FD9%u4E2A%u7C7B%u80AF%u5B9A%u662Fthread-safe%u7684%uFF0C%u6240%u4EE5UserDaoHibernate%u7684thread-safe%u53D6%u51B3%u4E8E%u5176%u7236%u7C7B%u3002%u800C%20UserManagerImpl%20%u7684%u5B89%u5168%u6027%u53C8%u53D6%u51B3%u4E8EUserDaoHibernate%uFF0C%u6700%u540E%u662FHibernateTemplate%u3002%u53EF%u4EE5%u770B%u51FA%uFF0C%u8FD9%u91CC%u7684bean%u90FD%u5C0F%u5FC3%u7FFC%u7FFC%u7684%u7EF4%u62A4%u5176%u6210%u5458%u53D8%u91CF%uFF0C%20%u6216%u8005%u57FA%u672C%u6CA1%u6709%u6210%u5458%u53D8%u91CF%uFF0C%u800C%u5C06thread-safe%u8F6C%u5AC1%u7ED9Spring%u7684API%u3002%u5982%u679C%u5F00%u53D1%u8005%u6309%u7167%u7EA6%u5B9A%u7684%u6216%u8005%u7528%u81EA%u52A8%u4EA7%u751F%u7684%u5DE5%u5177%uFF08appgen%u4E0D%u9519%uFF09%u6765%u7F16%u5199%u6570%20%u636E%u8BBF%u95EE%u5C42%uFF0C%u662F%u6CA1%u6709%u7EBF%u7A0B%u5B89%u5168%u6027%u7684%u95EE%u9898%u7684%u3002Spring%u672C%u8EAB%u4E0D%u63D0%u4F9B%u8FD9%u65B9%u9762%u7684%u4FDD%u8BC1%u3002_\*_%0A%20%20%0A%0A%20%20%20%20
