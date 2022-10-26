---
title: "gitlab ci/cd"
date: 2022-10-11T16:05:27+08:00
draft: false
categories: ["ci/cd","devops"]
---

1. [官方文档](https://docs.gitlab.cn/jh/ci/yaml/)
   
2. 关于大仓模式微服务的ci/cd构建
   1. cd应该针对master分支，也就是主分支代码提交时(发版时)才触发自动构建。
   2. 我们通常只想构建、测试和部署应用程序的那些已经更改的服务ーー而不是将所有服务放在一起，因为这可能非常耗时。使用only/change子句来实现。rule/changes应当是更好的实现。
   3. 根据gitlab官方的cicd实现，根目录的yml去include ci文件夹下的对应服务的yml文件，来实现一个大仓下多个服务的cicd。
    4. todo 如何去发布到不同服务器？
       1. 目前选用ansible

3. 踩坑：
   1. jobs config should contain at least one visible job
      1. https://gitlab.com/gitlab-org/gitlab/-/issues/341693
      2. 疑似已修复，gitlab15.2
   2. 请注意，GitLab Runner 的默认 pull policy 为 always，这意味着即使本地副本可用，runner 也会尝试从 GitLab 容器镜像库中提取 Docker 镜像。如果您更喜欢仅使用本地可用的 Docker 镜像，则可以在离线环境中将 GitLab Runner pull_policy 设置为 if-not-present。但是，如果不在离线环境中，我们建议将拉取策略设置保持为 always，因为这样可以在 CI/CD 流水线中使用更新的扫描程序。
      1. runner默认使用always策略，无法通过配置文件修改
   3.  protoc-gen-go: error:inconsistent package import paths
       1.  protoc-gen-go不存在报的这个错误，神秘
   4.  每一个构建都是隔离的，build protos这种指令应当放到before_script里让每个stage都触发生成

4. 技巧
   1. include template可以直接引用gitlab官方预制的一些模板，包括代码检查之类的。[链接](https://jihulab.com/gitlab-cn/gitlab/-/tree/master/lib/gitlab/ci/templates)
   2. gitlab提供了一批预定义变量. [链接](https://docs.gitlab.cn/jh/ci/variables/predefined_variables.html)

5. 命令
   ```
      dependencies: 通过提供要从中获取产物的作业列表，来限制将哪些产物传递给特定作业。也就是说在 deploy阶段dependencies指定build阶段的job可以取到build打出来的包。
      tags:用于选择 runner 的标签列表（选取运行对象）
      needs：在 stage 顺序之前执行的作业（发动效果的cost）
      extends：此作业继承自的配置条目（有继承就可以抽象一些公用的方法出来了）
      artifacts：使用 artifacts 指定在作业 succeeds, fails, 或 always 时附加到作业的文件和目录列表。(根据artfacts在build，deploy两个阶段传递打出来的包)
   ```

6. 参考文档  
[微服务架构下CI/CD如何落地](https://www.upyun.com/tech/article/602/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E4%B8%8B%20CI%2FCD%20%E5%A6%82%E4%BD%95%E8%90%BD%E5%9C%B0.html)    
[大仓模式及其在gitlab ci/cd上的应用](https://medium.com/swlh/on-monorepos-and-the-deployment-with-gitlab-ci-cd-bc080cfc6dce)  
[gitlab自己的ci/cd yml文件](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/.gitlab-ci.yml)