---
title: "go-kratos 模仿beershop的微服务demo运行及遇到的问题"
date: 2022-09-05T15:08:57+08:00
draft: false
categories: ["go-kratos"]
---


1. 错误如果不定义为kratos的错误类型，直接输出errors.error，会输出私有错误
```
{
  "errors": {
    "internal": [
      "error"
    ]
  }
}
```
无法显示具体信息，必须按[规定的错误处理](https://go-kratos.dev/docs/component/errors)，定义生成pb文件之后包装一下
```
    v1.ErrorParamError("jwt token missing")
    v1.ErrorParamError(err.Error())
```

 <br/> 

2. 尝试docker部署微服务，docker build 报错
```
failed to solve with frontend dockerfile.v0: failed to create LLB definition:   
unexpected status code [manifests stable-slim]: 403 Forbidden
```
设置 docker Engine => features:{buildkit:false}

<br/>

3. docker 里是不能用127.0.0.1访问宿主机的数据库的，需要配置里配宿主机ip  
root只能localhost访问，需要创建新用户, % 表示任意ip都能访问

<br/>

4. dokerfile 里的make build执行的就是makefile里的build命令，需要改指向的cmd路径

<br/>

5. 可以用这个指令 docker run -it \<image:version\> bash  
去排查docker运行时的问题

<br/>

6. casbin中间件 取不到req的mehod
只有ctx和一个interface的req
Request Method  
```
func(handler middleware.Handler) middleware.Handler {
    return func(ctx context.Context, req interface{}) (interface{}, error) {
        tr, _ := transport.FromServerContext(ctx)
        htr := tr.(*http.Transport)//http是kratos transport里的子包
        fmt.Println(htr.Request().Method, htr.Request().URL.Path)
        return nil, nil
    }
},
```

<br/>

7. goland新建项目，goget依赖成功但是一直飘红，外部库也没有实际添加
  解决方法，goland添加goproxy
![这是图片](/img/2.png) 