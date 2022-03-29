1、服务端主动关闭连接的缺点之一是会多占用服务器资源。服务端主动关闭连接之后会进入
`TIME_WAIT` 状态，在一段时间之内 hold 住一些内核资源。如果并发访问量很高，这会影响服务端的处
理能力。这似乎暗示我们应该把协议设计为客户端主动关闭，让 `TIME_WAIT` 状态分散到多台客户机器
上，化整为零。

2、客户端赖着不走怎么办？会不会造成拒绝服务攻击？或许有一个二者结合的方
案：客户端在收到响应之后就应该主动关闭，这样把 `TIME_WAIT` 留在客户端。服务端有一个定时器，
如果客户端若干秒钟之内没有主动断开，就踢掉它。这样善意的客户端会把 `TIME_WAIT` 留给自己，
buggy 的客户端会把 `TIME_WAIT` 留给服务端。或者干脆使用长连接协议，这样避免频繁创建销毁连接