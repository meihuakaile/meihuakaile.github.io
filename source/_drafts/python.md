---
title: 'python'
date: "2018/11/1"
tags: [python]
categories: ['python']
copyright: true
---
###
`Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-ybji6729/mysqlclient/`
解决：`sudo python3 -m pip install --upgrade setuptools`

### 指定python版本安装
`sudo python3 -m pip install 安装包`； eg：`sudo python3 -m pip install mysql`
###
python3.5 安装mysql`sudo apt-get install python3.5-dev libmysqlclient-dev`