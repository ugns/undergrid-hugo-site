---
title: "{{ replace .Name "-" " " | title }}"
featured_image: images/technology.jpg
type: post
author: jtbouse
date: {{ .Date }}
publishdate: {{ now.Format "2006-01-02" }}
draft: true
summary: |
    Text about this post
tags:
    - tag 1
categories:
    - cat 1
---

## Post Header

CONTENT
