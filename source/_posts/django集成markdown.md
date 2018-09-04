---
title: django集成markdown
date: 2018-04-26 16:38:49
tags:
- Django
- markdown
categories:
- Django学习
---

最近开发完Django blog的基本功能, 想在后台中集成markdown富文本编辑. 以下是开发流程

## mdeditor的使用
经过大致的比对, 最终选择了django-mdeditor这个库.Django-mdeditor 是基于[editor.md](https://github.com/pandao/editor.md) 的一个 django Markdown 文本编辑插件应用, 支持Editor.md的大部分功能, 这里只做简单的基本功能配置

### 基本配置使用
- 安装
```
pip install django-mdeditor
```
- setting设置
在setting.py的INSTALLED_APPS 中添加
```
INSTALLED_APPS = [
        ...
        'mdeditor',
    ]
```
- urls配置
在项目的总urls.py中添加

```py

path('mdeditor/', include('mdeditor.urls'))
```

- 同步
```
python manage.py makemigrations
python manage.py migrate
```

到这步基本的设置就完成了, 在后台编辑博客可以看到的界面如下:
![](http://p3euxxfa8.bkt.clouddn.com/a453617d942078f97dd676b48c5bb2d4.png)

### 后端生成html文本
在后台书写完成的博客, 需要在前端页面中展示. 将markdown生成对应的html页面的插件也很多. 这里选用[mistune](https://github.com/lepture/mistune)
- 自定义render
为了便于识别语言简写, 需要对mistune原本的render做一定的修改, 直接在view中继承Renderer即可

```py

class CustomerRender(mistune.Renderer):
    def block_code(self, code, lang=None):
        """Rendering block level code. ``pre > code``.

        :param code: text content of the code block.
        :param lang: language of the given code.
        """
        code = code.rstrip('\n')
        lang_dic = {
            'py': 'python',
            'js': 'javascript',
        }
        if not lang:
            code = escape(code, smart_amp=False)
            return '<pre><code>%s\n</code></pre>\n' % code
        code = escape(code, quote=True, smart_amp=False)
        return '<pre><code class="%s">%s\n</code></pre>\n' % (lang_dic.get(lang, lang), code)
``

以上代码中可以自己定义lang_dic.

这里这样处理是为了方便前端使用highlight.js高亮代码块

- 完成转换
```py

blog_content = mistune.markdown(blog.content)
```

### 前端使用highlight.js高亮代码块
在head头引入相关文件

```html

<link rel="stylesheet" href="{% static 'js/styles/rainbow.css' %}">
<script src="{% static 'js/highlight.pack.js' %}"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

现在前端也能正常显示了.效果图:
![](http://p3euxxfa8.bkt.clouddn.com/0dbec89b17a38cbc04222d34cbc92acd.png)
