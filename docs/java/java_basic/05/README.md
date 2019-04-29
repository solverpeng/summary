# JSON

## 简介
json的官方网络媒体类型：application/json。
扩展名为：.json。

是一种数据交换格式。

## 语法
数据使用键/值对表示
使用大括号保存对象，每个名称后面跟着一个 ':' 冒号，键值对之间用 ',' 逗号分隔。
使用方括号保存数组，数组使用 ',' 分割。

## JSON 与 XML文件对比
冗余度：
XML 比 JSON冗余，因此编写 JSON更快。
数组用法：
XML不包含数组，而 JSON 包含数组

由前台 JSON 字符串到后台的 Java数据类型，过程其实是根据 JSON 和 Java的类型映射码表进行的解码。且在Java中，JSONObject 对应的就是 java.util.Map，JSONArray 对应的是 java.util.List，可以使用 Map 或 List的标准操作访问他们。

## JS 中将 JSON 字符串转换为 JSON 对象
1. var jsonObj = eval('(' + jsonStr + ')')
2. var jsonObj = JSON.parse(jsonStr);

## JS 中将 JSON 对象转换为 JSON 字符串
1. var jsonStr = jsonObj.toJSONString();
2. var jsonStr = JSON.stringify(jsonObj);

## JSON 与 Ajax
Ajax dataType：预期服务器返回的数据类型，指定为 "json"，则将返回的Json字符串转换为Json对象或Json数组。

## Java 解码 Json
导包：net.sf.json.JSONArray 和 net.sf.json.JSONObject
解码：
将请求的 Json 字符串转化为 JSON 对象：
```java
JSONObject jsonObj = JSONObject.fromObject(str);
```

将请求的 Json 字符串转化为 JSON 数组对象：
```java
JSONArray jsonArr = JSONArray.fromObject(str);
if(jsonArr.size()>0){
	for(int i=0;i<jsonArr.size();i++){
		// 遍历 jsonarray 数组，把每一个对象转成 json 对象
		JSONObject jsonObj = jsonArr.getJSONObject(i);
	}
}
```

## 疑问
在不指定 dataType 的情况下发送ajax请求，
若在后台指定响应类型为 json，那我响应成功的回调函数接收到的值是 json 对象还是一个字符串?

在指定 dataType 的情况下发送ajax请求，
若在后台没有指定响应类型，响应成功的回调函数接收到的值是 json 对象还是一个字符串？

需不需要同时指定？