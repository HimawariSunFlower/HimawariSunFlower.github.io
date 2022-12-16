---
title: "ent与图数据库"
date: 2022-10-28T16:05:27+08:00
draft: false
categories: ["ent","GDB"]
---

1. 参考资料
   1. [ent](https://entgo.io/zh/docs/getting-started)
   2. [什么是图数据库GDB]{https://help.aliyun.com/document_detail/102799.html}
   3. https://log.zvz.im/2020/06/21/go-ent-graph-based-orm/
   4. [GDB的优点](https://bbs.huaweicloud.com/blogs/265577?utm_source=zhihu&utm_medium=bbs-ex&utm_campaign=ei&utm_content=content)
   5. ent库本身不支持根据struct T创建数据库数据，但是可以根据template生成对应方法去打补丁实现。
      2. [docs](https://entgo.io/zh/docs/faq#how-to-create-an-entity-from-a-struct-t)
