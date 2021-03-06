# 端口复用

---

## 端口复用

这个解决了我之前的一个疑问，即服务端在关闭后，马上重新启动服务器端会出错或者收不到连接。其实原因就是如果服务端是主动关闭方，主动方在发出最后一次ACK码以后会进入一个TIME_WAIT状态，这个时间就是2MSL，一般MSL是30S。这样之前的服务器进程就会一直占用这个端口，导致下一个进程会无法使用这个端口通信。**而端口复用就能解决这个问题！**

### setsockopt()函数：

**函数原型**

```cpp
int setsockopt(int sock, int level, int optname, const void *optval, socklen_t optlen);
```

**参数解释：**

- sock：将要被设置或者获取选项的套接字。-----**一般我们用监听的套接字**
- level：选项所在的协议层。
- optname：需要访问的选项名。
- optval：对于getsockopt()，指向返回选项值的缓冲。对于setsockopt()，指向包含新选项值的缓冲。
- optlen：对于getsockopt()，作为入口参数时，选项值的最大长度。作为出口参数时，选项值的实际长度。对于setsockopt()，现选项的长度。

**level指定控制套接字的层次.可以取三种值：**

1. SOL_SOCKET:通用套接字选项. ----**这个选项最常用！**

2. IPPROTO_IP:IP选项.

3. IPPROTO_TCP:TCP选项.　

**optname指定控制的方式(选项的名称，详细解释可以百度，这里讲两个：**　

1. SO_REUSEADDR：允许重用本地地址；
2. SO_REUSEPORT：允许重用本地接口。

*上述两者实现功能是一样的！*

**const void *optval**

传入一个地址，直接定义一个整型变量`int opt  = 1;`，并使用`&opt`传入地址，1即表示启用端口复用。

**optlen**

传入int型变量的大小，`sizeof(int)`。

以下举例最常用的用例：

```cpp
int opt = 1;
int re_val = setsockopt(lfd, SOL_SOCKET, SO_REUSEPORT, &opt, sizeof(int));
bind(lfd, (sockaddr *)&serv_addr, sizeof(sockaddr));
```

**注意调用位置，应该在套接字绑定之前设置！**

