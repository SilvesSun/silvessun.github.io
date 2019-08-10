---
title: python3使用ldap认证
date: 2018-04-18 11:29:37
tags:
- ldap3
- Python
categories:
- Python
- Django学习
---

最近在写后台登录时用到ldap认证. 将相关的知识记录下来.

首先是Python3中有专门的ldap3库, 其[文档](http://ldap3.readthedocs.io/tutorial_operations.html)

然后登录逻辑为:
```
用户登录时首先检查本地缓存中是否有相应的用户, 没有的情况下登录远程ldap服务器, 通过管理员账号查找所在域中是否有相关的账户
```

相关的代码实现
```py
if login_form.is_valid():
    user_name = request.POST.get('username', '')
    password = request.POST.get('password', '')
    user = authenticate(username=user_name, password=password)

    if user is not None:
        if user.is_active:
            login(request, user)
            return render(request, 'access/index.html', {'user': user})
    else:
        res = check_user(user_name, password)  ## 本地数据库不存在, 在ldap服务器中开始验证
        if res:
            user = UserProfile()
            user.username = user_name
            user.password = make_password(password)
            user.is_active = 0
            user.save()
            login(request, user)
            return render(request, 'access/index.html', {'user': user})
```

ldap 的简单验证代码为
```py
def check_user(user, password):
    server = Server(AUTH_LDAP_SERVER_URI, get_info=ALL)
    init_user = 'domian\\%s' % user
    try:
        Connection(server, user=init_user, password=password, auto_bind=True)
        return True
    except Exception as e:
        return False
```
