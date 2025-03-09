---
layout: default
title: Python 教學
---

# Python 教學

歡迎來到 Python 教學網站！這裡提供了一系列 Python 程式設計的教學資源。

## 目錄

{% for file in site.pages %}
  {% if file.path contains 'Python/' %}
* [{{ file.title }}]({{ file.path | remove: '.md' }})
  {% endif %}
{% endfor %}

## 關於

這個網站包含了從基礎到進階的 Python 程式設計教學內容。每個主題都有詳細的說明和範例，適合初學者學習。

[查看 GitHub 專案](https://github.com/Cheng511/python_teach) 