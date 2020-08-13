---
layout: post 
title: Django框架
description: Django框架
categories: 
  - 其他
---
## django.db
djanggo.db的package contents：  
backends （package）  
migration （package）  
models （package）  
transaction  
utils

### django.db.models
#### Django models中的Field
Django创建模型是通过继承django.db中的models实现，如：
```python
from django.db import models

class Question(models.Model)
    question_test = models.CharField(max_length=200)
    pub_date = models.DataTimeField('date published')
```
这里的Field包括以下几种类型，[官方文档](https://docs.djangoproject.com/zh-hans/2.2/ref/models/fields/#model-field-types)：
AutoField
BigAutoField
BigIntegerField
BinaryField
BooleanField
CharField
DateField
DateTimeField
DecimalField
DurationField
EmailField
FileField
FieldDoesNotExist
FilePathField
FloatField
GenericIPAddressField
IPAddressField
IntegerField
NullBooleanField
PositiveIntegerField
PositiveSmallIntegerField
SlugField
SmallIntegerField
TextField
TimeField
URLField
UUIDField
