---
title: "vue3基础手法指北之新手村篇"
date: 2023-01-13T15:01:27+08:00
draft: false
categories: ["vue3"]
---

<!-- vscode-markdown-toc -->
* 1. [vue基础循环指北](#vue)
	* 1.1. [指令](#)
	* 1.2. [响应式 ref与reactive](#refreactive)
	* 1.3. [生命周期](#-1)
* 2. [第三方插件](#-1)
	* 2.1. [浏览器插件 vue.js devtools](#vue.jsdevtools)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

<!-- 
    command+shift+p 
    generate TOC for md
-->

-----
# vue3基础手法指北之新手村篇
-- 市面上大火的游戏，虽然还不理解该游戏的设定但总之先从新手村出发

前置官方新手教程 [传送门](https://cn.vuejs.org/tutorial/#step-1)  

##  1. <a name='vue'></a>vue基础循环指北
-- 村好剑
###  1.1. <a name=''></a>指令
-- 这是火1-火4，这是冰1-冰4，好了，你该练gcd循环了
1. v-bind(:) 在模板中响应式的绑定attribute。  
   1. 用处：可以用来动态改变style或其他值，比如配合v-for来设置select里option的key，value。 
   2. [技能详情](https://cn.vuejs.org/api/built-in-directives.html#v-bind)
2. v-model 在表单输入元素或组件上创建双向绑定。
   1. 用处：绑定后表单输入值就可以改变script定义的ref/reactive变量
   2. [技能详情](https://cn.vuejs.org/api/built-in-directives.html#v-model)
3. v-on(@) 给元素绑定事件监听器。 [技能详情](https://cn.vuejs.org/api/built-in-directives.html#v-on)
4. v-if v-else v-else-if v-for v-show
   1. 不推荐同时使用v-if和v-for
   2. v-show通过display来实现v-if的效果，性能更有优势
5. v-text v-html v-solt v-once v-memo v-cloak v-pre

###  1.2. <a name='refreactive'></a>响应式 ref与reactive
-- 你要的是这个金斧头还是这个银斧头呢？  

    import { ref,reactive } from 'vue'

    let state1 = reactive({ count: 0 })  
    let state2 = ref({ count: 0 })  

    //在ts中修改值
    state1.count++ 
    state2.value.count++

武器备注：  
&ensp; ref几乎是reactive的完全上位替代。除了可能存在的性能区别，想不到用reactive的理由。  
1. reactive对原始类型(string,number...)无效
2. reactive被赋值或解构失去响应性,而ref可以对value进行随意的赋值结构
```
    state1 = reactive({count:1}) //会是state1失去响应式
    state2.value = {count:1} //正常运行
```
3. ref的唯一缺点就是在script中需要多写一个.value，不过在template里是不需要的。
```
    <script setup lang="ts">
    ...
    state2.value.count=3
    </script>

    <template>
    {{state2.count}}
    </template>
```


###  1.3. <a name='-1'></a>生命周期
-- 咋，瓦鲁多

1. onMounted 组件完成初始渲染并创建 DOM 节点后运行代码
   1. 用来初始化界面，获取填充服务端数据之类的
2. onActivated() 注册一个回调函数，若组件实例是 <KeepAlive> 缓存树的一部分，当组件被插入到 DOM 中时调用。
   1. 如果界面是keepAlive的话，只有第一次打开会触发onMounted，如果要做每次打开都触发的事情，得用这个。
3. onUpdated()注册一个回调函数，在组件因为响应式状态变更而更新其 DOM 树之后调用。 
   1. 每次数据更新都有你
4. onUnmounted() 注册一个回调函数，在组件实例被卸载之后调用。
   1. gc是你吗？gc  

其他的生命周期还没怎么用过，怎么说也还在新手村。[官方指北](https://cn.vuejs.org/api/composition-api-lifecycle.html#onmounted)


##  2. <a name='-1'></a>第三方插件
-- 原来你游也有act

###  2.1. <a name='vue.jsdevtools'></a>浏览器插件 vue.js devtools  
谷歌应用商店安装，再给个页面权限就可以实时监控~~你和队友的伤害~~,组件和绑定的数据，甚至可以监控pinia等三方库的数据。对于开发来说非常方便。
