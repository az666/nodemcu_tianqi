---
title: 利用arduino ide制作的桌面摆件
tags: esp8266
date: 2018-5-9 21:50:00
---
## 利用arduino ide制作的桌面摆件 ##

![暂且放在盒子里](https://i.loli.net/2018/06/17/5b265c62467da.jpg)
![支持微信扫一扫配网](https://i.loli.net/2018/06/17/5b265c8d2ce40.jpg)
<!--more-->
 - 工具
 - nodemcu
 - arduino ide
 - 串口屏（淘宝38元）
 

> 这是很早以前就想做的作品，因为它不像大的作品那样，所以比较拖沓，一直没做，最近想给女朋友做一个桌面摆件，所以用了一整天的时间来做这个小玩物，整体思路就是利用esp的网络功能来获取网页数据接口返回的数据包括时间，天气，甚至英语，发给串口屏来显示，其中也是一路艰辛，毕竟好久没玩硬件了，也是相当麻烦的。

----------

> 首先是串口屏的ui设计，极简，毕竟自己没有太高级的审美，串口屏的教程也是相当复杂的，之前淘宝店给的教程一直没空看，当时感觉老复杂了，就一直搁置了。这次果断拿出来认真看了看，突然发现还是很简单的，毕竟由相应的上位机使用。


----------

> 然后就是代码的编写，总结如下：
> 最厉害的就是自动配网的功能-堪称超级无敌：

```
void smartConfig()
{
  WiFi.mode(WIFI_STA);
  Serial.println("\r\nWait for Smartconfig");
  WiFi.beginSmartConfig();
  while (1)
  {
    Serial.print(".");
    digitalWrite(LED, 0);
    sao_sao();
    delay(500);
    digitalWrite(LED, 1);
    delay(500);
    if (WiFi.smartConfigDone())
    {
      Serial.println("SmartConfig Success");
      Serial.printf("SSID:%s\r\n", WiFi.SSID().c_str());
      Serial.printf("PSW:%s\r\n", WiFi.psk().c_str());
      break;
    }
  }
}
```

> 网页数据分析这次都是用的GET访问

```
void http() {
  HTTPClient http;
  http.begin(webadd);
  int httpCode = http.GET();
  if (httpCode > 0) {
    payload = http.getString();
    //Serial.println(payload);
    int a = payload.indexOf("101180101");//自己搜索链接可知指定城市的id郑州市的id为‘101180101’
    //indexof();在字符串中中查找字符位置，返回值为给定字符的第一个位置，查找失败返回-1
    com = payload.substring(a - 280, a + 12);
  }
  http.end();
}
```


----------

> 还有就是网页数据解析的精髓:

 - 字符处理--适用于网页源码的解析
 - JSON解析--适用于api反馈的json


----------

```
/*************************************截取字符数据处理*******************************************/
void data() {
  int i = 0;
  i = com.indexOf("tem1");
  tem1 = com.substring(i + 6, i + 8);
  i = 0;
  i = com.indexOf("tem2");  //tem2="24"
  tem2 = com.substring(i + 6, i + 8);
  i = 0;
  i = com.indexOf("temNow");
  temnow = com.substring(i + 8, i + 10);
  i = 0;
  i = com.indexOf("humidity");
  humi = com.substring(i + 10, i + 13); //humidity="27%"
  i = 0;
  i = com.indexOf("state1");
  state1 = com.substring(i + 8, i + 9);
  i = 0;
  i = com.indexOf("state2");
  state2 = com.substring(i + 7, i + 11);
  i = 0;
}
```


----------

```
/*********************************获取时间**************************************/
void get_time()
{
  HTTPClient http;
  http.begin(time_url);
  int httpCode = http.GET();
  if (httpCode > 0) {
    payload = http.getString();
    //Serial.println(payload);
    /************************************json 数据处理********************************************/
    DynamicJsonDocument doc;
    String  input =   payload;
    deserializeJson(doc, input);
    JsonObject& obj = doc.as<JsonObject>();
    String  daytime = obj["sysTime2"];
    time0 = daytime;
    time1 = daytime.substring(0, 10);//2018-06-17
    time2 =  daytime.substring(11, 19);//2018-06-17
  }
  http.end();
}
```


----------

> 整体代码放在GitHub里，欢迎关注
> [https://github.com/az666/nodemcu_tianqi](https://github.com/az666/nodemcu_tianqi)


----------

> 有问题欢迎讨论。
