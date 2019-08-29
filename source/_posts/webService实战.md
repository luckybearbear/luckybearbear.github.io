---
title: webService实战
date: 2019-08-10 15:39:53
tags: webService
categories: JAVA
---
# 所需依赖
 <!-- more -->
```
<dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>commons-discovery</groupId>
            <artifactId>commons-discovery</artifactId>
            <version>0.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.ws.xmlschema</groupId>
            <artifactId>xmlschema-core</artifactId>
            <version>2.2.1</version>
        </dependency>
        <dependency>
            <groupId>com.thoughtworks.xstream</groupId>
            <artifactId>xstream</artifactId>
            <version>1.4.10</version>
        </dependency>
        <dependency>
            <groupId>org.apache.axis</groupId>
            <artifactId>axis</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-jaxws</artifactId>
            <version>3.2.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http</artifactId>
            <version>3.2.7</version>
        </dependency>
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http-jetty</artifactId>
            <version>3.2.7</version>
        </dependency>
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc-api</artifactId>
            <version>1.1</version>
        </dependency>
```
# 定义实体、公共方法
![定义实体、公共方法](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20190810154550.png)
 ## model
```java
import java.util.List;

/**
 * Project:<记录接口传送结果或内容>
 * 
 * Comments:<>
 * 
 * 
 * @see
 * @param <T>
 */
public class RealtechMiot<T> {
	private WSRequest wsRequest;
	private WSResult wsResult;
	private List<T> datas;

	public RealtechMiot() {
	}

	public RealtechMiot(WSRequest wsRequest, List<T> datas) {
		this.wsRequest = wsRequest;
		this.datas = datas;
	}

	public RealtechMiot(WSResult wsResult, List<T> datas) {
		this.wsResult = wsResult;
		this.datas = datas;
	}

	public WSRequest getWsRequest() {
		return this.wsRequest;
	}

	public void setWsRequest(WSRequest wsRequest) {
		this.wsRequest = wsRequest;
	}

	public WSResult getWsResult() {
		return this.wsResult;
	}

	public void setWsResult(WSResult wsResult) {
		this.wsResult = wsResult;
	}

	public List<T> getDatas() {
		return this.datas;
	}

	public void setDatas(List<T> datas) {
		this.datas = datas;
	}
}

```
```java
import com.thoughtworks.xstream.annotations.XStreamAlias;

/**
 * Project:<接口>
 * <p>
 * Comments:<用来传送信息>
 * <p>
 *
 * @see
 */

@XStreamAlias("head")
public class WSRequest {
    public WSRequest() {
    }

    public WSRequest(String function_id) {
        this.function_id = function_id;
    }

    public WSRequest(String function_id, String version) {
        this.function_id = function_id;
        this.version = version;
    }

    @XStreamAlias("function_id")
    private String function_id;// 方法名
    private String version;// 方法版本

    public String getFunction_id() {
        return this.function_id;
    }

    public void setFunction_id(String function_id) {
        this.function_id = function_id;
    }

    public String getVersion() {
        return this.version;
    }

    public void setVersion(String version) {
        this.version = version;
    }
}

```
```java
import com.thoughtworks.xstream.annotations.XStreamAlias;

/**
 * Project:<接口>
 * <p>
 * Comments:<用来记录接口返回的结果>
 * <p>
 */
@XStreamAlias("head")
public class WSResult {
    public WSResult() {
    }

    public WSResult(boolean success, String code) {
        this.success = success;
        this.code = code;
    }

    public WSResult(boolean success, String code, String msg) {
        this.success = success;
        this.msg = msg;
        this.code = code;
    }

    private boolean success;
    private String msg;
    private String code;


    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public String getMsg() {
        return this.msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public String getCode() {
        return this.code;
    }

    public void setCode(String code) {
        this.code = code;
    }

}

```
```java
/**
 * Project:<接口>
 * 
 * Comments:<用来记录接口返回的结果>
 * 
 * 
 * @see data 一般为一个json或jsonArrayList
 */
public class XmlResult {
	private boolean success;
	private String msg;
	private Object data;
	private String errorCode;

	public XmlResult() {
	}

	public XmlResult(boolean success, String msg, Object data, String errorCode) {
		this.success = success;
		this.msg = msg;
		this.data = data;
		this.errorCode = errorCode;
	}

	public XmlResult(boolean success, String msg) {
		super();
		this.success = success;
		this.msg = msg;
	}

	public XmlResult(boolean success, String msg, Object data) {
		super();
		this.success = success;
		this.msg = msg;
		this.data = data;
	}

	public boolean isSuccess() {
		return success;
	}

	public void setSuccess(boolean success) {
		this.success = success;
	}

	public String getMsg() {
		return msg;
	}

	public void setMsg(String msg) {
		this.msg = msg;
	}

	public Object getData() {
		return data;
	}

	public void setData(Object data) {
		this.data = data;
	}

	public String getErrorCode() {
		return errorCode;
	}

	public void setErrorCode(String errorCode) {
		this.errorCode = errorCode;
	}
}

```
## util
```java
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerConfigurationException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.*;
import java.util.Properties;

public class DOMUtils {
	/**
	 * 初始化一个空Document对象返回。
	 * 
	 * @return a Document
	 */
	public static Document newXMLDocument() {
		try {
			return newDocumentBuilder().newDocument();
		}
		catch (ParserConfigurationException e) {
			throw new RuntimeException(e.getMessage());
		}
	}

	/**
	 * 初始化一个DocumentBuilder
	 * 
	 * @return a DocumentBuilder
	 * @throws ParserConfigurationException
	 */
	public static DocumentBuilder newDocumentBuilder()
			throws ParserConfigurationException {
		return newDocumentBuilderFactory().newDocumentBuilder();
	}

	/**
	 * 初始化一个DocumentBuilderFactory
	 * 
	 * @return a DocumentBuilderFactory
	 */
	public static DocumentBuilderFactory newDocumentBuilderFactory() {
		DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
		dbf.setNamespaceAware(true);
		return dbf;
	}

	/**
	 * 将传入的一个XML String转换成一个org.w3c.dom.Document对象返回。
	 * 
	 * @param xmlString
	 *            一个符合XML规范的字符串表达。
	 * @return a Document
	 */
	public static Document parseXMLDocument(String xmlString) {
		if (xmlString == null) {
			throw new IllegalArgumentException();
		}
		try {
			return newDocumentBuilder().parse(
					new InputSource(new StringReader(xmlString)));
		}
		catch (Exception e) {
			throw new RuntimeException(e.getMessage());
		}
	}

	/**
	 * 给定一个输入流，解析为一个org.w3c.dom.Document对象返回。
	 * 
	 * @param input
	 * @return a org.w3c.dom.Document
	 */
	public static Document parseXMLDocument(InputStream input) {
		if (input == null) {
			throw new IllegalArgumentException("参数为null！");
		}
		try {
			return newDocumentBuilder().parse(input);
		}
		catch (Exception e) {
			throw new RuntimeException(e.getMessage());
		}
	}

	/**
	 * 给定一个文件名，获取该文件并解析为一个org.w3c.dom.Document对象返回。
	 * 
	 * @param fileName
	 *            待解析文件的文件名
	 * @return a org.w3c.dom.Document
	 */
	public static Document loadXMLDocumentFromFile(String fileName) {
		if (fileName == null) {
			throw new IllegalArgumentException("未指定文件名及其物理路径！");
		}
		try {
			return newDocumentBuilder().parse(new File(fileName));
		}
		catch (SAXException e) {
			throw new IllegalArgumentException("目标文件（" + fileName
					+ "）不能被正确解析为XML！" + e.getMessage());
		}
		catch (IOException e) {
			throw new IllegalArgumentException("不能获取目标文件（" + fileName + "）！"
					+ e.getMessage());
		}
		catch (ParserConfigurationException e) {
			throw new RuntimeException(e.getMessage());
		}
	}

	/*
	 * 把dom文件转换为xml字符串
	 */
	public static String toStringFromDoc(Document document) {
		String result = null;

		if (document != null) {
			StringWriter strWtr = new StringWriter();
			StreamResult strResult = new StreamResult(strWtr);
			TransformerFactory tfac = TransformerFactory.newInstance();
			try {
				Transformer t = tfac.newTransformer();
				t.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
				t.setOutputProperty(OutputKeys.INDENT, "yes");
				t.setOutputProperty(OutputKeys.METHOD, "xml"); // xml, html,
				// text
				t.setOutputProperty(
						"{http://xml.apache.org/xslt}indent-amount", "4");
				t.transform(new DOMSource(document.getDocumentElement()),
						strResult);
			}
			catch (Exception e) {
				System.err.println("XML.toString(Document): " + e);
			}
			result = strResult.getWriter().toString();
			try {
				strWtr.close();
			}
			catch (IOException e) {
				e.printStackTrace();
			}
		}

		return result;
	}

	/*
	 * 把dom文件转换为xml字符串
	 */
	public static String toStringFromDoc(Element element) {
		String result = null;

		if (element != null) {
			StringWriter strWtr = new StringWriter();
			StreamResult strResult = new StreamResult(strWtr);
			TransformerFactory tfac = TransformerFactory.newInstance();
			try {
				Transformer t = tfac.newTransformer();
				t.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
				t.setOutputProperty(OutputKeys.INDENT, "yes");
				t.setOutputProperty(OutputKeys.METHOD, "xml"); // xml, html,
				// text
				t.setOutputProperty(
						"{http://xml.apache.org/xslt}indent-amount", "4");
				t.transform(new DOMSource(element), strResult);
			}
			catch (Exception e) {
				System.err.println("XML.toString(Document): " + e);
			}
			result = strResult.getWriter().toString();
			try {
				strWtr.close();
			}
			catch (IOException e) {
				e.printStackTrace();
			}
		}

		return result;
	}

	/**
	 * 给定一个节点，将该节点加入新构造的Document中。
	 * 
	 * @param node
	 *            a Document node
	 * @return a new Document
	 */

	public static Document newXMLDocument(Node node) {
		Document doc = newXMLDocument();
		doc.appendChild(doc.importNode(node, true));
		return doc;
	}

	/**
	 * 将传入的一个DOM Node对象输出成字符串。如果失败则返回一个空字符串""。
	 * 
	 * @param node
	 *            DOM Node 对象。
	 * @return a XML String from node
	 */

	/*
	 * public static String toString(Node node) { if (node == null) { throw new
	 * IllegalArgumentException(); } Transformer transformer = new
	 * Transformer(); if (transformer != null) { try { StringWriter sw = new
	 * StringWriter(); transformer .transform(new DOMSource(node), new
	 * StreamResult(sw)); return sw.toString(); } catch (TransformerException
	 * te) { throw new RuntimeException(te.getMessage()); } } return ""; }
	 */

	/**
	 * 将传入的一个DOM Node对象输出成字符串。如果失败则返回一个空字符串""。
	 * 
	 * @param node
	 *            DOM Node 对象。
	 * @return a XML String from node
	 */

	/*
	 * public static String toString(Node node) { if (node == null) { throw new
	 * IllegalArgumentException(); } Transformer transformer = new
	 * Transformer(); if (transformer != null) { try { StringWriter sw = new
	 * StringWriter(); transformer .transform(new DOMSource(node), new
	 * StreamResult(sw)); return sw.toString(); } catch (TransformerException
	 * te) { throw new RuntimeException(te.getMessage()); } } return ""; }
	 */

	/**
	 * 获取一个Transformer对象，由于使用时都做相同的初始化，所以提取出来作为公共方法。
	 * 
	 * @return a Transformer encoding gb2312
	 */

	public static Transformer newTransformer() {
		try {
			Transformer transformer = TransformerFactory.newInstance()
					.newTransformer();
			Properties properties = transformer.getOutputProperties();
			properties.setProperty(OutputKeys.ENCODING, "gb2312");
			properties.setProperty(OutputKeys.METHOD, "xml");
			properties.setProperty(OutputKeys.VERSION, "1.0");
			properties.setProperty(OutputKeys.INDENT, "no");
			transformer.setOutputProperties(properties);
			return transformer;
		}
		catch (TransformerConfigurationException tce) {
			throw new RuntimeException(tce.getMessage());
		}
	}

	/**
	 * 返回一段XML表述的错误信息。提示信息的TITLE为：系统错误。之所以使用字符串拼装，主要是这样做一般 不会有异常出现。
	 * 
	 * @param errMsg
	 *            提示错误信息
	 * @return a XML String show err msg
	 */
	/*
	 * public static String errXMLString(String errMsg) { StringBuffer msg = new
	 * StringBuffer(100);
	 * msg.append("<?xml version="1.0" encoding="gb2312" ?>");
	 * msg.append("<errNode title="系统错误" errMsg="" + errMsg + ""/>"); return
	 * msg.toString(); }
	 */
	/**
	 * 返回一段XML表述的错误信息。提示信息的TITLE为：系统错误
	 * 
	 * @param errMsg
	 *            提示错误信息
	 * @param errClass
	 *            抛出该错误的类，用于提取错误来源信息。
	 * @return a XML String show err msg
	 */
	/*
	 * public static String errXMLString(String errMsg, Class errClass) {
	 * StringBuffer msg = new StringBuffer(100);
	 * msg.append("<?xml version='1.0' encoding='gb2312' ?>");
	 * msg.append("<errNode title="
	 * 系统错误" errMsg=""+ errMsg + "" errSource=""+ errClass.getName()+ ""/>");
	 * 　return msg.toString(); }
	 */
	/**
	 * 返回一段XML表述的错误信息。
	 * 
	 * @param title
	 *            提示的title
	 * @param errMsg
	 *            提示错误信息
	 * @param errClass
	 *            抛出该错误的类，用于提取错误来源信息。
	 * @return a XML String show err msg
	 */

	public static String errXMLString(String title, String errMsg,
			Class errClass) {
		StringBuffer msg = new StringBuffer(100);
		msg.append("<?xml version='1.0' encoding='utf-8' ?>");
		msg.append("<errNode title=" + title + "errMsg=" + errMsg
				+ "errSource=" + errClass.getName() + "/>");
		return msg.toString();
	}

}
```
```java
import com.pexetech.core.util.EmptyUtils;
import com.pexetech.core.util.PropertiesFileUtil;
import com.pexetech.rest.common.model.WSResult;
import org.apache.axis.client.Call;
import org.apache.axis.client.Service;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import javax.xml.namespace.QName;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;
import java.io.StringReader;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;

public class XmlOperationUtil<T> {
    protected final Log _log = LogFactory.getLog(getClass());
    private final String PATTERN = "yyyy-MM-dd HH:mm:ss";
    private final SimpleDateFormat FORMAT = new SimpleDateFormat(PATTERN);

    public List<T> getTByname(Class<T> cls, NodeList nodeList) {
        List<T> tList = new ArrayList<T>();
        for (int i = 0; i < nodeList.getLength(); i++) {
            Element ele = (Element) nodeList.item(i);
            T t = XmlUtil.toBean(DOMUtils.toStringFromDoc(ele), cls);
            tList.add(t);
        }
        return tList;
    }

    public Element getElerootByTagName(String xml, String tagName) {
        NodeList bomNodeList = getElerootsByTagName(xml, tagName);
        Element bomRoot = (Element) bomNodeList.item(0);
        return bomRoot;
    }

    public NodeList getElerootsByTagName(String xml, String tagName) {
        if (EmptyUtils.isBlank(xml)) {
            return null;
        }
        Document document = createDocument(xml);
        return document.getElementsByTagName(tagName);
    }

    public WSResult createErrWSResult() {
        WSResult r = new WSResult();
        r.setCode("-100");
        r.setMsg("系统出现异常！");
        return r;
    }

    public Document createDocument(String xml) {
        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory
                    .newInstance();
            DocumentBuilder builder;
            builder = factory.newDocumentBuilder();
            if (EmptyUtils.isBlank(xml)) {
                return null;
            }
            StringReader sr = new StringReader(xml);
            InputSource is = new InputSource(sr);
            return builder.parse(is);
        } catch (SAXException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (ParserConfigurationException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }

    public String callWebService(String xml, String method) {
        try {
            // 指出service所在完整的URL
            String endpoint = PropertiesFileUtil.getInstance("api").get("webservicePath");
            //调用接口的targetNamespace
            String targetNamespace = "http://webservice.web.rest.pexetech.com/";
            //所调用接口的方法method
            method = method == null ? PropertiesFileUtil.getInstance("api").get("methodName") : method;
            //以上需要配置
            _log.info("发送信息：" + xml);
            // 创建一个服务(service)调用(call)
            Service service = new Service();
//            javax.xml.rpc.Call call= service.createCall();
            Call call = (Call) service.createCall();// 通过service创建call对象
            // 设置service所在URL
            call.setTargetEndpointAddress(new java.net.URL(endpoint));
            call.setOperationName(new QName(targetNamespace, method));
            call.setUseSOAPAction(true);
            //变量最好只是用String类型，其他类型会报错
            call.addParameter(new QName(targetNamespace, "xml"), org.apache.axis.encoding.XMLType.XSD_STRING, javax.xml.rpc.ParameterMode.IN);//设置参数名 state  第二个参数表示String类型,第三个参数表示入参
            call.setReturnType(org.apache.axis.encoding.XMLType.XSD_STRING);// 设置返回类型
            // String path = targetNamespace + method;
            // call.setSOAPActionURI(path);
            String jsonString = (String) call.invoke(new Object[]{xml});//此处为数组，有几个变量传几个变量
//			//将json字符串转换为JSON对象
//			JSON json = (JSON) JSON.parse(jsonString);
//			//将接送对象转为java对象,此处用object代替，用的时候转换为你需要是用的对象就行了
//			Object object = JSON.toJavaObject(json, Object.class);//注意别到错包com.alibaba.fastjson.JSON
            _log.info("返回结果：" + jsonString);
            return jsonString;
        } catch (Exception e) {
            _log.error("callWebService error ", e);
        }
        return null;
    }
}

```
```java
import com.thoughtworks.xstream.XStream;
import com.thoughtworks.xstream.io.xml.DomDriver;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;
import java.io.StringReader;

/**
 * 输出xml和解析xml的工具类
 *
 * @ClassName:XmlUtil
 * @author: chenyoulong Email: chen.youlong@payeco.com
 * @date :2012-9-29 上午9:51:28
 * @Description:TODO
 */
public class XmlUtil {

    /**
     * java 转换成xml
     *
     * @param obj 对象实例
     * @return String xml字符串
     * @Title: toXml
     * @Description: TODO
     */
    public static String toXml(Object obj) {
        XStream xstream = new XStream();
        XStream.setupDefaultSecurity(xstream);
        xstream.allowTypes(new Class[]{XmlUtil.class});
        // XStream xstream=new XStream(new DomDriver()); //直接用jaxp dom来解释
        // XStream xstream=new XStream(new DomDriver("utf-8"));
        // //指定编码解析器,直接用jaxp dom来解释

        // //如果没有这句，xml中的根元素会是<包.类名>；或者说：注解根本就没生效，所以的元素名就是类的属性
        xstream.processAnnotations(obj.getClass()); // 通过注解方式的，一定要有这句话
        return xstream.toXML(obj);
    }

    /**
     * 将传入xml文本转换成Java对象
     *
     * @param xmlStr
     * @param cls    xml对应的class类
     * @return T xml对应的class类的实例对象
     * <p>
     * 调用的方法实例：PersonBean person=XmlUtil.toBean(xmlStr,
     * PersonBean.class);
     * @Title: toBean
     * @Description: TODO
     */
    public static <T> T toBean(String xmlStr, Class<T> cls) {
        // 注意：不是new Xstream(); 否则报错：java.lang.NoClassDefFoundError:
        // org/xmlpull/v1/XmlPullParserFactory
        XStream xstream = new XStream(new DomDriver());
        xstream.processAnnotations(cls);
        T obj = (T) xstream.fromXML(xmlStr);
        return obj;
    }

    public static NodeList getRootEleList(String bomXml, String rootName) {
        Document document = createDocument(bomXml);
        NodeList bomNodeList = document.getElementsByTagName(rootName);
        return bomNodeList;
    }

    public static Element getRootEle(String bomXml, String rootName) {
        Document document = createDocument(bomXml);
        NodeList bomNodeList = document.getElementsByTagName(rootName);
        if (bomNodeList == null || bomNodeList.getLength() == 0) {
            return null;
        }
        Element bomRoot = (Element) bomNodeList.item(0);
        return bomRoot;
    }

    public static String getRootEleValue(String bomXml, String rootName) {
        Document document = createDocument(bomXml);
        NodeList bomNodeList = document.getElementsByTagName(rootName);
        if (bomNodeList == null || bomNodeList.getLength() == 0) {
            return null;
        }
        Element bomRoot = (Element) bomNodeList.item(0);
        return bomRoot.getTextContent();
    }

    // 返回根目录下元素的值
    public static String getRootDocumentValue(String bomXml, String rootName) {
        Document document = createDocument(bomXml);
        NodeList bomNodeList = document.getChildNodes();
        for (int i = 0; i < bomNodeList.getLength(); i++) {
            Element bomRoot = (Element) bomNodeList.item(i);
            NodeList childrenNodeList = bomRoot.getChildNodes();
            for (int j = 0; j < childrenNodeList.getLength(); j++) {
                if (childrenNodeList.item(j).hasChildNodes()) {
                    Element childrenRoot = (Element) childrenNodeList.item(j);
                    if (rootName.equals(childrenRoot.getTagName())) {
                        return childrenRoot.getTextContent();
                    }
                }
            }
        }
        return null;
    }

    public static String getRootEleValue(Element ele, String rootName) {
        NodeList bomNodeList = ele.getElementsByTagName(rootName);
        if (bomNodeList == null || bomNodeList.getLength() == 0) {
            return null;
        }
        Element bomRoot = (Element) bomNodeList.item(0);
        return bomRoot.getTextContent();
    }

    public static Document createDocument(String xml) {
        try {
            DocumentBuilderFactory factory = DocumentBuilderFactory
                    .newInstance();
            DocumentBuilder builder;
            builder = factory.newDocumentBuilder();

            StringReader sr = new StringReader(xml);
            InputSource is = new InputSource(sr);
            return builder.parse(is);
        } catch (SAXException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (ParserConfigurationException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return null;
    }

//    public static XmlResult sendWebService(String command, String xml) {
//        try {
//            // Client c = new Client(new
//            // URL("http://wuxin20642.xicp.net:19845/services/myService?wsdl"));
//            Client c = new Client(new URL("http://127.0.0.1:7890/HelloWorld?wsdl"));
//            Object[] result = c.invoke(command, new String[]{xml});
//            return new XmlResult(true, "call success.", (String) result[0]);
//        } catch (ConnectException ce) {
//            return new XmlResult(false, "call ConnectException.", null, "404");
//        } catch (Exception e) {
//            System.err.println(e.toString());
//        }
//        return new XmlResult(false, "call error.", null, "500");
//    }
}
```
```java
import com.pexetech.core.util.EmptyUtils;
import com.pexetech.rest.common.model.RealtechMiot;
import com.pexetech.rest.common.model.WSRequest;
import com.pexetech.rest.common.model.WSResult;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

import java.util.List;

public class XmlUtilForMiot<T> extends XmlOperationUtil<T> {
	public static final String ROOT = "realtech";
	public static final String BODY = "body";
	public static final String HEAD = "head";
	public static final String DATA = "data";

	public RealtechMiot<T> toRequestBean(String xmlStr, Class<T> cls) {
		Element rootEle = getRootEle(xmlStr);

		Element head = getElerootByTagName(DOMUtils.toStringFromDoc(rootEle),
				HEAD);
		WSRequest wsRequest = XmlUtil.toBean(DOMUtils.toStringFromDoc(head),
				WSRequest.class);
		List<T> tList = createTlist(cls, rootEle);
		RealtechMiot<T> tm = new RealtechMiot<T>(wsRequest, tList);
		return tm;
	}

	private List<T> createTlist(Class<T> cls, Element rootEle) {
		Element bodyEle = getBodyEle(DOMUtils.toStringFromDoc(rootEle));
		List<T> tList = null;
		if (cls != null) {
			NodeList nodeList = getElerootsByTagName(
					DOMUtils.toStringFromDoc(bodyEle), DATA);
			if (nodeList != null && nodeList.getLength() > 0) {
				tList = getTByname(cls, nodeList);
			}
		}
		return tList;
	}

	public RealtechMiot<T> toResultBean(String xmlStr, Class<T> cls) {
		Element rootEle = getRootEle(xmlStr);
		Element head = getElerootByTagName(DOMUtils.toStringFromDoc(rootEle),
				HEAD);
		WSResult wsResult = XmlUtil.toBean(DOMUtils.toStringFromDoc(head),
				WSResult.class);
		List<T> tList = createTlist(cls, rootEle);
		RealtechMiot<T> tm = new RealtechMiot<T>(wsResult, tList);
		return tm;
	}

	public String toRequestXml(WSRequest wsRequest, List<T> datas) {
		String head = XmlUtil.toXml(wsRequest);
		String dataStr = listToXmlStr(datas);

		String result = connectXmlStr(head, dataStr);
		return result;
	}

	public String toRequestXml(WSRequest wsRequest, Object datas) {
		String head = XmlUtil.toXml(wsRequest);
		String dataStr = listToXmlStr(datas);

		String result = connectXmlStr(head, dataStr);
		return result;
	}

	public String toResultXml(WSResult wsResult, List<T> datas) {
		try {
			String head = XmlUtil.toXml(wsResult);
			String dataStr = listToXmlStr(datas);

			String result = connectXmlStr(head, dataStr);
			return result;
		}
		catch (Exception e) {
			String head = XmlUtil.toXml(createErrWSResult());
			return connectXmlStr(head, null);
		}
	}

	private String connectXmlStr(String head, String dataStr) {
		if (dataStr == null || EmptyUtils.isBlank(dataStr)) {
			dataStr = "<data></data>";
		}
		String result = "<" + ROOT + ">" + head + "<" + BODY + ">" + dataStr
				+ "</" + BODY + ">" + "</" + ROOT + ">";
		// 处理xStreambug，一个"_"会变成2个
		if (result.contains("__")) {
			result = result.replace("__", "_");
		}
		return result;
	}

	private String listToXmlStr(List<T> datas) {
		String dataStr = "";
		if (datas != null) {
			for (T t : datas) {
				dataStr += XmlUtil.toXml(t);
			}
		}
		return dataStr;
	}

	private String listToXmlStr(Object datas) {
		String dataStr = XmlUtil.toXml(datas);
		return dataStr;
	}

	public Element getRootEle(String xml) {
		return getElerootByTagName(xml, ROOT);
	}

	public Element getBodyEle(String xml) {
		return getElerootByTagName(xml, BODY);
	}
}

```
# spring注入
applicationContext-webservice.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://cxf.apache.org/jaxws
        http://cxf.apache.org/schemas/jaxws.xsd">

    <jaxws:endpoint id="pexeService" implementor="com.pexetech.rest.web.webservice.PexeServiceImpl"
                    address="http://127.0.0.1:6789/webService"/>
    <!--物料入库-->
        <bean id="create_material_input" class="com.pexetech.rest.web.webservice.service.InventoryInputRestService">
            <property name="function" value="createMaterialInput"/>
        </bean>
</beans>
```
# 工厂
![](https://raw.githubusercontent.com/luckybearbear/img/master/hexo/20190810160713.png)
```java
public interface WSFunctionFactory {
	public String execute(String message);
}

```
```java
import com.pexetech.core.util.EmptyUtils;
import com.pexetech.rest.web.webservice.WSFunctionFactory;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import java.lang.reflect.InvocationTargetException;

public abstract class WebService implements WSFunctionFactory {
    protected final Log _log = LogFactory.getLog(getClass());
    private String _function;
    private String _companyId = "c3457cb41baa4b568e05799751de33e4";

    public String getCompanyId() {
        return _companyId;
    }

    public void setCompanyId(String companyId) {
        this._companyId = companyId;
    }

    public void setFunction(String function) {
        this._function = function;
    }

    public String getFunction() {
        return this._function;
    }

    public String executeMethod(String message) {
        return null;
    }

    public String execute(String message) {
        String s = null;
        try {
            if (!EmptyUtils.isBlank(getFunction())) {
                s = (String) this.getClass()
                        .getMethod(getFunction(), String.class)
                        .invoke(this, message);
            } else {
                s = executeMethod(message);
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (SecurityException e) {
            e.printStackTrace();
        }
        return s;
    }
}
```
## 提供者
```java
public class CreateUnbomRequestRestService extends WebService {
     private XmlUtilForMiot<PurchaseUnbomRequestPXView> iixml = new XmlUtilForMiot<PurchaseUnbomRequestPXView>();
      public String createUnbomRequest(String msg) {
             WSResult r = null;
        r = new WSResult(true, "100", "抛送成功");
                    return iixml.toResultXml(r, resultList);    
      }
    
}
```