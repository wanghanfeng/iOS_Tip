>JavaScript Object Notation (JSON)
JS对象标记

JSON是一种文本格式结构化数据的序列化。

JSON有四种原始类型：
````
1.字符串（string）
2.数值（number）
3.布尔（boolean）
4.空（null）
````

两种结构化类型：
````
1.对象（object）
2.数组（array）
````

字符串（string）是以0个或更多的Unicode字符组成的序列。

对象（object）是以键值对（name/value）组成的无序集合，键名（name）必须是字符串类型，键值必须是字符串（string）、数值（number）、布尔（boolean）、空（null）、对象（object）或数组（array）中的一种。

数组（array）是0个或多个值（value）的有序集合。

JSON的设计目标是让JSON成为最小的，便携的，文本的，JavaScript的子集。

JSON 内容类型有如下几种：
>application/json
application/x-javascript
text/javascript
text/x-javascript
text/x-json

根据[RFC 4627](http://www.ietf.org/rfc/rfc4627.txt)文件，
The MIME media type for JSON text is application/json.
 Type name: application
Subtype name: json

所以认为：The MIME media type for JSON text is application/json. The default encoding is UTF-8。