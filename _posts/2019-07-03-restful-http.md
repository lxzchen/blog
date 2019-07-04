---
layout: post
title: RESTful API设计最佳实践（转译）
date: 2019-07-03
Author: 态度大师
categories: 
tags: [restful api, 设计模式]
comments: true
---

发布资源的URL改如何设计？名词使用复数或者单数？对同一个资源，改用多少URL来描述发布？对创建一个资源应该用怎样的HTTP方法（POST,PUT,GET,DELETE）描述？该如何处置可选参数？最好的描述分页和版本的方法又是什么？在设计RESTful API的时候，因为存在很多种可能性，故在设计的时候需要十分的谨慎小心。在这篇文章中，一起来看看设计RESTful API的最佳实践。

#### 每个资源，使用两个URL
一个用来表示集合，另一个用来表示特定的某个元素。
> /employees         #资源集合
> /employees/56    #资源特定元素
#### 用名词替代动词表示URL
以下这些写法，只会让URL看起来完全没有经过精心设计：
> /getAllEmployees
> /getAllExternalEmployees
> /createEmployee
> /updateEmployee

作为替代方案，参考以下设计。
#### 使用HTTP方法来操作URL
> GET /employees
> GET /employees?state=external
> POST /employees
> PUT /employees/56

URL用来指定需要操作的资源，HTTP方法用来指定对资源进行何种操作。通过使用HTTP的方法，GET，POST，PUT，DELETE，可以用来表示CRUD函数功能（Create，Read，Update，Delete）。
- Read：使用GET方法读取资源。GET请求不会改变资源状态，无任何副作用，并且是幂等的。GET方法是包含只读语义。因此，可以缓存响应结果。
- Create：使用POST方法创建新的资源。
- Update：使用PUT方法更新已存在的资源。
- Delete：使用DELETE方法删除已存在的资源。

通过2个URL和4个HTTP方法，可以等价于一系列方法操作集合，如下所示：

 / | POST(Create) | GET(Read) | PUT(Update) | DELETE(Delete)
----|----|----|----|----
/employees | 创建新雇员  | 获取所有雇员 | 批量更新雇员 | 删除所有雇员
/employees/56 | ...  | 获取编号56雇员详情 | 更新编号56雇员详情 | 删除编号56雇员

#### 在表示集合URL上，使用POST来创建新的资源
对于一个主从式交互过程，创建一个资源该如何展示？
![在collection-URL上，使用POST方法来创建资源](/images/restful-http/2728140-390a78b8c2f10e70.jpg)
1. 客户端通过collection-URL /employees发送POST方法请求，请求消息体包含新资源“Albert Stark”的相关属性。
2. 服务端为新的雇员创建ID，创建雇员相关信息并向客户端发送回响应。响应体包含访问新创建资源对应的URL路径，/employees/21。
#### 在表示特定袁术URL上，使用PUT来表示更新资源
![使用PUT方法，更新已存在的资源](/images/restful-http/2728140-afc5390eff0cb3b2.jpg)
客户端使用PUT方法，通过URL路径 /employees/21 发送请求。HTTP消息体包含更新的属性信息（ID编号21雇员的新名字）。
服务端更新 ID编号21雇员的新名字，并通过返回HTTP状态码200，确认此次更新成功。
#### 坚持使用复数名词
相比于如下RESTful URL：
> /employee
> /employee/21

如下RESTful URL更为合适：
> /employees
> /employees/21

事实上，这只能算一种尝试。但复数形式更加的普遍。此外，这更加直观。尤其是在使用collection-URL，用来表示GET /employees?state=external POST /employees PUT /employees/56，但最重要的是，避免混合使用复数和单数形式，容易让人困惑并且易于出错。
#### 使用“？”来拼接可选或者复杂的参数
不要使用如下URL表示方法：
> GET /employees
> GET /externalEmployees
> GET /internalEmployees
> GET /internalAndSeniorEmployees

保持URL简单和URL集合最小化。使用一个基础的URL代表一个资源，将复杂的、可选的参数迁移到URL后作为字符串参数。如下：
> GET /employees?state=internal&maturity=senior 

#### 使用HTTP状态码
RESTful Web服务应该通过适当的状态码返回给客户端的请求。
- 2xx – 成功 – 操作成功
- 4xx – 客户端错误 – 客户端操作错误 (比如客户端发送非法请求或者本身身份并未认证)
- 5xx – 服务端错误 – 后台出现错误 (比如后台无法处理请求)

参考[维基上HTTP状态码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)  。不论如何，如果利用所有的状态码，势必会让API的调用者感到困惑。所有将状态码的使用保持在一个较小的集合内，通常如下所示：

2XX:Success | 3XX:Redirect | 4XX:Client Error | 5XX:Server Error 
---- | ---- | ---- | ----
200 OK | 301 Moved Permanently | 400 Bad Request | 500 Internal Server Error
201 Created | 304 Not Modified | 401 Unauthorized | .
. | . | 403 Forbidden | .
. | . | 404 Not Found | .

#### 提供有效的错误码或错误提示
除了要有适当的状态码，还需要在HTTP响应消息体中提供有用并稍显啰嗦的关于错误信息的描述。
请求：
> GET /employees?state=super
响应：
> // 400 Bad Request
{
"message": "You submitted an invalid state. Valid state values are 'internal' or 'external'",
"errorCode": 352,
"additionalInformation" : "http://www.domain.com/rest/errorcode/352"
}

#### 使用驼峰规则表示属性名称
对属性定义使用驼峰规则。
> { "yearOfBirth": 1982 }

不用使用下划线(year_of_birth) 或者全大写字母开头。通常来说，客户端通常使用JavaScript来解析RESTful web服务返回的响应信息。常见于，客户端通常将JSON转换成JavaScript对象，比如：
> var person=JSON.parse(response)

因此，使用JavaScript的约定是的JavaScript更容易阅读和习惯性表达。
比较如下表达方式：
> person.year_of_birth  //violates JavaScript convention
> person.YearOfBirth    //suggests constructor method

和
> person.yearOfBirth    //nice!

#### 总是强制在URL中包含版本号
在发布RESTful API的时候，需要带上版本号。强制将版本号发布到URL上，这样API能更加容易发生演进。将版本号加入API，调用的客户端可以根据他们自身情况有条理的对现有API迁移和过度。客户端不会因为突然出现新的行为发生改变的API有太多的困惑。
使用“v”开头，并跟上数字版本号作为标记。
> /v1/employees

不需要使用小版本号（“v1.2”），因为API的改变不会非常频繁。相比较于API和文档语义，作为API的实现可以经常性的改进。
#### 提供分页查询支持
一次将所有资源从数据库返回绝对不是一个好的注意。所以，需要提供分页机制。一般使用参数 offset 和 limit，向数据库提供过滤。
> /employees?offset=30&limit=15       #returns the employees 30 to 45

需要对客户端忽略的参数提供默认值，防止返回全部的查询资源结果。通常如 offset=0 和 limit=10 是比较适当的。假如后台检索十分昂贵，则需要增加更多的参数限制。
> /employees       #returns the employees 0 to 10

此外，在使用分页时，需要返回客户端资源总数，
请求：
> GET /employees

响应：
> {
> "offset": 0,
> "limit": 10,
> "total": 3465,
> "employees": [
>   //...
>  ]
> }

#### 对非资源型响应使用动词表达
有时候，一个API的响应并不包括资源，如下：
> GET /translate?from=de_DE&to=en_US&text=Hallo
> GET /calculate?para2=23&para2=432

这种情况下，API不返回任何资源。相反，需要执行相关操作和返回结果。因为，在URL设计上，需要使用动词用来替代名词，用以区分非资源响应和资源相关的响应。
#### 考虑特定资源和跨域资源查找
提供对资源指定类型的查找是方便的。只要使用一致的collection-URL，然后加上查找关键词即可。
> GET /employees?query=Paul
加入需要提供一个所有资源的查询，需要一个稍微不同的方法。想想上面提到的关于非资源类URL设计，使用动词替代名称。因此，设计的查询URL可参考如下：
> GET /search?query=Paul   //returns employees, customers, suppliers etc.

#### 通过API，提供其他链接
理想上来说，可以不需要客户端动态构造URL，来访问REST API，可以考虑下面一个例子。
一个客户端想访问一个雇员的工资清单。此外，他还知道通过增加关键字“salaryStatements”到雇员URL（如：/employees/21/salaryStatements）访问工资清单。这种字符串串联非常容易出错，并非很难维护。但假如改变工资清单的访问方式（如使用新的“salary-statements”或者“paySlips”），则所有客户端都无法访问。
最好的方式是在响应中提供相关链接，比如：
请求：
> GET /employees/

响应：
> //...
   {
      "id":1,
      "name":"Paul",
      "links": [
         {
            "rel": "salary",
            "href": "/employees/1/salaryStatements"
         }
      ]
   },
//...

这样，假如客户端依赖返回的链接获取工资清单，即使改变API，也不会出现无法访问的问题，因为客户端总是可以获取一个合法的URL（即使一直在改变URL）。另外一个好处是，服务端的API变的更加具有自我描述性，需要更少的文档。
当在分页使用中，可以提供下一页或者上一页的链接。仅仅提供链接和相关的 offset 和 limit 参数。
> GET /employees?offset=20&limit=10

> {
  "offset": 20,
  "limit": 10,
  "total": 3465,
  "employees": [
    //...
  ],
  "links": [
     {
        "rel": "nextPage",
        "href": "/employees?offset=30&limit=10"
     },
     {
        "rel": "previousPage",
        "href": "/employees?offset=10&limit=10"
     }
  ]
}

 

以上文章的 [原文地址](https://blog.philipphauer.de/restful-api-design-best-practices/)  ，可以看原文如何写。

PS：第一次翻译，翻译的很局促，大家见谅。有表达不合适的，大家提，我修改。
