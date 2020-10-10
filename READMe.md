# 同源与跨域
## 同源策略


> 部分内容摘自阮一峰网络日志http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
浏览器安全的基石是"同源政策"[（same-origin policy）](https://en.wikipedia.org/wiki/Same-origin_policy)

###  含义
1995年，同源政策由 Netscape 公司引入浏览器。目前，所有浏览器都实行这个政策。
最初，它的含义是指，A网页设置的 Cookie，B网页不能打开，除非这两个网页"同源"。所谓"同源"指的是"三个相同"。
* 协议相同
* 域名相同
* 端口相同

例：

当前页面url | 被请求页面url | 是否跨域 | 原因
:-----: | :-----: | :----: | :----:
`http://www.test.com/` | `http://www.test.com/index.html` | 否 | 同源（协议、域名、端口号相同）
`http://www.test.com/` | `https://www.test.com/index.html` |	跨域 |	协议不同（http/https）
`http://www.test.com/` | `http://www.baidu.com/` |	跨域 |	主域名不同（test/baidu）
`http://www.test.com/` | `http://blog.test.com/` |	跨域 |	子域名不同（www/blog）
`http://www.test.com:8080/` | `http://www.test.com:7001/` |	跨域 |	端口号不同（8080/7001）

### 目的

同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。

设想这样一种情况：A网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取A网站的 Cookie，会发生什么？

很显然，如果 Cookie 包含隐私（比如存款总额），这些信息就会泄漏。更可怕的是，Cookie 往往用来保存用户的登录状态，如果用户没有退出登录，其他网站就可以冒充用户，为所欲为。因为浏览器同时还规定，提交表单不受同源政策的限制。

由此可见，"同源政策"是必需的，否则 Cookie 可以共享，互联网就毫无安全可言了。

### 限制范围

随着互联网的发展，"同源政策"越来越严格。目前，如果非同源，共有三种行为受到限制。

1. Cookie、LocalStorage 和 IndexDB 无法读取。
2. DOM 无法获得。
3. AJAX 请求不能发送。

### 常见跨域报错
![image](https://github.com/HankBass/front-end-UI-comparison/raw/master/images/cross.png)

## 跨域解决方案
### JSONP
JSONP是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，老式浏览器全部支持，服务器改造非常小。
它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。
首先，网页动态插入`<script>`元素，由它向跨源网址发出请求。
```
function addScriptTag(src) {
  var script = document.createElement('script');
  script.setAttribute("type","text/javascript");
  script.src = src;
  document.body.appendChild(script);
}

window.onload = function () {
  addScriptTag('http://example.com/ip?callback=foo');
}

function foo(data) {
  console.log('Your public IP address is: ' + data.ip);
};
```
上面代码通过动态添加`<script>`元素，向服务器`example.com`发出请求。注意，该请求的查询字符串有一个`callback`参数，用来指定回调函数的名字，这对于JSONP是必需的。
服务器收到这个请求以后，会将数据放在回调函数的参数位置返回。
```
foo({
  "ip": "8.8.8.8"
});
```
由于`<script>`元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了`foo`函数，该函数就会立即调用。作为参数的`JSON`数据被视为`JavaScript`对象，而不是字符串，因此避免了使用`JSON.parse`的步骤。

### WebSocke
`WebSocke`t是一种通信协议，使用`ws://`（非加密）和`wss://`（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。
下面是一个例子，浏览器发出的`WebSocket`请求的头信息（摘自维基百科）。
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```
上面代码中，有一个字段是`Origin`，表示该请求的请求源（origin），即发自哪个域名。
正是因为有了`Origin`这个字段，所以`WebSocket`才没有实行同源政策。因为服务器可以根据这个字段，判断是否许可本次通信。如果该域名在白名单内，服务器就会做出如下回应。
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

原生WebSocket API使用起来不太方便,我们使用 Socket.io ,它很好地封装了 webSocket接口,提供了更简单,灵活的接口,也对不支持webSocket的浏览器提供了向下兼容。
```
//前端
<script>
    // 定义数据
    let Data = {
        username : 'jade',
        password : 123456
    }
    let socket = new WebSocket('ws://localhost:4000');
    socket.onopen = function() {
        socket.send(JSON.stringify(Data)); // 向服务器发送数据
    }
    socket.onmessage = function(e) {
        console.log(JSON.parse(e.data)); // 接收服务器返回的数据
    }
</script>

// 后端
let express = require('express');
let app = express();
let WebSocket = require('ws');
let wss =new WebSocket.Server({port:4000})


// 定义数据
let Data = {
    username : 'jade',
    password : 123456
}

wss.on('connection', function(ws) {
    ws.on('message',function(data) {
        console.log(JSON.parse(data))
        ws.send(JSON.stringify(Data))
    })
})

```

### postMessage

`HTML5`为了解决这个问题，引入了一个全新的`API`：跨文档通信 `API`（Cross-document messaging）。
这个API为`window`对象新增了一个`window.postMessage`方法，允许跨窗口通信，不论这两个窗口是否同源。
举例来说，父窗口`http://aaa.com`向子窗口`http://bbb.com`发消息，调用`postMessage`方法就可以了。
```
var popup = window.open('http://bbb.com', 'title');
popup.postMessage('Hello World!', 'http://bbb.com');
```

`postMessage`方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin），即"协议 + 域名 + 端口"。也可以设为*，表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似。
```
window.opener.postMessage('Nice to see you', 'http://aaa.com');
```
父窗口和子窗口都可以通过`message`事件，监听对方的消息。
```
window.addEventListener('message', function(e) {
  console.log(e.data);
},false)
```
`message`事件的事件对象`event`，提供以下三个属性。
* event.source：发送消息的窗口
* event.origin: 消息发向的网址
* event.data: 消息内容

下面的例子是，子窗口通过`event.source`属性引用父窗口，然后发送消息。
```
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  event.source.postMessage('Nice to see you!', '*');
}
```
`event.origin`属性可以过滤不是发给本窗口的消息。
```
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  if (event.origin !== 'http://aaa.com') return;
  if (event.data === 'Hello World') {
      event.source.postMessage('Hello', event.origin);
  } else {
    console.log(event.data);
  }
}
```
### CORS

`CORS`需要浏览器和后端同时支持。IE8和 IE9需要通过 `XDomainRequest`来实现
浏览器会自动进行 `CORS`通信,实现`CORS`通信的关键是后端。只要后端实现了`CORS`,实现了跨域。
服务端设置 `Access-Control-Allow-Origin` 就可以开启`CORS`。该属性表示哪些域名可以访问资源,如果设置通配符则表示所有网站都可以访问资源。
虽然设置`CORS`和前端没有什么关系,但是通过这种方式解决跨域问题的话,会在发送请求时出现两种情况,分别为简单请求和复杂请求。
#### 简单请求
只要同时满足以下两大条件，就属于简单请求。
```
（1) 请求方法是以下三种方法之一：

HEAD
GET
POST
（2）HTTP的头信息不超出以下几种字段：

Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
```

#### 复杂请求
不符合以上条件的请求就肯定是复杂请求了。复杂请求的CORS请求,会在正式通信之前,增加一次HTTP查询请,称为"预检"请求,该请求是option方法的 , 通过该请求来知道服务端是否允许跨域请求。
我们用 PUT 向后台请求时, 属于复杂请求,后台需如下配置:
```
// 允许哪个方法访问我
res.setHeader('Access-Control-Allow-Methods', 'PUT')
// 预检的存活时间
res.setHeader('Access-Control-Max-Age', 6)
// OPTIONS请求不做任何处理
if (req.method === 'OPTIONS') {
  res.end()
}
// 定义后台返回的内容
app.put('/getData', function(req, res) {
  console.log(req.headers)
  res.end('我不爱你')
})

```
接下来我们看下一个完整请求的例子,并且介绍下CORS请求相关的字段
```
//index.html
// 前端代码
<script>
    let xhr = new XMLHttpRequest();
    document.cookie = 'name=hw';
    xhr.withCredentials = true; //前端设置是否带 cookie
    xhr.open('PUT','http://localhost:4000/getData',true);
    xhr.setRequestHeader('name','hw');
    xhr.onreadystatechange = function() {
        if(xhr.readyState === 4) {
            if(xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
                console.log(JSON.parse(xhr.response));
                console.log(xhr.getResponseHeader('name'))
            }
        }
    }
    xhr.send();
</script>


// server.js
let express = require('express');
let app = express();
app.use(express.static(__dirname))
app.listen(3000)

// 后端代码
let express = require('express')
let app = express()
let whitList = ['http://localhost:3000'] //设置白名单
app.use(function(req, res, next) {
  let origin = req.headers.origin
  if (whitList.includes(origin)) {
    // 设置哪个源可以访问我
    res.setHeader('Access-Control-Allow-Origin', origin)
    // 允许携带哪个头访问我
    res.setHeader('Access-Control-Allow-Headers', 'name')
    // 允许哪个方法访问我 用逗号隔开的允许使用的 HTTP request methods 列表
    res.setHeader('Access-Control-Allow-Methods', 'PUT')
    // 允许携带cookie
    res.setHeader('Access-Control-Allow-Credentials', true)
    // 预检的存活时间 3600（单位为秒，超时时间为1小时）
    res.setHeader('Access-Control-Max-Age', 3600)
    // 允许返回的头
    res.setHeader('Access-Control-Expose-Headers', 'name')
    if (req.method === 'OPTIONS') {
      res.end() // OPTIONS请求不做任何处理
    }
  }
  next()
})
app.put('/getData', function(req, res) {
  let data = {
      username : 'zs',
      password : 123456
  }
  console.log(req.headers)
  res.setHeader('name', 'jw') //返回一个响应头，后台需设置
  res.end(JSON.stringify(data))
})
app.get('/getData', function(req, res) {
  console.log(req.headers)
  res.end('he')
})
app.listen(4000)

```

> 详情请参考阮一峰[文章](http://www.ruanyifeng.com/blog/2016/04/cors.html)
### Node中间件

同源策略是浏览器需要遵循的标准,而如果是服务器向服务器请求就无需遵循同源策略。
代理服务器,需要做以下几个步骤:
1. 接受客户端请求
2. 将请求转发给服务器
3. 拿到服务器响应数据
4. 将响应转发给客户端

![image](https://github.com/HankBass/front-end-UI-comparison/blob/master/images/node%20proxy.png?raw=true)

> Vue 在dev模式下，可以通过vue.config.js设置proxy来实现跨域，实际就是webpack启动了Node的服务用于代理

### Nginx
和Node中间件实现跨域的原理相似，都是基于服务器无需遵循同源策略

常用Nginx配置如下
```
server
{
    listen 3002;
    server_name localhost;
    location /web {
        proxy_pass http://localhost:3000;

        #   指定允许跨域的方法，*代表所有
        add_header Access-Control-Allow-Methods *;

        #   预检命令的缓存，如果不缓存每次会发送两次请求
        add_header Access-Control-Max-Age 3600;
        #   带cookie请求需要加上这个字段，并设置为true
        add_header Access-Control-Allow-Credentials true;

        #   表示允许这个域跨域调用（客户端发送请求的域名和端口）
        #   $http_origin动态获取请求客户端请求的域   不用*的原因是带cookie的请求不支持*号
        add_header Access-Control-Allow-Origin $http_origin;

        #   表示请求头的字段 动态获取
        add_header Access-Control-Allow-Headers
        $http_access_control_request_headers;

        #   OPTIONS预检命令，预检命令通过时才发送请求
        #   检查请求的类型是不是预检命令
        if ($request_method = OPTIONS){
            return 200;
        }
    }
}

```

### 总结
开发环境 | 生产环境
:-----: | :-----: |
cors | cors
proxy | nginx
> 日常开发中，常用的跨域方式一般是CORS和Nginx，但因为后台有时候不愿意额外增加工作量，故Nginx使用的频率更高


