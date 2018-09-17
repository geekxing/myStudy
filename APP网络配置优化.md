# APP网络配置优化
## 性能
1. 应该关注在弱网环境下开发
2. 使用IPv6（减少RTT）
3. 使用ECN（减少丢包或重传以提升性能）
4. Multipath TCP（基于包而不是基于连接重新选择路径）
5. TCP fast open（发一个初始数据包避免三次握手）
6. QUIC（coming soon）

## 安全
#### TLS 1.3
￼![Alt tls1.3](https://github.com/geekxing/myStudy/blob/master/resources/tls1_3_1.jpeg?raw=true "tls1.3")-
#### 防止中间人攻击  
1.CA给服务器下发证书，并且要给一个叫certificate transparency log的同步证书，然后log给服务器下发奖章。服务器和客户端交互时，服务器把这些打包下发，客户端进行验证，如果是中间人攻击，因为伪造的证书没有再log处登记，是没有奖章的，客户端验证时就会拒绝交互。
![Alt 正规CA证书验证流程](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2010.jpeg?raw=true "正规CA证书验证流程")
![Alt 伪造CA证书验证流程](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2011.jpeg?raw=true "伪造CA证书验证流程")
#### API Choices  
Apple给出的几种写网络时可以使用的API选择
![Alt API Choices](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2012.jpeg?raw=true "API Choices")
Apple强烈推荐开发者和三方开发者放弃老旧的BSD sockets那套API，转而拥抱iOS12推出的全新网络框架Network.framework。  
其中，URLSession就是基于这个框架之上的专注于HTTP应用层的接口，当然它也提供StreamTask这种面向TCP传输层的接口。  
另外，从iOS12开始，URLSession支持HTTP2满足以下条件的对不同域名的连接可以复用：
<ul>
 <li>能匹配到IP地址</li>
 <li>不同的域名可以被同一个TLS证书覆盖</li>
</ul>
![Alt URLSession](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2013.jpeg?raw=true "URLSession")

## 网络优化方面
#### 网络延迟 Lantency
现在网络基建设施已经非常完善，所以我们从软件层面上来看，造成网络延迟的主要因素有两个：  
<ol>
<li>DNS解析</li>
<li>TCP连接的建立</li> 
</ol>
针对HTTP/1.1的优化方案很多，但都不完美。直到HTTP/2出现...
![Alt HTTP/2](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2014.jpeg?raw=true "HTTP/2")
HTTP/2的**TCP连接多路复用、设置请求优先级、压缩请求头**优化了网络资源，提高了带宽利用率。
![Alt 减少请求数据包的大小](https://github.com/geekxing/myStudy/blob/master/resources/Pasted%20Graphic%2015.jpeg?raw=true "减少请求数据包的大小")

#### 网络响应 Responsiveness
Qos, network service type, URLSession Adaptable Connectivity 代替 SCReachability       
#### 充分利用系统资源 System Resources
<ul>
<li>大文件传输使用后台sessions</li>
<li>根据实际情况看要不要对网络资源做Cache</li>
<li>URLSessions objects的创建时耗费资源的，应避免大量创建</li>
</ul>
 
  

