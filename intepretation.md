# UML类图
```mermaid
graph LR
A[Handlers]-.-B[std::vector&#60std::pair&#60std::unique_ptr&#60detail::MatcherBase>, Handler>>]
A1[Handler]-.-B1[std::function&#60void&#40const Request &, Response &&#41>]
A1-5[socket_t]-.-B1-5[int]
A2[SocketOptions]-.-B2[std::function&#60void&#40socket_t sock&#41&#62]
```


```mermaid
classDiagram

class Request {
    <<>>
}

class Response{
    <<>>
}

class MatcherBase {
    <<Abstract>>
}

class PathParamsMatcher {
    <<>>
}

MatcherBase <|-- PathParamsMatcher

class TaskQueue {
    <<Abstract>>
}

class ThreadPool {
    <<>>
}

TaskQueue <|-- ThreadPool

class scope_exit {
    <>
}

class Server {
    Handlers get_handlers_;
    std::function&#60TaskQueue *&#40void&#41&#62 new_task_queue; %% 用于返回一个线程池ThreadPool对象
    listen(std::string, int port, int socket_flags) bool
    Get(const std::string &pattern, Handler handler) Server
    bind_to_port(const std::string &host, int port, int socket_flags = 0) bool %% 把ip地址和端口绑定给一个套接字
    bind_internal(const std::string &host, int port, int socket_flags) bool
    SocketOptions socket_options_;
}

```

# 流程图

```mermaid
graph TB

subgraph S[void default_socket_options&#40socket_t sock&#41]
    S0[int yes=1]-->S1
    S1[setsockopt&#40sock, 1, 15, reinterpret_cast&#60const void *&#62&#40&yes&#41, sizeof&#40&yes&#41&#41]
    S1-.->S11[用于设置套接字选项的函数<br>允许在创建的套接字上设置一些参数,用于控制套接字的行为和属性<br>1:SOL_SOCKET,表示设置的是套接字级别的选项<br>15:SO_REUSEPORT,允许地址重用,常用于服务端套接字,防止地址已经被使用的错误]
end
O(开始)-->A
A[Server srv]-.->A1[给new_task_queue分配一个function对象,预创建线程池]
A-->B[srv.Get&#40&#41]
B-.->B1
subgraph B1[给get_handlers_添加元素,其中元素的函数对象是如下的lambda表达式]
    B11[&#91&#93&#40Request&, Response& res&#41&#123res.set_content&#40&#34hello world&#34,&#34text&#47plain&#34&#41&#59&#125]
end
B-->C[svr.listen&#40&#34localhost&#34, 8080&#41]
C-.->C1
subgraph C1[listen process]
    C11[Server::bind_to_port&#40&#34localhost&#34, 8080, 0&#41]

    C11-.->C111
    subgraph C111[bind_to_port process]
        C112[bind_internal&#40&#34localhost&#34, 8080, 0&#41&#41]
        C112-.->C1111
        subgraph C1111[bind_internal process]
            C1112[create_server_socket&#40&#34localhost&#34, 8080, 0, &#41]
        end
    end
    C11-->C12[Server::listen_internal&#40&#41]
end

```