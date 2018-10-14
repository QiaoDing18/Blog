title: Ajax
tags:
  - 前端
categories:
  - web
author: 乔丁
date: 2018-04-27 14:31:00
---
## Ajax原理及实现
原理：通过xhr对象向服务器发出异步请求，从服务器获取数据，实现页面的局部刷新
实现步骤：
1、创建xhr对象
2、设置服务器的连接信息xhr.opn(请求方式，url，异步还是同步)
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
3、注册回调函数。onreadystatechange是发生变化执行xhr.onreadystatechange=callback;
4、发送数据xhr.send();
5、获取相应数据 JSON.parse(xhr.responseText);

Ajax的工作原理相当于在用户和服务器之间加了—个中间层(AJAX引擎),使用户操作与服务器响应异步化。并不是所有的用户请求都提交给服务器,像—些数据验证和数据处理等都交给Ajax引擎自己来做, 只有确定需要从服务器读取新数据时再由Ajax引擎代为向服务器提交请求。

## Ajax和jsonp
首先在客户端注册一个callback, 然后把callback的名字传给服务器。
此时，服务器先生成 json 数据。然后将这个callback参数作为函数名来包裹住JSON数据，返回给客户端。
客户端浏览器，动态执行回调函数，将返回的数据作为参数，传入到了客户端预先定义好的 callback 函数里.