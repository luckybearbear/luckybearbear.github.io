---
title: PhantomJSDriver自动生成echart图片
date: 2019-08-29 15:06:08
tags: PhantomJS
categories: JAVA
---
# 依赖

```
<dependency>
	<groupId>com.codeborne</groupId>
	<artifactId>phantomjsdriver</artifactId>
	<version>1.2.1</version>
</dependency>

<dependency>
	<groupId>xml-apis</groupId>
	<artifactId>xml-apis</artifactId>
	<version>1.4.01</version>
</dependency>
```
<!-- more -->

# 公共方法

```java

import com.pexetech.core.util.EmptyUtils;
import com.pexetech.core.util.PropertiesFileUtil;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.phantomjs.PhantomJSDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriverService;
import org.openqa.selenium.remote.DesiredCapabilities;
import sun.misc.BASE64Decoder;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * describe: saveSurfModelUrlToImgUtil
 *
 * @author zj
 * @date 2019/05/23
 */
public class PhantomJSDriverUtil {
    /**
     * @param surfData echart所需数据
     * @param basePath 访问路径
     * @param telPath 模板路径
     * @param code 存储路径
     * @return
     */
    public static String saveSurfModelUrlToImg(String surfData,String basePath,String telPath,String code) {
        List<String> imageBase64List = new ArrayList<String>();
        PhantomJSDriver driver=getPhantomJSDriver();
        // 让浏览器访问空间主页
        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
        driver.get(basePath+telPath);
        JavascriptExecutor js = (JavascriptExecutor) driver;
        //设置surf数据到页面
        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
        //展示echarts
        js.executeScript("showImg("+surfData+")");
        //加入一段休眠时间，防止js执行没完成就进行的 读取echart图片数据的功能。
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //获取echart图片数据

        driver.manage().timeouts().implicitlyWait(5, TimeUnit.SECONDS);
        String phTxt=(String) js.executeScript("return returnEchartImg(rhEcharts)");
        //imageBase64放到list中
        imageBase64List.add(phTxt.replace("data:image/png;base64,",""));
        //获取图片存储路径
        String imgName = System.currentTimeMillis()+".png";
        String imgpath ="";
        String osname = System.getProperties().getProperty("os.name");
        //判断系统的环境win or Linux
        if (osname.equals("Linux")) {
            imgpath = PropertiesFileUtil.getInstance().get("fileRoot");
        }else{
            imgpath = "D:\\fileRoot";
        }
        String imgFilePath = imgpath+File.separator+code+ File.separator+imgName;
        Base64ToImage(phTxt.replace("data:image/png;base64,",""),imgFilePath,imgpath+File.separator+code);
        driver.close();
        driver.quit();
        imgFilePath =  basePath+"/showPicture/detail?imgFile="+URLEncoder.encode(imgFilePath);
        return imgFilePath;
    }

    private static  PhantomJSDriver getPhantomJSDriver() {
        //设置必要参数
        DesiredCapabilities dcaps = new DesiredCapabilities();
        //ssl证书支持
        dcaps.setCapability("acceptSslCerts", true);
        //截屏支持
        dcaps.setCapability("takesScreenshot", true);
        //css搜索支持
        dcaps.setCapability("cssSelectorsEnabled", true);
        //js支持
        dcaps.setJavascriptEnabled(true);
        //驱动支持
        String osname = System.getProperties().getProperty("os.name");
        //判断系统的环境win or Linux
        if (osname.equals("Linux")) {
            dcaps.setCapability(PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY,"/usr/bin/phantomjs");
        } else {
            dcaps.setCapability(PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY,"D:\\phantomjs-2.1.1-windows\\bin\\phantomjs.exe");
        }
        //创建无界面浏览器对象
        PhantomJSDriver driver = new PhantomJSDriver(dcaps);
        return  driver;
    }
    /**
     * base64字符串转换成图片
     * @param imgStr		base64字符串
     * @param imgFilePath	图片存放路径
     * @return
     */
    public static boolean Base64ToImage(String imgStr,String imgFilePath,String savePth) { // 对字节数组字符串进行Base64解码并生成图片

        // 图像数据为空
        if (EmptyUtils.isEmpty(imgStr)){
            return false;
        }

        File saveFile = new File(savePth);
        if (!saveFile.exists()){
            saveFile.mkdirs();
        }
        BASE64Decoder decoder = new BASE64Decoder();
        try {
            // Base64解码
            byte[] b = decoder.decodeBuffer(imgStr);
            for (int i = 0; i < b.length; ++i) {
                // 调整异常数据
                if (b[i] < 0) {
                    b[i] += 256;
                }
            }

            OutputStream out = new FileOutputStream(imgFilePath);
            out.write(b);
            out.flush();
            out.close();

            return true;
        } catch (Exception e) {
            return false;
        }

    }
}

```
# html写法
```js
function returnEchartImg(echartObj) {
        return echartObj.getDataURL();
    }
```

# 问题汇总
## echart图片在outlook客户端查看是显示黑底
option 缺少 `backgroundColor:'#FFFFFF',`