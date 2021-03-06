
# 用django读取http post的数据时不能直接在request.POST里取出数据

此时body里的数据还是字符串/二进制编码, 需要用函数读取

```python
info = json.load(request.body)
```


# 使用 django 的 manytomany feild 新增数据添加有多对多关系的数据

以本博客为例, tags 和 artical 时多对多关系, 需要先选出所有相关的 tag, 在逐一对他们操作

``` py
tags = ArticalTag.objects.filter(tag_name__in=info.get('tags'))
for tag in tags:
    record.tags.add(tag) 
```

# 对django发送post时出现以下问题: 同源策略禁止读取位于xxx的远程资源。（原因：CORS 请求未能成功）

在进行真正的post请求前, 前端会发送一个OPTIONS请求  
它用于获取当前URL所支持的方法。若请求成功，则它会在HTTP头中包含一个名为“Allow”的头，值是所支持的方法，如“GET, POST”  

在django中添加中间件, 在返回的请求中加入```Access-Control-Allow-Origin = "*"```, 这样所有ip都可以对站点发送post了(但是不那么安全)

```py
# 在 django 项目根目录里新建 somewhat 文件夹
# 在其中新建 foo.py, 里面写 bar 方法
# 在 settings.py 里的 MIDDLEWARE 里注册这一条'somewhat.foo.bar',
# foo/bar/somewhat 的名字可以随便取, 一致即可

from django.utils.deprecation import MiddlewareMixin
class bar(MiddlewareMixin):
    def process_response(self, request, response):
        response["Access-Control-Allow-Origin"] = "*"
        return response
```

# 对django发送post时出现以下问题: 同源策略禁止读取位于xxx的远程资源。（原因：CORS 预检响应的 'Access-Control-Allow-Headers'，不允许使用头 'content-type'）

参见[这篇文章](http://xiong35.cn/blog2.0/articles/trap/38)  
在上述文章的基础上, 向 bar 函数中添加以下代码, 干脆把所有控制解除(不太安全)

```python
response["Access-Control-Allow-Origin"] = "*"
response["Access-Control-Allow-Credentials"] = "true"
response["Access-Control-Allow-Methods"] = "*"
response["Access-Control-Allow-Headers"] = "Content-Type,Access-Token"
response["Access-Control-Expose-Headers"] = "*"
```

参考文章: [r.f.](https://www.cnblogs.com/caimuqing/p/6733405.html)


# django中使用基于数据库+类视图的缓存, 用manage.py新建数据表

运行```python manage.py createcachetable TABLE_NAME```  
在settings.py中添加

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'TABLE_NAME',
    }
}
```

### 需要对类视图使用[方法装饰器]

将类方法伪装成普通方法, 再使用缓存装饰器

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page

class HelloPage(View):
    @method_decorator(cache_page(60*60*24))
    def get(self,request):
        print("enter HelloPage.get")

        return HttpResponse('Hello world ...')
```

# diango中使用基于redis的缓存(linux)

## 配置redis环境

```shellscript
pip install django-redis
pip install django-redis-cache
redis-server
```

[参考教程](https://www.runoob.com/redis/redis-install.html)

## 设置django的缓存后端

setting.py 中添加

```python
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient'
        }
    }
}
```
