---
title: Django缓存
date: 2018-02-05 14:47:41
tags:
- Django
- 缓存
categories:
- Django
- Web
---

## 什么是缓存
Web缓存是指一个Web资源（如html页面，图片，js，数据等）存在于Web服务器和客户端（浏览器）之间的副本。缓存会根据进来的请求保存输出内容的副本；当下一个请求来到的时候，如果是相同的URL，缓存会根据缓存机制决定是直接使用副本响应访问请求，还是向源服务器再次发送请求。比较常见的就是浏览器会缓存访问过网站的网页，当再次访问这个URL地址的时候，如果网页没有更新，就不会再次下载网页，而是直接使用本地缓存的网页。只有当网站明确标识资源已经更新，浏览器才会再次下载网页

## 缓存的作用
缓存的目的是为了避免重复的计算, 特别是对一些比较耗时间, 资源的运算. 使用缓存可以减少网络带宽消耗, 降低服务器压力, 减少网络延迟, 加快网页的打开速度.

## 缓存的类型
- 数据库缓存
- 服务器端缓存
  - 代理服务器缓存
  -  CDN缓存
- 浏览器端缓存
- Web应用层缓存

## Django的缓存
Django提供了一个稳定的缓存系统让你缓存动态页面的结果，这样在接下来有相同的请求就可以直接使用缓存中的数据，避免不必要的重复计算。 另外Django还提供了不同粒度数据的缓存，例如： 你可以缓存整个页面，也可以缓存某个部分，甚至缓存整个网站。

### 设定缓存
使用缓存首先需要一些基本的设定工作, 需要告诉Django缓存的数据应该放在哪里, 在文件系统中, 在数据库中还是直接在内存中. Django的缓存设置是setting.py中的 CACHES的BACKEND. 相关的默认代码
```py
{
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}
```

CACHES设置必须配置‘default’缓存；还可以指定任何数量的附加高速缓存。如果您正在使用本地内存高速缓存之外的其他高速缓存后端，或者需要定义多个高速缓存，这就需要添加其他高速缓存项。

可用的高速缓存选项有:
- 'django.core.cache.backends.db.DatabaseCache'
- 'django.core.cache.backends.dummy.DummyCache'
- 'django.core.cache.backends.filebased.FileBasedCache'
- 'django.core.cache.backends.locmem.LocMemCache'
- 'django.core.cache.backends.memcached.MemcachedCache'
- 'django.core.cache.backends.memcached.PyLibMCCache'

### 内存缓存
Memcached是迄今为止可用于Django的最快，最有效的缓存类型，Memcached是完全基于内存的缓存框架

在安装了Memcached本身之后，你将需要安装Memcached Python绑定，它没有直接和Django绑定

需要在Django中使用 Memcached 时
- 将 BACKEND 设置为django.core.cache.backends.memcached.MemcachedCache 或者 django.core.cache.backends.memcached.PyLibMCCache (取决于你所选绑定memcached的方式)
- 将 LOCATION 设置为 ip:port 值，ip 是 Memcached 守护进程的ip地址， port 是Memcached 运行的端口。或者设置为 unix:path 值，path 是 Memcached Unix socket file的路径.

```py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}
```

Memcached 可以让几个服务器的缓存共享, 这意味着可以在多台服务器上运行 Memcached 服务, 这些程序会将这几个机器当做同一个缓存.

```py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}
```
上面的例子中就共享了两个缓存

### 数据库缓存
设置
```py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
```

指定缓存为django.core.cache.backends.db.DatabaseCache, 同时指定缓存位置为my_cache_table表

#### 创建缓存表
```
python manage.py createcachetable
```
这将在你的数据库中创建一个Django的基于数据库缓存系统预期的特定格式的数据表。表名会从 LOCATION中获得。

#### 多数据库
如果在多数据库的情况下使用数据库缓存，还必须为数据库缓存表设置路由说明

例如, 下面的路由会分配所有缓存读操作到 cache_replica， 和所有写操作到cache_primary. 缓存表只会同步到 cache_primary:
```py
class CacheRouter(object):
    """A router to control all database cache operations"""

    def db_for_read(self, model, **hints):
        "All cache read operations go to the replica"
        if model._meta.app_label == 'django_cache':
            return 'cache_replica'
        return None

    def db_for_write(self, model, **hints):
        "All cache write operations go to primary"
        if model._meta.app_label == 'django_cache':
            return 'cache_primary'
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        "Only install the cache model on primary"
        if app_label == 'django_cache':
            return db == 'cache_primary'
        return None
```

### 文件系统缓存
### 本地内存缓存
