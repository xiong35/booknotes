
- 用postman在body里添加raw参数, 在request.POST里取不出数据
- info = json.load(request.body)

- 新增数据时添加有多对多关系的数据
``` py
tags = ArticalTag.objects.filter(tag_name__in=info.get('tags'))
for tag in tags:
    record.tags.add(tag) 
```

- 已拦截跨源请求：同源策略禁止读取位于 http://127.0.0.1:8000/artical/blog/ 的远程资源。（原因：CORS 请求未能成功）
```py
from django.utils.deprecation import MiddlewareMixin
class CORSMid(MiddlewareMixin):
    def process_response(self, request, response):
        response["Access-Control-Allow-Origin"] = "*"
        return response
```