# Java httpclient获取手机号码归属地


免费手机号码归属地API查询接口，截止2019年9月29日，第二个参数不知道啥意思，推荐淘宝的接口，返回值明显，易操作。

一、淘宝网API

API地址：

http://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=17316085037

参数：

tel：手机号码

返回：
```
__GetZoneResult_ = {
    mts:'1731608',
    province:'北京',
    catName:'中国电信',
    telString:'17316085037',
	areaVid:'29400',
	ispVid:'138238560',
	carrier:'北京电信'
}
```



二、百付宝API

API地址：

https://www.baifubao.com/callback?cmd=1059&callback=phone&phone=17316085037 
参数： 
phone：手机号码 
callback：回调函数 
cmd：未知（必须） 
返回：
```
/*fgg_again*/phone({"meta":{"result":"0","result_info":"","jump_url":""},"data":{"operator":"\u7535\u4fe1","area":"\u5317\u4eac","area_operator":"\u5317\u4eac\u7535\u4fe1","support_price":{"500":"507","1000":"998","2000":"1999","3000":"2998","5000":"4998","10000":"9980","20000":"19990","30000":"29985","50000":"49974"},"promotion_info":[]}})
```


然后封装一个HTTPClientUtil:
```
package com.zhrb.util;

import org.apache.commons.httpclient.DefaultHttpMethodRetryHandler;
import org.apache.commons.httpclient.HttpClient;
import org.apache.commons.httpclient.HttpException;
import org.apache.commons.httpclient.methods.GetMethod;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.params.HttpMethodParams;

import java.io.IOException;

/**
 * HttpClient 工具类 以get/post的方式发送数据到指定的http接口 
 ** @author zhrb
 **/
  public class HttpClientUtil {

   /**
 * get方式 ** @param url
  * @return
  */
  public static String getHttp(String url) {
      String responseMsg = "";
      // 1.构造HttpClient的实例
  HttpClient httpClient = new HttpClient();
      // 用于测试的http接口的url
 // 2.创建GetMethod的实例  GetMethod getMethod = new GetMethod(url);
      // 使用系统系统的默认的恢复策略
  getMethod.getParams().setParameter(HttpMethodParams.RETRY_HANDLER,
            new DefaultHttpMethodRetryHandler());
      try {
         // 3.执行getMethod,调用http接口
  httpClient.executeMethod(getMethod);
         // 4.读取内容
  byte[] responseBody = getMethod.getResponseBody();
         // 5.处理返回的内容
  responseMsg = new String(responseBody,"GB2312");

      } catch (HttpException e) {
         //e.printStackTrace();
  } catch (IOException e) {
         //e.printStackTrace();
  } finally {
         // 6.释放连接
  getMethod.releaseConnection();
      }
      return responseMsg;
   }

   /**
 * post方式<br/>  
 *    
 * 构造PostMethod的实例<br/>  
 * PostMethod postMethod = new PostMethod(url);<br/> 
  * 把参数值放入到PostMethod对象中<br/> 
   * 方式1：<br/>  
   * NameValuePair[] data = { new NameValuePair("param1", param1), new<br/>  
   * NameValuePair("param2", param2) }; postMethod.setRequestBody(data);<br/>  
   * 方式2：<br/>  
   * postMethod.addParameter("param1", param1);<br/>  
   * postMethod.addParameter("param2", param2);<br/>  
   ** @return
  */
  public static String postHttp(PostMethod postMethod) {
      String responseMsg = "";

      // 1.构造HttpClient的实例
  HttpClient httpClient = new HttpClient();
      httpClient.getParams().setContentCharset("UTF-8");
      try {
         // 4.执行postMethod,调用http接口
  httpClient.executeMethod(postMethod);// 200
 // 5.读取内容  responseMsg = postMethod.getResponseBodyAsString();
      } catch (HttpException e) {
         //e.printStackTrace();
   } catch (IOException e) {
         //e.printStackTrace();
  } finally {
         // 7.释放连接
  postMethod.releaseConnection();
      }
      return responseMsg;
   }
}
```

然后建立测试类GetPhonePovince
```
package com.zhrb.testDemo;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.zhrb.util.HttpClientUtil;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @ClassName GetPhonePovince
 * @Description
  * @Author Administrator
 * @Date 2019/9/29 9:29
 * @Version
  */ public class GetPhonePovince {
    public static void main(String[] args) {
        String[] phones = {"18198156780",
                "18003434568"
  };
        List<String> phoneList = new ArrayList<>();
        phoneList = Arrays.asList(phones);
        String url = "http://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=";
        String phoneAndPro = "";
        List<String> phoneAndProList = new ArrayList<>();
        for (int i = 0;i < phoneList.size();i ++){
            phoneAndPro = getPro(phoneList.get(i),url);
            phoneAndProList.add(phoneAndPro);
        }
        System.out.println(phoneAndProList);
    }

    private static String getPro(String phone,String url){
        String phoneAndPro = "";
        String response = HttpClientUtil.getHttp(url+phone);
        int index=response.indexOf("=");
        String pro = response.substring(index+1);
        JSONObject jsonObject = JSON.parseObject(pro);
        if ("".equals(pro) || pro == null){
            System.out.println("pro:"+pro);
            throw new RuntimeException();
        }else {
            String telString = (String)jsonObject.get("telString");
            String province = (String)jsonObject.get("province");
            phoneAndPro = telString + "-" + province;
        }
        return phoneAndPro;
    }
}
```

然后打印出来的手机号码及省份信息如下：
```
[18198156780-贵州, 18003434568-山西]
```
