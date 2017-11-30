# Keep Alive

在HTTP早期，每个HTTP请求都要求打开一个TCP socket连接，并且使用一次之后就断开这个TCP连接。

使用keep-alive可以改善这种状态，即在一次TCP连接中可以持续发送多份数据而不会断开连接。通过使用keep-alive机制，可以减少TCP连接建立次数，也意味着可以减少[TIME_WAIT](/Network/TCP/time_wait.md)状态连接，以此提高性能和提高HTTPd服务器的吞吐率(更少的TCP连接意味着更少的系统内核调用,socket的accept()和close()调用)。
