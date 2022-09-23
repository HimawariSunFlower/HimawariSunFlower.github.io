---
title: "gorm 遇到的问题"
date: 2022-09-19T11:11:27+08:00
draft: true
categories: ["gorm"]
---


1. updates(&struct{}),值为0不会跟新，需要updates(map)或者save()或者update()
2. 