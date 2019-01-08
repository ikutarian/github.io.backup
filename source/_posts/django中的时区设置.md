---
title: django中的时区设置
date: 2018-09-17 10:44:15
tags:
  - Django
  - 时区
categories:
  - Python
---

在settings.py中设置了 

```
TIME_ZONE = 'Asia/Shanghai' 
USE_TZ = True 
```

现在的北京时间是 22点35分。django shell 中运行 timezone.now() 结果如下

```
>>> from django.utils import timezone 
>>> timezone.get_current_timezone_name() 
'Asia/Shanghai' 
>>> timezone.now() 
datetime.datetime(2018, 6, 11, 14, 35, 22, 946256, tzinfo=<UTC>)
```

显示的时间不是我现在的 22:35 而是 14：35。timezone 是 UTC，而不是我设置的 `'Asia/Shanghai'`

为什么是这个结果？

<!-- more -->

> Django 如果开启了 Time Zone 功能，则所有的存储和内部处理，甚至包括直接 print 显示全都是 UTC 的。只有通过模板进行表单输入/渲染输出的时候，才会执行 UTC 本地时间的转换。

> 建议后台处理时间的时候，最好完全使用 UTC，不要考虑本地时间的存在。而显示时间的时候，也避免手动转换，尽量使用 Django 模板系统代劳

启用 `USE_TZ = True` 后，处理时间方面，有两条 “黄金法则”

> 1. 保证存储到数据库中的是 UTC 时间；
> 2. 在函数之间传递时间参数时，确保时间已经转换成 UTC 时间；

 比如，通常获取当前时间用的是

```python
import datetime
now = datetime.datetime.now()
```

启用 `USE_TZ = True` 后，需要写成

```python
import datetime 
from django.utils.timezone import utc
utcnow = datetime.datetime.utcnow().replace(tzinfo=utc)
```

至于模板，除非应用支持用户设置自己所在的时区，通常我们不需要关心模板的时区问题。模板在展示时间的时候，会使用 `settings.TIME_ZONE` 中的设置自动把 UTC 时间转成 `settings.TIME_ZONE` 所在时区的时间渲染

## 参考

* [django中的时区设置TIME_ZONE,USE_TZ](https://blog.csdn.net/w6299702/article/details/38782607)
* [为什么Django设置时区为TIME_ZONE = 'Asia/Shanghai' USE_TZ = True后，存入mysql中的时间只能是UTC时间](https://blog.csdn.net/qq_27361945/article/details/80580795)