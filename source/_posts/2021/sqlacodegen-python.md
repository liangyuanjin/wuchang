---
title: Sqlacodegen
date: 2021-03-29 18:14:55
tags:
  - Python
  - Mysql
categories: Python
description: Python 杂项
top_img: https://cdn.pixabay.com/photo/2021/01/05/06/40/boat-5889919_1280.png
cover: https://cdn.pixabay.com/photo/2021/01/19/17/59/mask-5932015_1280.png
---
## 背景

> 在日常开发中，有的是库和表已经存在，但是这时候如果想从库中导出 `Python` 的 orm `sqlalchemy` 的模型，可以使用 `sqlacodegen` 这个第三方库来实现

## 按表导出模型

```bash
sqlacodegen --outfile db.py --tables table1,table2 mysql+pymysql://root:passwd@127.0.0.1:3306/db_name
```

## 按库导出模型

```bash
sqlacodegen --outfile db.py mysql+pymysql://root:passwd@127.0.0.1:3306/db_name
```

## 导出详情如下

```python
# coding: utf-8
from sqlalchemy import Column, DateTime, Integer, Numeric, String, Text
from flask_sqlalchemy import SQLAlchemy


db = SQLAlchemy()


class Good(db.Model):
    __tablename__ = 'goods'

    create_time = db.Column(db.DateTime)
    update_time = db.Column(db.DateTime)
    delete_time = db.Column(db.DateTime)
    id = db.Column(db.Integer, primary_key=True)
    category_id = db.Column(db.Integer, nullable=False, index=True)
    goods_sn = db.Column(db.String(255), nullable=False, index=True)
    name = db.Column(db.String(255), nullable=False)
    brand_id = db.Column(db.Integer, nullable=False, index=True)
    goods_num = db.Column(db.Integer, nullable=False, index=True)
    keywords = db.Column(db.String(255), nullable=False)
    goods_brief = db.Column(db.String(255), nullable=False)
    goods_desc = db.Column(db.Text)
    is_on_sale = db.Column(db.Integer, nullable=False)
    sort_order = db.Column(db.Integer, nullable=False, index=True)
    is_delete = db.Column(db.Integer, nullable=False)
    attribute_category = db.Column(db.Integer, nullable=False, index=True)
    counter_price = db.Column(db.Numeric(10, 2), nullable=False)
    extra_price = db.Column(db.Numeric(10, 2), nullable=False)
    is_new = db.Column(db.Integer, nullable=False)
    goods_unit = db.Column(db.String(255), nullable=False)
    primary_pic_url = db.Column(db.String(255), nullable=False)
    list_pic_url = db.Column(db.String(255), nullable=False)
    retail_price = db.Column(db.Numeric(10, 2), nullable=False)
    sell_volume = db.Column(db.Integer, nullable=False)
    primary_product_id = db.Column(db.Integer, nullable=False)
    unit_price = db.Column(db.Numeric(10, 2), nullable=False)
    promotion_desc = db.Column(db.String(255), nullable=False)
    promotion_tag = db.Column(db.String(255), nullable=False)
    app_exclusive_price = db.Column(db.Numeric(10, 2), nullable=False)
    is_app_exclusive = db.Column(db.Integer, nullable=False)
    is_limited = db.Column(db.Integer, nullable=False)
    is_hot = db.Column(db.Integer, nullable=False)


class GoodsAttribute(db.Model):
    __tablename__ = 'goods_attribute'

    create_time = db.Column(db.DateTime)
    update_time = db.Column(db.DateTime)
    delete_time = db.Column(db.DateTime)
    id = db.Column(db.Integer, primary_key=True)
    goods_id = db.Column(db.Integer, nullable=False, index=True)
    attribute_id = db.Column(db.Integer, nullable=False, index=True)
    values = db.Column(db.Text)
```

迁移过程可能出现的错误

```text
1:错误信息
  ERROR [root] Error: Can't locate revision identified by '6222b4449499'
2:解决办法
  进入数据库删除 alembic_version 表
  再次迁移数据库
```

> 如果是 flask 项目， 可以使用 `flask_sqlacodegen` 来实现

```bash
# 1: 迁移整个数据库
flask-sqlacodegen --flask --outfile models.py mysql+pymysql://root:root@localhost:3306/test
# 2: 迁移库中某些表
flask-sqlacodegen --flask --outfile models.py mysql+pymysql://root:root@localhost:3306/test --tables teacher,student
```
