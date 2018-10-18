============================
8.1 改变对象的字符串显示
============================

----------
问题
----------
你想改变对象实例的打印或显示输出，让它们更具可读性。

----------
解决方案
----------
要改变一个实例的字符串表示，可重新定义它的 ``__str__()`` 和 ``__repr__()`` 方法。例如：

.. code-block:: python

    class Pair:
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def __repr__(self):
            return 'Pair({0.x!r}, {0.y!r})'.format(self)

        def __str__(self):
            return '({0.x!s}, {0.y!s})'.format(self)

``__repr__()`` 方法返回一个实例的代码表示形式，通常用来重新构造这个实例。
内置的 ``repr()`` 函数返回这个字符串，跟我们使用交互式解释器显示的值是一样的。
``__str__()`` 方法将实例转换为一个字符串，使用 ``str()`` 或 ``print()`` 函数会输出这个字符串。比如：

.. code-block:: python

    >>> p = Pair(3, 4)
    >>> p
    Pair(3, 4) # __repr__() output
    >>> print(p)
    (3, 4) # __str__() output
    >>>

我们在这里还演示了在格式化的时候怎样使用不同的字符串表现形式。
特别来讲，``!r`` 格式化代码指明输出使用 ``__repr__()`` 来代替默认的 ``__str__()`` 。
你可以用前面的类来试着测试下：

.. code-block:: python

    >>> p = Pair(3, 4)
    >>> print('p is {0!r}'.format(p))
    p is Pair(3, 4)
    >>> print('p is {0}'.format(p))
    p is (3, 4)
    >>>

----------
讨论
----------
自定义 ``__repr__()`` 和 ``__str__()`` 通常是很好的习惯，因为它能简化调试和实例输出。
例如，如果仅仅只是打印输出或日志输出某个实例，那么程序员会看到实例更加详细与有用的信息。

``__repr__()`` 生成的文本字符串标准做法是需要让 ``eval(repr(x)) == x`` 为真。
如果实在不能这样子做，应该创建一个有用的文本表示，并使用 < 和 > 括起来。比如：

.. code-block:: python

    >>> f = open('file.dat')
    >>> f
    <_io.TextIOWrapper name='file.dat' mode='r' encoding='UTF-8'>
    >>>

如果 ``__str__()`` 没有被定义，那么就会使用 ``__repr__()`` 来代替输出。

上面的 ``format()`` 方法的使用看上去很有趣，格式化代码 ``{0.x}`` 对应的是第1个参数的x属性。
因此，在下面的函数中，0实际上指的就是 ``self`` 本身：

.. code-block:: python

    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)

作为这种实现的一个替代，你也可以使用 ``%`` 操作符，就像下面这样：

.. code-block:: python

    def __repr__(self):
        return 'Pair(%r, %r)' % (self.x, self.y)

============================
8.2 自定义字符串的格式化
============================

----------
问题
----------
你想通过 ``format()`` 函数和字符串方法使得一个对象能支持自定义的格式化。

----------
解决方案
----------
为了自定义字符串的格式化，我们需要在类上面定义 ``__format__()`` 方法。例如：

.. code-block:: python

    _formats = {
        'ymd' : '{d.year}-{d.month}-{d.day}',
        'mdy' : '{d.month}/{d.day}/{d.year}',
        'dmy' : '{d.day}/{d.month}/{d.year}'
        }

    class Date:
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

        def __format__(self, code):
            if code == '':
                code = 'ymd'
            fmt = _formats[code]
            return fmt.format(d=self)

现在 ``Date`` 类的实例可以支持格式化操作了，如同下面这样：

.. code-block:: python

    >>> d = Date(2012, 12, 21)
    >>> format(d)
    '2012-12-21'
    >>> format(d, 'mdy')
    '12/21/2012'
    >>> 'The date is {:ymd}'.format(d)
    'The date is 2012-12-21'
    >>> 'The date is {:mdy}'.format(d)
    'The date is 12/21/2012'
    >>>

----------
讨论
----------
``__format__()`` 方法给Python的字符串格式化功能提供了一个钩子。
这里需要着重强调的是格式化代码的解析工作完全由类自己决定。因此，格式化代码可以是任何值。
例如，参考下面来自 ``datetime`` 模块中的代码：

.. code-block:: python

    >>> from datetime import date
    >>> d = date(2012, 12, 21)
    >>> format(d)
    '2012-12-21'
    >>> format(d,'%A, %B %d, %Y')
    'Friday, December 21, 2012'
    >>> 'The end is {:%d %b %Y}. Goodbye'.format(d)
    'The end is 21 Dec 2012. Goodbye'
    >>>

对于内置类型的格式化有一些标准的约定。
可以参考 `string模块文档 <https://docs.python.org/3/library/string.html>`_ 说明。
============================
8.3 让对象支持上下文管理协议
============================

----------
问题
----------
你想让你的对象支持上下文管理协议(with语句)。

----------
解决方案
----------
为了让一个对象兼容 ``with`` 语句，你需要实现 ``__enter__()`` 和 ``__exit__()`` 方法。
例如，考虑如下的一个类，它能为我们创建一个网络连接：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM

    class LazyConnection:
        def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
            self.address = address
            self.family = family
            self.type = type
            self.sock = None

        def __enter__(self):
            if self.sock is not None:
                raise RuntimeError('Already connected')
            self.sock = socket(self.family, self.type)
            self.sock.connect(self.address)
            return self.sock

        def __exit__(self, exc_ty, exc_val, tb):
            self.sock.close()
            self.sock = None

这个类的关键特点在于它表示了一个网络连接，但是初始化的时候并不会做任何事情(比如它并没有建立一个连接)。
连接的建立和关闭是使用 ``with`` 语句自动完成的，例如：

.. code-block:: python

    from functools import partial

    conn = LazyConnection(('www.python.org', 80))
    # Connection closed
    with conn as s:
        # conn.__enter__() executes: connection open
        s.send(b'GET /index.html HTTP/1.0\r\n')
        s.send(b'Host: www.python.org\r\n')
        s.send(b'\r\n')
        resp = b''.join(iter(partial(s.recv, 8192), b''))
        # conn.__exit__() executes: connection closed

----------
讨论
----------
编写上下文管理器的主要原理是你的代码会放到 ``with`` 语句块中执行。
当出现 ``with`` 语句的时候，对象的 ``__enter__()`` 方法被触发，
它返回的值(如果有的话)会被赋值给 ``as`` 声明的变量。然后，``with`` 语句块里面的代码开始执行。
最后，``__exit__()`` 方法被触发进行清理工作。

不管 ``with`` 代码块中发生什么，上面的控制流都会执行完，就算代码块中发生了异常也是一样的。
事实上，``__exit__()`` 方法的第三个参数包含了异常类型、异常值和追溯信息(如果有的话)。
``__exit__()`` 方法能自己决定怎样利用这个异常信息，或者忽略它并返回一个None值。
如果 ``__exit__()`` 返回 ``True`` ，那么异常会被清空，就好像什么都没发生一样，
``with`` 语句后面的程序继续在正常执行。

还有一个细节问题就是 ``LazyConnection`` 类是否允许多个 ``with`` 语句来嵌套使用连接。
很显然，上面的定义中一次只能允许一个socket连接，如果正在使用一个socket的时候又重复使用 ``with`` 语句，
就会产生一个异常了。不过你可以像下面这样修改下上面的实现来解决这个问题：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM

    class LazyConnection:
        def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
            self.address = address
            self.family = family
            self.type = type
            self.connections = []

        def __enter__(self):
            sock = socket(self.family, self.type)
            sock.connect(self.address)
            self.connections.append(sock)
            return sock

        def __exit__(self, exc_ty, exc_val, tb):
            self.connections.pop().close()

    # Example use
    from functools import partial

    conn = LazyConnection(('www.python.org', 80))
    with conn as s1:
        pass
        with conn as s2:
            pass
            # s1 and s2 are independent sockets

在第二个版本中，``LazyConnection`` 类可以被看做是某个连接工厂。在内部，一个列表被用来构造一个栈。
每次 ``__enter__()`` 方法执行的时候，它复制创建一个新的连接并将其加入到栈里面。
``__exit__()`` 方法简单的从栈中弹出最后一个连接并关闭它。
这里稍微有点难理解，不过它能允许嵌套使用 ``with`` 语句创建多个连接，就如上面演示的那样。

在需要管理一些资源比如文件、网络连接和锁的编程环境中，使用上下文管理器是很普遍的。
这些资源的一个主要特征是它们必须被手动的关闭或释放来确保程序的正确运行。
例如，如果你请求了一个锁，那么你必须确保之后释放了它，否则就可能产生死锁。
通过实现 ``__enter__()`` 和 ``__exit__()`` 方法并使用 ``with`` 语句可以很容易的避免这些问题，
因为 ``__exit__()`` 方法可以让你无需担心这些了。

在 ``contextmanager`` 模块中有一个标准的上下文管理方案模板，可参考9.22小节。
同时在12.6小节中还有一个对本节示例程序的线程安全的修改版。

============================
8.4 创建大量对象时节省内存方法
============================

----------
问题
----------
你的程序要创建大量(可能上百万)的对象，导致占用很大的内存。

----------
解决方案
----------
对于主要是用来当成简单的数据结构的类而言，你可以通过给类添加 ``__slots__`` 属性来极大的减少实例所占的内存。比如：

.. code-block:: python

    class Date:
        __slots__ = ['year', 'month', 'day']
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

当你定义 ``__slots__`` 后，Python就会为实例使用一种更加紧凑的内部表示。
实例通过一个很小的固定大小的数组来构建，而不是为每个实例定义一个字典，这跟元组或列表很类似。
在 ``__slots__`` 中列出的属性名在内部被映射到这个数组的指定小标上。
使用slots一个不好的地方就是我们不能再给实例添加新的属性了，只能使用在 ``__slots__`` 中定义的那些属性名。

----------
讨论
----------
使用slots后节省的内存会跟存储属性的数量和类型有关。
不过，一般来讲，使用到的内存总量和将数据存储在一个元组中差不多。
为了给你一个直观认识，假设你不使用slots直接存储一个Date实例，
在64位的Python上面要占用428字节，而如果使用了slots，内存占用下降到156字节。
如果程序中需要同时创建大量的日期实例，那么这个就能极大的减小内存使用量了。

尽管slots看上去是一个很有用的特性，很多时候你还是得减少对它的使用冲动。
Python的很多特性都依赖于普通的基于字典的实现。
另外，定义了slots后的类不再支持一些普通类特性了，比如多继承。
大多数情况下，你应该只在那些经常被使用到的用作数据结构的类上定义slots
(比如在程序中需要创建某个类的几百万个实例对象)。

关于 ``__slots__`` 的一个常见误区是它可以作为一个封装工具来防止用户给实例增加新的属性。
尽管使用slots可以达到这样的目的，但是这个并不是它的初衷。
``__slots__`` 更多的是用来作为一个内存优化工具。

============================
8.5 在类中封装属性名
============================

----------
问题
----------
你想封装类的实例上面的“私有”数据，但是Python语言并没有访问控制。

----------
解决方案
----------
Python程序员不去依赖语言特性去封装数据，而是通过遵循一定的属性和方法命名规约来达到这个效果。
第一个约定是任何以单下划线_开头的名字都应该是内部实现。比如：

.. code-block:: python

    class A:
        def __init__(self):
            self._internal = 0 # An internal attribute
            self.public = 1 # A public attribute

        def public_method(self):
            '''
            A public method
            '''
            pass

        def _internal_method(self):
            pass

Python并不会真的阻止别人访问内部名称。但是如果你这么做肯定是不好的，可能会导致脆弱的代码。
同时还要注意到，使用下划线开头的约定同样适用于模块名和模块级别函数。
例如，如果你看到某个模块名以单下划线开头(比如_socket)，那它就是内部实现。
类似的，模块级别函数比如 ``sys._getframe()`` 在使用的时候就得加倍小心了。

你还可能会遇到在类定义中使用两个下划线(__)开头的命名。比如：

.. code-block:: python

    class B:
        def __init__(self):
            self.__private = 0

        def __private_method(self):
            pass

        def public_method(self):
            pass
            self.__private_method()

使用双下划线开始会导致访问名称变成其他形式。
比如，在前面的类B中，私有属性会被分别重命名为 ``_B__private`` 和 ``_B__private_method`` 。
这时候你可能会问这样重命名的目的是什么，答案就是继承——这种属性通过继承是无法被覆盖的。比如：

.. code-block:: python

    class C(B):
        def __init__(self):
            super().__init__()
            self.__private = 1 # Does not override B.__private

        # Does not override B.__private_method()
        def __private_method(self):
            pass

这里，私有名称 ``__private`` 和 ``__private_method``
被重命名为 ``_C__private`` 和 ``_C__private_method`` ，这个跟父类B中的名称是完全不同的。

----------
讨论
----------
上面提到有两种不同的编码约定(单下划线和双下划线)来命名私有属性，那么问题就来了：到底哪种方式好呢？
大多数而言，你应该让你的非公共名称以单下划线开头。但是，如果你清楚你的代码会涉及到子类，
并且有些内部属性应该在子类中隐藏起来，那么才考虑使用双下划线方案。

还有一点要注意的是，有时候你定义的一个变量和某个保留关键字冲突，这时候可以使用单下划线作为后缀，例如：

.. code-block:: python

    lambda_ = 2.0 # Trailing _ to avoid clash with lambda keyword

这里我们并不使用单下划线前缀的原因是它避免误解它的使用初衷
(如使用单下划线前缀的目的是为了防止命名冲突而不是指明这个属性是私有的)。
通过使用单下划线后缀可以解决这个问题。
============================
8.6 创建可管理的属性
============================

----------
问题
----------
你想给某个实例attribute增加除访问与修改之外的其他处理逻辑，比如类型检查或合法性验证。

----------
解决方案
----------
自定义某个属性的一种简单方法是将它定义为一个property。
例如，下面的代码定义了一个property，增加对一个属性简单的类型检查：

.. code-block:: python

    class Person:
        def __init__(self, first_name):
            self.first_name = first_name

        # Getter function
        @property
        def first_name(self):
            return self._first_name

        # Setter function
        @first_name.setter
        def first_name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._first_name = value

        # Deleter function (optional)
        @first_name.deleter
        def first_name(self):
            raise AttributeError("Can't delete attribute")

上述代码中有三个相关联的方法，这三个方法的名字都必须一样。
第一个方法是一个 ``getter`` 函数，它使得 ``first_name`` 成为一个属性。
其他两个方法给 ``first_name`` 属性添加了 ``setter`` 和 ``deleter`` 函数。
需要强调的是只有在 ``first_name`` 属性被创建后，
后面的两个装饰器 ``@first_name.setter`` 和 ``@first_name.deleter`` 才能被定义。

property的一个关键特征是它看上去跟普通的attribute没什么两样，
但是访问它的时候会自动触发 ``getter`` 、``setter`` 和 ``deleter`` 方法。例如：

.. code-block:: python

    >>> a = Person('Guido')
    >>> a.first_name # Calls the getter
    'Guido'
    >>> a.first_name = 42 # Calls the setter
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "prop.py", line 14, in first_name
            raise TypeError('Expected a string')
    TypeError: Expected a string
    >>> del a.first_name
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: can`t delete attribute
    >>>

在实现一个property的时候，底层数据(如果有的话)仍然需要存储在某个地方。
因此，在get和set方法中，你会看到对 ``_first_name`` 属性的操作，这也是实际数据保存的地方。
另外，你可能还会问为什么 ``__init__()`` 方法中设置了 ``self.first_name`` 而不是 ``self._first_name`` 。
在这个例子中，我们创建一个property的目的就是在设置attribute的时候进行检查。
因此，你可能想在初始化的时候也进行这种类型检查。通过设置 ``self.first_name`` ，自动调用 ``setter`` 方法，
这个方法里面会进行参数的检查，否则就是直接访问 ``self._first_name`` 了。

还能在已存在的get和set方法基础上定义property。例如：

.. code-block:: python

    class Person:
        def __init__(self, first_name):
            self.set_first_name(first_name)

        # Getter function
        def get_first_name(self):
            return self._first_name

        # Setter function
        def set_first_name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._first_name = value

        # Deleter function (optional)
        def del_first_name(self):
            raise AttributeError("Can't delete attribute")

        # Make a property from existing get/set methods
        name = property(get_first_name, set_first_name, del_first_name)

----------
讨论
----------
一个property属性其实就是一系列相关绑定方法的集合。如果你去查看拥有property的类，
就会发现property本身的fget、fset和fdel属性就是类里面的普通方法。比如：

.. code-block:: python

    >>> Person.first_name.fget
    <function Person.first_name at 0x1006a60e0>
    >>> Person.first_name.fset
    <function Person.first_name at 0x1006a6170>
    >>> Person.first_name.fdel
    <function Person.first_name at 0x1006a62e0>
    >>>

通常来讲，你不会直接取调用fget或者fset，它们会在访问property的时候自动被触发。

只有当你确实需要对attribute执行其他额外的操作的时候才应该使用到property。
有时候一些从其他编程语言(比如Java)过来的程序员总认为所有访问都应该通过getter和setter，
所以他们认为代码应该像下面这样写：

.. code-block:: python

    class Person:
        def __init__(self, first_name):
            self.first_name = first_name

        @property
        def first_name(self):
            return self._first_name

        @first_name.setter
        def first_name(self, value):
            self._first_name = value

不要写这种没有做任何其他额外操作的property。
首先，它会让你的代码变得很臃肿，并且还会迷惑阅读者。
其次，它还会让你的程序运行起来变慢很多。
最后，这样的设计并没有带来任何的好处。
特别是当你以后想给普通attribute访问添加额外的处理逻辑的时候，
你可以将它变成一个property而无需改变原来的代码。
因为访问attribute的代码还是保持原样。

Properties还是一种定义动态计算attribute的方法。
这种类型的attributes并不会被实际的存储，而是在需要的时候计算出来。比如：

.. code-block:: python

    import math
    class Circle:
        def __init__(self, radius):
            self.radius = radius

        @property
        def area(self):
            return math.pi * self.radius ** 2

        @property
        def diameter(self):
            return self.radius * 2

        @property
        def perimeter(self):
            return 2 * math.pi * self.radius

在这里，我们通过使用properties，将所有的访问接口形式统一起来，
对半径、直径、周长和面积的访问都是通过属性访问，就跟访问简单的attribute是一样的。
如果不这样做的话，那么就要在代码中混合使用简单属性访问和方法调用。
下面是使用的实例：

.. code-block:: python

    >>> c = Circle(4.0)
    >>> c.radius
    4.0
    >>> c.area  # Notice lack of ()
    50.26548245743669
    >>> c.perimeter  # Notice lack of ()
    25.132741228718345
    >>>

尽管properties可以实现优雅的编程接口，但有些时候你还是会想直接使用getter和setter函数。例如：

.. code-block:: python

    >>> p = Person('Guido')
    >>> p.get_first_name()
    'Guido'
    >>> p.set_first_name('Larry')
    >>>

这种情况的出现通常是因为Python代码被集成到一个大型基础平台架构或程序中。
例如，有可能是一个Python类准备加入到一个基于远程过程调用的大型分布式系统中。
这种情况下，直接使用get/set方法(普通方法调用)而不是property或许会更容易兼容。

最后一点，不要像下面这样写有大量重复代码的property定义：

.. code-block:: python

    class Person:
        def __init__(self, first_name, last_name):
            self.first_name = first_name
            self.last_name = last_name

        @property
        def first_name(self):
            return self._first_name

        @first_name.setter
        def first_name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._first_name = value

        # Repeated property code, but for a different name (bad!)
        @property
        def last_name(self):
            return self._last_name

        @last_name.setter
        def last_name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._last_name = value

重复代码会导致臃肿、易出错和丑陋的程序。好消息是，通过使用装饰器或闭包，有很多种更好的方法来完成同样的事情。
可以参考8.9和9.21小节的内容。
============================
8.7 调用父类方法
============================

----------
问题
----------
你想在子类中调用父类的某个已经被覆盖的方法。

----------
解决方案
----------
为了调用父类(超类)的一个方法，可以使用 ``super()`` 函数，比如：

.. code-block:: python

    class A:
        def spam(self):
            print('A.spam')

    class B(A):
        def spam(self):
            print('B.spam')
            super().spam()  # Call parent spam()

``super()`` 函数的一个常见用法是在 ``__init__()`` 方法中确保父类被正确的初始化了：

.. code-block:: python

    class A:
        def __init__(self):
            self.x = 0

    class B(A):
        def __init__(self):
            super().__init__()
            self.y = 1

``super()`` 的另外一个常见用法出现在覆盖Python特殊方法的代码中，比如：

.. code-block:: python

    class Proxy:
        def __init__(self, obj):
            self._obj = obj

        # Delegate attribute lookup to internal obj
        def __getattr__(self, name):
            return getattr(self._obj, name)

        # Delegate attribute assignment
        def __setattr__(self, name, value):
            if name.startswith('_'):
                super().__setattr__(name, value) # Call original __setattr__
            else:
                setattr(self._obj, name, value)

在上面代码中，``__setattr__()`` 的实现包含一个名字检查。
如果某个属性名以下划线(_)开头，就通过 ``super()`` 调用原始的 ``__setattr__()`` ，
否则的话就委派给内部的代理对象 ``self._obj`` 去处理。
这看上去有点意思，因为就算没有显式的指明某个类的父类， ``super()`` 仍然可以有效的工作。

----------
讨论
----------
实际上，大家对于在Python中如何正确使用 ``super()`` 函数普遍知之甚少。
你有时候会看到像下面这样直接调用父类的一个方法：

.. code-block:: python

    class Base:
        def __init__(self):
            print('Base.__init__')

    class A(Base):
        def __init__(self):
            Base.__init__(self)
            print('A.__init__')

尽管对于大部分代码而言这么做没什么问题，但是在更复杂的涉及到多继承的代码中就有可能导致很奇怪的问题发生。
比如，考虑如下的情况：

.. code-block:: python

    class Base:
        def __init__(self):
            print('Base.__init__')

    class A(Base):
        def __init__(self):
            Base.__init__(self)
            print('A.__init__')

    class B(Base):
        def __init__(self):
            Base.__init__(self)
            print('B.__init__')

    class C(A,B):
        def __init__(self):
            A.__init__(self)
            B.__init__(self)
            print('C.__init__')

如果你运行这段代码就会发现 ``Base.__init__()`` 被调用两次，如下所示：

.. code-block:: python

    >>> c = C()
    Base.__init__
    A.__init__
    Base.__init__
    B.__init__
    C.__init__
    >>>

可能两次调用 ``Base.__init__()`` 没什么坏处，但有时候却不是。
另一方面，假设你在代码中换成使用 ``super()`` ，结果就很完美了：

.. code-block:: python

    class Base:
        def __init__(self):
            print('Base.__init__')

    class A(Base):
        def __init__(self):
            super().__init__()
            print('A.__init__')

    class B(Base):
        def __init__(self):
            super().__init__()
            print('B.__init__')

    class C(A,B):
        def __init__(self):
            super().__init__()  # Only one call to super() here
            print('C.__init__')

运行这个新版本后，你会发现每个 ``__init__()`` 方法只会被调用一次了：

.. code-block:: python

    >>> c = C()
    Base.__init__
    B.__init__
    A.__init__
    C.__init__
    >>>

为了弄清它的原理，我们需要花点时间解释下Python是如何实现继承的。
对于你定义的每一个类，Python会计算出一个所谓的方法解析顺序(MRO)列表。
这个MRO列表就是一个简单的所有基类的线性顺序表。例如：

.. code-block:: python

    >>> C.__mro__
    (<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>,
    <class '__main__.Base'>, <class 'object'>)
    >>>

为了实现继承，Python会在MRO列表上从左到右开始查找基类，直到找到第一个匹配这个属性的类为止。

而这个MRO列表的构造是通过一个C3线性化算法来实现的。
我们不去深究这个算法的数学原理，它实际上就是合并所有父类的MRO列表并遵循如下三条准则：

* 子类会先于父类被检查
* 多个父类会根据它们在列表中的顺序被检查
* 如果对下一个类存在两个合法的选择，选择第一个父类

老实说，你所要知道的就是MRO列表中的类顺序会让你定义的任意类层级关系变得有意义。

当你使用 ``super()`` 函数时，Python会在MRO列表上继续搜索下一个类。
只要每个重定义的方法统一使用 ``super()`` 并只调用它一次，
那么控制流最终会遍历完整个MRO列表，每个方法也只会被调用一次。
这也是为什么在第二个例子中你不会调用两次 ``Base.__init__()`` 的原因。

``super()`` 有个令人吃惊的地方是它并不一定去查找某个类在MRO中下一个直接父类，
你甚至可以在一个没有直接父类的类中使用它。例如，考虑如下这个类：

.. code-block:: python

    class A:
        def spam(self):
            print('A.spam')
            super().spam()

如果你试着直接使用这个类就会出错：

.. code-block:: python

    >>> a = A()
    >>> a.spam()
    A.spam
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 4, in spam
    AttributeError: 'super' object has no attribute 'spam'
    >>>

但是，如果你使用多继承的话看看会发生什么：

.. code-block:: python

    >>> class B:
    ...     def spam(self):
    ...         print('B.spam')
    ...
    >>> class C(A,B):
    ...     pass
    ...
    >>> c = C()
    >>> c.spam()
    A.spam
    B.spam
    >>>

你可以看到在类A中使用 ``super().spam()`` 实际上调用的是跟类A毫无关系的类B中的 ``spam()`` 方法。
这个用类C的MRO列表就可以完全解释清楚了：

.. code-block:: python

    >>> C.__mro__
    (<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>,
    <class 'object'>)
    >>>

在定义混入类的时候这样使用 ``super()`` 是很普遍的。可以参考8.13和8.18小节。

然而，由于 ``super()`` 可能会调用不是你想要的方法，你应该遵循一些通用原则。
首先，确保在继承体系中所有相同名字的方法拥有可兼容的参数签名(比如相同的参数个数和参数名称)。
这样可以确保 ``super()`` 调用一个非直接父类方法时不会出错。
其次，最好确保最顶层的类提供了这个方法的实现，这样的话在MRO上面的查找链肯定可以找到某个确定的方法。

在Python社区中对于 ``super()`` 的使用有时候会引来一些争议。
尽管如此，如果一切顺利的话，你应该在你最新代码中使用它。
Raymond Hettinger为此写了一篇非常好的文章
`“Python’s super() Considered Super!” <http://rhettinger.wordpress.com/2011/05/26/super-considered-super>`_ ，
通过大量的例子向我们解释了为什么 ``super()`` 是极好的。

============================
8.8 子类中扩展property
============================

----------
问题
----------
在子类中，你想要扩展定义在父类中的property的功能。

----------
解决方案
----------
考虑如下的代码，它定义了一个property：

.. code-block:: python

    class Person:
        def __init__(self, name):
            self.name = name

        # Getter function
        @property
        def name(self):
            return self._name

        # Setter function
        @name.setter
        def name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._name = value

        # Deleter function
        @name.deleter
        def name(self):
            raise AttributeError("Can't delete attribute")

下面是一个示例类，它继承自Person并扩展了 ``name`` 属性的功能：

.. code-block:: python

    class SubPerson(Person):
        @property
        def name(self):
            print('Getting name')
            return super().name

        @name.setter
        def name(self, value):
            print('Setting name to', value)
            super(SubPerson, SubPerson).name.__set__(self, value)

        @name.deleter
        def name(self):
            print('Deleting name')
            super(SubPerson, SubPerson).name.__delete__(self)

接下来使用这个新类：

.. code-block:: python

    >>> s = SubPerson('Guido')
    Setting name to Guido
    >>> s.name
    Getting name
    'Guido'
    >>> s.name = 'Larry'
    Setting name to Larry
    >>> s.name = 42
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 16, in name
            raise TypeError('Expected a string')
    TypeError: Expected a string
    >>>

如果你仅仅只想扩展property的某一个方法，那么可以像下面这样写：

.. code-block:: python

    class SubPerson(Person):
        @Person.name.getter
        def name(self):
            print('Getting name')
            return super().name

或者，你只想修改setter方法，就这么写：

.. code-block:: python

    class SubPerson(Person):
        @Person.name.setter
        def name(self, value):
            print('Setting name to', value)
            super(SubPerson, SubPerson).name.__set__(self, value)

----------
讨论
----------
在子类中扩展一个property可能会引起很多不易察觉的问题，
因为一个property其实是 ``getter``、``setter`` 和 ``deleter`` 方法的集合，而不是单个方法。
因此，当你扩展一个property的时候，你需要先确定你是否要重新定义所有的方法还是说只修改其中某一个。

在第一个例子中，所有的property方法都被重新定义。
在每一个方法中，使用了 ``super()`` 来调用父类的实现。
在 ``setter`` 函数中使用 ``super(SubPerson, SubPerson).name.__set__(self, value)`` 的语句是没有错的。
为了委托给之前定义的setter方法，需要将控制权传递给之前定义的name属性的 ``__set__()`` 方法。
不过，获取这个方法的唯一途径是使用类变量而不是实例变量来访问它。
这也是为什么我们要使用 ``super(SubPerson, SubPerson)`` 的原因。

如果你只想重定义其中一个方法，那只使用 @property 本身是不够的。比如，下面的代码就无法工作：

.. code-block:: python

    class SubPerson(Person):
        @property  # Doesn't work
        def name(self):
            print('Getting name')
            return super().name

如果你试着运行会发现setter函数整个消失了：

.. code-block:: python

    >>> s = SubPerson('Guido')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 5, in __init__
            self.name = name
    AttributeError: can't set attribute
    >>>

你应该像之前说过的那样修改代码：

.. code-block:: python

    class SubPerson(Person):
        @Person.name.getter
        def name(self):
            print('Getting name')
            return super().name

这么写后，property之前已经定义过的方法会被复制过来，而getter函数被替换。然后它就能按照期望的工作了：

.. code-block:: python

    >>> s = SubPerson('Guido')
    >>> s.name
    Getting name
    'Guido'
    >>> s.name = 'Larry'
    >>> s.name
    Getting name
    'Larry'
    >>> s.name = 42
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 16, in name
            raise TypeError('Expected a string')
    TypeError: Expected a string
    >>>

在这个特别的解决方案中，我们没办法使用更加通用的方式去替换硬编码的 ``Person`` 类名。
如果你不知道到底是哪个基类定义了property，
那你只能通过重新定义所有property并使用 ``super()`` 来将控制权传递给前面的实现。

值得注意的是上面演示的第一种技术还可以被用来扩展一个描述器(在8.9小节我们有专门的介绍)。比如：

.. code-block:: python

    # A descriptor
    class String:
        def __init__(self, name):
            self.name = name

        def __get__(self, instance, cls):
            if instance is None:
                return self
            return instance.__dict__[self.name]

        def __set__(self, instance, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            instance.__dict__[self.name] = value

    # A class with a descriptor
    class Person:
        name = String('name')

        def __init__(self, name):
            self.name = name

    # Extending a descriptor with a property
    class SubPerson(Person):
        @property
        def name(self):
            print('Getting name')
            return super().name

        @name.setter
        def name(self, value):
            print('Setting name to', value)
            super(SubPerson, SubPerson).name.__set__(self, value)

        @name.deleter
        def name(self):
            print('Deleting name')
            super(SubPerson, SubPerson).name.__delete__(self)

最后值得注意的是，读到这里时，你应该会发现子类化 ``setter`` 和 ``deleter`` 方法其实是很简单的。
这里演示的解决方案同样适用，但是在 `Python的issue页面 <http://bugs.python.org/issue14965>`_
报告的一个bug，或许会使得将来的Python版本中出现一个更加简洁的方法。

============================
8.9 创建新的类或实例属性
============================

----------
问题
----------
你想创建一个新的拥有一些额外功能的实例属性类型，比如类型检查。

----------
解决方案
----------
如果你想创建一个全新的实例属性，可以通过一个描述器类的形式来定义它的功能。下面是一个例子：

.. code-block:: python

    # Descriptor attribute for an integer type-checked attribute
    class Integer:
        def __init__(self, name):
            self.name = name

        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                return instance.__dict__[self.name]

        def __set__(self, instance, value):
            if not isinstance(value, int):
                raise TypeError('Expected an int')
            instance.__dict__[self.name] = value

        def __delete__(self, instance):
            del instance.__dict__[self.name]

一个描述器就是一个实现了三个核心的属性访问操作(get, set, delete)的类，
分别为 ``__get__()`` 、``__set__()`` 和 ``__delete__()`` 这三个特殊的方法。
这些方法接受一个实例作为输入，之后相应的操作实例底层的字典。

为了使用一个描述器，需将这个描述器的实例作为类属性放到一个类的定义中。例如：

.. code-block:: python

    class Point:
        x = Integer('x')
        y = Integer('y')

        def __init__(self, x, y):
            self.x = x
            self.y = y

当你这样做后，所有对描述器属性(比如x或y)的访问会被
``__get__()`` 、``__set__()`` 和 ``__delete__()`` 方法捕获到。例如：

.. code-block:: python

    >>> p = Point(2, 3)
    >>> p.x # Calls Point.x.__get__(p,Point)
    2
    >>> p.y = 5 # Calls Point.y.__set__(p, 5)
    >>> p.x = 2.3 # Calls Point.x.__set__(p, 2.3)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "descrip.py", line 12, in __set__
            raise TypeError('Expected an int')
    TypeError: Expected an int
    >>>

作为输入，描述器的每一个方法会接受一个操作实例。
为了实现请求操作，会相应的操作实例底层的字典(__dict__属性)。
描述器的 ``self.name`` 属性存储了在实例字典中被实际使用到的key。

----------
讨论
----------
描述器可实现大部分Python类特性中的底层魔法，
包括 ``@classmethod`` 、``@staticmethod`` 、``@property`` ，甚至是 ``__slots__`` 特性。

通过定义一个描述器，你可以在底层捕获核心的实例操作(get, set, delete)，并且可完全自定义它们的行为。
这是一个强大的工具，有了它你可以实现很多高级功能，并且它也是很多高级库和框架中的重要工具之一。

描述器的一个比较困惑的地方是它只能在类级别被定义，而不能为每个实例单独定义。因此，下面的代码是无法工作的：

.. code-block:: python

    # Does NOT work
    class Point:
        def __init__(self, x, y):
            self.x = Integer('x') # No! Must be a class variable
            self.y = Integer('y')
            self.x = x
            self.y = y

同时，``__get__()`` 方法实现起来比看上去要复杂得多：

.. code-block:: python

    # Descriptor attribute for an integer type-checked attribute
    class Integer:

        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                return instance.__dict__[self.name]

``__get__()`` 看上去有点复杂的原因归结于实例变量和类变量的不同。
如果一个描述器被当做一个类变量来访问，那么 ``instance`` 参数被设置成 ``None`` 。
这种情况下，标准做法就是简单的返回这个描述器本身即可(尽管你还可以添加其他的自定义操作)。例如：

.. code-block:: python

    >>> p = Point(2,3)
    >>> p.x # Calls Point.x.__get__(p, Point)
    2
    >>> Point.x # Calls Point.x.__get__(None, Point)
    <__main__.Integer object at 0x100671890>
    >>>


描述器通常是那些使用到装饰器或元类的大型框架中的一个组件。同时它们的使用也被隐藏在后面。
举个例子，下面是一些更高级的基于描述器的代码，并涉及到一个类装饰器：

.. code-block:: python

    # Descriptor for a type-checked attribute
    class Typed:
        def __init__(self, name, expected_type):
            self.name = name
            self.expected_type = expected_type
        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                return instance.__dict__[self.name]

        def __set__(self, instance, value):
            if not isinstance(value, self.expected_type):
                raise TypeError('Expected ' + str(self.expected_type))
            instance.__dict__[self.name] = value
        def __delete__(self, instance):
            del instance.__dict__[self.name]

    # Class decorator that applies it to selected attributes
    def typeassert(**kwargs):
        def decorate(cls):
            for name, expected_type in kwargs.items():
                # Attach a Typed descriptor to the class
                setattr(cls, name, Typed(name, expected_type))
            return cls
        return decorate

    # Example use
    @typeassert(name=str, shares=int, price=float)
    class Stock:
        def __init__(self, name, shares, price):
            self.name = name
            self.shares = shares
            self.price = price

最后要指出的一点是，如果你只是想简单的自定义某个类的单个属性访问的话就不用去写描述器了。
这种情况下使用8.6小节介绍的property技术会更加容易。
当程序中有很多重复代码的时候描述器就很有用了
(比如你想在你代码的很多地方使用描述器提供的功能或者将它作为一个函数库特性)。


============================
8.10 使用延迟计算属性
============================

----------
问题
----------
你想将一个只读属性定义成一个property，并且只在访问的时候才会计算结果。
但是一旦被访问后，你希望结果值被缓存起来，不用每次都去计算。

----------
解决方案
----------
定义一个延迟属性的一种高效方法是通过使用一个描述器类，如下所示：

.. code-block:: python

    class lazyproperty:
        def __init__(self, func):
            self.func = func

        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                value = self.func(instance)
                setattr(instance, self.func.__name__, value)
                return value

你需要像下面这样在一个类中使用它：

.. code-block:: python

    import math

    class Circle:
        def __init__(self, radius):
            self.radius = radius

        @lazyproperty
        def area(self):
            print('Computing area')
            return math.pi * self.radius ** 2

        @lazyproperty
        def perimeter(self):
            print('Computing perimeter')
            return 2 * math.pi * self.radius

下面在一个交互环境中演示它的使用：

.. code-block:: python

    >>> c = Circle(4.0)
    >>> c.radius
    4.0
    >>> c.area
    Computing area
    50.26548245743669
    >>> c.area
    50.26548245743669
    >>> c.perimeter
    Computing perimeter
    25.132741228718345
    >>> c.perimeter
    25.132741228718345
    >>>

仔细观察你会发现消息 ``Computing area`` 和 ``Computing perimeter`` 仅仅出现一次。

----------
讨论
----------
很多时候，构造一个延迟计算属性的主要目的是为了提升性能。
例如，你可以避免计算这些属性值，除非你真的需要它们。
这里演示的方案就是用来实现这样的效果的，
只不过它是通过以非常高效的方式使用描述器的一个精妙特性来达到这种效果的。

正如在其他小节(如8.9小节)所讲的那样，当一个描述器被放入一个类的定义时，
每次访问属性时它的 ``__get__()`` 、``__set__()`` 和 ``__delete__()`` 方法就会被触发。
不过，如果一个描述器仅仅只定义了一个 ``__get__()`` 方法的话，它比通常的具有更弱的绑定。
特别地，只有当被访问属性不在实例底层的字典中时 ``__get__()`` 方法才会被触发。

``lazyproperty`` 类利用这一点，使用 ``__get__()`` 方法在实例中存储计算出来的值，
这个实例使用相同的名字作为它的property。
这样一来，结果值被存储在实例字典中并且以后就不需要再去计算这个property了。
你可以尝试更深入的例子来观察结果：

.. code-block:: python

    >>> c = Circle(4.0)
    >>> # Get instance variables
    >>> vars(c)
    {'radius': 4.0}

    >>> # Compute area and observe variables afterward
    >>> c.area
    Computing area
    50.26548245743669
    >>> vars(c)
    {'area': 50.26548245743669, 'radius': 4.0}

    >>> # Notice access doesn't invoke property anymore
    >>> c.area
    50.26548245743669

    >>> # Delete the variable and see property trigger again
    >>> del c.area
    >>> vars(c)
    {'radius': 4.0}
    >>> c.area
    Computing area
    50.26548245743669
    >>>

这种方案有一个小缺陷就是计算出的值被创建后是可以被修改的。例如：

.. code-block:: python

    >>> c.area
    Computing area
    50.26548245743669
    >>> c.area = 25
    >>> c.area
    25
    >>>

如果你担心这个问题，那么可以使用一种稍微没那么高效的实现，就像下面这样：

.. code-block:: python

    def lazyproperty(func):
        name = '_lazy_' + func.__name__
        @property
        def lazy(self):
            if hasattr(self, name):
                return getattr(self, name)
            else:
                value = func(self)
                setattr(self, name, value)
                return value
        return lazy

如果你使用这个版本，就会发现现在修改操作已经不被允许了：

.. code-block:: python

    >>> c = Circle(4.0)
    >>> c.area
    Computing area
    50.26548245743669
    >>> c.area
    50.26548245743669
    >>> c.area = 25
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>>

然而，这种方案有一个缺点就是所有get操作都必须被定向到属性的 ``getter`` 函数上去。
这个跟之前简单的在实例字典中查找值的方案相比效率要低一点。
如果想获取更多关于property和可管理属性的信息，可以参考8.6小节。而描述器的相关内容可以在8.9小节找到。

============================
8.11 简化数据结构的初始化
============================

----------
问题
----------
你写了很多仅仅用作数据结构的类，不想写太多烦人的 ``__init__()`` 函数

----------
解决方案
----------
可以在一个基类中写一个公用的 ``__init__()`` 函数：

.. code-block:: python

    import math

    class Structure1:
        # Class variable that specifies expected fields
        _fields = []

        def __init__(self, *args):
            if len(args) != len(self._fields):
                raise TypeError('Expected {} arguments'.format(len(self._fields)))
            # Set the arguments
            for name, value in zip(self._fields, args):
                setattr(self, name, value)

然后使你的类继承自这个基类:

.. code-block:: python

    # Example class definitions
    class Stock(Structure1):
        _fields = ['name', 'shares', 'price']

    class Point(Structure1):
        _fields = ['x', 'y']

    class Circle(Structure1):
        _fields = ['radius']

        def area(self):
            return math.pi * self.radius ** 2

使用这些类的示例：

.. code-block:: python

    >>> s = Stock('ACME', 50, 91.1)
    >>> p = Point(2, 3)
    >>> c = Circle(4.5)
    >>> s2 = Stock('ACME', 50)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "structure.py", line 6, in __init__
            raise TypeError('Expected {} arguments'.format(len(self._fields)))
    TypeError: Expected 3 arguments

如果还想支持关键字参数，可以将关键字参数设置为实例属性：

.. code-block:: python

    class Structure2:
        _fields = []

        def __init__(self, *args, **kwargs):
            if len(args) > len(self._fields):
                raise TypeError('Expected {} arguments'.format(len(self._fields)))

            # Set all of the positional arguments
            for name, value in zip(self._fields, args):
                setattr(self, name, value)

            # Set the remaining keyword arguments
            for name in self._fields[len(args):]:
                setattr(self, name, kwargs.pop(name))

            # Check for any remaining unknown arguments
            if kwargs:
                raise TypeError('Invalid argument(s): {}'.format(','.join(kwargs)))
    # Example use
    if __name__ == '__main__':
        class Stock(Structure2):
            _fields = ['name', 'shares', 'price']

        s1 = Stock('ACME', 50, 91.1)
        s2 = Stock('ACME', 50, price=91.1)
        s3 = Stock('ACME', shares=50, price=91.1)
        # s3 = Stock('ACME', shares=50, price=91.1, aa=1)

你还能将不在 ``_fields`` 中的名称加入到属性中去：

.. code-block:: python

    class Structure3:
        # Class variable that specifies expected fields
        _fields = []

        def __init__(self, *args, **kwargs):
            if len(args) != len(self._fields):
                raise TypeError('Expected {} arguments'.format(len(self._fields)))

            # Set the arguments
            for name, value in zip(self._fields, args):
                setattr(self, name, value)

            # Set the additional arguments (if any)
            extra_args = kwargs.keys() - self._fields
            for name in extra_args:
                setattr(self, name, kwargs.pop(name))

            if kwargs:
                raise TypeError('Duplicate values for {}'.format(','.join(kwargs)))

    # Example use
    if __name__ == '__main__':
        class Stock(Structure3):
            _fields = ['name', 'shares', 'price']

        s1 = Stock('ACME', 50, 91.1)
        s2 = Stock('ACME', 50, 91.1, date='8/2/2012')



----------
讨论
----------
当你需要使用大量很小的数据结构类的时候，
相比手工一个个定义 ``__init__()`` 方法而已，使用这种方式可以大大简化代码。

在上面的实现中我们使用了 ``setattr()`` 函数类设置属性值，
你可能不想用这种方式，而是想直接更新实例字典，就像下面这样：

.. code-block:: python

    class Structure:
        # Class variable that specifies expected fields
        _fields= []
        def __init__(self, *args):
            if len(args) != len(self._fields):
                raise TypeError('Expected {} arguments'.format(len(self._fields)))

            # Set the arguments (alternate)
            self.__dict__.update(zip(self._fields,args))


尽管这也可以正常工作，但是当定义子类的时候问题就来了。
当一个子类定义了 ``__slots__`` 或者通过property(或描述器)来包装某个属性，
那么直接访问实例字典就不起作用了。我们上面使用 ``setattr()`` 会显得更通用些，因为它也适用于子类情况。

这种方法唯一不好的地方就是对某些IDE而言，在显示帮助函数时可能不太友好。比如：

.. code-block:: python

    >>> help(Stock)
    Help on class Stock in module __main__:
    class Stock(Structure)
    ...
    | Methods inherited from Structure:
    |
    | __init__(self, *args, **kwargs)
    |
    ...
    >>>

可以参考9.16小节来强制在 ``__init__()`` 方法中指定参数的类型签名。
============================
8.12 定义接口或者抽象基类
============================

----------
问题
----------
你想定义一个接口或抽象类，并且通过执行类型检查来确保子类实现了某些特定的方法

----------
解决方案
----------
使用 ``abc`` 模块可以很轻松的定义抽象基类：

.. code-block:: python

    from abc import ABCMeta, abstractmethod

    class IStream(metaclass=ABCMeta):
        @abstractmethod
        def read(self, maxbytes=-1):
            pass

        @abstractmethod
        def write(self, data):
            pass

抽象类的一个特点是它不能直接被实例化，比如你想像下面这样做是不行的：

.. code-block:: python

    a = IStream() # TypeError: Can't instantiate abstract class
                    # IStream with abstract methods read, write

抽象类的目的就是让别的类继承它并实现特定的抽象方法：

.. code-block:: python

    class SocketStream(IStream):
        def read(self, maxbytes=-1):
            pass

        def write(self, data):
            pass

抽象基类的一个主要用途是在代码中检查某些类是否为特定类型，实现了特定接口：

.. code-block:: python

    def serialize(obj, stream):
        if not isinstance(stream, IStream):
            raise TypeError('Expected an IStream')
        pass

除了继承这种方式外，还可以通过注册方式来让某个类实现抽象基类：

.. code-block:: python

    import io

    # Register the built-in I/O classes as supporting our interface
    IStream.register(io.IOBase)

    # Open a normal file and type check
    f = open('foo.txt')
    isinstance(f, IStream) # Returns True


``@abstractmethod`` 还能注解静态方法、类方法和 ``properties`` 。
你只需保证这个注解紧靠在函数定义前即可：

.. code-block:: python

    class A(metaclass=ABCMeta):
        @property
        @abstractmethod
        def name(self):
            pass

        @name.setter
        @abstractmethod
        def name(self, value):
            pass

        @classmethod
        @abstractmethod
        def method1(cls):
            pass

        @staticmethod
        @abstractmethod
        def method2():
            pass

----------
讨论
----------
标准库中有很多用到抽象基类的地方。``collections`` 模块定义了很多跟容器和迭代器(序列、映射、集合等)有关的抽象基类。
``numbers`` 库定义了跟数字对象(整数、浮点数、有理数等)有关的基类。``io`` 库定义了很多跟I/O操作相关的基类。

你可以使用预定义的抽象类来执行更通用的类型检查，例如：

.. code-block:: python

    import collections

    # Check if x is a sequence
    if isinstance(x, collections.Sequence):
    ...

    # Check if x is iterable
    if isinstance(x, collections.Iterable):
    ...

    # Check if x has a size
    if isinstance(x, collections.Sized):
    ...

    # Check if x is a mapping
    if isinstance(x, collections.Mapping):

尽管ABCs可以让我们很方便的做类型检查，但是我们在代码中最好不要过多的使用它。
因为Python的本质是一门动态编程语言，其目的就是给你更多灵活性，
强制类型检查或让你代码变得更复杂，这样做无异于舍本求末。
============================
8.13 实现数据模型的类型约束
============================

----------
问题
----------
你想定义某些在属性赋值上面有限制的数据结构。

----------
解决方案
----------
在这个问题中，你需要在对某些实例属性赋值时进行检查。
所以你要自定义属性赋值函数，这种情况下最好使用描述器。

下面的代码使用描述器实现了一个系统类型和赋值验证框架：

.. code-block:: python

    # Base class. Uses a descriptor to set a value
    class Descriptor:
        def __init__(self, name=None, **opts):
            self.name = name
            for key, value in opts.items():
                setattr(self, key, value)

        def __set__(self, instance, value):
            instance.__dict__[self.name] = value


    # Descriptor for enforcing types
    class Typed(Descriptor):
        expected_type = type(None)

        def __set__(self, instance, value):
            if not isinstance(value, self.expected_type):
                raise TypeError('expected ' + str(self.expected_type))
            super().__set__(instance, value)


    # Descriptor for enforcing values
    class Unsigned(Descriptor):
        def __set__(self, instance, value):
            if value < 0:
                raise ValueError('Expected >= 0')
            super().__set__(instance, value)


    class MaxSized(Descriptor):
        def __init__(self, name=None, **opts):
            if 'size' not in opts:
                raise TypeError('missing size option')
            super().__init__(name, **opts)

        def __set__(self, instance, value):
            if len(value) >= self.size:
                raise ValueError('size must be < ' + str(self.size))
            super().__set__(instance, value)


这些类就是你要创建的数据模型或类型系统的基础构建模块。
下面就是我们实际定义的各种不同的数据类型：

.. code-block:: python

    class Integer(Typed):
        expected_type = int

    class UnsignedInteger(Integer, Unsigned):
        pass

    class Float(Typed):
        expected_type = float

    class UnsignedFloat(Float, Unsigned):
        pass

    class String(Typed):
        expected_type = str

    class SizedString(String, MaxSized):
        pass

然后使用这些自定义数据类型，我们定义一个类：

.. code-block:: python

    class Stock:
        # Specify constraints
        name = SizedString('name', size=8)
        shares = UnsignedInteger('shares')
        price = UnsignedFloat('price')

        def __init__(self, name, shares, price):
            self.name = name
            self.shares = shares
            self.price = price

然后测试这个类的属性赋值约束，可发现对某些属性的赋值违法了约束是不合法的：

.. code-block:: python

    >>> s.name
    'ACME'
    >>> s.shares = 75
    >>> s.shares = -10
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 17, in __set__
            super().__set__(instance, value)
        File "example.py", line 23, in __set__
            raise ValueError('Expected >= 0')
    ValueError: Expected >= 0
    >>> s.price = 'a lot'
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 16, in __set__
            raise TypeError('expected ' + str(self.expected_type))
    TypeError: expected <class 'float'>
    >>> s.name = 'ABRACADABRA'
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 17, in __set__
            super().__set__(instance, value)
        File "example.py", line 35, in __set__
            raise ValueError('size must be < ' + str(self.size))
    ValueError: size must be < 8
    >>>

还有一些技术可以简化上面的代码，其中一种是使用类装饰器：

.. code-block:: python

    # Class decorator to apply constraints
    def check_attributes(**kwargs):
        def decorate(cls):
            for key, value in kwargs.items():
                if isinstance(value, Descriptor):
                    value.name = key
                    setattr(cls, key, value)
                else:
                    setattr(cls, key, value(key))
            return cls

        return decorate

    # Example
    @check_attributes(name=SizedString(size=8),
                      shares=UnsignedInteger,
                      price=UnsignedFloat)
    class Stock:
        def __init__(self, name, shares, price):
            self.name = name
            self.shares = shares
            self.price = price

另外一种方式是使用元类：

.. code-block:: python

    # A metaclass that applies checking
    class checkedmeta(type):
        def __new__(cls, clsname, bases, methods):
            # Attach attribute names to the descriptors
            for key, value in methods.items():
                if isinstance(value, Descriptor):
                    value.name = key
            return type.__new__(cls, clsname, bases, methods)

    # Example
    class Stock2(metaclass=checkedmeta):
        name = SizedString(size=8)
        shares = UnsignedInteger()
        price = UnsignedFloat()

        def __init__(self, name, shares, price):
            self.name = name
            self.shares = shares
            self.price = price


----------
讨论
----------
本节使用了很多高级技术，包括描述器、混入类、``super()`` 的使用、类装饰器和元类。
不可能在这里一一详细展开来讲，但是可以在8.9、8.18、9.19小节找到更多例子。
但是，我在这里还是要提一下几个需要注意的点。

首先，在 ``Descriptor`` 基类中你会看到有个 ``__set__()`` 方法，却没有相应的 ``__get__()`` 方法。
如果一个描述仅仅是从底层实例字典中获取某个属性值的话，那么没必要去定义 ``__get__()`` 方法。

所有描述器类都是基于混入类来实现的。比如 ``Unsigned`` 和 ``MaxSized`` 要跟其他继承自 ``Typed`` 类混入。
这里利用多继承来实现相应的功能。

混入类的一个比较难理解的地方是，调用 ``super()`` 函数时，你并不知道究竟要调用哪个具体类。
你需要跟其他类结合后才能正确的使用，也就是必须合作才能产生效果。

使用类装饰器和元类通常可以简化代码。上面两个例子中你会发现你只需要输入一次属性名即可了。

.. code-block:: python

    # Normal
    class Point:
        x = Integer('x')
        y = Integer('y')

    # Metaclass
    class Point(metaclass=checkedmeta):
        x = Integer()
        y = Integer()

所有方法中，类装饰器方案应该是最灵活和最高明的。
首先，它并不依赖任何其他新的技术，比如元类。其次，装饰器可以很容易的添加或删除。

最后，装饰器还能作为混入类的替代技术来实现同样的效果;

.. code-block:: python

    # Decorator for applying type checking
    def Typed(expected_type, cls=None):
        if cls is None:
            return lambda cls: Typed(expected_type, cls)
        super_set = cls.__set__

        def __set__(self, instance, value):
            if not isinstance(value, expected_type):
                raise TypeError('expected ' + str(expected_type))
            super_set(self, instance, value)

        cls.__set__ = __set__
        return cls


    # Decorator for unsigned values
    def Unsigned(cls):
        super_set = cls.__set__

        def __set__(self, instance, value):
            if value < 0:
                raise ValueError('Expected >= 0')
            super_set(self, instance, value)

        cls.__set__ = __set__
        return cls


    # Decorator for allowing sized values
    def MaxSized(cls):
        super_init = cls.__init__

        def __init__(self, name=None, **opts):
            if 'size' not in opts:
                raise TypeError('missing size option')
            super_init(self, name, **opts)

        cls.__init__ = __init__

        super_set = cls.__set__

        def __set__(self, instance, value):
            if len(value) >= self.size:
                raise ValueError('size must be < ' + str(self.size))
            super_set(self, instance, value)

        cls.__set__ = __set__
        return cls


    # Specialized descriptors
    @Typed(int)
    class Integer(Descriptor):
        pass


    @Unsigned
    class UnsignedInteger(Integer):
        pass


    @Typed(float)
    class Float(Descriptor):
        pass


    @Unsigned
    class UnsignedFloat(Float):
        pass


    @Typed(str)
    class String(Descriptor):
        pass


    @MaxSized
    class SizedString(String):
        pass

这种方式定义的类跟之前的效果一样，而且执行速度会更快。
设置一个简单的类型属性的值，装饰器方式要比之前的混入类的方式几乎快100%。
现在你应该庆幸自己读完了本节全部内容了吧？^_^
============================
8.14 实现自定义容器
============================

----------
问题
----------
你想实现一个自定义的类来模拟内置的容器类功能，比如列表和字典。但是你不确定到底要实现哪些方法。

----------
解决方案
----------
``collections`` 定义了很多抽象基类，当你想自定义容器类的时候它们会非常有用。
比如你想让你的类支持迭代，那就让你的类继承 ``collections.Iterable`` 即可：

.. code-block:: python

    import collections
    class A(collections.Iterable):
        pass

不过你需要实现 ``collections.Iterable`` 所有的抽象方法，否则会报错:

.. code-block:: python

    >>> a = A()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: Can't instantiate abstract class A with abstract methods __iter__
    >>>

你只要实现 ``__iter__()`` 方法就不会报错了(参考4.2和4.7小节)。

你可以先试着去实例化一个对象，在错误提示中可以找到需要实现哪些方法：

.. code-block:: python

    >>> import collections
    >>> collections.Sequence()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: Can't instantiate abstract class Sequence with abstract methods \
    __getitem__, __len__
    >>>

下面是一个简单的示例，继承自上面Sequence抽象类，并且实现元素按照顺序存储：

.. code-block:: python

    class SortedItems(collections.Sequence):
        def __init__(self, initial=None):
            self._items = sorted(initial) if initial is not None else []

        # Required sequence methods
        def __getitem__(self, index):
            return self._items[index]

        def __len__(self):
            return len(self._items)

        # Method for adding an item in the right location
        def add(self, item):
            bisect.insort(self._items, item)


    items = SortedItems([5, 1, 3])
    print(list(items))
    print(items[0], items[-1])
    items.add(2)
    print(list(items))

可以看到，SortedItems跟普通的序列没什么两样，支持所有常用操作，包括索引、迭代、包含判断，甚至是切片操作。

这里面使用到了 ``bisect`` 模块，它是一个在排序列表中插入元素的高效方式。可以保证元素插入后还保持顺序。

----------
讨论
----------
使用 ``collections`` 中的抽象基类可以确保你自定义的容器实现了所有必要的方法。并且还能简化类型检查。
你的自定义容器会满足大部分类型检查需要，如下所示：

.. code-block:: python

    >>> items = SortedItems()
    >>> import collections
    >>> isinstance(items, collections.Iterable)
    True
    >>> isinstance(items, collections.Sequence)
    True
    >>> isinstance(items, collections.Container)
    True
    >>> isinstance(items, collections.Sized)
    True
    >>> isinstance(items, collections.Mapping)
    False
    >>>

``collections`` 中很多抽象类会为一些常见容器操作提供默认的实现，
这样一来你只需要实现那些你最感兴趣的方法即可。假设你的类继承自 ``collections.MutableSequence`` ，如下：

.. code-block:: python

    class Items(collections.MutableSequence):
        def __init__(self, initial=None):
            self._items = list(initial) if initial is not None else []

        # Required sequence methods
        def __getitem__(self, index):
            print('Getting:', index)
            return self._items[index]

        def __setitem__(self, index, value):
            print('Setting:', index, value)
            self._items[index] = value

        def __delitem__(self, index):
            print('Deleting:', index)
            del self._items[index]

        def insert(self, index, value):
            print('Inserting:', index, value)
            self._items.insert(index, value)

        def __len__(self):
            print('Len')
            return len(self._items)

如果你创建 ``Items`` 的实例，你会发现它支持几乎所有的核心列表方法(如append()、remove()、count()等)。
下面是使用演示：

.. code-block:: python

    >>> a = Items([1, 2, 3])
    >>> len(a)
    Len
    3
    >>> a.append(4)
    Len
    Inserting: 3 4
    >>> a.append(2)
    Len
    Inserting: 4 2
    >>> a.count(2)
    Getting: 0
    Getting: 1
    Getting: 2
    Getting: 3
    Getting: 4
    Getting: 5
    2
    >>> a.remove(3)
    Getting: 0
    Getting: 1
    Getting: 2
    Deleting: 2
    >>>

本小节只是对Python抽象类功能的抛砖引玉。``numbers`` 模块提供了一个类似的跟整数类型相关的抽象类型集合。
可以参考8.12小节来构造更多自定义抽象基类。
============================
8.15 属性的代理访问
============================

----------
问题
----------
你想将某个实例的属性访问代理到内部另一个实例中去，目的可能是作为继承的一个替代方法或者实现代理模式。

----------
解决方案
----------
简单来说，代理是一种编程模式，它将某个操作转移给另外一个对象来实现。
最简单的形式可能是像下面这样：

.. code-block:: python

    class A:
        def spam(self, x):
            pass

        def foo(self):
            pass


    class B1:
        """简单的代理"""

        def __init__(self):
            self._a = A()

        def spam(self, x):
            # Delegate to the internal self._a instance
            return self._a.spam(x)

        def foo(self):
            # Delegate to the internal self._a instance
            return self._a.foo()

        def bar(self):
            pass

如果仅仅就两个方法需要代理，那么像这样写就足够了。但是，如果有大量的方法需要代理，
那么使用 ``__getattr__()`` 方法或许或更好些：

.. code-block:: python

    class B2:
        """使用__getattr__的代理，代理方法比较多时候"""

        def __init__(self):
            self._a = A()

        def bar(self):
            pass

        # Expose all of the methods defined on class A
        def __getattr__(self, name):
            """这个方法在访问的attribute不存在的时候被调用
            the __getattr__() method is actually a fallback method
            that only gets called when an attribute is not found"""
            return getattr(self._a, name)

``__getattr__`` 方法是在访问attribute不存在的时候被调用，使用演示：

.. code-block:: python

    b = B()
    b.bar() # Calls B.bar() (exists on B)
    b.spam(42) # Calls B.__getattr__('spam') and delegates to A.spam

另外一个代理例子是实现代理模式，例如：

.. code-block:: python

    # A proxy class that wraps around another object, but
    # exposes its public attributes
    class Proxy:
        def __init__(self, obj):
            self._obj = obj

        # Delegate attribute lookup to internal obj
        def __getattr__(self, name):
            print('getattr:', name)
            return getattr(self._obj, name)

        # Delegate attribute assignment
        def __setattr__(self, name, value):
            if name.startswith('_'):
                super().__setattr__(name, value)
            else:
                print('setattr:', name, value)
                setattr(self._obj, name, value)

        # Delegate attribute deletion
        def __delattr__(self, name):
            if name.startswith('_'):
                super().__delattr__(name)
            else:
                print('delattr:', name)
                delattr(self._obj, name)

使用这个代理类时，你只需要用它来包装下其他类即可：

.. code-block:: python

    class Spam:
        def __init__(self, x):
            self.x = x

        def bar(self, y):
            print('Spam.bar:', self.x, y)

    # Create an instance
    s = Spam(2)
    # Create a proxy around it
    p = Proxy(s)
    # Access the proxy
    print(p.x)  # Outputs 2
    p.bar(3)  # Outputs "Spam.bar: 2 3"
    p.x = 37  # Changes s.x to 37

通过自定义属性访问方法，你可以用不同方式自定义代理类行为(比如加入日志功能、只读访问等)。

----------
讨论
----------
代理类有时候可以作为继承的替代方案。例如，一个简单的继承如下：

.. code-block:: python

    class A:
        def spam(self, x):
            print('A.spam', x)
        def foo(self):
            print('A.foo')

    class B(A):
        def spam(self, x):
            print('B.spam')
            super().spam(x)
        def bar(self):
            print('B.bar')

使用代理的话，就是下面这样：

.. code-block:: python

    class A:
        def spam(self, x):
            print('A.spam', x)
        def foo(self):
            print('A.foo')

    class B:
        def __init__(self):
            self._a = A()
        def spam(self, x):
            print('B.spam', x)
            self._a.spam(x)
        def bar(self):
            print('B.bar')
        def __getattr__(self, name):
            return getattr(self._a, name)

当实现代理模式时，还有些细节需要注意。
首先，``__getattr__()`` 实际是一个后备方法，只有在属性不存在时才会调用。
因此，如果代理类实例本身有这个属性的话，那么不会触发这个方法的。
另外，``__setattr__()`` 和 ``__delattr__()`` 需要额外的魔法来区分代理实例和被代理实例 ``_obj`` 的属性。
一个通常的约定是只代理那些不以下划线 ``_`` 开头的属性(代理类只暴露被代理类的公共属性)。

还有一点需要注意的是，``__getattr__()`` 对于大部分以双下划线(__)开始和结尾的属性并不适用。
比如，考虑如下的类：

.. code-block:: python

    class ListLike:
        """__getattr__对于双下划线开始和结尾的方法是不能用的，需要一个个去重定义"""

        def __init__(self):
            self._items = []

        def __getattr__(self, name):
            return getattr(self._items, name)

如果是创建一个ListLike对象，会发现它支持普通的列表方法，如append()和insert()，
但是却不支持len()、元素查找等。例如：

.. code-block:: python

    >>> a = ListLike()
    >>> a.append(2)
    >>> a.insert(0, 1)
    >>> a.sort()
    >>> len(a)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: object of type 'ListLike' has no len()
    >>> a[0]
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: 'ListLike' object does not support indexing
    >>>

为了让它支持这些方法，你必须手动的实现这些方法代理：

.. code-block:: python

    class ListLike:
        """__getattr__对于双下划线开始和结尾的方法是不能用的，需要一个个去重定义"""

        def __init__(self):
            self._items = []

        def __getattr__(self, name):
            return getattr(self._items, name)

        # Added special methods to support certain list operations
        def __len__(self):
            return len(self._items)

        def __getitem__(self, index):
            return self._items[index]

        def __setitem__(self, index, value):
            self._items[index] = value

        def __delitem__(self, index):
            del self._items[index]

11.8小节还有一个在远程方法调用环境中使用代理的例子。
============================
8.16 在类中定义多个构造器
============================

----------
问题
----------
你想实现一个类，除了使用 ``__init__()`` 方法外，还有其他方式可以初始化它。

----------
解决方案
----------
为了实现多个构造器，你需要使用到类方法。例如：

.. code-block:: python

    import time

    class Date:
        """方法一：使用类方法"""
        # Primary constructor
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

        # Alternate constructor
        @classmethod
        def today(cls):
            t = time.localtime()
            return cls(t.tm_year, t.tm_mon, t.tm_mday)

直接调用类方法即可，下面是使用示例：

.. code-block:: python

    a = Date(2012, 12, 21) # Primary
    b = Date.today() # Alternate

----------
讨论
----------
类方法的一个主要用途就是定义多个构造器。它接受一个 ``class`` 作为第一个参数(cls)。
你应该注意到了这个类被用来创建并返回最终的实例。在继承时也能工作的很好：

.. code-block:: python

    class NewDate(Date):
        pass

    c = Date.today() # Creates an instance of Date (cls=Date)
    d = NewDate.today() # Creates an instance of NewDate (cls=NewDate)

============================
8.17 创建不调用init方法的实例
============================

----------
问题
----------
你想创建一个实例，但是希望绕过执行 ``__init__()`` 方法。

----------
解决方案
----------
可以通过 ``__new__()`` 方法创建一个未初始化的实例。例如考虑如下这个类：

.. code-block:: python

    class Date:
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

下面演示如何不调用 ``__init__()`` 方法来创建这个Date实例：

.. code-block:: python

    >>> d = Date.__new__(Date)
    >>> d
    <__main__.Date object at 0x1006716d0>
    >>> d.year
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: 'Date' object has no attribute 'year'
    >>>

结果可以看到，这个Date实例的属性year还不存在，所以你需要手动初始化：

.. code-block:: python

    >>> data = {'year':2012, 'month':8, 'day':29}
    >>> for key, value in data.items():
    ...     setattr(d, key, value)
    ...
    >>> d.year
    2012
    >>> d.month
    8
    >>>

----------
讨论
----------
当我们在反序列对象或者实现某个类方法构造函数时需要绕过 ``__init__()`` 方法来创建对象。
例如，对于上面的Date来讲，有时候你可能会像下面这样定义一个新的构造函数 ``today()`` ：

.. code-block:: python

    from time import localtime

    class Date:
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

        @classmethod
        def today(cls):
            d = cls.__new__(cls)
            t = localtime()
            d.year = t.tm_year
            d.month = t.tm_mon
            d.day = t.tm_mday
            return d

同样，在你反序列化JSON数据时产生一个如下的字典对象：

.. code-block:: python

    data = { 'year': 2012, 'month': 8, 'day': 29 }

如果你想将它转换成一个Date类型实例，可以使用上面的技术。

当你通过这种非常规方式来创建实例的时候，最好不要直接去访问底层实例字典，除非你真的清楚所有细节。
否则的话，如果这个类使用了 ``__slots__`` 、properties 、descriptors 或其他高级技术的时候代码就会失效。
而这时候使用 ``setattr()`` 方法会让你的代码变得更加通用。

============================
8.18 利用Mixins扩展类功能
============================

----------
问题
----------
你有很多有用的方法，想使用它们来扩展其他类的功能。但是这些类并没有任何继承的关系。
因此你不能简单的将这些方法放入一个基类，然后被其他类继承。

----------
解决方案
----------
通常当你想自定义类的时候会碰上这些问题。可能是某个库提供了一些基础类，
你可以利用它们来构造你自己的类。

假设你想扩展映射对象，给它们添加日志、唯一性设置、类型检查等等功能。下面是一些混入类：

.. code-block:: python

    class LoggedMappingMixin:
        """
        Add logging to get/set/delete operations for debugging.
        """
        __slots__ = ()  # 混入类都没有实例变量，因为直接实例化混入类没有任何意义

        def __getitem__(self, key):
            print('Getting ' + str(key))
            return super().__getitem__(key)

        def __setitem__(self, key, value):
            print('Setting {} = {!r}'.format(key, value))
            return super().__setitem__(key, value)

        def __delitem__(self, key):
            print('Deleting ' + str(key))
            return super().__delitem__(key)


    class SetOnceMappingMixin:
        '''
        Only allow a key to be set once.
        '''
        __slots__ = ()

        def __setitem__(self, key, value):
            if key in self:
                raise KeyError(str(key) + ' already set')
            return super().__setitem__(key, value)


    class StringKeysMappingMixin:
        '''
        Restrict keys to strings only
        '''
        __slots__ = ()

        def __setitem__(self, key, value):
            if not isinstance(key, str):
                raise TypeError('keys must be strings')
            return super().__setitem__(key, value)

这些类单独使用起来没有任何意义，事实上如果你去实例化任何一个类，除了产生异常外没任何作用。
它们是用来通过多继承来和其他映射对象混入使用的。例如：

.. code-block:: python

    class LoggedDict(LoggedMappingMixin, dict):
        pass

    d = LoggedDict()
    d['x'] = 23
    print(d['x'])
    del d['x']

    from collections import defaultdict

    class SetOnceDefaultDict(SetOnceMappingMixin, defaultdict):
        pass


    d = SetOnceDefaultDict(list)
    d['x'].append(2)
    d['x'].append(3)
    # d['x'] = 23  # KeyError: 'x already set'

这个例子中，可以看到混入类跟其他已存在的类(比如dict、defaultdict和OrderedDict)结合起来使用，一个接一个。
结合后就能发挥正常功效了。

----------
讨论
----------
混入类在标准库中很多地方都出现过，通常都是用来像上面那样扩展某些类的功能。
它们也是多继承的一个主要用途。比如，当你编写网络代码时候，
你会经常使用 ``socketserver`` 模块中的 ``ThreadingMixIn`` 来给其他网络相关类增加多线程支持。
例如，下面是一个多线程的XML-RPC服务：

.. code-block:: python

    from xmlrpc.server import SimpleXMLRPCServer
    from socketserver import ThreadingMixIn
    class ThreadedXMLRPCServer(ThreadingMixIn, SimpleXMLRPCServer):
        pass

同时在一些大型库和框架中也会发现混入类的使用，用途同样是增强已存在的类的功能和一些可选特征。

对于混入类，有几点需要记住。首先是，混入类不能直接被实例化使用。
其次，混入类没有自己的状态信息，也就是说它们并没有定义 ``__init__()`` 方法，并且没有实例属性。
这也是为什么我们在上面明确定义了 ``__slots__ = ()`` 。

还有一种实现混入类的方式就是使用类装饰器，如下所示：

.. code-block:: python

    def LoggedMapping(cls):
        """第二种方式：使用类装饰器"""
        cls_getitem = cls.__getitem__
        cls_setitem = cls.__setitem__
        cls_delitem = cls.__delitem__

        def __getitem__(self, key):
            print('Getting ' + str(key))
            return cls_getitem(self, key)

        def __setitem__(self, key, value):
            print('Setting {} = {!r}'.format(key, value))
            return cls_setitem(self, key, value)

        def __delitem__(self, key):
            print('Deleting ' + str(key))
            return cls_delitem(self, key)

        cls.__getitem__ = __getitem__
        cls.__setitem__ = __setitem__
        cls.__delitem__ = __delitem__
        return cls


    @LoggedMapping
    class LoggedDict(dict):
        pass

这个效果跟之前的是一样的，而且不再需要使用多继承了。参考9.12小节获取更多类装饰器的信息，
参考8.13小节查看更多混入类和类装饰器的例子。
============================
8.19 实现状态对象或者状态机
============================

----------
问题
----------
你想实现一个状态机或者是在不同状态下执行操作的对象，但是又不想在代码中出现太多的条件判断语句。

----------
解决方案
----------
在很多程序中，有些对象会根据状态的不同来执行不同的操作。比如考虑如下的一个连接对象：

.. code-block:: python

    class Connection:
        """普通方案，好多个判断语句，效率低下~~"""

        def __init__(self):
            self.state = 'CLOSED'

        def read(self):
            if self.state != 'OPEN':
                raise RuntimeError('Not open')
            print('reading')

        def write(self, data):
            if self.state != 'OPEN':
                raise RuntimeError('Not open')
            print('writing')

        def open(self):
            if self.state == 'OPEN':
                raise RuntimeError('Already open')
            self.state = 'OPEN'

        def close(self):
            if self.state == 'CLOSED':
                raise RuntimeError('Already closed')
            self.state = 'CLOSED'

这样写有很多缺点，首先是代码太复杂了，好多的条件判断。其次是执行效率变低，
因为一些常见的操作比如read()、write()每次执行前都需要执行检查。

一个更好的办法是为每个状态定义一个对象：

.. code-block:: python

    class Connection1:
        """新方案——对每个状态定义一个类"""

        def __init__(self):
            self.new_state(ClosedConnectionState)

        def new_state(self, newstate):
            self._state = newstate
            # Delegate to the state class

        def read(self):
            return self._state.read(self)

        def write(self, data):
            return self._state.write(self, data)

        def open(self):
            return self._state.open(self)

        def close(self):
            return self._state.close(self)


    # Connection state base class
    class ConnectionState:
        @staticmethod
        def read(conn):
            raise NotImplementedError()

        @staticmethod
        def write(conn, data):
            raise NotImplementedError()

        @staticmethod
        def open(conn):
            raise NotImplementedError()

        @staticmethod
        def close(conn):
            raise NotImplementedError()


    # Implementation of different states
    class ClosedConnectionState(ConnectionState):
        @staticmethod
        def read(conn):
            raise RuntimeError('Not open')

        @staticmethod
        def write(conn, data):
            raise RuntimeError('Not open')

        @staticmethod
        def open(conn):
            conn.new_state(OpenConnectionState)

        @staticmethod
        def close(conn):
            raise RuntimeError('Already closed')


    class OpenConnectionState(ConnectionState):
        @staticmethod
        def read(conn):
            print('reading')

        @staticmethod
        def write(conn, data):
            print('writing')

        @staticmethod
        def open(conn):
            raise RuntimeError('Already open')

        @staticmethod
        def close(conn):
            conn.new_state(ClosedConnectionState)

下面是使用演示：

.. code-block:: python

    >>> c = Connection()
    >>> c._state
    <class '__main__.ClosedConnectionState'>
    >>> c.read()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example.py", line 10, in read
            return self._state.read(self)
        File "example.py", line 43, in read
            raise RuntimeError('Not open')
    RuntimeError: Not open
    >>> c.open()
    >>> c._state
    <class '__main__.OpenConnectionState'>
    >>> c.read()
    reading
    >>> c.write('hello')
    writing
    >>> c.close()
    >>> c._state
    <class '__main__.ClosedConnectionState'>
    >>>

----------
讨论
----------
如果代码中出现太多的条件判断语句的话，代码就会变得难以维护和阅读。
这里的解决方案是将每个状态抽取出来定义成一个类。

这里看上去有点奇怪，每个状态对象都只有静态方法，并没有存储任何的实例属性数据。
实际上，所有状态信息都只存储在 ``Connection`` 实例中。
在基类中定义的 ``NotImplementedError`` 是为了确保子类实现了相应的方法。
这里你或许还想使用8.12小节讲解的抽象基类方式。

设计模式中有一种模式叫状态模式，这一小节算是一个初步入门！
============================
8.20 通过字符串调用对象方法
============================

----------
问题
----------
你有一个字符串形式的方法名称，想通过它调用某个对象的对应方法。

----------
解决方案
----------
最简单的情况，可以使用 ``getattr()`` ：

.. code-block:: python

    import math

    class Point:
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def __repr__(self):
            return 'Point({!r:},{!r:})'.format(self.x, self.y)

        def distance(self, x, y):
            return math.hypot(self.x - x, self.y - y)


    p = Point(2, 3)
    d = getattr(p, 'distance')(0, 0)  # Calls p.distance(0, 0)

另外一种方法是使用 ``operator.methodcaller()`` ，例如：

.. code-block:: python

    import operator
    operator.methodcaller('distance', 0, 0)(p)

当你需要通过相同的参数多次调用某个方法时，使用 ``operator.methodcaller`` 就很方便了。
比如你需要排序一系列的点，就可以这样做：

.. code-block:: python

    points = [
        Point(1, 2),
        Point(3, 0),
        Point(10, -3),
        Point(-5, -7),
        Point(-1, 8),
        Point(3, 2)
    ]
    # Sort by distance from origin (0, 0)
    points.sort(key=operator.methodcaller('distance', 0, 0))

----------
讨论
----------
调用一个方法实际上是两部独立操作，第一步是查找属性，第二步是函数调用。
因此，为了调用某个方法，你可以首先通过 ``getattr()`` 来查找到这个属性，然后再去以函数方式调用它即可。

``operator.methodcaller()`` 创建一个可调用对象，并同时提供所有必要参数，
然后调用的时候只需要将实例对象传递给它即可，比如：

.. code-block:: python

    >>> p = Point(3, 4)
    >>> d = operator.methodcaller('distance', 0, 0)
    >>> d(p)
    5.0
    >>>

通过方法名称字符串来调用方法通常出现在需要模拟 ``case`` 语句或实现访问者模式的时候。
参考下一小节获取更多高级例子。

============================
8.21 实现访问者模式
============================

----------
问题
----------
你要处理由大量不同类型的对象组成的复杂数据结构，每一个对象都需要需要进行不同的处理。
比如，遍历一个树形结构，然后根据每个节点的相应状态执行不同的操作。

----------
解决方案
----------
这里遇到的问题在编程领域中是很普遍的，有时候会构建一个由大量不同对象组成的数据结构。
假设你要写一个表示数学表达式的程序，那么你可能需要定义如下的类：

.. code-block:: python

    class Node:
        pass

    class UnaryOperator(Node):
        def __init__(self, operand):
            self.operand = operand

    class BinaryOperator(Node):
        def __init__(self, left, right):
            self.left = left
            self.right = right

    class Add(BinaryOperator):
        pass

    class Sub(BinaryOperator):
        pass

    class Mul(BinaryOperator):
        pass

    class Div(BinaryOperator):
        pass

    class Negate(UnaryOperator):
        pass

    class Number(Node):
        def __init__(self, value):
            self.value = value

然后利用这些类构建嵌套数据结构，如下所示：

.. code-block:: python

    # Representation of 1 + 2 * (3 - 4) / 5
    t1 = Sub(Number(3), Number(4))
    t2 = Mul(Number(2), t1)
    t3 = Div(t2, Number(5))
    t4 = Add(Number(1), t3)

这样做的问题是对于每个表达式，每次都要重新定义一遍，有没有一种更通用的方式让它支持所有的数字和操作符呢。
这里我们使用访问者模式可以达到这样的目的：

.. code-block:: python

    class NodeVisitor:
        def visit(self, node):
            methname = 'visit_' + type(node).__name__
            meth = getattr(self, methname, None)
            if meth is None:
                meth = self.generic_visit
            return meth(node)

        def generic_visit(self, node):
            raise RuntimeError('No {} method'.format('visit_' + type(node).__name__))

为了使用这个类，可以定义一个类继承它并且实现各种 ``visit_Name()`` 方法，其中Name是node类型。
例如，如果你想求表达式的值，可以这样写：

.. code-block:: python

    class Evaluator(NodeVisitor):
        def visit_Number(self, node):
            return node.value

        def visit_Add(self, node):
            return self.visit(node.left) + self.visit(node.right)

        def visit_Sub(self, node):
            return self.visit(node.left) - self.visit(node.right)

        def visit_Mul(self, node):
            return self.visit(node.left) * self.visit(node.right)

        def visit_Div(self, node):
            return self.visit(node.left) / self.visit(node.right)

        def visit_Negate(self, node):
            return -node.operand

使用示例：

.. code-block:: python

    >>> e = Evaluator()
    >>> e.visit(t4)
    0.6
    >>>

作为一个不同的例子，下面定义一个类在一个栈上面将一个表达式转换成多个操作序列：

.. code-block:: python

    class StackCode(NodeVisitor):
        def generate_code(self, node):
            self.instructions = []
            self.visit(node)
            return self.instructions

        def visit_Number(self, node):
            self.instructions.append(('PUSH', node.value))

        def binop(self, node, instruction):
            self.visit(node.left)
            self.visit(node.right)
            self.instructions.append((instruction,))

        def visit_Add(self, node):
            self.binop(node, 'ADD')

        def visit_Sub(self, node):
            self.binop(node, 'SUB')

        def visit_Mul(self, node):
            self.binop(node, 'MUL')

        def visit_Div(self, node):
            self.binop(node, 'DIV')

        def unaryop(self, node, instruction):
            self.visit(node.operand)
            self.instructions.append((instruction,))

        def visit_Negate(self, node):
            self.unaryop(node, 'NEG')

使用示例：

.. code-block:: python

    >>> s = StackCode()
    >>> s.generate_code(t4)
    [('PUSH', 1), ('PUSH', 2), ('PUSH', 3), ('PUSH', 4), ('SUB',),
    ('MUL',), ('PUSH', 5), ('DIV',), ('ADD',)]
    >>>

----------
讨论
----------
刚开始的时候你可能会写大量的if/else语句来实现，
这里访问者模式的好处就是通过 ``getattr()`` 来获取相应的方法，并利用递归来遍历所有的节点：

.. code-block:: python

    def binop(self, node, instruction):
        self.visit(node.left)
        self.visit(node.right)
        self.instructions.append((instruction,))

还有一点需要指出的是，这种技术也是实现其他语言中switch或case语句的方式。
比如，如果你正在写一个HTTP框架，你可能会写这样一个请求分发的控制器：

.. code-block:: python

    class HTTPHandler:
        def handle(self, request):
            methname = 'do_' + request.request_method
            getattr(self, methname)(request)
        def do_GET(self, request):
            pass
        def do_POST(self, request):
            pass
        def do_HEAD(self, request):
            pass

访问者模式一个缺点就是它严重依赖递归，如果数据结构嵌套层次太深可能会有问题，
有时候会超过Python的递归深度限制(参考 ``sys.getrecursionlimit()`` )。

可以参照8.22小节，利用生成器或迭代器来实现非递归遍历算法。

在跟解析和编译相关的编程中使用访问者模式是非常常见的。
Python本身的 ``ast`` 模块值的关注下，可以去看看源码。
9.24小节演示了一个利用 ``ast`` 模块来处理Python源代码的例子。

============================
8.22 不用递归实现访问者模式
============================

----------
问题
----------
你使用访问者模式遍历一个很深的嵌套树形数据结构，并且因为超过嵌套层级限制而失败。
你想消除递归，并同时保持访问者编程模式。

----------
解决方案
----------
通过巧妙的使用生成器可以在树遍历或搜索算法中消除递归。
在8.21小节中，我们给出了一个访问者类。
下面我们利用一个栈和生成器重新实现这个类：

.. code-block:: python

    import types

    class Node:
        pass

    class NodeVisitor:
        def visit(self, node):
            stack = [node]
            last_result = None
            while stack:
                try:
                    last = stack[-1]
                    if isinstance(last, types.GeneratorType):
                        stack.append(last.send(last_result))
                        last_result = None
                    elif isinstance(last, Node):
                        stack.append(self._visit(stack.pop()))
                    else:
                        last_result = stack.pop()
                except StopIteration:
                    stack.pop()

            return last_result

        def _visit(self, node):
            methname = 'visit_' + type(node).__name__
            meth = getattr(self, methname, None)
            if meth is None:
                meth = self.generic_visit
            return meth(node)

        def generic_visit(self, node):
            raise RuntimeError('No {} method'.format('visit_' + type(node).__name__))

如果你使用这个类，也能达到相同的效果。事实上你完全可以将它作为上一节中的访问者模式的替代实现。
考虑如下代码，遍历一个表达式的树：

.. code-block:: python

    class UnaryOperator(Node):
        def __init__(self, operand):
            self.operand = operand

    class BinaryOperator(Node):
        def __init__(self, left, right):
            self.left = left
            self.right = right

    class Add(BinaryOperator):
        pass

    class Sub(BinaryOperator):
        pass

    class Mul(BinaryOperator):
        pass

    class Div(BinaryOperator):
        pass

    class Negate(UnaryOperator):
        pass

    class Number(Node):
        def __init__(self, value):
            self.value = value

    # A sample visitor class that evaluates expressions
    class Evaluator(NodeVisitor):
        def visit_Number(self, node):
            return node.value

        def visit_Add(self, node):
            return self.visit(node.left) + self.visit(node.right)

        def visit_Sub(self, node):
            return self.visit(node.left) - self.visit(node.right)

        def visit_Mul(self, node):
            return self.visit(node.left) * self.visit(node.right)

        def visit_Div(self, node):
            return self.visit(node.left) / self.visit(node.right)

        def visit_Negate(self, node):
            return -self.visit(node.operand)

    if __name__ == '__main__':
        # 1 + 2*(3-4) / 5
        t1 = Sub(Number(3), Number(4))
        t2 = Mul(Number(2), t1)
        t3 = Div(t2, Number(5))
        t4 = Add(Number(1), t3)
        # Evaluate it
        e = Evaluator()
        print(e.visit(t4))  # Outputs 0.6

如果嵌套层次太深那么上述的Evaluator就会失效：

.. code-block:: python

    >>> a = Number(0)
    >>> for n in range(1, 100000):
    ... a = Add(a, Number(n))
    ...
    >>> e = Evaluator()
    >>> e.visit(a)
    Traceback (most recent call last):
    ...
        File "visitor.py", line 29, in _visit
    return meth(node)
        File "visitor.py", line 67, in visit_Add
    return self.visit(node.left) + self.visit(node.right)
    RuntimeError: maximum recursion depth exceeded
    >>>

现在我们稍微修改下上面的Evaluator：

.. code-block:: python

    class Evaluator(NodeVisitor):
        def visit_Number(self, node):
            return node.value

        def visit_Add(self, node):
            yield (yield node.left) + (yield node.right)

        def visit_Sub(self, node):
            yield (yield node.left) - (yield node.right)

        def visit_Mul(self, node):
            yield (yield node.left) * (yield node.right)

        def visit_Div(self, node):
            yield (yield node.left) / (yield node.right)

        def visit_Negate(self, node):
            yield - (yield node.operand)

再次运行，就不会报错了：

.. code-block:: python

    >>> a = Number(0)
    >>> for n in range(1,100000):
    ...     a = Add(a, Number(n))
    ...
    >>> e = Evaluator()
    >>> e.visit(a)
    4999950000
    >>>

如果你还想添加其他自定义逻辑也没问题：

.. code-block:: python

    class Evaluator(NodeVisitor):
        ...
        def visit_Add(self, node):
            print('Add:', node)
            lhs = yield node.left
            print('left=', lhs)
            rhs = yield node.right
            print('right=', rhs)
            yield lhs + rhs
        ...

下面是简单的测试：

.. code-block:: python

    >>> e = Evaluator()
    >>> e.visit(t4)
    Add: <__main__.Add object at 0x1006a8d90>
    left= 1
    right= -0.4
    0.6
    >>>

----------
讨论
----------
这一小节我们演示了生成器和协程在程序控制流方面的强大功能。
避免递归的一个通常方法是使用一个栈或队列的数据结构。
例如，深度优先的遍历算法，第一次碰到一个节点时将其压入栈中，处理完后弹出栈。``visit()`` 方法的核心思路就是这样。

另外一个需要理解的就是生成器中yield语句。当碰到yield语句时，生成器会返回一个数据并暂时挂起。
上面的例子使用这个技术来代替了递归。例如，之前我们是这样写递归：

.. code-block:: python

    value = self.visit(node.left)

现在换成yield语句：

.. code-block:: python

    value = yield node.left

它会将 ``node.left`` 返回给 ``visit()`` 方法，然后 ``visit()`` 方法调用那个节点相应的 ``visit_Name()`` 方法。
yield暂时将程序控制器让出给调用者，当执行完后，结果会赋值给value，

看完这一小节，你也许想去寻找其它没有yield语句的方案。但是这么做没有必要，你必须处理很多棘手的问题。
例如，为了消除递归，你必须要维护一个栈结构，如果不使用生成器，代码会变得很臃肿，到处都是栈操作语句、回调函数等。
实际上，使用yield语句可以让你写出非常漂亮的代码，它消除了递归但是看上去又很像递归实现，代码很简洁。
============================
8.23 循环引用数据结构的内存管理
============================

----------
问题
----------
你的程序创建了很多循环引用数据结构(比如树、图、观察者模式等)，你碰到了内存管理难题。

----------
解决方案
----------
一个简单的循环引用数据结构例子就是一个树形结构，双亲节点有指针指向孩子节点，孩子节点又返回来指向双亲节点。
这种情况下，可以考虑使用 ``weakref`` 库中的弱引用。例如：

.. code-block:: python

    import weakref

    class Node:
        def __init__(self, value):
            self.value = value
            self._parent = None
            self.children = []

        def __repr__(self):
            return 'Node({!r:})'.format(self.value)

        # property that manages the parent as a weak-reference
        @property
        def parent(self):
            return None if self._parent is None else self._parent()

        @parent.setter
        def parent(self, node):
            self._parent = weakref.ref(node)

        def add_child(self, child):
            self.children.append(child)
            child.parent = self

这种是想方式允许parent静默终止。例如：

.. code-block:: python

    >>> root = Node('parent')
    >>> c1 = Node('child')
    >>> root.add_child(c1)
    >>> print(c1.parent)
    Node('parent')
    >>> del root
    >>> print(c1.parent)
    None
    >>>

----------
讨论
----------
循环引用的数据结构在Python中是一个很棘手的问题，因为正常的垃圾回收机制不能适用于这种情形。
例如考虑如下代码：

.. code-block:: python

    # Class just to illustrate when deletion occurs
    class Data:
        def __del__(self):
            print('Data.__del__')

    # Node class involving a cycle
    class Node:
        def __init__(self):
            self.data = Data()
            self.parent = None
            self.children = []

        def add_child(self, child):
            self.children.append(child)
            child.parent = self

下面我们使用这个代码来做一些垃圾回收试验：

.. code-block:: python

    >>> a = Data()
    >>> del a # Immediately deleted
    Data.__del__
    >>> a = Node()
    >>> del a # Immediately deleted
    Data.__del__
    >>> a = Node()
    >>> a.add_child(Node())
    >>> del a # Not deleted (no message)
    >>>

可以看到，最后一个的删除时打印语句没有出现。原因是Python的垃圾回收机制是基于简单的引用计数。
当一个对象的引用数变成0的时候才会立即删除掉。而对于循环引用这个条件永远不会成立。
因此，在上面例子中最后部分，父节点和孩子节点互相拥有对方的引用，导致每个对象的引用计数都不可能变成0。

Python有另外的垃圾回收器来专门针对循环引用的，但是你永远不知道它什么时候会触发。
另外你还可以手动的触发它，但是代码看上去很挫：

.. code-block:: python

    >>> import gc
    >>> gc.collect() # Force collection
    Data.__del__
    Data.__del__
    >>>

如果循环引用的对象自己还定义了自己的 ``__del__()`` 方法，那么会让情况变得更糟糕。
假设你像下面这样给Node定义自己的 ``__del__()`` 方法：

.. code-block:: python

    # Node class involving a cycle
    class Node:
        def __init__(self):
            self.data = Data()
            self.parent = None
            self.children = []

        def add_child(self, child):
            self.children.append(child)
            child.parent = self

        # NEVER DEFINE LIKE THIS.
        # Only here to illustrate pathological behavior
        def __del__(self):
            del self.data
            del.parent
            del.children

这种情况下，垃圾回收永远都不会去回收这个对象的，还会导致内存泄露。
如果你试着去运行它会发现，``Data.__del__`` 消息永远不会出现了,甚至在你强制内存回收时：

.. code-block:: python

    >>> a = Node()
    >>> a.add_child(Node()
    >>> del a # No message (not collected)
    >>> import gc
    >>> gc.collect() # No message (not collected)
    >>>

弱引用消除了引用循环的这个问题，本质来讲，弱引用就是一个对象指针，它不会增加它的引用计数。
你可以通过 ``weakref`` 来创建弱引用。例如：

.. code-block:: python

    >>> import weakref
    >>> a = Node()
    >>> a_ref = weakref.ref(a)
    >>> a_ref
    <weakref at 0x100581f70; to 'Node' at 0x1005c5410>
    >>>
为了访问弱引用所引用的对象，你可以像函数一样去调用它即可。如果那个对象还存在就会返回它，否则就返回一个None。
由于原始对象的引用计数没有增加，那么就可以去删除它了。例如;

.. code-block:: python

    >>> print(a_ref())
    <__main__.Node object at 0x1005c5410>
    >>> del a
    Data.__del__
    >>> print(a_ref())
    None
    >>>

通过这里演示的弱引用技术，你会发现不再有循环引用问题了，一旦某个节点不被使用了，垃圾回收器立即回收它。
你还能参考8.25小节关于弱引用的另外一个例子。
============================
8.24 让类支持比较操作
============================

----------
问题
----------
你想让某个类的实例支持标准的比较运算(比如>=,!=,<=,<等)，但是又不想去实现那一大丢的特殊方法。

----------
解决方案
----------
Python类对每个比较操作都需要实现一个特殊方法来支持。
例如为了支持>=操作符，你需要定义一个 ``__ge__()`` 方法。
尽管定义一个方法没什么问题，但如果要你实现所有可能的比较方法那就有点烦人了。

装饰器 ``functools.total_ordering`` 就是用来简化这个处理的。
使用它来装饰一个来，你只需定义一个 ``__eq__()`` 方法，
外加其他方法(__lt__, __le__, __gt__, or __ge__)中的一个即可。
然后装饰器会自动为你填充其它比较方法。

作为例子，我们构建一些房子，然后给它们增加一些房间，最后通过房子大小来比较它们：

.. code-block:: python

    from functools import total_ordering

    class Room:
        def __init__(self, name, length, width):
            self.name = name
            self.length = length
            self.width = width
            self.square_feet = self.length * self.width

    @total_ordering
    class House:
        def __init__(self, name, style):
            self.name = name
            self.style = style
            self.rooms = list()

        @property
        def living_space_footage(self):
            return sum(r.square_feet for r in self.rooms)

        def add_room(self, room):
            self.rooms.append(room)

        def __str__(self):
            return '{}: {} square foot {}'.format(self.name,
                    self.living_space_footage,
                    self.style)

        def __eq__(self, other):
            return self.living_space_footage == other.living_space_footage

        def __lt__(self, other):
            return self.living_space_footage < other.living_space_footage

这里我们只是给House类定义了两个方法：``__eq__()`` 和 ``__lt__()`` ，它就能支持所有的比较操作：

.. code-block:: python

    # Build a few houses, and add rooms to them
    h1 = House('h1', 'Cape')
    h1.add_room(Room('Master Bedroom', 14, 21))
    h1.add_room(Room('Living Room', 18, 20))
    h1.add_room(Room('Kitchen', 12, 16))
    h1.add_room(Room('Office', 12, 12))
    h2 = House('h2', 'Ranch')
    h2.add_room(Room('Master Bedroom', 14, 21))
    h2.add_room(Room('Living Room', 18, 20))
    h2.add_room(Room('Kitchen', 12, 16))
    h3 = House('h3', 'Split')
    h3.add_room(Room('Master Bedroom', 14, 21))
    h3.add_room(Room('Living Room', 18, 20))
    h3.add_room(Room('Office', 12, 16))
    h3.add_room(Room('Kitchen', 15, 17))
    houses = [h1, h2, h3]
    print('Is h1 bigger than h2?', h1 > h2) # prints True
    print('Is h2 smaller than h3?', h2 < h3) # prints True
    print('Is h2 greater than or equal to h1?', h2 >= h1) # Prints False
    print('Which one is biggest?', max(houses)) # Prints 'h3: 1101-square-foot Split'
    print('Which is smallest?', min(houses)) # Prints 'h2: 846-square-foot Ranch'

----------
讨论
----------
其实 ``total_ordering`` 装饰器也没那么神秘。
它就是定义了一个从每个比较支持方法到所有需要定义的其他方法的一个映射而已。
比如你定义了 ``__le__()`` 方法，那么它就被用来构建所有其他的需要定义的那些特殊方法。
实际上就是在类里面像下面这样定义了一些特殊方法：

.. code-block:: python

    class House:
        def __eq__(self, other):
            pass
        def __lt__(self, other):
            pass
        # Methods created by @total_ordering
        __le__ = lambda self, other: self < other or self == other
        __gt__ = lambda self, other: not (self < other or self == other)
        __ge__ = lambda self, other: not (self < other)
        __ne__ = lambda self, other: not self == other

当然，你自己去写也很容易，但是使用 ``@total_ordering`` 可以简化代码，何乐而不为呢。
============================
8.25 创建缓存实例
============================

----------
问题
----------
在创建一个类的对象时，如果之前使用同样参数创建过这个对象， 你想返回它的缓存引用。

----------
解决方案
----------
这种通常是因为你希望相同参数创建的对象时单例的。
在很多库中都有实际的例子，比如 ``logging`` 模块，使用相同的名称创建的 ``logger`` 实例永远只有一个。例如：

.. code-block:: python

    >>> import logging
    >>> a = logging.getLogger('foo')
    >>> b = logging.getLogger('bar')
    >>> a is b
    False
    >>> c = logging.getLogger('foo')
    >>> a is c
    True
    >>>

为了达到这样的效果，你需要使用一个和类本身分开的工厂函数，例如：

.. code-block:: python

    # The class in question
    class Spam:
        def __init__(self, name):
            self.name = name

    # Caching support
    import weakref
    _spam_cache = weakref.WeakValueDictionary()
    def get_spam(name):
        if name not in _spam_cache:
            s = Spam(name)
            _spam_cache[name] = s
        else:
            s = _spam_cache[name]
        return s

然后做一个测试，你会发现跟之前那个日志对象的创建行为是一致的：

.. code-block:: python

    >>> a = get_spam('foo')
    >>> b = get_spam('bar')
    >>> a is b
    False
    >>> c = get_spam('foo')
    >>> a is c
    True
    >>>

----------
讨论
----------
编写一个工厂函数来修改普通的实例创建行为通常是一个比较简单的方法。
但是我们还能否找到更优雅的解决方案呢？

例如，你可能会考虑重新定义类的 ``__new__()`` 方法，就像下面这样：

.. code-block:: python

    # Note: This code doesn't quite work
    import weakref

    class Spam:
        _spam_cache = weakref.WeakValueDictionary()
        def __new__(cls, name):
            if name in cls._spam_cache:
                return cls._spam_cache[name]
            else:
                self = super().__new__(cls)
                cls._spam_cache[name] = self
                return self
        def __init__(self, name):
            print('Initializing Spam')
            self.name = name

初看起来好像可以达到预期效果，但是问题是 ``__init__()`` 每次都会被调用，不管这个实例是否被缓存了。例如：

.. code-block:: python

    >>> s = Spam('Dave')
    Initializing Spam
    >>> t = Spam('Dave')
    Initializing Spam
    >>> s is t
    True
    >>>

这个或许不是你想要的效果，因此这种方法并不可取。

上面我们使用到了弱引用计数，对于垃圾回收来讲是很有帮助的，关于这个我们在8.23小节已经讲过了。
当我们保持实例缓存时，你可能只想在程序中使用到它们时才保存。
一个 ``WeakValueDictionary`` 实例只会保存那些在其它地方还在被使用的实例。
否则的话，只要实例不再被使用了，它就从字典中被移除了。观察下下面的测试结果：

.. code-block:: python

    >>> a = get_spam('foo')
    >>> b = get_spam('bar')
    >>> c = get_spam('foo')
    >>> list(_spam_cache)
    ['foo', 'bar']
    >>> del a
    >>> del c
    >>> list(_spam_cache)
    ['bar']
    >>> del b
    >>> list(_spam_cache)
    []
    >>>

对于大部分程序而已，这里代码已经够用了。不过还是有一些更高级的实现值得了解下。

首先是这里使用到了一个全局变量，并且工厂函数跟类放在一块。我们可以通过将缓存代码放到一个单独的缓存管理器中：

.. code-block:: python

    import weakref

    class CachedSpamManager:
        def __init__(self):
            self._cache = weakref.WeakValueDictionary()

        def get_spam(self, name):
            if name not in self._cache:
                s = Spam(name)
                self._cache[name] = s
            else:
                s = self._cache[name]
            return s

        def clear(self):
                self._cache.clear()

    class Spam:
        manager = CachedSpamManager()
        def __init__(self, name):
            self.name = name

        def get_spam(name):
            return Spam.manager.get_spam(name)

这样的话代码更清晰，并且也更灵活，我们可以增加更多的缓存管理机制，只需要替代manager即可。

还有一点就是，我们暴露了类的实例化给用户，用户很容易去直接实例化这个类，而不是使用工厂方法，如：

.. code-block:: python

    >>> a = Spam('foo')
    >>> b = Spam('foo')
    >>> a is b
    False
    >>>

有几种方式可以防止用户这样做，第一个是将类的名字修改为以下划线(_)开头，提示用户别直接调用它。
第二种就是让这个类的 ``__init__()`` 方法抛出一个异常，让它不能被初始化：

.. code-block:: python

    class Spam:
        def __init__(self, *args, **kwargs):
            raise RuntimeError("Can't instantiate directly")

        # Alternate constructor
        @classmethod
        def _new(cls, name):
            self = cls.__new__(cls)
            self.name = name

然后修改缓存管理器代码，使用 ``Spam._new()`` 来创建实例，而不是直接调用 ``Spam()`` 构造函数：

.. code-block:: python

    # ------------------------最后的修正方案------------------------
    class CachedSpamManager2:
        def __init__(self):
            self._cache = weakref.WeakValueDictionary()

        def get_spam(self, name):
            if name not in self._cache:
                temp = Spam3._new(name)  # Modified creation
                self._cache[name] = temp
            else:
                temp = self._cache[name]
            return temp

        def clear(self):
                self._cache.clear()

    class Spam3:
        def __init__(self, *args, **kwargs):
            raise RuntimeError("Can't instantiate directly")

        # Alternate constructor
        @classmethod
        def _new(cls, name):
            self = cls.__new__(cls)
            self.name = name
            return self

最后这样的方案就已经足够好了。
缓存和其他构造模式还可以使用9.13小节中的元类实现的更优雅一点(使用了更高级的技术)。
============================
9.1 在函数上添加包装器
============================

----------
问题
----------
你想在函数上添加一个包装器，增加额外的操作处理(比如日志、计时等)。

----------
解决方案
----------
如果你想使用额外的代码包装一个函数，可以定义一个装饰器函数，例如：

.. code-block:: python

    import time
    from functools import wraps

    def timethis(func):
        '''
        Decorator that reports the execution time.
        '''
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            end = time.time()
            print(func.__name__, end-start)
            return result
        return wrapper

下面是使用装饰器的例子：

.. code-block:: python

    >>> @timethis
    ... def countdown(n):
    ...     '''
    ...     Counts down
    ...     '''
    ...     while n > 0:
    ...         n -= 1
    ...
    >>> countdown(100000)
    countdown 0.008917808532714844
    >>> countdown(10000000)
    countdown 0.87188299392912
    >>>

----------
讨论
----------
一个装饰器就是一个函数，它接受一个函数作为参数并返回一个新的函数。
当你像下面这样写：

.. code-block:: python

    @timethis
    def countdown(n):
        pass

跟像下面这样写其实效果是一样的：

.. code-block:: python

    def countdown(n):
        pass
    countdown = timethis(countdown)

顺便说一下，内置的装饰器比如 ``@staticmethod, @classmethod,@property`` 原理也是一样的。
例如，下面这两个代码片段是等价的：

.. code-block:: python

    class A:
        @classmethod
        def method(cls):
            pass

    class B:
        # Equivalent definition of a class method
        def method(cls):
            pass
        method = classmethod(method)

在上面的 ``wrapper()`` 函数中，
装饰器内部定义了一个使用 ``*args`` 和  ``**kwargs`` 来接受任意参数的函数。
在这个函数里面调用了原始函数并将其结果返回，不过你还可以添加其他额外的代码(比如计时)。
然后这个新的函数包装器被作为结果返回来代替原始函数。

需要强调的是装饰器并不会修改原始函数的参数签名以及返回值。
使用 ``*args`` 和  ``**kwargs`` 目的就是确保任何参数都能适用。
而返回结果值基本都是调用原始函数 ``func(*args, **kwargs)`` 的返回结果，其中func就是原始函数。

刚开始学习装饰器的时候，会使用一些简单的例子来说明，比如上面演示的这个。
不过实际场景使用时，还是有一些细节问题要注意的。
比如上面使用 ``@wraps(func)`` 注解是很重要的，
它能保留原始函数的元数据(下一小节会讲到)，新手经常会忽略这个细节。
接下来的几个小节我们会更加深入的讲解装饰器函数的细节问题，如果你想构造你自己的装饰器函数，需要认真看一下。

============================
9.2 创建装饰器时保留函数元信息
============================

----------
问题
----------
你写了一个装饰器作用在某个函数上，但是这个函数的重要的元信息比如名字、文档字符串、注解和参数签名都丢失了。

----------
解决方案
----------
任何时候你定义装饰器的时候，都应该使用 ``functools`` 库中的 ``@wraps`` 装饰器来注解底层包装函数。例如：

.. code-block:: python

    import time
    from functools import wraps
    def timethis(func):
        '''
        Decorator that reports the execution time.
        '''
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            end = time.time()
            print(func.__name__, end-start)
            return result
        return wrapper

下面我们使用这个被包装后的函数并检查它的元信息：

.. code-block:: python

    >>> @timethis
    ... def countdown(n):
    ...     '''
    ...     Counts down
    ...     '''
    ...     while n > 0:
    ...         n -= 1
    ...
    >>> countdown(100000)
    countdown 0.008917808532714844
    >>> countdown.__name__
    'countdown'
    >>> countdown.__doc__
    '\n\tCounts down\n\t'
    >>> countdown.__annotations__
    {'n': <class 'int'>}
    >>>

----------
讨论
----------
在编写装饰器的时候复制元信息是一个非常重要的部分。如果你忘记了使用 ``@wraps`` ，
那么你会发现被装饰函数丢失了所有有用的信息。比如如果忽略 ``@wraps`` 后的效果是下面这样的：

.. code-block:: python

    >>> countdown.__name__
    'wrapper'
    >>> countdown.__doc__
    >>> countdown.__annotations__
    {}
    >>>

``@wraps`` 有一个重要特征是它能让你通过属性 ``__wrapped__`` 直接访问被包装函数。例如:

.. code-block:: python

    >>> countdown.__wrapped__(100000)
    >>>

``__wrapped__`` 属性还能让被装饰函数正确暴露底层的参数签名信息。例如：

.. code-block:: python

    >>> from inspect import signature
    >>> print(signature(countdown))
    (n:int)
    >>>

一个很普遍的问题是怎样让装饰器去直接复制原始函数的参数签名信息，
如果想自己手动实现的话需要做大量的工作，最好就简单的使用 ``@wraps`` 装饰器。
通过底层的 ``__wrapped__`` 属性访问到函数签名信息。更多关于签名的内容可以参考9.16小节。

============================
9.3 解除一个装饰器
============================

----------
问题
----------
一个装饰器已经作用在一个函数上，你想撤销它，直接访问原始的未包装的那个函数。

----------
解决方案
----------
假设装饰器是通过 ``@wraps`` (参考9.2小节)来实现的，那么你可以通过访问 ``__wrapped__`` 属性来访问原始函数：

.. code-block:: python

    >>> @somedecorator
    >>> def add(x, y):
    ...     return x + y
    ...
    >>> orig_add = add.__wrapped__
    >>> orig_add(3, 4)
    7
    >>>

----------
讨论
----------
直接访问未包装的原始函数在调试、内省和其他函数操作时是很有用的。
但是我们这里的方案仅仅适用于在包装器中正确使用了 ``@wraps`` 或者直接设置了 ``__wrapped__`` 属性的情况。

如果有多个包装器，那么访问 ``__wrapped__`` 属性的行为是不可预知的，应该避免这样做。
在Python3.3中，它会略过所有的包装层，比如，假如你有如下的代码：

.. code-block:: python

    from functools import wraps

    def decorator1(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print('Decorator 1')
            return func(*args, **kwargs)
        return wrapper

    def decorator2(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print('Decorator 2')
            return func(*args, **kwargs)
        return wrapper

    @decorator1
    @decorator2
    def add(x, y):
        return x + y

下面我们在Python3.3下测试：

.. code-block:: python

    >>> add(2, 3)
    Decorator 1
    Decorator 2
    5
    >>> add.__wrapped__(2, 3)
    5
    >>>

下面我们在Python3.4下测试：

.. code-block:: python

    >>> add(2, 3)
    Decorator 1
    Decorator 2
    5
    >>> add.__wrapped__(2, 3)
    Decorator 2
    5
    >>>

最后要说的是，并不是所有的装饰器都使用了 ``@wraps`` ，因此这里的方案并不全部适用。
特别的，内置的装饰器 ``@staticmethod`` 和 ``@classmethod`` 就没有遵循这个约定
(它们把原始函数存储在属性 ``__func__`` 中)。
============================
9.4 定义一个带参数的装饰器
============================

----------
问题
----------
你想定义一个可以接受参数的装饰器

----------
解决方案
----------
我们用一个例子详细阐述下接受参数的处理过程。
假设你想写一个装饰器，给函数添加日志功能，同时允许用户指定日志的级别和其他的选项。
下面是这个装饰器的定义和使用示例：

.. code-block:: python

    from functools import wraps
    import logging

    def logged(level, name=None, message=None):
        """
        Add logging to a function. level is the logging
        level, name is the logger name, and message is the
        log message. If name and message aren't specified,
        they default to the function's module and name.
        """
        def decorate(func):
            logname = name if name else func.__module__
            log = logging.getLogger(logname)
            logmsg = message if message else func.__name__

            @wraps(func)
            def wrapper(*args, **kwargs):
                log.log(level, logmsg)
                return func(*args, **kwargs)
            return wrapper
        return decorate

    # Example use
    @logged(logging.DEBUG)
    def add(x, y):
        return x + y

    @logged(logging.CRITICAL, 'example')
    def spam():
        print('Spam!')

初看起来，这种实现看上去很复杂，但是核心思想很简单。
最外层的函数 ``logged()`` 接受参数并将它们作用在内部的装饰器函数上面。
内层的函数 ``decorate()`` 接受一个函数作为参数，然后在函数上面放置一个包装器。
这里的关键点是包装器是可以使用传递给 ``logged()`` 的参数的。

----------
讨论
----------
定义一个接受参数的包装器看上去比较复杂主要是因为底层的调用序列。特别的，如果你有下面这个代码：

.. code-block:: python

    @decorator(x, y, z)
    def func(a, b):
        pass

装饰器处理过程跟下面的调用是等效的;

.. code-block:: python

    def func(a, b):
        pass
    func = decorator(x, y, z)(func)

``decorator(x, y, z)`` 的返回结果必须是一个可调用对象，它接受一个函数作为参数并包装它，
可以参考9.7小节中另外一个可接受参数的包装器例子。
============================
9.5 可自定义属性的装饰器
============================

----------
问题
----------
你想写一个装饰器来包装一个函数，并且允许用户提供参数在运行时控制装饰器行为。

----------
解决方案
----------
引入一个访问函数，使用 ``nonlocal`` 来修改内部变量。
然后这个访问函数被作为一个属性赋值给包装函数。

.. code-block:: python

    from functools import wraps, partial
    import logging
    # Utility decorator to attach a function as an attribute of obj
    def attach_wrapper(obj, func=None):
        if func is None:
            return partial(attach_wrapper, obj)
        setattr(obj, func.__name__, func)
        return func

    def logged(level, name=None, message=None):
        '''
        Add logging to a function. level is the logging
        level, name is the logger name, and message is the
        log message. If name and message aren't specified,
        they default to the function's module and name.
        '''
        def decorate(func):
            logname = name if name else func.__module__
            log = logging.getLogger(logname)
            logmsg = message if message else func.__name__

            @wraps(func)
            def wrapper(*args, **kwargs):
                log.log(level, logmsg)
                return func(*args, **kwargs)

            # Attach setter functions
            @attach_wrapper(wrapper)
            def set_level(newlevel):
                nonlocal level
                level = newlevel

            @attach_wrapper(wrapper)
            def set_message(newmsg):
                nonlocal logmsg
                logmsg = newmsg

            return wrapper

        return decorate

    # Example use
    @logged(logging.DEBUG)
    def add(x, y):
        return x + y

    @logged(logging.CRITICAL, 'example')
    def spam():
        print('Spam!')

下面是交互环境下的使用例子：

.. code-block:: python

    >>> import logging
    >>> logging.basicConfig(level=logging.DEBUG)
    >>> add(2, 3)
    DEBUG:__main__:add
    5
    >>> # Change the log message
    >>> add.set_message('Add called')
    >>> add(2, 3)
    DEBUG:__main__:Add called
    5
    >>> # Change the log level
    >>> add.set_level(logging.WARNING)
    >>> add(2, 3)
    WARNING:__main__:Add called
    5
    >>>

----------
讨论
----------
这一小节的关键点在于访问函数(如 ``set_message()`` 和 ``set_level()`` )，它们被作为属性赋给包装器。
每个访问函数允许使用 ``nonlocal`` 来修改函数内部的变量。

还有一个令人吃惊的地方是访问函数会在多层装饰器间传播(如果你的装饰器都使用了 ``@functools.wraps`` 注解)。
例如，假设你引入另外一个装饰器，比如9.2小节中的 ``@timethis`` ，像下面这样：

.. code-block:: python

    @timethis
    @logged(logging.DEBUG)
    def countdown(n):
        while n > 0:
            n -= 1

你会发现访问函数依旧有效：

.. code-block:: python

    >>> countdown(10000000)
    DEBUG:__main__:countdown
    countdown 0.8198461532592773
    >>> countdown.set_level(logging.WARNING)
    >>> countdown.set_message("Counting down to zero")
    >>> countdown(10000000)
    WARNING:__main__:Counting down to zero
    countdown 0.8225970268249512
    >>>

你还会发现即使装饰器像下面这样以相反的方向排放，效果也是一样的：

.. code-block:: python

    @logged(logging.DEBUG)
    @timethis
    def countdown(n):
        while n > 0:
            n -= 1

还能通过使用lambda表达式代码来让访问函数的返回不同的设定值：

.. code-block:: python

    @attach_wrapper(wrapper)
    def get_level():
        return level

    # Alternative
    wrapper.get_level = lambda: level

一个比较难理解的地方就是对于访问函数的首次使用。例如，你可能会考虑另外一个方法直接访问函数的属性，如下：

.. code-block:: python

    @wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.log.log(wrapper.level, wrapper.logmsg)
        return func(*args, **kwargs)

    # Attach adjustable attributes
    wrapper.level = level
    wrapper.logmsg = logmsg
    wrapper.log = log

这个方法也可能正常工作，但前提是它必须是最外层的装饰器才行。
如果它的上面还有另外的装饰器(比如上面提到的 ``@timethis`` 例子)，那么它会隐藏底层属性，使得修改它们没有任何作用。
而通过使用访问函数就能避免这样的局限性。

最后提一点，这一小节的方案也可以作为9.9小节中装饰器类的另一种实现方法。

============================
9.6 带可选参数的装饰器
============================

----------
问题
----------
你想写一个装饰器，既可以不传参数给它，比如 ``@decorator`` ，
也可以传递可选参数给它，比如 ``@decorator(x,y,z)`` 。

----------
解决方案
----------
下面是9.5小节中日志装饰器的一个修改版本：

.. code-block:: python

    from functools import wraps, partial
    import logging

    def logged(func=None, *, level=logging.DEBUG, name=None, message=None):
        if func is None:
            return partial(logged, level=level, name=name, message=message)

        logname = name if name else func.__module__
        log = logging.getLogger(logname)
        logmsg = message if message else func.__name__

        @wraps(func)
        def wrapper(*args, **kwargs):
            log.log(level, logmsg)
            return func(*args, **kwargs)

        return wrapper

    # Example use
    @logged
    def add(x, y):
        return x + y

    @logged(level=logging.CRITICAL, name='example')
    def spam():
        print('Spam!')

可以看到，``@logged`` 装饰器可以同时不带参数或带参数。

----------
讨论
----------
这里提到的这个问题就是通常所说的编程一致性问题。
当我们使用装饰器的时候，大部分程序员习惯了要么不给它们传递任何参数，要么给它们传递确切参数。
其实从技术上来讲，我们可以定义一个所有参数都是可选的装饰器，就像下面这样：

.. code-block:: python

    @logged()
    def add(x, y):
        return x+y

但是，这种写法并不符合我们的习惯，有时候程序员忘记加上后面的括号会导致错误。
这里我们向你展示了如何以一致的编程风格来同时满足没有括号和有括号两种情况。

为了理解代码是如何工作的，你需要非常熟悉装饰器是如何作用到函数上以及它们的调用规则。
对于一个像下面这样的简单装饰器：

.. code-block:: python

    # Example use
    @logged
    def add(x, y):
        return x + y

这个调用序列跟下面等价：

.. code-block:: python

    def add(x, y):
        return x + y

    add = logged(add)

这时候，被装饰函数会被当做第一个参数直接传递给 ``logged`` 装饰器。
因此，``logged()`` 中的第一个参数就是被包装函数本身。所有其他参数都必须有默认值。

而对于一个下面这样有参数的装饰器：

.. code-block:: python

    @logged(level=logging.CRITICAL, name='example')
    def spam():
        print('Spam!')

调用序列跟下面等价：

.. code-block:: python

    def spam():
        print('Spam!')
    spam = logged(level=logging.CRITICAL, name='example')(spam)

初始调用 ``logged()`` 函数时，被包装函数并没有传递进来。
因此在装饰器内，它必须是可选的。这个反过来会迫使其他参数必须使用关键字来指定。
并且，但这些参数被传递进来后，装饰器要返回一个接受一个函数参数并包装它的函数(参考9.5小节)。
为了这样做，我们使用了一个技巧，就是利用 ``functools.partial`` 。
它会返回一个未完全初始化的自身，除了被包装函数外其他参数都已经确定下来了。
可以参考7.8小节获取更多 ``partial()`` 方法的知识。
============================
9.7 利用装饰器强制函数上的类型检查
============================

----------
问题
----------
作为某种编程规约，你想在对函数参数进行强制类型检查。

----------
解决方案
----------
在演示实际代码前，先说明我们的目标：能对函数参数类型进行断言，类似下面这样：

.. code-block:: python

    >>> @typeassert(int, int)
    ... def add(x, y):
    ...     return x + y
    ...
    >>>
    >>> add(2, 3)
    5
    >>> add(2, 'hello')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "contract.py", line 33, in wrapper
    TypeError: Argument y must be <class 'int'>
    >>>

下面是使用装饰器技术来实现 ``@typeassert`` ：

.. code-block:: python

    from inspect import signature
    from functools import wraps

    def typeassert(*ty_args, **ty_kwargs):
        def decorate(func):
            # If in optimized mode, disable type checking
            if not __debug__:
                return func

            # Map function argument names to supplied types
            sig = signature(func)
            bound_types = sig.bind_partial(*ty_args, **ty_kwargs).arguments

            @wraps(func)
            def wrapper(*args, **kwargs):
                bound_values = sig.bind(*args, **kwargs)
                # Enforce type assertions across supplied arguments
                for name, value in bound_values.arguments.items():
                    if name in bound_types:
                        if not isinstance(value, bound_types[name]):
                            raise TypeError(
                                'Argument {} must be {}'.format(name, bound_types[name])
                                )
                return func(*args, **kwargs)
            return wrapper
        return decorate

可以看出这个装饰器非常灵活，既可以指定所有参数类型，也可以只指定部分。
并且可以通过位置或关键字来指定参数类型。下面是使用示例：

.. code-block:: python

    >>> @typeassert(int, z=int)
    ... def spam(x, y, z=42):
    ...     print(x, y, z)
    ...
    >>> spam(1, 2, 3)
    1 2 3
    >>> spam(1, 'hello', 3)
    1 hello 3
    >>> spam(1, 'hello', 'world')
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "contract.py", line 33, in wrapper
    TypeError: Argument z must be <class 'int'>
    >>>

----------
讨论
----------
这节是高级装饰器示例，引入了很多重要的概念。

首先，装饰器只会在函数定义时被调用一次。
有时候你去掉装饰器的功能，那么你只需要简单的返回被装饰函数即可。
下面的代码中，如果全局变量　``__debug__`` 被设置成了False(当你使用-O或-OO参数的优化模式执行程序时)，
那么就直接返回未修改过的函数本身：

.. code-block:: python

    def decorate(func):
        # If in optimized mode, disable type checking
        if not __debug__:
            return func

其次，这里还对被包装函数的参数签名进行了检查，我们使用了 ``inspect.signature()`` 函数。
简单来讲，它运行你提取一个可调用对象的参数签名信息。例如：

.. code-block:: python

    >>> from inspect import signature
    >>> def spam(x, y, z=42):
    ...     pass
    ...
    >>> sig = signature(spam)
    >>> print(sig)
    (x, y, z=42)
    >>> sig.parameters
    mappingproxy(OrderedDict([('x', <Parameter at 0x10077a050 'x'>),
    ('y', <Parameter at 0x10077a158 'y'>), ('z', <Parameter at 0x10077a1b0 'z'>)]))
    >>> sig.parameters['z'].name
    'z'
    >>> sig.parameters['z'].default
    42
    >>> sig.parameters['z'].kind
    <_ParameterKind: 'POSITIONAL_OR_KEYWORD'>
    >>>

装饰器的开始部分，我们使用了 ``bind_partial()`` 方法来执行从指定类型到名称的部分绑定。
下面是例子演示：

.. code-block:: python

    >>> bound_types = sig.bind_partial(int,z=int)
    >>> bound_types
    <inspect.BoundArguments object at 0x10069bb50>
    >>> bound_types.arguments
    OrderedDict([('x', <class 'int'>), ('z', <class 'int'>)])
    >>>

在这个部分绑定中，你可以注意到缺失的参数被忽略了(比如并没有对y进行绑定)。
不过最重要的是创建了一个有序字典 ``bound_types.arguments`` 。
这个字典会将参数名以函数签名中相同顺序映射到指定的类型值上面去。
在我们的装饰器例子中，这个映射包含了我们要强制指定的类型断言。

在装饰器创建的实际包装函数中使用到了 ``sig.bind()`` 方法。
``bind()`` 跟 ``bind_partial()`` 类似，但是它不允许忽略任何参数。因此有了下面的结果：

.. code-block:: python

    >>> bound_values = sig.bind(1, 2, 3)
    >>> bound_values.arguments
    OrderedDict([('x', 1), ('y', 2), ('z', 3)])
    >>>

使用这个映射我们可以很轻松的实现我们的强制类型检查：

.. code-block:: python

    >>> for name, value in bound_values.arguments.items():
    ...     if name in bound_types.arguments:
    ...         if not isinstance(value, bound_types.arguments[name]):
    ...             raise TypeError()
    ...
    >>>

不过这个方案还有点小瑕疵，它对于有默认值的参数并不适用。
比如下面的代码可以正常工作，尽管items的类型是错误的：

.. code-block:: python

    >>> @typeassert(int, list)
    ... def bar(x, items=None):
    ...     if items is None:
    ...         items = []
    ...     items.append(x)
    ...     return items
    >>> bar(2)
    [2]
    >>> bar(2,3)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "contract.py", line 33, in wrapper
    TypeError: Argument items must be <class 'list'>
    >>> bar(4, [1, 2, 3])
    [1, 2, 3, 4]
    >>>

最后一点是关于适用装饰器参数和函数注解之间的争论。
例如，为什么不像下面这样写一个装饰器来查找函数中的注解呢？

.. code-block:: python

    @typeassert
    def spam(x:int, y, z:int = 42):
        print(x,y,z)

一个可能的原因是如果使用了函数参数注解，那么就被限制了。
如果注解被用来做类型检查就不能做其他事情了。而且 ``@typeassert`` 不能再用于使用注解做其他事情的函数了。
而使用上面的装饰器参数灵活性大多了，也更加通用。

可以在PEP 362以及 ``inspect`` 模块中找到更多关于函数参数对象的信息。在9.16小节还有另外一个例子。
============================
9.8 将装饰器定义为类的一部分
============================

----------
问题
----------
你想在类中定义装饰器，并将其作用在其他函数或方法上。

----------
解决方案
----------
在类里面定义装饰器很简单，但是你首先要确认它的使用方式。比如到底是作为一个实例方法还是类方法。
下面我们用例子来阐述它们的不同：

.. code-block:: python

    from functools import wraps

    class A:
        # Decorator as an instance method
        def decorator1(self, func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                print('Decorator 1')
                return func(*args, **kwargs)
            return wrapper

        # Decorator as a class method
        @classmethod
        def decorator2(cls, func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                print('Decorator 2')
                return func(*args, **kwargs)
            return wrapper

下面是一使用例子：

.. code-block:: python

    # As an instance method
    a = A()
    @a.decorator1
    def spam():
        pass
    # As a class method
    @A.decorator2
    def grok():
        pass

仔细观察可以发现一个是实例调用，一个是类调用。

----------
讨论
----------
在类中定义装饰器初看上去好像很奇怪，但是在标准库中有很多这样的例子。
特别的，``@property`` 装饰器实际上是一个类，它里面定义了三个方法 ``getter(), setter(), deleter()`` ,
每一个方法都是一个装饰器。例如：

.. code-block:: python

    class Person:
        # Create a property instance
        first_name = property()

        # Apply decorator methods
        @first_name.getter
        def first_name(self):
            return self._first_name

        @first_name.setter
        def first_name(self, value):
            if not isinstance(value, str):
                raise TypeError('Expected a string')
            self._first_name = value

它为什么要这么定义的主要原因是各种不同的装饰器方法会在关联的 ``property`` 实例上操作它的状态。
因此，任何时候只要你碰到需要在装饰器中记录或绑定信息，那么这不失为一种可行方法。

在类中定义装饰器有个难理解的地方就是对于额外参数 ``self`` 或 ``cls`` 的正确使用。
尽管最外层的装饰器函数比如 ``decorator1()`` 或 ``decorator2()`` 需要提供一个 ``self`` 或 ``cls`` 参数，
但是在两个装饰器内部被创建的 ``wrapper()`` 函数并不需要包含这个 ``self`` 参数。
你唯一需要这个参数是在你确实要访问包装器中这个实例的某些部分的时候。其他情况下都不用去管它。

对于类里面定义的包装器还有一点比较难理解，就是在涉及到继承的时候。
例如，假设你想让在A中定义的装饰器作用在子类B中。你需要像下面这样写：

.. code-block:: python

    class B(A):
        @A.decorator2
        def bar(self):
            pass

也就是说，装饰器要被定义成类方法并且你必须显式的使用父类名去调用它。
你不能使用 ``@B.decorator2`` ，因为在方法定义时，这个类B还没有被创建。
============================
9.9 将装饰器定义为类
============================

----------
问题
----------
你想使用一个装饰器去包装函数，但是希望返回一个可调用的实例。
你需要让你的装饰器可以同时工作在类定义的内部和外部。

----------
解决方案
----------
为了将装饰器定义成一个实例，你需要确保它实现了 ``__call__()`` 和 ``__get__()`` 方法。
例如，下面的代码定义了一个类，它在其他函数上放置一个简单的记录层：

.. code-block:: python

    import types
    from functools import wraps

    class Profiled:
        def __init__(self, func):
            wraps(func)(self)
            self.ncalls = 0

        def __call__(self, *args, **kwargs):
            self.ncalls += 1
            return self.__wrapped__(*args, **kwargs)

        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                return types.MethodType(self, instance)

你可以将它当做一个普通的装饰器来使用，在类里面或外面都可以：

.. code-block:: python

    @Profiled
    def add(x, y):
        return x + y

    class Spam:
        @Profiled
        def bar(self, x):
            print(self, x)

在交互环境中的使用示例：

.. code-block:: python

    >>> add(2, 3)
    5
    >>> add(4, 5)
    9
    >>> add.ncalls
    2
    >>> s = Spam()
    >>> s.bar(1)
    <__main__.Spam object at 0x10069e9d0> 1
    >>> s.bar(2)
    <__main__.Spam object at 0x10069e9d0> 2
    >>> s.bar(3)
    <__main__.Spam object at 0x10069e9d0> 3
    >>> Spam.bar.ncalls
    3

----------
讨论
----------
将装饰器定义成类通常是很简单的。但是这里还是有一些细节需要解释下，特别是当你想将它作用在实例方法上的时候。

首先，使用 ``functools.wraps()`` 函数的作用跟之前还是一样，将被包装函数的元信息复制到可调用实例中去。

其次，通常很容易会忽视上面的 ``__get__()`` 方法。如果你忽略它，保持其他代码不变再次运行，
你会发现当你去调用被装饰实例方法时出现很奇怪的问题。例如：

.. code-block:: python

    >>> s = Spam()
    >>> s.bar(3)
    Traceback (most recent call last):
    ...
    TypeError: bar() missing 1 required positional argument: 'x'

出错原因是当方法函数在一个类中被查找时，它们的 ``__get__()`` 方法依据描述器协议被调用，
在8.9小节已经讲述过描述器协议了。在这里，``__get__()`` 的目的是创建一个绑定方法对象
(最终会给这个方法传递self参数)。下面是一个例子来演示底层原理：

.. code-block:: python

    >>> s = Spam()
    >>> def grok(self, x):
    ...     pass
    ...
    >>> grok.__get__(s, Spam)
    <bound method Spam.grok of <__main__.Spam object at 0x100671e90>>
    >>>

``__get__()`` 方法是为了确保绑定方法对象能被正确的创建。
``type.MethodType()`` 手动创建一个绑定方法来使用。只有当实例被使用的时候绑定方法才会被创建。
如果这个方法是在类上面来访问，
那么 ``__get__()`` 中的instance参数会被设置成None并直接返回 ``Profiled`` 实例本身。
这样的话我们就可以提取它的 ``ncalls`` 属性了。

如果你想避免一些混乱，也可以考虑另外一个使用闭包和 ``nonlocal`` 变量实现的装饰器，这个在9.5小节有讲到。例如：

.. code-block:: python

    import types
    from functools import wraps

    def profiled(func):
        ncalls = 0
        @wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal ncalls
            ncalls += 1
            return func(*args, **kwargs)
        wrapper.ncalls = lambda: ncalls
        return wrapper

    # Example
    @profiled
    def add(x, y):
        return x + y

这个方式跟之前的效果几乎一样，除了对于 ``ncalls`` 的访问现在是通过一个被绑定为属性的函数来实现，例如：

.. code-block:: python

    >>> add(2, 3)
    5
    >>> add(4, 5)
    9
    >>> add.ncalls()
    2
    >>>

============================
9.10 为类和静态方法提供装饰器
============================

----------
问题
----------
你想给类或静态方法提供装饰器。

----------
解决方案
----------
给类或静态方法提供装饰器是很简单的，不过要确保装饰器在 ``@classmethod`` 或 ``@staticmethod`` 之前。例如：

.. code-block:: python

    import time
    from functools import wraps

    # A simple decorator
    def timethis(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            r = func(*args, **kwargs)
            end = time.time()
            print(end-start)
            return r
        return wrapper

    # Class illustrating application of the decorator to different kinds of methods
    class Spam:
        @timethis
        def instance_method(self, n):
            print(self, n)
            while n > 0:
                n -= 1

        @classmethod
        @timethis
        def class_method(cls, n):
            print(cls, n)
            while n > 0:
                n -= 1

        @staticmethod
        @timethis
        def static_method(n):
            print(n)
            while n > 0:
                n -= 1

装饰后的类和静态方法可正常工作，只不过增加了额外的计时功能：

.. code-block:: python

    >>> s = Spam()
    >>> s.instance_method(1000000)
    <__main__.Spam object at 0x1006a6050> 1000000
    0.11817407608032227
    >>> Spam.class_method(1000000)
    <class '__main__.Spam'> 1000000
    0.11334395408630371
    >>> Spam.static_method(1000000)
    1000000
    0.11740279197692871
    >>>

----------
讨论
----------
如果你把装饰器的顺序写错了就会出错。例如，假设你像下面这样写：

.. code-block:: python

    class Spam:
        @timethis
        @staticmethod
        def static_method(n):
            print(n)
            while n > 0:
                n -= 1

那么你调用这个静态方法时就会报错：

.. code-block:: python

    >>> Spam.static_method(1000000)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "timethis.py", line 6, in wrapper
    start = time.time()
    TypeError: 'staticmethod' object is not callable
    >>>

问题在于 ``@classmethod`` 和 ``@staticmethod`` 实际上并不会创建可直接调用的对象，
而是创建特殊的描述器对象(参考8.9小节)。因此当你试着在其他装饰器中将它们当做函数来使用时就会出错。
确保这种装饰器出现在装饰器链中的第一个位置可以修复这个问题。

当我们在抽象基类中定义类方法和静态方法(参考8.12小节)时，这里讲到的知识就很有用了。
例如，如果你想定义一个抽象类方法，可以使用类似下面的代码：

.. code-block:: python

    from abc import ABCMeta, abstractmethod
    class A(metaclass=ABCMeta):
        @classmethod
        @abstractmethod
        def method(cls):
            pass

在这段代码中，``@classmethod`` 跟 ``@abstractmethod`` 两者的顺序是有讲究的，如果你调换它们的顺序就会出错。
============================
9.11 装饰器为被包装函数增加参数
============================

----------
问题
----------
你想在装饰器中给被包装函数增加额外的参数，但是不能影响这个函数现有的调用规则。

----------
解决方案
----------
可以使用关键字参数来给被包装函数增加额外参数。考虑下面的装饰器：

.. code-block:: python

    from functools import wraps

    def optional_debug(func):
        @wraps(func)
        def wrapper(*args, debug=False, **kwargs):
            if debug:
                print('Calling', func.__name__)
            return func(*args, **kwargs)

        return wrapper

.. code-block:: python

    >>> @optional_debug
    ... def spam(a,b,c):
    ...     print(a,b,c)
    ...
    >>> spam(1,2,3)
    1 2 3
    >>> spam(1,2,3, debug=True)
    Calling spam
    1 2 3
    >>>

----------
讨论
----------
通过装饰器来给被包装函数增加参数的做法并不常见。
尽管如此，有时候它可以避免一些重复代码。例如，如果你有下面这样的代码：

.. code-block:: python

    def a(x, debug=False):
        if debug:
            print('Calling a')

    def b(x, y, z, debug=False):
        if debug:
            print('Calling b')

    def c(x, y, debug=False):
        if debug:
            print('Calling c')

那么你可以将其重构成这样：

.. code-block:: python

    from functools import wraps
    import inspect

    def optional_debug(func):
        if 'debug' in inspect.getargspec(func).args:
            raise TypeError('debug argument already defined')

        @wraps(func)
        def wrapper(*args, debug=False, **kwargs):
            if debug:
                print('Calling', func.__name__)
            return func(*args, **kwargs)
        return wrapper

    @optional_debug
    def a(x):
        pass

    @optional_debug
    def b(x, y, z):
        pass

    @optional_debug
    def c(x, y):
        pass

这种实现方案之所以行得通，在于强制关键字参数很容易被添加到接受 ``*args`` 和 ``**kwargs`` 参数的函数中。
通过使用强制关键字参数，它被作为一个特殊情况被挑选出来，
并且接下来仅仅使用剩余的位置和关键字参数去调用这个函数时，这个特殊参数会被排除在外。
也就是说，它并不会被纳入到 ``**kwargs`` 中去。

还有一个难点就是如何去处理被添加的参数与被包装函数参数直接的名字冲突。
例如，如果装饰器 ``@optional_debug`` 作用在一个已经拥有一个 ``debug`` 参数的函数上时会有问题。
这里我们增加了一步名字检查。

上面的方案还可以更完美一点，因为精明的程序员应该发现了被包装函数的函数签名其实是错误的。例如：

.. code-block:: python

    >>> @optional_debug
    ... def add(x,y):
    ...     return x+y
    ...
    >>> import inspect
    >>> print(inspect.signature(add))
    (x, y)
    >>>

通过如下的修改，可以解决这个问题：

.. code-block:: python

    from functools import wraps
    import inspect

    def optional_debug(func):
        if 'debug' in inspect.getargspec(func).args:
            raise TypeError('debug argument already defined')

        @wraps(func)
        def wrapper(*args, debug=False, **kwargs):
            if debug:
                print('Calling', func.__name__)
            return func(*args, **kwargs)

        sig = inspect.signature(func)
        parms = list(sig.parameters.values())
        parms.append(inspect.Parameter('debug',
                    inspect.Parameter.KEYWORD_ONLY,
                    default=False))
        wrapper.__signature__ = sig.replace(parameters=parms)
        return wrapper


通过这样的修改，包装后的函数签名就能正确的显示 ``debug`` 参数的存在了。例如：

.. code-block:: python

    >>> @optional_debug
    ... def add(x,y):
    ...     return x+y
    ...
    >>> print(inspect.signature(add))
    (x, y, *, debug=False)
    >>> add(2,3)
    5
    >>>

参考9.16小节获取更多关于函数签名的信息。
============================
9.12 使用装饰器扩充类的功能
============================

----------
问题
----------
你想通过反省或者重写类定义的某部分来修改它的行为，但是你又不希望使用继承或元类的方式。

----------
解决方案
----------
这种情况可能是类装饰器最好的使用场景了。例如，下面是一个重写了特殊方法 ``__getattribute__`` 的类装饰器，
可以打印日志：

.. code-block:: python

    def log_getattribute(cls):
        # Get the original implementation
        orig_getattribute = cls.__getattribute__

        # Make a new definition
        def new_getattribute(self, name):
            print('getting:', name)
            return orig_getattribute(self, name)

        # Attach to the class and return
        cls.__getattribute__ = new_getattribute
        return cls

    # Example use
    @log_getattribute
    class A:
        def __init__(self,x):
            self.x = x
        def spam(self):
            pass

下面是使用效果：

.. code-block:: python

    >>> a = A(42)
    >>> a.x
    getting: x
    42
    >>> a.spam()
    getting: spam
    >>>

----------
讨论
----------
类装饰器通常可以作为其他高级技术比如混入或元类的一种非常简洁的替代方案。
比如，上面示例中的另外一种实现使用到继承：

.. code-block:: python

    class LoggedGetattribute:
        def __getattribute__(self, name):
            print('getting:', name)
            return super().__getattribute__(name)

    # Example:
    class A(LoggedGetattribute):
        def __init__(self,x):
            self.x = x
        def spam(self):
            pass

这种方案也行得通，但是为了去理解它，你就必须知道方法调用顺序、``super()`` 以及其它8.7小节介绍的继承知识。
某种程度上来讲，类装饰器方案就显得更加直观，并且它不会引入新的继承体系。它的运行速度也更快一些，
因为他并不依赖 ``super()`` 函数。

如果你系想在一个类上面使用多个类装饰器，那么就需要注意下顺序问题。
例如，一个装饰器A会将其装饰的方法完整替换成另一种实现，
而另一个装饰器B只是简单的在其装饰的方法中添加点额外逻辑。
那么这时候装饰器A就需要放在装饰器B的前面。

你还可以回顾一下8.13小节另外一个关于类装饰器的有用的例子。
============================
9.13 使用元类控制实例的创建
============================

----------
问题
----------
你想通过改变实例创建方式来实现单例、缓存或其他类似的特性。

----------
解决方案
----------
Python程序员都知道，如果你定义了一个类，就能像函数一样的调用它来创建实例，例如：

.. code-block:: python

    class Spam:
        def __init__(self, name):
            self.name = name

    a = Spam('Guido')
    b = Spam('Diana')

如果你想自定义这个步骤，你可以定义一个元类并自己实现 ``__call__()`` 方法。

为了演示，假设你不想任何人创建这个类的实例：

.. code-block:: python

    class NoInstances(type):
        def __call__(self, *args, **kwargs):
            raise TypeError("Can't instantiate directly")

    # Example
    class Spam(metaclass=NoInstances):
        @staticmethod
        def grok(x):
            print('Spam.grok')

这样的话，用户只能调用这个类的静态方法，而不能使用通常的方法来创建它的实例。例如：

.. code-block:: python

    >>> Spam.grok(42)
    Spam.grok
    >>> s = Spam()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "example1.py", line 7, in __call__
            raise TypeError("Can't instantiate directly")
    TypeError: Can't instantiate directly
    >>>

现在，假如你想实现单例模式（只能创建唯一实例的类），实现起来也很简单：

.. code-block:: python

    class Singleton(type):
        def __init__(self, *args, **kwargs):
            self.__instance = None
            super().__init__(*args, **kwargs)

        def __call__(self, *args, **kwargs):
            if self.__instance is None:
                self.__instance = super().__call__(*args, **kwargs)
                return self.__instance
            else:
                return self.__instance

    # Example
    class Spam(metaclass=Singleton):
        def __init__(self):
            print('Creating Spam')

那么Spam类就只能创建唯一的实例了，演示如下：

.. code-block:: python

    >>> a = Spam()
    Creating Spam
    >>> b = Spam()
    >>> a is b
    True
    >>> c = Spam()
    >>> a is c
    True
    >>>

最后，假设你想创建8.25小节中那样的缓存实例。下面我们可以通过元类来实现：

.. code-block:: python

    import weakref

    class Cached(type):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)
            self.__cache = weakref.WeakValueDictionary()

        def __call__(self, *args):
            if args in self.__cache:
                return self.__cache[args]
            else:
                obj = super().__call__(*args)
                self.__cache[args] = obj
                return obj

    # Example
    class Spam(metaclass=Cached):
        def __init__(self, name):
            print('Creating Spam({!r})'.format(name))
            self.name = name

然后我也来测试一下：

.. code-block:: python

    >>> a = Spam('Guido')
    Creating Spam('Guido')
    >>> b = Spam('Diana')
    Creating Spam('Diana')
    >>> c = Spam('Guido') # Cached
    >>> a is b
    False
    >>> a is c # Cached value returned
    True
    >>>

----------
讨论
----------
利用元类实现多种实例创建模式通常要比不使用元类的方式优雅得多。

假设你不使用元类，你可能需要将类隐藏在某些工厂函数后面。
比如为了实现一个单例，你你可能会像下面这样写：

.. code-block:: python

    class _Spam:
        def __init__(self):
            print('Creating Spam')

    _spam_instance = None

    def Spam():
        global _spam_instance

        if _spam_instance is not None:
            return _spam_instance
        else:
            _spam_instance = _Spam()
            return _spam_instance

尽管使用元类可能会涉及到比较高级点的技术，但是它的代码看起来会更加简洁舒服，而且也更加直观。

更多关于创建缓存实例、弱引用等内容，请参考8.25小节。
============================
9.14 捕获类的属性定义顺序
============================

----------
问题
----------
你想自动记录一个类中属性和方法定义的顺序，
然后可以利用它来做很多操作（比如序列化、映射到数据库等等）。

----------
解决方案
----------
利用元类可以很容易的捕获类的定义信息。下面是一个例子，使用了一个OrderedDict来记录描述器的定义顺序：

.. code-block:: python

    from collections import OrderedDict

    # A set of descriptors for various types
    class Typed:
        _expected_type = type(None)
        def __init__(self, name=None):
            self._name = name

        def __set__(self, instance, value):
            if not isinstance(value, self._expected_type):
                raise TypeError('Expected ' + str(self._expected_type))
            instance.__dict__[self._name] = value

    class Integer(Typed):
        _expected_type = int

    class Float(Typed):
        _expected_type = float

    class String(Typed):
        _expected_type = str

    # Metaclass that uses an OrderedDict for class body
    class OrderedMeta(type):
        def __new__(cls, clsname, bases, clsdict):
            d = dict(clsdict)
            order = []
            for name, value in clsdict.items():
                if isinstance(value, Typed):
                    value._name = name
                    order.append(name)
            d['_order'] = order
            return type.__new__(cls, clsname, bases, d)

        @classmethod
        def __prepare__(cls, clsname, bases):
            return OrderedDict()

在这个元类中，执行类主体时描述器的定义顺序会被一个 ``OrderedDict``捕获到，
生成的有序名称从字典中提取出来并放入类属性 ``_order`` 中。这样的话类中的方法可以通过多种方式来使用它。
例如，下面是一个简单的类，使用这个排序字典来实现将一个类实例的数据序列化为一行CSV数据：

.. code-block:: python

    class Structure(metaclass=OrderedMeta):
        def as_csv(self):
            return ','.join(str(getattr(self,name)) for name in self._order)

    # Example use
    class Stock(Structure):
        name = String()
        shares = Integer()
        price = Float()

        def __init__(self, name, shares, price):
            self.name = name
            self.shares = shares
            self.price = price

我们在交互式环境中测试一下这个Stock类：

.. code-block:: python

    >>> s = Stock('GOOG',100,490.1)
    >>> s.name
    'GOOG'
    >>> s.as_csv()
    'GOOG,100,490.1'
    >>> t = Stock('AAPL','a lot', 610.23)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "dupmethod.py", line 34, in __init__
    TypeError: shares expects <class 'int'>
    >>>

----------
讨论
----------
本节一个关键点就是OrderedMeta元类中定义的 `` __prepare__()`` 方法。
这个方法会在开始定义类和它的父类的时候被执行。它必须返回一个映射对象以便在类定义体中被使用到。
我们这里通过返回了一个OrderedDict而不是一个普通的字典，可以很容易的捕获定义的顺序。

如果你想构造自己的类字典对象，可以很容易的扩展这个功能。比如，下面的这个修改方案可以防止重复的定义：

.. code-block:: python

    from collections import OrderedDict

    class NoDupOrderedDict(OrderedDict):
        def __init__(self, clsname):
            self.clsname = clsname
            super().__init__()
        def __setitem__(self, name, value):
            if name in self:
                raise TypeError('{} already defined in {}'.format(name, self.clsname))
            super().__setitem__(name, value)

    class OrderedMeta(type):
        def __new__(cls, clsname, bases, clsdict):
            d = dict(clsdict)
            d['_order'] = [name for name in clsdict if name[0] != '_']
            return type.__new__(cls, clsname, bases, d)

        @classmethod
        def __prepare__(cls, clsname, bases):
            return NoDupOrderedDict(clsname)

下面我们测试重复的定义会出现什么情况：

.. code-block:: python

    >>> class A(metaclass=OrderedMeta):
    ... def spam(self):
    ... pass
    ... def spam(self):
    ... pass
    ...
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 4, in A
        File "dupmethod2.py", line 25, in __setitem__
            (name, self.clsname))
    TypeError: spam already defined in A
    >>>

最后还有一点很重要，就是在 ``__new__()`` 方法中对于元类中被修改字典的处理。
尽管类使用了另外一个字典来定义，在构造最终的 ``class`` 对象的时候，
我们仍然需要将这个字典转换为一个正确的 ``dict`` 实例。
通过语句 ``d = dict(clsdict)`` 来完成这个效果。

对于很多应用程序而已，能够捕获类定义的顺序是一个看似不起眼却又非常重要的特性。
例如，在对象关系映射中，我们通常会看到下面这种方式定义的类：

.. code-block:: python

    class Stock(Model):
        name = String()
        shares = Integer()
        price = Float()

在框架底层，我们必须捕获定义的顺序来将对象映射到元组或数据库表中的行（就类似于上面例子中的 ``as_csv()`` 的功能）。
这节演示的技术非常简单，并且通常会比其他类似方法（通常都要在描述器类中维护一个隐藏的计数器）要简单的多。

============================
9.15 定义有可选参数的元类
============================

----------
问题
----------
你想定义一个元类，允许类定义时提供可选参数，这样可以控制或配置类型的创建过程。

----------
解决方案
----------
在定义类的时候，Python允许我们使用 ``metaclass``关键字参数来指定特定的元类。
例如使用抽象基类：

.. code-block:: python

    from abc import ABCMeta, abstractmethod
    class IStream(metaclass=ABCMeta):
        @abstractmethod
        def read(self, maxsize=None):
            pass

        @abstractmethod
        def write(self, data):
            pass

然而，在自定义元类中我们还可以提供其他的关键字参数，如下所示：

.. code-block:: python

    class Spam(metaclass=MyMeta, debug=True, synchronize=True):
        pass

为了使元类支持这些关键字参数，你必须确保在 ``__prepare__()`` , ``__new__()`` 和 ``__init__()`` 方法中
都使用强制关键字参数。就像下面这样：

.. code-block:: python

    class MyMeta(type):
        # Optional
        @classmethod
        def __prepare__(cls, name, bases, *, debug=False, synchronize=False):
            # Custom processing
            pass
            return super().__prepare__(name, bases)

        # Required
        def __new__(cls, name, bases, ns, *, debug=False, synchronize=False):
            # Custom processing
            pass
            return super().__new__(cls, name, bases, ns)

        # Required
        def __init__(self, name, bases, ns, *, debug=False, synchronize=False):
            # Custom processing
            pass
            super().__init__(name, bases, ns)

----------
讨论
----------
给一个元类添加可选关键字参数需要你完全弄懂类创建的所有步骤，
因为这些参数会被传递给每一个相关的方法。
``__prepare__()`` 方法在所有类定义开始执行前首先被调用，用来创建类命名空间。
通常来讲，这个方法只是简单的返回一个字典或其他映射对象。
``__new__()`` 方法被用来实例化最终的类对象。它在类的主体被执行完后开始执行。
``__init__()`` 方法最后被调用，用来执行其他的一些初始化工作。

当我们构造元类的时候，通常只需要定义一个 ``__new__()`` 或 ``__init__()`` 方法，但不是两个都定义。
但是，如果需要接受其他的关键字参数的话，这两个方法就要同时提供，并且都要提供对应的参数签名。
默认的 ``__prepare__()`` 方法接受任意的关键字参数，但是会忽略它们，
所以只有当这些额外的参数可能会影响到类命名空间的创建时你才需要去定义 ``__prepare__()`` 方法。

通过使用强制关键字参数，在类的创建过程中我们必须通过关键字来指定这些参数。

使用关键字参数配置一个元类还可以视作对类变量的一种替代方式。例如：

.. code-block:: python

    class Spam(metaclass=MyMeta):
        debug = True
        synchronize = True
        pass

将这些属性定义为参数的好处在于它们不会污染类的名称空间，
这些属性仅仅只从属于类的创建阶段，而不是类中的语句执行阶段。
另外，它们在 ``__prepare__()`` 方法中是可以被访问的，因为这个方法会在所有类主体执行前被执行。
但是类变量只能在元类的 ``__new__()`` 和 ``__init__()`` 方法中可见。

===============================
9.16 *args和**kwargs的强制参数签名
===============================

----------
问题
----------
你有一个函数或方法，它使用*args和**kwargs作为参数，这样使得它比较通用，
但有时候你想检查传递进来的参数是不是某个你想要的类型。

----------
解决方案
----------
对任何涉及到操作函数调用签名的问题，你都应该使用 ``inspect`` 模块中的签名特性。
我们最主要关注两个类：``Signature`` 和 ``Parameter`` 。下面是一个创建函数前面的交互例子：

.. code-block:: python

    >>> from inspect import Signature, Parameter
    >>> # Make a signature for a func(x, y=42, *, z=None)
    >>> parms = [ Parameter('x', Parameter.POSITIONAL_OR_KEYWORD),
    ...         Parameter('y', Parameter.POSITIONAL_OR_KEYWORD, default=42),
    ...         Parameter('z', Parameter.KEYWORD_ONLY, default=None) ]
    >>> sig = Signature(parms)
    >>> print(sig)
    (x, y=42, *, z=None)
    >>>

一旦你有了一个签名对象，你就可以使用它的 ``bind()`` 方法很容易的将它绑定到 ``*args`` 和 ``**kwargs`` 上去。
下面是一个简单的演示：

.. code-block:: python

    >>> def func(*args, **kwargs):
    ...     bound_values = sig.bind(*args, **kwargs)
    ...     for name, value in bound_values.arguments.items():
    ...         print(name,value)
    ...
    >>> # Try various examples
    >>> func(1, 2, z=3)
    x 1
    y 2
    z 3
    >>> func(1)
    x 1
    >>> func(1, z=3)
    x 1
    z 3
    >>> func(y=2, x=1)
    x 1
    y 2
    >>> func(1, 2, 3, 4)
    Traceback (most recent call last):
    ...
        File "/usr/local/lib/python3.3/inspect.py", line 1972, in _bind
            raise TypeError('too many positional arguments')
    TypeError: too many positional arguments
    >>> func(y=2)
    Traceback (most recent call last):
    ...
        File "/usr/local/lib/python3.3/inspect.py", line 1961, in _bind
            raise TypeError(msg) from None
    TypeError: 'x' parameter lacking default value
    >>> func(1, y=2, x=3)
    Traceback (most recent call last):
    ...
        File "/usr/local/lib/python3.3/inspect.py", line 1985, in _bind
            '{arg!r}'.format(arg=param.name))
    TypeError: multiple values for argument 'x'
    >>>

可以看出来，通过将签名和传递的参数绑定起来，可以强制函数调用遵循特定的规则，比如必填、默认、重复等等。

下面是一个强制函数签名更具体的例子。在代码中，我们在基类中先定义了一个非常通用的 ``__init__()`` 方法，
然后我们强制所有的子类必须提供一个特定的参数签名。

.. code-block:: python

    from inspect import Signature, Parameter

    def make_sig(*names):
        parms = [Parameter(name, Parameter.POSITIONAL_OR_KEYWORD)
                for name in names]
        return Signature(parms)

    class Structure:
        __signature__ = make_sig()
        def __init__(self, *args, **kwargs):
            bound_values = self.__signature__.bind(*args, **kwargs)
            for name, value in bound_values.arguments.items():
                setattr(self, name, value)

    # Example use
    class Stock(Structure):
        __signature__ = make_sig('name', 'shares', 'price')

    class Point(Structure):
        __signature__ = make_sig('x', 'y')

下面是使用这个 ``Stock`` 类的示例：

.. code-block:: python

    >>> import inspect
    >>> print(inspect.signature(Stock))
    (name, shares, price)
    >>> s1 = Stock('ACME', 100, 490.1)
    >>> s2 = Stock('ACME', 100)
    Traceback (most recent call last):
    ...
    TypeError: 'price' parameter lacking default value
    >>> s3 = Stock('ACME', 100, 490.1, shares=50)
    Traceback (most recent call last):
    ...
    TypeError: multiple values for argument 'shares'
    >>>

----------
讨论
----------
在我们需要构建通用函数库、编写装饰器或实现代理的时候，对于 ``*args`` 和 ``**kwargs`` 的使用是很普遍的。
但是，这样的函数有一个缺点就是当你想要实现自己的参数检验时，代码就会笨拙混乱。在8.11小节里面有这样一个例子。
这时候我们可以通过一个签名对象来简化它。

在最后的一个方案实例中，我们还可以通过使用自定义元类来创建签名对象。下面演示怎样来实现：

.. code-block:: python

    from inspect import Signature, Parameter

    def make_sig(*names):
        parms = [Parameter(name, Parameter.POSITIONAL_OR_KEYWORD)
                for name in names]
        return Signature(parms)

    class StructureMeta(type):
        def __new__(cls, clsname, bases, clsdict):
            clsdict['__signature__'] = make_sig(*clsdict.get('_fields',[]))
            return super().__new__(cls, clsname, bases, clsdict)

    class Structure(metaclass=StructureMeta):
        _fields = []
        def __init__(self, *args, **kwargs):
            bound_values = self.__signature__.bind(*args, **kwargs)
            for name, value in bound_values.arguments.items():
                setattr(self, name, value)

    # Example
    class Stock(Structure):
        _fields = ['name', 'shares', 'price']

    class Point(Structure):
        _fields = ['x', 'y']

当我们自定义签名的时候，将签名存储在特定的属性 ``__signature__`` 中通常是很有用的。
这样的话，在使用 ``inspect`` 模块执行内省的代码就能发现签名并将它作为调用约定。

.. code-block:: python

    >>> import inspect
    >>> print(inspect.signature(Stock))
    (name, shares, price)
    >>> print(inspect.signature(Point))
    (x, y)
    >>>

==============================
9.17 在类上强制使用编程规约
==============================

----------
问题
----------
你的程序包含一个很大的类继承体系，你希望强制执行某些编程规约（或者代码诊断）来帮助程序员保持清醒。

----------
解决方案
----------
如果你想监控类的定义，通常可以通过定义一个元类。一个基本元类通常是继承自 ``type`` 并重定义它的 ``__new__()`` 方法
或者是 ``__init__()`` 方法。比如：

.. code-block:: python

    class MyMeta(type):
        def __new__(self, clsname, bases, clsdict):
            # clsname is name of class being defined
            # bases is tuple of base classes
            # clsdict is class dictionary
            return super().__new__(cls, clsname, bases, clsdict)

另一种是，定义 ``__init__()`` 方法：

.. code-block:: python

    class MyMeta(type):
        def __init__(self, clsname, bases, clsdict):
            super().__init__(clsname, bases, clsdict)
            # clsname is name of class being defined
            # bases is tuple of base classes
            # clsdict is class dictionary

为了使用这个元类，你通常要将它放到到一个顶级父类定义中，然后其他的类继承这个顶级父类。例如：

.. code-block:: python

    class Root(metaclass=MyMeta):
        pass

    class A(Root):
        pass

    class B(Root):
        pass

元类的一个关键特点是它允许你在定义的时候检查类的内容。在重新定义 ``__init__()`` 方法中，
你可以很轻松的检查类字典、父类等等。并且，一旦某个元类被指定给了某个类，那么就会被继承到所有子类中去。
因此，一个框架的构建者就能在大型的继承体系中通过给一个顶级父类指定一个元类去捕获所有下面子类的定义。

作为一个具体的应用例子，下面定义了一个元类，它会拒绝任何有混合大小写名字作为方法的类定义（可能是想气死Java程序员^_^）：

.. code-block:: python

    class NoMixedCaseMeta(type):
        def __new__(cls, clsname, bases, clsdict):
            for name in clsdict:
                if name.lower() != name:
                    raise TypeError('Bad attribute name: ' + name)
            return super().__new__(cls, clsname, bases, clsdict)

    class Root(metaclass=NoMixedCaseMeta):
        pass

    class A(Root):
        def foo_bar(self): # Ok
            pass

    class B(Root):
        def fooBar(self): # TypeError
            pass

作为更高级和实用的例子，下面有一个元类，它用来检测重载方法，确保它的调用参数跟父类中原始方法有着相同的参数签名。

.. code-block:: python

    from inspect import signature
    import logging

    class MatchSignaturesMeta(type):

        def __init__(self, clsname, bases, clsdict):
            super().__init__(clsname, bases, clsdict)
            sup = super(self, self)
            for name, value in clsdict.items():
                if name.startswith('_') or not callable(value):
                    continue
                # Get the previous definition (if any) and compare the signatures
                prev_dfn = getattr(sup,name,None)
                if prev_dfn:
                    prev_sig = signature(prev_dfn)
                    val_sig = signature(value)
                    if prev_sig != val_sig:
                        logging.warning('Signature mismatch in %s. %s != %s',
                                        value.__qualname__, prev_sig, val_sig)

    # Example
    class Root(metaclass=MatchSignaturesMeta):
        pass

    class A(Root):
        def foo(self, x, y):
            pass

        def spam(self, x, *, z):
            pass

    # Class with redefined methods, but slightly different signatures
    class B(A):
        def foo(self, a, b):
            pass

        def spam(self,x,z):
            pass

如果你运行这段代码，就会得到下面这样的输出结果：

.. code-block:: python

    WARNING:root:Signature mismatch in B.spam. (self, x, *, z) != (self, x, z)
    WARNING:root:Signature mismatch in B.foo. (self, x, y) != (self, a, b)

这种警告信息对于捕获一些微妙的程序bug是很有用的。例如，如果某个代码依赖于传递给方法的关键字参数，
那么当子类改变参数名字的时候就会调用出错。

----------
讨论
----------
在大型面向对象的程序中，通常将类的定义放在元类中控制是很有用的。
元类可以监控类的定义，警告编程人员某些没有注意到的可能出现的问题。

有人可能会说，像这样的错误可以通过程序分析工具或IDE去做会更好些。诚然，这些工具是很有用。
但是，如果你在构建一个框架或函数库供其他人使用，那么你没办法去控制使用者要使用什么工具。
因此，对于这种类型的程序，如果可以在元类中做检测或许可以带来更好的用户体验。

在元类中选择重新定义 ``__new__()`` 方法还是 ``__init__()`` 方法取决于你想怎样使用结果类。
``__new__()`` 方法在类创建之前被调用，通常用于通过某种方式（比如通过改变类字典的内容）修改类的定义。
而 ``__init__()`` 方法是在类被创建之后被调用，当你需要完整构建类对象的时候会很有用。
在最后一个例子中，这是必要的，因为它使用了 ``super()`` 函数来搜索之前的定义。
它只能在类的实例被创建之后，并且相应的方法解析顺序也已经被设置好了。

最后一个例子还演示了Python的函数签名对象的使用。
实际上，元类将每个可调用定义放在一个类中，搜索前一个定义（如果有的话），
然后通过使用 ``inspect.signature()`` 来简单的比较它们的调用签名。

最后一点，代码中有一行使用了 ``super(self, self)`` 并不是排版错误。
当使用元类的时候，我们要时刻记住一点就是 ``self`` 实际上是一个类对象。
因此，这条语句其实就是用来寻找位于继承体系中构建 ``self`` 父类的定义。
==============================
9.18 以编程方式定义类
==============================

----------
问题
----------
你在写一段代码，最终需要创建一个新的类对象。你考虑将类的定义源代码以字符串的形式发布出去。
并且使用函数比如 ``exec()`` 来执行它，但是你想寻找一个更加优雅的解决方案。

----------
解决方案
----------
你可以使用函数 ``types.new_class()`` 来初始化新的类对象。
你需要做的只是提供类的名字、父类元组、关键字参数，以及一个用成员变量填充类字典的回调函数。例如：

.. code-block:: python

    # stock.py
    # Example of making a class manually from parts

    # Methods
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price
    def cost(self):
        return self.shares * self.price

    cls_dict = {
        '__init__' : __init__,
        'cost' : cost,
    }

    # Make a class
    import types

    Stock = types.new_class('Stock', (), {}, lambda ns: ns.update(cls_dict))
    Stock.__module__ = __name__

这种方式会构建一个普通的类对象，并且按照你的期望工作：

.. code-block:: python

    >>> s = Stock('ACME', 50, 91.1)
    >>> s
    <stock.Stock object at 0x1006a9b10>
    >>> s.cost()
    4555.0
    >>>

这种方法中，一个比较难理解的地方是在调用完 ``types.new_class()`` 对 ``Stock.__module__`` 的赋值。
每次当一个类被定义后，它的 ``__module__`` 属性包含定义它的模块名。
这个名字用于生成 ``__repr__()`` 方法的输出。它同样也被用于很多库，比如 ``pickle`` 。
因此，为了让你创建的类是“正确”的，你需要确保这个属性也设置正确了。

如果你想创建的类需要一个不同的元类，可以通过 ``types.new_class()`` 第三个参数传递给它。例如：

.. code-block:: python

    >>> import abc
    >>> Stock = types.new_class('Stock', (), {'metaclass': abc.ABCMeta},
    ...                         lambda ns: ns.update(cls_dict))
    ...
    >>> Stock.__module__ = __name__
    >>> Stock
    <class '__main__.Stock'>
    >>> type(Stock)
    <class 'abc.ABCMeta'>
    >>>

第三个参数还可以包含其他的关键字参数。比如，一个类的定义如下：

.. code-block:: python

    class Spam(Base, debug=True, typecheck=False):
        pass

那么可以将其翻译成如下的 ``new_class()`` 调用形式：

.. code-block:: python

    Spam = types.new_class('Spam', (Base,),
                            {'debug': True, 'typecheck': False},
                            lambda ns: ns.update(cls_dict))

``new_class()`` 第四个参数最神秘，它是一个用来接受类命名空间的映射对象的函数。
通常这是一个普通的字典，但是它实际上是 ``__prepare__()`` 方法返回的任意对象，这个在9.14小节已经介绍过了。
这个函数需要使用上面演示的 ``update()`` 方法给命名空间增加内容。

----------
讨论
----------
很多时候如果能构造新的类对象是很有用的。
有个很熟悉的例子是调用 ``collections.namedtuple()`` 函数，例如：


.. code-block:: python

    >>> Stock = collections.namedtuple('Stock', ['name', 'shares', 'price'])
    >>> Stock
    <class '__main__.Stock'>
    >>>

``namedtuple()`` 使用 ``exec()`` 而不是上面介绍的技术。但是，下面通过一个简单的变化，
我们直接创建一个类：

.. code-block:: python

    import operator
    import types
    import sys

    def named_tuple(classname, fieldnames):
        # Populate a dictionary of field property accessors
        cls_dict = { name: property(operator.itemgetter(n))
                    for n, name in enumerate(fieldnames) }

        # Make a __new__ function and add to the class dict
        def __new__(cls, *args):
            if len(args) != len(fieldnames):
                raise TypeError('Expected {} arguments'.format(len(fieldnames)))
            return tuple.__new__(cls, args)

        cls_dict['__new__'] = __new__

        # Make the class
        cls = types.new_class(classname, (tuple,), {},
                            lambda ns: ns.update(cls_dict))

        # Set the module to that of the caller
        cls.__module__ = sys._getframe(1).f_globals['__name__']
        return cls

这段代码的最后部分使用了一个所谓的"框架魔法"，通过调用 ``sys._getframe()`` 来获取调用者的模块名。
另外一个框架魔法例子在2.15小节中有介绍过。

下面的例子演示了前面的代码是如何工作的：

.. code-block:: python

    >>> Point = named_tuple('Point', ['x', 'y'])
    >>> Point
    <class '__main__.Point'>
    >>> p = Point(4, 5)
    >>> len(p)
    2
    >>> p.x
    4
    >>> p.y
    5
    >>> p.x = 2
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>> print('%s %s' % p)
    4 5
    >>>

这项技术一个很重要的方面是它对于元类的正确使用。
你可能像通过直接实例化一个元类来直接创建一个类：

.. code-block:: python

    Stock = type('Stock', (), cls_dict)

这种方法的问题在于它忽略了一些关键步骤，比如对于元类中 ``__prepare__()`` 方法的调用。
通过使用 ``types.new_class()`` ，你可以保证所有的必要初始化步骤都能得到执行。
比如，``types.new_class()`` 第四个参数的回调函数接受 ``__prepare__()`` 方法返回的映射对象。


如果你仅仅只是想执行准备步骤，可以使用 ``types.prepare_class()`` 。例如：

.. code-block:: python

    import types
    metaclass, kwargs, ns = types.prepare_class('Stock', (), {'metaclass': type})

它会查找合适的元类并调用它的 ``__prepare__()`` 方法。
然后这个元类保存它的关键字参数，准备命名空间后被返回。

更多信息, 请参考 `PEP 3115 <https://www.python.org/dev/peps/pep-3115/>`_ ,
以及 `Python documentation <https://docs.python.org/3/reference/datamodel.html#metaclasses>`_ .
==============================
9.19 在定义的时候初始化类的成员
==============================

----------
问题
----------
你想在类被定义的时候就初始化一部分类的成员，而不是要等到实例被创建后。


----------
解决方案
----------
在类定义时就执行初始化或设置操作是元类的一个典型应用场景。本质上讲，一个元类会在定义时被触发，
这时候你可以执行一些额外的操作。

下面是一个例子，利用这个思路来创建类似于 ``collections`` 模块中的命名元组的类：

.. code-block:: python

    import operator

    class StructTupleMeta(type):
        def __init__(cls, *args, **kwargs):
            super().__init__(*args, **kwargs)
            for n, name in enumerate(cls._fields):
                setattr(cls, name, property(operator.itemgetter(n)))

    class StructTuple(tuple, metaclass=StructTupleMeta):
        _fields = []
        def __new__(cls, *args):
            if len(args) != len(cls._fields):
                raise ValueError('{} arguments required'.format(len(cls._fields)))
            return super().__new__(cls,args)

这段代码可以用来定义简单的基于元组的数据结构，如下所示：

.. code-block:: python

    class Stock(StructTuple):
        _fields = ['name', 'shares', 'price']

    class Point(StructTuple):
        _fields = ['x', 'y']

下面演示它如何工作：

.. code-block:: python

    >>> s = Stock('ACME', 50, 91.1)
    >>> s
    ('ACME', 50, 91.1)
    >>> s[0]
    'ACME'
    >>> s.name
    'ACME'
    >>> s.shares * s.price
    4555.0
    >>> s.shares = 23
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>>

----------
讨论
----------

这一小节中，类 ``StructTupleMeta`` 获取到类属性 ``_fields`` 中的属性名字列表，
然后将它们转换成相应的可访问特定元组槽的方法。函数 ``operator.itemgetter()`` 创建一个访问器函数，
然后 ``property()`` 函数将其转换成一个属性。

本节最难懂的部分是知道不同的初始化步骤是什么时候发生的。
``StructTupleMeta`` 中的 ``__init__()`` 方法只在每个类被定义时被调用一次。
``cls`` 参数就是那个被定义的类。实际上，上述代码使用了 ``_fields`` 类变量来保存新的被定义的类，
然后给它再添加一点新的东西。

``StructTuple`` 类作为一个普通的基类，供其他使用者来继承。
这个类中的 ``__new__()`` 方法用来构造新的实例。
这里使用 ``__new__()`` 并不是很常见，主要是因为我们要修改元组的调用签名，
使得我们可以像普通的实例调用那样创建实例。就像下面这样：

.. code-block:: python

    s = Stock('ACME', 50, 91.1) # OK
    s = Stock(('ACME', 50, 91.1)) # Error

跟 ``__init__()`` 不同的是，``__new__()`` 方法在实例被创建之前被触发。
由于元组是不可修改的，所以一旦它们被创建了就不可能对它做任何改变。而 ``__init__()`` 会在实例创建的最后被触发，
这样的话我们就可以做我们想做的了。这也是为什么 ``__new__()`` 方法已经被定义了。

尽管本节很短，还是需要你能仔细研读，深入思考Python类是如何被定义的，实例是如何被创建的，
还有就是元类和类的各个不同的方法究竟在什么时候被调用。

`PEP 422 <http://www.python.org/dev/peps/pep-0422>`_
提供了一个解决本节问题的另外一种方法。
但是，截止到我写这本书的时候，它还没被采纳和接受。
尽管如此，如果你使用的是Python 3.3或更高的版本，那么还是值得去看一下的。
==============================
9.20 利用函数注解实现方法重载
==============================

----------
问题
----------
你已经学过怎样使用函数参数注解，那么你可能会想利用它来实现基于类型的方法重载。
但是你不确定应该怎样去实现（或者到底行得通不）。

----------
解决方案
----------
本小节的技术是基于一个简单的技术，那就是Python允许参数注解，代码可以像下面这样写：

.. code-block:: python

    class Spam:
        def bar(self, x:int, y:int):
            print('Bar 1:', x, y)

        def bar(self, s:str, n:int = 0):
            print('Bar 2:', s, n)

    s = Spam()
    s.bar(2, 3) # Prints Bar 1: 2 3
    s.bar('hello') # Prints Bar 2: hello 0

下面是我们第一步的尝试，使用到了一个元类和描述器：

.. code-block:: python

    # multiple.py
    import inspect
    import types

    class MultiMethod:
        '''
        Represents a single multimethod.
        '''
        def __init__(self, name):
            self._methods = {}
            self.__name__ = name

        def register(self, meth):
            '''
            Register a new method as a multimethod
            '''
            sig = inspect.signature(meth)

            # Build a type signature from the method's annotations
            types = []
            for name, parm in sig.parameters.items():
                if name == 'self':
                    continue
                if parm.annotation is inspect.Parameter.empty:
                    raise TypeError(
                        'Argument {} must be annotated with a type'.format(name)
                    )
                if not isinstance(parm.annotation, type):
                    raise TypeError(
                        'Argument {} annotation must be a type'.format(name)
                    )
                if parm.default is not inspect.Parameter.empty:
                    self._methods[tuple(types)] = meth
                types.append(parm.annotation)

            self._methods[tuple(types)] = meth

        def __call__(self, *args):
            '''
            Call a method based on type signature of the arguments
            '''
            types = tuple(type(arg) for arg in args[1:])
            meth = self._methods.get(types, None)
            if meth:
                return meth(*args)
            else:
                raise TypeError('No matching method for types {}'.format(types))

        def __get__(self, instance, cls):
            '''
            Descriptor method needed to make calls work in a class
            '''
            if instance is not None:
                return types.MethodType(self, instance)
            else:
                return self

    class MultiDict(dict):
        '''
        Special dictionary to build multimethods in a metaclass
        '''
        def __setitem__(self, key, value):
            if key in self:
                # If key already exists, it must be a multimethod or callable
                current_value = self[key]
                if isinstance(current_value, MultiMethod):
                    current_value.register(value)
                else:
                    mvalue = MultiMethod(key)
                    mvalue.register(current_value)
                    mvalue.register(value)
                    super().__setitem__(key, mvalue)
            else:
                super().__setitem__(key, value)

    class MultipleMeta(type):
        '''
        Metaclass that allows multiple dispatch of methods
        '''
        def __new__(cls, clsname, bases, clsdict):
            return type.__new__(cls, clsname, bases, dict(clsdict))

        @classmethod
        def __prepare__(cls, clsname, bases):
            return MultiDict()

为了使用这个类，你可以像下面这样写：

.. code-block:: python

    class Spam(metaclass=MultipleMeta):
        def bar(self, x:int, y:int):
            print('Bar 1:', x, y)

        def bar(self, s:str, n:int = 0):
            print('Bar 2:', s, n)

    # Example: overloaded __init__
    import time

    class Date(metaclass=MultipleMeta):
        def __init__(self, year: int, month:int, day:int):
            self.year = year
            self.month = month
            self.day = day

        def __init__(self):
            t = time.localtime()
            self.__init__(t.tm_year, t.tm_mon, t.tm_mday)

下面是一个交互示例来验证它能正确的工作：

.. code-block:: python

    >>> s = Spam()
    >>> s.bar(2, 3)
    Bar 1: 2 3
    >>> s.bar('hello')
    Bar 2: hello 0
    >>> s.bar('hello', 5)
    Bar 2: hello 5
    >>> s.bar(2, 'hello')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "multiple.py", line 42, in __call__
            raise TypeError('No matching method for types {}'.format(types))
    TypeError: No matching method for types (<class 'int'>, <class 'str'>)
    >>> # Overloaded __init__
    >>> d = Date(2012, 12, 21)
    >>> # Get today's date
    >>> e = Date()
    >>> e.year
    2012
    >>> e.month
    12
    >>> e.day
    3
    >>>

----------
讨论
----------
坦白来讲，相对于通常的代码而已本节使用到了很多的魔法代码。
但是，它却能让我们深入理解元类和描述器的底层工作原理，
并能加深对这些概念的印象。因此，就算你并不会立即去应用本节的技术，
它的一些底层思想却会影响到其它涉及到元类、描述器和函数注解的编程技术。

本节的实现中的主要思路其实是很简单的。``MutipleMeta`` 元类使用它的 ``__prepare__()`` 方法
来提供一个作为 ``MultiDict`` 实例的自定义字典。这个跟普通字典不一样的是，
``MultiDict`` 会在元素被设置的时候检查是否已经存在，如果存在的话，重复的元素会在 ``MultiMethod``
实例中合并。

``MultiMethod`` 实例通过构建从类型签名到函数的映射来收集方法。
在这个构建过程中，函数注解被用来收集这些签名然后构建这个映射。
这个过程在 ``MultiMethod.register()`` 方法中实现。
这种映射的一个关键特点是对于多个方法，所有参数类型都必须要指定，否则就会报错。

为了让 ``MultiMethod`` 实例模拟一个调用，它的 ``__call__()`` 方法被实现了。
这个方法从所有排除 ``slef`` 的参数中构建一个类型元组，在内部map中查找这个方法，
然后调用相应的方法。为了能让 ``MultiMethod`` 实例在类定义时正确操作，``__get__()`` 是必须得实现的。
它被用来构建正确的绑定方法。比如：

.. code-block:: python

    >>> b = s.bar
    >>> b
    <bound method Spam.bar of <__main__.Spam object at 0x1006a46d0>>
    >>> b.__self__
    <__main__.Spam object at 0x1006a46d0>
    >>> b.__func__
    <__main__.MultiMethod object at 0x1006a4d50>
    >>> b(2, 3)
    Bar 1: 2 3
    >>> b('hello')
    Bar 2: hello 0
    >>>

不过本节的实现还有一些限制，其中一个是它不能使用关键字参数。例如：

.. code-block:: python

    >>> s.bar(x=2, y=3)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: __call__() got an unexpected keyword argument 'y'

    >>> s.bar(s='hello')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: __call__() got an unexpected keyword argument 's'
    >>>

也许有其他的方法能添加这种支持，但是它需要一个完全不同的方法映射方式。
问题在于关键字参数的出现是没有顺序的。当它跟位置参数混合使用时，
那你的参数就会变得比较混乱了，这时候你不得不在 ``__call__()`` 方法中先去做个排序。

同样对于继承也是有限制的，例如，类似下面这种代码就不能正常工作：

.. code-block:: python

    class A:
        pass

    class B(A):
        pass

    class C:
        pass

    class Spam(metaclass=MultipleMeta):
        def foo(self, x:A):
            print('Foo 1:', x)

        def foo(self, x:C):
            print('Foo 2:', x)

原因是因为 ``x:A`` 注解不能成功匹配子类实例（比如B的实例），如下：

.. code-block:: python

    >>> s = Spam()
    >>> a = A()
    >>> s.foo(a)
    Foo 1: <__main__.A object at 0x1006a5310>
    >>> c = C()
    >>> s.foo(c)
    Foo 2: <__main__.C object at 0x1007a1910>
    >>> b = B()
    >>> s.foo(b)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "multiple.py", line 44, in __call__
            raise TypeError('No matching method for types {}'.format(types))
    TypeError: No matching method for types (<class '__main__.B'>,)
    >>>

作为使用元类和注解的一种替代方案，可以通过描述器来实现类似的效果。例如：

.. code-block:: python

    import types

    class multimethod:
        def __init__(self, func):
            self._methods = {}
            self.__name__ = func.__name__
            self._default = func

        def match(self, *types):
            def register(func):
                ndefaults = len(func.__defaults__) if func.__defaults__ else 0
                for n in range(ndefaults+1):
                    self._methods[types[:len(types) - n]] = func
                return self
            return register

        def __call__(self, *args):
            types = tuple(type(arg) for arg in args[1:])
            meth = self._methods.get(types, None)
            if meth:
                return meth(*args)
            else:
                return self._default(*args)

        def __get__(self, instance, cls):
            if instance is not None:
                return types.MethodType(self, instance)
            else:
                return self

为了使用描述器版本，你需要像下面这样写：

.. code-block:: python

    class Spam:
        @multimethod
        def bar(self, *args):
            # Default method called if no match
            raise TypeError('No matching method for bar')

        @bar.match(int, int)
        def bar(self, x, y):
            print('Bar 1:', x, y)

        @bar.match(str, int)
        def bar(self, s, n = 0):
            print('Bar 2:', s, n)

描述器方案同样也有前面提到的限制（不支持关键字参数和继承）。

所有事物都是平等的，有好有坏，也许最好的办法就是在普通代码中避免使用方法重载。
不过有些特殊情况下还是有意义的，比如基于模式匹配的方法重载程序中。
举个例子，8.21小节中的访问者模式可以修改为一个使用方法重载的类。
但是，除了这个以外，通常不应该使用方法重载（就简单的使用不同名称的方法就行了）。

在Python社区对于实现方法重载的讨论已经由来已久。
对于引发这个争论的原因，可以参考下Guido van Rossum的这篇博客：
`Five-Minute Multimethods in Python <http://www.artima.com/weblogs/viewpost.jsp?thread=101605>`_

==============================
9.21 避免重复的属性方法
==============================

----------
问题
----------
你在类中需要重复的定义一些执行相同逻辑的属性方法，比如进行类型检查，怎样去简化这些重复代码呢？

----------
解决方案
----------
考虑下一个简单的类，它的属性由属性方法包装：

.. code-block:: python

    class Person:
        def __init__(self, name ,age):
            self.name = name
            self.age = age

        @property
        def name(self):
            return self._name

        @name.setter
        def name(self, value):
            if not isinstance(value, str):
                raise TypeError('name must be a string')
            self._name = value

        @property
        def age(self):
            return self._age

        @age.setter
        def age(self, value):
            if not isinstance(value, int):
                raise TypeError('age must be an int')
            self._age = value

可以看到，为了实现属性值的类型检查我们写了很多的重复代码。
只要你以后看到类似这样的代码，你都应该想办法去简化它。
一个可行的方法是创建一个函数用来定义属性并返回它。例如：

.. code-block:: python

    def typed_property(name, expected_type):
        storage_name = '_' + name

        @property
        def prop(self):
            return getattr(self, storage_name)

        @prop.setter
        def prop(self, value):
            if not isinstance(value, expected_type):
                raise TypeError('{} must be a {}'.format(name, expected_type))
            setattr(self, storage_name, value)

        return prop

    # Example use
    class Person:
        name = typed_property('name', str)
        age = typed_property('age', int)

        def __init__(self, name, age):
            self.name = name
            self.age = age

----------
讨论
----------
本节我们演示内部函数或者闭包的一个重要特性，它们很像一个宏。例子中的函数 ``typed_property()``
看上去有点难理解，其实它所做的仅仅就是为你生成属性并返回这个属性对象。
因此，当在一个类中使用它的时候，效果跟将它里面的代码放到类定义中去是一样的。
尽管属性的 ``getter`` 和 ``setter`` 方法访问了本地变量如 ``name`` , ``expected_type``
以及 ``storate_name`` ，这个很正常，这些变量的值会保存在闭包当中。


我们还可以使用 ``functools.partial()`` 来稍稍改变下这个例子，很有趣。例如，你可以像下面这样：

.. code-block:: python

    from functools import partial

    String = partial(typed_property, expected_type=str)
    Integer = partial(typed_property, expected_type=int)

    # Example:
    class Person:
        name = String('name')
        age = Integer('age')

        def __init__(self, name, age):
            self.name = name
            self.age = age

其实你可以发现，这里的代码跟8.13小节中的类型系统描述器代码有些相似。

==============================
9.22 定义上下文管理器的简单方法
==============================

----------
问题
----------
你想自己去实现一个新的上下文管理器，以便使用with语句。

----------
解决方案
----------
实现一个新的上下文管理器的最简单的方法就是使用 ``contexlib`` 模块中的 ``@contextmanager`` 装饰器。
下面是一个实现了代码块计时功能的上下文管理器例子：

.. code-block:: python

    import time
    from contextlib import contextmanager

    @contextmanager
    def timethis(label):
        start = time.time()
        try:
            yield
        finally:
            end = time.time()
            print('{}: {}'.format(label, end - start))

    # Example use
    with timethis('counting'):
        n = 10000000
        while n > 0:
            n -= 1

在函数 ``timethis()`` 中，``yield`` 之前的代码会在上下文管理器中作为 ``__enter__()`` 方法执行，
所有在 ``yield`` 之后的代码会作为 ``__exit__()`` 方法执行。
如果出现了异常，异常会在yield语句那里抛出。

下面是一个更加高级一点的上下文管理器，实现了列表对象上的某种事务：

.. code-block:: python

    @contextmanager
    def list_transaction(orig_list):
        working = list(orig_list)
        yield working
        orig_list[:] = working

这段代码的作用是任何对列表的修改只有当所有代码运行完成并且不出现异常的情况下才会生效。
下面我们来演示一下：

.. code-block:: python

    >>> items = [1, 2, 3]
    >>> with list_transaction(items) as working:
    ...     working.append(4)
    ...     working.append(5)
    ...
    >>> items
    [1, 2, 3, 4, 5]
    >>> with list_transaction(items) as working:
    ...     working.append(6)
    ...     working.append(7)
    ...     raise RuntimeError('oops')
    ...
    Traceback (most recent call last):
        File "<stdin>", line 4, in <module>
    RuntimeError: oops
    >>> items
    [1, 2, 3, 4, 5]
    >>>

----------
讨论
----------
通常情况下，如果要写一个上下文管理器，你需要定义一个类，里面包含一个 ``__enter__()`` 和一个
``__exit__()`` 方法，如下所示：

.. code-block:: python

    import time

    class timethis:
        def __init__(self, label):
            self.label = label

        def __enter__(self):
            self.start = time.time()

        def __exit__(self, exc_ty, exc_val, exc_tb):
            end = time.time()
            print('{}: {}'.format(self.label, end - self.start))

尽管这个也不难写，但是相比较写一个简单的使用 ``@contextmanager`` 注解的函数而言还是稍显乏味。

``@contextmanager`` 应该仅仅用来写自包含的上下文管理函数。
如果你有一些对象(比如一个文件、网络连接或锁)，需要支持 ``with`` 语句，那么你就需要单独实现
``__enter__()`` 方法和 ``__exit__()`` 方法。
==============================
9.23 在局部变量域中执行代码
==============================

----------
问题
----------
你想在使用范围内执行某个代码片段，并且希望在执行后所有的结果都不可见。

----------
解决方案
----------
为了理解这个问题，先试试一个简单场景。首先，在全局命名空间内执行一个代码片段：

.. code-block:: python

    >>> a = 13
    >>> exec('b = a + 1')
    >>> print(b)
    14
    >>>

然后，再在一个函数中执行同样的代码：

.. code-block:: python

    >>> def test():
    ...     a = 13
    ...     exec('b = a + 1')
    ...     print(b)
    ...
    >>> test()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "<stdin>", line 4, in test
    NameError: global name 'b' is not defined
    >>>

可以看出，最后抛出了一个NameError异常，就跟在 ``exec()`` 语句从没执行过一样。
要是你想在后面的计算中使用到 ``exec()`` 执行结果的话就会有问题了。

为了修正这样的错误，你需要在调用 ``exec()`` 之前使用 ``locals()`` 函数来得到一个局部变量字典。
之后你就能从局部字典中获取修改过后的变量值了。例如：

.. code-block:: python

    >>> def test():
    ...     a = 13
    ...     loc = locals()
    ...     exec('b = a + 1')
    ...     b = loc['b']
    ...     print(b)
    ...
    >>> test()
    14
    >>>

----------
讨论
----------
实际上对于 ``exec()`` 的正确使用是比较难的。大多数情况下当你要考虑使用 ``exec()`` 的时候，
还有另外更好的解决方案（比如装饰器、闭包、元类等等）。

然而，如果你仍然要使用 ``exec()`` ，本节列出了一些如何正确使用它的方法。
默认情况下，``exec()`` 会在调用者局部和全局范围内执行代码。然而，在函数里面，
传递给 ``exec()`` 的局部范围是拷贝实际局部变量组成的一个字典。
因此，如果 ``exec()`` 如果执行了修改操作，这种修改后的结果对实际局部变量值是没有影响的。
下面是另外一个演示它的例子：

.. code-block:: python

    >>> def test1():
    ...     x = 0
    ...     exec('x += 1')
    ...     print(x)
    ...
    >>> test1()
    0
    >>>

上面代码里，当你调用 ``locals()`` 获取局部变量时，你获得的是传递给 ``exec()`` 的局部变量的一个拷贝。
通过在代码执行后审查这个字典的值，那就能获取修改后的值了。下面是一个演示例子：

.. code-block:: python

    >>> def test2():
    ...     x = 0
    ...     loc = locals()
    ...     print('before:', loc)
    ...     exec('x += 1')
    ...     print('after:', loc)
    ...     print('x =', x)
    ...
    >>> test2()
    before: {'x': 0}
    after: {'loc': {...}, 'x': 1}
    x = 0
    >>>

仔细观察最后一步的输出，除非你将 ``loc`` 中被修改后的值手动赋值给x，否则x变量值是不会变的。

在使用 ``locals()`` 的时候，你需要注意操作顺序。每次它被调用的时候，
``locals()`` 会获取局部变量值中的值并覆盖字典中相应的变量。
请注意观察下下面这个试验的输出结果：

.. code-block:: python

    >>> def test3():
    ...     x = 0
    ...     loc = locals()
    ...     print(loc)
    ...     exec('x += 1')
    ...     print(loc)
    ...     locals()
    ...     print(loc)
    ...
    >>> test3()
    {'x': 0}
    {'loc': {...}, 'x': 1}
    {'loc': {...}, 'x': 0}
    >>>

注意最后一次调用 ``locals()`` 的时候x的值是如何被覆盖掉的。

作为 ``locals()`` 的一个替代方案，你可以使用你自己的字典，并将它传递给 ``exec()`` 。例如：

.. code-block:: python

    >>> def test4():
    ...     a = 13
    ...     loc = { 'a' : a }
    ...     glb = { }
    ...     exec('b = a + 1', glb, loc)
    ...     b = loc['b']
    ...     print(b)
    ...
    >>> test4()
    14
    >>>

大部分情况下，这种方式是使用 ``exec()`` 的最佳实践。
你只需要保证全局和局部字典在后面代码访问时已经被初始化。

还有一点，在使用 ``exec()`` 之前，你可能需要问下自己是否有其他更好的替代方案。
大多数情况下当你要考虑使用 ``exec()`` 的时候，
还有另外更好的解决方案，比如装饰器、闭包、元类，或其他一些元编程特性。
==============================
9.24 解析与分析Python源码
==============================

----------
问题
----------
你想写解析并分析Python源代码的程序。

----------
解决方案
----------
大部分程序员知道Python能够计算或执行字符串形式的源代码。例如：

.. code-block:: python

    >>> x = 42
    >>> eval('2 + 3*4 + x')
    56
    >>> exec('for i in range(10): print(i)')
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9
    >>>

尽管如此，``ast`` 模块能被用来将Python源码编译成一个可被分析的抽象语法树（AST）。例如：

.. code-block:: python

    >>> import ast
    >>> ex = ast.parse('2 + 3*4 + x', mode='eval')
    >>> ex
    <_ast.Expression object at 0x1007473d0>
    >>> ast.dump(ex)
    "Expression(body=BinOp(left=BinOp(left=Num(n=2), op=Add(),
    right=BinOp(left=Num(n=3), op=Mult(), right=Num(n=4))), op=Add(),
    right=Name(id='x', ctx=Load())))"

    >>> top = ast.parse('for i in range(10): print(i)', mode='exec')
    >>> top
    <_ast.Module object at 0x100747390>
    >>> ast.dump(top)
    "Module(body=[For(target=Name(id='i', ctx=Store()),
    iter=Call(func=Name(id='range', ctx=Load()), args=[Num(n=10)],
    keywords=[], starargs=None, kwargs=None),
    body=[Expr(value=Call(func=Name(id='print', ctx=Load()),
    args=[Name(id='i', ctx=Load())], keywords=[], starargs=None,
    kwargs=None))], orelse=[])])"
    >>>

分析源码树需要你自己更多的学习，它是由一系列AST节点组成的。
分析这些节点最简单的方法就是定义一个访问者类，实现很多 ``visit_NodeName()`` 方法，
``NodeName()`` 匹配那些你感兴趣的节点。下面是这样一个类，记录了哪些名字被加载、存储和删除的信息。

.. code-block:: python

    import ast

    class CodeAnalyzer(ast.NodeVisitor):
        def __init__(self):
            self.loaded = set()
            self.stored = set()
            self.deleted = set()

        def visit_Name(self, node):
            if isinstance(node.ctx, ast.Load):
                self.loaded.add(node.id)
            elif isinstance(node.ctx, ast.Store):
                self.stored.add(node.id)
            elif isinstance(node.ctx, ast.Del):
                self.deleted.add(node.id)

    # Sample usage
    if __name__ == '__main__':
        # Some Python code
        code = '''
        for i in range(10):
            print(i)
        del i
        '''

        # Parse into an AST
        top = ast.parse(code, mode='exec')

        # Feed the AST to analyze name usage
        c = CodeAnalyzer()
        c.visit(top)
        print('Loaded:', c.loaded)
        print('Stored:', c.stored)
        print('Deleted:', c.deleted)

如果你运行这个程序，你会得到下面这样的输出：

.. code-block:: python

    Loaded: {'i', 'range', 'print'}
    Stored: {'i'}
    Deleted: {'i'}

最后，AST可以通过 ``compile()`` 函数来编译并执行。例如：

.. code-block:: python

    >>> exec(compile(top,'<stdin>', 'exec'))
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9
    >>>

----------
讨论
----------
当你能够分析源代码并从中获取信息的时候，你就能写很多代码分析、优化或验证工具了。
例如，相比盲目的传递一些代码片段到类似 ``exec()`` 函数中，你可以先将它转换成一个AST，
然后观察它的细节看它到底是怎样做的。
你还可以写一些工具来查看某个模块的全部源码，并且在此基础上执行某些静态分析。

需要注意的是，如果你知道自己在干啥，你还能够重写AST来表示新的代码。
下面是一个装饰器例子，可以通过重新解析函数体源码、
重写AST并重新创建函数代码对象来将全局访问变量降为函数体作用范围，

.. code-block:: python

    # namelower.py
    import ast
    import inspect

    # Node visitor that lowers globally accessed names into
    # the function body as local variables.
    class NameLower(ast.NodeVisitor):
        def __init__(self, lowered_names):
            self.lowered_names = lowered_names

        def visit_FunctionDef(self, node):
            # Compile some assignments to lower the constants
            code = '__globals = globals()\n'
            code += '\n'.join("{0} = __globals['{0}']".format(name)
                                for name in self.lowered_names)
            code_ast = ast.parse(code, mode='exec')

            # Inject new statements into the function body
            node.body[:0] = code_ast.body

            # Save the function object
            self.func = node

    # Decorator that turns global names into locals
    def lower_names(*namelist):
        def lower(func):
            srclines = inspect.getsource(func).splitlines()
            # Skip source lines prior to the @lower_names decorator
            for n, line in enumerate(srclines):
                if '@lower_names' in line:
                    break

            src = '\n'.join(srclines[n+1:])
            # Hack to deal with indented code
            if src.startswith((' ','\t')):
                src = 'if 1:\n' + src
            top = ast.parse(src, mode='exec')

            # Transform the AST
            cl = NameLower(namelist)
            cl.visit(top)

            # Execute the modified AST
            temp = {}
            exec(compile(top,'','exec'), temp, temp)

            # Pull out the modified code object
            func.__code__ = temp[func.__name__].__code__
            return func
        return lower

为了使用这个代码，你可以像下面这样写：

.. code-block:: python

    INCR = 1
    @lower_names('INCR')
    def countdown(n):
        while n > 0:
            n -= INCR

装饰器会将 ``countdown()`` 函数重写为类似下面这样子：

.. code-block:: python

    def countdown(n):
        __globals = globals()
        INCR = __globals['INCR']
        while n > 0:
            n -= INCR

在性能测试中，它会让函数运行快20%

现在，你是不是想为你所有的函数都加上这个装饰器呢？或许不会。
但是，这却是对于一些高级技术比如AST操作、源码操作等等的一个很好的演示说明

本节受另外一个在 ``ActiveState`` 中处理Python字节码的章节的启示。
使用AST是一个更加高级点的技术，并且也更简单些。参考下面一节获得字节码的更多信息。

==============================
9.25 拆解Python字节码
==============================

----------
问题
----------
你想通过将你的代码反编译成低级的字节码来查看它底层的工作机制。

----------
解决方案
----------
``dis`` 模块可以被用来输出任何Python函数的反编译结果。例如：

.. code-block:: python

    >>> def countdown(n):
    ... while n > 0:
    ...     print('T-minus', n)
    ...     n -= 1
    ... print('Blastoff!')
    ...
    >>> import dis
    >>> dis.dis(countdown)
    ...
    >>>

----------
讨论
----------
当你想要知道你的程序底层的运行机制的时候，``dis`` 模块是很有用的。比如如果你想试着理解性能特征。
被 ``dis()`` 函数解析的原始字节码如下所示：

.. code-block:: python

    >>> countdown.__code__.co_code
    b"x'\x00|\x00\x00d\x01\x00k\x04\x00r)\x00t\x00\x00d\x02\x00|\x00\x00\x83
    \x02\x00\x01|\x00\x00d\x03\x008}\x00\x00q\x03\x00Wt\x00\x00d\x04\x00\x83
    \x01\x00\x01d\x00\x00S"
    >>>

如果你想自己解释这段代码，你需要使用一些在 ``opcode`` 模块中定义的常量。例如：

.. code-block:: python

    >>> c = countdown.__code__.co_code
    >>> import opcode
    >>> opcode.opname[c[0]]
    >>> opcode.opname[c[0]]
    'SETUP_LOOP'
    >>> opcode.opname[c[3]]
    'LOAD_FAST'
    >>>

奇怪的是，在 ``dis`` 模块中并没有函数让你以编程方式很容易的来处理字节码。
不过，下面的生成器函数可以将原始字节码序列转换成 ``opcodes`` 和参数。

.. code-block:: python

    import opcode

    def generate_opcodes(codebytes):
        extended_arg = 0
        i = 0
        n = len(codebytes)
        while i < n:
            op = codebytes[i]
            i += 1
            if op >= opcode.HAVE_ARGUMENT:
                oparg = codebytes[i] + codebytes[i+1]*256 + extended_arg
                extended_arg = 0
                i += 2
                if op == opcode.EXTENDED_ARG:
                    extended_arg = oparg * 65536
                    continue
            else:
                oparg = None
            yield (op, oparg)

使用方法如下：

.. code-block:: python

    >>> for op, oparg in generate_opcodes(countdown.__code__.co_code):
    ...     print(op, opcode.opname[op], oparg)

这种方式很少有人知道，你可以利用它替换任何你想要替换的函数的原始字节码。
下面我们用一个示例来演示整个过程：

.. code-block:: python

    >>> def add(x, y):
    ...     return x + y
    ...
    >>> c = add.__code__
    >>> c
    <code object add at 0x1007beed0, file "<stdin>", line 1>
    >>> c.co_code
    b'|\x00\x00|\x01\x00\x17S'
    >>>
    >>> # Make a completely new code object with bogus byte code
    >>> import types
    >>> newbytecode = b'xxxxxxx'
    >>> nc = types.CodeType(c.co_argcount, c.co_kwonlyargcount,
    ...     c.co_nlocals, c.co_stacksize, c.co_flags, newbytecode, c.co_consts,
    ...     c.co_names, c.co_varnames, c.co_filename, c.co_name,
    ...     c.co_firstlineno, c.co_lnotab)
    >>> nc
    <code object add at 0x10069fe40, file "<stdin>", line 1>
    >>> add.__code__ = nc
    >>> add(2,3)
    Segmentation fault

你可以像这样耍大招让解释器奔溃。但是，对于编写更高级优化和元编程工具的程序员来讲，
他们可能真的需要重写字节码。本节最后的部分演示了这个是怎样做到的。你还可以参考另外一个类似的例子：
`this code on ActiveState <http://code.activestate.com/recipes/277940-decorator-for-bindingconstants-at-compile-time/>`_
============================
10.1 构建一个模块的层级包
============================

----------
问题
----------
你想将你的代码组织成由很多分层模块构成的包。

----------
解决方案
----------
封装成包是很简单的。在文件系统上组织你的代码，并确保每个目录都定义了一个__init__.py文件。
例如：

.. code-block:: python

    graphics/
        __init__.py
        primitive/
            __init__.py
            line.py
            fill.py
            text.py
        formats/
            __init__.py
            png.py
            jpg.py

一旦你做到了这一点，你应该能够执行各种import语句，如下：

.. code-block:: python

    import graphics.primitive.line
    from graphics.primitive import line
    import graphics.formats.jpg as jpg

----------
讨论
----------
定义模块的层次结构就像在文件系统上建立目录结构一样容易。
文件__init__.py的目的是要包含不同运行级别的包的可选的初始化代码。
举个例子，如果你执行了语句import graphics， 文件graphics/__init__.py将被导入,建立graphics命名空间的内容。像import graphics.format.jpg这样导入，文件graphics/__init__.py和文件graphics/formats/__init__.py将在文件graphics/formats/jpg.py导入之前导入。


绝大部分时候让__init__.py空着就好。但是有些情况下可能包含代码。
举个例子，__init__.py能够用来自动加载子模块:

.. code-block:: python

    # graphics/formats/__init__.py
    from . import jpg
    from . import png


像这样一个文件,用户可以仅仅通过import grahpics.formats来代替import graphics.formats.jpg以及import graphics.formats.png。


__init__.py的其他常用用法包括将多个文件合并到一个逻辑命名空间，这将在10.4小节讨论。


敏锐的程序员会发现，即使没有__init__.py文件存在，python仍然会导入包。如果你没有定义__init__.py时，实际上创建了一个所谓的“命名空间包”，这将在10.5小节讨论。万物平等，如果你着手创建一个新的包的话，包含一个__init__.py文件吧。
============================
10.2 控制模块被全部导入的内容
============================

----------
问题
----------
当使用'from module import *' 语句时，希望对从模块或包导出的符号进行精确控制。


----------
解决方案
----------
在你的模块中定义一个变量 __all__ 来明确地列出需要导出的内容。

举个例子:

.. code-block:: python

    # somemodule.py
    def spam():
        pass

    def grok():
        pass

    blah = 42
    # Only export 'spam' and 'grok'
    __all__ = ['spam', 'grok']

----------
讨论
----------
尽管强烈反对使用 'from module import *', 但是在定义了大量变量名的模块中频繁使用。
如果你不做任何事, 这样的导入将会导入所有不以下划线开头的。
另一方面,如果定义了 __all__ , 那么只有被列举出的东西会被导出。



如果你将 __all__ 定义成一个空列表, 没有东西将被导入。
如果 __all__ 包含未定义的名字, 在导入时引起AttributeError。

============================
10.3 使用相对路径名导入包中子模块
============================

----------
问题
----------
将代码组织成包,想用import语句从另一个包名没有硬编码过的包中导入子模块。



----------
解决方案
----------
使用包的相对导入，使一个模块导入同一个包的另一个模块
举个例子，假设在你的文件系统上有mypackage包，组织如下：


.. code-block:: python

    mypackage/
        __init__.py
        A/
            __init__.py
            spam.py
            grok.py
        B/
            __init__.py
            bar.py

如果模块mypackage.A.spam要导入同目录下的模块grok，它应该包括的import语句如下：


.. code-block:: python

    # mypackage/A/spam.py
    from . import grok

如果模块mypackage.A.spam要导入不同目录下的模块B.bar，它应该使用的import语句如下：


.. code-block:: python

    # mypackage/A/spam.py
    from ..B import bar

两个import语句都没包含顶层包名，而是使用了spam.py的相对路径。

----------
讨论
----------
在包内，既可以使用相对路径也可以使用绝对路径来导入。
举个例子：

.. code-block:: python

    # mypackage/A/spam.py
    from mypackage.A import grok # OK
    from . import grok # OK
    import grok # Error (not found)

像mypackage.A这样使用绝对路径名的不利之处是这将顶层包名硬编码到你的源码中。如果你想重新组织它，你的代码将更脆，很难工作。 举个例子，如果你改变了包名，你就必须检查所有文件来修正源码。 同样，硬编码的名称会使移动代码变得困难。举个例子，也许有人想安装两个不同版本的软件包，只通过名称区分它们。 如果使用相对导入，那一切都ok，然而使用绝对路径名很可能会出问题。


import语句的 ``.`` 和 ``..`` 看起来很滑稽, 但它指定目录名.为当前目录，..B为目录../B。这种语法只适用于import。
举个例子：

.. code-block:: python

    from . import grok # OK
    import .grok # ERROR

尽管使用相对导入看起来像是浏览文件系统，但是不能到定义包的目录之外。也就是说，使用点的这种模式从不是包的目录中导入将会引发错误。


最后，相对导入只适用于在合适的包中的模块。尤其是在顶层的脚本的简单模块中，它们将不起作用。如果包的部分被作为脚本直接执行，那它们将不起作用 
例如：

.. code-block:: python

    % python3 mypackage/A/spam.py # Relative imports fail

另一方面，如果你使用Python的-m选项来执行先前的脚本，相对导入将会正确运行。
例如：


.. code-block:: python

    % python3 -m mypackage.A.spam # Relative imports work

更多的包的相对导入的背景知识,请看 `PEP 328 <http://www.python.org/dev/peps/pep-0328>`_ .


============================
10.4 将模块分割成多个文件
============================

----------
问题
----------
你想将一个模块分割成多个文件。但是你不想将分离的文件统一成一个逻辑模块时使已有的代码遭到破坏。


----------
解决方案
----------
程序模块可以通过变成包来分割成多个独立的文件。考虑下下面简单的模块：


.. code-block:: python

    # mymodule.py
    class A:
        def spam(self):
            print('A.spam')

    class B(A):
        def bar(self):
            print('B.bar')

假设你想mymodule.py分为两个文件，每个定义的一个类。要做到这一点，首先用mymodule目录来替换文件mymodule.py。
这这个目录下，创建以下文件：


.. code-block:: python

    mymodule/
        __init__.py
        a.py
        b.py

在a.py文件中插入以下代码：

.. code-block:: python

    # a.py
    class A:
        def spam(self):
            print('A.spam')

在b.py文件中插入以下代码：

.. code-block:: python

    # b.py
    from .a import A
    class B(A):
        def bar(self):
            print('B.bar')

最后，在 __init__.py 中，将2个文件粘合在一起：

.. code-block:: python

    # __init__.py
    from .a import A
    from .b import B

如果按照这些步骤，所产生的包MyModule将作为一个单一的逻辑模块：

.. code-block:: python

    >>> import mymodule
    >>> a = mymodule.A()
    >>> a.spam()
    A.spam
    >>> b = mymodule.B()
    >>> b.bar()
    B.bar
    >>>

----------
讨论
----------
在这个章节中的主要问题是一个设计问题，不管你是否希望用户使用很多小模块或只是一个模块。举个例子，在一个大型的代码库中，你可以将这一切都分割成独立的文件，让用户使用大量的import语句，就像这样：


.. code-block:: python

    from mymodule.a import A
    from mymodule.b import B
    ...

这样能工作，但这让用户承受更多的负担，用户要知道不同的部分位于何处。通常情况下，将这些统一起来，使用一条import将更加容易，就像这样：


.. code-block:: python

    from mymodule import A, B

对后者而言，让mymodule成为一个大的源文件是最常见的。但是，这一章节展示了如何合并多个文件合并成一个单一的逻辑命名空间。
这样做的关键是创建一个包目录，使用 __init__.py 文件来将每部分粘合在一起。


当一个模块被分割，你需要特别注意交叉引用的文件名。举个例子，在这一章节中，B类需要访问A类作为基类。用包的相对导入 from .a import A 来获取。


整个章节都使用包的相对导入来避免将顶层模块名硬编码到源代码中。这使得重命名模块或者将它移动到别的位置更容易。（见10.3小节）


作为这一章节的延伸，将介绍延迟导入。如图所示，__init__.py文件一次导入所有必需的组件的。但是对于一个很大的模块，可能你只想组件在需要时被加载。
要做到这一点，__init__.py有细微的变化：

.. code-block:: python

    # __init__.py
    def A():
        from .a import A
        return A()

    def B():
        from .b import B
        return B()

在这个版本中，类A和类B被替换为在第一次访问时加载所需的类的函数。对于用户，这看起来不会有太大的不同。
例如：

.. code-block:: python

    >>> import mymodule
    >>> a = mymodule.A()
    >>> a.spam()
    A.spam
    >>>

延迟加载的主要缺点是继承和类型检查可能会中断。你可能会稍微改变你的代码，例如:

.. code-block:: python

    if isinstance(x, mymodule.A): # Error
    ...

    if isinstance(x, mymodule.a.A): # Ok
    ...

延迟加载的真实例子, 见标准库 multiprocessing/__init__.py 的源码.


==============================
10.5 利用命名空间导入目录分散的代码
==============================

----------
问题
----------
你可能有大量的代码，由不同的人来分散地维护。每个部分被组织为文件目录，如一个包。然而，你希望能用共同的包前缀将所有组件连接起来，不是将每一个部分作为独立的包来安装。

----------
解决方案
----------
从本质上讲，你要定义一个顶级Python包，作为一个大集合分开维护子包的命名空间。这个问题经常出现在大的应用框架中，框架开发者希望鼓励用户发布插件或附加包。


在统一不同的目录里统一相同的命名空间，但是要删去用来将组件联合起来的__init__.py文件。假设你有Python代码的两个不同的目录如下：


.. code-block:: python

    foo-package/
        spam/
            blah.py

    bar-package/
        spam/
            grok.py

在这2个目录里，都有着共同的命名空间spam。在任何一个目录里都没有__init__.py文件。


让我们看看，如果将foo-package和bar-package都加到python模块路径并尝试导入会发生什么


.. code-block:: python

    >>> import sys
    >>> sys.path.extend(['foo-package', 'bar-package'])
    >>> import spam.blah
    >>> import spam.grok
    >>>

两个不同的包目录被合并到一起，你可以导入spam.blah和spam.grok，并且它们能够工作。


----------
讨论
----------
在这里工作的机制被称为“包命名空间”的一个特征。从本质上讲，包命名空间是一种特殊的封装设计，为合并不同的目录的代码到一个共同的命名空间。对于大的框架，这可能是有用的，因为它允许一个框架的部分被单独地安装下载。它也使人们能够轻松地为这样的框架编写第三方附加组件和其他扩展。

包命名空间的关键是确保顶级目录中没有__init__.py文件来作为共同的命名空间。缺失__init__.py文件使得在导入包的时候会发生有趣的事情：这并没有产生错误，解释器创建了一个由所有包含匹配包名的目录组成的列表。特殊的包命名空间模块被创建，只读的目录列表副本被存储在其__path__变量中。
举个例子：

.. code-block:: python

    >>> import spam
    >>> spam.__path__
    _NamespacePath(['foo-package/spam', 'bar-package/spam'])
    >>>

在定位包的子组件时，目录__path__将被用到(例如, 当导入spam.grok或者spam.blah的时候).

包命名空间的一个重要特点是任何人都可以用自己的代码来扩展命名空间。举个例子，假设你自己的代码目录像这样：


.. code-block:: python

    my-package/
        spam/
            custom.py

如果你将你的代码目录和其他包一起添加到sys.path，这将无缝地合并到别的spam包目录中：


.. code-block:: python

    >>> import spam.custom
    >>> import spam.grok
    >>> import spam.blah
    >>>

一个包是否被作为一个包命名空间的主要方法是检查其__file__属性。如果没有，那包是个命名空间。这也可以由其字符表现形式中的“namespace”这个词体现出来。


.. code-block:: python

    >>> spam.__file__
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: 'module' object has no attribute '__file__'
    >>> spam
    <module 'spam' (namespace)>
    >>>

更多的包命名空间信息可以查看
`PEP 420 <https://www.python.org/dev/peps/pep-0420/>`_.

==============================
10.6 重新加载模块
==============================

----------
问题
----------
你想重新加载已经加载的模块，因为你对其源码进行了修改。

----------
解决方案
----------
使用imp.reload()来重新加载先前加载的模块。举个例子：

.. code-block:: python

    >>> import spam
    >>> import imp
    >>> imp.reload(spam)
    <module 'spam' from './spam.py'>
    >>>

----------
讨论
----------
重新加载模块在开发和调试过程中常常很有用。但在生产环境中的代码使用会不安全，因为它并不总是像您期望的那样工作。


reload()擦除了模块底层字典的内容，并通过重新执行模块的源代码来刷新它。模块对象本身的身份保持不变。因此，该操作在程序中所有已经被导入了的地方更新了模块。


尽管如此，reload()没有更新像"from module import name"这样使用import语句导入的定义。举个例子：

.. code-block:: python

    # spam.py
    def bar():
        print('bar')

    def grok():
        print('grok')

现在启动交互式会话：

.. code-block:: python

    >>> import spam
    >>> from spam import grok
    >>> spam.bar()
    bar
    >>> grok()
    grok
    >>>

不退出Python修改spam.py的源码，将grok()函数改成这样：


.. code-block:: python

    def grok():
        print('New grok')

现在回到交互式会话，重新加载模块，尝试下这个实验：

.. code-block:: python

    >>> import imp
    >>> imp.reload(spam)
    <module 'spam' from './spam.py'>
    >>> spam.bar()
    bar
    >>> grok() # Notice old output
    grok
    >>> spam.grok() # Notice new output
    New grok
    >>>

在这个例子中，你看到有2个版本的grok()函数被加载。通常来说，这不是你想要的，而是令人头疼的事。


因此，在生产环境中可能需要避免重新加载模块。在交互环境下调试，解释程序并试图弄懂它。
===========================
10.7 运行目录或压缩文件
===========================

----------
问题
----------
您有一个已成长为包含多个文件的应用，它已远不再是一个简单的脚本，你想向用户提供一些简单的方法运行这个程序。

----------
解决方案
----------
如果你的应用程序已经有多个文件，你可以把你的应用程序放进它自己的目录并添加一个__main__.py文件。 举个例子，你可以像这样创建目录：

.. code-block:: python

    myapplication/
        spam.py
        bar.py
        grok.py
        __main__.py

如果__main__.py存在，你可以简单地在顶级目录运行Python解释器：

.. code-block:: python

    bash % python3 myapplication

解释器将执行__main__.py文件作为主程序。

如果你将你的代码打包成zip文件，这种技术同样也适用，举个例子：

.. code-block:: python

    bash % ls
    spam.py bar.py grok.py __main__.py
    bash % zip -r myapp.zip *.py
    bash % python3 myapp.zip
    ... output from __main__.py ...

----------
讨论
----------
创建一个目录或zip文件并添加__main__.py文件来将一个更大的Python应用打包是可行的。这和作为标准库被安装到Python库的代码包是有一点区别的。相反，这只是让别人执行的代码包。


由于目录和zip文件与正常文件有一点不同，你可能还需要增加一个shell脚本，使执行更加容易。例如，如果代码文件名为myapp.zip，你可以创建这样一个顶级脚本：


.. code-block:: bash

    #!/usr/bin/env python3 /usr/local/bin/myapp.zip

================================
10.8 读取位于包中的数据文件
================================

----------
问题
----------
你的包中包含代码需要去读取的数据文件。你需要尽可能地用最便捷的方式来做这件事。

----------
解决方案
----------
假设你的包中的文件组织成如下：

.. code-block:: python

    mypackage/
        __init__.py
        somedata.dat
        spam.py

现在假设spam.py文件需要读取somedata.dat文件中的内容。你可以用以下代码来完成：

.. code-block:: python

    # spam.py
    import pkgutil
    data = pkgutil.get_data(__package__, 'somedata.dat')

由此产生的变量是包含该文件的原始内容的字节字符串。

----------
讨论
----------
要读取数据文件，你可能会倾向于编写使用内置的I/ O功能的代码，如open()。但是这种方法也有一些问题。


首先，一个包对解释器的当前工作目录几乎没有控制权。因此，编程时任何I/O操作都必须使用绝对文件名。由于每个模块包含有完整路径的__file__变量，这弄清楚它的路径不是不可能，但它很凌乱。


第二，包通常安装作为.zip或.egg文件，这些文件并不像在文件系统上的一个普通目录里那样被保存。因此，你试图用open()对一个包含数据文件的归档文件进行操作，它根本不会工作。


pkgutil.get_data()函数是一个读取数据文件的高级工具，不用管包是如何安装以及安装在哪。它只是工作并将文件内容以字节字符串返回给你


get_data()的第一个参数是包含包名的字符串。你可以直接使用包名，也可以使用特殊的变量，比如__package__。第二个参数是包内文件的相对名称。如果有必要，可以使用标准的Unix命名规范到不同的目录，只要最后的目录仍然位于包中。
================================
10.9 将文件夹加入到sys.path
================================

----------
问题
----------
你无法导入你的Python代码因为它所在的目录不在sys.path里。你想将添加新目录到Python路径，但是不想硬链接到你的代码。


----------
解决方案
----------
有两种常用的方式将新目录添加到sys.path。第一种，你可以使用PYTHONPATH环境变量来添加。例如：

.. code-block:: python

    bash % env PYTHONPATH=/some/dir:/other/dir python3
    Python 3.3.0 (default, Oct 4 2012, 10:17:33)
    [GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import sys
    >>> sys.path
    ['', '/some/dir', '/other/dir', ...]
    >>>

在自定义应用程序中，这样的环境变量可在程序启动时设置或通过shell脚本。

第二种方法是创建一个.pth文件，将目录列举出来，像这样：

.. code-block:: python

    # myapplication.pth
    /some/dir
    /other/dir

这个.pth文件需要放在某个Python的site-packages目录，通常位于/usr/local/lib/python3.3/site-packages 或者 ~/.local/lib/python3.3/sitepackages。当解释器启动时，.pth文件里列举出来的存在于文件系统的目录将被添加到sys.path。安装一个.pth文件可能需要管理员权限，如果它被添加到系统级的Python解释器。


----------
讨论
----------
比起费力地找文件，你可能会倾向于写一个代码手动调节sys.path的值。例如:

.. code-block:: python

    import sys
    sys.path.insert(0, '/some/dir')
    sys.path.insert(0, '/other/dir')

虽然这能“工作”，它是在实践中极为脆弱，应尽量避免使用。这种方法的问题是，它将目录名硬编码到了你的源代码。如果你的代码被移到一个新的位置，这会导致维护问题。更好的做法是在不修改源代码的情况下，将path配置到其他地方。如果您使用模块级的变量来精心构造一个适当的绝对路径，有时你可以解决硬编码目录的问题，比如__file__。举个例子：

.. code-block:: python

    import sys
    from os.path import abspath, join, dirname
    sys.path.insert(0, join(abspath(dirname(__file__)), 'src'))

这将src目录添加到path里，和执行插入步骤的代码在同一个目录里。

site-packages目录是第三方包和模块安装的目录。如果你手动安装你的代码，它将被安装到site-packages目录。虽然用于配置path的.pth文件必须放置在site-packages里，但它配置的路径可以是系统上任何你希望的目录。因此，你可以把你的代码放在一系列不同的目录，只要那些目录包含在.pth文件里。

================================
10.10 通过字符串名导入模块
================================

----------
问题
----------
你想导入一个模块，但是模块的名字在字符串里。你想对字符串调用导入命令。

----------
解决方案
----------
使用importlib.import_module()函数来手动导入名字为字符串给出的一个模块或者包的一部分。举个例子：

.. code-block:: python

    >>> import importlib
    >>> math = importlib.import_module('math')
    >>> math.sin(2)
    0.9092974268256817
    >>> mod = importlib.import_module('urllib.request')
    >>> u = mod.urlopen('http://www.python.org')
    >>>

import_module只是简单地执行和import相同的步骤，但是返回生成的模块对象。你只需要将其存储在一个变量，然后像正常的模块一样使用。


如果你正在使用的包，import_module()也可用于相对导入。但是，你需要给它一个额外的参数。例如：

.. code-block:: python

    import importlib
    # Same as 'from . import b'
    b = importlib.import_module('.b', __package__)

----------
讨论
----------
使用import_module()手动导入模块的问题通常出现在以某种方式编写修改或覆盖模块的代码时候。例如，也许你正在执行某种自定义导入机制，需要通过名称来加载一个模块，通过补丁加载代码。


在旧的代码，有时你会看到用于导入的内建函数__import__()。尽管它能工作，但是importlib.import_module() 通常更容易使用。

自定义导入过程的高级实例见10.11小节

================================
10.11 通过钩子远程加载模块
================================

----------
问题
----------
你想自定义Python的import语句，使得它能从远程机器上面透明的加载模块。

----------
解决方案
----------
首先要提出来的是安全问题。本节讨论的思想如果没有一些额外的安全和认知机制的话会很糟糕。
也就是说，我们的主要目的是深入分析Python的import语句机制。
如果你理解了本节内部原理，你就能够为其他任何目的而自定义import。
有了这些，让我们继续向前走。

本节核心是设计导入语句的扩展功能。有很多种方法可以做这个，
不过为了演示的方便，我们开始先构造下面这个Python代码结构：

::

    testcode/
        spam.py
        fib.py
        grok/
            __init__.py
            blah.py

这些文件的内容并不重要，不过我们在每个文件中放入了少量的简单语句和函数，
这样你可以测试它们并查看当它们被导入时的输出。例如：

.. code-block:: python

    # spam.py
    print("I'm spam")

    def hello(name):
        print('Hello %s' % name)

    # fib.py
    print("I'm fib")

    def fib(n):
        if n < 2:
            return 1
        else:
            return fib(n-1) + fib(n-2)

    # grok/__init__.py
    print("I'm grok.__init__")

    # grok/blah.py
    print("I'm grok.blah")

这里的目的是允许这些文件作为模块被远程访问。
也许最简单的方式就是将它们发布到一个web服务器上面。在testcode目录中像下面这样运行Python：

::

    bash % cd testcode
    bash % python3 -m http.server 15000
    Serving HTTP on 0.0.0.0 port 15000 ...

服务器运行起来后再启动一个单独的Python解释器。
确保你可以使用 ``urllib`` 访问到远程文件。例如：

.. code-block:: python

    >>> from urllib.request import urlopen
    >>> u = urlopen('http://localhost:15000/fib.py')
    >>> data = u.read().decode('utf-8')
    >>> print(data)
    # fib.py
    print("I'm fib")

    def fib(n):
        if n < 2:
            return 1
        else:
            return fib(n-1) + fib(n-2)
    >>>

从这个服务器加载源代码是接下来本节的基础。
为了替代手动的通过 ``urlopen()`` 来收集源文件，
我们通过自定义import语句来在后台自动帮我们做到。

加载远程模块的第一种方法是创建一个显式的加载函数来完成它。例如：

.. code-block:: python

    import imp
    import urllib.request
    import sys

    def load_module(url):
        u = urllib.request.urlopen(url)
        source = u.read().decode('utf-8')
        mod = sys.modules.setdefault(url, imp.new_module(url))
        code = compile(source, url, 'exec')
        mod.__file__ = url
        mod.__package__ = ''
        exec(code, mod.__dict__)
        return mod

这个函数会下载源代码，并使用 ``compile()`` 将其编译到一个代码对象中，
然后在一个新创建的模块对象的字典中来执行它。下面是使用这个函数的方式：

.. code-block:: python

    >>> fib = load_module('http://localhost:15000/fib.py')
    I'm fib
    >>> fib.fib(10)
    89
    >>> spam = load_module('http://localhost:15000/spam.py')
    I'm spam
    >>> spam.hello('Guido')
    Hello Guido
    >>> fib
    <module 'http://localhost:15000/fib.py' from 'http://localhost:15000/fib.py'>
    >>> spam
    <module 'http://localhost:15000/spam.py' from 'http://localhost:15000/spam.py'>
    >>>

正如你所见，对于简单的模块这个是行得通的。
不过它并没有嵌入到通常的import语句中，如果要支持更高级的结构比如包就需要更多的工作了。

一个更酷的做法是创建一个自定义导入器。第一种方法是创建一个元路径导入器。如下：

.. code-block:: python

    # urlimport.py
    import sys
    import importlib.abc
    import imp
    from urllib.request import urlopen
    from urllib.error import HTTPError, URLError
    from html.parser import HTMLParser

    # Debugging
    import logging
    log = logging.getLogger(__name__)

    # Get links from a given URL
    def _get_links(url):
        class LinkParser(HTMLParser):
            def handle_starttag(self, tag, attrs):
                if tag == 'a':
                    attrs = dict(attrs)
                    links.add(attrs.get('href').rstrip('/'))
        links = set()
        try:
            log.debug('Getting links from %s' % url)
            u = urlopen(url)
            parser = LinkParser()
            parser.feed(u.read().decode('utf-8'))
        except Exception as e:
            log.debug('Could not get links. %s', e)
        log.debug('links: %r', links)
        return links

    class UrlMetaFinder(importlib.abc.MetaPathFinder):
        def __init__(self, baseurl):
            self._baseurl = baseurl
            self._links = { }
            self._loaders = { baseurl : UrlModuleLoader(baseurl) }

        def find_module(self, fullname, path=None):
            log.debug('find_module: fullname=%r, path=%r', fullname, path)
            if path is None:
                baseurl = self._baseurl
            else:
                if not path[0].startswith(self._baseurl):
                    return None
                baseurl = path[0]
            parts = fullname.split('.')
            basename = parts[-1]
            log.debug('find_module: baseurl=%r, basename=%r', baseurl, basename)

            # Check link cache
            if basename not in self._links:
                self._links[baseurl] = _get_links(baseurl)

            # Check if it's a package
            if basename in self._links[baseurl]:
                log.debug('find_module: trying package %r', fullname)
                fullurl = self._baseurl + '/' + basename
                # Attempt to load the package (which accesses __init__.py)
                loader = UrlPackageLoader(fullurl)
                try:
                    loader.load_module(fullname)
                    self._links[fullurl] = _get_links(fullurl)
                    self._loaders[fullurl] = UrlModuleLoader(fullurl)
                    log.debug('find_module: package %r loaded', fullname)
                except ImportError as e:
                    log.debug('find_module: package failed. %s', e)
                    loader = None
                return loader
            # A normal module
            filename = basename + '.py'
            if filename in self._links[baseurl]:
                log.debug('find_module: module %r found', fullname)
                return self._loaders[baseurl]
            else:
                log.debug('find_module: module %r not found', fullname)
                return None

        def invalidate_caches(self):
            log.debug('invalidating link cache')
            self._links.clear()

    # Module Loader for a URL
    class UrlModuleLoader(importlib.abc.SourceLoader):
        def __init__(self, baseurl):
            self._baseurl = baseurl
            self._source_cache = {}

        def module_repr(self, module):
            return '<urlmodule %r from %r>' % (module.__name__, module.__file__)

        # Required method
        def load_module(self, fullname):
            code = self.get_code(fullname)
            mod = sys.modules.setdefault(fullname, imp.new_module(fullname))
            mod.__file__ = self.get_filename(fullname)
            mod.__loader__ = self
            mod.__package__ = fullname.rpartition('.')[0]
            exec(code, mod.__dict__)
            return mod

        # Optional extensions
        def get_code(self, fullname):
            src = self.get_source(fullname)
            return compile(src, self.get_filename(fullname), 'exec')

        def get_data(self, path):
            pass

        def get_filename(self, fullname):
            return self._baseurl + '/' + fullname.split('.')[-1] + '.py'

        def get_source(self, fullname):
            filename = self.get_filename(fullname)
            log.debug('loader: reading %r', filename)
            if filename in self._source_cache:
                log.debug('loader: cached %r', filename)
                return self._source_cache[filename]
            try:
                u = urlopen(filename)
                source = u.read().decode('utf-8')
                log.debug('loader: %r loaded', filename)
                self._source_cache[filename] = source
                return source
            except (HTTPError, URLError) as e:
                log.debug('loader: %r failed. %s', filename, e)
                raise ImportError("Can't load %s" % filename)

        def is_package(self, fullname):
            return False

    # Package loader for a URL
    class UrlPackageLoader(UrlModuleLoader):
        def load_module(self, fullname):
            mod = super().load_module(fullname)
            mod.__path__ = [ self._baseurl ]
            mod.__package__ = fullname

        def get_filename(self, fullname):
            return self._baseurl + '/' + '__init__.py'

        def is_package(self, fullname):
            return True

    # Utility functions for installing/uninstalling the loader
    _installed_meta_cache = { }
    def install_meta(address):
        if address not in _installed_meta_cache:
            finder = UrlMetaFinder(address)
            _installed_meta_cache[address] = finder
            sys.meta_path.append(finder)
            log.debug('%r installed on sys.meta_path', finder)

    def remove_meta(address):
        if address in _installed_meta_cache:
            finder = _installed_meta_cache.pop(address)
            sys.meta_path.remove(finder)
            log.debug('%r removed from sys.meta_path', finder)

下面是一个交互会话，演示了如何使用前面的代码：

.. code-block:: python

    >>> # importing currently fails
    >>> import fib
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'
    >>> # Load the importer and retry (it works)
    >>> import urlimport
    >>> urlimport.install_meta('http://localhost:15000')
    >>> import fib
    I'm fib
    >>> import spam
    I'm spam
    >>> import grok.blah
    I'm grok.__init__
    I'm grok.blah
    >>> grok.blah.__file__
    'http://localhost:15000/grok/blah.py'
    >>>

这个特殊的方案会安装一个特别的查找器 ``UrlMetaFinder`` 实例，
作为 ``sys.meta_path`` 中最后的实体。
当模块被导入时，会依据 ``sys.meta_path`` 中的查找器定位模块。
在这个例子中，``UrlMetaFinder`` 实例是最后一个查找器方案，
当模块在任何一个普通地方都找不到的时候就触发它。

作为常见的实现方案，``UrlMetaFinder`` 类包装在一个用户指定的URL上。
在内部，查找器通过抓取指定URL的内容构建合法的链接集合。
导入的时候，模块名会跟已有的链接作对比。如果找到了一个匹配的，
一个单独的 ``UrlModuleLoader`` 类被用来从远程机器上加载源代码并创建最终的模块对象。
这里缓存链接的一个原因是避免不必要的HTTP请求重复导入。

自定义导入的第二种方法是编写一个钩子直接嵌入到 ``sys.path`` 变量中去，
识别某些目录命名模式。
在 ``urlimport.py`` 中添加如下的类和支持函数：

.. code-block:: python

    # urlimport.py
    # ... include previous code above ...
    # Path finder class for a URL
    class UrlPathFinder(importlib.abc.PathEntryFinder):
        def __init__(self, baseurl):
            self._links = None
            self._loader = UrlModuleLoader(baseurl)
            self._baseurl = baseurl

        def find_loader(self, fullname):
            log.debug('find_loader: %r', fullname)
            parts = fullname.split('.')
            basename = parts[-1]
            # Check link cache
            if self._links is None:
                self._links = [] # See discussion
                self._links = _get_links(self._baseurl)

            # Check if it's a package
            if basename in self._links:
                log.debug('find_loader: trying package %r', fullname)
                fullurl = self._baseurl + '/' + basename
                # Attempt to load the package (which accesses __init__.py)
                loader = UrlPackageLoader(fullurl)
                try:
                    loader.load_module(fullname)
                    log.debug('find_loader: package %r loaded', fullname)
                except ImportError as e:
                    log.debug('find_loader: %r is a namespace package', fullname)
                    loader = None
                return (loader, [fullurl])

            # A normal module
            filename = basename + '.py'
            if filename in self._links:
                log.debug('find_loader: module %r found', fullname)
                return (self._loader, [])
            else:
                log.debug('find_loader: module %r not found', fullname)
                return (None, [])

        def invalidate_caches(self):
            log.debug('invalidating link cache')
            self._links = None

    # Check path to see if it looks like a URL
    _url_path_cache = {}
    def handle_url(path):
        if path.startswith(('http://', 'https://')):
            log.debug('Handle path? %s. [Yes]', path)
            if path in _url_path_cache:
                finder = _url_path_cache[path]
            else:
                finder = UrlPathFinder(path)
                _url_path_cache[path] = finder
            return finder
        else:
            log.debug('Handle path? %s. [No]', path)

    def install_path_hook():
        sys.path_hooks.append(handle_url)
        sys.path_importer_cache.clear()
        log.debug('Installing handle_url')

    def remove_path_hook():
        sys.path_hooks.remove(handle_url)
        sys.path_importer_cache.clear()
        log.debug('Removing handle_url')

要使用这个路径查找器，你只需要在 ``sys.path`` 中加入URL链接。例如：

.. code-block:: python

    >>> # Initial import fails
    >>> import fib
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'

    >>> # Install the path hook
    >>> import urlimport
    >>> urlimport.install_path_hook()

    >>> # Imports still fail (not on path)
    >>> import fib
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'

    >>> # Add an entry to sys.path and watch it work
    >>> import sys
    >>> sys.path.append('http://localhost:15000')
    >>> import fib
    I'm fib
    >>> import grok.blah
    I'm grok.__init__
    I'm grok.blah
    >>> grok.blah.__file__
    'http://localhost:15000/grok/blah.py'
    >>>

关键点就是 ``handle_url()`` 函数，它被添加到了 ``sys.path_hooks`` 变量中。
当 ``sys.path`` 的实体被处理时，会调用 ``sys.path_hooks`` 中的函数。
如果任何一个函数返回了一个查找器对象，那么这个对象就被用来为 ``sys.path`` 实体加载模块。

远程模块加载跟其他的加载使用方法几乎是一样的。例如：

.. code-block:: python

    >>> fib
    <urlmodule 'fib' from 'http://localhost:15000/fib.py'>
    >>> fib.__name__
    'fib'
    >>> fib.__file__
    'http://localhost:15000/fib.py'
    >>> import inspect
    >>> print(inspect.getsource(fib))
    # fib.py
    print("I'm fib")

    def fib(n):
        if n < 2:
            return 1
        else:
            return fib(n-1) + fib(n-2)
    >>>

----------
讨论
----------
在详细讨论之前，有点要强调的是，Python的模块、包和导入机制是整个语言中最复杂的部分，
即使经验丰富的Python程序员也很少能精通它们。
我在这里推荐一些值的去读的文档和书籍，包括
`importlib module <https://docs.python.org/3/library/importlib.html>`_
和 `PEP 302 <http://www.python.org/dev/peps/pep-0302>`_.
文档内容在这里不会被重复提到，不过我在这里会讨论一些最重要的部分。

首先，如果你想创建一个新的模块对象，使用 ``imp.new_module()`` 函数：

.. code-block:: python

    >>> import imp
    >>> m = imp.new_module('spam')
    >>> m
    <module 'spam'>
    >>> m.__name__
    'spam'
    >>>

模块对象通常有一些期望属性，包括 ``__file__`` （运行模块加载语句的文件名）
和 ``__package__`` (包名)。

其次，模块会被解释器缓存起来。模块缓存可以在字典 ``sys.modules`` 中被找到。
因为有了这个缓存机制，通常可以将缓存和模块的创建通过一个步骤完成：

.. code-block:: python

    >>> import sys
    >>> import imp
    >>> m = sys.modules.setdefault('spam', imp.new_module('spam'))
    >>> m
    <module 'spam'>
    >>>

如果给定模块已经存在那么就会直接获得已经被创建过的模块，例如：

.. code-block:: python

    >>> import math
    >>> m = sys.modules.setdefault('math', imp.new_module('math'))
    >>> m
    <module 'math' from '/usr/local/lib/python3.3/lib-dynload/math.so'>
    >>> m.sin(2)
    0.9092974268256817
    >>> m.cos(2)
    -0.4161468365471424
    >>>

由于创建模块很简单，很容易编写简单函数比如第一部分的 ``load_module()`` 函数。
这个方案的一个缺点是很难处理复杂情况比如包的导入。
为了处理一个包，你要重新实现普通import语句的底层逻辑（比如检查目录，查找__init__.py文件，
执行那些文件，设置路径等）。这个复杂性就是为什么最好直接扩展import语句而不是自定义函数的一个原因。

扩展import语句很简单，但是会有很多移动操作。
最高层上，导入操作被一个位于sys.meta_path列表中的“元路径”查找器处理。
如果你输出它的值，会看到下面这样：

.. code-block:: python

    >>> from pprint import pprint
    >>> pprint(sys.meta_path)
    [<class '_frozen_importlib.BuiltinImporter'>,
    <class '_frozen_importlib.FrozenImporter'>,
    <class '_frozen_importlib.PathFinder'>]
    >>>

当执行一个语句比如 ``import fib`` 时，解释器会遍历sys.mata_path中的查找器对象，
调用它们的 ``find_module()`` 方法定位正确的模块加载器。
可以通过实验来看看：

.. code-block:: python

    >>> class Finder:
    ...     def find_module(self, fullname, path):
    ...         print('Looking for', fullname, path)
    ...         return None
    ...
    >>> import sys
    >>> sys.meta_path.insert(0, Finder()) # Insert as first entry
    >>> import math
    Looking for math None
    >>> import types
    Looking for types None
    >>> import threading
    Looking for threading None
    Looking for time None
    Looking for traceback None
    Looking for linecache None
    Looking for tokenize None
    Looking for token None
    >>>

注意看 ``find_module()`` 方法是怎样在每一个导入就被触发的。
这个方法中的path参数的作用是处理包。
多个包被导入，就是一个可在包的 ``__path__`` 属性中找到的路径列表。
要找到包的子组件就要检查这些路径。
比如注意对于 ``xml.etree`` 和 ``xml.etree.ElementTree`` 的路径配置：

.. code-block:: python

    >>> import xml.etree.ElementTree
    Looking for xml None
    Looking for xml.etree ['/usr/local/lib/python3.3/xml']
    Looking for xml.etree.ElementTree ['/usr/local/lib/python3.3/xml/etree']
    Looking for warnings None
    Looking for contextlib None
    Looking for xml.etree.ElementPath ['/usr/local/lib/python3.3/xml/etree']
    Looking for _elementtree None
    Looking for copy None
    Looking for org None
    Looking for pyexpat None
    Looking for ElementC14N None
    >>>

在 ``sys.meta_path`` 上查找器的位置很重要，将它从队头移到队尾，然后再试试导入看：

.. code-block:: python

    >>> del sys.meta_path[0]
    >>> sys.meta_path.append(Finder())
    >>> import urllib.request
    >>> import datetime

现在你看不到任何输出了，因为导入被sys.meta_path中的其他实体处理。
这时候，你只有在导入不存在模块的时候才能看到它被触发：

.. code-block:: python

    >>> import fib
    Looking for fib None
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'
    >>> import xml.superfast
    Looking for xml.superfast ['/usr/local/lib/python3.3/xml']
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'xml.superfast'
    >>>

你之前安装过一个捕获未知模块的查找器，这个是 ``UrlMetaFinder`` 类的关键。
一个 ``UrlMetaFinder`` 实例被添加到 ``sys.meta_path`` 的末尾，作为最后一个查找器方案。
如果被请求的模块名不能定位，就会被这个查找器处理掉。
处理包的时候需要注意，在path参数中指定的值需要被检查，看它是否以查找器中注册的URL开头。
如果不是，该子模块必须归属于其他查找器并被忽略掉。

对于包的其他处理可在 ``UrlPackageLoader`` 类中被找到。
这个类不会导入包名，而是去加载对应的 ``__init__.py`` 文件。
它也会设置模块的 ``__path__`` 属性，这一步很重要，
因为在加载包的子模块时这个值会被传给后面的 ``find_module()`` 调用。
基于路径的导入钩子是这些思想的一个扩展，但是采用了另外的方法。
我们都知道，``sys.path`` 是一个Python查找模块的目录列表，例如：


.. code-block:: python

    >>> from pprint import pprint
    >>> import sys
    >>> pprint(sys.path)
    ['',
    '/usr/local/lib/python33.zip',
    '/usr/local/lib/python3.3',
    '/usr/local/lib/python3.3/plat-darwin',
    '/usr/local/lib/python3.3/lib-dynload',
    '/usr/local/lib/...3.3/site-packages']
    >>>

在 ``sys.path`` 中的每一个实体都会被额外的绑定到一个查找器对象上。
你可以通过查看 ``sys.path_importer_cache`` 去看下这些查找器：

.. code-block:: python

    >>> pprint(sys.path_importer_cache)
    {'.': FileFinder('.'),
    '/usr/local/lib/python3.3': FileFinder('/usr/local/lib/python3.3'),
    '/usr/local/lib/python3.3/': FileFinder('/usr/local/lib/python3.3/'),
    '/usr/local/lib/python3.3/collections': FileFinder('...python3.3/collections'),
    '/usr/local/lib/python3.3/encodings': FileFinder('...python3.3/encodings'),
    '/usr/local/lib/python3.3/lib-dynload': FileFinder('...python3.3/lib-dynload'),
    '/usr/local/lib/python3.3/plat-darwin': FileFinder('...python3.3/plat-darwin'),
    '/usr/local/lib/python3.3/site-packages': FileFinder('...python3.3/site-packages'),
    '/usr/local/lib/python33.zip': None}
    >>>

``sys.path_importer_cache`` 比 ``sys.path`` 会更大点，
因为它会为所有被加载代码的目录记录它们的查找器。
这包括包的子目录，这些通常在 ``sys.path`` 中是不存在的。

要执行 ``import fib`` ，会顺序检查 ``sys.path`` 中的目录。
对于每个目录，名称“fib”会被传给相应的 ``sys.path_importer_cache`` 中的查找器。
这个可以让你创建自己的查找器并在缓存中放入一个实体。试试这个：

.. code-block:: python

    >>> class Finder:
    ... def find_loader(self, name):
    ...     print('Looking for', name)
    ...     return (None, [])
    ...
    >>> import sys
    >>> # Add a "debug" entry to the importer cache
    >>> sys.path_importer_cache['debug'] = Finder()
    >>> # Add a "debug" directory to sys.path
    >>> sys.path.insert(0, 'debug')
    >>> import threading
    Looking for threading
    Looking for time
    Looking for traceback
    Looking for linecache
    Looking for tokenize
    Looking for token
    >>>

在这里，你可以为名字“debug”创建一个新的缓存实体并将它设置成 ``sys.path`` 上的第一个。
在所有接下来的导入中，你会看到你的查找器被触发了。
不过，由于它返回 (None, [])，那么处理进程会继续处理下一个实体。

``sys.path_importer_cache`` 的使用被一个存储在 ``sys.path_hooks`` 中的函数列表控制。
试试下面的例子，它会清除缓存并给 ``sys.path_hooks`` 添加一个新的路径检查函数

.. code-block:: python

    >>> sys.path_importer_cache.clear()
    >>> def check_path(path):
    ...     print('Checking', path)
    ...     raise ImportError()
    ...
    >>> sys.path_hooks.insert(0, check_path)
    >>> import fib
    Checked debug
    Checking .
    Checking /usr/local/lib/python33.zip
    Checking /usr/local/lib/python3.3
    Checking /usr/local/lib/python3.3/plat-darwin
    Checking /usr/local/lib/python3.3/lib-dynload
    Checking /Users/beazley/.local/lib/python3.3/site-packages
    Checking /usr/local/lib/python3.3/site-packages
    Looking for fib
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'
    >>>

正如你所见，``check_path()`` 函数被每个 ``sys.path`` 中的实体调用。
不顾，由于抛出了 ``ImportError`` 异常，
啥都不会发生了（仅仅将检查转移到sys.path_hooks的下一个函数）。

知道了怎样sys.path是怎样被处理的，你就能构建一个自定义路径检查函数来查找文件名，不然URL。例如：

.. code-block:: python

    >>> def check_url(path):
    ...     if path.startswith('http://'):
    ...         return Finder()
    ...     else:
    ...         raise ImportError()
    ...
    >>> sys.path.append('http://localhost:15000')
    >>> sys.path_hooks[0] = check_url
    >>> import fib
    Looking for fib # Finder output!
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'

    >>> # Notice installation of Finder in sys.path_importer_cache
    >>> sys.path_importer_cache['http://localhost:15000']
    <__main__.Finder object at 0x10064c850>
    >>>

这就是本节最后部分的关键点。事实上，一个用来在sys.path中查找URL的自定义路径检查函数已经构建完毕。
当它们被碰到的时候，一个新的 ``UrlPathFinder`` 实例被创建并被放入 ``sys.path_importer_cache``.
之后，所有需要检查 ``sys.path`` 的导入语句都会使用你的自定义查找器。

基于路径导入的包处理稍微有点复杂，并且跟 ``find_loader()`` 方法返回值有关。
对于简单模块，``find_loader()`` 返回一个元组(loader, None)，
其中的loader是一个用于导入模块的加载器实例。

对于一个普通的包，``find_loader()`` 返回一个元组(loader, path)，
其中的loader是一个用于导入包（并执行__init__.py）的加载器实例，
path是一个会初始化包的 ``__path__`` 属性的目录列表。
例如，如果基础URL是 http://localhost:15000 并且一个用户执行 ``import grok`` ,
那么 ``find_loader()`` 返回的path就会是 [ 'http://localhost:15000/grok' ]

``find_loader()`` 还要能处理一个命名空间包。
一个命名空间包中有一个合法的包目录名，但是不存在__init__.py文件。
这样的话，``find_loader()`` 必须返回一个元组(None, path)，
path是一个目录列表，由它来构建包的定义有__init__.py文件的__path__属性。
对于这种情况，导入机制会继续前行去检查sys.path中的目录。
如果找到了命名空间包，所有的结果路径被加到一起来构建最终的命名空间包。
关于命名空间包的更多信息请参考10.5小节。

所有的包都包含了一个内部路径设置，可以在__path__属性中看到，例如：

.. code-block:: python

    >>> import xml.etree.ElementTree
    >>> xml.__path__
    ['/usr/local/lib/python3.3/xml']
    >>> xml.etree.__path__
    ['/usr/local/lib/python3.3/xml/etree']
    >>>

之前提到，__path__的设置是通过 ``find_loader()`` 方法返回值控制的。
不过，__path__接下来也被sys.path_hooks中的函数处理。
因此，但包的子组件被加载后，位于__path__中的实体会被 ``handle_url()`` 函数检查。
这会导致新的 ``UrlPathFinder`` 实例被创建并且被加入到 ``sys.path_importer_cache`` 中。

还有个难点就是 ``handle_url()`` 函数以及它跟内部使用的 ``_get_links()`` 函数之间的交互。
如果你的查找器实现需要使用到其他模块（比如urllib.request），
有可能这些模块会在查找器操作期间进行更多的导入。
它可以导致 ``handle_url()`` 和其他查找器部分陷入一种递归循环状态。
为了解释这种可能性，实现中有一个被创建的查找器缓存（每一个URL一个）。
它可以避免创建重复查找器的问题。
另外，下面的代码片段可以确保查找器不会在初始化链接集合的时候响应任何导入请求：

.. code-block:: python

    # Check link cache
    if self._links is None:
        self._links = [] # See discussion
        self._links = _get_links(self._baseurl)

最后，查找器的 ``invalidate_caches()`` 方法是一个工具方法，用来清理内部缓存。
这个方法再用户调用 ``importlib.invalidate_caches()`` 的时候被触发。
如果你想让URL导入者重新读取链接列表的话可以使用它。

对比下两种方案（修改sys.meta_path或使用一个路径钩子）。
使用sys.meta_path的导入者可以按照自己的需要自由处理模块。
例如，它们可以从数据库中导入或以不同于一般模块/包处理方式导入。
这种自由同样意味着导入者需要自己进行内部的一些管理。
另外，基于路径的钩子只是适用于对sys.path的处理。
通过这种扩展加载的模块跟普通方式加载的特性是一样的。

如果到现在为止你还是不是很明白，那么可以通过增加一些日志打印来测试下本节。像下面这样：

.. code-block:: python

    >>> import logging
    >>> logging.basicConfig(level=logging.DEBUG)
    >>> import urlimport
    >>> urlimport.install_path_hook()
    DEBUG:urlimport:Installing handle_url
    >>> import fib
    DEBUG:urlimport:Handle path? /usr/local/lib/python33.zip. [No]
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    ImportError: No module named 'fib'
    >>> import sys
    >>> sys.path.append('http://localhost:15000')
    >>> import fib
    DEBUG:urlimport:Handle path? http://localhost:15000. [Yes]
    DEBUG:urlimport:Getting links from http://localhost:15000
    DEBUG:urlimport:links: {'spam.py', 'fib.py', 'grok'}
    DEBUG:urlimport:find_loader: 'fib'
    DEBUG:urlimport:find_loader: module 'fib' found
    DEBUG:urlimport:loader: reading 'http://localhost:15000/fib.py'
    DEBUG:urlimport:loader: 'http://localhost:15000/fib.py' loaded
    I'm fib
    >>>

最后，建议你花点时间看看 `PEP 302 <http://www.python.org/dev/peps/pep-0302>`_
以及importlib的文档。
================================
10.12 导入模块的同时修改模块
================================

----------
问题
----------
你想给某个已存在模块中的函数添加装饰器。
不过，前提是这个模块已经被导入并且被使用过。

----------
解决方案
----------
这里问题的本质就是你想在模块被加载时执行某个动作。
可能是你想在一个模块被加载时触发某个回调函数来通知你。

这个问题可以使用10.11小节中同样的导入钩子机制来实现。下面是一个可能的方案：

.. code-block:: python

    # postimport.py
    import importlib
    import sys
    from collections import defaultdict

    _post_import_hooks = defaultdict(list)

    class PostImportFinder:
        def __init__(self):
            self._skip = set()

        def find_module(self, fullname, path=None):
            if fullname in self._skip:
                return None
            self._skip.add(fullname)
            return PostImportLoader(self)

    class PostImportLoader:
        def __init__(self, finder):
            self._finder = finder

        def load_module(self, fullname):
            importlib.import_module(fullname)
            module = sys.modules[fullname]
            for func in _post_import_hooks[fullname]:
                func(module)
            self._finder._skip.remove(fullname)
            return module

    def when_imported(fullname):
        def decorate(func):
            if fullname in sys.modules:
                func(sys.modules[fullname])
            else:
                _post_import_hooks[fullname].append(func)
            return func
        return decorate

    sys.meta_path.insert(0, PostImportFinder())

这样，你就可以使用 ``when_imported()`` 装饰器了，例如：

.. code-block:: python

    >>> from postimport import when_imported
    >>> @when_imported('threading')
    ... def warn_threads(mod):
    ...     print('Threads? Are you crazy?')
    ...
    >>>
    >>> import threading
    Threads? Are you crazy?
    >>>

作为一个更实际的例子，你可能想在已存在的定义上面添加装饰器，如下所示：

.. code-block:: python

    from functools import wraps
    from postimport import when_imported

    def logged(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print('Calling', func.__name__, args, kwargs)
            return func(*args, **kwargs)
        return wrapper

    # Example
    @when_imported('math')
    def add_logging(mod):
        mod.cos = logged(mod.cos)
        mod.sin = logged(mod.sin)

----------
讨论
----------
本节技术依赖于10.11小节中讲述过的导入钩子，并稍作修改。

``@when_imported`` 装饰器的作用是注册在导入时被激活的处理器函数。
该装饰器检查sys.modules来查看模块是否真的已经被加载了。
如果是的话，该处理器被立即调用。不然，处理器被添加到 ``_post_import_hooks`` 字典中的一个列表中去。
``_post_import_hooks`` 的作用就是收集所有的为每个模块注册的处理器对象。
一个模块可以注册多个处理器。

要让模块导入后触发添加的动作，``PostImportFinder`` 类被设置为sys.meta_path第一个元素。
它会捕获所有模块导入操作。

本节中的 ``PostImportFinder`` 的作用并不是加载模块，而是自带导入完成后触发相应的动作。
实际的导入被委派给位于sys.meta_path中的其他查找器。
``PostImportLoader`` 类中的 ``imp.import_module()`` 函数被递归的调用。
为了避免陷入无线循环，``PostImportFinder`` 保持了一个所有被加载过的模块集合。
如果一个模块名存在就会直接被忽略掉。

当一个模块被 ``imp.import_module()`` 加载后，
所有在_post_import_hooks被注册的处理器被调用，使用新加载模块作为一个参数。

有一点需要注意的是本机不适用于那些通过 ``imp.reload()`` 被显式加载的模块。
也就是说，如果你加载一个之前已被加载过的模块，那么导入处理器将不会再被触发。
另外，要是你从sys.modules中删除模块然后再重新导入，处理器又会再一次触发。

更多关于导入后钩子信息请参考 `PEP 369 <https://www.python.org/dev/peps/pep-0369>`_.


================================
10.13 安装私有的包
================================

----------
问题
----------
你想要安装一个第三方包，但是没有权限将它安装到系统Python库中去。
或者，你可能想要安装一个供自己使用的包，而不是系统上面所有用户。

----------
解决方案
----------
Python有一个用户安装目录，通常类似"~/.local/lib/python3.3/site-packages"。
要强制在这个目录中安装包，可使用安装选项“--user”。例如：

.. code-block:: python

    python3 setup.py install --user

或者

.. code-block:: python

    pip install --user packagename

在sys.path中用户的“site-packages”目录位于系统的“site-packages”目录之前。
因此，你安装在里面的包就比系统已安装的包优先级高
（尽管并不总是这样，要取决于第三方包管理器，比如distribute或pip）。


----------
讨论
----------
通常包会被安装到系统的site-packages目录中去，路径类似“/usr/local/lib/python3.3/site-packages”。
不过，这样做需要有管理员权限并且使用sudo命令。
就算你有这样的权限去执行命令，使用sudo去安装一个新的，可能没有被验证过的包有时候也不安全。

安装包到用户目录中通常是一个有效的方案，它允许你创建一个自定义安装。

另外，你还可以创建一个虚拟环境，这个我们在下一节会讲到。
================================
10.14 创建新的Python环境
================================

----------
问题
----------
你想创建一个新的Python环境，用来安装模块和包。
不过，你不想安装一个新的Python克隆，也不想对系统Python环境产生影响。

----------
解决方案
----------
你可以使用 ``pyvenv`` 命令创建一个新的“虚拟”环境。
这个命令被安装在Python解释器同一目录，或Windows上面的Scripts目录中。下面是一个例子：

.. code-block:: python

    bash % pyvenv Spam
    bash %

传给 ``pyvenv`` 命令的名字是将要被创建的目录名。当被创建后，Span目录像下面这样：

.. code-block:: python

    bash % cd Spam
    bash % ls
    bin include lib pyvenv.cfg
    bash %

在bin目录中，你会找到一个可以使用的Python解释器：

.. code-block:: python

    bash % Spam/bin/python3
    Python 3.3.0 (default, Oct 6 2012, 15:45:22)
    [GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> from pprint import pprint
    >>> import sys
    >>> pprint(sys.path)
    ['',
    '/usr/local/lib/python33.zip',
    '/usr/local/lib/python3.3',
    '/usr/local/lib/python3.3/plat-darwin',
    '/usr/local/lib/python3.3/lib-dynload',
    '/Users/beazley/Spam/lib/python3.3/site-packages']
    >>>

这个解释器的特点就是他的site-packages目录被设置为新创建的环境。
如果你要安装第三方包，它们会被安装在那里，而不是通常系统的site-packages目录。

----------
讨论
----------
创建虚拟环境通常是为了安装和管理第三方包。
正如你在例子中看到的那样，``sys.path`` 变量包含来自于系统Python的目录，
而 site-packages目录已经被重定位到一个新的目录。

有了一个新的虚拟环境，下一步就是安装一个包管理器，比如distribute或pip。
但安装这样的工具和包的时候，你需要确保你使用的是虚拟环境的解释器。
它会将包安装到新创建的site-packages目录中去。

尽管一个虚拟环境看上去是Python安装的一个复制，
不过它实际上只包含了少量几个文件和一些符号链接。
所有标准库函文件和可执行解释器都来自原来的Python安装。
因此，创建这样的环境是很容易的，并且几乎不会消耗机器资源。

默认情况下，虚拟环境是空的，不包含任何额外的第三方库。如果你想将一个已经安装的包作为虚拟环境的一部分，
可以使用“--system-site-packages”选项来创建虚拟环境，例如：

.. code-block:: python

    bash % pyvenv --system-site-packages Spam
    bash %

跟多关于 ``pyvenv`` 和虚拟环境的信息可以参考
`PEP 405 <https://www.python.org/dev/peps/pep-0405/>`_.
================================
10.15 分发包
================================

----------
问题
----------
你已经编写了一个有用的库，想将它分享给其他人。

----------
解决方案
----------
如果你想分发你的代码，第一件事就是给它一个唯一的名字，并且清理它的目录结构。
例如，一个典型的函数库包会类似下面这样：

.. code-block:: python

    projectname/
        README.txt
        Doc/
            documentation.txt
        projectname/
            __init__.py
            foo.py
            bar.py
            utils/
                __init__.py
                spam.py
                grok.py
        examples/
            helloworld.py
            ...

要让你的包可以发布出去，首先你要编写一个 ``setup.py`` ，类似下面这样：

.. code-block:: python

    # setup.py
    from distutils.core import setup

    setup(name='projectname',
        version='1.0',
        author='Your Name',
        author_email='you@youraddress.com',
        url='http://www.you.com/projectname',
        packages=['projectname', 'projectname.utils'],
    )

下一步，就是创建一个 ``MANIFEST.in`` 文件，列出所有在你的包中需要包含进来的非源码文件：

.. code-block:: python

    # MANIFEST.in
    include *.txt
    recursive-include examples *
    recursive-include Doc *

确保 ``setup.py`` 和 ``MANIFEST.in`` 文件放在你的包的最顶级目录中。
一旦你已经做了这些，你就可以像下面这样执行命令来创建一个源码分发包了：

.. code-block:: python

    % bash python3 setup.py sdist

它会创建一个文件比如"projectname-1.0.zip" 或 “projectname-1.0.tar.gz”,
具体依赖于你的系统平台。如果一切正常，
这个文件就可以发送给别人使用或者上传至 `Python Package Index <http://pypi.python.org/>`_.

----------
讨论
----------
对于纯Python代码，编写一个普通的 ``setup.py`` 文件通常很简单。
一个可能的问题是你必须手动列出所有构成包源码的子目录。
一个常见错误就是仅仅只列出一个包的最顶级目录，忘记了包含包的子组件。
这也是为什么在 ``setup.py`` 中对于包的说明包含了列表
``packages=['projectname', 'projectname.utils']``

大部分Python程序员都知道，有很多第三方包管理器供选择，包括setuptools、distribute等等。
有些是为了替代标准库中的distutils。注意如果你依赖这些包，
用户可能不能安装你的软件，除非他们已经事先安装过所需要的包管理器。
正因如此，你更应该时刻记住越简单越好的道理。
最好让你的代码使用标准的Python 3安装。
如果其他包也需要的话，可以通过一个可选项来支持。

对于涉及到C扩展的代码打包与分发就更复杂点了。
第15章对关于C扩展的这方面知识有一些详细讲解，特别是在15.2小节中。
============================
11.1 作为客户端与HTTP服务交互
============================

----------
问题
----------
你需要通过HTTP协议以客户端的方式访问多种服务。例如，下载数据或者与基于REST的API进行交互。

----------
解决方案
----------
对于简单的事情来说，通常使用 ``urllib.request`` 模块就够了。例如，发送一个简单的HTTP GET请求到远程的服务上，可以这样做：

.. code-block:: python

    from urllib import request, parse

    # Base URL being accessed
    url = 'http://httpbin.org/get'

    # Dictionary of query parameters (if any)
    parms = {
       'name1' : 'value1',
       'name2' : 'value2'
    }

    # Encode the query string
    querystring = parse.urlencode(parms)

    # Make a GET request and read the response
    u = request.urlopen(url+'?' + querystring)
    resp = u.read()

如果你需要使用POST方法在请求主体中发送查询参数，可以将参数编码后作为可选参数提供给 ``urlopen()`` 函数，就像这样：

.. code-block:: python

    from urllib import request, parse

    # Base URL being accessed
    url = 'http://httpbin.org/post'

    # Dictionary of query parameters (if any)
    parms = {
       'name1' : 'value1',
       'name2' : 'value2'
    }

    # Encode the query string
    querystring = parse.urlencode(parms)

    # Make a POST request and read the response
    u = request.urlopen(url, querystring.encode('ascii'))
    resp = u.read()

如果你需要在发出的请求中提供一些自定义的HTTP头，例如修改 ``user-agent`` 字段,可以创建一个包含字段值的字典，并创建一个Request实例然后将其传给 ``urlopen()`` ，如下：

.. code-block:: python

    from urllib import request, parse
    ...

    # Extra headers
    headers = {
        'User-agent' : 'none/ofyourbusiness',
        'Spam' : 'Eggs'
    }

    req = request.Request(url, querystring.encode('ascii'), headers=headers)

    # Make a request and read the response
    u = request.urlopen(req)
    resp = u.read()

如果需要交互的服务比上面的例子都要复杂，也许应该去看看 requests 库（https://pypi.python.org/pypi/requests）。例如，下面这个示例采用requests库重新实现了上面的操作：

.. code-block:: python

    import requests

    # Base URL being accessed
    url = 'http://httpbin.org/post'

    # Dictionary of query parameters (if any)
    parms = {
       'name1' : 'value1',
       'name2' : 'value2'
    }

    # Extra headers
    headers = {
        'User-agent' : 'none/ofyourbusiness',
        'Spam' : 'Eggs'
    }

    resp = requests.post(url, data=parms, headers=headers)

    # Decoded text returned by the request
    text = resp.text

关于requests库，一个值得一提的特性就是它能以多种方式从请求中返回响应结果的内容。从上面的代码来看， ``resp.text`` 带给我们的是以Unicode解码的响应文本。但是，如果去访问 ``resp.content`` ，就会得到原始的二进制数据。另一方面，如果访问 ``resp.json`` ，那么就会得到JSON格式的响应内容。

下面这个示例利用 ``requests`` 库发起一个HEAD请求，并从响应中提取出一些HTTP头数据的字段：

.. code-block:: python

    import requests

    resp = requests.head('http://www.python.org/index.html')

    status = resp.status_code
    last_modified = resp.headers['last-modified']
    content_type = resp.headers['content-type']
    content_length = resp.headers['content-length']

下面是一个利用requests通过基本认证登录Pypi的例子：

.. code-block:: python

    import requests

    resp = requests.get('http://pypi.python.org/pypi?:action=login',
                        auth=('user','password'))

下面是一个利用requests将HTTP cookies从一个请求传递到另一个的例子：

.. code-block:: python

    import requests

    # First request
    resp1 = requests.get(url)
    ...

    # Second requests with cookies received on first requests
    resp2 = requests.get(url, cookies=resp1.cookies)

最后但并非最不重要的一个例子是用requests上传内容：

.. code-block:: python

    import requests
    url = 'http://httpbin.org/post'
    files = { 'file': ('data.csv', open('data.csv', 'rb')) }

    r = requests.post(url, files=files)


----------
讨论
----------
对于真的很简单HTTP客户端代码，用内置的 ``urllib`` 模块通常就足够了。但是，如果你要做的不仅仅只是简单的GET或POST请求，那就真的不能再依赖它的功能了。这时候就是第三方模块比如 ``requests`` 大显身手的时候了。

例如，如果你决定坚持使用标准的程序库而不考虑像 ``requests`` 这样的第三方库，那么也许就不得不使用底层的 ``http.client`` 模块来实现自己的代码。比方说，下面的代码展示了如何执行一个HEAD请求：

.. code-block:: python

    from http.client import HTTPConnection
    from urllib import parse

    c = HTTPConnection('www.python.org', 80)
    c.request('HEAD', '/index.html')
    resp = c.getresponse()

    print('Status', resp.status)
    for name, value in resp.getheaders():
        print(name, value)


同样地，如果必须编写涉及代理、认证、cookies以及其他一些细节方面的代码，那么使用 ``urllib`` 就显得特别别扭和啰嗦。比方说，下面这个示例实现在Python包索引上的认证：

.. code-block:: python

    import urllib.request

    auth = urllib.request.HTTPBasicAuthHandler()
    auth.add_password('pypi','http://pypi.python.org','username','password')
    opener = urllib.request.build_opener(auth)

    r = urllib.request.Request('http://pypi.python.org/pypi?:action=login')
    u = opener.open(r)
    resp = u.read()

    # From here. You can access more pages using opener
    ...

坦白说，所有的这些操作在 ``requests`` 库中都变得简单的多。

在开发过程中测试HTTP客户端代码常常是很令人沮丧的，因为所有棘手的细节问题都需要考虑（例如cookies、认证、HTTP头、编码方式等）。要完成这些任务，考虑使用httpbin服务（http://httpbin.org）。这个站点会接收发出的请求，然后以JSON的形式将相应信息回传回来。下面是一个交互式的例子：

.. code-block:: python

    >>> import requests
    >>> r = requests.get('http://httpbin.org/get?name=Dave&n=37',
    ...     headers = { 'User-agent': 'goaway/1.0' })
    >>> resp = r.json
    >>> resp['headers']
    {'User-Agent': 'goaway/1.0', 'Content-Length': '', 'Content-Type': '',
    'Accept-Encoding': 'gzip, deflate, compress', 'Connection':
    'keep-alive', 'Host': 'httpbin.org', 'Accept': '*/*'}
    >>> resp['args']
    {'name': 'Dave', 'n': '37'}
    >>>

在要同一个真正的站点进行交互前，先在 httpbin.org 这样的网站上做实验常常是可取的办法。尤其是当我们面对3次登录失败就会关闭账户这样的风险时尤为有用（不要尝试自己编写HTTP认证客户端来登录你的银行账户）。

尽管本节没有涉及， ``request`` 库还对许多高级的HTTP客户端协议提供了支持，比如OAuth。 ``requests`` 模块的文档（http://docs.python-requests.org)质量很高（坦白说比在这短短的一节的篇幅中所提供的任何信息都好），可以参考文档以获得更多地信息。
============================
11.2 创建TCP服务器
============================

----------
问题
----------
你想实现一个服务器，通过TCP协议和客户端通信。

----------
解决方案
----------
创建一个TCP服务器的一个简单方法是使用 ``socketserver`` 库。例如，下面是一个简单的应答服务器：

.. code-block:: python

    from socketserver import BaseRequestHandler, TCPServer

    class EchoHandler(BaseRequestHandler):
        def handle(self):
            print('Got connection from', self.client_address)
            while True:

                msg = self.request.recv(8192)
                if not msg:
                    break
                self.request.send(msg)

    if __name__ == '__main__':
        serv = TCPServer(('', 20000), EchoHandler)
        serv.serve_forever()

在这段代码中，你定义了一个特殊的处理类，实现了一个 ``handle()`` 方法，用来为客户端连接服务。
``request`` 属性是客户端socket，``client_address`` 有客户端地址。
为了测试这个服务器，运行它并打开另外一个Python进程连接这个服务器：

.. code-block:: python

    >>> from socket import socket, AF_INET, SOCK_STREAM
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s.connect(('localhost', 20000))
    >>> s.send(b'Hello')
    5
    >>> s.recv(8192)
    b'Hello'
    >>>

很多时候，可以很容易的定义一个不同的处理器。下面是一个使用 ``StreamRequestHandler``
基类将一个类文件接口放置在底层socket上的例子：

.. code-block:: python

    from socketserver import StreamRequestHandler, TCPServer

    class EchoHandler(StreamRequestHandler):
        def handle(self):
            print('Got connection from', self.client_address)
            # self.rfile is a file-like object for reading
            for line in self.rfile:
                # self.wfile is a file-like object for writing
                self.wfile.write(line)

    if __name__ == '__main__':
        serv = TCPServer(('', 20000), EchoHandler)
        serv.serve_forever()

----------
讨论
----------
``socketserver`` 可以让我们很容易的创建简单的TCP服务器。
但是，你需要注意的是，默认情况下这种服务器是单线程的，一次只能为一个客户端连接服务。
如果你想处理多个客户端，可以初始化一个 ``ForkingTCPServer`` 或者是 ``ThreadingTCPServer`` 对象。例如：

.. code-block:: python

    from socketserver import ThreadingTCPServer


    if __name__ == '__main__':
        serv = ThreadingTCPServer(('', 20000), EchoHandler)
        serv.serve_forever()

使用fork或线程服务器有个潜在问题就是它们会为每个客户端连接创建一个新的进程或线程。
由于客户端连接数是没有限制的，因此一个恶意的黑客可以同时发送大量的连接让你的服务器奔溃。

如果你担心这个问题，你可以创建一个预先分配大小的工作线程池或进程池。
你先创建一个普通的非线程服务器，然后在一个线程池中使用 ``serve_forever()`` 方法来启动它们。

.. code-block:: python

    if __name__ == '__main__':
        from threading import Thread
        NWORKERS = 16
        serv = TCPServer(('', 20000), EchoHandler)
        for n in range(NWORKERS):
            t = Thread(target=serv.serve_forever)
            t.daemon = True
            t.start()
        serv.serve_forever()

一般来讲，一个 ``TCPServer`` 在实例化的时候会绑定并激活相应的 ``socket`` 。
不过，有时候你想通过设置某些选项去调整底下的 `socket`` ，可以设置参数 ``bind_and_activate=False`` 。如下：

.. code-block:: python

    if __name__ == '__main__':
        serv = TCPServer(('', 20000), EchoHandler, bind_and_activate=False)
        # Set up various socket options
        serv.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
        # Bind and activate
        serv.server_bind()
        serv.server_activate()
        serv.serve_forever()

上面的 ``socket`` 选项是一个非常普遍的配置项，它允许服务器重新绑定一个之前使用过的端口号。
由于要被经常使用到，它被放置到类变量中，可以直接在 ``TCPServer`` 上面设置。
在实例化服务器的时候去设置它的值，如下所示：

.. code-block:: python

    if __name__ == '__main__':
        TCPServer.allow_reuse_address = True
        serv = TCPServer(('', 20000), EchoHandler)
        serv.serve_forever()

在上面示例中，我们演示了两种不同的处理器基类（ ``BaseRequestHandler`` 和 ``StreamRequestHandler`` ）。
``StreamRequestHandler`` 更加灵活点，能通过设置其他的类变量来支持一些新的特性。比如：

.. code-block:: python

    import socket

    class EchoHandler(StreamRequestHandler):
        # Optional settings (defaults shown)
        timeout = 5                      # Timeout on all socket operations
        rbufsize = -1                    # Read buffer size
        wbufsize = 0                     # Write buffer size
        disable_nagle_algorithm = False  # Sets TCP_NODELAY socket option
        def handle(self):
            print('Got connection from', self.client_address)
            try:
                for line in self.rfile:
                    # self.wfile is a file-like object for writing
                    self.wfile.write(line)
            except socket.timeout:
                print('Timed out!')

最后，还需要注意的是巨大部分Python的高层网络模块（比如HTTP、XML-RPC等）都是建立在 ``socketserver`` 功能之上。
也就是说，直接使用 ``socket`` 库来实现服务器也并不是很难。
下面是一个使用 ``socket`` 直接编程实现的一个服务器简单例子：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM

    def echo_handler(address, client_sock):
        print('Got connection from {}'.format(address))
        while True:
            msg = client_sock.recv(8192)
            if not msg:
                break
            client_sock.sendall(msg)
        client_sock.close()

    def echo_server(address, backlog=5):
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(address)
        sock.listen(backlog)
        while True:
            client_sock, client_addr = sock.accept()
            echo_handler(client_addr, client_sock)

    if __name__ == '__main__':
        echo_server(('', 20000))

============================
11.3 创建UDP服务器
============================

----------
问题
----------
你想实现一个基于UDP协议的服务器来与客户端通信。

----------
解决方案
----------
跟TCP一样，UDP服务器也可以通过使用 ``socketserver`` 库很容易的被创建。
例如，下面是一个简单的时间服务器：

.. code-block:: python

    from socketserver import BaseRequestHandler, UDPServer
    import time

    class TimeHandler(BaseRequestHandler):
        def handle(self):
            print('Got connection from', self.client_address)
            # Get message and client socket
            msg, sock = self.request
            resp = time.ctime()
            sock.sendto(resp.encode('ascii'), self.client_address)

    if __name__ == '__main__':
        serv = UDPServer(('', 20000), TimeHandler)
        serv.serve_forever()

跟之前一样，你先定义一个实现 ``handle()`` 特殊方法的类，为客户端连接服务。
这个类的 ``request`` 属性是一个包含了数据报和底层socket对象的元组。``client_address`` 包含了客户端地址。

我们来测试下这个服务器，首先运行它，然后打开另外一个Python进程向服务器发送消息：

.. code-block:: python

    >>> from socket import socket, AF_INET, SOCK_DGRAM
    >>> s = socket(AF_INET, SOCK_DGRAM)
    >>> s.sendto(b'', ('localhost', 20000))
    0
    >>> s.recvfrom(8192)
    (b'Wed Aug 15 20:35:08 2012', ('127.0.0.1', 20000))
    >>>

----------
讨论
----------
一个典型的UDP服务器接收到达的数据报(消息)和客户端地址。如果服务器需要做应答，
它要给客户端回发一个数据报。对于数据报的传送，
你应该使用socket的 ``sendto()`` 和 ``recvfrom()`` 方法。
尽管传统的 ``send()`` 和 ``recv()`` 也可以达到同样的效果，
但是前面的两个方法对于UDP连接而言更普遍。

由于没有底层的连接，UPD服务器相对于TCP服务器来讲实现起来更加简单。
不过，UDP天生是不可靠的（因为通信没有建立连接，消息可能丢失）。
因此需要由你自己来决定该怎样处理丢失消息的情况。这个已经不在本书讨论范围内了，
不过通常来说，如果可靠性对于你程序很重要，你需要借助于序列号、重试、超时以及一些其他方法来保证。
UDP通常被用在那些对于可靠传输要求不是很高的场合。例如，在实时应用如多媒体流以及游戏领域，
无需返回恢复丢失的数据包（程序只需简单的忽略它并继续向前运行）。

``UDPServer`` 类是单线程的，也就是说一次只能为一个客户端连接服务。
实际使用中，这个无论是对于UDP还是TCP都不是什么大问题。
如果你想要并发操作，可以实例化一个 ``ForkingUDPServer`` 或 ``ThreadingUDPServer`` 对象：

.. code-block:: python

    from socketserver import ThreadingUDPServer

       if __name__ == '__main__':
        serv = ThreadingUDPServer(('',20000), TimeHandler)
        serv.serve_forever()

直接使用 ``socket`` 来实现一个UDP服务器也不难，下面是一个例子：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_DGRAM
    import time

    def time_server(address):
        sock = socket(AF_INET, SOCK_DGRAM)
        sock.bind(address)
        while True:
            msg, addr = sock.recvfrom(8192)
            print('Got message from', addr)
            resp = time.ctime()
            sock.sendto(resp.encode('ascii'), addr)

    if __name__ == '__main__':
        time_server(('', 20000))

===============================
11.4 通过CIDR地址生成对应的IP地址集
===============================

----------
问题
----------
你有一个CIDR网络地址比如“123.45.67.89/27”，你想将其转换成它所代表的所有IP
（比如，“123.45.67.64”, “123.45.67.65”, …, “123.45.67.95”)）

----------
解决方案
----------
可以使用 ``ipaddress`` 模块很容易的实现这样的计算。例如：

.. code-block:: python

    >>> import ipaddress
    >>> net = ipaddress.ip_network('123.45.67.64/27')
    >>> net
    IPv4Network('123.45.67.64/27')
    >>> for a in net:
    ...     print(a)
    ...
    123.45.67.64
    123.45.67.65
    123.45.67.66
    123.45.67.67
    123.45.67.68
    ...
    123.45.67.95
    >>>

    >>> net6 = ipaddress.ip_network('12:3456:78:90ab:cd:ef01:23:30/125')
    >>> net6
    IPv6Network('12:3456:78:90ab:cd:ef01:23:30/125')
    >>> for a in net6:
    ...     print(a)
    ...
    12:3456:78:90ab:cd:ef01:23:30
    12:3456:78:90ab:cd:ef01:23:31
    12:3456:78:90ab:cd:ef01:23:32
    12:3456:78:90ab:cd:ef01:23:33
    12:3456:78:90ab:cd:ef01:23:34
    12:3456:78:90ab:cd:ef01:23:35
    12:3456:78:90ab:cd:ef01:23:36
    12:3456:78:90ab:cd:ef01:23:37
    >>>

``Network`` 也允许像数组一样的索引取值，例如：

.. code-block:: python

    >>> net.num_addresses
    32
    >>> net[0]

    IPv4Address('123.45.67.64')
    >>> net[1]
    IPv4Address('123.45.67.65')
    >>> net[-1]
    IPv4Address('123.45.67.95')
    >>> net[-2]
    IPv4Address('123.45.67.94')
    >>>

另外，你还可以执行网络成员检查之类的操作：

.. code-block:: python

    >>> a = ipaddress.ip_address('123.45.67.69')
    >>> a in net
    True
    >>> b = ipaddress.ip_address('123.45.67.123')
    >>> b in net
    False
    >>>

一个IP地址和网络地址能通过一个IP接口来指定，例如：

.. code-block:: python

    >>> inet = ipaddress.ip_interface('123.45.67.73/27')
    >>> inet.network
    IPv4Network('123.45.67.64/27')
    >>> inet.ip
    IPv4Address('123.45.67.73')
    >>>

----------
讨论
----------
``ipaddress`` 模块有很多类可以表示IP地址、网络和接口。
当你需要操作网络地址（比如解析、打印、验证等）的时候会很有用。

要注意的是，``ipaddress`` 模块跟其他一些和网络相关的模块比如 ``socket`` 库交集很少。
所以，你不能使用 ``IPv4Address`` 的实例来代替一个地址字符串，你首先得显式的使用 ``str()`` 转换它。例如：

.. code-block:: python

    >>> a = ipaddress.ip_address('127.0.0.1')
    >>> from socket import socket, AF_INET, SOCK_STREAM
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s.connect((a, 8080))
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: Can't convert 'IPv4Address' object to str implicitly
    >>> s.connect((str(a), 8080))
    >>>

更多相关内容，请参考 `An Introduction to the ipaddress Module <https://docs.python.org/3/howto/ipaddress.html>`_
===============================
11.5 创建一个简单的REST接口
===============================

----------
问题
----------
你想使用一个简单的REST接口通过网络远程控制或访问你的应用程序，但是你又不想自己去安装一个完整的web框架。

----------
解决方案
----------
构建一个REST风格的接口最简单的方法是创建一个基于WSGI标准（PEP 3333）的很小的库，下面是一个例子：

.. code-block:: python

    # resty.py

    import cgi

    def notfound_404(environ, start_response):
        start_response('404 Not Found', [ ('Content-type', 'text/plain') ])
        return [b'Not Found']

    class PathDispatcher:
        def __init__(self):
            self.pathmap = { }

        def __call__(self, environ, start_response):
            path = environ['PATH_INFO']
            params = cgi.FieldStorage(environ['wsgi.input'],
                                      environ=environ)
            method = environ['REQUEST_METHOD'].lower()
            environ['params'] = { key: params.getvalue(key) for key in params }
            handler = self.pathmap.get((method,path), notfound_404)
            return handler(environ, start_response)

        def register(self, method, path, function):
            self.pathmap[method.lower(), path] = function
            return function

为了使用这个调度器，你只需要编写不同的处理器，就像下面这样：

.. code-block:: python

    import time

    _hello_resp = '''\
    <html>
      <head>
         <title>Hello {name}</title>
       </head>
       <body>
         <h1>Hello {name}!</h1>
       </body>
    </html>'''

    def hello_world(environ, start_response):
        start_response('200 OK', [ ('Content-type','text/html')])
        params = environ['params']
        resp = _hello_resp.format(name=params.get('name'))
        yield resp.encode('utf-8')

    _localtime_resp = '''\
    <?xml version="1.0"?>
    <time>
      <year>{t.tm_year}</year>
      <month>{t.tm_mon}</month>
      <day>{t.tm_mday}</day>
      <hour>{t.tm_hour}</hour>
      <minute>{t.tm_min}</minute>
      <second>{t.tm_sec}</second>
    </time>'''

    def localtime(environ, start_response):
        start_response('200 OK', [ ('Content-type', 'application/xml') ])
        resp = _localtime_resp.format(t=time.localtime())
        yield resp.encode('utf-8')

    if __name__ == '__main__':
        from resty import PathDispatcher
        from wsgiref.simple_server import make_server

        # Create the dispatcher and register functions
        dispatcher = PathDispatcher()
        dispatcher.register('GET', '/hello', hello_world)
        dispatcher.register('GET', '/localtime', localtime)

        # Launch a basic server
        httpd = make_server('', 8080, dispatcher)
        print('Serving on port 8080...')
        httpd.serve_forever()

要测试下这个服务器，你可以使用一个浏览器或 ``urllib`` 和它交互。例如：

.. code-block:: python

    >>> u = urlopen('http://localhost:8080/hello?name=Guido')
    >>> print(u.read().decode('utf-8'))
    <html>
      <head>
         <title>Hello Guido</title>
       </head>
       <body>
         <h1>Hello Guido!</h1>
       </body>
    </html>

    >>> u = urlopen('http://localhost:8080/localtime')
    >>> print(u.read().decode('utf-8'))
    <?xml version="1.0"?>
    <time>
      <year>2012</year>
      <month>11</month>
      <day>24</day>
      <hour>14</hour>
      <minute>49</minute>
      <second>17</second>
    </time>
    >>>

----------
讨论
----------
在编写REST接口时，通常都是服务于普通的HTTP请求。但是跟那些功能完整的网站相比，你通常只需要处理数据。
这些数据以各种标准格式编码，比如XML、JSON或CSV。
尽管程序看上去很简单，但是以这种方式提供的API对于很多应用程序来讲是非常有用的。

例如，长期运行的程序可能会使用一个REST API来实现监控或诊断。
大数据应用程序可以使用REST来构建一个数据查询或提取系统。
REST还能用来控制硬件设备比如机器人、传感器、工厂或灯泡。
更重要的是，REST API已经被大量客户端编程环境所支持，比如Javascript, Android, iOS等。
因此，利用这种接口可以让你开发出更加复杂的应用程序。

为了实现一个简单的REST接口，你只需让你的程序代码满足Python的WSGI标准即可。
WSGI被标准库支持，同时也被绝大部分第三方web框架支持。
因此，如果你的代码遵循这个标准，在后面的使用过程中就会更加的灵活！

在WSGI中，你可以像下面这样约定的方式以一个可调用对象形式来实现你的程序。

.. code-block:: python

    import cgi

    def wsgi_app(environ, start_response):
        pass

``environ`` 属性是一个字典，包含了从web服务器如Apache[参考Internet RFC 3875]提供的CGI接口中获取的值。
要将这些不同的值提取出来，你可以像这么这样写：

.. code-block:: python

    def wsgi_app(environ, start_response):
        method = environ['REQUEST_METHOD']
        path = environ['PATH_INFO']
        # Parse the query parameters
        params = cgi.FieldStorage(environ['wsgi.input'], environ=environ)

我们展示了一些常见的值。``environ['REQUEST_METHOD']`` 代表请求类型如GET、POST、HEAD等。
``environ['PATH_INFO']`` 表示被请求资源的路径。
调用 ``cgi.FieldStorage()`` 可以从请求中提取查询参数并将它们放入一个类字典对象中以便后面使用。

``start_response`` 参数是一个为了初始化一个请求对象而必须被调用的函数。
第一个参数是返回的HTTP状态值，第二个参数是一个(名,值)元组列表，用来构建返回的HTTP头。例如：

.. code-block:: python

    def wsgi_app(environ, start_response):
        pass
        start_response('200 OK', [('Content-type', 'text/plain')])

为了返回数据，一个WSGI程序必须返回一个字节字符串序列。可以像下面这样使用一个列表来完成：

.. code-block:: python

    def wsgi_app(environ, start_response):
        pass
        start_response('200 OK', [('Content-type', 'text/plain')])
        resp = []
        resp.append(b'Hello World\n')
        resp.append(b'Goodbye!\n')
        return resp

或者，你还可以使用 ``yield`` ：

.. code-block:: python

    def wsgi_app(environ, start_response):
        pass
        start_response('200 OK', [('Content-type', 'text/plain')])
        yield b'Hello World\n'
        yield b'Goodbye!\n'

这里要强调的一点是最后返回的必须是字节字符串。如果返回结果包含文本字符串，必须先将其编码成字节。
当然，并没有要求你返回的一定是文本，你可以很轻松的编写一个生成图片的程序。

尽管WSGI程序通常被定义成一个函数，不过你也可以使用类实例来实现，只要它实现了合适的 ``__call__()`` 方法。例如：

.. code-block:: python

    class WSGIApplication:
        def __init__(self):
            ...
        def __call__(self, environ, start_response)
           ...

我们已经在上面使用这种技术创建 ``PathDispatcher`` 类。
这个分发器仅仅只是管理一个字典，将(方法,路径)对映射到处理器函数上面。
当一个请求到来时，它的方法和路径被提取出来，然后被分发到对应的处理器上面去。
另外，任何查询变量会被解析后放到一个字典中，以 ``environ['params']`` 形式存储。
后面这个步骤太常见，所以建议你在分发器里面完成，这样可以省掉很多重复代码。
使用分发器的时候，你只需简单的创建一个实例，然后通过它注册各种WSGI形式的函数。
编写这些函数应该超级简单了，只要你遵循 ``start_response()`` 函数的编写规则，并且最后返回字节字符串即可。

当编写这种函数的时候还需注意的一点就是对于字符串模板的使用。
没人愿意写那种到处混合着 ``print()`` 函数 、XML和大量格式化操作的代码。
我们上面使用了三引号包含的预先定义好的字符串模板。
这种方式的可以让我们很容易的在以后修改输出格式(只需要修改模板本身，而不用动任何使用它的地方)。

最后，使用WSGI还有一个很重要的部分就是没有什么地方是针对特定web服务器的。
因为标准对于服务器和框架是中立的，你可以将你的程序放入任何类型服务器中。
我们使用下面的代码测试测试本节代码：

.. code-block:: python

    if __name__ == '__main__':
        from wsgiref.simple_server import make_server

        # Create the dispatcher and register functions
        dispatcher = PathDispatcher()
        pass

        # Launch a basic server
        httpd = make_server('', 8080, dispatcher)
        print('Serving on port 8080...')
        httpd.serve_forever()

上面代码创建了一个简单的服务器，然后你就可以来测试下你的实现是否能正常工作。
最后，当你准备进一步扩展你的程序的时候，你可以修改这个代码，让它可以为特定服务器工作。

WSGI本身是一个很小的标准。因此它并没有提供一些高级的特性比如认证、cookies、重定向等。
这些你自己实现起来也不难。不过如果你想要更多的支持，可以考虑第三方库，比如 ``WebOb`` 或者 ``Paste``
===============================
11.6 通过XML-RPC实现简单的远程调用
===============================

----------
问题
----------
你想找到一个简单的方式去执行运行在远程机器上面的Python程序中的函数或方法。

----------
解决方案
----------
实现一个远程方法调用的最简单方式是使用XML-RPC。下面我们演示一下一个实现了键-值存储功能的简单服务器：

.. code-block:: python

    from xmlrpc.server import SimpleXMLRPCServer

    class KeyValueServer:
        _rpc_methods_ = ['get', 'set', 'delete', 'exists', 'keys']
        def __init__(self, address):
            self._data = {}
            self._serv = SimpleXMLRPCServer(address, allow_none=True)
            for name in self._rpc_methods_:
                self._serv.register_function(getattr(self, name))

        def get(self, name):
            return self._data[name]

        def set(self, name, value):
            self._data[name] = value

        def delete(self, name):
            del self._data[name]

        def exists(self, name):
            return name in self._data

        def keys(self):
            return list(self._data)

        def serve_forever(self):
            self._serv.serve_forever()

    # Example
    if __name__ == '__main__':
        kvserv = KeyValueServer(('', 15000))
        kvserv.serve_forever()

下面我们从一个客户端机器上面来访问服务器：

.. code-block:: python

    >>> from xmlrpc.client import ServerProxy
    >>> s = ServerProxy('http://localhost:15000', allow_none=True)
    >>> s.set('foo', 'bar')
    >>> s.set('spam', [1, 2, 3])
    >>> s.keys()
    ['spam', 'foo']
    >>> s.get('foo')
    'bar'
    >>> s.get('spam')
    [1, 2, 3]
    >>> s.delete('spam')
    >>> s.exists('spam')
    False
    >>>

----------
讨论
----------
XML-RPC 可以让我们很容易的构造一个简单的远程调用服务。你所需要做的仅仅是创建一个服务器实例，
通过它的方法 ``register_function()`` 来注册函数，然后使用方法 ``serve_forever()`` 启动它。
在上面我们将这些步骤放在一起写到一个类中，不够这并不是必须的。比如你还可以像下面这样创建一个服务器：

.. code-block:: python

    from xmlrpc.server import SimpleXMLRPCServer
    def add(x,y):
        return x+y

    serv = SimpleXMLRPCServer(('', 15000))
    serv.register_function(add)
    serv.serve_forever()

XML-RPC暴露出来的函数只能适用于部分数据类型，比如字符串、整形、列表和字典。
对于其他类型就得需要做些额外的功课了。
例如，如果你想通过 XML-RPC 传递一个对象实例，实际上只有他的实例字典被处理：

.. code-block:: python

    >>> class Point:
    ...     def __init__(self, x, y):
    ...             self.x = x
    ...             self.y = y
    ...
    >>> p = Point(2, 3)
    >>> s.set('foo', p)
    >>> s.get('foo')
    {'x': 2, 'y': 3}
    >>>

类似的，对于二进制数据的处理也跟你想象的不太一样：

.. code-block:: python

    >>> s.set('foo', b'Hello World')
    >>> s.get('foo')
    <xmlrpc.client.Binary object at 0x10131d410>

    >>> _.data
    b'Hello World'
    >>>

一般来讲，你不应该将 XML-RPC 服务以公共API的方式暴露出来。
对于这种情况，通常分布式应用程序会是一个更好的选择。

XML-RPC的一个缺点是它的性能。``SimpleXMLRPCServer`` 的实现是单线程的，
所以它不适合于大型程序，尽管我们在11.2小节中演示过它是可以通过多线程来执行的。
另外，由于 XML-RPC 将所有数据都序列化为XML格式，所以它会比其他的方式运行的慢一些。
但是它也有优点，这种方式的编码可以被绝大部分其他编程语言支持。
通过使用这种方式，其他语言的客户端程序都能访问你的服务。

虽然XML-RPC有很多缺点，但是如果你需要快速构建一个简单远程过程调用系统的话，它仍然值得去学习的。
有时候，简单的方案就已经足够了。
===============================
11.7 在不同的Python解释器之间交互
===============================

----------
问题
----------
你在不同的机器上面运行着多个Python解释器实例，并希望能够在这些解释器之间通过消息来交换数据。

----------
解决方案
----------
通过使用 ``multiprocessing.connection`` 模块可以很容易的实现解释器之间的通信。
下面是一个简单的应答服务器例子：

.. code-block:: python

    from multiprocessing.connection import Listener
    import traceback

    def echo_client(conn):
        try:
            while True:
                msg = conn.recv()
                conn.send(msg)
        except EOFError:
            print('Connection closed')

    def echo_server(address, authkey):
        serv = Listener(address, authkey=authkey)
        while True:
            try:
                client = serv.accept()

                echo_client(client)
            except Exception:
                traceback.print_exc()

    echo_server(('', 25000), authkey=b'peekaboo')

然后客户端连接服务器并发送消息的简单示例：

.. code-block:: python

    >>> from multiprocessing.connection import Client
    >>> c = Client(('localhost', 25000), authkey=b'peekaboo')
    >>> c.send('hello')
    >>> c.recv()
    'hello'
    >>> c.send(42)
    >>> c.recv()
    42
    >>> c.send([1, 2, 3, 4, 5])
    >>> c.recv()
    [1, 2, 3, 4, 5]
    >>>

跟底层socket不同的是，每个消息会完整保存（每一个通过send()发送的对象能通过recv()来完整接受）。
另外，所有对象会通过pickle序列化。因此，任何兼容pickle的对象都能在此连接上面被发送和接受。

----------
讨论
----------
目前有很多用来实现各种消息传输的包和函数库，比如ZeroMQ、Celery等。
你还有另外一种选择就是自己在底层socket基础之上来实现一个消息传输层。
但是你想要简单一点的方案，那么这时候 ``multiprocessing.connection`` 就派上用场了。
仅仅使用一些简单的语句即可实现多个解释器之间的消息通信。

如果你的解释器运行在同一台机器上面，那么你可以使用另外的通信机制，比如Unix域套接字或者是Windows命名管道。
要想使用UNIX域套接字来创建一个连接，只需简单的将地址改写一个文件名即可：

.. code-block:: python

    s = Listener('/tmp/myconn', authkey=b'peekaboo')

要想使用Windows命名管道来创建连接，只需像下面这样使用一个文件名：

.. code-block:: python

    s = Listener(r'\\.\pipe\myconn', authkey=b'peekaboo')

一个通用准则是，你不要使用 ``multiprocessing`` 来实现一个对外的公共服务。
``Client()`` 和 ``Listener()`` 中的 ``authkey`` 参数用来认证发起连接的终端用户。
如果密钥不对会产生一个异常。此外，该模块最适合用来建立长连接（而不是大量的短连接），
例如，两个解释器之间启动后就开始建立连接并在处理某个问题过程中会一直保持连接状态。

如果你需要对底层连接做更多的控制，比如需要支持超时、非阻塞I/O或其他类似的特性，
你最好使用另外的库或者是在高层socket上来实现这些特性。
===============================
11.8 实现远程方法调用
===============================

----------
问题
----------
你想在一个消息传输层如 ``sockets`` 、``multiprocessing connections`` 或 ``ZeroMQ``
的基础之上实现一个简单的远程过程调用（RPC）。

----------
解决方案
----------

将函数请求、参数和返回值使用pickle编码后，在不同的解释器直接传送pickle字节字符串，可以很容易的实现RPC。
下面是一个简单的PRC处理器，可以被整合到一个服务器中去：

.. code-block:: python

    # rpcserver.py

    import pickle
    class RPCHandler:
        def __init__(self):
            self._functions = { }

        def register_function(self, func):
            self._functions[func.__name__] = func

        def handle_connection(self, connection):
            try:
                while True:
                    # Receive a message
                    func_name, args, kwargs = pickle.loads(connection.recv())
                    # Run the RPC and send a response
                    try:
                        r = self._functions[func_name](*args,**kwargs)
                        connection.send(pickle.dumps(r))
                    except Exception as e:
                        connection.send(pickle.dumps(e))
            except EOFError:
                 pass

要使用这个处理器，你需要将它加入到一个消息服务器中。你有很多种选择，
但是使用 ``multiprocessing`` 库是最简单的。下面是一个RPC服务器例子：

.. code-block:: python

    from multiprocessing.connection import Listener
    from threading import Thread

    def rpc_server(handler, address, authkey):
        sock = Listener(address, authkey=authkey)
        while True:
            client = sock.accept()
            t = Thread(target=handler.handle_connection, args=(client,))
            t.daemon = True
            t.start()

    # Some remote functions
    def add(x, y):
        return x + y

    def sub(x, y):
        return x - y

    # Register with a handler
    handler = RPCHandler()
    handler.register_function(add)
    handler.register_function(sub)

    # Run the server
    rpc_server(handler, ('localhost', 17000), authkey=b'peekaboo')

为了从一个远程客户端访问服务器，你需要创建一个对应的用来传送请求的RPC代理类。例如

.. code-block:: python

    import pickle

    class RPCProxy:
        def __init__(self, connection):
            self._connection = connection
        def __getattr__(self, name):
            def do_rpc(*args, **kwargs):
                self._connection.send(pickle.dumps((name, args, kwargs)))
                result = pickle.loads(self._connection.recv())
                if isinstance(result, Exception):
                    raise result
                return result
            return do_rpc

要使用这个代理类，你需要将其包装到一个服务器的连接上面，例如：

.. code-block:: python

    >>> from multiprocessing.connection import Client
    >>> c = Client(('localhost', 17000), authkey=b'peekaboo')
    >>> proxy = RPCProxy(c)
    >>> proxy.add(2, 3)

    5
    >>> proxy.sub(2, 3)
    -1
    >>> proxy.sub([1, 2], 4)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "rpcserver.py", line 37, in do_rpc
        raise result
    TypeError: unsupported operand type(s) for -: 'list' and 'int'
    >>>

要注意的是很多消息层（比如 ``multiprocessing`` ）已经使用pickle序列化了数据。
如果是这样的话，对 ``pickle.dumps()`` 和 ``pickle.loads()`` 的调用要去掉。

----------
讨论
----------
``RPCHandler`` 和 ``RPCProxy`` 的基本思路是很比较简单的。
如果一个客户端想要调用一个远程函数，比如 ``foo(1, 2, z=3)``
,代理类创建一个包含了函数名和参数的元组 ``('foo', (1, 2), {'z': 3})`` 。
这个元组被pickle序列化后通过网络连接发生出去。
这一步在 ``RPCProxy`` 的 ``__getattr__()`` 方法返回的 ``do_rpc()`` 闭包中完成。
服务器接收后通过pickle反序列化消息，查找函数名看看是否已经注册过，然后执行相应的函数。
执行结果(或异常)被pickle序列化后返回发送给客户端。我们的实例需要依赖 ``multiprocessing`` 进行通信。
不过，这种方式可以适用于其他任何消息系统。例如，如果你想在ZeroMQ之上实习RPC，
仅仅只需要将连接对象换成合适的ZeroMQ的socket对象即可。

由于底层需要依赖pickle，那么安全问题就需要考虑了
（因为一个聪明的黑客可以创建特定的消息，能够让任意函数通过pickle反序列化后被执行）。
因此你永远不要允许来自不信任或未认证的客户端的RPC。特别是你绝对不要允许来自Internet的任意机器的访问，
这种只能在内部被使用，位于防火墙后面并且不要对外暴露。

作为pickle的替代，你也许可以考虑使用JSON、XML或一些其他的编码格式来序列化消息。
例如，本机实例可以很容易的改写成JSON编码方案。还需要将 ``pickle.loads()`` 和  ``pickle.dumps()``
替换成 ``json.loads()`` 和 ``json.dumps()`` 即可：

.. code-block:: python

    # jsonrpcserver.py
    import json

    class RPCHandler:
        def __init__(self):
            self._functions = { }

        def register_function(self, func):
            self._functions[func.__name__] = func

        def handle_connection(self, connection):
            try:
                while True:
                    # Receive a message
                    func_name, args, kwargs = json.loads(connection.recv())
                    # Run the RPC and send a response
                    try:
                        r = self._functions[func_name](*args,**kwargs)
                        connection.send(json.dumps(r))
                    except Exception as e:
                        connection.send(json.dumps(str(e)))
            except EOFError:
                 pass

    # jsonrpcclient.py
    import json

    class RPCProxy:
        def __init__(self, connection):
            self._connection = connection
        def __getattr__(self, name):
            def do_rpc(*args, **kwargs):
                self._connection.send(json.dumps((name, args, kwargs)))
                result = json.loads(self._connection.recv())
                return result
            return do_rpc

实现RPC的一个比较复杂的问题是如何去处理异常。至少，当方法产生异常时服务器不应该奔溃。
因此，返回给客户端的异常所代表的含义就要好好设计了。
如果你使用pickle，异常对象实例在客户端能被反序列化并抛出。如果你使用其他的协议，那得想想另外的方法了。
不过至少，你应该在响应中返回异常字符串。我们在JSON的例子中就是使用的这种方式。

对于其他的RPC实现例子，我推荐你看看在XML-RPC中使用的 ``SimpleXMLRPCServer`` 和 ``ServerProxy`` 的实现，
也就是11.6小节中的内容。
===============================
11.9 简单的客户端认证
===============================

----------
问题
----------
你想在分布式系统中实现一个简单的客户端连接认证功能，又不想像SSL那样的复杂。

----------
解决方案
----------
可以利用 ``hmac`` 模块实现一个连接握手，从而实现一个简单而高效的认证过程。下面是代码示例：

.. code-block:: python

    import hmac
    import os

    def client_authenticate(connection, secret_key):
        '''
        Authenticate client to a remote service.
        connection represents a network connection.
        secret_key is a key known only to both client/server.
        '''
        message = connection.recv(32)
        hash = hmac.new(secret_key, message)
        digest = hash.digest()
        connection.send(digest)

    def server_authenticate(connection, secret_key):
        '''
        Request client authentication.
        '''
        message = os.urandom(32)
        connection.send(message)
        hash = hmac.new(secret_key, message)
        digest = hash.digest()
        response = connection.recv(len(digest))
        return hmac.compare_digest(digest,response)

基本原理是当连接建立后，服务器给客户端发送一个随机的字节消息（这里例子中使用了 ``os.urandom()`` 返回值）。
客户端和服务器同时利用hmac和一个只有双方知道的密钥来计算出一个加密哈希值。然后客户端将它计算出的摘要发送给服务器，
服务器通过比较这个值和自己计算的是否一致来决定接受或拒绝连接。摘要的比较需要使用 ``hmac.compare_digest()`` 函数。
使用这个函数可以避免遭到时间分析攻击，不要用简单的比较操作符（==）。
为了使用这些函数，你需要将它集成到已有的网络或消息代码中。例如，对于sockets，服务器代码应该类似下面：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM

    secret_key = b'peekaboo'
    def echo_handler(client_sock):
        if not server_authenticate(client_sock, secret_key):
            client_sock.close()
            return
        while True:

            msg = client_sock.recv(8192)
            if not msg:
                break
            client_sock.sendall(msg)

    def echo_server(address):
        s = socket(AF_INET, SOCK_STREAM)
        s.bind(address)
        s.listen(5)
        while True:
            c,a = s.accept()
            echo_handler(c)

    echo_server(('', 18000))

    Within a client, you would do this:

    from socket import socket, AF_INET, SOCK_STREAM

    secret_key = b'peekaboo'

    s = socket(AF_INET, SOCK_STREAM)
    s.connect(('localhost', 18000))
    client_authenticate(s, secret_key)
    s.send(b'Hello World')
    resp = s.recv(1024)

----------
讨论
----------
``hmac`` 认证的一个常见使用场景是内部消息通信系统和进程间通信。
例如，如果你编写的系统涉及到一个集群中多个处理器之间的通信，
你可以使用本节方案来确保只有被允许的进程之间才能彼此通信。
事实上，基于 ``hmac`` 的认证被 ``multiprocessing`` 模块使用来实现子进程直接的通信。

还有一点需要强调的是连接认证和加密是两码事。
认证成功之后的通信消息是以明文形式发送的，任何人只要想监听这个连接线路都能看到消息（尽管双方的密钥不会被传输）。

hmac认证算法基于哈希函数如MD5和SHA-1，关于这个在IETF RFC 2104中有详细介绍。
===============================
11.10 在网络服务中加入SSL
===============================

----------
问题
----------
你想实现一个基于sockets的网络服务，客户端和服务器通过SSL协议认证并加密传输的数据。

----------
解决方案
----------
``ssl`` 模块能为底层socket连接添加SSL的支持。
``ssl.wrap_socket()`` 函数接受一个已存在的socket作为参数并使用SSL层来包装它。
例如，下面是一个简单的应答服务器，能在服务器端为所有客户端连接做认证。

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM
    import ssl

    KEYFILE = 'server_key.pem'   # Private key of the server
    CERTFILE = 'server_cert.pem' # Server certificate (given to client)

    def echo_client(s):
        while True:
            data = s.recv(8192)
            if data == b'':
                break
            s.send(data)
        s.close()
        print('Connection closed')

    def echo_server(address):
        s = socket(AF_INET, SOCK_STREAM)
        s.bind(address)
        s.listen(1)

        # Wrap with an SSL layer requiring client certs
        s_ssl = ssl.wrap_socket(s,
                                keyfile=KEYFILE,
                                certfile=CERTFILE,
                                server_side=True
                                )
        # Wait for connections
        while True:
            try:
                c,a = s_ssl.accept()
                print('Got connection', c, a)
                echo_client(c)
            except Exception as e:
                print('{}: {}'.format(e.__class__.__name__, e))

    echo_server(('', 20000))

下面我们演示一个客户端连接服务器的交互例子。客户端会请求服务器来认证并确认连接：

.. code-block:: python

    >>> from socket import socket, AF_INET, SOCK_STREAM
    >>> import ssl
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s_ssl = ssl.wrap_socket(s,
                    cert_reqs=ssl.CERT_REQUIRED,
                    ca_certs = 'server_cert.pem')
    >>> s_ssl.connect(('localhost', 20000))
    >>> s_ssl.send(b'Hello World?')
    12
    >>> s_ssl.recv(8192)
    b'Hello World?'
    >>>

这种直接处理底层socket方式有个问题就是它不能很好的跟标准库中已存在的网络服务兼容。
例如，绝大部分服务器代码（HTTP、XML-RPC等）实际上是基于 ``socketserver`` 库的。
客户端代码在一个较高层上实现。我们需要另外一种稍微不同的方式来将SSL添加到已存在的服务中：

首先，对于服务器而言，可以通过像下面这样使用一个mixin类来添加SSL：

.. code-block:: python

        import ssl

        class SSLMixin:
        '''
        Mixin class that adds support for SSL to existing servers based
        on the socketserver module.
        '''
        def __init__(self, *args,
                     keyfile=None, certfile=None, ca_certs=None,
                     cert_reqs=ssl.CERT_NONE,
                     **kwargs):
            self._keyfile = keyfile
            self._certfile = certfile
            self._ca_certs = ca_certs
            self._cert_reqs = cert_reqs
            super().__init__(*args, **kwargs)

        def get_request(self):
            client, addr = super().get_request()
            client_ssl = ssl.wrap_socket(client,
                                         keyfile = self._keyfile,
                                         certfile = self._certfile,
                                         ca_certs = self._ca_certs,
                                         cert_reqs = self._cert_reqs,
                                         server_side = True)
            return client_ssl, addr

为了使用这个mixin类，你可以将它跟其他服务器类混合。例如，下面是定义一个基于SSL的XML-RPC服务器例子：

.. code-block:: python

    # XML-RPC server with SSL

    from xmlrpc.server import SimpleXMLRPCServer

    class SSLSimpleXMLRPCServer(SSLMixin, SimpleXMLRPCServer):
        pass

    Here's the XML-RPC server from Recipe 11.6 modified only slightly to use SSL:

    import ssl
    from xmlrpc.server import SimpleXMLRPCServer
    from sslmixin import SSLMixin

    class SSLSimpleXMLRPCServer(SSLMixin, SimpleXMLRPCServer):
        pass

    class KeyValueServer:
        _rpc_methods_ = ['get', 'set', 'delete', 'exists', 'keys']
        def __init__(self, *args, **kwargs):
            self._data = {}
            self._serv = SSLSimpleXMLRPCServer(*args, allow_none=True, **kwargs)
            for name in self._rpc_methods_:
                self._serv.register_function(getattr(self, name))

        def get(self, name):
            return self._data[name]

        def set(self, name, value):
            self._data[name] = value

        def delete(self, name):
            del self._data[name]

        def exists(self, name):
            return name in self._data

        def keys(self):
            return list(self._data)

        def serve_forever(self):
            self._serv.serve_forever()

    if __name__ == '__main__':
        KEYFILE='server_key.pem'    # Private key of the server
        CERTFILE='server_cert.pem'  # Server certificate
        kvserv = KeyValueServer(('', 15000),
                                keyfile=KEYFILE,
                                certfile=CERTFILE)
        kvserv.serve_forever()

使用这个服务器时，你可以使用普通的 ``xmlrpc.client`` 模块来连接它。
只需要在URL中指定 ``https:`` 即可，例如：

.. code-block:: python

    >>> from xmlrpc.client import ServerProxy
    >>> s = ServerProxy('https://localhost:15000', allow_none=True)
    >>> s.set('foo','bar')
    >>> s.set('spam', [1, 2, 3])
    >>> s.keys()
    ['spam', 'foo']
    >>> s.get('foo')
    'bar'
    >>> s.get('spam')
    [1, 2, 3]
    >>> s.delete('spam')
    >>> s.exists('spam')
    False
    >>>

对于SSL客户端来讲一个比较复杂的问题是如何确认服务器证书或为服务器提供客户端认证（比如客户端证书）。
不幸的是，暂时还没有一个标准方法来解决这个问题，需要自己去研究。
不过，下面给出一个例子，用来建立一个安全的XML-RPC连接来确认服务器证书：

.. code-block:: python

    from xmlrpc.client import SafeTransport, ServerProxy
    import ssl

    class VerifyCertSafeTransport(SafeTransport):
        def __init__(self, cafile, certfile=None, keyfile=None):
            SafeTransport.__init__(self)
            self._ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLSv1)
            self._ssl_context.load_verify_locations(cafile)
            if certfile:
                self._ssl_context.load_cert_chain(certfile, keyfile)
            self._ssl_context.verify_mode = ssl.CERT_REQUIRED

        def make_connection(self, host):
            # Items in the passed dictionary are passed as keyword
            # arguments to the http.client.HTTPSConnection() constructor.
            # The context argument allows an ssl.SSLContext instance to
            # be passed with information about the SSL configuration
            s = super().make_connection((host, {'context': self._ssl_context}))

            return s

    # Create the client proxy
    s = ServerProxy('https://localhost:15000',
                    transport=VerifyCertSafeTransport('server_cert.pem'),
                    allow_none=True)

服务器将证书发送给客户端，客户端来确认它的合法性。这种确认可以是相互的。
如果服务器想要确认客户端，可以将服务器启动代码修改如下：

.. code-block:: python

    if __name__ == '__main__':
        KEYFILE='server_key.pem'   # Private key of the server
        CERTFILE='server_cert.pem' # Server certificate
        CA_CERTS='client_cert.pem' # Certificates of accepted clients

        kvserv = KeyValueServer(('', 15000),
                                keyfile=KEYFILE,
                                certfile=CERTFILE,
                                ca_certs=CA_CERTS,
                                cert_reqs=ssl.CERT_REQUIRED,
                                )
        kvserv.serve_forever()

为了让XML-RPC客户端发送证书，修改 ``ServerProxy`` 的初始化代码如下：

.. code-block:: python

    # Create the client proxy
    s = ServerProxy('https://localhost:15000',
                    transport=VerifyCertSafeTransport('server_cert.pem',
                                                      'client_cert.pem',
                                                      'client_key.pem'),
                    allow_none=True)

----------
讨论
----------
试着去运行本节的代码能测试你的系统配置能力和理解SSL。
可能最大的挑战是如何一步步的获取初始配置key、证书和其他所需依赖。

我解释下到底需要啥，每一个SSL连接终端一般都会有一个私钥和一个签名证书文件。
这个证书包含了公钥并在每一次连接的时候都会发送给对方。
对于公共服务器，它们的证书通常是被权威证书机构比如Verisign、Equifax或其他类似机构（需要付费的）签名过的。
为了确认服务器签名，客户端回保存一份包含了信任授权机构的证书列表文件。
例如，web浏览器保存了主要的认证机构的证书，并使用它来为每一个HTTPS连接确认证书的合法性。
对本小节示例而言，只是为了测试，我们可以创建自签名的证书，下面是主要步骤：


    bash % openssl req -new -x509 -days 365 -nodes -out server_cert.pem \
               -keyout server_key.pem
    Generating a 1024 bit RSA private key
    ..........................................++++++
    ...++++++

    writing new private key to 'server_key.pem'

     -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
     -----
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:Illinois
    Locality Name (eg, city) []:Chicago
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Dabeaz, LLC
    Organizational Unit Name (eg, section) []:
    Common Name (eg, YOUR name) []:localhost
    Email Address []:
    bash %

在创建证书的时候，各个值的设定可以是任意的，但是”Common Name“的值通常要包含服务器的DNS主机名。
如果你只是在本机测试，那么就使用”localhost“，否则使用服务器的域名。

    -----BEGIN RSA PRIVATE KEY-----
    MIICXQIBAAKBgQCZrCNLoEyAKF+f9UNcFaz5Osa6jf7qkbUl8si5xQrY3ZYC7juu
    nL1dZLn/VbEFIITaUOgvBtPv1qUWTJGwga62VSG1oFE0ODIx3g2Nh4sRf+rySsx2
    L4442nx0z4O5vJQ7k6eRNHAZUUnCL50+YvjyLyt7ryLSjSuKhCcJsbZgPwIDAQAB
    AoGAB5evrr7eyL4160tM5rHTeATlaLY3UBOe5Z8XN8Z6gLiB/ucSX9AysviVD/6F
    3oD6z2aL8jbeJc1vHqjt0dC2dwwm32vVl8mRdyoAsQpWmiqXrkvP4Bsl04VpBeHw
    Qt8xNSW9SFhceL3LEvw9M8i9MV39viih1ILyH8OuHdvJyFECQQDLEjl2d2ppxND9
    PoLqVFAirDfX2JnLTdWbc+M11a9Jdn3hKF8TcxfEnFVs5Gav1MusicY5KB0ylYPb
    YbTvqKc7AkEAwbnRBO2VYEZsJZp2X0IZqP9ovWokkpYx+PE4+c6MySDgaMcigL7v
    WDIHJG1CHudD09GbqENasDzyb2HAIW4CzQJBAKDdkv+xoW6gJx42Auc2WzTcUHCA
    eXR/+BLpPrhKykzbvOQ8YvS5W764SUO1u1LWs3G+wnRMvrRvlMCZKgggBjkCQQCG
    Jewto2+a+WkOKQXrNNScCDE5aPTmZQc5waCYq4UmCZQcOjkUOiN3ST1U5iuxRqfb
    V/yX6fw0qh+fLWtkOs/JAkA+okMSxZwqRtfgOFGBfwQ8/iKrnizeanTQ3L6scFXI
    CHZXdJ3XQ6qUmNxNn7iJ7S/LDawo1QfWkCfD9FYoxBlg
    -----END RSA PRIVATE KEY-----

服务器证书文件server_cert.pem内容类似下面这样：

    -----BEGIN CERTIFICATE-----
    MIIC+DCCAmGgAwIBAgIJAPMd+vi45js3MA0GCSqGSIb3DQEBBQUAMFwxCzAJBgNV
    BAYTAlVTMREwDwYDVQQIEwhJbGxpbm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIG
    A1UEChMLRGFiZWF6LCBMTEMxEjAQBgNVBAMTCWxvY2FsaG9zdDAeFw0xMzAxMTEx
    ODQyMjdaFw0xNDAxMTExODQyMjdaMFwxCzAJBgNVBAYTAlVTMREwDwYDVQQIEwhJ
    bGxpbm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIGA1UEChMLRGFiZWF6LCBMTEMx
    EjAQBgNVBAMTCWxvY2FsaG9zdDCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEA
    mawjS6BMgChfn/VDXBWs+TrGuo3+6pG1JfLIucUK2N2WAu47rpy9XWS5/1WxBSCE
    2lDoLwbT79alFkyRsIGutlUhtaBRNDgyMd4NjYeLEX/q8krMdi+OONp8dM+DubyU

    O5OnkTRwGVFJwi+dPmL48i8re68i0o0rioQnCbG2YD8CAwEAAaOBwTCBvjAdBgNV
    HQ4EFgQUrtoLHHgXiDZTr26NMmgKJLJLFtIwgY4GA1UdIwSBhjCBg4AUrtoLHHgX
    iDZTr26NMmgKJLJLFtKhYKReMFwxCzAJBgNVBAYTAlVTMREwDwYDVQQIEwhJbGxp
    bm9pczEQMA4GA1UEBxMHQ2hpY2FnbzEUMBIGA1UEChMLRGFiZWF6LCBMTEMxEjAQ
    BgNVBAMTCWxvY2FsaG9zdIIJAPMd+vi45js3MAwGA1UdEwQFMAMBAf8wDQYJKoZI
    hvcNAQEFBQADgYEAFci+dqvMG4xF8UTnbGVvZJPIzJDRee6Nbt6AHQo9pOdAIMAu
    WsGCplSOaDNdKKzl+b2UT2Zp3AIW4Qd51bouSNnR4M/gnr9ZD1ZctFd3jS+C5XRp
    D3vvcW5lAnCCC80P6rXy7d7hTeFu5EYKtRGXNvVNd/06NALGDflrrOwxF3Y=
    -----END CERTIFICATE-----

在服务器端代码中，私钥和证书文件会被传给SSL相关的包装函数。证书来自于客户端，
私钥应该在保存在服务器中，并加以安全保护。

在客户端代码中，需要保存一个合法证书授权文件来确认服务器证书。
如果你没有这个文件，你可以在客户端复制一份服务器的证书并使用它来确认。
连接建立后，服务器会提供它的证书，然后你就能使用已经保存的证书来确认它是否正确。

服务器也能选择是否要确认客户端的身份。如果要这样做的话，客户端需要有自己的私钥和认证文件。
服务器也需要保存一个被信任证书授权文件来确认客户端证书。

如果你要在真实环境中为你的网络服务加上SSL的支持，这小节只是一个入门介绍而已。
你还应该参考其他的文档，做好花费不少时间来测试它正常工作的准备。反正，就是得慢慢折腾吧~ ^_^

==============================
11.11 进程间传递Socket文件描述符
==============================

----------
问题
----------
你有多个Python解释器进程在同时运行，你想将某个打开的文件描述符从一个解释器传递给另外一个。
比如，假设有个服务器进程相应连接请求，但是实际的相应逻辑是在另一个解释器中执行的。

----------
解决方案
----------
为了在多个进程中传递文件描述符，你首先需要将它们连接到一起。在Unix机器上，你可能需要使用Unix域套接字，
而在windows上面你需要使用命名管道。不过你无需真的需要去操作这些底层，
通常使用 ``multiprocessing`` 模块来创建这样的连接会更容易一些。

一旦一个连接被创建，你可以使用 ``multiprocessing.reduction`` 中的
``send_handle()`` 和 ``recv_handle()`` 函数在不同的处理器直接传递文件描述符。
下面的例子演示了最基本的用法：

.. code-block:: python

    import multiprocessing
    from multiprocessing.reduction import recv_handle, send_handle
    import socket

    def worker(in_p, out_p):
        out_p.close()
        while True:
            fd = recv_handle(in_p)
            print('CHILD: GOT FD', fd)
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM, fileno=fd) as s:
                while True:
                    msg = s.recv(1024)
                    if not msg:
                        break
                    print('CHILD: RECV {!r}'.format(msg))
                    s.send(msg)

    def server(address, in_p, out_p, worker_pid):
        in_p.close()
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
        s.bind(address)
        s.listen(1)
        while True:
            client, addr = s.accept()
            print('SERVER: Got connection from', addr)
            send_handle(out_p, client.fileno(), worker_pid)
            client.close()

    if __name__ == '__main__':
        c1, c2 = multiprocessing.Pipe()
        worker_p = multiprocessing.Process(target=worker, args=(c1,c2))
        worker_p.start()

        server_p = multiprocessing.Process(target=server,
                      args=(('', 15000), c1, c2, worker_p.pid))
        server_p.start()

        c1.close()
        c2.close()

在这个例子中，两个进程被创建并通过一个 ``multiprocessing`` 管道连接起来。
服务器进程打开一个socket并等待客户端连接请求。
工作进程仅仅使用 ``recv_handle()`` 在管道上面等待接收一个文件描述符。
当服务器接收到一个连接，它将产生的socket文件描述符通过 ``send_handle()`` 传递给工作进程。
工作进程接收到socket后向客户端回应数据，然后此次连接关闭。

如果你使用Telnet或类似工具连接到服务器，下面是一个演示例子：

    bash % python3 passfd.py
    SERVER: Got connection from ('127.0.0.1', 55543)
    CHILD: GOT FD 7
    CHILD: RECV b'Hello\r\n'
    CHILD: RECV b'World\r\n'

此例最重要的部分是服务器接收到的客户端socket实际上被另外一个不同的进程处理。
服务器仅仅只是将其转手并关闭此连接，然后等待下一个连接。

----------
讨论
----------
对于大部分程序员来讲在不同进程之间传递文件描述符好像没什么必要。
但是，有时候它是构建一个可扩展系统的很有用的工具。例如，在一个多核机器上面，
你可以有多个Python解释器实例，将文件描述符传递给其它解释器来实现负载均衡。

``send_handle()`` 和 ``recv_handle()`` 函数只能够用于 ``multiprocessing`` 连接。
使用它们来代替管道的使用（参考11.7节），只要你使用的是Unix域套接字或Windows管道。
例如，你可以让服务器和工作者各自以单独的程序来启动。下面是服务器的实现例子：

.. code-block:: python

    # servermp.py
    from multiprocessing.connection import Listener
    from multiprocessing.reduction import send_handle
    import socket

    def server(work_address, port):
        # Wait for the worker to connect
        work_serv = Listener(work_address, authkey=b'peekaboo')
        worker = work_serv.accept()
        worker_pid = worker.recv()

        # Now run a TCP/IP server and send clients to worker
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
        s.bind(('', port))
        s.listen(1)
        while True:
            client, addr = s.accept()
            print('SERVER: Got connection from', addr)

            send_handle(worker, client.fileno(), worker_pid)
            client.close()

    if __name__ == '__main__':
        import sys
        if len(sys.argv) != 3:
            print('Usage: server.py server_address port', file=sys.stderr)
            raise SystemExit(1)

        server(sys.argv[1], int(sys.argv[2]))

运行这个服务器，只需要执行 `python3 servermp.py /tmp/servconn 15000` ，下面是相应的工作者代码：

.. code-block:: python

    # workermp.py

    from multiprocessing.connection import Client
    from multiprocessing.reduction import recv_handle
    import os
    from socket import socket, AF_INET, SOCK_STREAM

    def worker(server_address):
        serv = Client(server_address, authkey=b'peekaboo')
        serv.send(os.getpid())
        while True:
            fd = recv_handle(serv)
            print('WORKER: GOT FD', fd)
            with socket(AF_INET, SOCK_STREAM, fileno=fd) as client:
                while True:
                    msg = client.recv(1024)
                    if not msg:
                        break
                    print('WORKER: RECV {!r}'.format(msg))
                    client.send(msg)

    if __name__ == '__main__':
        import sys
        if len(sys.argv) != 2:
            print('Usage: worker.py server_address', file=sys.stderr)
            raise SystemExit(1)

        worker(sys.argv[1])

要运行工作者，执行执行命令 `python3 workermp.py /tmp/servconn` .
效果跟使用Pipe()例子是完全一样的。
文件描述符的传递会涉及到UNIX域套接字的创建和套接字的 ``sendmsg()`` 方法。
不过这种技术并不常见，下面是使用套接字来传递描述符的另外一种实现：

.. code-block:: python

    # server.py
    import socket

    import struct

    def send_fd(sock, fd):
        '''
        Send a single file descriptor.
        '''
        sock.sendmsg([b'x'],
                     [(socket.SOL_SOCKET, socket.SCM_RIGHTS, struct.pack('i', fd))])
        ack = sock.recv(2)
        assert ack == b'OK'

    def server(work_address, port):
        # Wait for the worker to connect
        work_serv = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
        work_serv.bind(work_address)
        work_serv.listen(1)
        worker, addr = work_serv.accept()

        # Now run a TCP/IP server and send clients to worker
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
        s.bind(('',port))
        s.listen(1)
        while True:
            client, addr = s.accept()
            print('SERVER: Got connection from', addr)
            send_fd(worker, client.fileno())
            client.close()

    if __name__ == '__main__':
        import sys
        if len(sys.argv) != 3:
            print('Usage: server.py server_address port', file=sys.stderr)
            raise SystemExit(1)

        server(sys.argv[1], int(sys.argv[2]))

下面是使用套接字的工作者实现：

.. code-block:: python

    # worker.py
    import socket
    import struct

    def recv_fd(sock):
        '''
        Receive a single file descriptor
        '''
        msg, ancdata, flags, addr = sock.recvmsg(1,
                                         socket.CMSG_LEN(struct.calcsize('i')))

        cmsg_level, cmsg_type, cmsg_data = ancdata[0]
        assert cmsg_level == socket.SOL_SOCKET and cmsg_type == socket.SCM_RIGHTS
        sock.sendall(b'OK')

        return struct.unpack('i', cmsg_data)[0]

    def worker(server_address):
        serv = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
        serv.connect(server_address)
        while True:
            fd = recv_fd(serv)
            print('WORKER: GOT FD', fd)
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM, fileno=fd) as client:
                while True:
                    msg = client.recv(1024)
                    if not msg:
                        break
                    print('WORKER: RECV {!r}'.format(msg))
                    client.send(msg)

    if __name__ == '__main__':
        import sys
        if len(sys.argv) != 2:
            print('Usage: worker.py server_address', file=sys.stderr)
            raise SystemExit(1)

        worker(sys.argv[1])

如果你想在你的程序中传递文件描述符，建议你参阅其他一些更加高级的文档，
比如 ``Unix Network Programming by W. Richard Stevens  (Prentice  Hall,  1990)`` .
在Windows上传递文件描述符跟Unix是不一样的，建议你研究下 ``multiprocessing.reduction`` 中的源代码看看其工作原理。

==============================
11.12 理解事件驱动的IO
==============================

----------
问题
----------
你应该已经听过基于事件驱动或异步I/O的包，但是你还不能完全理解它的底层到底是怎样工作的，
或者是如果使用它的话会对你的程序产生什么影响。

----------
解决方案
----------
事件驱动I/O本质上来讲就是将基本I/O操作（比如读和写）转化为你程序需要处理的事件。
例如，当数据在某个socket上被接受后，它会转换成一个 ``receive`` 事件，然后被你定义的回调方法或函数来处理。
作为一个可能的起始点，一个事件驱动的框架可能会以一个实现了一系列基本事件处理器方法的基类开始：

.. code-block:: python

    class EventHandler:
        def fileno(self):
            'Return the associated file descriptor'
            raise NotImplemented('must implement')

        def wants_to_receive(self):
            'Return True if receiving is allowed'
            return False

        def handle_receive(self):
            'Perform the receive operation'
            pass

        def wants_to_send(self):
            'Return True if sending is requested'
            return False

        def handle_send(self):
            'Send outgoing data'
            pass

这个类的实例作为插件被放入类似下面这样的事件循环中：
