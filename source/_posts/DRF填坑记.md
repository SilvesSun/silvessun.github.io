---
title: DRF填坑记
date: 2019-04-18 10:35:27
tags:
- drf
- 实战
categories:
- 读书笔记
---

基本情况. 现在使用DRF来返回序列化的结果, 其中相关的代码为

```Python
# blog/model.py
class Blog(models.Model, ReadNumExpandMethod):
    title = models.CharField(max_length=50, verbose_name=u'标题')
    blog_type = models.ForeignKey(BlogType, on_delete=models.DO_NOTHING, verbose_name=u'博客类型')
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.DO_NOTHING, verbose_name=u'作者')
    created_time = models.DateTimeField(auto_now_add=True)
    updated_time = models.DateTimeField(auto_now=True)
    read_details = GenericRelation(ReadDetail)
    tags = models.ManyToManyField('Tag', verbose_name='标签', blank=True)
    overview = models.CharField(max_length=200, default='')

    class Meta:
        verbose_name = u"博客"
        verbose_name_plural = verbose_name
        ordering = ['-created_time']
        # app_label = 'blog'

    def __str__(self):
        return self.title

    def tags_list(self):
        return ','.join(i.name for i in self.tags.all())

# read_statistics/model.py
class ReadNum(models.Model):
    read_num = models.IntegerField(default=0)

    content_type = models.ForeignKey(ContentType, on_delete=models.DO_NOTHING)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')


class ReadNumExpandMethod(object):
    def get_read_num(self):
        try:
            ct = ContentType.objects.get_for_model(self)
            readnum = ReadNum.objects.get(content_type=ct, object_id=self.pk)
            return readnum.read_num
        except exceptions.ObjectDoesNotExist:
            return 0


class ReadDetail(models.Model):
    date = models.DateField(default=timezone.now)
    read_num = models.IntegerField(default=0)

    content_type = models.ForeignKey(ContentType, on_delete=models.DO_NOTHING)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')


```

# 1. CORS

`Access to XMLHttpRequest at 'http://127.0.0.1:8000/blog/' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

解决办法:
  安装`django-cors-headers`, `pip install django-cors-headers`

  配置`settings.py`, 
  ```py
  INSTALLED_APPS = (
  ...
  'corsheaders',   
  ...
  )
  MIDDLEWARE_CLASSES
  = (
  ...   
  'corsheaders.middleware.CorsMiddleware',
  'django.middleware.common.CommonMiddleware',   
  ...
  )

  CORS_ORIGIN_ALLOW_ALL = False
  CORS_ORIGIN_WHITELIST = (
    '127.0.0.1:3000',
    'localhost:3000'

  )
```

# 2.TypeError: Object of type 'GenericRelatedObjectManager' is not JSON serializable

通用视图序列化失败.

这里的序列化器为:

```Python
class BlogSerializer(serializers.ModelSerializer):
    created_time = serializers.DateTimeField(format="%Y-%m-%d")

    class Meta:
        model = Blog

        fields = ('url', 'title', 'content', 'created_time', 'blog_type', 'tags', 'id', 'read_details')
        depth = 1
```

导致出错的字段为`read_details`, 这个字段的 `relation` 为 `GenericRelation`. 依据文档, 需要自定义返回字段的值

```Python
class BlogSerializer(serializers.ModelSerializer):
    created_time = serializers.DateTimeField(format="%Y-%m-%d")
    read_num = serializers.SerializerMethodField()


    # 调用对象的 `get_read_num` 方法, 直接获取到对应的值
    def get_read_num(self, obj):
        return obj.get_read_num()

    class Meta:
        model = Blog

        fields = ('url', 'title', 'content', 'created_time', 'blog_type', 'tags', 'id', 'read_num')
        depth = 1
```

这样序列化成功

```JSON
{
    "url": "http://127.0.0.1:8000/blog/53/",
    "title": "数据库事务",
    "content": "....",
    "created_time": "2019-03-20",
    "blog_type": {
        "id": 21,
        "type_name": "数据库"
    },
    "tags": [
        {
            "id": 72,
            "name": "事务",
            "created_time": "2019-03-20T11:03:55Z",
            "last_modified_time": "2019-03-20T11:03:55Z"
        }
    ],
    "id": 53,
    "read_num": 14
}
```