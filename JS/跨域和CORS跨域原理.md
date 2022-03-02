## 跨域和CORS跨域原理

### 跨域

跨域的产生来源于现代浏览器所通用的**同源策略**，所谓同源策略，是指只有在地址的：

1. 协议名
2. 域名
3. 端口号

一致的情况下，才允许访问相同的 cookie、localstorage，以及访问页面的 DOM 或是发送 Ajax 请求。**若是在不同源的情况下访问，就成为跨域**。

首先，**跨域只存在于浏览器端，因为我们知道浏览器的形态是很开放的，所以我们需要对它有所限制**。

所以同源策略它的产生主要是为了保证用户信息的安全，防止恶意的网站窃取数据。分为两种：**Ajax同源策略**与**DOM同源策略**。

### CORS 跨域原理

跨域资源共享（CORS）是一种机制，是 W3C 标准。它允许浏览器向跨域服务器，发出 XMLHTTPRequest 或 Fetch 请求。并且整个 CORS 通信过程都是浏览器自动完成的，不需要用于参与。

而使用这种跨域资源共享的前提是，浏览器必须支持这个功能，并且服务器端也必须同意这种"跨域"请求。因此实现CORS的关键是服务器需要服务器。通常是有以下几个配置：

- Access-Control-Allow-Origin
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers
- Access-Control-Allow-Credentials
- Access-Control-Max-Age

过程分析：

- **浏览器先根据同源策略对前端页面和后台交互地址做匹配**，若同源，则直接发送数据请求；若不同源，则发送跨域请求。
  
- 服务器收到浏览器跨域请求后，根据自身配置返回对应文件头。若未配置过任何允许跨域，则文件头里不包含 **Access-Control-Allow-origin** 字段，若配置过域名，则返回 **Access-Control-Allow-origin + 对应配置规则里的域名的方式**。
  
- 浏览器根据接受到的响应头里的 Access-Control-Allow-origin 字段做匹配，若无该字段，说明不允许跨域，从而抛出一个错误；若有该字段，则对字段内容和当前域名做比对，如果同源，则说明可以跨域，浏览器接受该响应；若不同源，则说明该域名不可跨域，浏览器不接受该响应，并抛出一个错误。

另外在 CORS 中有**简单请求**和**非简单请求**。简单请求是不会触发 CORS 的预检请求的，而非简单请求则会触发预检请求。

预检请求要求必须首先使用 OPTIONS 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。**"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响**。

#### CORS 的哪些是简单请求

简单请求不会触发CORS的预检请求，若请求满足所有下述条件，则该请求可视为“简单请求”：

简单回答：

- 只能使用 GET、HEAD、POST 方法。使用 POST 方法向服务器发送数据时，Content-Type 只能使用 application/x-www-form-urlencoded、multipart/form-data或text/plain 编码格式。
- 请求时不能使用自定义的 HTTP Headers。

#### CORS的预检请求具体是怎样的

若是跨域的非简单请求的话，浏览器会首先向服务器发送一个预检请求，以获知服务器是否允许该实际请求。

- 浏览器给服务器发送一个OPTIONS方法的请求，该请求会携带下面两个首部字段
- Access-Control-Request-Method：实际请求要用到的方法
- Access-Control-Request-Headers: 实际请求会携带哪些首部字段


若是服务器接受后续请求，则这次预请求的响应体中会携带下面的一些字段
- Access-Control-Allow-Methods: 服务器允许使用的方法
- Access-Control-Allow-Origin: 服务器允许访问的域名
- Access-Control-Allow-Headers: 服务器允许的首部字段
- Access-Control-Max-Age: 该响应的有效时间(s),在有效时间内浏览器无需再为同一个请求发送预检请求

