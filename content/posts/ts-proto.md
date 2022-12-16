---
title: "使用vue3+typescript实现与后端的grpc通信"
date: 2022-12-16T09:30:27+08:00
draft: true
categories: ["ts","grpc","vue3"]
---
使用vue3+typescript实现grpc, makefile:

    NODE_BIN_PATH=./node_modules/.bin
    OUT_DIR=./src/proto

    ifeq ($(GOHOSTOS), windows)
        #the `find.exe` is different from `find` in bash/shell.
        #to see https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/find.
        #changed to use git-bash.exe to run find cli or other cli friendly, caused of every developer has a Git.
        Git_Bash= $(subst cmd\,bin\bash.exe,$(dir $(shell where git)))
        INTERNAL_PROTO_FILES=$(shell $(Git_Bash) -c "find app -name *.proto")
        API_PROTO_FILES=$(shell $(Git_Bash) -c 'find proto -path proto/third_party -prune -o -name "*.proto" | grep -v "^proto/third_party"')
    else
        INTERNAL_PROTO_FILES=$(shell find app -name *.proto)
        API_PROTO_FILES=$(shell find proto -path proto/third_party -prune -o -name "*.proto" | grep -v "^proto/third_party")
    endif


    1.grpc-web: 
        .PHONY: grpc-web
        grpc-web:
            protoc \
                --proto_path=./proto \
                --proto_path=./proto \
                --plugin=protoc-gen-ts=$(NODE_BIN_PATH)/protoc-gen-ts \
                --js_out="import_style=commonjs,binary:${OUT_DIR}" \
                --ts_out="service=grpc-web:${OUT_DIR}" \
                $(API_PROTO_FILES)

    2.ts-protoc-gen,直接生成ts，js文件，import有问题:
        .PHONY: grpc 
        #ts import问题
        grpc:
            ${NODE_BIN_PATH}/grpc_tools_node_protoc \
                --proto_path=./proto \
                --proto_path=./proto/third_party \
                --plugin=protoc-gen-ts=$(NODE_BIN_PATH)/protoc-gen-ts \
                --js_out="import_style=commonjs,binary:${OUT_DIR}" \
                --ts_out="service=grpc-node,mode=grpc-js:${OUT_DIR}" \
                --grpc_out=grpc_js:./proto \
                $(API_PROTO_FILES)

        .PHONY: grpc1
        #ts import问题
        # error protoc-gen-js: program not found or is not executable
        # 解决方法(protobuf降级)：https://stackoverflow.com/questions/72572040/protoc-gen-js-program-not-found-or-is-not-executable
        # brew安装指定版本protobuf：https://www.wandouip.com/t5i195895/
        grpc1:
            protoc \
                --proto_path=./proto \
                --proto_path=./proto/third_party \
                --plugin=protoc-gen-ts=$(NODE_BIN_PATH)/protoc-gen-ts \
                --plugin=protoc-gen-grpc=${NODE_BIN_PATH}/grpc_tools_node_protoc_plugin \
                --js_out=import_style=commonjs,binary:${OUT_DIR} \
                --ts_out=service=grpc-node,mode=grpc-js:${OUT_DIR} \
                --grpc_out=grpc_js:${OUT_DIR} \
                $(API_PROTO_FILES)
    3.grpc-proto-loaders，解决ts-protoc-gen的import问题: 
        .PHONY: loader 
        loader:
            $(NODE_BIN_PATH)/proto-loader-gen-types --longs=String --enums=String --defaults --oneofs --grpcLib=@grpc/grpc-js --outDir=proto/ $(API_PROTO_FILES)
    4.grpc-ts,生成ts和client:
        .PHONY: grpc-ts
        grpc-ts:
                $(NODE_BIN_PATH)/protoc \
                --proto_path=./proto \
                --proto_path=./proto/third_party \
                --ts_out=${OUT_DIR} \
                $(API_PROTO_FILES)
        


问题

    1.proto_path_js 找不到该文件或指令  
        降级protoc;  

    2.import 三方包比如google/protobuf/timestamp，生成的ts文件import不到  
        使用grpc-proto-loader ,google三方包只包含google/protubuf,不包含google/api;
        使用protobuf.js-cli 多层嵌套的子命名空间不到出要自己修改,感觉难用
        todo 去掉这部分依赖？对于前端来说这些是不需要的

    3.浏览器情况下，生成的ts文件找不到get，set方法(grpc-web生成的ts文件)
        疑似vite的问题？
        grpc-web的问题 https://github.com/grpc/grpc-web/issues/1242

    4.require/exports is not defined
        vite不兼容这种写法了
        import { viteCommonjs } from "@originjs/vite-plugin-commonjs";

    5.envoy yaml文件怎么写（测试环境配置，仅供参考）
        
       admin:
        access_log_path: /tmp/admin_access.log
        address:
            socket_address: { address: 0.0.0.0, port_value: 10001 } //不重要？

        static_resources:
        listeners:
            - name: listener_0
            address:
                socket_address: { address: 0.0.0.0, port_value: 10000 } //dock儿对外端口
            filter_chains:
                - filters:
                - name: envoy.filters.network.http_connection_manager
                    typed_config:
                    "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
                    codec_type: auto
                    stat_prefix: ingress_http
                    route_config:
                        name: xxx.grpc.v1 //你的服务名
                        virtual_hosts:
                        - name: 192.168.20.9 //服务的ip
                            domains: ["*"]
                            routes:
                            - match: { prefix: "/" }
                                route:
                                cluster: xxx.grpc.v1 //你的服务名
                                timeout: 0s
                        
                            cors:
                            allow_origin_string_match:
                                - prefix: "*"
                            allow_methods: GET, PUT, DELETE, POST, OPTIONS
                            allow_headers: keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,custom-header-1,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout
                            max_age: "1728000"
                            expose_headers: custom-header-1,grpc-status,grpc-message
                    http_filters:
                        - name: envoy.filters.http.grpc_web
                        typed_config:
                            "@type": type.googleapis.com/envoy.extensions.filters.http.grpc_web.v3.GrpcWeb
                        - name: envoy.filters.http.cors
                        typed_config:
                            "@type": type.googleapis.com/envoy.extensions.filters.http.cors.v3.Cors
                        - name: envoy.filters.http.router
                        typed_config:
                            "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router
            clusters:
                - name: xxx.grpc.v1 //你的服务名
                connect_timeout: 0.25s
                type: logical_dns
                http2_protocol_options: {}
                lb_policy: round_robin
                load_assignment:
                    cluster_name: cluster_0
                    endpoints:
                    - lb_endpoints:
                        - endpoint:
                            address:
                            socket_address:
                                address: 192.168.20.9 //服务的ip
                                port_value: 9101 //服务端口

最终方案  
    1.使用grpc-ts生产service.ts和service.client.ts  
    2.使用@protobuf-ts/grpcweb-transport让web使用fetch来通讯  

        import * as pb from "../proto/grpc-ts/staff.client";
        import {GrpcWebFetchTransport} from '@protobuf-ts/grpcweb-transport';

        const transport = new GrpcWebFetchTransport({
            baseUrl: "http://localhost:10000"
        });

        const client = new pb.StaffClient(transport);

        export { client };  
  
3.使用envoy当代理
       

相关资料

&ensp; [grpc浏览器文章](https://grpc.io/blog/state-of-grpc-web/#f17)  
&ensp; [降级google-protobuf](https://stackoverflow.com/questions/72572040/protoc-gen-js-program-not-found-or-is-not-executable)  
&ensp; [使用grpc-proto-loader](https://github.com/grpc/grpc/issues/13010)    
&ensp; npm:  [npm中文网](https://www.npmjs.cn/getting-started/what-is-npm/)  
&ensp; protobuf.sj [link](http://febeacon.com/protobuf_docs_zh_cn/routes/performance.html)  
&ensp; [grpc官网,node分支](https://grpc.io/docs/languages/node/)  
&ensp; [examples](https://github.com/badsyntax/grpc-js-typescript)
&ensp; [envoy配置参考](https://icloudnative.io/envoy-handbook/docs/gettingstarted/quick-start/)
