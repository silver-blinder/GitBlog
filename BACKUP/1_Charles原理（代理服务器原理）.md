# [Charles原理（代理服务器原理）](https://github.com/silver-blinder/GitBlog_/issues/1)

### 浏览器设置代理 —— 建立桥梁

假如在浏览器配置响应的代理如下：（SwitchyOmega）

```
127.0.0.1:5388
```

浏览器发出请求 —> 发送请求到你本机上的 `127.0.0.1` 的 `5388` 端口

（浏览器的网络请求会被代理设置（如 SwitchyOmega）重定向到 Charles 所监听的本地端口。）

### Charles 接收到浏览器的请求

Charles 监听着端口 5388，浏览器把“我想访问 www.example.com 的请求”送到了 127.0.0.1:5388，Charles 就接到了。（限制端口号：网络流量统一监控）

如果启用了 **Rewrite**，此时会对**请求的 URL、Header、Body** 进行修改（修改成localhost）

### Charles 转发请求

Charles 接到浏览器的请求后，就会自己用你电脑的真实网卡（网络接zz s口 wifi）来发送这个请求。

Charles 实际是：

- 用这个内网 IP 作为源地址，
- 向公司服务器或者互联网发请求。

### 请求服务器

返回结果（网页内容、数据等）发回这个 IP，而这个 IP 就是你的电脑（内网）

### Charles 收到服务器返回，再交给浏览器

Charles 拿到响应后，再把它交给浏览器。

这时候，浏览器才像接到了你要的文件，进行展示