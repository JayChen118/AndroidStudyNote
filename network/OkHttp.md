重要的类

1 Route 封装了连接服务器所需要的地址信息。
2 Platform 针对不同平台反射不同的class。
3 Connection 实际的连接类，包含Http连接的socket跟stream。被Http client调用。程序可以通过它监控HTTP请求的状态。
4 ConnectionPool 连接池，管理Connection，可以设置的选项有：Connection是否保持在池中、存活数、存活时间。
5 Request HTTP请求的实体，具有不可变性，使用Builder模式创建。
6 Response HTTP响应的实体，该类的实例不具备不可变性，响应的body是一次性的值，只能被消费一次。其它属性是不可变的。
7 Call 是已经做好被执行准备的请求，可以取消，它代表了一对请求和响应，不能被执行两次。
8 Dispatcher 代表用于执行异步请求的策略。使用ExecutorService在内部运行，如果要使用自己的执行器，则应该确保能同时执行配置中最大数目的请求。
9 HttpEngine 处理HTTP请求及响应。
10 Internal 用于提升内部的API到实现层。
11 Cache 缓存HTTP与HTTPS的响应到文件系统，达到重用及节省带宽的目的。
强制使用网络connection.addRequestProperty("Cache-Control", "no-cache")
或让服务器检查缓存是否过期connection.addRequestProperty("Cache-Control", "max-age=0");
强制使用缓存connection.addRequestProperty("Cache-Control", "only-if-cached");
或者设定容许缓存时间connection.addRequestProperty("Cache-Control", "max-stale=" + maxStale);

