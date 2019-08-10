---
title: Django表单
date: 2018-07-13 12:25:25
tags:
- Django
- 表单
categories: Web
---

![](http://p3euxxfa8.bkt.clouddn.com//18-6-26/12332190.jpg)

今天把Django中表单相关知识总结下, 以作备忘.

html表单的存在使得用户可以和网站进行交互, 从而使得网站并不仅仅作为内容展示, 而是有了更具有实际意义的意义

# 登录表单
首先以一个登录表单作为例子, 来看看表单在Django中的使用.

常见的最简单的html表单如下所示:

```html
<form action="/login" method="post">
    <label for="username">Your name: </label>
    <input id="username" type="text" name="username" value="{{ current_name }}">

    <label for="password">Your name: </label>
    <input id="password" type="text" name="password" value="{{ password }}">
    <input type="submit" value="OK">
</form>
```

要生成这样的一个表单, 在Django中常见的做法是新建一个forms.py文件, 然后继承自form.Form, 然后加入需要的字段, 如下所示:

```py
[forms.py]
class UserForm(forms.Form):
    username = forms.CharField(label='User Name')
    password = forms.CharField(widget=forms.PasswordInput())
```

这里label是提供给前端方便读取的表头项, Widgets定义了html中input的type项[Each form field has a corresponding Widget class, which in turn corresponds to an HTML form widget such as <input type="text">]
这样就创建了一个简单的登录表单. 然后可以在views.py中使用它. 常见的用法是

```py
[views.py]
if request.method == 'POST':
    user_form = UserForm(request.POST)
    ....
else:
    user_form = UserForm()
```

这里的逻辑是如果是post请求, 就用提交的数据实例化表单, 否则返回一个全新的表单给前端

完成表单的实例化后, 就可以在前端模板中使用它

```html
[login.html]
{% for field in login_form %}
    <tr>
        <td  height="30"><label for="{{ field.id_for_label }}">{{ field.label }}</label></td>
        <td>{{ field }}</td>
    </tr>
{% endfor %}

```

# ModelForm和Form
form在Django源码中的位置结构如图所示:

![](http://p3euxxfa8.bkt.clouddn.com/2018-07-16-11-02-44.png)

在Django中常用的Form有两种, 一种就是上面提到的forms.Form, 一种是ModelForm. 其中ModelForm将用户定义的模型和HTML表单直接联系起来(这是Django admin的基础所在.)

不论是ModelForm还是forms.Form, 都继承自BaseForm. BaseForm中定义了一系列基础的方法, 比如init, clean, add_error等. 

# 自定义初始化表单

有时候需要自定义或者动态的初始化表单. 我遇到的场景就是表单中包含一个多选框, 但是多选框的选项是动态的, 不可能一开始就写死.

对于选择框, 常见的写法可能是这样的

```py
class WorkflowJob(models.Model):
    ADJUST_CHOICE=((u'是',u'是'),(u'否',u'否'),)
    projectteam = models.ForeignKey(ProjectTeam,verbose_name=u'项目组',related_name='WorkflowJob_projectteam',default=7)
    adjust = models.CharField (u'是否可调整',max_length=30,choices=ADJUST_CHOICE,default='否')
```

先将选项的所有可能以元组的形式列出, 然后指定相关的input的选项. 

如果是要动态获取选项, 比如想要动态获取分组信息, 则代码为:

```py
class UserForm(forms.Form):
    role = forms.MultipleChoiceField(widget=forms.CheckboxSelectMultiple)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['role'].choices = Group.objects.all().values_list('id', 'name')

```

对应生成的表单如下

![](http://p3euxxfa8.bkt.clouddn.com/2018-07-16-11-31-04.png)

这里生成的表单中的value对应的就是用户组的id值, 当post请求的时候如果是多选, 传回的是一个id组成的列表, 所以后端代码为:

```py

if request.method == 'POST':
    user_edit_form = UserEditForm(request.POST)
    context = {}
    if user_edit_form.is_valid():
        username = user_edit_form.cleaned_data.get('username', '')
        displayname = user_edit_form.cleaned_data.get('displayname', '')
        password = user_edit_form.cleaned_data.get('password', '')
        email = user_edit_form.cleaned_data.get('email', '')
        role = user_edit_form.cleaned_data.get('role', '')
        agent = user_edit_form.cleaned_data.get('agent_code', '')
        vendor = user_edit_form.cleaned_data.get('vendor_code', '')
        factory = user_edit_form.cleaned_data.get('factory_code', '')

        user = User.objects.get(username=username)
        user.display_name = displayname
        user.password = make_password(password)
        user.email = email
        user.agent_code = agent
        user.vendor_code = vendor
        user.factory_code = factory

        role_ids = [int(_id) for _id in role]

        for _id in role_ids:
            group = Group.objects.get(pk=_id)
            user.groups.add(group)

        user.save()
```

这里提到了post传回的role是一个列表形式的, 所以当有需要修改数据的时候, 相应的也要将对应的数据取出来后以列表传给role. 

```py
role = user.groups.values_list()

role_ids = [_role[0] for _role in role]

user_dict = {
        ....
        'role': role_ids,
        ....
    }
edit_form = UserEditForm(user_dict)
```

# is_valid() 的验证逻辑

在post请求中, 完成实例化后就需要对数据进行验证, 也就是form.is_valid()方法. 

```py
 def is_valid(self):
    """
    Returns True if the form has no errors. Otherwise, False. If errors are
    being ignored, returns False.
    """
    return self.is_bound and not self.errors
```

这里self.errors调用的是self.full_clean()


```py
def full_clean(self):
    """
    Cleans all of self.data and populates self._errors and
    self.cleaned_data.
    """
    self._errors = ErrorDict()
    if not self.is_bound:  # Stop further processing.
        return
    self.cleaned_data = {}
    # If the form is permitted to be empty, and none of the form data has
    # changed from the initial data, short circuit any validation.
    if self.empty_permitted and not self.has_changed():
        return

    self._clean_fields()
    self._clean_form()
    self._post_clean()
```

可以看到验证表单的过程是 is_bound -> self._clean_fields() -> self._clean_form()

重点看下 `self._clean_fields()`

```py
def _clean_fields(self):
    for name, field in self.fields.items():
        # value_from_datadict() gets the data from the data dictionaries.
        # Each widget type knows how to retrieve its own data, because some
        # widgets split data over several HTML fields.
        if field.disabled:
            value = self.initial.get(name, field.initial)
        else:
            value = field.widget.value_from_datadict(self.data, self.files, self.add_prefix(name))
        try:
            if isinstance(field, FileField):
                initial = self.initial.get(name, field.initial)
                value = field.clean(value, initial)
            else:
                value = field.clean(value)
            self.cleaned_data[name] = value
            if hasattr(self, 'clean_%s' % name):
                value = getattr(self, 'clean_%s' % name)()
                self.cleaned_data[name] = value
        except ValidationError as e:
            self.add_error(name, e)
```

可以看到, 经过一系列的验证后, 这一句

```py
if hasattr(self, 'clean_%s' % name):
    value = getattr(self, 'clean_%s' % name)()
    self.cleaned_data[name] = value
```

指明了如何自定义字段的验证方式. 只需要复写一个clean_前缀的name方法即可. 例如

```py
class UserAddForm(forms.Form):
    username = forms.CharField(label='User Name',
                               widget=forms.TextInput(attrs={'class': 'form-control',
                                                             'style': 'width: 254px;height:21px'}))
    ....

    def clean_username(self):
        username = self.cleaned_data.get('username')
        if len(username) < 3:
            raise forms.ValidationError('username should more than 3 words')

        if User.objects.filter(username=username).exists():
            raise forms.ValidationError('user already exist')
        return username
```

