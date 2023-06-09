# myframe's websocket/http server module

# 依赖
- myframe框架 https://github.com/lkpworkspace/myframe
    - myframe框架的编译安装可以参考myframe的README
- 模块通信协议库 https://github.com/lkpworkspace/proto
    - proto模块的编译安装可以参考proto的README
- websocketpp
    ```sh
    sudo apt-get install libwebsocketpp-dev
    ```

# 模块安装
```sh
git clone https://github.com/lkpworkspace/web_service.git
cd web_service && mkdir build && cd build
cmake .. && make install
```

# 模块配置
模块默认安装在~/myframe/下,可以通过修改配置文件来配置该模块  
路径在~/myframe/service
```json
{
    "type":"library",
    "lib":"libweb_service.so",
    "actor":{
        "web_service":[
            {
                "instance_name":"web_service",
                "instance_params":"",
                "instance_config":{
                    "server_ca":"",
                    "server_key":"",
                    "server_port":"80",
                    "dst_addr":""
                }
            }
        ]
    }
}
```
- 模块名：actor.web_service.web_service
    - 模块名由3部分组成 模块类型.模块名.模块实例名
- server_ca/server_key: 这两个选项可以不用配置
- server_port：设置服务器监听端口号
- dst_addr: 配置服务器收到消息发往目的模块名称

# 模块通信
- 模块间通信使用protobuf进行通信，消息格式见 https://github.com/lkpworkspace/proto/blob/main/proto/http.proto

- 服务器收到消息会将消息转换成HttpReq格式发送给接收模块,接收模块收到消息经过处理后，可以将消息包装成HttpResp格式发送出去
    ```c++
    // 接收模块接收处理消息
    auto http_req = msg->GetAnyData<std::shared_ptr<myframe::HttpReq>>();
    // TODO: 处理消息
    // ...
    // 回复消息
    auto resp = std::make_shared<myframe::HttpResp>();
    resp->set_code(code);
    resp->set_body(body);
    auto resp_msg = std::make_shared<myframe::Msg>();
    resp_msg->SetAnyData(resp);
    auto mailbox = GetMailbox();
    mailbox->Send(msg->GetSrc(), resp_msg);
    ```
