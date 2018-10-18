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

.. code-block:: python

    import select

    def event_loop(handlers):
        while True:
            wants_recv = [h for h in handlers if h.wants_to_receive()]
            wants_send = [h for h in handlers if h.wants_to_send()]
            can_recv, can_send, _ = select.select(wants_recv, wants_send, [])
            for h in can_recv:
                h.handle_receive()
            for h in can_send:
                h.handle_send()

事件循环的关键部分是 ``select()`` 调用，它会不断轮询文件描述符从而激活它。  
在调用 ``select()`` 之前，事件循环会询问所有的处理器来决定哪一个想接受或发生。  
然后它将结果列表提供给 ``select()`` 。然后 ``select()`` 返回准备接受或发送的对象组成的列表。  
然后相应的 ``handle_receive()`` 或 ``handle_send()`` 方法被触发。  

编写应用程序的时候，``EventHandler`` 的实例会被创建。例如，下面是两个简单的基于UDP网络服务的处理器例子：

.. code-block:: python

    import socket
    import time

    class UDPServer(EventHandler):
        def __init__(self, address):
            self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            self.sock.bind(address)

        def fileno(self):
            return self.sock.fileno()

        def wants_to_receive(self):
            return True

    class UDPTimeServer(UDPServer):
        def handle_receive(self):
            msg, addr = self.sock.recvfrom(1)
            self.sock.sendto(time.ctime().encode('ascii'), addr)

    class UDPEchoServer(UDPServer):
        def handle_receive(self):
            msg, addr = self.sock.recvfrom(8192)
            self.sock.sendto(msg, addr)

    if __name__ == '__main__':
        handlers = [ UDPTimeServer(('',14000)), UDPEchoServer(('',15000))  ]
        event_loop(handlers)

测试这段代码，试着从另外一个Python解释器连接它：

.. code-block:: python

    >>> from socket import *
    >>> s = socket(AF_INET, SOCK_DGRAM)
    >>> s.sendto(b'',('localhost',14000))
    0
    >>> s.recvfrom(128)
    (b'Tue Sep 18 14:29:23 2012', ('127.0.0.1', 14000))
    >>> s.sendto(b'Hello',('localhost',15000))
    5
    >>> s.recvfrom(128)
    (b'Hello', ('127.0.0.1', 15000))
    >>>

实现一个TCP服务器会更加复杂一点，因为每一个客户端都要初始化一个新的处理器对象。
下面是一个TCP应答客户端例子：

.. code-block:: python

    class TCPServer(EventHandler):
        def __init__(self, address, client_handler, handler_list):
            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
            self.sock.bind(address)
            self.sock.listen(1)
            self.client_handler = client_handler
            self.handler_list = handler_list

        def fileno(self):
            return self.sock.fileno()

        def wants_to_receive(self):
            return True

        def handle_receive(self):
            client, addr = self.sock.accept()
            # Add the client to the event loop's handler list
            self.handler_list.append(self.client_handler(client, self.handler_list))

    class TCPClient(EventHandler):
        def __init__(self, sock, handler_list):
            self.sock = sock
            self.handler_list = handler_list
            self.outgoing = bytearray()

        def fileno(self):
            return self.sock.fileno()

        def close(self):
            self.sock.close()
            # Remove myself from the event loop's handler list
            self.handler_list.remove(self)

        def wants_to_send(self):
            return True if self.outgoing else False

        def handle_send(self):
            nsent = self.sock.send(self.outgoing)
            self.outgoing = self.outgoing[nsent:]

    class TCPEchoClient(TCPClient):
        def wants_to_receive(self):
            return True

        def handle_receive(self):
            data = self.sock.recv(8192)
            if not data:
                self.close()
            else:
                self.outgoing.extend(data)

    if __name__ == '__main__':
       handlers = []
       handlers.append(TCPServer(('',16000), TCPEchoClient, handlers))
       event_loop(handlers)

TCP例子的关键点是从处理器中列表增加和删除客户端的操作。
对每一个连接，一个新的处理器被创建并加到列表中。当连接被关闭后，每个客户端负责将其从列表中删除。
如果你运行程序并试着用Telnet或类似工具连接，它会将你发送的消息回显给你。并且它能很轻松的处理多客户端连接。

----------
讨论
----------
实际上所有的事件驱动框架原理跟上面的例子相差无几。实际的实现细节和软件架构可能不一样，
但是在最核心的部分，都会有一个轮询的循环来检查活动socket，并执行响应操作。

事件驱动I/O的一个可能好处是它能处理非常大的并发连接，而不需要使用多线程或多进程。
也就是说，``select()`` 调用（或其他等效的）能监听大量的socket并响应它们中任何一个产生事件的。
在循环中一次处理一个事件，并不需要其他的并发机制。

事件驱动I/O的缺点是没有真正的同步机制。
如果任何事件处理器方法阻塞或执行一个耗时计算，它会阻塞所有的处理进程。
调用那些并不是事件驱动风格的库函数也会有问题，同样要是某些库函数调用会阻塞，那么也会导致整个事件循环停止。

对于阻塞或耗时计算的问题可以通过将事件发送个其他单独的现场或进程来处理。
不过，在事件循环中引入多线程和多进程是比较棘手的，
下面的例子演示了如何使用 ``concurrent.futures`` 模块来实现：

.. code-block:: python

    from concurrent.futures import ThreadPoolExecutor
    import os

    class ThreadPoolHandler(EventHandler):
        def __init__(self, nworkers):
            if os.name == 'posix':
                self.signal_done_sock, self.done_sock = socket.socketpair()
            else:
                server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                server.bind(('127.0.0.1', 0))
                server.listen(1)
                self.signal_done_sock = socket.socket(socket.AF_INET,
                                                      socket.SOCK_STREAM)
                self.signal_done_sock.connect(server.getsockname())
                self.done_sock, _ = server.accept()
                server.close()

            self.pending = []
            self.pool = ThreadPoolExecutor(nworkers)

        def fileno(self):
            return self.done_sock.fileno()

        # Callback that executes when the thread is done
        def _complete(self, callback, r):

            self.pending.append((callback, r.result()))
            self.signal_done_sock.send(b'x')

        # Run a function in a thread pool
        def run(self, func, args=(), kwargs={},*,callback):
            r = self.pool.submit(func, *args, **kwargs)
            r.add_done_callback(lambda r: self._complete(callback, r))

        def wants_to_receive(self):
            return True

        # Run callback functions of completed work
        def handle_receive(self):
            # Invoke all pending callback functions
            for callback, result in self.pending:
                callback(result)
                self.done_sock.recv(1)
            self.pending = []

在代码中，``run()`` 方法被用来将工作提交给回调函数池，处理完成后被激发。
实际工作被提交给 ``ThreadPoolExecutor`` 实例。
不过一个难点是协调计算结果和事件循环，为了解决它，我们创建了一对socket并将其作为某种信号量机制来使用。
当线程池完成工作后，它会执行类中的 ``_complete()`` 方法。
这个方法再某个socket上写入字节之前会讲挂起的回调函数和结果放入队列中。
``fileno()`` 方法返回另外的那个socket。
因此，这个字节被写入时，它会通知事件循环，
然后 ``handle_receive()`` 方法被激活并为所有之前提交的工作执行回调函数。
坦白讲，说了这么多连我自己都晕了。
下面是一个简单的服务器，演示了如何使用线程池来实现耗时的计算：

.. code-block:: python

    # A really bad Fibonacci implementation
    def fib(n):
        if n < 2:
            return 1
        else:
            return fib(n - 1) + fib(n - 2)

    class UDPFibServer(UDPServer):
        def handle_receive(self):
            msg, addr = self.sock.recvfrom(128)
            n = int(msg)
            pool.run(fib, (n,), callback=lambda r: self.respond(r, addr))

        def respond(self, result, addr):
            self.sock.sendto(str(result).encode('ascii'), addr)

    if __name__ == '__main__':
        pool = ThreadPoolHandler(16)
        handlers = [ pool, UDPFibServer(('',16000))]
        event_loop(handlers)

运行这个服务器，然后试着用其它Python程序来测试它：

.. code-block:: python

    from socket import *
    sock = socket(AF_INET, SOCK_DGRAM)
    for x in range(40):
        sock.sendto(str(x).encode('ascii'), ('localhost', 16000))
        resp = sock.recvfrom(8192)
        print(resp[0])

你应该能在不同窗口中重复的执行这个程序，并且不会影响到其他程序，尽管当数字便越来越大时候它会变得越来越慢。

已经阅读完了这一小节，那么你应该使用这里的代码吗？也许不会。你应该选择一个可以完成同样任务的高级框架。
不过，如果你理解了基本原理，你就能理解这些框架所使用的核心技术。
作为对回调函数编程的替代，事件驱动编码有时候会使用到协程，参考12.12小节的一个例子。

==============================
11.13 发送与接收大型数组
==============================

----------
问题
----------
你要通过网络连接发送和接受连续数据的大型数组，并尽量减少数据的复制操作。

----------
解决方案
----------
下面的函数利用 ``memoryviews`` 来发送和接受大数组：

.. code-block:: python

    # zerocopy.py

    def send_from(arr, dest):
        view = memoryview(arr).cast('B')
        while len(view):
            nsent = dest.send(view)
            view = view[nsent:]

    def recv_into(arr, source):
        view = memoryview(arr).cast('B')
        while len(view):
            nrecv = source.recv_into(view)
            view = view[nrecv:]

为了测试程序，首先创建一个通过socket连接的服务器和客户端程序：

.. code-block:: python

    >>> from socket import *
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s.bind(('', 25000))
    >>> s.listen(1)
    >>> c,a = s.accept()
    >>>

在客户端（另外一个解释器中）：

.. code-block:: python

    >>> from socket import *
    >>> c = socket(AF_INET, SOCK_STREAM)
    >>> c.connect(('localhost', 25000))
    >>>

本节的目标是你能通过连接传输一个超大数组。这种情况的话，可以通过 ``array`` 模块或 ``numpy`` 模块来创建数组：

.. code-block:: python

    # Server
    >>> import numpy
    >>> a = numpy.arange(0.0, 50000000.0)
    >>> send_from(a, c)
    >>>

    # Client
    >>> import numpy
    >>> a = numpy.zeros(shape=50000000, dtype=float)
    >>> a[0:10]
    array([ 0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.])
    >>> recv_into(a, c)
    >>> a[0:10]
    array([ 0.,  1.,  2.,  3.,  4.,  5.,  6.,  7.,  8.,  9.])
    >>>

----------
讨论
----------
在数据密集型分布式计算和平行计算程序中，自己写程序来实现发送/接受大量数据并不常见。
不过，要是你确实想这样做，你可能需要将你的数据转换成原始字节，以便给低层的网络函数使用。
你可能还需要将数据切割成多个块，因为大部分和网络相关的函数并不能一次性发送或接受超大数据块。

一种方法是使用某种机制序列化数据——可能将其转换成一个字节字符串。
不过，这样最终会创建数据的一个复制。
就算你只是零碎的做这些，你的代码最终还是会有大量的小型复制操作。

本节通过使用内存视图展示了一些魔法操作。
本质上，一个内存视图就是一个已存在数组的覆盖层。不仅仅是那样，
内存视图还能以不同的方式转换成不同类型来表现数据。
这个就是下面这个语句的目的：

.. code-block:: python

    view = memoryview(arr).cast('B')

它接受一个数组 arr并将其转换为一个无符号字节的内存视图。这个视图能被传递给socket相关函数，
比如 ``socket.send()`` 或 ``send.recv_into()`` 。
在内部，这些方法能够直接操作这个内存区域。例如，``sock.send()`` 直接从内存中发生数据而不需要复制。
``send.recv_into()`` 使用这个内存区域作为接受操作的输入缓冲区。

剩下的一个难点就是socket函数可能只操作部分数据。
通常来讲，我们得使用很多不同的 ``send()`` 和 ``recv_into()`` 来传输整个数组。
不用担心，每次操作后，视图会通过发送或接受字节数量被切割成新的视图。
新的视图同样也是内存覆盖层。因此，还是没有任何的复制操作。

这里有个问题就是接受者必须事先知道有多少数据要被发送，
以便它能预分配一个数组或者确保它能将接受的数据放入一个已经存在的数组中。
如果没办法知道的话，发送者就得先将数据大小发送过来，然后再发送实际的数组数据。

============================
12.1 启动与停止线程
============================

----------
问题
----------
你要为需要并发执行的代码创建/销毁线程

----------
解决方案
----------
``threading`` 库可以在单独的线程中执行任何的在 Python 中可以调用的对象。你可以创建一个 ``Thread`` 对象并将你要执行的对象以 target 参数的形式提供给该对象。
下面是一个简单的例子：

.. code-block:: python

   # Code to execute in an independent thread
   import time
   def countdown(n):
       while n > 0:
           print('T-minus', n)
           n -= 1
           time.sleep(5)

   # Create and launch a thread
   from threading import Thread
   t = Thread(target=countdown, args=(10,))
   t.start()

当你创建好一个线程对象后，该对象并不会立即执行，除非你调用它的 ``start()`` 方法（当你调用 ``start()`` 方法时，它会调用你传递进来的函数，并把你传递进来的参数传递给该函数）。Python中的线程会在一个单独的系统级线程中执行（比如说一个 POSIX 线程或者一个 Windows 线程），这些线程将由操作系统来全权管理。线程一旦启动，将独立执行直到目标函数返回。你可以查询一个线程对象的状态，看它是否还在执行：

.. code-block:: python

   if t.is_alive():
       print('Still running')
   else:
       print('Completed')

你也可以将一个线程加入到当前线程，并等待它终止：

.. code-block:: python

   t.join()

Python解释器直到所有线程都终止前仍保持运行。对于需要长时间运行的线程或者需要一直运行的后台任务，你应当考虑使用后台线程。
例如：

.. code-block:: python

   t = Thread(target=countdown, args=(10,), daemon=True)
   t.start()

后台线程无法等待，不过，这些线程会在主线程终止时自动销毁。
除了如上所示的两个操作，并没有太多可以对线程做的事情。你无法结束一个线程，无法给它发送信号，无法调整它的调度，也无法执行其他高级操作。如果需要这些特性，你需要自己添加。比如说，如果你需要终止线程，那么这个线程必须通过编程在某个特定点轮询来退出。你可以像下边这样把线程放入一个类中：

.. code-block:: python

   class CountdownTask:
       def __init__(self):
           self._running = True

       def terminate(self):
           self._running = False

       def run(self, n):
           while self._running and n > 0:
               print('T-minus', n)
               n -= 1
               time.sleep(5)

   c = CountdownTask()
   t = Thread(target=c.run, args=(10,))
   t.start()
   c.terminate() # Signal termination
   t.join()      # Wait for actual termination (if needed)

如果线程执行一些像I/O这样的阻塞操作，那么通过轮询来终止线程将使得线程之间的协调变得非常棘手。比如，如果一个线程一直阻塞在一个I/O操作上，它就永远无法返回，也就无法检查自己是否已经被结束了。要正确处理这些问题，你需要利用超时循环来小心操作线程。
例子如下：

.. code-block:: python

   class IOTask:
       def terminate(self):
           self._running = False

       def run(self, sock):
           # sock is a socket
           sock.settimeout(5)        # Set timeout period
           while self._running:
               # Perform a blocking I/O operation w/ timeout
               try:
                   data = sock.recv(8192)
                   break
               except socket.timeout:
                   continue
               # Continued processing
               ...
           # Terminated
           return
----------
讨论
----------
由于全局解释锁（GIL）的原因，Python 的线程被限制到同一时刻只允许一个线程执行这样一个执行模型。所以，Python 的线程更适用于处理I/O和其他需要并发执行的阻塞操作（比如等待I/O、等待从数据库获取数据等等），而不是需要多处理器并行的计算密集型任务。

有时你会看到下边这种通过继承 ``Thread`` 类来实现的线程：

.. code-block:: python

   from threading import Thread

   class CountdownThread(Thread):
       def __init__(self, n):
           super().__init__()
           self.n = n
       def run(self):
           while self.n > 0:

               print('T-minus', self.n)
               self.n -= 1
               time.sleep(5)

   c = CountdownThread(5)
   c.start()

尽管这样也可以工作，但这使得你的代码依赖于 ``threading`` 库，所以你的这些代码只能在线程上下文中使用。上文所写的那些代码、函数都是与 ``threading`` 库无关的，这样就使得这些代码可以被用在其他的上下文中，可能与线程有关，也可能与线程无关。比如，你可以通过 ``multiprocessing`` 模块在一个单独的进程中执行你的代码：

.. code-block:: python

   import multiprocessing
   c = CountdownTask(5)
   p = multiprocessing.Process(target=c.run)
   p.start()


再次重申，这段代码仅适用于 CountdownTask 类是以独立于实际的并发手段（多线程、多进程等等）实现的情况。
============================
12.2 判断线程是否已经启动
============================

----------
问题
----------

你已经启动了一个线程，但是你想知道它是不是真的已经开始运行了。

----------
解决方案
----------

线程的一个关键特性是每个线程都是独立运行且状态不可预测。如果程序中的其他线程需要通过判断某个线程的状态来确定自己下一步的操作，这时线程同步问题就会变得非常棘手。为了解决这些问题，我们需要使用 ``threading`` 库中的 ``Event`` 对象。
``Event`` 对象包含一个可由线程设置的信号标志，它允许线程等待某些事件的发生。在初始情况下，event 对象中的信号标志被设置为假。如果有线程等待一个 event 对象，而这个 event 对象的标志为假，那么这个线程将会被一直阻塞直至该标志为真。一个线程如果将一个 event 对象的信号标志设置为真，它将唤醒所有等待这个 event 对象的线程。如果一个线程等待一个已经被设置为真的 event 对象，那么它将忽略这个事件，继续执行。
下边的代码展示了如何使用 ``Event`` 来协调线程的启动：

.. code-block:: python

   from threading import Thread, Event
   import time

   # Code to execute in an independent thread
   def countdown(n, started_evt):
       print('countdown starting')
       started_evt.set()
       while n > 0:
           print('T-minus', n)
           n -= 1
           time.sleep(5)

   # Create the event object that will be used to signal startup
   started_evt = Event()

   # Launch the thread and pass the startup event
   print('Launching countdown')
   t = Thread(target=countdown, args=(10,started_evt))
   t.start()

   # Wait for the thread to start
   started_evt.wait()
   print('countdown is running')

当你执行这段代码，“countdown is running” 总是显示在 “countdown starting” 之后显示。这是由于使用 event 来协调线程，使得主线程要等到 ``countdown()`` 函数输出启动信息后，才能继续执行。

----------
讨论
----------
event 对象最好单次使用，就是说，你创建一个 event 对象，让某个线程等待这个对象，一旦这个对象被设置为真，你就应该丢弃它。尽管可以通过 ``clear()`` 方法来重置 event 对象，但是很难确保安全地清理 event 对象并对它重新赋值。很可能会发生错过事件、死锁或者其他问题（特别是，你无法保证重置 event 对象的代码会在线程再次等待这个 event 对象之前执行）。如果一个线程需要不停地重复使用 event 对象，你最好使用 ``Condition`` 对象来代替。下面的代码使用 ``Condition`` 对象实现了一个周期定时器，每当定时器超时的时候，其他线程都可以监测到：

.. code-block:: python

   import threading
   import time

   class PeriodicTimer:
       def __init__(self, interval):
           self._interval = interval
           self._flag = 0
           self._cv = threading.Condition()

       def start(self):
           t = threading.Thread(target=self.run)
           t.daemon = True

           t.start()

       def run(self):
           '''
           Run the timer and notify waiting threads after each interval
           '''
           while True:
               time.sleep(self._interval)
               with self._cv:
                    self._flag ^= 1
                    self._cv.notify_all()

       def wait_for_tick(self):
           '''
           Wait for the next tick of the timer
           '''
           with self._cv:
               last_flag = self._flag
               while last_flag == self._flag:
                   self._cv.wait()

   # Example use of the timer
   ptimer = PeriodicTimer(5)
   ptimer.start()

   # Two threads that synchronize on the timer
   def countdown(nticks):
       while nticks > 0:
           ptimer.wait_for_tick()
           print('T-minus', nticks)
           nticks -= 1

   def countup(last):
       n = 0
       while n < last:
           ptimer.wait_for_tick()
           print('Counting', n)
           n += 1

   threading.Thread(target=countdown, args=(10,)).start()
   threading.Thread(target=countup, args=(5,)).start()

event对象的一个重要特点是当它被设置为真时会唤醒所有等待它的线程。如果你只想唤醒单个线程，最好是使用信号量或者 ``Condition`` 对象来替代。考虑一下这段使用信号量实现的代码：

.. code-block:: python

    # Worker thread
    def worker(n, sema):
        # Wait to be signaled
        sema.acquire()

        # Do some work
        print('Working', n)

    # Create some threads
    sema = threading.Semaphore(0)
    nworkers = 10
    for n in range(nworkers):
        t = threading.Thread(target=worker, args=(n, sema,))
        t.start()

运行上边的代码将会启动一个线程池，但是并没有什么事情发生。这是因为所有的线程都在等待获取信号量。每次信号量被释放，只有一个线程会被唤醒并执行，示例如下：

.. code-block:: python

   >>> sema.release()
   Working 0
   >>> sema.release()
   Working 1
   >>>

编写涉及到大量的线程间同步问题的代码会让你痛不欲生。比较合适的方式是使用队列来进行线程间通信或者每个把线程当作一个Actor，利用Actor模型来控制并发。下一节将会介绍到队列，而Actor模型将在12.10节介绍。============================
12.3 线程间通信
============================

----------
问题
----------

你的程序中有多个线程，你需要在这些线程之间安全地交换信息或数据

----------
解决方案
----------
从一个线程向另一个线程发送数据最安全的方式可能就是使用 ``queue`` 库中的队列了。创建一个被多个线程共享的 ``Queue`` 对象，这些线程通过使用 ``put()`` 和 ``get()`` 操作来向队列中添加或者删除元素。
例如：

.. code-block:: python

   from queue import Queue
   from threading import Thread

   # A thread that produces data
   def producer(out_q):
       while True:
           # Produce some data
           ...
           out_q.put(data)

   # A thread that consumes data
   def consumer(in_q):
       while True:
   # Get some data
           data = in_q.get()
           # Process the data
           ...

   # Create the shared queue and launch both threads
   q = Queue()
   t1 = Thread(target=consumer, args=(q,))
   t2 = Thread(target=producer, args=(q,))
   t1.start()
   t2.start()

``Queue`` 对象已经包含了必要的锁，所以你可以通过它在多个线程间多安全地共享数据。
当使用队列时，协调生产者和消费者的关闭问题可能会有一些麻烦。一个通用的解决方法是在队列中放置一个特殊的值，当消费者读到这个值的时候，终止执行。例如：

.. code-block:: python

   from queue import Queue
   from threading import Thread

   # Object that signals shutdown
   _sentinel = object()

   # A thread that produces data
   def producer(out_q):
       while running:
           # Produce some data
           ...
           out_q.put(data)

       # Put the sentinel on the queue to indicate completion
       out_q.put(_sentinel)

   # A thread that consumes data
   def consumer(in_q):
       while True:
           # Get some data
           data = in_q.get()

           # Check for termination
           if data is _sentinel:
               in_q.put(_sentinel)
               break

           # Process the data
           ...

本例中有一个特殊的地方：消费者在读到这个特殊值之后立即又把它放回到队列中，将之传递下去。这样，所有监听这个队列的消费者线程就可以全部关闭了。
尽管队列是最常见的线程间通信机制，但是仍然可以自己通过创建自己的数据结构并添加所需的锁和同步机制来实现线程间通信。最常见的方法是使用 ``Condition`` 变量来包装你的数据结构。下边这个例子演示了如何创建一个线程安全的优先级队列，如同1.5节中介绍的那样。

.. code-block:: python

   import heapq
   import threading

   class PriorityQueue:
       def __init__(self):
           self._queue = []
           self._count = 0
           self._cv = threading.Condition()
       def put(self, item, priority):
           with self._cv:
               heapq.heappush(self._queue, (-priority, self._count, item))
               self._count += 1
               self._cv.notify()

       def get(self):
           with self._cv:
               while len(self._queue) == 0:
                   self._cv.wait()
               return heapq.heappop(self._queue)[-1]

使用队列来进行线程间通信是一个单向、不确定的过程。通常情况下，你没有办法知道接收数据的线程是什么时候接收到的数据并开始工作的。不过队列对象提供一些基本完成的特性，比如下边这个例子中的 ``task_done()`` 和 ``join()`` ：

.. code-block:: python

   from queue import Queue
   from threading import Thread

   # A thread that produces data
   def producer(out_q):
       while running:
           # Produce some data
           ...
           out_q.put(data)

   # A thread that consumes data
   def consumer(in_q):
       while True:
           # Get some data
           data = in_q.get()

           # Process the data
           ...
           # Indicate completion
           in_q.task_done()

   # Create the shared queue and launch both threads
   q = Queue()
   t1 = Thread(target=consumer, args=(q,))
   t2 = Thread(target=producer, args=(q,))
   t1.start()
   t2.start()

   # Wait for all produced items to be consumed
   q.join()

如果一个线程需要在一个“消费者”线程处理完特定的数据项时立即得到通知，你可以把要发送的数据和一个 ``Event`` 放到一起使用，这样“生产者”就可以通过这个Event对象来监测处理的过程了。示例如下：

.. code-block:: python

   from queue import Queue
   from threading import Thread, Event

   # A thread that produces data
   def producer(out_q):
       while running:
           # Produce some data
           ...
           # Make an (data, event) pair and hand it to the consumer
           evt = Event()
           out_q.put((data, evt))
           ...
           # Wait for the consumer to process the item
           evt.wait()

   # A thread that consumes data
   def consumer(in_q):
       while True:
           # Get some data
           data, evt = in_q.get()
           # Process the data
           ...
           # Indicate completion
           evt.set()

----------
讨论
----------
基于简单队列编写多线程程序在多数情况下是一个比较明智的选择。从线程安全队列的底层实现来看，你无需在你的代码中使用锁和其他底层的同步机制，这些只会把你的程序弄得乱七八糟。此外，使用队列这种基于消息的通信机制可以被扩展到更大的应用范畴，比如，你可以把你的程序放入多个进程甚至是分布式系统而无需改变底层的队列结构。
使用线程队列有一个要注意的问题是，向队列中添加数据项时并不会复制此数据项，线程间通信实际上是在线程间传递对象引用。如果你担心对象的共享状态，那你最好只传递不可修改的数据结构（如：整型、字符串或者元组）或者一个对象的深拷贝。例如：

.. code-block:: python

   from queue import Queue
   from threading import Thread
   import copy

   # A thread that produces data
   def producer(out_q):
       while True:
           # Produce some data
           ...
           out_q.put(copy.deepcopy(data))

   # A thread that consumes data
   def consumer(in_q):
       while True:
           # Get some data
           data = in_q.get()
           # Process the data
           ...
``Queue`` 对象提供一些在当前上下文很有用的附加特性。比如在创建 Queue 对象时提供可选的 ``size`` 参数来限制可以添加到队列中的元素数量。对于“生产者”与“消费者”速度有差异的情况，为队列中的元素数量添加上限是有意义的。比如，一个“生产者”产生项目的速度比“消费者” “消费”的速度快，那么使用固定大小的队列就可以在队列已满的时候阻塞队列，以免未预期的连锁效应扩散整个程序造成死锁或者程序运行失常。在通信的线程之间进行“流量控制”是一个看起来容易实现起来困难的问题。如果你发现自己曾经试图通过摆弄队列大小来解决一个问题，这也许就标志着你的程序可能存在脆弱设计或者固有的可伸缩问题。
``get()`` 和 ``put()`` 方法都支持非阻塞方式和设定超时，例如：

.. code-block:: python

   import queue
   q = queue.Queue()

   try:
       data = q.get(block=False)
   except queue.Empty:
       ...

   try:
       q.put(item, block=False)
   except queue.Full:
       ...

   try:
       data = q.get(timeout=5.0)
   except queue.Empty:
       ...

这些操作都可以用来避免当执行某些特定队列操作时发生无限阻塞的情况，比如，一个非阻塞的 ``put()`` 方法和一个固定大小的队列一起使用，这样当队列已满时就可以执行不同的代码。比如输出一条日志信息并丢弃。

.. code-block:: python

   def producer(q):
       ...
       try:
           q.put(item, block=False)
       except queue.Full:
           log.warning('queued item %r discarded!', item)

如果你试图让消费者线程在执行像 ``q.get()`` 这样的操作时，超时自动终止以便检查终止标志，你应该使用 ``q.get()`` 的可选参数 ``timeout`` ，如下：

.. code-block:: python

   _running = True

   def consumer(q):
       while _running:
           try:
               item = q.get(timeout=5.0)
               # Process item
               ...
           except queue.Empty:
               pass

最后，有 ``q.qsize()`` ， ``q.full()`` ， ``q.empty()`` 等实用方法可以获取一个队列的当前大小和状态。但要注意，这些方法都不是线程安全的。可能你对一个队列使用 ``empty()`` 判断出这个队列为空，但同时另外一个线程可能已经向这个队列中插入一个数据项。所以，你最好不要在你的代码中使用这些方法。
============================
12.4 给关键部分加锁
============================

----------
问题
----------

你需要对多线程程序中的临界区加锁以避免竞争条件。

----------
解决方案
----------
要在多线程程序中安全使用可变对象，你需要使用 threading 库中的 ``Lock`` 对象，就像下边这个例子这样：

.. code-block:: python

   import threading

   class SharedCounter:
       '''
       A counter object that can be shared by multiple threads.
       '''
       def __init__(self, initial_value = 0):
           self._value = initial_value
           self._value_lock = threading.Lock()

       def incr(self,delta=1):
           '''
           Increment the counter with locking
           '''
           with self._value_lock:
                self._value += delta

       def decr(self,delta=1):
           '''
           Decrement the counter with locking
           '''
           with self._value_lock:
                self._value -= delta

``Lock`` 对象和 ``with`` 语句块一起使用可以保证互斥执行，就是每次只有一个线程可以执行 with 语句包含的代码块。with 语句会在这个代码块执行前自动获取锁，在执行结束后自动释放锁。

----------
讨论
----------
线程调度本质上是不确定的，因此，在多线程程序中错误地使用锁机制可能会导致随机数据损坏或者其他的异常行为，我们称之为竞争条件。为了避免竞争条件，最好只在临界区（对临界资源进行操作的那部分代码）使用锁。
在一些“老的” Python 代码中，显式获取和释放锁是很常见的。下边是一个上一个例子的变种：

.. code-block:: python

   import threading

   class SharedCounter:
       '''
       A counter object that can be shared by multiple threads.
       '''
       def __init__(self, initial_value = 0):
           self._value = initial_value
           self._value_lock = threading.Lock()

       def incr(self,delta=1):
           '''
           Increment the counter with locking
           '''
           self._value_lock.acquire()
           self._value += delta
           self._value_lock.release()

       def decr(self,delta=1):
           '''
           Decrement the counter with locking
           '''
           self._value_lock.acquire()
           self._value -= delta
           self._value_lock.release()

相比于这种显式调用的方法，with 语句更加优雅，也更不容易出错，特别是程序员可能会忘记调用 release() 方法或者程序在获得锁之后产生异常这两种情况（使用 with 语句可以保证在这两种情况下仍能正确释放锁）。
为了避免出现死锁的情况，使用锁机制的程序应该设定为每个线程一次只允许获取一个锁。如果不能这样做的话，你就需要更高级的死锁避免机制，我们将在12.5节介绍。
在 ``threading`` 库中还提供了其他的同步原语，比如 ``RLock`` 和 ``Semaphore`` 对象。但是根据以往经验，这些原语是用于一些特殊的情况，如果你只是需要简单地对可变对象进行锁定，那就不应该使用它们。一个 ``RLock`` （可重入锁）可以被同一个线程多次获取，主要用来实现基于监测对象模式的锁定和同步。在使用这种锁的情况下，当锁被持有时，只有一个线程可以使用完整的函数或者类中的方法。比如，你可以实现一个这样的 SharedCounter 类：

.. code-block:: python

   import threading

   class SharedCounter:
       '''
       A counter object that can be shared by multiple threads.
       '''
       _lock = threading.RLock()
       def __init__(self, initial_value = 0):
           self._value = initial_value

       def incr(self,delta=1):
           '''
           Increment the counter with locking
           '''
           with SharedCounter._lock:
               self._value += delta

       def decr(self,delta=1):
           '''
           Decrement the counter with locking
           '''
           with SharedCounter._lock:
                self.incr(-delta)

在上边这个例子中，没有对每一个实例中的可变对象加锁，取而代之的是一个被所有实例共享的类级锁。这个锁用来同步类方法，具体来说就是，这个锁可以保证一次只有一个线程可以调用这个类方法。不过，与一个标准的锁不同的是，已经持有这个锁的方法在调用同样使用这个锁的方法时，无需再次获取锁。比如 decr 方法。
这种实现方式的一个特点是，无论这个类有多少个实例都只用一个锁。因此在需要大量使用计数器的情况下内存效率更高。不过这样做也有缺点，就是在程序中使用大量线程并频繁更新计数器时会有争用锁的问题。
信号量对象是一个建立在共享计数器基础上的同步原语。如果计数器不为0，with 语句将计数器减1，线程被允许执行。with 语句执行结束后，计数器加１。如果计数器为0，线程将被阻塞，直到其他线程结束将计数器加1。尽管你可以在程序中像标准锁一样使用信号量来做线程同步，但是这种方式并不被推荐，因为使用信号量为程序增加的复杂性会影响程序性能。相对于简单地作为锁使用，信号量更适用于那些需要在线程之间引入信号或者限制的程序。比如，你需要限制一段代码的并发访问量，你就可以像下面这样使用信号量完成：

.. code-block:: python

   from threading import Semaphore
   import urllib.request

   # At most, five threads allowed to run at once
   _fetch_url_sema = Semaphore(5)

   def fetch_url(url):
       with _fetch_url_sema:
           return urllib.request.urlopen(url)

如果你对线程同步原语的底层理论和实现感兴趣，可以参考操作系统相关书籍，绝大多数都有提及。
============================
12.5 防止死锁的加锁机制
============================

----------
问题
----------
你正在写一个多线程程序，其中线程需要一次获取多个锁，此时如何避免死锁问题。

----------
解决方案
----------
在多线程程序中，死锁问题很大一部分是由于线程同时获取多个锁造成的。举个例子：一个线程获取了第一个锁，然后在获取第二个锁的
时候发生阻塞，那么这个线程就可能阻塞其他线程的执行，从而导致整个程序假死。
解决死锁问题的一种方案是为程序中的每一个锁分配一个唯一的id，然后只允许按照升序规则来使用多个锁，这个规则使用上下文管理器
是非常容易实现的，示例如下：

.. code-block:: python

   import threading
   from contextlib import contextmanager

   # Thread-local state to stored information on locks already acquired
   _local = threading.local()

   @contextmanager
   def acquire(*locks):
       # Sort locks by object identifier
       locks = sorted(locks, key=lambda x: id(x))

       # Make sure lock order of previously acquired locks is not violated
       acquired = getattr(_local,'acquired',[])
       if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
           raise RuntimeError('Lock Order Violation')

       # Acquire all of the locks
       acquired.extend(locks)
       _local.acquired = acquired

       try:
           for lock in locks:
               lock.acquire()
           yield
       finally:
           # Release locks in reverse order of acquisition
           for lock in reversed(locks):
               lock.release()
           del acquired[-len(locks):]

如何使用这个上下文管理器呢？你可以按照正常途径创建一个锁对象，但不论是单个锁还是多个锁中都使用 ``acquire()`` 函数来申请锁，
示例如下：

.. code-block:: python

   import threading
   x_lock = threading.Lock()
   y_lock = threading.Lock()

   def thread_1():
       while True:
           with acquire(x_lock, y_lock):
               print('Thread-1')

   def thread_2():
       while True:
           with acquire(y_lock, x_lock):
               print('Thread-2')

   t1 = threading.Thread(target=thread_1)
   t1.daemon = True
   t1.start()

   t2 = threading.Thread(target=thread_2)
   t2.daemon = True
   t2.start()

如果你执行这段代码，你会发现它即使在不同的函数中以不同的顺序获取锁也没有发生死锁。
其关键在于，在第一段代码中，我们对这些锁进行了排序。通过排序，使得不管用户以什么样的顺序来请求锁，这些锁都会按照固定的顺序被获取。
如果有多个 ``acquire()`` 操作被嵌套调用，可以通过线程本地存储（TLS）来检测潜在的死锁问题。
假设你的代码是这样写的：

.. code-block:: python

   import threading
   x_lock = threading.Lock()
   y_lock = threading.Lock()

   def thread_1():

       while True:
           with acquire(x_lock):
               with acquire(y_lock):
                   print('Thread-1')

   def thread_2():
       while True:
           with acquire(y_lock):
               with acquire(x_lock):
                   print('Thread-2')

   t1 = threading.Thread(target=thread_1)
   t1.daemon = True
   t1.start()

   t2 = threading.Thread(target=thread_2)
   t2.daemon = True
   t2.start()

如果你运行这个版本的代码，必定会有一个线程发生崩溃，异常信息可能像这样：

.. code-block:: python

   Exception in thread Thread-1:
   Traceback (most recent call last):
     File "/usr/local/lib/python3.3/threading.py", line 639, in _bootstrap_inner
       self.run()
     File "/usr/local/lib/python3.3/threading.py", line 596, in run
       self._target(*self._args, **self._kwargs)
     File "deadlock.py", line 49, in thread_1
       with acquire(y_lock):
     File "/usr/local/lib/python3.3/contextlib.py", line 48, in __enter__
       return next(self.gen)
     File "deadlock.py", line 15, in acquire
       raise RuntimeError("Lock Order Violation")
   RuntimeError: Lock Order Violation
   >>>

发生崩溃的原因在于，每个线程都记录着自己已经获取到的锁。 ``acquire()`` 函数会检查之前已经获取的锁列表，
由于锁是按照升序排列获取的，所以函数会认为之前已获取的锁的id必定小于新申请到的锁，这时就会触发异常。

----------
讨论
----------
死锁是每一个多线程程序都会面临的一个问题（就像它是每一本操作系统课本的共同话题一样）。根据经验来讲，尽可能保证每一个
线程只能同时保持一个锁，这样程序就不会被死锁问题所困扰。一旦有线程同时申请多个锁，一切就不可预料了。

死锁的检测与恢复是一个几乎没有优雅的解决方案的扩展话题。一个比较常用的死锁检测与恢复的方案是引入看门狗计数器。当线程正常
运行的时候会每隔一段时间重置计数器，在没有发生死锁的情况下，一切都正常进行。一旦发生死锁，由于无法重置计数器导致定时器
超时，这时程序会通过重启自身恢复到正常状态。

避免死锁是另外一种解决死锁问题的方式，在进程获取锁的时候会严格按照对象id升序排列获取，经过数学证明，这样保证程序不会进入
死锁状态。证明就留给读者作为练习了。避免死锁的主要思想是，单纯地按照对象id递增的顺序加锁不会产生循环依赖，而循环依赖是
死锁的一个必要条件，从而避免程序进入死锁状态。

下面以一个关于线程死锁的经典问题：“哲学家就餐问题”，作为本节最后一个例子。题目是这样的：五位哲学家围坐在一张桌子前，每个人
面前有一碗饭和一只筷子。在这里每个哲学家可以看做是一个独立的线程，而每只筷子可以看做是一个锁。每个哲学家可以处在静坐、
思考、吃饭三种状态中的一个。需要注意的是，每个哲学家吃饭是需要两只筷子的，这样问题就来了：如果每个哲学家都拿起自己左边的筷子，
那么他们五个都只能拿着一只筷子坐在那儿，直到饿死。此时他们就进入了死锁状态。
下面是一个简单的使用死锁避免机制解决“哲学家就餐问题”的实现：

.. code-block:: python

   import threading

   # The philosopher thread
   def philosopher(left, right):
       while True:
           with acquire(left,right):
                print(threading.currentThread(), 'eating')

   # The chopsticks (represented by locks)
   NSTICKS = 5
   chopsticks = [threading.Lock() for n in range(NSTICKS)]

   # Create all of the philosophers
   for n in range(NSTICKS):
       t = threading.Thread(target=philosopher,
                            args=(chopsticks[n],chopsticks[(n+1) % NSTICKS]))
       t.start()

最后，要特别注意到，为了避免死锁，所有的加锁操作必须使用 ``acquire()`` 函数。如果代码中的某部分绕过acquire
函数直接申请锁，那么整个死锁避免机制就不起作用了。
============================
12.6 保存线程的状态信息
============================

----------
问题
----------
你需要保存正在运行线程的状态，这个状态对于其他的线程是不可见的。

----------
解决方案
----------
有时在多线程编程中，你需要只保存当前运行线程的状态。
要这么做，可使用 ``thread.local()`` 创建一个本地线程存储对象。
对这个对象的属性的保存和读取操作都只会对执行线程可见，而其他线程并不可见。

作为使用本地存储的一个有趣的实际例子，
考虑在8.3小节定义过的 ``LazyConnection`` 上下文管理器类。
下面我们对它进行一些小的修改使得它可以适用于多线程：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM
    import threading

    class LazyConnection:
        def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
            self.address = address
            self.family = AF_INET
            self.type = SOCK_STREAM
            self.local = threading.local()

        def __enter__(self):
            if hasattr(self.local, 'sock'):
                raise RuntimeError('Already connected')
            self.local.sock = socket(self.family, self.type)
            self.local.sock.connect(self.address)
            return self.local.sock

        def __exit__(self, exc_ty, exc_val, tb):
            self.local.sock.close()
            del self.local.sock

代码中，自己观察对于 ``self.local`` 属性的使用。
它被初始化尾一个 ``threading.local()`` 实例。
其他方法操作被存储为 ``self.local.sock`` 的套接字对象。
有了这些就可以在多线程中安全的使用 ``LazyConnection`` 实例了。例如：

.. code-block:: python

    from functools import partial
    def test(conn):
        with conn as s:
            s.send(b'GET /index.html HTTP/1.0\r\n')
            s.send(b'Host: www.python.org\r\n')

            s.send(b'\r\n')
            resp = b''.join(iter(partial(s.recv, 8192), b''))

        print('Got {} bytes'.format(len(resp)))

    if __name__ == '__main__':
        conn = LazyConnection(('www.python.org', 80))

        t1 = threading.Thread(target=test, args=(conn,))
        t2 = threading.Thread(target=test, args=(conn,))
        t1.start()
        t2.start()
        t1.join()
        t2.join()

它之所以行得通的原因是每个线程会创建一个自己专属的套接字连接（存储为self.local.sock）。
因此，当不同的线程执行套接字操作时，由于操作的是不同的套接字，因此它们不会相互影响。

----------
讨论
----------
在大部分程序中创建和操作线程特定状态并不会有什么问题。
不过，当出了问题的时候，通常是因为某个对象被多个线程使用到，用来操作一些专用的系统资源，
比如一个套接字或文件。你不能让所有线程贡献一个单独对象，
因为多个线程同时读和写的时候会产生混乱。
本地线程存储通过让这些资源只能在被使用的线程中可见来解决这个问题。

本节中，使用 ``thread.local()`` 可以让 ``LazyConnection`` 类支持一个线程一个连接，
而不是对于所有的进程都只有一个连接。

其原理是，每个 ``threading.local()`` 实例为每个线程维护着一个单独的实例字典。
所有普通实例操作比如获取、修改和删除值仅仅操作这个字典。
每个线程使用一个独立的字典就可以保证数据的隔离了。

============================
12.7 创建一个线程池
============================

----------
问题
----------
你创建一个工作者线程池，用来相应客户端请求或执行其他的工作。

----------
解决方案
----------
``concurrent.futures`` 函数库有一个 ``ThreadPoolExecutor`` 类可以被用来完成这个任务。
下面是一个简单的TCP服务器，使用了一个线程池来响应客户端：

.. code-block:: python

    from socket import AF_INET, SOCK_STREAM, socket
    from concurrent.futures import ThreadPoolExecutor

    def echo_client(sock, client_addr):
        '''
        Handle a client connection
        '''
        print('Got connection from', client_addr)
        while True:
            msg = sock.recv(65536)
            if not msg:
                break
            sock.sendall(msg)
        print('Client closed connection')
        sock.close()

    def echo_server(addr):
        pool = ThreadPoolExecutor(128)
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(addr)
        sock.listen(5)
        while True:
            client_sock, client_addr = sock.accept()
            pool.submit(echo_client, client_sock, client_addr)

    echo_server(('',15000))

如果你想手动创建你自己的线程池，
通常可以使用一个Queue来轻松实现。下面是一个稍微不同但是手动实现的例子：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM
    from threading import Thread
    from queue import Queue

    def echo_client(q):
        '''
        Handle a client connection
        '''
        sock, client_addr = q.get()
        print('Got connection from', client_addr)
        while True:
            msg = sock.recv(65536)
            if not msg:
                break
            sock.sendall(msg)
        print('Client closed connection')

        sock.close()

    def echo_server(addr, nworkers):
        # Launch the client workers
        q = Queue()
        for n in range(nworkers):
            t = Thread(target=echo_client, args=(q,))
            t.daemon = True
            t.start()

        # Run the server
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(addr)
        sock.listen(5)
        while True:
            client_sock, client_addr = sock.accept()
            q.put((client_sock, client_addr))

    echo_server(('',15000), 128)

使用 ``ThreadPoolExecutor`` 相对于手动实现的一个好处在于它使得
任务提交者更方便的从被调用函数中获取返回值。例如，你可能会像下面这样写：

.. code-block:: python

    from concurrent.futures import ThreadPoolExecutor
    import urllib.request

    def fetch_url(url):
        u = urllib.request.urlopen(url)
        data = u.read()
        return data

    pool = ThreadPoolExecutor(10)
    # Submit work to the pool
    a = pool.submit(fetch_url, 'http://www.python.org')
    b = pool.submit(fetch_url, 'http://www.pypy.org')

    # Get the results back
    x = a.result()
    y = b.result()

例子中返回的handle对象会帮你处理所有的阻塞与协作，然后从工作线程中返回数据给你。
特别的，``a.result()`` 操作会阻塞进程直到对应的函数执行完成并返回一个结果。

----------
讨论
----------
通常来讲，你应该避免编写线程数量可以无限制增长的程序。例如，看看下面这个服务器：

.. code-block:: python

    from threading import Thread
    from socket import socket, AF_INET, SOCK_STREAM

    def echo_client(sock, client_addr):
        '''
        Handle a client connection
        '''
        print('Got connection from', client_addr)
        while True:
            msg = sock.recv(65536)
            if not msg:
                break
            sock.sendall(msg)
        print('Client closed connection')
        sock.close()

    def echo_server(addr, nworkers):
        # Run the server
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(addr)
        sock.listen(5)
        while True:
            client_sock, client_addr = sock.accept()
            t = Thread(target=echo_client, args=(client_sock, client_addr))
            t.daemon = True
            t.start()

    echo_server(('',15000))

尽管这个也可以工作，
但是它不能抵御有人试图通过创建大量线程让你服务器资源枯竭而崩溃的攻击行为。
通过使用预先初始化的线程池，你可以设置同时运行线程的上限数量。

你可能会关心创建大量线程会有什么后果。
现代操作系统可以很轻松的创建几千个线程的线程池。
甚至，同时几千个线程等待工作并不会对其他代码产生性能影响。
当然了，如果所有线程同时被唤醒并立即在CPU上执行，那就不同了——特别是有了全局解释器锁GIL。
通常，你应该只在I/O处理相关代码中使用线程池。

创建大的线程池的一个可能需要关注的问题是内存的使用。
例如，如果你在OS X系统上面创建2000个线程，系统显示Python进程使用了超过9GB的虚拟内存。
不过，这个计算通常是有误差的。当创建一个线程时，操作系统会预留一个虚拟内存区域来
放置线程的执行栈（通常是8MB大小）。但是这个内存只有一小片段被实际映射到真实内存中。
因此，Python进程使用到的真实内存其实很小
（比如，对于2000个线程来讲，只使用到了70MB的真实内存，而不是9GB）。
如果你担心虚拟内存大小，可以使用 ``threading.stack_size()`` 函数来降低它。例如：

.. code-block:: python

    import threading
    threading.stack_size(65536)

如果你加上这条语句并再次运行前面的创建2000个线程试验，
你会发现Python进程只使用到了大概210MB的虚拟内存，而真实内存使用量没有变。
注意线程栈大小必须至少为32768字节，通常是系统内存页大小（4096、8192等）的整数倍。
============================
12.8 简单的并行编程
============================

----------
问题
----------
你有个程序要执行CPU密集型工作，你想让他利用多核CPU的优势来运行的快一点。

----------
解决方案
----------
``concurrent.futures`` 库提供了一个 ``ProcessPoolExecutor`` 类，
可被用来在一个单独的Python解释器中执行计算密集型函数。
不过，要使用它，你首先要有一些计算密集型的任务。
我们通过一个简单而实际的例子来演示它。假定你有个Apache web服务器日志目录的gzip压缩包：

::

    logs/
       20120701.log.gz
       20120702.log.gz
       20120703.log.gz
       20120704.log.gz
       20120705.log.gz
       20120706.log.gz
       ...


进一步假设每个日志文件内容类似下面这样：

::

    124.115.6.12 - - [10/Jul/2012:00:18:50 -0500] "GET /robots.txt ..." 200 71
    210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /ply/ ..." 200 11875
    210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /favicon.ico ..." 404 369
    61.135.216.105 - - [10/Jul/2012:00:20:04 -0500] "GET /blog/atom.xml ..." 304 -
    ...

下面是一个脚本，在这些日志文件中查找出所有访问过robots.txt文件的主机：

.. code-block:: python

    # findrobots.py

    import gzip
    import io
    import glob

    def find_robots(filename):
        '''
        Find all of the hosts that access robots.txt in a single log file
        '''
        robots = set()
        with gzip.open(filename) as f:
            for line in io.TextIOWrapper(f,encoding='ascii'):
                fields = line.split()
                if fields[6] == '/robots.txt':
                    robots.add(fields[0])
        return robots

    def find_all_robots(logdir):
        '''
        Find all hosts across and entire sequence of files
        '''
        files = glob.glob(logdir+'/*.log.gz')
        all_robots = set()
        for robots in map(find_robots, files):
            all_robots.update(robots)
        return all_robots

    if __name__ == '__main__':
        robots = find_all_robots('logs')
        for ipaddr in robots:
            print(ipaddr)

前面的程序使用了通常的map-reduce风格来编写。
函数 ``find_robots()`` 在一个文件名集合上做map操作，并将结果汇总为一个单独的结果，
也就是 ``find_all_robots()`` 函数中的 ``all_robots`` 集合。
现在，假设你想要修改这个程序让它使用多核CPU。
很简单——只需要将map()操作替换为一个 ``concurrent.futures`` 库中生成的类似操作即可。
下面是一个简单修改版本：

.. code-block:: python

    # findrobots.py

    import gzip
    import io
    import glob
    from concurrent import futures

    def find_robots(filename):
        '''
        Find all of the hosts that access robots.txt in a single log file

        '''
        robots = set()
        with gzip.open(filename) as f:
            for line in io.TextIOWrapper(f,encoding='ascii'):
                fields = line.split()
                if fields[6] == '/robots.txt':
                    robots.add(fields[0])
        return robots

    def find_all_robots(logdir):
        '''
        Find all hosts across and entire sequence of files
        '''
        files = glob.glob(logdir+'/*.log.gz')
        all_robots = set()
        with futures.ProcessPoolExecutor() as pool:
            for robots in pool.map(find_robots, files):
                all_robots.update(robots)
        return all_robots

    if __name__ == '__main__':
        robots = find_all_robots('logs')
        for ipaddr in robots:
            print(ipaddr)

通过这个修改后，运行这个脚本产生同样的结果，但是在四核机器上面比之前快了3.5倍。
实际的性能优化效果根据你的机器CPU数量的不同而不同。

----------
讨论
----------
``ProcessPoolExecutor`` 的典型用法如下：

.. code-block:: python

    from concurrent.futures import ProcessPoolExecutor

    with ProcessPoolExecutor() as pool:
        ...
        do work in parallel using pool
        ...

其原理是，一个 ``ProcessPoolExecutor`` 创建N个独立的Python解释器，
N是系统上面可用CPU的个数。你可以通过提供可选参数给 ``ProcessPoolExecutor(N)`` 来修改
处理器数量。这个处理池会一直运行到with块中最后一个语句执行完成，
然后处理池被关闭。不过，程序会一直等待直到所有提交的工作被处理完成。

被提交到池中的工作必须被定义为一个函数。有两种方法去提交。
如果你想让一个列表推导或一个 ``map()`` 操作并行执行的话，可使用 ``pool.map()`` :

.. code-block:: python

    # A function that performs a lot of work
    def work(x):
        ...
        return result

    # Nonparallel code
    results = map(work, data)

    # Parallel implementation
    with ProcessPoolExecutor() as pool:
        results = pool.map(work, data)

另外，你可以使用 ``pool.submit()`` 来手动的提交单个任务：

.. code-block:: python

    # Some function
    def work(x):
        ...
        return result

    with ProcessPoolExecutor() as pool:
        ...
        # Example of submitting work to the pool
        future_result = pool.submit(work, arg)

        # Obtaining the result (blocks until done)
        r = future_result.result()
        ...

如果你手动提交一个任务，结果是一个 ``Future`` 实例。
要获取最终结果，你需要调用它的 ``result()`` 方法。
它会阻塞进程直到结果被返回来。

如果不想阻塞，你还可以使用一个回调函数，例如：

.. code-block:: python

    def when_done(r):
        print('Got:', r.result())

    with ProcessPoolExecutor() as pool:
         future_result = pool.submit(work, arg)
         future_result.add_done_callback(when_done)

回调函数接受一个 ``Future`` 实例，被用来获取最终的结果（比如通过调用它的result()方法）。
尽管处理池很容易使用，在设计大程序的时候还是有很多需要注意的地方，如下几点：

• 这种并行处理技术只适用于那些可以被分解为互相独立部分的问题。

• 被提交的任务必须是简单函数形式。对于方法、闭包和其他类型的并行执行还不支持。

• 函数参数和返回值必须兼容pickle，因为要使用到进程间的通信，所有解释器之间的交换数据必须被序列化

• 被提交的任务函数不应保留状态或有副作用。除了打印日志之类简单的事情，
一旦启动你不能控制子进程的任何行为，因此最好保持简单和纯洁——函数不要去修改环境。

• 在Unix上进程池通过调用 ``fork()`` 系统调用被创建，
它会克隆Python解释器，包括fork时的所有程序状态。
而在Windows上，克隆解释器时不会克隆状态。
实际的fork操作会在第一次调用 ``pool.map()`` 或 ``pool.submit()`` 后发生。

• 当你混合使用进程池和多线程的时候要特别小心。
你应该在创建任何线程之前先创建并激活进程池（比如在程序启动的main线程中创建进程池）。
============================
12.9 Python的全局锁问题
============================

----------
问题
----------
你已经听说过全局解释器锁GIL，担心它会影响到多线程程序的执行性能。

----------
解决方案
----------
尽管Python完全支持多线程编程，
但是解释器的C语言实现部分在完全并行执行时并不是线程安全的。
实际上，解释器被一个全局解释器锁保护着，它确保任何时候都只有一个Python线程执行。
GIL最大的问题就是Python的多线程程序并不能利用多核CPU的优势
（比如一个使用了多个线程的计算密集型程序只会在一个单CPU上面运行）。

在讨论普通的GIL之前，有一点要强调的是GIL只会影响到那些严重依赖CPU的程序（比如计算型的）。
如果你的程序大部分只会涉及到I/O，比如网络交互，那么使用多线程就很合适，
因为它们大部分时间都在等待。实际上，你完全可以放心的创建几千个Python线程，
现代操作系统运行这么多线程没有任何压力，没啥可担心的。

而对于依赖CPU的程序，你需要弄清楚执行的计算的特点。
例如，优化底层算法要比使用多线程运行快得多。
类似的，由于Python是解释执行的，如果你将那些性能瓶颈代码移到一个C语言扩展模块中，
速度也会提升的很快。如果你要操作数组，那么使用NumPy这样的扩展会非常的高效。
最后，你还可以考虑下其他可选实现方案，比如PyPy，它通过一个JIT编译器来优化执行效率
（不过在写这本书的时候它还不能支持Python 3）。

还有一点要注意的是，线程不是专门用来优化性能的。
一个CPU依赖型程序可能会使用线程来管理一个图形用户界面、一个网络连接或其他服务。
这时候，GIL会产生一些问题，因为如果一个线程长期持有GIL的话会导致其他非CPU型线程一直等待。
事实上，一个写的不好的C语言扩展会导致这个问题更加严重，
尽管代码的计算部分会比之前运行的更快些。

说了这么多，现在想说的是我们有两种策略来解决GIL的缺点。
首先，如果你完全工作于Python环境中，你可以使用 ``multiprocessing`` 模块来创建一个进程池，
并像协同处理器一样的使用它。例如，假如你有如下的线程代码：

.. code-block:: python

    # Performs a large calculation (CPU bound)
    def some_work(args):
        ...
        return result

    # A thread that calls the above function
    def some_thread():
        while True:
            ...
            r = some_work(args)
        ...

修改代码，使用进程池：

.. code-block:: python

    # Processing pool (see below for initiazation)
    pool = None

    # Performs a large calculation (CPU bound)
    def some_work(args):
        ...
        return result

    # A thread that calls the above function
    def some_thread():
        while True:
            ...
            r = pool.apply(some_work, (args))
            ...

    # Initiaze the pool
    if __name__ == '__main__':
        import multiprocessing
        pool = multiprocessing.Pool()

这个通过使用一个技巧利用进程池解决了GIL的问题。
当一个线程想要执行CPU密集型工作时，会将任务发给进程池。
然后进程池会在另外一个进程中启动一个单独的Python解释器来工作。
当线程等待结果的时候会释放GIL。
并且，由于计算任务在单独解释器中执行，那么就不会受限于GIL了。
在一个多核系统上面，你会发现这个技术可以让你很好的利用多CPU的优势。

另外一个解决GIL的策略是使用C扩展编程技术。
主要思想是将计算密集型任务转移给C，跟Python独立，在工作的时候在C代码中释放GIL。
这可以通过在C代码中插入下面这样的特殊宏来完成：

::

    #include "Python.h"
    ...

    PyObject *pyfunc(PyObject *self, PyObject *args) {
       ...
       Py_BEGIN_ALLOW_THREADS
       // Threaded C code
       ...
       Py_END_ALLOW_THREADS
       ...
    }

如果你使用其他工具访问C语言，比如对于Cython的ctypes库，你不需要做任何事。
例如，ctypes在调用C时会自动释放GIL。

----------
讨论
----------
许多程序员在面对线程性能问题的时候，马上就会怪罪GIL，什么都是它的问题。
其实这样子太不厚道也太天真了点。
作为一个真实的例子，在多线程的网络编程中神秘的 ``stalls``
可能是因为其他原因比如一个DNS查找延时，而跟GIL毫无关系。
最后你真的需要先去搞懂你的代码是否真的被GIL影响到。
同时还要明白GIL大部分都应该只关注CPU的处理而不是I/O.

如果你准备使用一个处理器池，注意的是这样做涉及到数据序列化和在不同Python解释器通信。
被执行的操作需要放在一个通过def语句定义的Python函数中，不能是lambda、闭包可调用实例等，
并且函数参数和返回值必须要兼容pickle。
同样，要执行的任务量必须足够大以弥补额外的通信开销。

另外一个难点是当混合使用线程和进程池的时候会让你很头疼。
如果你要同时使用两者，最好在程序启动时，创建任何线程之前先创建一个单例的进程池。
然后线程使用同样的进程池来进行它们的计算密集型工作。

C扩展最重要的特征是它们和Python解释器是保持独立的。
也就是说，如果你准备将Python中的任务分配到C中去执行，
你需要确保C代码的操作跟Python保持独立，
这就意味着不要使用Python数据结构以及不要调用Python的C API。
另外一个就是你要确保C扩展所做的工作是足够的，值得你这样做。
也就是说C扩展担负起了大量的计算任务，而不是少数几个计算。

这些解决GIL的方案并不能适用于所有问题。
例如，某些类型的应用程序如果被分解为多个进程处理的话并不能很好的工作，
也不能将它的部分代码改成C语言执行。
对于这些应用程序，你就要自己需求解决方案了
（比如多进程访问共享内存区，多解析器运行于同一个进程等）。
或者，你还可以考虑下其他的解释器实现，比如PyPy。

了解更多关于在C扩展中释放GIL，请参考15.7和15.10小节。
============================
12.10 定义一个Actor任务
============================

----------
问题
----------
你想定义跟actor模式中类似“actors”角色的任务

----------
解决方案
----------
actor模式是一种最古老的也是最简单的并行和分布式计算解决方案。
事实上，它天生的简单性是它如此受欢迎的重要原因之一。
简单来讲，一个actor就是一个并发执行的任务，只是简单的执行发送给它的消息任务。
响应这些消息时，它可能还会给其他actor发送更进一步的消息。
actor之间的通信是单向和异步的。因此，消息发送者不知道消息是什么时候被发送，
也不会接收到一个消息已被处理的回应或通知。

结合使用一个线程和一个队列可以很容易的定义actor，例如：

.. code-block:: python

    from queue import Queue
    from threading import Thread, Event

    # Sentinel used for shutdown
    class ActorExit(Exception):
        pass

    class Actor:
        def __init__(self):
            self._mailbox = Queue()

        def send(self, msg):
            '''
            Send a message to the actor
            '''
            self._mailbox.put(msg)

        def recv(self):
            '''
            Receive an incoming message
            '''
            msg = self._mailbox.get()
            if msg is ActorExit:
                raise ActorExit()
            return msg

        def close(self):
            '''
            Close the actor, thus shutting it down
            '''
            self.send(ActorExit)

        def start(self):
            '''
            Start concurrent execution
            '''
            self._terminated = Event()
            t = Thread(target=self._bootstrap)

            t.daemon = True
            t.start()

        def _bootstrap(self):
            try:
                self.run()
            except ActorExit:
                pass
            finally:
                self._terminated.set()

        def join(self):
            self._terminated.wait()

        def run(self):
            '''
            Run method to be implemented by the user
            '''
            while True:
                msg = self.recv()

    # Sample ActorTask
    class PrintActor(Actor):
        def run(self):
            while True:
                msg = self.recv()
                print('Got:', msg)

    # Sample use
    p = PrintActor()
    p.start()
    p.send('Hello')
    p.send('World')
    p.close()
    p.join()

这个例子中，你使用actor实例的 ``send()`` 方法发送消息给它们。
其机制是，这个方法会将消息放入一个队里中，
然后将其转交给处理被接受消息的一个内部线程。
``close()`` 方法通过在队列中放入一个特殊的哨兵值（ActorExit）来关闭这个actor。
用户可以通过继承Actor并定义实现自己处理逻辑run()方法来定义新的actor。
``ActorExit`` 异常的使用就是用户自定义代码可以在需要的时候来捕获终止请求
（异常被get()方法抛出并传播出去）。

如果你放宽对于同步和异步消息发送的要求，
类actor对象还可以通过生成器来简化定义。例如：

.. code-block:: python

    def print_actor():
        while True:

            try:
                msg = yield      # Get a message
                print('Got:', msg)
            except GeneratorExit:
                print('Actor terminating')

    # Sample use
    p = print_actor()
    next(p)     # Advance to the yield (ready to receive)
    p.send('Hello')
    p.send('World')
    p.close()

----------
讨论
----------
actor模式的魅力就在于它的简单性。
实际上，这里仅仅只有一个核心操作 ``send()`` .
甚至，对于在基于actor系统中的“消息”的泛化概念可以已多种方式被扩展。
例如，你可以以元组形式传递标签消息，让actor执行不同的操作，如下：

.. code-block:: python

    class TaggedActor(Actor):
        def run(self):
            while True:
                 tag, *payload = self.recv()
                 getattr(self,'do_'+tag)(*payload)

        # Methods correponding to different message tags
        def do_A(self, x):
            print('Running A', x)

        def do_B(self, x, y):
            print('Running B', x, y)

    # Example
    a = TaggedActor()
    a.start()
    a.send(('A', 1))      # Invokes do_A(1)
    a.send(('B', 2, 3))   # Invokes do_B(2,3)

作为另外一个例子，下面的actor允许在一个工作者中运行任意的函数，
并且通过一个特殊的Result对象返回结果：

.. code-block:: python

    from threading import Event
    class Result:
        def __init__(self):
            self._evt = Event()
            self._result = None

        def set_result(self, value):
            self._result = value

            self._evt.set()

        def result(self):
            self._evt.wait()
            return self._result

    class Worker(Actor):
        def submit(self, func, *args, **kwargs):
            r = Result()
            self.send((func, args, kwargs, r))
            return r

        def run(self):
            while True:
                func, args, kwargs, r = self.recv()
                r.set_result(func(*args, **kwargs))

    # Example use
    worker = Worker()
    worker.start()
    r = worker.submit(pow, 2, 3)
    print(r.result())

最后，“发送”一个任务消息的概念可以被扩展到多进程甚至是大型分布式系统中去。
例如，一个类actor对象的 ``send()`` 方法可以被编程让它能在一个套接字连接上传输数据
或通过某些消息中间件（比如AMQP、ZMQ等）来发送。
============================
12.11 实现消息发布/订阅模型
============================

----------
问题
----------
你有一个基于线程通信的程序，想让它们实现发布/订阅模式的消息通信。

----------
解决方案
----------
要实现发布/订阅的消息通信模式，
你通常要引入一个单独的“交换机”或“网关”对象作为所有消息的中介。
也就是说，不直接将消息从一个任务发送到另一个，而是将其发送给交换机，
然后由交换机将它发送给一个或多个被关联任务。下面是一个非常简单的交换机实现例子：

.. code-block:: python

    from collections import defaultdict

    class Exchange:
        def __init__(self):
            self._subscribers = set()

        def attach(self, task):
            self._subscribers.add(task)

        def detach(self, task):
            self._subscribers.remove(task)

        def send(self, msg):
            for subscriber in self._subscribers:
                subscriber.send(msg)

    # Dictionary of all created exchanges
    _exchanges = defaultdict(Exchange)

    # Return the Exchange instance associated with a given name
    def get_exchange(name):
        return _exchanges[name]

一个交换机就是一个普通对象，负责维护一个活跃的订阅者集合，并为绑定、解绑和发送消息提供相应的方法。
每个交换机通过一个名称定位，``get_exchange()`` 通过给定一个名称返回相应的 ``Exchange`` 实例。

下面是一个简单例子，演示了如何使用一个交换机：

.. code-block:: python

    # Example of a task.  Any object with a send() method

    class Task:
        ...
        def send(self, msg):
            ...

    task_a = Task()
    task_b = Task()

    # Example of getting an exchange
    exc = get_exchange('name')

    # Examples of subscribing tasks to it
    exc.attach(task_a)
    exc.attach(task_b)

    # Example of sending messages
    exc.send('msg1')
    exc.send('msg2')

    # Example of unsubscribing
    exc.detach(task_a)
    exc.detach(task_b)

尽管对于这个问题有很多的变种，不过万变不离其宗。
消息会被发送给一个交换机，然后交换机会将它们发送给被绑定的订阅者。

----------
讨论
----------
通过队列发送消息的任务或线程的模式很容易被实现并且也非常普遍。
不过，使用发布/订阅模式的好处更加明显。

首先，使用一个交换机可以简化大部分涉及到线程通信的工作。
无需去写通过多进程模块来操作多个线程，你只需要使用这个交换机来连接它们。
某种程度上，这个就跟日志模块的工作原理类似。
实际上，它可以轻松的解耦程序中多个任务。

其次，交换机广播消息给多个订阅者的能力带来了一个全新的通信模式。
例如，你可以使用多任务系统、广播或扇出。
你还可以通过以普通订阅者身份绑定来构建调试和诊断工具。
例如，下面是一个简单的诊断类，可以显示被发送的消息：

.. code-block:: python

    class DisplayMessages:
        def __init__(self):
            self.count = 0
        def send(self, msg):
            self.count += 1
            print('msg[{}]: {!r}'.format(self.count, msg))

    exc = get_exchange('name')
    d = DisplayMessages()
    exc.attach(d)

最后，该实现的一个重要特点是它能兼容多个“task-like”对象。
例如，消息接受者可以是actor（12.10小节介绍）、协程、网络连接或任何实现了正确的 ``send()`` 方法的东西。

关于交换机的一个可能问题是对于订阅者的正确绑定和解绑。
为了正确的管理资源，每一个绑定的订阅者必须最终要解绑。
在代码中通常会是像下面这样的模式：

.. code-block:: python

    exc = get_exchange('name')
    exc.attach(some_task)
    try:
        ...
    finally:
        exc.detach(some_task)

某种意义上，这个和使用文件、锁和类似对象很像。
通常很容易会忘记最后的 ``detach()`` 步骤。
为了简化这个，你可以考虑使用上下文管理器协议。
例如，在交换机对象上增加一个 ``subscribe()`` 方法，如下：

.. code-block:: python

    from contextlib import contextmanager
    from collections import defaultdict

    class Exchange:
        def __init__(self):
            self._subscribers = set()

        def attach(self, task):
            self._subscribers.add(task)

        def detach(self, task):
            self._subscribers.remove(task)

        @contextmanager
        def subscribe(self, *tasks):
            for task in tasks:
                self.attach(task)
            try:
                yield
            finally:
                for task in tasks:
                    self.detach(task)

        def send(self, msg):
            for subscriber in self._subscribers:
                subscriber.send(msg)

    # Dictionary of all created exchanges
    _exchanges = defaultdict(Exchange)

    # Return the Exchange instance associated with a given name
    def get_exchange(name):
        return _exchanges[name]

    # Example of using the subscribe() method
    exc = get_exchange('name')
    with exc.subscribe(task_a, task_b):
         ...
         exc.send('msg1')
         exc.send('msg2')
         ...

    # task_a and task_b detached here

最后还应该注意的是关于交换机的思想有很多种的扩展实现。
例如，交换机可以实现一整个消息通道集合或提供交换机名称的模式匹配规则。
交换机还可以被扩展到分布式计算程序中（比如，将消息路由到不同机器上面的任务中去）。

============================
12.12 使用生成器代替线程
============================

----------
问题
----------
你想使用生成器（协程）替代系统线程来实现并发。这个有时又被称为用户级线程或绿色线程。

----------
解决方案
----------
要使用生成器实现自己的并发，你首先要对生成器函数和 ``yield`` 语句有深刻理解。
``yield`` 语句会让一个生成器挂起它的执行，这样就可以编写一个调度器，
将生成器当做某种“任务”并使用任务协作切换来替换它们的执行。
要演示这种思想，考虑下面两个使用简单的 ``yield`` 语句的生成器函数：

.. code-block:: python

    # Two simple generator functions
    def countdown(n):
        while n > 0:
            print('T-minus', n)
            yield
            n -= 1
        print('Blastoff!')

    def countup(n):
        x = 0
        while x < n:
            print('Counting up', x)
            yield
            x += 1

这些函数在内部使用yield语句，下面是一个实现了简单任务调度器的代码：

.. code-block:: python

    from collections import deque

    class TaskScheduler:
        def __init__(self):
            self._task_queue = deque()

        def new_task(self, task):
            '''
            Admit a newly started task to the scheduler

            '''
            self._task_queue.append(task)

        def run(self):
            '''
            Run until there are no more tasks
            '''
            while self._task_queue:
                task = self._task_queue.popleft()
                try:
                    # Run until the next yield statement
                    next(task)
                    self._task_queue.append(task)
                except StopIteration:
                    # Generator is no longer executing
                    pass

    # Example use
    sched = TaskScheduler()
    sched.new_task(countdown(10))
    sched.new_task(countdown(5))
    sched.new_task(countup(15))
    sched.run()

``TaskScheduler`` 类在一个循环中运行生成器集合——每个都运行到碰到yield语句为止。
运行这个例子，输出如下：

::

    T-minus 10
    T-minus 5
    Counting up 0
    T-minus 9
    T-minus 4
    Counting up 1
    T-minus 8
    T-minus 3
    Counting up 2
    T-minus 7
    T-minus 2
    ...

到此为止，我们实际上已经实现了一个“操作系统”的最小核心部分。
生成器函数就是认为，而yield语句是任务挂起的信号。
调度器循环检查任务列表直到没有任务要执行为止。

实际上，你可能想要使用生成器来实现简单的并发。
那么，在实现actor或网络服务器的时候你可以使用生成器来替代线程的使用。

下面的代码演示了使用生成器来实现一个不依赖线程的actor：

.. code-block:: python

    from collections import deque

    class ActorScheduler:
        def __init__(self):
            self._actors = { }          # Mapping of names to actors
            self._msg_queue = deque()   # Message queue

        def new_actor(self, name, actor):
            '''
            Admit a newly started actor to the scheduler and give it a name
            '''
            self._msg_queue.append((actor,None))
            self._actors[name] = actor

        def send(self, name, msg):
            '''
            Send a message to a named actor
            '''
            actor = self._actors.get(name)
            if actor:
                self._msg_queue.append((actor,msg))

        def run(self):
            '''
            Run as long as there are pending messages.
            '''
            while self._msg_queue:
                actor, msg = self._msg_queue.popleft()
                try:
                     actor.send(msg)
                except StopIteration:
                     pass

    # Example use
    if __name__ == '__main__':
        def printer():
            while True:
                msg = yield
                print('Got:', msg)

        def counter(sched):
            while True:
                # Receive the current count
                n = yield
                if n == 0:
                    break
                # Send to the printer task
                sched.send('printer', n)
                # Send the next count to the counter task (recursive)

                sched.send('counter', n-1)

        sched = ActorScheduler()
        # Create the initial actors
        sched.new_actor('printer', printer())
        sched.new_actor('counter', counter(sched))

        # Send an initial message to the counter to initiate
        sched.send('counter', 10000)
        sched.run()

完全弄懂这段代码需要更深入的学习，但是关键点在于收集消息的队列。
本质上，调度器在有需要发送的消息时会一直运行着。
计数生成器会给自己发送消息并在一个递归循环中结束。

下面是一个更加高级的例子，演示了使用生成器来实现一个并发网络应用程序：

.. code-block:: python

    from collections import deque
    from select import select

    # This class represents a generic yield event in the scheduler
    class YieldEvent:
        def handle_yield(self, sched, task):
            pass
        def handle_resume(self, sched, task):
            pass

    # Task Scheduler
    class Scheduler:
        def __init__(self):
            self._numtasks = 0       # Total num of tasks
            self._ready = deque()    # Tasks ready to run
            self._read_waiting = {}  # Tasks waiting to read
            self._write_waiting = {} # Tasks waiting to write

        # Poll for I/O events and restart waiting tasks
        def _iopoll(self):
            rset,wset,eset = select(self._read_waiting,
                                    self._write_waiting,[])
            for r in rset:
                evt, task = self._read_waiting.pop(r)
                evt.handle_resume(self, task)
            for w in wset:
                evt, task = self._write_waiting.pop(w)
                evt.handle_resume(self, task)

        def new(self,task):
            '''
            Add a newly started task to the scheduler
            '''

            self._ready.append((task, None))
            self._numtasks += 1

        def add_ready(self, task, msg=None):
            '''
            Append an already started task to the ready queue.
            msg is what to send into the task when it resumes.
            '''
            self._ready.append((task, msg))

        # Add a task to the reading set
        def _read_wait(self, fileno, evt, task):
            self._read_waiting[fileno] = (evt, task)

        # Add a task to the write set
        def _write_wait(self, fileno, evt, task):
            self._write_waiting[fileno] = (evt, task)

        def run(self):
            '''
            Run the task scheduler until there are no tasks
            '''
            while self._numtasks:
                 if not self._ready:
                      self._iopoll()
                 task, msg = self._ready.popleft()
                 try:
                     # Run the coroutine to the next yield
                     r = task.send(msg)
                     if isinstance(r, YieldEvent):
                         r.handle_yield(self, task)
                     else:
                         raise RuntimeError('unrecognized yield event')
                 except StopIteration:
                     self._numtasks -= 1

    # Example implementation of coroutine-based socket I/O
    class ReadSocket(YieldEvent):
        def __init__(self, sock, nbytes):
            self.sock = sock
            self.nbytes = nbytes
        def handle_yield(self, sched, task):
            sched._read_wait(self.sock.fileno(), self, task)
        def handle_resume(self, sched, task):
            data = self.sock.recv(self.nbytes)
            sched.add_ready(task, data)

    class WriteSocket(YieldEvent):
        def __init__(self, sock, data):
            self.sock = sock
            self.data = data
        def handle_yield(self, sched, task):

            sched._write_wait(self.sock.fileno(), self, task)
        def handle_resume(self, sched, task):
            nsent = self.sock.send(self.data)
            sched.add_ready(task, nsent)

    class AcceptSocket(YieldEvent):
        def __init__(self, sock):
            self.sock = sock
        def handle_yield(self, sched, task):
            sched._read_wait(self.sock.fileno(), self, task)
        def handle_resume(self, sched, task):
            r = self.sock.accept()
            sched.add_ready(task, r)

    # Wrapper around a socket object for use with yield
    class Socket(object):
        def __init__(self, sock):
            self._sock = sock
        def recv(self, maxbytes):
            return ReadSocket(self._sock, maxbytes)
        def send(self, data):
            return WriteSocket(self._sock, data)
        def accept(self):
            return AcceptSocket(self._sock)
        def __getattr__(self, name):
            return getattr(self._sock, name)

    if __name__ == '__main__':
        from socket import socket, AF_INET, SOCK_STREAM
        import time

        # Example of a function involving generators.  This should
        # be called using line = yield from readline(sock)
        def readline(sock):
            chars = []
            while True:
                c = yield sock.recv(1)
                if not c:
                    break
                chars.append(c)
                if c == b'\n':
                    break
            return b''.join(chars)

        # Echo server using generators
        class EchoServer:
            def __init__(self,addr,sched):
                self.sched = sched
                sched.new(self.server_loop(addr))

            def server_loop(self,addr):
                s = Socket(socket(AF_INET,SOCK_STREAM))

                s.bind(addr)
                s.listen(5)
                while True:
                    c,a = yield s.accept()
                    print('Got connection from ', a)
                    self.sched.new(self.client_handler(Socket(c)))

            def client_handler(self,client):
                while True:
                    line = yield from readline(client)
                    if not line:
                        break
                    line = b'GOT:' + line
                    while line:
                        nsent = yield client.send(line)
                        line = line[nsent:]
                client.close()
                print('Client closed')

        sched = Scheduler()
        EchoServer(('',16000),sched)
        sched.run()

这段代码有点复杂。不过，它实现了一个小型的操作系统。
有一个就绪的任务队列，并且还有因I/O休眠的任务等待区域。
还有很多调度器负责在就绪队列和I/O等待区域之间移动任务。

----------
讨论
----------
在构建基于生成器的并发框架时，通常会使用更常见的yield形式：

.. code-block:: python

    def some_generator():
        ...
        result = yield data
        ...

使用这种形式的yield语句的函数通常被称为“协程”。
通过调度器，yield语句在一个循环中被处理，如下：

.. code-block:: python

    f = some_generator()

    # Initial result. Is None to start since nothing has been computed
    result = None
    while True:
        try:
            data = f.send(result)
            result = ... do some calculation ...
        except StopIteration:
            break

这里的逻辑稍微有点复杂。不过，被传给 ``send()`` 的值定义了在yield语句醒来时的返回值。
因此，如果一个yield准备在对之前yield数据的回应中返回结果时，会在下一次 ``send()`` 操作返回。
如果一个生成器函数刚开始运行，发送一个None值会让它排在第一个yield语句前面。

除了发送值外，还可以在一个生成器上面执行一个 ``close()`` 方法。
它会导致在执行yield语句时抛出一个 ``GeneratorExit`` 异常，从而终止执行。
如果进一步设计，一个生成器可以捕获这个异常并执行清理操作。
同样还可以使用生成器的 ``throw()`` 方法在yield语句执行时生成一个任意的执行指令。
一个任务调度器可利用它来在运行的生成器中处理错误。

最后一个例子中使用的 ``yield from`` 语句被用来实现协程，可以被其它生成器作为子程序或过程来调用。
本质上就是将控制权透明的传输给新的函数。
不像普通的生成器，一个使用 ``yield from`` 被调用的函数可以返回一个作为 ``yield from`` 语句结果的值。
关于 ``yield from`` 的更多信息可以在 `PEP 380 <http://www.python.org/dev/peps/pep-0380>`_ 中找到。

最后，如果使用生成器编程，要提醒你的是它还是有很多缺点的。
特别是，你得不到任何线程可以提供的好处。例如，如果你执行CPU依赖或I/O阻塞程序，
它会将整个任务挂起知道操作完成。为了解决这个问题，
你只能选择将操作委派给另外一个可以独立运行的线程或进程。
另外一个限制是大部分Python库并不能很好的兼容基于生成器的线程。
如果你选择这个方案，你会发现你需要自己改写很多标准库函数。
作为本节提到的协程和相关技术的一个基础背景，可以查看 `PEP 342 <http://www.python.org/dev/peps/pep-0342>`_
和 `“协程和并发的一门有趣课程” <http://www.dabeaz.com/coroutines>`_

PEP 3156 同样有一个关于使用协程的异步I/O模型。
特别的，你不可能自己去实现一个底层的协程调度器。
不过，关于协程的思想是很多流行库的基础，
包括 `gevent <http://www.gevent.org/>`_,
`greenlet <http://pypi.python.org/pypi/greenlet>`_,
`Stackless Python <http://www.stackless.com/>`_ 以及其他类似工程。

============================
12.13 多个线程队列轮询
============================

----------
问题
----------
你有一个线程队列集合，想为到来的元素轮询它们，
就跟你为一个客户端请求去轮询一个网络连接集合的方式一样。

----------
解决方案
----------
对于轮询问题的一个常见解决方案中有个很少有人知道的技巧，包含了一个隐藏的回路网络连接。
本质上讲其思想就是：对于每个你想要轮询的队列，你创建一对连接的套接字。
然后你在其中一个套接字上面编写代码来标识存在的数据，
另外一个套接字被传给 ``select()`` 或类似的一个轮询数据到达的函数。下面的例子演示了这个思想：

.. code-block:: python

    import queue
    import socket
    import os

    class PollableQueue(queue.Queue):
        def __init__(self):
            super().__init__()
            # Create a pair of connected sockets
            if os.name == 'posix':
                self._putsocket, self._getsocket = socket.socketpair()
            else:
                # Compatibility on non-POSIX systems
                server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                server.bind(('127.0.0.1', 0))
                server.listen(1)
                self._putsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                self._putsocket.connect(server.getsockname())
                self._getsocket, _ = server.accept()
                server.close()

        def fileno(self):
            return self._getsocket.fileno()

        def put(self, item):
            super().put(item)
            self._putsocket.send(b'x')

        def get(self):
            self._getsocket.recv(1)
            return super().get()

在这个代码中，一个新的 ``Queue`` 实例类型被定义，底层是一个被连接套接字对。
在Unix机器上的 ``socketpair()`` 函数能轻松的创建这样的套接字。
在Windows上面，你必须使用类似代码来模拟它。
然后定义普通的 ``get()`` 和 ``put()`` 方法在这些套接字上面来执行I/O操作。
``put()`` 方法再将数据放入队列后会写一个单字节到某个套接字中去。
而 ``get()`` 方法在从队列中移除一个元素时会从另外一个套接字中读取到这个单字节数据。

``fileno()`` 方法使用一个函数比如 ``select()`` 来让这个队列可以被轮询。
它仅仅只是暴露了底层被 ``get()`` 函数使用到的socket的文件描述符而已。

下面是一个例子，定义了一个为到来的元素监控多个队列的消费者：

.. code-block:: python

    import select
    import threading

    def consumer(queues):
        '''
        Consumer that reads data on multiple queues simultaneously
        '''
        while True:
            can_read, _, _ = select.select(queues,[],[])
            for r in can_read:
                item = r.get()
                print('Got:', item)

    q1 = PollableQueue()
    q2 = PollableQueue()
    q3 = PollableQueue()
    t = threading.Thread(target=consumer, args=([q1,q2,q3],))
    t.daemon = True
    t.start()

    # Feed data to the queues
    q1.put(1)
    q2.put(10)
    q3.put('hello')
    q2.put(15)
    ...

如果你试着运行它，你会发现这个消费者会接受到所有的被放入的元素，不管元素被放进了哪个队列中。

----------
讨论
----------
对于轮询非类文件对象，比如队列通常都是比较棘手的问题。
例如，如果你不使用上面的套接字技术，
你唯一的选择就是编写代码来循环遍历这些队列并使用一个定时器。像下面这样：

.. code-block:: python

    import time
    def consumer(queues):
        while True:
            for q in queues:
                if not q.empty():
                    item = q.get()
                    print('Got:', item)

            # Sleep briefly to avoid 100% CPU
            time.sleep(0.01)

这样做其实不合理，还会引入其他的性能问题。
例如，如果新的数据被加入到一个队列中，至少要花10毫秒才能被发现。
如果你之前的轮询还要去轮询其他对象，比如网络套接字那还会有更多问题。
例如，如果你想同时轮询套接字和队列，你可能要像下面这样使用：

.. code-block:: python

    import select

    def event_loop(sockets, queues):
        while True:
            # polling with a timeout
            can_read, _, _ = select.select(sockets, [], [], 0.01)
            for r in can_read:
                handle_read(r)
            for q in queues:
                if not q.empty():
                    item = q.get()
                    print('Got:', item)

这个方案通过将队列和套接字等同对待来解决了大部分的问题。
一个单独的 ``select()`` 调用可被同时用来轮询。
使用超时或其他基于时间的机制来执行周期性检查并没有必要。
甚至，如果数据被加入到一个队列，消费者几乎可以实时的被通知。
尽管会有一点点底层的I/O损耗，使用它通常会获得更好的响应时间并简化编程。

============================
12.14 在Unix系统上面启动守护进程
============================

----------
问题
----------
你想编写一个作为一个在Unix或类Unix系统上面运行的守护进程运行的程序。

----------
解决方案
----------
创建一个正确的守护进程需要一个精确的系统调用序列以及对于细节的控制。
下面的代码展示了怎样定义一个守护进程，可以启动后很容易的停止它。

.. code-block:: python

    #!/usr/bin/env python3
    # daemon.py

    import os
    import sys

    import atexit
    import signal

    def daemonize(pidfile, *, stdin='/dev/null',
                              stdout='/dev/null',
                              stderr='/dev/null'):

        if os.path.exists(pidfile):
            raise RuntimeError('Already running')

        # First fork (detaches from parent)
        try:
            if os.fork() > 0:
                raise SystemExit(0)   # Parent exit
        except OSError as e:
            raise RuntimeError('fork #1 failed.')

        os.chdir('/')
        os.umask(0)
        os.setsid()
        # Second fork (relinquish session leadership)
        try:
            if os.fork() > 0:
                raise SystemExit(0)
        except OSError as e:
            raise RuntimeError('fork #2 failed.')

        # Flush I/O buffers
        sys.stdout.flush()
        sys.stderr.flush()

        # Replace file descriptors for stdin, stdout, and stderr
        with open(stdin, 'rb', 0) as f:
            os.dup2(f.fileno(), sys.stdin.fileno())
        with open(stdout, 'ab', 0) as f:
            os.dup2(f.fileno(), sys.stdout.fileno())
        with open(stderr, 'ab', 0) as f:
            os.dup2(f.fileno(), sys.stderr.fileno())

        # Write the PID file
        with open(pidfile,'w') as f:
            print(os.getpid(),file=f)

        # Arrange to have the PID file removed on exit/signal
        atexit.register(lambda: os.remove(pidfile))

        # Signal handler for termination (required)
        def sigterm_handler(signo, frame):
            raise SystemExit(1)

        signal.signal(signal.SIGTERM, sigterm_handler)

    def main():
        import time
        sys.stdout.write('Daemon started with pid {}\n'.format(os.getpid()))
        while True:
            sys.stdout.write('Daemon Alive! {}\n'.format(time.ctime()))
            time.sleep(10)

    if __name__ == '__main__':
        PIDFILE = '/tmp/daemon.pid'

        if len(sys.argv) != 2:
            print('Usage: {} [start|stop]'.format(sys.argv[0]), file=sys.stderr)
            raise SystemExit(1)

        if sys.argv[1] == 'start':
            try:
                daemonize(PIDFILE,
                          stdout='/tmp/daemon.log',
                          stderr='/tmp/dameon.log')
            except RuntimeError as e:
                print(e, file=sys.stderr)
                raise SystemExit(1)

            main()

        elif sys.argv[1] == 'stop':
            if os.path.exists(PIDFILE):
                with open(PIDFILE) as f:
                    os.kill(int(f.read()), signal.SIGTERM)
            else:
                print('Not running', file=sys.stderr)
                raise SystemExit(1)

        else:
            print('Unknown command {!r}'.format(sys.argv[1]), file=sys.stderr)
            raise SystemExit(1)

要启动这个守护进程，用户需要使用如下的命令：

::

    bash % daemon.py start
    bash % cat /tmp/daemon.pid
    2882
    bash % tail -f /tmp/daemon.log
    Daemon started with pid 2882
    Daemon Alive! Fri Oct 12 13:45:37 2012
    Daemon Alive! Fri Oct 12 13:45:47 2012
    ...

守护进程可以完全在后台运行，因此这个命令会立即返回。
不过，你可以像上面那样查看与它相关的pid文件和日志。要停止这个守护进程，使用：

::

    bash % daemon.py stop
    bash %

----------
讨论
----------
本节定义了一个函数 ``daemonize()`` ，在程序启动时被调用使得程序以一个守护进程来运行。
``daemonize()`` 函数只接受关键字参数，这样的话可选参数在被使用时就更清晰了。
它会强制用户像下面这样使用它：

::

    daemonize('daemon.pid',
              stdin='/dev/null,
              stdout='/tmp/daemon.log',
              stderr='/tmp/daemon.log')

而不是像下面这样含糊不清的调用：
::

    # Illegal. Must use keyword arguments
    daemonize('daemon.pid',
              '/dev/null', '/tmp/daemon.log','/tmp/daemon.log')

创建一个守护进程的步骤看上去不是很易懂，但是大体思想是这样的，
首先，一个守护进程必须要从父进程中脱离。
这是由 ``os.fork()`` 操作来完成的，并立即被父进程终止。

在子进程变成孤儿后，调用 ``os.setsid()`` 创建了一个全新的进程会话，并设置子进程为首领。
它会设置这个子进程为新的进程组的首领，并确保不会再有控制终端。
如果这些听上去太魔幻，因为它需要将守护进程同终端分离开并确保信号机制对它不起作用。
调用 ``os.chdir()`` 和 ``os.umask(0)`` 改变了当前工作目录并重置文件权限掩码。
修改目录通常是个好主意，因为这样可以使得它不再工作在被启动时的目录。

另外一个调用 ``os.fork()`` 在这里更加神秘点。
这一步使得守护进程失去了获取新的控制终端的能力并且让它更加独立
（本质上，该daemon放弃了它的会话首领低位，因此再也没有权限去打开控制终端了）。
尽管你可以忽略这一步，但是最好不要这么做。

一旦守护进程被正确的分离，它会重新初始化标准I/O流指向用户指定的文件。
这一部分有点难懂。跟标准I/O流相关的文件对象的引用在解释器中多个地方被找到
（sys.stdout, sys.__stdout__等）。
仅仅简单的关闭 ``sys.stdout`` 并重新指定它是行不通的，
因为没办法知道它是否全部都是用的是 ``sys.stdout`` 。
这里，我们打开了一个单独的文件对象，并调用 ``os.dup2()`` ，
用它来代替被 ``sys.stdout`` 使用的文件描述符。
这样，``sys.stdout`` 使用的原始文件会被关闭并由新的来替换。
还要强调的是任何用于文件编码或文本处理的标准I/O流还会保留原状。

守护进程的一个通常实践是在一个文件中写入进程ID，可以被其他程序后面使用到。
``daemonize()`` 函数的最后部分写了这个文件，但是在程序终止时删除了它。
``atexit.register()`` 函数注册了一个函数在Python解释器终止时执行。
一个对于SIGTERM的信号处理器的定义同样需要被优雅的关闭。
信号处理器简单的抛出了 ``SystemExit()`` 异常。
或许这一步看上去没必要，但是没有它，
终止信号会使得不执行 ``atexit.register()`` 注册的清理操作的时候就杀掉了解释器。
一个杀掉进程的例子代码可以在程序最后的 ``stop`` 命令的操作中看到。

更多关于编写守护进程的信息可以查看《UNIX 环境高级编程》, 第二版
by W. Richard Stevens and Stephen A. Rago (Addison-Wesley, 2005)。
尽管它是关注与C语言编程，但是所有的内容都适用于Python，
因为所有需要的POSIX函数都可以在标准库中找到。
==============================
13.1 通过重定向/管道/文件接受输入
==============================

----------
问题
----------
你希望你的脚本接受任何用户认为最简单的输入方式。包括将命令行的输出通过管道传递给该脚本、
重定向文件到该脚本，或在命令行中传递一个文件名或文件名列表给该脚本。

----------
解决方案
----------
Python内置的 ``fileinput`` 模块让这个变得简单。如果你有一个下面这样的脚本：

.. code-block:: python

    #!/usr/bin/env python3
    import fileinput

    with fileinput.input() as f_input:
        for line in f_input:
            print(line, end='')

那么你就能以前面提到的所有方式来为此脚本提供输入。假设你将此脚本保存为 ``filein.py`` 并将其变为可执行文件，
那么你可以像下面这样调用它，得到期望的输出：

.. code-block:: bash

    $ ls | ./filein.py          # Prints a directory listing to stdout.
    $ ./filein.py /etc/passwd   # Reads /etc/passwd to stdout.
    $ ./filein.py < /etc/passwd # Reads /etc/passwd to stdout.

----------
讨论
----------
``fileinput.input()`` 创建并返回一个 ``FileInput`` 类的实例。
该实例除了拥有一些有用的帮助方法外，它还可被当做一个上下文管理器使用。
因此，整合起来，如果我们要写一个打印多个文件输出的脚本，那么我们需要在输出中包含文件名和行号，如下所示：

.. code-block:: python

    >>> import fileinput
    >>> with fileinput.input('/etc/passwd') as f:
    >>>     for line in f:
    ...         print(f.filename(), f.lineno(), line, end='')
    ...
    /etc/passwd 1 ##
    /etc/passwd 2 # User Database
    /etc/passwd 3 #

    <other output omitted>

通过将它作为一个上下文管理器使用，可以确保它不再使用时文件能自动关闭，
而且我们在之后还演示了 ``FileInput`` 的一些有用的帮助方法来获取输出中的一些其他信息。
==============================
13.2 终止程序并给出错误信息
==============================

----------
问题
----------
你想向标准错误打印一条消息并返回某个非零状态码来终止程序运行

----------
解决方案
----------
你有一个程序像下面这样终止，抛出一个 ``SystemExit`` 异常，使用错误消息作为参数。例如：

.. code-block:: python

    raise SystemExit('It failed!')

它会将消息在 ``sys.stderr`` 中打印，然后程序以状态码1退出。

----------
讨论
----------
本节虽然很短小，但是它能解决在写脚本时的一个常见问题。
也就是说，当你想要终止某个程序时，你可能会像下面这样写：

.. code-block:: python

    import sys
    sys.stderr.write('It failed!\n')
    raise SystemExit(1)

如果你直接将消息作为参数传给 ``SystemExit()`` ，那么你可以省略其他步骤，
比如import语句或将错误消息写入 ``sys.stderr``
==============================
13.3 解析命令行选项
==============================

----------
问题
----------
你的程序如何能够解析命令行选项（位于sys.argv中）

----------
解决方案
----------
``argparse`` 模块可被用来解析命令行选项。下面一个简单例子演示了最基本的用法：

.. code-block:: python

    # search.py
    '''
    Hypothetical command-line tool for searching a collection of
    files for one or more text patterns.
    '''
    import argparse
    parser = argparse.ArgumentParser(description='Search some files')

    parser.add_argument(dest='filenames',metavar='filename', nargs='*')

    parser.add_argument('-p', '--pat',metavar='pattern', required=True,
                        dest='patterns', action='append',
                        help='text pattern to search for')

    parser.add_argument('-v', dest='verbose', action='store_true',
                        help='verbose mode')

    parser.add_argument('-o', dest='outfile', action='store',
                        help='output file')

    parser.add_argument('--speed', dest='speed', action='store',
                        choices={'slow','fast'}, default='slow',
                        help='search speed')

    args = parser.parse_args()

    # Output the collected arguments
    print(args.filenames)
    print(args.patterns)
    print(args.verbose)
    print(args.outfile)
    print(args.speed)

该程序定义了一个如下使用的命令行解析器：

.. code-block:: python

    bash % python3 search.py -h
    usage: search.py [-h] [-p pattern] [-v] [-o OUTFILE] [--speed {slow,fast}]
                     [filename [filename ...]]

    Search some files

    positional arguments:
      filename

    optional arguments:
      -h, --help            show this help message and exit
      -p pattern, --pat pattern
                            text pattern to search for
      -v                    verbose mode
      -o OUTFILE            output file
      --speed {slow,fast}   search speed

下面的部分演示了程序中的数据部分。仔细观察print()语句的打印输出。

.. code-block:: python

    bash % python3 search.py foo.txt bar.txt
    usage: search.py [-h] -p pattern [-v] [-o OUTFILE] [--speed {fast,slow}]
                     [filename [filename ...]]
    search.py: error: the following arguments are required: -p/--pat

    bash % python3 search.py -v -p spam --pat=eggs foo.txt bar.txt
    filenames = ['foo.txt', 'bar.txt']
    patterns  = ['spam', 'eggs']
    verbose   = True
    outfile   = None
    speed     = slow

    bash % python3 search.py -v -p spam --pat=eggs foo.txt bar.txt -o results
    filenames = ['foo.txt', 'bar.txt']
    patterns  = ['spam', 'eggs']
    verbose   = True
    outfile   = results
    speed     = slow

    bash % python3 search.py -v -p spam --pat=eggs foo.txt bar.txt -o results \
                 --speed=fast
    filenames = ['foo.txt', 'bar.txt']
    patterns  = ['spam', 'eggs']
    verbose   = True
    outfile   = results
    speed     = fast

对于选项值的进一步处理由程序来决定，用你自己的逻辑来替代 ``print()`` 函数。

----------
讨论
----------
``argparse`` 模块是标准库中最大的模块之一，拥有大量的配置选项。
本节只是演示了其中最基础的一些特性，帮助你入门。

为了解析命令行选项，你首先要创建一个 ``ArgumentParser`` 实例，
并使用 ``add_argument()`` 方法声明你想要支持的选项。
在每个 ``add_argument()`` 调用中，``dest`` 参数指定解析结果被指派给属性的名字。
``metavar`` 参数被用来生成帮助信息。``action`` 参数指定跟属性对应的处理逻辑，
通常的值为 ``store`` ,被用来存储某个值或将多个参数值收集到一个列表中。
下面的参数收集所有剩余的命令行参数到一个列表中。在本例中它被用来构造一个文件名列表：

.. code-block:: python

    parser.add_argument(dest='filenames',metavar='filename', nargs='*')

下面的参数根据参数是否存在来设置一个 ``Boolean`` 标志：

.. code-block:: python

    parser.add_argument('-v', dest='verbose', action='store_true',
                        help='verbose mode')

下面的参数接受一个单独值并将其存储为一个字符串：

.. code-block:: python

    parser.add_argument('-o', dest='outfile', action='store',
                        help='output file')

下面的参数说明允许某个参数重复出现多次，并将它们追加到一个列表中去。
``required`` 标志表示该参数至少要有一个。``-p`` 和 ``--pat`` 表示两个参数名形式都可使用。

.. code-block:: python

    parser.add_argument('-p', '--pat',metavar='pattern', required=True,
                        dest='patterns', action='append',
                        help='text pattern to search for')

最后，下面的参数说明接受一个值，但是会将其和可能的选择值做比较，以检测其合法性：

.. code-block:: python

    parser.add_argument('--speed', dest='speed', action='store',
                        choices={'slow','fast'}, default='slow',
                        help='search speed')

一旦参数选项被指定，你就可以执行 ``parser.parse()`` 方法了。
它会处理 ``sys.argv`` 的值并返回一个结果实例。
每个参数值会被设置成该实例中 ``add_argument()`` 方法的 ``dest`` 参数指定的属性值。

还很多种其他方法解析命令行选项。
例如，你可能会手动的处理 ``sys.argv`` 或者使用 ``getopt`` 模块。
但是，如果你采用本节的方式，将会减少很多冗余代码，底层细节 ``argparse`` 模块已经帮你处理了。
你可能还会碰到使用 ``optparse`` 库解析选项的代码。
尽管 ``optparse`` 和 ``argparse`` 很像，但是后者更先进，因此在新的程序中你应该使用它。
==============================
13.4 运行时弹出密码输入提示
==============================

----------
问题
----------
你写了个脚本，运行时需要一个密码。此脚本是交互式的，因此不能将密码在脚本中硬编码，
而是需要弹出一个密码输入提示，让用户自己输入。

----------
解决方案
----------
这时候Python的 ``getpass`` 模块正是你所需要的。你可以让你很轻松的弹出密码输入提示，
并且不会在用户终端回显密码。下面是具体代码：

.. code-block:: python

    import getpass

    user = getpass.getuser()
    passwd = getpass.getpass()

    if svc_login(user, passwd):    # You must write svc_login()
       print('Yay!')
    else:
       print('Boo!')

在此代码中，``svc_login()`` 是你要实现的处理密码的函数，具体的处理过程你自己决定。

----------
讨论
----------
注意在前面代码中 ``getpass.getuser()`` 不会弹出用户名的输入提示。
它会根据该用户的shell环境或者会依据本地系统的密码库（支持 `pwd` 模块的平台）来使用当前用户的登录名，

如果你想显示的弹出用户名输入提示，使用内置的 ``input`` 函数：

.. code-block:: python

    user = input('Enter your username: ')

还有一点很重要，有些系统可能不支持 ``getpass()`` 方法隐藏输入密码。
这种情况下，Python会提前警告你这些问题（例如它会警告你说密码会以明文形式显示）
==============================
13.5 获取终端的大小
==============================

----------
问题
----------
你需要知道当前终端的大小以便正确的格式化输出。

----------
解决方案
----------
使用 ``os.get_terminal_size()`` 函数来做到这一点。

代码示例：

.. code-block:: python

    >>> import os
    >>> sz = os.get_terminal_size()
    >>> sz
    os.terminal_size(columns=80, lines=24)
    >>> sz.columns
    80
    >>> sz.lines
    24
    >>>

----------
讨论
----------
有太多方式来得知终端大小了，从读取环境变量到执行底层的 ``ioctl()`` 函数等等。
不过，为什么要去研究这些复杂的办法而不是仅仅调用一个简单的函数呢？
==============================
13.6 执行外部命令并获取它的输出
==============================

----------
问题
----------
你想执行一个外部命令并以Python字符串的形式获取执行结果。

----------
解决方案
----------
使用 ``subprocess.check_output()`` 函数。例如：

.. code-block:: python

    import subprocess
    out_bytes = subprocess.check_output(['netstat','-a'])

这段代码执行一个指定的命令并将执行结果以一个字节字符串的形式返回。
如果你需要文本形式返回，加一个解码步骤即可。例如：

.. code-block:: python

    out_text = out_bytes.decode('utf-8')

如果被执行的命令以非零码返回，就会抛出异常。
下面的例子捕获到错误并获取返回码：

.. code-block:: python

    try:
        out_bytes = subprocess.check_output(['cmd','arg1','arg2'])
    except subprocess.CalledProcessError as e:
        out_bytes = e.output       # Output generated before error
        code      = e.returncode   # Return code

默认情况下，``check_output()`` 仅仅返回输入到标准输出的值。
如果你需要同时收集标准输出和错误输出，使用 ``stderr`` 参数：

.. code-block:: python

    out_bytes = subprocess.check_output(['cmd','arg1','arg2'],
                                        stderr=subprocess.STDOUT)

如果你需要用一个超时机制来执行命令，使用 ``timeout`` 参数：

.. code-block:: python

    try:
        out_bytes = subprocess.check_output(['cmd','arg1','arg2'], timeout=5)
    except subprocess.TimeoutExpired as e:
        ...

通常来讲，命令的执行不需要使用到底层shell环境（比如sh、bash）。
一个字符串列表会被传递给一个低级系统命令，比如 ``os.execve()`` 。
如果你想让命令被一个shell执行，传递一个字符串参数，并设置参数 ``shell=True`` .
有时候你想要Python去执行一个复杂的shell命令的时候这个就很有用了，比如管道流、I/O重定向和其他特性。例如：

.. code-block:: python

    out_bytes = subprocess.check_output('grep python | wc > out', shell=True)

需要注意的是在shell中执行命令会存在一定的安全风险，特别是当参数来自于用户输入时。
这时候可以使用 ``shlex.quote()`` 函数来讲参数正确的用双引用引起来。

----------
讨论
----------
使用 ``check_output()`` 函数是执行外部命令并获取其返回值的最简单方式。
但是，如果你需要对子进程做更复杂的交互，比如给它发送输入，你得采用另外一种方法。
这时候可直接使用 ``subprocess.Popen`` 类。例如：

.. code-block:: python

    import subprocess

    # Some text to send
    text = b'''
    hello world
    this is a test
    goodbye
    '''

    # Launch a command with pipes
    p = subprocess.Popen(['wc'],
              stdout = subprocess.PIPE,
              stdin = subprocess.PIPE)

    # Send the data and get the output
    stdout, stderr = p.communicate(text)

    # To interpret as text, decode
    out = stdout.decode('utf-8')
    err = stderr.decode('utf-8')

``subprocess`` 模块对于依赖TTY的外部命令不合适用。
例如，你不能使用它来自动化一个用户输入密码的任务（比如一个ssh会话）。
这时候，你需要使用到第三方模块了，比如基于著名的 ``expect`` 家族的工具（pexpect或类似的）
==============================
13.7 复制或者移动文件和目录
==============================

----------
问题
----------
你想要复制或移动文件和目录，但是又不想调用shell命令。

----------
解决方案
----------
``shutil`` 模块有很多便捷的函数可以复制文件和目录。使用起来非常简单，比如：

.. code-block:: python

    import shutil

    # Copy src to dst. (cp src dst)
    shutil.copy(src, dst)

    # Copy files, but preserve metadata (cp -p src dst)
    shutil.copy2(src, dst)

    # Copy directory tree (cp -R src dst)
    shutil.copytree(src, dst)

    # Move src to dst (mv src dst)
    shutil.move(src, dst)

这些函数的参数都是字符串形式的文件或目录名。
底层语义模拟了类似的Unix命令，如上面的注释部分。

默认情况下，对于符号链接而已这些命令处理的是它指向的东西。
例如，如果源文件是一个符号链接，那么目标文件将会是符号链接指向的文件。
如果你只想复制符号链接本身，那么需要指定关键字参数 ``follow_symlinks`` ,如下：

.. code-block:: python
    shutil.copy2(src, dst, follow_symlinks=False)

如果你想保留被复制目录中的符号链接，像这样做：

.. code-block:: python

    shutil.copytree(src, dst, symlinks=True)

``copytree()`` 可以让你在复制过程中选择性的忽略某些文件或目录。
你可以提供一个忽略函数，接受一个目录名和文件名列表作为输入，返回一个忽略的名称列表。例如：

.. code-block:: python

    def ignore_pyc_files(dirname, filenames):
        return [name in filenames if name.endswith('.pyc')]

    shutil.copytree(src, dst, ignore=ignore_pyc_files)

由于忽略某种模式的文件名是很常见的，因此一个便捷的函数 ``ignore_patterns()`` 已经包含在里面了。例如：

.. code-block:: python

    shutil.copytree(src, dst, ignore=shutil.ignore_patterns('*~', '*.pyc'))

----------
讨论
----------
使用 ``shutil`` 复制文件和目录也忒简单了点吧。
不过，对于文件元数据信息，``copy2()`` 这样的函数只能尽自己最大能力来保留它。
访问时间、创建时间和权限这些基本信息会被保留，
但是对于所有者、ACLs、资源fork和其他更深层次的文件元信息就说不准了，
这个还得依赖于底层操作系统类型和用户所拥有的访问权限。
你通常不会去使用 ``shutil.copytree()`` 函数来执行系统备份。
当处理文件名的时候，最好使用 ``os.path`` 中的函数来确保最大的可移植性（特别是同时要适用于Unix和Windows）。
例如：

.. code-block:: python

    >>> filename = '/Users/guido/programs/spam.py'
    >>> import os.path
    >>> os.path.basename(filename)
    'spam.py'
    >>> os.path.dirname(filename)
    '/Users/guido/programs'
    >>> os.path.split(filename)
    ('/Users/guido/programs', 'spam.py')
    >>> os.path.join('/new/dir', os.path.basename(filename))
    '/new/dir/spam.py'
    >>> os.path.expanduser('~/guido/programs/spam.py')
    '/Users/guido/programs/spam.py'
    >>>

使用 ``copytree()`` 复制文件夹的一个棘手的问题是对于错误的处理。
例如，在复制过程中，函数可能会碰到损坏的符号链接，因为权限无法访问文件的问题等等。
为了解决这个问题，所有碰到的问题会被收集到一个列表中并打包为一个单独的异常，到了最后再抛出。
下面是一个例子：

.. code-block:: python

    try:
        shutil.copytree(src, dst)
    except shutil.Error as e:
        for src, dst, msg in e.args[0]:
             # src is source name
             # dst is destination name
             # msg is error message from exception
             print(dst, src, msg)

如果你提供关键字参数 ``ignore_dangling_symlinks=True`` ，
这时候 ``copytree()`` 会忽略掉无效符号链接。

本节演示的这些函数都是最常见的。不过，``shutil`` 还有更多的和复制数据相关的操作。
它的文档很值得一看，参考 `Python documentation <https://docs.python.org/3/library/shutil.html>`_
==============================
13.8 创建和解压归档文件
==============================

----------
问题
----------
你需要创建或解压常见格式的归档文件（比如.tar, .tgz或.zip）

----------
解决方案
----------
``shutil`` 模块拥有两个函数—— ``make_archive()`` 和 ``unpack_archive()`` 可派上用场。
例如：

.. code-block:: python

    >>> import shutil
    >>> shutil.unpack_archive('Python-3.3.0.tgz')

    >>> shutil.make_archive('py33','zip','Python-3.3.0')
    '/Users/beazley/Downloads/py33.zip'
    >>>

``make_archive()`` 的第二个参数是期望的输出格式。
可以使用 ``get_archive_formats()`` 获取所有支持的归档格式列表。例如：

.. code-block:: python

    >>> shutil.get_archive_formats()
    [('bztar', "bzip2'ed tar-file"), ('gztar', "gzip'ed tar-file"),
     ('tar', 'uncompressed tar file'), ('zip', 'ZIP file')]
    >>>

----------
讨论
----------
Python还有其他的模块可用来处理多种归档格式（比如tarfile, zipfile, gzip, bz2）的底层细节。
不过，如果你仅仅只是要创建或提取某个归档，就没有必要使用底层库了。
可以直接使用 ``shutil`` 中的这些高层函数。

这些函数还有很多其他选项，用于日志打印、预检、文件权限等等。
参考 `shutil文档 <https://docs.python.org/3/library/shutil.html>`_==============================
13.9 通过文件名查找文件
==============================

----------
问题
----------
你需要写一个涉及到文件查找操作的脚本，比如对日志归档文件的重命名工具，
你不想在Python脚本中调用shell，或者你要实现一些shell不能做的功能。

----------
解决方案
----------
查找文件，可使用 ``os.walk()`` 函数，传一个顶级目录名给它。
下面是一个例子，查找特定的文件名并答应所有符合条件的文件全路径：

.. code-block:: python

    #!/usr/bin/env python3.3
    import os

    def findfile(start, name):
        for relpath, dirs, files in os.walk(start):
            if name in files:
                full_path = os.path.join(start, relpath, name)
                print(os.path.normpath(os.path.abspath(full_path)))

    if __name__ == '__main__':
        findfile(sys.argv[1], sys.argv[2])

保存脚本为文件findfile.py，然后在命令行中执行它。
指定初始查找目录以及名字作为位置参数，如下：

.. code-block:: python
    bash % ./findfile.py . myfile.txt

----------
讨论
----------
``os.walk()`` 方法为我们遍历目录树，
每次进入一个目录，它会返回一个三元组，包含相对于查找目录的相对路径，一个该目录下的目录名列表，
以及那个目录下面的文件名列表。

对于每个元组，只需检测一下目标文件名是否在文件列表中。如果是就使用 ``os.path.join()`` 合并路径。
为了避免奇怪的路径名比如 ``././foo//bar`` ，使用了另外两个函数来修正结果。
第一个是 ``os.path.abspath()`` ,它接受一个路径，可能是相对路径，最后返回绝对路径。
第二个是 ``os.path.normpath()`` ，用来返回正常路径，可以解决双斜杆、对目录的多重引用的问题等。

尽管这个脚本相对于UNIX平台上面的很多查找来讲要简单很多，它还有跨平台的优势。
并且，还能很轻松的加入其他的功能。
我们再演示一个例子，下面的函数打印所有最近被修改过的文件：

.. code-block:: python

    #!/usr/bin/env python3.3

    import os
    import time

    def modified_within(top, seconds):
        now = time.time()
        for path, dirs, files in os.walk(top):
            for name in files:
                fullpath = os.path.join(path, name)
                if os.path.exists(fullpath):
                    mtime = os.path.getmtime(fullpath)
                    if mtime > (now - seconds):
                        print(fullpath)

    if __name__ == '__main__':
        import sys
        if len(sys.argv) != 3:
            print('Usage: {} dir seconds'.format(sys.argv[0]))
            raise SystemExit(1)

        modified_within(sys.argv[1], float(sys.argv[2]))

在此函数的基础之上，使用os,os.path,glob等类似模块，你就能实现更加复杂的操作了。
可参考5.11小节和5.13小节等相关章节。
==============================
13.10 读取配置文件
==============================

----------
问题
----------
怎样读取普通.ini格式的配置文件？

----------
解决方案
----------
``configparser`` 模块能被用来读取配置文件。例如，假设你有如下的配置文件：

::

    ; config.ini
    ; Sample configuration file

    [installation]
    library=%(prefix)s/lib
    include=%(prefix)s/include
    bin=%(prefix)s/bin
    prefix=/usr/local

    # Setting related to debug configuration
    [debug]
    log_errors=true
    show_warnings=False

    [server]
    port: 8080
    nworkers: 32
    pid-file=/tmp/spam.pid
    root=/www/root
    signature:
        =================================
        Brought to you by the Python Cookbook
        =================================

下面是一个读取和提取其中值的例子：

.. code-block:: python

    >>> from configparser import ConfigParser
    >>> cfg = ConfigParser()
    >>> cfg.read('config.ini')
    ['config.ini']
    >>> cfg.sections()
    ['installation', 'debug', 'server']
    >>> cfg.get('installation','library')
    '/usr/local/lib'
    >>> cfg.getboolean('debug','log_errors')

    True
    >>> cfg.getint('server','port')
    8080
    >>> cfg.getint('server','nworkers')
    32
    >>> print(cfg.get('server','signature'))

    \=================================
    Brought to you by the Python Cookbook
    \=================================
    >>>

如果有需要，你还能修改配置并使用 ``cfg.write()`` 方法将其写回到文件中。例如：

.. code-block:: python

    >>> cfg.set('server','port','9000')
    >>> cfg.set('debug','log_errors','False')
    >>> import sys
    >>> cfg.write(sys.stdout)

::

    [installation]
    library = %(prefix)s/lib
    include = %(prefix)s/include
    bin = %(prefix)s/bin
    prefix = /usr/local

    [debug]
    log_errors = False
    show_warnings = False

    [server]
    port = 9000
    nworkers = 32
    pid-file = /tmp/spam.pid
    root = /www/root
    signature =
              =================================
              Brought to you by the Python Cookbook
              =================================
    >>>

----------
讨论
----------
配置文件作为一种可读性很好的格式，非常适用于存储程序中的配置数据。
在每个配置文件中，配置数据会被分组（比如例子中的“installation”、 “debug” 和 “server”）。
每个分组在其中指定对应的各个变量值。

对于可实现同样功能的配置文件和Python源文件是有很大的不同的。
首先，配置文件的语法要更自由些，下面的赋值语句是等效的：

::

    prefix=/usr/local
    prefix: /usr/local

配置文件中的名字是不区分大小写的。例如：

::

    >>> cfg.get('installation','PREFIX')
    '/usr/local'
    >>> cfg.get('installation','prefix')
    '/usr/local'
    >>>

在解析值的时候，``getboolean()`` 方法查找任何可行的值。例如下面都是等价的：

::

    log_errors = true
    log_errors = TRUE
    log_errors = Yes
    log_errors = 1

或许配置文件和Python代码最大的不同在于，它并不是从上而下的顺序执行。
文件是安装一个整体被读取的。如果碰到了变量替换，它实际上已经被替换完成了。
例如，在下面这个配置中，``prefix`` 变量在使用它的变量之前或之后定义都是可以的：

::

    [installation]
    library=%(prefix)s/lib
    include=%(prefix)s/include
    bin=%(prefix)s/bin
    prefix=/usr/local

``ConfigParser`` 有个容易被忽视的特性是它能一次读取多个配置文件然后合并成一个配置。
例如，假设一个用户像下面这样构造了他们的配置文件：

::

    ; ~/.config.ini
    [installation]
    prefix=/Users/beazley/test

    [debug]
    log_errors=False

读取这个文件，它就能跟之前的配置合并起来。如：

.. code-block:: python

    >>> # Previously read configuration
    >>> cfg.get('installation', 'prefix')
    '/usr/local'

    >>> # Merge in user-specific configuration
    >>> import os
    >>> cfg.read(os.path.expanduser('~/.config.ini'))
    ['/Users/beazley/.config.ini']

    >>> cfg.get('installation', 'prefix')
    '/Users/beazley/test'
    >>> cfg.get('installation', 'library')
    '/Users/beazley/test/lib'
    >>> cfg.getboolean('debug', 'log_errors')
    False
    >>>

仔细观察下 ``prefix`` 变量是怎样覆盖其他相关变量的，比如 ``library`` 的设定值。
产生这种结果的原因是变量的改写采取的是后发制人策略，以最后一个为准。
你可以像下面这样做试验：

.. code-block:: python

    >>> cfg.get('installation','library')
    '/Users/beazley/test/lib'
    >>> cfg.set('installation','prefix','/tmp/dir')
    >>> cfg.get('installation','library')
    '/tmp/dir/lib'
    >>>

最后还有很重要一点要注意的是Python并不能支持.ini文件在其他程序（比如windows应用程序）中的所有特性。
确保你已经参阅了configparser文档中的语法详情以及支持特性。

==============================
13.11 给简单脚本增加日志功能
==============================

----------
问题
----------
你希望在脚本和程序中将诊断信息写入日志文件。

----------
解决方案
----------
打印日志最简单方式是使用 ``logging`` 模块。例如：

.. code-block:: python

    import logging

    def main():
        # Configure the logging system
        logging.basicConfig(
            filename='app.log',
            level=logging.ERROR
        )

        # Variables (to make the calls that follow work)
        hostname = 'www.python.org'
        item = 'spam'
        filename = 'data.csv'
        mode = 'r'

        # Example logging calls (insert into your program)
        logging.critical('Host %s unknown', hostname)
        logging.error("Couldn't find %r", item)
        logging.warning('Feature is deprecated')
        logging.info('Opening file %r, mode=%r', filename, mode)
        logging.debug('Got here')

    if __name__ == '__main__':
        main()

上面五个日志调用（critical(), error(), warning(), info(), debug()）以降序方式表示不同的严重级别。
``basicConfig()`` 的 ``level`` 参数是一个过滤器。
所有级别低于此级别的日志消息都会被忽略掉。
每个logging操作的参数是一个消息字符串，后面再跟一个或多个参数。
构造最终的日志消息的时候我们使用了%操作符来格式化消息字符串。

运行这个程序后，在文件 ``app.log`` 中的内容应该是下面这样：

::

    CRITICAL:root:Host www.python.org unknown
    ERROR:root:Could not find 'spam'

如果你想改变输出等级，你可以修改 ``basicConfig()`` 调用中的参数。例如：

.. code-block:: python

    logging.basicConfig(
         filename='app.log',
         level=logging.WARNING,
         format='%(levelname)s:%(asctime)s:%(message)s')

最后输出变成如下：

::

    CRITICAL:2012-11-20 12:27:13,595:Host www.python.org unknown
    ERROR:2012-11-20 12:27:13,595:Could not find 'spam'
    WARNING:2012-11-20 12:27:13,595:Feature is deprecated

上面的日志配置都是硬编码到程序中的。如果你想使用配置文件，
可以像下面这样修改 ``basicConfig()`` 调用：

.. code-block:: python

    import logging
    import logging.config

    def main():
        # Configure the logging system
        logging.config.fileConfig('logconfig.ini')
        ...

创建一个下面这样的文件，名字叫 ``logconfig.ini`` ：

::

    [loggers]
    keys=root

    [handlers]
    keys=defaultHandler

    [formatters]
    keys=defaultFormatter

    [logger_root]
    level=INFO
    handlers=defaultHandler
    qualname=root

    [handler_defaultHandler]
    class=FileHandler
    formatter=defaultFormatter
    args=('app.log', 'a')

    [formatter_defaultFormatter]
    format=%(levelname)s:%(name)s:%(message)s

如果你想修改配置，可以直接编辑文件logconfig.ini即可。

----------
讨论
----------
尽管对于 ``logging`` 模块而已有很多更高级的配置选项，
不过这里的方案对于简单的程序和脚本已经足够了。
只想在调用日志操作前先执行下basicConfig()函数方法，你的程序就能产生日志输出了。

如果你想要你的日志消息写到标准错误中，而不是日志文件中，调用 ``basicConfig()`` 时不传文件名参数即可。例如：

.. code-block:: python

    logging.basicConfig(level=logging.INFO)

``basicConfig()`` 在程序中只能被执行一次。如果你稍后想改变日志配置，
就需要先获取 ``root logger`` ，然后直接修改它。例如：

.. code-block:: python

    logging.getLogger().level = logging.DEBUG

需要强调的是本节只是演示了 ``logging`` 模块的一些基本用法。
它可以做更多更高级的定制。
关于日志定制化一个很好的资源是 `Logging Cookbook <https://docs.python.org/3/howto/logging-cookbook.html>`_
==============================
13.12 给函数库增加日志功能
==============================

----------
问题
----------
你想给某个函数库增加日志功能，但是又不能影响到那些不使用日志功能的程序。

----------
解决方案
----------
对于想要执行日志操作的函数库而已，你应该创建一个专属的 ``logger`` 对象，并且像下面这样初始化配置：

.. code-block:: python

    # somelib.py

    import logging
    log = logging.getLogger(__name__)
    log.addHandler(logging.NullHandler())

    # Example function (for testing)
    def func():
        log.critical('A Critical Error!')
        log.debug('A debug message')

使用这个配置，默认情况下不会打印日志。例如：

.. code-block:: python

    >>> import somelib
    >>> somelib.func()
    >>>

不过，如果配置过日志系统，那么日志消息打印就开始生效，例如：

::

    >>> import logging
    >>> logging.basicConfig()
    >>> somelib.func()
    CRITICAL:somelib:A Critical Error!
    >>>

----------
讨论
----------
通常来讲，你不应该在函数库代码中自己配置日志系统，或者是已经假定有个已经存在的日志配置了。

调用 ``getLogger(__name__)`` 创建一个和调用模块同名的logger模块。
由于模块都是唯一的，因此创建的logger也将是唯一的。

``log.addHandler(logging.NullHandler())`` 操作将一个空处理器绑定到刚刚已经创建好的logger对象上。
一个空处理器默认会忽略调用所有的日志消息。
因此，如果使用该函数库的时候还没有配置日志，那么将不会有消息或警告出现。

还有一点就是对于各个函数库的日志配置可以是相互独立的，不影响其他库的日志配置。
例如，对于如下的代码：

.. code-block:: python

    >>> import logging
    >>> logging.basicConfig(level=logging.ERROR)

    >>> import somelib
    >>> somelib.func()
    CRITICAL:somelib:A Critical Error!

    >>> # Change the logging level for 'somelib' only
    >>> logging.getLogger('somelib').level=logging.DEBUG
    >>> somelib.func()
    CRITICAL:somelib:A Critical Error!
    DEBUG:somelib:A debug message
    >>>

在这里，根日志被配置成仅仅输出ERROR或更高级别的消息。
不过 ，``somelib`` 的日志级别被单独配置成可以输出debug级别的消息，它的优先级比全局配置高。
像这样更改单独模块的日志配置对于调试来讲是很方便的，
因为你无需去更改任何的全局日志配置——只需要修改你想要更多输出的模块的日志等级。

`Logging HOWTO <https://docs.python.org/3/howto/logging.html>`_
详细介绍了如何配置日志模块和其他有用技巧，可以参阅下。
==============================
13.13 实现一个计时器
==============================

----------
问题
----------
你想记录程序执行多个任务所花费的时间

----------
解决方案
----------
``time`` 模块包含很多函数来执行跟时间有关的函数。
尽管如此，通常我们会在此基础之上构造一个更高级的接口来模拟一个计时器。例如：

.. code-block:: python

    import time

    class Timer:
        def __init__(self, func=time.perf_counter):
            self.elapsed = 0.0
            self._func = func
            self._start = None

        def start(self):
            if self._start is not None:
                raise RuntimeError('Already started')
            self._start = self._func()

        def stop(self):
            if self._start is None:
                raise RuntimeError('Not started')
            end = self._func()
            self.elapsed += end - self._start
            self._start = None

        def reset(self):
            self.elapsed = 0.0

        @property
        def running(self):
            return self._start is not None

        def __enter__(self):
            self.start()
            return self

        def __exit__(self, *args):
            self.stop()

这个类定义了一个可以被用户根据需要启动、停止和重置的计时器。
它会在 ``elapsed`` 属性中记录整个消耗时间。
下面是一个例子来演示怎样使用它：

.. code-block:: python

    def countdown(n):
        while n > 0:
            n -= 1

    # Use 1: Explicit start/stop
    t = Timer()
    t.start()
    countdown(1000000)
    t.stop()
    print(t.elapsed)

    # Use 2: As a context manager
    with t:
        countdown(1000000)

    print(t.elapsed)

    with Timer() as t2:
        countdown(1000000)
    print(t2.elapsed)

----------
讨论
----------
本节提供了一个简单而实用的类来实现时间记录以及耗时计算。
同时也是对使用with语句以及上下文管理器协议的一个很好的演示。

在计时中要考虑一个底层的时间函数问题。一般来说，
使用 ``time.time()`` 或 ``time.clock()`` 计算的时间精度因操作系统的不同会有所不同。
而使用 ``time.perf_counter()`` 函数可以确保使用系统上面最精确的计时器。

上述代码中由 ``Timer`` 类记录的时间是钟表时间，并包含了所有休眠时间。
如果你只想计算该进程所花费的CPU时间，应该使用 ``time.process_time()`` 来代替：

.. code-block:: python

    t = Timer(time.process_time)
    with t:
        countdown(1000000)
    print(t.elapsed)

``time.perf_counter()`` 和 ``time.process_time()`` 都会返回小数形式的秒数时间。
实际的时间值没有任何意义，为了得到有意义的结果，你得执行两次函数然后计算它们的差值。

更多关于计时和性能分析的例子请参考14.13小节。
==============================
13.14 限制内存和CPU的使用量
==============================

----------
问题
----------
你想对在Unix系统上面运行的程序设置内存或CPU的使用限制。

----------
解决方案
----------
``resource`` 模块能同时执行这两个任务。例如，要限制CPU时间，可以像下面这样做：

.. code-block:: python

    import signal
    import resource
    import os

    def time_exceeded(signo, frame):
        print("Time's up!")
        raise SystemExit(1)

    def set_max_runtime(seconds):
        # Install the signal handler and set a resource limit
        soft, hard = resource.getrlimit(resource.RLIMIT_CPU)
        resource.setrlimit(resource.RLIMIT_CPU, (seconds, hard))
        signal.signal(signal.SIGXCPU, time_exceeded)

    if __name__ == '__main__':
        set_max_runtime(15)
        while True:
            pass

程序运行时，``SIGXCPU`` 信号在时间过期时被生成，然后执行清理并退出。

要限制内存使用，设置可使用的总内存值即可，如下：

.. code-block:: python

    import resource

    def limit_memory(maxsize):
        soft, hard = resource.getrlimit(resource.RLIMIT_AS)
        resource.setrlimit(resource.RLIMIT_AS, (maxsize, hard))

像这样设置了内存限制后，程序运行到没有多余内存时会抛出 ``MemoryError`` 异常。

----------
讨论
----------
在本节例子中，``setrlimit()`` 函数被用来设置特定资源上面的软限制和硬限制。
软限制是一个值，当超过这个值的时候操作系统通常会发送一个信号来限制或通知该进程。
硬限制是用来指定软限制能设定的最大值。通常来讲，这个由系统管理员通过设置系统级参数来决定。
尽管硬限制可以改小一点，但是最好不要使用用户进程去修改。

``setrlimit()`` 函数还能被用来设置子进程数量、打开文件数以及类似系统资源的限制。
更多详情请参考 ``resource`` 模块的文档。

需要注意的是本节内容只能适用于Unix系统，并且不保证所有系统都能如期工作。
比如我们在测试的时候，它能在Linux上面正常运行，但是在OS X上却不能。
==============================
13.15 启动一个WEB浏览器
==============================

----------
问题
----------
你想通过脚本启动浏览器并打开指定的URL网页

----------
解决方案
----------
``webbrowser`` 模块能被用来启动一个浏览器，并且与平台无关。例如：

.. code-block:: python

    >>> import webbrowser
    >>> webbrowser.open('http://www.python.org')
    True
    >>>

它会使用默认浏览器打开指定网页。如果你还想对网页打开方式做更多控制，还可以使用下面这些函数：

.. code-block:: python

    >>> # Open the page in a new browser window
    >>> webbrowser.open_new('http://www.python.org')
    True
    >>>

    >>> # Open the page in a new browser tab
    >>> webbrowser.open_new_tab('http://www.python.org')
    True
    >>>

这样就可以打开一个新的浏览器窗口或者标签，只要浏览器支持就行。

如果你想指定浏览器类型，可以使用 ``webbrowser.get()`` 函数来指定某个特定浏览器。例如：

.. code-block:: python

    >>> c = webbrowser.get('firefox')
    >>> c.open('http://www.python.org')
    True
    >>> c.open_new_tab('http://docs.python.org')
    True
    >>>

对于支持的浏览器名称列表可查阅`Python文档 <http://docs.python.org/3/library/webbrowser.html>`_

----------
讨论
----------
在脚本中打开浏览器有时候会很有用。例如，某个脚本执行某个服务器发布任务，
你想快速打开一个浏览器来确保它已经正常运行了。
或者是某个程序以HTML网页格式输出数据，你想打开浏览器查看结果。
不管是上面哪种情况，使用 ``webbrowser`` 模块都是一个简单实用的解决方案。

==============================
14.1 测试stdout输出
==============================

----------
问题
----------
你的程序中有个方法会输出到标准输出中（sys.stdout）。也就是说它会将文本打印到屏幕上面。
你想写个测试来证明它，给定一个输入，相应的输出能正常显示出来。

----------
解决方案
----------
使用 ``unittest.mock`` 模块中的 ``patch()`` 函数，
使用起来非常简单，可以为单个测试模拟 ``sys.stdout`` 然后回滚，
并且不产生大量的临时变量或在测试用例直接暴露状态变量。

作为一个例子，我们在 ``mymodule`` 模块中定义如下一个函数：

.. code-block:: python

    # mymodule.py

    def urlprint(protocol, host, domain):
        url = '{}://{}.{}'.format(protocol, host, domain)
        print(url)

默认情况下内置的 ``print`` 函数会将输出发送到 ``sys.stdout`` 。
为了测试输出真的在那里，你可以使用一个替身对象来模拟它，然后使用断言来确认结果。
使用 ``unittest.mock`` 模块的 ``patch()`` 方法可以很方便的在测试运行的上下文中替换对象，
并且当测试完成时候自动返回它们的原有状态。下面是对 ``mymodule`` 模块的测试代码：

.. code-block:: python

    from io import StringIO
    from unittest import TestCase
    from unittest.mock import patch
    import mymodule

    class TestURLPrint(TestCase):
        def test_url_gets_to_stdout(self):
            protocol = 'http'
            host = 'www'
            domain = 'example.com'
            expected_url = '{}://{}.{}\n'.format(protocol, host, domain)

            with patch('sys.stdout', new=StringIO()) as fake_out:
                mymodule.urlprint(protocol, host, domain)
                self.assertEqual(fake_out.getvalue(), expected_url)

----------
讨论
----------
``urlprint()`` 函数接受三个参数，测试方法开始会先设置每一个参数的值。
``expected_url`` 变量被设置成包含期望的输出的字符串。

``unittest.mock.patch()`` 函数被用作一个上下文管理器，使用 ``StringIO`` 对象来代替 ``sys.stdout`` .
``fake_out`` 变量是在该进程中被创建的模拟对象。
在with语句中使用它可以执行各种检查。当with语句结束时，``patch`` 会将所有东西恢复到测试开始前的状态。
有一点需要注意的是某些对Python的C扩展可能会忽略掉 ``sys.stdout`` 的配置二直接写入到标准输出中。
限于篇幅，本节不会涉及到这方面的讲解，它适用于纯Python代码。
如果你真的需要在C扩展中捕获I/O，你可以先打开一个临时文件，然后将标准输出重定向到该文件中。
更多关于捕获以字符串形式捕获I/O和 ``StringIO`` 对象请参阅5.6小节。

==============================
14.2 在单元测试中给对象打补丁
==============================

----------
问题
----------
你写的单元测试中需要给指定的对象打补丁，
用来断言它们在测试中的期望行为（比如，断言被调用时的参数个数，访问指定的属性等）。

----------
解决方案
----------
``unittest.mock.patch()`` 函数可被用来解决这个问题。
``patch()`` 还可被用作一个装饰器、上下文管理器或单独使用，尽管并不常见。
例如，下面是一个将它当做装饰器使用的例子：

.. code-block:: python

    from unittest.mock import patch
    import example

    @patch('example.func')
    def test1(x, mock_func):
        example.func(x)       # Uses patched example.func
        mock_func.assert_called_with(x)

它还可以被当做一个上下文管理器：

.. code-block:: python

    with patch('example.func') as mock_func:
        example.func(x)      # Uses patched example.func
        mock_func.assert_called_with(x)

最后，你还可以手动的使用它打补丁：

.. code-block:: python

    p = patch('example.func')
    mock_func = p.start()
    example.func(x)
    mock_func.assert_called_with(x)
    p.stop()

如果可能的话，你能够叠加装饰器和上下文管理器来给多个对象打补丁。例如：

.. code-block:: python

    @patch('example.func1')
    @patch('example.func2')
    @patch('example.func3')
    def test1(mock1, mock2, mock3):
        ...

    def test2():
        with patch('example.patch1') as mock1, \
             patch('example.patch2') as mock2, \
             patch('example.patch3') as mock3:
        ...

----------
讨论
----------
``patch()`` 接受一个已存在对象的全路径名，将其替换为一个新的值。
原来的值会在装饰器函数或上下文管理器完成后自动恢复回来。
默认情况下，所有值会被 ``MagicMock`` 实例替代。例如：

.. code-block:: python

    >>> x = 42
    >>> with patch('__main__.x'):
    ...     print(x)
    ...
    <MagicMock name='x' id='4314230032'>
    >>> x
    42
    >>>

不过，你可以通过给 ``patch()`` 提供第二个参数来将值替换成任何你想要的：

.. code-block:: python

    >>> x
    42
    >>> with patch('__main__.x', 'patched_value'):
    ...     print(x)
    ...
    patched_value
    >>> x
    42
    >>>

被用来作为替换值的 ``MagicMock`` 实例能够模拟可调用对象和实例。
他们记录对象的使用信息并允许你执行断言检查，例如：

.. code-block:: python

    >>> from unittest.mock import MagicMock
    >>> m = MagicMock(return_value = 10)
    >>> m(1, 2, debug=True)
    10
    >>> m.assert_called_with(1, 2, debug=True)
    >>> m.assert_called_with(1, 2)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File ".../unittest/mock.py", line 726, in assert_called_with
        raise AssertionError(msg)
    AssertionError: Expected call: mock(1, 2)
    Actual call: mock(1, 2, debug=True)
    >>>

    >>> m.upper.return_value = 'HELLO'
    >>> m.upper('hello')
    'HELLO'
    >>> assert m.upper.called

    >>> m.split.return_value = ['hello', 'world']
    >>> m.split('hello world')
    ['hello', 'world']
    >>> m.split.assert_called_with('hello world')
    >>>

    >>> m['blah']
    <MagicMock name='mock.__getitem__()' id='4314412048'>
    >>> m.__getitem__.called
    True
    >>> m.__getitem__.assert_called_with('blah')
    >>>

一般来讲，这些操作会在一个单元测试中完成。例如，假设你已经有了像下面这样的函数：

.. code-block:: python

    # example.py
    from urllib.request import urlopen
    import csv

    def dowprices():
        u = urlopen('http://finance.yahoo.com/d/quotes.csv?s=@^DJI&f=sl1')
        lines = (line.decode('utf-8') for line in u)
        rows = (row for row in csv.reader(lines) if len(row) == 2)
        prices = { name:float(price) for name, price in rows }
        return prices

正常来讲，这个函数会使用 ``urlopen()`` 从Web上面获取数据并解析它。
在单元测试中，你可以给它一个预先定义好的数据集。下面是使用补丁操作的例子:

.. code-block:: python

    import unittest
    from unittest.mock import patch
    import io
    import example

    sample_data = io.BytesIO(b'''\
    "IBM",91.1\r
    "AA",13.25\r
    "MSFT",27.72\r
    \r
    ''')

    class Tests(unittest.TestCase):
        @patch('example.urlopen', return_value=sample_data)
        def test_dowprices(self, mock_urlopen):
            p = example.dowprices()
            self.assertTrue(mock_urlopen.called)
            self.assertEqual(p,
                             {'IBM': 91.1,
                              'AA': 13.25,
                              'MSFT' : 27.72})

    if __name__ == '__main__':
        unittest.main()

本例中，位于 ``example`` 模块中的 ``urlopen()`` 函数被一个模拟对象替代，
该对象会返回一个包含测试数据的 ``ByteIO()``.

还有一点，在打补丁时我们使用了 ``example.urlopen`` 来代替 ``urllib.request.urlopen`` 。
当你创建补丁的时候，你必须使用它们在测试代码中的名称。
由于测试代码使用了 ``from urllib.request import urlopen`` ,那么 ``dowprices()`` 函数
中使用的 ``urlopen()`` 函数实际上就位于 ``example`` 模块了。

本节实际上只是对 ``unittest.mock`` 模块的一次浅尝辄止。
更多更高级的特性，请参考 `官方文档 <http://docs.python.org/3/library/unittest.mock>`_
===============================
14.3 在单元测试中测试异常情况
===============================

----------
问题
----------
你想写个测试用例来准确的判断某个异常是否被抛出。

----------
解决方案
----------
对于异常的测试可使用 ``assertRaises()`` 方法。
例如，如果你想测试某个函数抛出了 ``ValueError`` 异常，像下面这样写：

.. code-block:: python

    import unittest

    # A simple function to illustrate
    def parse_int(s):
        return int(s)

    class TestConversion(unittest.TestCase):
        def test_bad_int(self):
            self.assertRaises(ValueError, parse_int, 'N/A')

如果你想测试异常的具体值，需要用到另外一种方法：

.. code-block:: python

    import errno

    class TestIO(unittest.TestCase):
        def test_file_not_found(self):
            try:
                f = open('/file/not/found')
            except IOError as e:
                self.assertEqual(e.errno, errno.ENOENT)

            else:
                self.fail('IOError not raised')

----------
讨论
----------
``assertRaises()`` 方法为测试异常存在性提供了一个简便方法。
一个常见的陷阱是手动去进行异常检测。比如：

.. code-block:: python

    class TestConversion(unittest.TestCase):
        def test_bad_int(self):
            try:
                r = parse_int('N/A')
            except ValueError as e:
                self.assertEqual(type(e), ValueError)

这种方法的问题在于它很容易遗漏其他情况，比如没有任何异常抛出的时候。
那么你还得需要增加另外的检测过程，如下面这样：

.. code-block:: python

    class TestConversion(unittest.TestCase):
        def test_bad_int(self):
            try:
                r = parse_int('N/A')
            except ValueError as e:
                self.assertEqual(type(e), ValueError)
            else:
                self.fail('ValueError not raised')

``assertRaises()`` 方法会处理所有细节，因此你应该使用它。

``assertRaises()`` 的一个缺点是它测不了异常具体的值是多少。
为了测试异常值，可以使用 ``assertRaisesRegex()`` 方法，
它可同时测试异常的存在以及通过正则式匹配异常的字符串表示。例如：

.. code-block:: python

    class TestConversion(unittest.TestCase):
        def test_bad_int(self):
            self.assertRaisesRegex(ValueError, 'invalid literal .*',
                                           parse_int, 'N/A')

``assertRaises()`` 和 ``assertRaisesRegex()``
还有一个容易忽略的地方就是它们还能被当做上下文管理器使用：

.. code-block:: python

    class TestConversion(unittest.TestCase):
        def test_bad_int(self):
            with self.assertRaisesRegex(ValueError, 'invalid literal .*'):
                r = parse_int('N/A')

但你的测试涉及到多个执行步骤的时候这种方法就很有用了。

==============================
14.4 将测试输出用日志记录到文件中
==============================

----------
问题
----------
你希望将单元测试的输出写到到某个文件中去，而不是打印到标准输出。

----------
解决方案
----------
运行单元测试一个常见技术就是在测试文件底部加入下面这段代码片段：

.. code-block:: python

    import unittest

    class MyTest(unittest.TestCase):
        pass

    if __name__ == '__main__':
        unittest.main()

这样的话测试文件就是可执行的，并且会将运行测试的结果打印到标准输出上。
如果你想重定向输出，就需要像下面这样修改 ``main()`` 函数：

.. code-block:: python

    import sys

    def main(out=sys.stderr, verbosity=2):
        loader = unittest.TestLoader()
        suite = loader.loadTestsFromModule(sys.modules[__name__])
        unittest.TextTestRunner(out,verbosity=verbosity).run(suite)

    if __name__ == '__main__':
        with open('testing.out', 'w') as f:
            main(f)

----------
讨论
----------
本节感兴趣的部分并不是将测试结果重定向到一个文件中，
而是通过这样做向你展示了 ``unittest`` 模块中一些值得关注的内部工作原理。

``unittest`` 模块首先会组装一个测试套件。
这个测试套件包含了你定义的各种方法。一旦套件组装完成，它所包含的测试就可以被执行了。

这两步是分开的，``unittest.TestLoader`` 实例被用来组装测试套件。
``loadTestsFromModule()`` 是它定义的方法之一，用来收集测试用例。
它会为 ``TestCase`` 类扫描某个模块并将其中的测试方法提取出来。
如果你想进行细粒度的控制，
可以使用 ``loadTestsFromTestCase()`` 方法来从某个继承TestCase的类中提取测试方法。
``TextTestRunner`` 类是一个测试运行类的例子，
这个类的主要用途是执行某个测试套件中包含的测试方法。
这个类跟执行 ``unittest.main()`` 函数所使用的测试运行器是一样的。
不过，我们在这里对它进行了一些列底层配置，包括输出文件和提升级别。
尽管本节例子代码很少，但是能指导你如何对 ``unittest`` 框架进行更进一步的自定义。
要想自定义测试套件的装配方式，你可以对 ``TestLoader`` 类执行更多的操作。
为了自定义测试运行，你可以构造一个自己的测试运行类来模拟 ``TextTestRunner`` 的功能。
而这些已经超出了本节的范围。``unittest`` 模块的文档对底层实现原理有更深入的讲解，可以去看看。
==============================
14.5 忽略或期望测试失败
==============================

----------
问题
----------
你想在单元测试中忽略或标记某些测试会按照预期运行失败。

----------
解决方案
----------
``unittest`` 模块有装饰器可用来控制对指定测试方法的处理，例如：

.. code-block:: python

    import unittest
    import os
    import platform

    class Tests(unittest.TestCase):
        def test_0(self):
            self.assertTrue(True)

        @unittest.skip('skipped test')
        def test_1(self):
            self.fail('should have failed!')

        @unittest.skipIf(os.name=='posix', 'Not supported on Unix')
        def test_2(self):
            import winreg

        @unittest.skipUnless(platform.system() == 'Darwin', 'Mac specific test')
        def test_3(self):
            self.assertTrue(True)

        @unittest.expectedFailure
        def test_4(self):
            self.assertEqual(2+2, 5)

    if __name__ == '__main__':
        unittest.main()

如果你在Mac上运行这段代码，你会得到如下输出：

::

    bash % python3 testsample.py -v
    test_0 (__main__.Tests) ... ok
    test_1 (__main__.Tests) ... skipped 'skipped test'
    test_2 (__main__.Tests) ... skipped 'Not supported on Unix'
    test_3 (__main__.Tests) ... ok
    test_4 (__main__.Tests) ... expected failure

    ----------------------------------------------------------------------
    Ran 5 tests in 0.002s

    OK (skipped=2, expected failures=1)

----------
讨论
----------
``skip()`` 装饰器能被用来忽略某个你不想运行的测试。
``skipIf()`` 和 ``skipUnless()``
对于你只想在某个特定平台或Python版本或其他依赖成立时才运行测试的时候非常有用。
使用 ``@expected`` 的失败装饰器来标记那些确定会失败的测试，并且对这些测试你不想让测试框架打印更多信息。

忽略方法的装饰器还可以被用来装饰整个测试类，比如：

.. code-block:: python

    @unittest.skipUnless(platform.system() == 'Darwin', 'Mac specific tests')
    class DarwinTests(unittest.TestCase):
        pass

==============================
14.6 处理多个异常
==============================

----------
问题
----------
你有一个代码片段可能会抛出多个不同的异常，怎样才能不创建大量重复代码就能处理所有的可能异常呢？

----------
解决方案
----------
如果你可以用单个代码块处理不同的异常，可以将它们放入一个元组中，如下所示：

.. code-block:: python

    try:
        client_obj.get_url(url)
    except (URLError, ValueError, SocketTimeout):
        client_obj.remove_url(url)

在这个例子中，元祖中任何一个异常发生时都会执行 ``remove_url()`` 方法。
如果你想对其中某个异常进行不同的处理，可以将其放入另外一个 ``except`` 语句中：

.. code-block:: python

    try:
        client_obj.get_url(url)
    except (URLError, ValueError):
        client_obj.remove_url(url)
    except SocketTimeout:
        client_obj.handle_url_timeout(url)

很多的异常会有层级关系，对于这种情况，你可能使用它们的一个基类来捕获所有的异常。例如，下面的代码：

.. code-block:: python

    try:
        f = open(filename)
    except (FileNotFoundError, PermissionError):
        pass

可以被重写为：

.. code-block:: python

    try:
        f = open(filename)
    except OSError:
        pass

``OSError`` 是 ``FileNotFoundError`` 和 ``PermissionError`` 异常的基类。

----------
讨论
----------
尽管处理多个异常本身并没什么特殊的，不过你可以使用 ``as`` 关键字来获得被抛出异常的引用：

.. code-block:: python

    try:
        f = open(filename)
    except OSError as e:
        if e.errno == errno.ENOENT:
            logger.error('File not found')
        elif e.errno == errno.EACCES:
            logger.error('Permission denied')
        else:
            logger.error('Unexpected error: %d', e.errno)

这个例子中， ``e`` 变量指向一个被抛出的 ``OSError`` 异常实例。
这个在你想更进一步分析这个异常的时候会很有用，比如基于某个状态码来处理它。

同时还要注意的时候 ``except`` 语句是顺序检查的，第一个匹配的会执行。
你可以很容易的构造多个 ``except`` 同时匹配的情形，比如：

::

    >>> f = open('missing')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    FileNotFoundError: [Errno 2] No such file or directory: 'missing'
    >>> try:
    ...     f = open('missing')
    ... except OSError:
    ...     print('It failed')
    ... except FileNotFoundError:
    ...     print('File not found')
    ...
    It failed
    >>>

这里的 ``FileNotFoundError`` 语句并没有执行的原因是 ``OSError`` 更一般，它可匹配 ``FileNotFoundError`` 异常，
于是就是第一个匹配的。
在调试的时候，如果你对某个特定异常的类成层级关系不是很确定，
你可以通过查看该异常的 ``__mro__`` 属性来快速浏览。比如：

::

    >>> FileNotFoundError.__mro__
    (<class 'FileNotFoundError'>, <class 'OSError'>, <class 'Exception'>,
     <class 'BaseException'>, <class 'object'>)
    >>>

上面列表中任何一个直到 ``BaseException`` 的类都能被用于 ``except`` 语句。
==============================
14.7 捕获所有异常
==============================

----------
问题
----------
怎样捕获代码中的所有异常？

----------
解决方案
----------
想要捕获所有的异常，可以直接捕获 ``Exception`` 即可：

.. code-block:: python

    try:
       ...
    except Exception as e:
       ...
       log('Reason:', e)       # Important!

这个将会捕获除了 ``SystemExit`` 、 ``KeyboardInterrupt`` 和 ``GeneratorExit`` 之外的所有异常。
如果你还想捕获这三个异常，将 ``Exception`` 改成 ``BaseException`` 即可。

----------
讨论
----------
捕获所有异常通常是由于程序员在某些复杂操作中并不能记住所有可能的异常。
如果你不是很细心的人，这也是编写不易调试代码的一个简单方法。

正因如此，如果你选择捕获所有异常，那么在某个地方（比如日志文件、打印异常到屏幕）打印确切原因就比较重要了。
如果你没有这样做，有时候你看到异常打印时可能摸不着头脑，就像下面这样：

.. code-block:: python

    def parse_int(s):
        try:
            n = int(v)
        except Exception:
            print("Couldn't parse")

试着运行这个函数，结果如下：

::

    >>> parse_int('n/a')
    Couldn't parse
    >>> parse_int('42')
    Couldn't parse
    >>>

这时候你就会挠头想：“这咋回事啊？” 假如你像下面这样重写这个函数：

.. code-block:: python

    def parse_int(s):
        try:
            n = int(v)
        except Exception as e:
            print("Couldn't parse")
            print('Reason:', e)

这时候你能获取如下输出，指明了有个编程错误：

::

    >>> parse_int('42')
    Couldn't parse
    Reason: global name 'v' is not defined
    >>>

很明显，你应该尽可能将异常处理器定义的精准一些。
不过，要是你必须捕获所有异常，确保打印正确的诊断信息或将异常传播出去，这样不会丢失掉异常。
==============================
14.8 创建自定义异常
==============================

----------
问题
----------
在你构建的应用程序中，你想将底层异常包装成自定义的异常。

----------
解决方案
----------
创建新的异常很简单——定义新的类，让它继承自 ``Exception`` （或者是任何一个已存在的异常类型）。
例如，如果你编写网络相关的程序，你可能会定义一些类似如下的异常：

.. code-block:: python

    class NetworkError(Exception):
        pass

    class HostnameError(NetworkError):
        pass

    class TimeoutError(NetworkError):
        pass

    class ProtocolError(NetworkError):
        pass

然后用户就可以像通常那样使用这些异常了，例如：

.. code-block:: python

    try:
        msg = s.recv()
    except TimeoutError as e:
        ...
    except ProtocolError as e:
        ...

----------
讨论
----------
自定义异常类应该总是继承自内置的 ``Exception`` 类，
或者是继承自那些本身就是从 ``Exception`` 继承而来的类。
尽管所有类同时也继承自 ``BaseException`` ，但你不应该使用这个基类来定义新的异常。
``BaseException`` 是为系统退出异常而保留的，比如 ``KeyboardInterrupt`` 或 ``SystemExit``
以及其他那些会给应用发送信号而退出的异常。
因此，捕获这些异常本身没什么意义。
这样的话，假如你继承 ``BaseException``
可能会导致你的自定义异常不会被捕获而直接发送信号退出程序运行。

在程序中引入自定义异常可以使得你的代码更具可读性，能清晰显示谁应该阅读这个代码。
还有一种设计是将自定义异常通过继承组合起来。在复杂应用程序中，
使用基类来分组各种异常类也是很有用的。它可以让用户捕获一个范围很窄的特定异常，比如下面这样的：

.. code-block:: python

    try:
        s.send(msg)
    except ProtocolError:
        ...

你还能捕获更大范围的异常，就像下面这样：

.. code-block:: python

    try:
        s.send(msg)
    except NetworkError:
        ...

如果你想定义的新异常重写了 ``__init__()`` 方法，
确保你使用所有参数调用 ``Exception.__init__()`` ，例如：

.. code-block:: python

    class CustomError(Exception):
        def __init__(self, message, status):
            super().__init__(message, status)
            self.message = message
            self.status = status

看上去有点奇怪，不过Exception的默认行为是接受所有传递的参数并将它们以元组形式存储在 ``.args`` 属性中.
很多其他函数库和部分Python库默认所有异常都必须有 ``.args`` 属性，
因此如果你忽略了这一步，你会发现有些时候你定义的新异常不会按照期望运行。
为了演示 ``.args`` 的使用，考虑下下面这个使用内置的 `RuntimeError`` 异常的交互会话，
注意看raise语句中使用的参数个数是怎样的：

::

    >>> try:
    ...     raise RuntimeError('It failed')
    ... except RuntimeError as e:
    ...     print(e.args)
    ...
    ('It failed',)
    >>> try:
    ...     raise RuntimeError('It failed', 42, 'spam')
    ... except RuntimeError as e:

    ...     print(e.args)
    ...
    ('It failed', 42, 'spam')
    >>>

关于创建自定义异常的更多信息，请参考`Python官方文档 <https://docs.python.org/3/tutorial/errors.html>`_
==============================
14.9 捕获异常后抛出另外的异常
==============================

----------
问题
----------
你想捕获一个异常后抛出另外一个不同的异常，同时还得在异常回溯中保留两个异常的信息。

----------
解决方案
----------
为了链接异常，使用 ``raise from`` 语句来代替简单的 ``raise`` 语句。
它会让你同时保留两个异常的信息。例如：

::

    >>> def example():
    ...     try:
    ...             int('N/A')
    ...     except ValueError as e:
    ...             raise RuntimeError('A parsing error occurred') from e
    ...
    >>> example()
    Traceback (most recent call last):
      File "<stdin>", line 3, in example
    ValueError: invalid literal for int() with base 10: 'N/A'

上面的异常是下面的异常产生的直接原因：

::

    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 5, in example
    RuntimeError: A parsing error occurred
    >>>

在回溯中可以看到，两个异常都被捕获。
要想捕获这样的异常，你可以使用一个简单的 ``except`` 语句。
不过，你还可以通过查看异常对象的 ``__cause__`` 属性来跟踪异常链。例如：

.. code-block:: python

    try:
        example()
    except RuntimeError as e:
        print("It didn't work:", e)

        if e.__cause__:
            print('Cause:', e.__cause__)

当在 ``except`` 块中又有另外的异常被抛出时会导致一个隐藏的异常链的出现。例如：

::

    >>> def example2():
    ...     try:
    ...             int('N/A')
    ...     except ValueError as e:
    ...             print("Couldn't parse:", err)
    ...
    >>>
    >>> example2()
    Traceback (most recent call last):
      File "<stdin>", line 3, in example2
    ValueError: invalid literal for int() with base 10: 'N/A'

在处理上述异常的时候，另外一个异常发生了：

::

    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 5, in example2
    NameError: global name 'err' is not defined
    >>>

这个例子中，你同时获得了两个异常的信息，但是对异常的解释不同。
这时候，``NameError`` 异常被作为程序最终异常被抛出，而不是位于解析异常的直接回应中。

如果，你想忽略掉异常链，可使用 ``raise from None`` :

::

    >>> def example3():
    ...     try:
    ...             int('N/A')
    ...     except ValueError:
    ...             raise RuntimeError('A parsing error occurred') from None
    ...
    >>>
    example3()
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 5, in example3
    RuntimeError: A parsing error occurred
    >>>

----------
讨论
----------
在设计代码时，在另外一个 ``except`` 代码块中使用 ``raise`` 语句的时候你要特别小心了。
大多数情况下，这种 ``raise`` 语句都应该被改成 ``raise from`` 语句。也就是说你应该使用下面这种形式：

::

    try:
       ...
    except SomeException as e:
       raise DifferentException() from e

这样做的原因是你应该显示的将原因链接起来。
也就是说，``DifferentException`` 是直接从 ``SomeException`` 衍生而来。
这种关系可以从回溯结果中看出来。

如果你像下面这样写代码，你仍然会得到一个链接异常，
不过这个并没有很清晰的说明这个异常链到底是内部异常还是某个未知的编程错误。

.. code-block:: python

    try:
       ...
    except SomeException:
       raise DifferentException()

当你使用 ``raise from`` 语句的话，就很清楚的表明抛出的是第二个异常。

最后一个例子中隐藏异常链信息。
尽管隐藏异常链信息不利于回溯，同时它也丢失了很多有用的调试信息。
不过万事皆平等，有时候只保留适当的信息也是很有用的。
==============================
14.10 重新抛出被捕获的异常
==============================

----------
问题
----------
你在一个 ``except`` 块中捕获了一个异常，现在想重新抛出它。

----------
解决方案
----------
简单的使用一个单独的 ``rasie`` 语句即可，例如：

::

    >>> def example():
    ...     try:
    ...             int('N/A')
    ...     except ValueError:
    ...             print("Didn't work")
    ...             raise
    ...

    >>> example()
    Didn't work
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 3, in example
    ValueError: invalid literal for int() with base 10: 'N/A'
    >>>

----------
讨论
----------
这个问题通常是当你需要在捕获异常后执行某个操作（比如记录日志、清理等），但是之后想将异常传播下去。
一个很常见的用法是在捕获所有异常的处理器中：

.. code-block:: python

    try:
       ...
    except Exception as e:
       # Process exception information in some way
       ...

       # Propagate the exception
       raise

==============================
14.11 输出警告信息
==============================

----------
问题
----------
你希望自己的程序能生成警告信息（比如废弃特性或使用问题）。

----------
解决方案
----------
要输出一个警告消息，可使用 ``warning.warn()`` 函数。例如：

.. code-block:: python

    import warnings

    def func(x, y, logfile=None, debug=False):
        if logfile is not None:
             warnings.warn('logfile argument deprecated', DeprecationWarning)
        ...

``warn()`` 的参数是一个警告消息和一个警告类，警告类有如下几种：UserWarning,  DeprecationWarning,
SyntaxWarning, RuntimeWarning, ResourceWarning, 或 FutureWarning.

对警告的处理取决于你如何运行解释器以及一些其他配置。
例如，如果你使用 ``-W all`` 选项去运行Python，你会得到如下的输出：

::

    bash % python3 -W all example.py
    example.py:5: DeprecationWarning: logfile argument is deprecated
      warnings.warn('logfile argument is deprecated', DeprecationWarning)

通常来讲，警告会输出到标准错误上。如果你想讲警告转换为异常，可以使用 ``-W error`` 选项：

::

    bash % python3 -W error example.py
    Traceback (most recent call last):
      File "example.py", line 10, in <module>
        func(2, 3, logfile='log.txt')
      File "example.py", line 5, in func
        warnings.warn('logfile argument is deprecated', DeprecationWarning)
    DeprecationWarning: logfile argument is deprecated
    bash %

----------
讨论
----------
在你维护软件，提示用户某些信息，但是又不需要将其上升为异常级别，那么输出警告信息就会很有用了。
例如，假设你准备修改某个函数库或框架的功能，你可以先为你要更改的部分输出警告信息，同时向后兼容一段时间。
你还可以警告用户一些对代码有问题的使用方式。

作为另外一个内置函数库的警告使用例子，下面演示了一个没有关闭文件就销毁它时产生的警告消息：

.. code-block:: python

    >>> import warnings
    >>> warnings.simplefilter('always')
    >>> f = open('/etc/passwd')
    >>> del f
    __main__:1: ResourceWarning: unclosed file <_io.TextIOWrapper name='/etc/passwd'
     mode='r' encoding='UTF-8'>
    >>>

默认情况下，并不是所有警告消息都会出现。``-W`` 选项能控制警告消息的输出。
``-W all`` 会输出所有警告消息，``-W ignore`` 忽略掉所有警告，``-W error`` 将警告转换成异常。
另外一种选择，你还可以使用 ``warnings.simplefilter()`` 函数控制输出。
``always`` 参数会让所有警告消息出现，```ignore`` 忽略调所有的警告，``error`` 将警告转换成异常。

对于简单的生成警告消息的情况这些已经足够了。
``warnings`` 模块对过滤和警告消息处理提供了大量的更高级的配置选项。
更多信息请参考 `Python文档 <https://docs.python.org/3/library/warnings.html>`_
==============================
14.12 调试基本的程序崩溃错误
==============================

----------
问题
----------
你的程序崩溃后该怎样去调试它？

----------
解决方案
----------
如果你的程序因为某个异常而崩溃，运行 ``python3 -i someprogram.py`` 可执行简单的调试。
``-i`` 选项可让程序结束后打开一个交互式shell。
然后你就能查看环境，例如，假设你有下面的代码：

.. code-block:: python

    # sample.py

    def func(n):
        return n + 10

    func('Hello')

运行 ``python3 -i sample.py`` 会有类似如下的输出：

::

    bash % python3 -i sample.py
    Traceback (most recent call last):
      File "sample.py", line 6, in <module>
        func('Hello')
      File "sample.py", line 4, in func
        return n + 10
    TypeError: Can't convert 'int' object to str implicitly
    >>> func(10)
    20
    >>>

如果你看不到上面这样的，可以在程序崩溃后打开Python的调试器。例如：

::

    >>> import pdb
    >>> pdb.pm()
    > sample.py(4)func()
    -> return n + 10
    (Pdb) w
      sample.py(6)<module>()
    -> func('Hello')
    > sample.py(4)func()
    -> return n + 10
    (Pdb) print n
    'Hello'
    (Pdb) q
    >>>

如果你的代码所在的环境很难获取交互shell（比如在某个服务器上面），
通常可以捕获异常后自己打印跟踪信息。例如：

.. code-block:: python

    import traceback
    import sys

    try:
        func(arg)
    except:
        print('**** AN ERROR OCCURRED ****')
        traceback.print_exc(file=sys.stderr)

要是你的程序没有崩溃，而只是产生了一些你看不懂的结果，
你在感兴趣的地方插入一下 ``print()`` 语句也是个不错的选择。
不过，要是你打算这样做，有一些小技巧可以帮助你。
首先，``traceback.print_stack()`` 函数会你程序运行到那个点的时候创建一个跟踪栈。例如：

::

    >>> def sample(n):
    ...     if n > 0:
    ...             sample(n-1)
    ...     else:
    ...             traceback.print_stack(file=sys.stderr)
    ...
    >>> sample(5)
      File "<stdin>", line 1, in <module>
      File "<stdin>", line 3, in sample
      File "<stdin>", line 3, in sample
      File "<stdin>", line 3, in sample
      File "<stdin>", line 3, in sample
      File "<stdin>", line 3, in sample
      File "<stdin>", line 5, in sample
    >>>

另外，你还可以像下面这样使用 ``pdb.set_trace()`` 在任何地方手动的启动调试器：

.. code-block:: python

    import pdb

    def func(arg):
        ...
        pdb.set_trace()
        ...

当程序比较大而你想调试控制流程以及函数参数的时候这个就比较有用了。
例如，一旦调试器开始运行，你就能够使用 ``print`` 来观测变量值或敲击某个命令比如 ``w`` 来获取追踪信息。

----------
讨论
----------
不要将调试弄的过于复杂化。一些简单的错误只需要观察程序堆栈信息就能知道了，
实际的错误一般是堆栈的最后一行。
你在开发的时候，也可以在你需要调试的地方插入一下 ``print()``
函数来诊断信息（只需要最后发布的时候删除这些打印语句即可）。

调试器的一个常见用法是观测某个已经崩溃的函数中的变量。
知道怎样在函数崩溃后进入调试器是一个很有用的技能。

当你想解剖一个非常复杂的程序，底层的控制逻辑你不是很清楚的时候，
插入 ``pdb.set_trace()`` 这样的语句就很有用了。

实际上，程序会一直运行到碰到 ``set_trace()`` 语句位置，然后立马进入调试器。
然后你就可以做更多的事了。

如果你使用IDE来做Python开发，通常IDE都会提供自己的调试器来替代pdb。
更多这方面的信息可以参考你使用的IDE手册。
==============================
14.13 给你的程序做性能测试
==============================

----------
问题
----------

你想测试你的程序运行所花费的时间并做性能测试。

----------
解决方案
----------
如果你只是简单的想测试下你的程序整体花费的时间，
通常使用Unix时间函数就行了，比如：

::

    bash % time python3 someprogram.py
    real 0m13.937s
    user 0m12.162s
    sys  0m0.098s
    bash %

如果你还需要一个程序各个细节的详细报告，可以使用 ``cProfile`` 模块：

::

    bash % python3 -m cProfile someprogram.py
             859647 function calls in 16.016 CPU seconds

       Ordered by: standard name

       ncalls  tottime  percall  cumtime  percall filename:lineno(function)
       263169    0.080    0.000    0.080    0.000 someprogram.py:16(frange)
          513    0.001    0.000    0.002    0.000 someprogram.py:30(generate_mandel)
       262656    0.194    0.000   15.295    0.000 someprogram.py:32(<genexpr>)
            1    0.036    0.036   16.077   16.077 someprogram.py:4(<module>)
       262144   15.021    0.000   15.021    0.000 someprogram.py:4(in_mandelbrot)
            1    0.000    0.000    0.000    0.000 os.py:746(urandom)
            1    0.000    0.000    0.000    0.000 png.py:1056(_readable)
            1    0.000    0.000    0.000    0.000 png.py:1073(Reader)
            1    0.227    0.227    0.438    0.438 png.py:163(<module>)
          512    0.010    0.000    0.010    0.000 png.py:200(group)
        ...
    bash %

不过通常情况是介于这两个极端之间。比如你已经知道代码运行时在少数几个函数中花费了绝大部分时间。
对于这些函数的性能测试，可以使用一个简单的装饰器：

.. code-block:: python

    # timethis.py

    import time
    from functools import wraps

    def timethis(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            r = func(*args, **kwargs)
            end = time.perf_counter()
            print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
            return r
        return wrapper

要使用这个装饰器，只需要将其放置在你要进行性能测试的函数定义前即可，比如：

::

    >>> @timethis
    ... def countdown(n):
    ...     while n > 0:
    ...             n -= 1
    ...
    >>> countdown(10000000)
    __main__.countdown : 0.803001880645752
    >>>

要测试某个代码块运行时间，你可以定义一个上下文管理器，例如：

.. code-block:: python

    from contextlib import contextmanager

    @contextmanager
    def timeblock(label):
        start = time.perf_counter()
        try:
            yield
        finally:
            end = time.perf_counter()
            print('{} : {}'.format(label, end - start))

下面是使用这个上下文管理器的例子：

::

    >>> with timeblock('counting'):
    ...     n = 10000000
    ...     while n > 0:
    ...             n -= 1
    ...
    counting : 1.5551159381866455
    >>>

对于测试很小的代码片段运行性能，使用 ``timeit`` 模块会很方便，例如：

::

    >>> from timeit import timeit
    >>> timeit('math.sqrt(2)', 'import math')
    0.1432319980012835
    >>> timeit('sqrt(2)', 'from math import sqrt')
    0.10836604500218527
    >>>

``timeit`` 会执行第一个参数中语句100万次并计算运行时间。
第二个参数是运行测试之前配置环境。如果你想改变循环执行次数，
可以像下面这样设置 ``number`` 参数的值：

::

    >>> timeit('math.sqrt(2)', 'import math', number=10000000)
    1.434852126003534
    >>> timeit('sqrt(2)', 'from math import sqrt', number=10000000)
    1.0270336690009572
    >>>

----------
讨论
----------
当执行性能测试的时候，需要注意的是你获取的结果都是近似值。
``time.perf_counter()`` 函数会在给定平台上获取最高精度的计时值。
不过，它仍然还是基于时钟时间，很多因素会影响到它的精确度，比如机器负载。
如果你对于执行时间更感兴趣，使用 ``time.process_time()`` 来代替它。例如：

.. code-block:: python

    from functools import wraps
    def timethis(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start = time.process_time()
            r = func(*args, **kwargs)
            end = time.process_time()
            print('{}.{} : {}'.format(func.__module__, func.__name__, end - start))
            return r
        return wrapper

最后，如果你想进行更深入的性能分析，那么你需要详细阅读 ``time`` 、``timeit`` 和其他相关模块的文档。
这样你可以理解和平台相关的差异以及一些其他陷阱。
还可以参考13.13小节中相关的一个创建计时器类的例子。
==============================
14.14 加速程序运行
==============================

----------
问题
----------
你的程序运行太慢，你想在不使用复杂技术比如C扩展或JIT编译器的情况下加快程序运行速度。

----------
解决方案
----------
关于程序优化的第一个准则是“不要优化”，第二个准则是“不要优化那些无关紧要的部分”。
如果你的程序运行缓慢，首先你得使用14.13小节的技术先对它进行性能测试找到问题所在。

通常来讲你会发现你得程序在少数几个热点地方花费了大量时间，
比如内存的数据处理循环。一旦你定位到这些点，你就可以使用下面这些实用技术来加速程序运行。

**使用函数**

很多程序员刚开始会使用Python语言写一些简单脚本。
当编写脚本的时候，通常习惯了写毫无结构的代码，比如：

.. code-block:: python

    # somescript.py

    import sys
    import csv

    with open(sys.argv[1]) as f:
         for row in csv.reader(f):

             # Some kind of processing
             pass

很少有人知道，像这样定义在全局范围的代码运行起来要比定义在函数中运行慢的多。
这种速度差异是由于局部变量和全局变量的实现方式（使用局部变量要更快些）。
因此，如果你想让程序运行更快些，只需要将脚本语句放入函数中即可：

.. code-block:: python

    # somescript.py
    import sys
    import csv

    def main(filename):
        with open(filename) as f:
             for row in csv.reader(f):
                 # Some kind of processing
                 pass

    main(sys.argv[1])

速度的差异取决于实际运行的程序，不过根据经验，使用函数带来15-30%的性能提升是很常见的。

**尽可能去掉属性访问**

每一次使用点(.)操作符来访问属性的时候会带来额外的开销。
它会触发特定的方法，比如 ``__getattribute__()`` 和 ``__getattr__()`` ，这些方法会进行字典操作操作。

通常你可以使用 ``from module import name`` 这样的导入形式，以及使用绑定的方法。
假设你有如下的代码片段：

.. code-block:: python

    import math

    def compute_roots(nums):
        result = []
        for n in nums:
            result.append(math.sqrt(n))
        return result

    # Test
    nums = range(1000000)
    for n in range(100):
        r = compute_roots(nums)

在我们机器上面测试的时候，这个程序花费了大概40秒。现在我们修改 ``compute_roots()`` 函数如下：

.. code-block:: python

    from math import sqrt

    def compute_roots(nums):

        result = []
        result_append = result.append
        for n in nums:
            result_append(sqrt(n))
        return result

修改后的版本运行时间大概是29秒。唯一不同之处就是消除了属性访问。
用 ``sqrt()`` 代替了 ``math.sqrt()`` 。
``The result.append()`` 方法被赋给一个局部变量 ``result_append`` ，然后在内部循环中使用它。

不过，这些改变只有在大量重复代码中才有意义，比如循环。
因此，这些优化也只是在某些特定地方才应该被使用。

**理解局部变量**

之前提过，局部变量会比全局变量运行速度快。
对于频繁访问的名称，通过将这些名称变成局部变量可以加速程序运行。
例如，看下之前对于 ``compute_roots()`` 函数进行修改后的版本：

.. code-block:: python

    import math

    def compute_roots(nums):
        sqrt = math.sqrt
        result = []
        result_append = result.append
        for n in nums:
            result_append(sqrt(n))
        return result

在这个版本中，``sqrt`` 从 ``match`` 模块被拿出并放入了一个局部变量中。
如果你运行这个代码，大概花费25秒（对于之前29秒又是一个改进）。
这个额外的加速原因是因为对于局部变量 ``sqrt`` 的查找要快于全局变量 ``sqrt``

对于类中的属性访问也同样适用于这个原理。
通常来讲，查找某个值比如 ``self.name`` 会比访问一个局部变量要慢一些。
在内部循环中，可以将某个需要频繁访问的属性放入到一个局部变量中。例如：

.. code-block:: python

    # Slower
    class SomeClass:
        ...
        def method(self):
             for x in s:
                 op(self.value)

    # Faster
    class SomeClass:

        ...
        def method(self):
             value = self.value
             for x in s:
                 op(value)

**避免不必要的抽象**

任何时候当你使用额外的处理层（比如装饰器、属性访问、描述器）去包装你的代码时，都会让程序运行变慢。
比如看下如下的这个类：

.. code-block:: python

    class A:
        def __init__(self, x, y):
            self.x = x
            self.y = y
        @property
        def y(self):
            return self._y
        @y.setter
        def y(self, value):
            self._y = value


现在进行一个简单测试：

::

    >>> from timeit import timeit
    >>> a = A(1,2)
    >>> timeit('a.x', 'from __main__ import a')
    0.07817923510447145
    >>> timeit('a.y', 'from __main__ import a')
    0.35766440676525235
    >>>

可以看到，访问属性y相比属性x而言慢的不止一点点，大概慢了4.5倍。
如果你在意性能的话，那么就需要重新审视下对于y的属性访问器的定义是否真的有必要了。
如果没有必要，就使用简单属性吧。
如果仅仅是因为其他编程语言需要使用getter/setter函数就去修改代码风格，这个真的没有必要。

**使用内置的容器**

内置的数据类型比如字符串、元组、列表、集合和字典都是使用C来实现的，运行起来非常快。
如果你想自己实现新的数据结构（比如链接列表、平衡树等），
那么要想在性能上达到内置的速度几乎不可能，因此，还是乖乖的使用内置的吧。

**避免创建不必要的数据结构或复制**

有时候程序员想显摆下，构造一些并没有必要的数据结构。例如，有人可能会像下面这样写：

.. code-block:: python

    values = [x for x in sequence]
    squares = [x*x for x in values]

也许这里的想法是首先将一些值收集到一个列表中，然后使用列表推导来执行操作。
不过，第一个列表完全没有必要，可以简单的像下面这样写：

.. code-block:: python

    squares = [x*x for x in sequence]

与此相关，还要注意下那些对Python的共享数据机制过于偏执的程序所写的代码。
有些人并没有很好的理解或信任Python的内存模型，滥用 ``copy.deepcopy()`` 之类的函数。
通常在这些代码中是可以去掉复制操作的。

----------
讨论
----------
在优化之前，有必要先研究下使用的算法。
选择一个复杂度为 O(n log n) 的算法要比你去调整一个复杂度为 O(n**2) 的算法所带来的性能提升要大得多。

如果你觉得你还是得进行优化，那么请从整体考虑。
作为一般准则，不要对程序的每一个部分都去优化,因为这些修改会导致代码难以阅读和理解。
你应该专注于优化产生性能瓶颈的地方，比如内部循环。

你还要注意微小优化的结果。例如考虑下面创建一个字典的两种方式：

.. code-block:: python

    a = {
        'name' : 'AAPL',
        'shares' : 100,
        'price' : 534.22
    }

    b = dict(name='AAPL', shares=100, price=534.22)

后面一种写法更简洁一些（你不需要在关键字上输入引号）。
不过，如果你将这两个代码片段进行性能测试对比时，会发现使用 ``dict()`` 的方式会慢了3倍。
看到这个，你是不是有冲动把所有使用 ``dict()`` 的代码都替换成第一种。
不够，聪明的程序员只会关注他应该关注的地方，比如内部循环。在其他地方，这点性能损失没有什么影响。

如果你的优化要求比较高，本节的这些简单技术满足不了，那么你可以研究下基于即时编译（JIT）技术的一些工具。
例如，PyPy工程是Python解释器的另外一种实现，它会分析你的程序运行并对那些频繁执行的部分生成本机机器码。
它有时候能极大的提升性能，通常可以接近C代码的速度。
不过可惜的是，到写这本书位置，PyPy还不能完全支持Python3.
因此，这个是你将来需要去研究的。你还可以考虑下Numba工程，
Numba是一个在你使用装饰器来选择Python函数进行优化时的动态编译器。
这些函数会使用LLVM被编译成本地机器码。它同样可以极大的提升性能。
但是，跟PyPy一样，它对于Python 3的支持现在还停留在实验阶段。

最后我引用John Ousterhout说过的话作为结尾：“最好的性能优化是从不工作到工作状态的迁移”。
直到你真的需要优化的时候再去考虑它。确保你程序正确的运行通常比让它运行更快要更重要一些（至少开始是这样的）.

==============================
15.1 使用ctypes访问C代码
==============================

----------
问题
----------
你有一些C函数已经被编译到共享库或DLL中。你希望可以使用纯Python代码调用这些函数，
而不用编写额外的C代码或使用第三方扩展工具。

----------
解决方案
----------
对于需要调用C代码的一些小的问题，通常使用Python标准库中的 ``ctypes`` 模块就足够了。
要使用 ``ctypes`` ，你首先要确保你要访问的C代码已经被编译到和Python解释器兼容
（同样的架构、字大小、编译器等）的某个共享库中了。
为了进行本节的演示，假设你有一个共享库名字叫 ``libsample.so`` ，里面的内容就是15章介绍部分那样。
另外还假设这个 ``libsample.so`` 文件被放置到位于 ``sample.py`` 文件相同的目录中了。

要访问这个函数库，你要先构建一个包装它的Python模块，如下这样：

.. code-block:: python

    # sample.py
    import ctypes
    import os

    # Try to locate the .so file in the same directory as this file
    _file = 'libsample.so'
    _path = os.path.join(*(os.path.split(__file__)[:-1] + (_file,)))
    _mod = ctypes.cdll.LoadLibrary(_path)

    # int gcd(int, int)
    gcd = _mod.gcd
    gcd.argtypes = (ctypes.c_int, ctypes.c_int)
    gcd.restype = ctypes.c_int

    # int in_mandel(double, double, int)
    in_mandel = _mod.in_mandel
    in_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)
    in_mandel.restype = ctypes.c_int

    # int divide(int, int, int *)
    _divide = _mod.divide
    _divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))
    _divide.restype = ctypes.c_int

    def divide(x, y):
        rem = ctypes.c_int()
        quot = _divide(x, y, rem)

        return quot,rem.value

    # void avg(double *, int n)
    # Define a special type for the 'double *' argument
    class DoubleArrayType:
        def from_param(self, param):
            typename = type(param).__name__
            if hasattr(self, 'from_' + typename):
                return getattr(self, 'from_' + typename)(param)
            elif isinstance(param, ctypes.Array):
                return param
            else:
                raise TypeError("Can't convert %s" % typename)

        # Cast from array.array objects
        def from_array(self, param):
            if param.typecode != 'd':
                raise TypeError('must be an array of doubles')
            ptr, _ = param.buffer_info()
            return ctypes.cast(ptr, ctypes.POINTER(ctypes.c_double))

        # Cast from lists/tuples
        def from_list(self, param):
            val = ((ctypes.c_double)*len(param))(*param)
            return val

        from_tuple = from_list

        # Cast from a numpy array
        def from_ndarray(self, param):
            return param.ctypes.data_as(ctypes.POINTER(ctypes.c_double))

    DoubleArray = DoubleArrayType()
    _avg = _mod.avg
    _avg.argtypes = (DoubleArray, ctypes.c_int)
    _avg.restype = ctypes.c_double

    def avg(values):
        return _avg(values, len(values))

    # struct Point { }
    class Point(ctypes.Structure):
        _fields_ = [('x', ctypes.c_double),
                    ('y', ctypes.c_double)]

    # double distance(Point *, Point *)
    distance = _mod.distance
    distance.argtypes = (ctypes.POINTER(Point), ctypes.POINTER(Point))
    distance.restype = ctypes.c_double

如果一切正常，你就可以加载并使用里面定义的C函数了。例如：

::

    >>> import sample
    >>> sample.gcd(35,42)
    7
    >>> sample.in_mandel(0,0,500)
    1
    >>> sample.in_mandel(2.0,1.0,500)
    0
    >>> sample.divide(42,8)
    (5, 2)
    >>> sample.avg([1,2,3])
    2.0
    >>> p1 = sample.Point(1,2)
    >>> p2 = sample.Point(4,5)
    >>> sample.distance(p1,p2)
    4.242640687119285
    >>>

----------
讨论
----------
本小节有很多值得我们详细讨论的地方。
首先是对于C和Python代码一起打包的问题，如果你在使用 ``ctypes`` 来访问编译后的C代码，
那么需要确保这个共享库放在 ``sample.py`` 模块同一个地方。
一种可能是将生成的 ``.so`` 文件放置在要使用它的Python代码同一个目录下。
我们在 ``recipe—sample.py`` 中使用 ``__file__`` 变量来查看它被安装的位置，
然后构造一个指向同一个目录中的 ``libsample.so`` 文件的路径。

如果C函数库被安装到其他地方，那么你就要修改相应的路径。
如果C函数库在你机器上被安装为一个标准库了，
那么可以使用 ``ctypes.util.find_library()`` 函数来查找：

::

    >>> from ctypes.util import find_library
    >>> find_library('m')
    '/usr/lib/libm.dylib'
    >>> find_library('pthread')
    '/usr/lib/libpthread.dylib'
    >>> find_library('sample')
    '/usr/local/lib/libsample.so'
    >>>

一旦你知道了C函数库的位置，那么就可以像下面这样使用 ``ctypes.cdll.LoadLibrary()`` 来加载它，
其中 ``_path`` 是标准库的全路径：

.. code-block:: python

    _mod = ctypes.cdll.LoadLibrary(_path)

函数库被加载后，你需要编写几个语句来提取特定的符号并指定它们的类型。
就像下面这个代码片段一样：

.. code-block:: python

    # int in_mandel(double, double, int)
    in_mandel = _mod.in_mandel
    in_mandel.argtypes = (ctypes.c_double, ctypes.c_double, ctypes.c_int)
    in_mandel.restype = ctypes.c_int

在这段代码中，``.argtypes`` 属性是一个元组，包含了某个函数的输入按时，
而 ``.restype`` 就是相应的返回类型。
``ctypes`` 定义了大量的类型对象（比如c_double, c_int, c_short, c_float等），
代表了对应的C数据类型。如果你想让Python能够传递正确的参数类型并且正确的转换数据的话，
那么这些类型签名的绑定是很重要的一步。如果你没有这么做，不但代码不能正常运行，
还可能会导致整个解释器进程挂掉。
使用ctypes有一个麻烦点的地方是原生的C代码使用的术语可能跟Python不能明确的对应上来。
``divide()`` 函数是一个很好的例子，它通过一个参数除以另一个参数返回一个结果值。
尽管这是一个很常见的C技术，但是在Python中却不知道怎样清晰的表达出来。
例如，你不能像下面这样简单的做：

::

    >>> divide = _mod.divide
    >>> divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))
    >>> x = 0
    >>> divide(10, 3, x)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ctypes.ArgumentError: argument 3: <class 'TypeError'>: expected LP_c_int
    instance instead of int
    >>>

就算这个能正确的工作，它会违反Python对于整数的不可更改原则，并且可能会导致整个解释器陷入一个黑洞中。
对于涉及到指针的参数，你通常需要先构建一个相应的ctypes对象并像下面这样传进去：

::

    >>> x = ctypes.c_int()
    >>> divide(10, 3, x)
    3
    >>> x.value
    1
    >>>

在这里，一个 ``ctypes.c_int`` 实例被创建并作为一个指针被传进去。
跟普通Python整形不同的是，一个 ``c_int`` 对象是可以被修改的。
``.value`` 属性可被用来获取或更改这个值。

对于那些不像Python的C调用，通常可以写一个小的包装函数。
这里，我们让 ``divide()`` 函数通过元组来返回两个结果：

.. code-block:: python

    # int divide(int, int, int *)
    _divide = _mod.divide
    _divide.argtypes = (ctypes.c_int, ctypes.c_int, ctypes.POINTER(ctypes.c_int))
    _divide.restype = ctypes.c_int

    def divide(x, y):
        rem = ctypes.c_int()
        quot = _divide(x,y,rem)
        return quot, rem.value

``avg()`` 函数又是一个新的挑战。C代码期望接受到一个指针和一个数组的长度值。
但是，在Python中，我们必须考虑这个问题：数组是啥？它是一个列表？一个元组？
还是 ``array`` 模块中的一个数组？还是一个 ``numpy`` 数组？还是说所有都是？
实际上，一个Python“数组”有多种形式，你可能想要支持多种可能性。

``DoubleArrayType`` 演示了怎样处理这种情况。
在这个类中定义了一个单个方法 ``from_param()`` 。
这个方法的角色是接受一个单个参数然后将其向下转换为一个合适的ctypes对象
（本例中是一个 ``ctypes.c_double`` 的指针）。
在 ``from_param()`` 中，你可以做任何你想做的事。
参数的类型名被提取出来并被用于分发到一个更具体的方法中去。
例如，如果一个列表被传递过来，那么 ``typename`` 就是 ``list`` ，
然后 ``from_list`` 方法被调用。

对于列表和元组，``from_list`` 方法将其转换为一个 ``ctypes`` 的数组对象。
这个看上去有点奇怪，下面我们使用一个交互式例子来将一个列表转换为一个 ``ctypes`` 数组：

::

    >>> nums = [1, 2, 3]
    >>> a = (ctypes.c_double * len(nums))(*nums)
    >>> a
    <__main__.c_double_Array_3 object at 0x10069cd40>
    >>> a[0]
    1.0
    >>> a[1]
    2.0
    >>> a[2]
    3.0
    >>>

对于数组对象，``from_array()`` 提取底层的内存指针并将其转换为一个 ``ctypes`` 指针对象。例如：

::

    >>> import array
    >>> a = array.array('d',[1,2,3])
    >>> a
    array('d', [1.0, 2.0, 3.0])
    >>> ptr_ = a.buffer_info()
    >>> ptr
    4298687200
    >>> ctypes.cast(ptr, ctypes.POINTER(ctypes.c_double))
    <__main__.LP_c_double object at 0x10069cd40>
    >>>

``from_ndarray()`` 演示了对于 ``numpy`` 数组的转换操作。
通过定义 ``DoubleArrayType`` 类并在 ``avg()`` 类型签名中使用它，
那么这个函数就能接受多个不同的类数组输入了：

::

    >>> import sample
    >>> sample.avg([1,2,3])
    2.0
    >>> sample.avg((1,2,3))
    2.0
    >>> import array
    >>> sample.avg(array.array('d',[1,2,3]))
    2.0
    >>> import numpy
    >>> sample.avg(numpy.array([1.0,2.0,3.0]))
    2.0
    >>>

本节最后一部分向你演示了怎样处理一个简单的C结构。
对于结构体，你只需要像下面这样简单的定义一个类，包含相应的字段和类型即可：

.. code-block:: python

    class Point(ctypes.Structure):
        _fields_ = [('x', ctypes.c_double),
                    ('y', ctypes.c_double)]

一旦类被定义后，你就可以在类型签名中或者是需要实例化结构体的代码中使用它。例如：

::

    >>> p1 = sample.Point(1,2)
    >>> p2 = sample.Point(4,5)
    >>> p1.x
    1.0
    >>> p1.y
    2.0
    >>> sample.distance(p1,p2)
    4.242640687119285
    >>>

最后一些小的提示：如果你想在Python中访问一些小的C函数，那么 ``ctypes`` 是一个很有用的函数库。
尽管如此，如果你想要去访问一个很大的库，那么可能就需要其他的方法了，比如 ``Swig`` (15.9节会讲到) 或
Cython（15.10节）。

对于大型库的访问有个主要问题，由于ctypes并不是完全自动化，
那么你就必须花费大量时间来编写所有的类型签名，就像例子中那样。
如果函数库够复杂，你还得去编写很多小的包装函数和支持类。
另外，除非你已经完全精通了所有底层的C接口细节，包括内存分配和错误处理机制，
通常一个很小的代码缺陷、访问越界或其他类似错误就能让Python程序奔溃。

作为 ``ctypes`` 的一个替代，你还可以考虑下CFFI。CFFI提供了很多类似的功能，
但是使用C语法并支持更多高级的C代码类型。
到写这本书为止，CFFI还是一个相对较新的工程，
但是它的流行度正在快速上升。
甚至还有在讨论在Python将来的版本中将它包含进去。因此，这个真的值得一看。

==============================
15.2 简单的C扩展模块
==============================

----------
问题
----------
你想不依靠其他工具，直接使用Python的扩展API来编写一些简单的C扩展模块。

----------
解决方案
----------
对于简单的C代码，构建一个自定义扩展模块是很容易的。
作为第一步，你需要确保你的C代码有一个正确的头文件。例如：

::

    /* sample.h */

    #include <math.h>

    extern int gcd(int, int);
    extern int in_mandel(double x0, double y0, int n);
    extern int divide(int a, int b, int *remainder);
    extern double avg(double *a, int n);

    typedef struct Point {
        double x,y;
    } Point;

    extern double distance(Point *p1, Point *p2);

通常来讲，这个头文件要对应一个已经被单独编译过的库。
有了这些，下面我们演示下编写扩展函数的一个简单例子：

::

    #include "Python.h"
    #include "sample.h"

    /* int gcd(int, int) */
    static PyObject *py_gcd(PyObject *self, PyObject *args) {
      int x, y, result;

      if (!PyArg_ParseTuple(args,"ii", &x, &y)) {
        return NULL;
      }
      result = gcd(x,y);
      return Py_BuildValue("i", result);
    }

    /* int in_mandel(double, double, int) */
    static PyObject *py_in_mandel(PyObject *self, PyObject *args) {
      double x0, y0;
      int n;
      int result;

      if (!PyArg_ParseTuple(args, "ddi", &x0, &y0, &n)) {
        return NULL;
      }
      result = in_mandel(x0,y0,n);
      return Py_BuildValue("i", result);
    }

    /* int divide(int, int, int *) */
    static PyObject *py_divide(PyObject *self, PyObject *args) {
      int a, b, quotient, remainder;
      if (!PyArg_ParseTuple(args, "ii", &a, &b)) {
        return NULL;
      }
      quotient = divide(a,b, &remainder);
      return Py_BuildValue("(ii)", quotient, remainder);
    }

    /* Module method table */
    static PyMethodDef SampleMethods[] = {
      {"gcd",  py_gcd, METH_VARARGS, "Greatest common divisor"},
      {"in_mandel", py_in_mandel, METH_VARARGS, "Mandelbrot test"},
      {"divide", py_divide, METH_VARARGS, "Integer division"},
      { NULL, NULL, 0, NULL}
    };

    /* Module structure */
    static struct PyModuleDef samplemodule = {
      PyModuleDef_HEAD_INIT,

      "sample",           /* name of module */
      "A sample module",  /* Doc string (may be NULL) */
      -1,                 /* Size of per-interpreter state or -1 */
      SampleMethods       /* Method table */
    };

    /* Module initialization function */
    PyMODINIT_FUNC
    PyInit_sample(void) {
      return PyModule_Create(&samplemodule);
    }

要绑定这个扩展模块，像下面这样创建一个 ``setup.py`` 文件：

.. code-block:: python

    # setup.py
    from distutils.core import setup, Extension

    setup(name='sample',
          ext_modules=[
            Extension('sample',
                      ['pysample.c'],
                      include_dirs = ['/some/dir'],
                      define_macros = [('FOO','1')],
                      undef_macros = ['BAR'],
                      library_dirs = ['/usr/local/lib'],
                      libraries = ['sample']
                      )
            ]
    )

为了构建最终的函数库，只需简单的使用 ``python3 buildlib.py build_ext --inplace`` 命令即可：

::

    bash % python3 setup.py build_ext --inplace
    running build_ext
    building 'sample' extension
    gcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes
     -I/usr/local/include/python3.3m -c pysample.c
     -o build/temp.macosx-10.6-x86_64-3.3/pysample.o
    gcc -bundle -undefined dynamic_lookup
    build/temp.macosx-10.6-x86_64-3.3/pysample.o \
     -L/usr/local/lib -lsample -o sample.so
    bash %

如上所示，它会创建一个名字叫 ``sample.so`` 的共享库。当被编译后，你就能将它作为一个模块导入进来了：

::

    >>> import sample
    >>> sample.gcd(35, 42)
    7
    >>> sample.in_mandel(0, 0, 500)
    1
    >>> sample.in_mandel(2.0, 1.0, 500)

    0
    >>> sample.divide(42, 8)
    (5, 2)
    >>>

如果你是在Windows机器上面尝试这些步骤，可能会遇到各种环境和编译问题，你需要花更多点时间去配置。
Python的二进制分发通常使用了Microsoft  Visual Studio来构建。
为了让这些扩展能正常工作，你需要使用同样或兼容的工具来编译它。
参考相应的 `Python文档 <https://docs.python.org/3/extending/windows.html>`_

----------
讨论
----------
在尝试任何手写扩展之前，最好能先参考下Python文档中的
`扩展和嵌入Python解释器 <https://docs.python.org/3/extending/index.html>`_ .
Python的C扩展API很大，在这里整个去讲述它没什么实际意义。
不过对于最核心的部分还是可以讨论下的。

首先，在扩展模块中，你写的函数都是像下面这样的一个普通原型：

::

    static PyObject *py_func(PyObject *self, PyObject *args) {
      ...
    }

``PyObject`` 是一个能表示任何Python对象的C数据类型。
在一个高级层面，一个扩展函数就是一个接受一个Python对象
（在 PyObject *args中）元组并返回一个新Python对象的C函数。
函数的 ``self`` 参数对于简单的扩展函数没有被使用到，
不过如果你想定义新的类或者是C中的对象类型的话就能派上用场了。比如如果扩展函数是一个类的一个方法，
那么 ``self`` 就能引用那个实例了。

``PyArg_ParseTuple()`` 函数被用来将Python中的值转换成C中对应表示。
它接受一个指定输入格式的格式化字符串作为输入，比如“i”代表整数，“d”代表双精度浮点数，
同样还有存放转换后结果的C变量的地址。
如果输入的值不匹配这个格式化字符串，就会抛出一个异常并返回一个NULL值。
通过检查并返回NULL，一个合适的异常会在调用代码中被抛出。

``Py_BuildValue()`` 函数被用来根据C数据类型创建Python对象。
它同样接受一个格式化字符串来指定期望类型。
在扩展函数中，它被用来返回结果给Python。
``Py_BuildValue()`` 的一个特性是它能构建更加复杂的对象类型，比如元组和字典。
在 ``py_divide()`` 代码中，一个例子演示了怎样返回一个元组。不过，下面还有一些实例：

::

    return Py_BuildValue("i", 34);      // Return an integer
    return Py_BuildValue("d", 3.4);     // Return a double
    return Py_BuildValue("s", "Hello"); // Null-terminated UTF-8 string
    return Py_BuildValue("(ii)", 3, 4); // Tuple (3, 4)

在扩展模块底部，你会发现一个函数表，比如本节中的 ``SampleMethods`` 表。
这个表可以列出C函数、Python中使用的名字、文档字符串。
所有模块都需要指定这个表，因为它在模块初始化时要被使用到。

最后的函数 ``PyInit_sample()`` 是模块初始化函数，但该模块第一次被导入时执行。
这个函数的主要工作是在解释器中注册模块对象。

最后一个要点需要提出来，使用C函数来扩展Python要考虑的事情还有很多，本节只是一小部分。
（实际上，C API包含了超过500个函数）。你应该将本节当做是一个入门篇。
更多高级内容，可以看看 ``PyArg_ParseTuple()`` 和 ``Py_BuildValue()`` 函数的文档，
然后进一步扩展开。
==============================
15.3 编写扩展函数操作数组
==============================

----------
问题
----------
你想编写一个C扩展函数来操作数组，可能是被array模块或类似Numpy库所创建。
不过，你想让你的函数更加通用，而不是针对某个特定的库所生成的数组。

----------
解决方案
----------
为了能让接受和处理数组具有可移植性，你需要使用到 `Buffer Protocol` .
下面是一个手写的C扩展函数例子，
用来接受数组数据并调用本章开篇部分的 ``avg(double *buf, int len)`` 函数：

::

    /* Call double avg(double *, int) */
    static PyObject *py_avg(PyObject *self, PyObject *args) {
      PyObject *bufobj;
      Py_buffer view;
      double result;
      /* Get the passed Python object */
      if (!PyArg_ParseTuple(args, "O", &bufobj)) {
        return NULL;
      }

      /* Attempt to extract buffer information from it */

      if (PyObject_GetBuffer(bufobj, &view,
          PyBUF_ANY_CONTIGUOUS | PyBUF_FORMAT) == -1) {
        return NULL;
      }

      if (view.ndim != 1) {
        PyErr_SetString(PyExc_TypeError, "Expected a 1-dimensional array");
        PyBuffer_Release(&view);
        return NULL;
      }

      /* Check the type of items in the array */
      if (strcmp(view.format,"d") != 0) {
        PyErr_SetString(PyExc_TypeError, "Expected an array of doubles");
        PyBuffer_Release(&view);
        return NULL;
      }

      /* Pass the raw buffer and size to the C function */
      result = avg(view.buf, view.shape[0]);

      /* Indicate we're done working with the buffer */
      PyBuffer_Release(&view);
      return Py_BuildValue("d", result);
    }

下面我们演示下这个扩展函数是如何工作的：

::

    >>> import array
    >>> avg(array.array('d',[1,2,3]))
    2.0
    >>> import numpy
    >>> avg(numpy.array([1.0,2.0,3.0]))
    2.0
    >>> avg([1,2,3])
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: 'list' does not support the buffer interface
    >>> avg(b'Hello')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: Expected an array of doubles
    >>> a = numpy.array([[1.,2.,3.],[4.,5.,6.]])
    >>> avg(a[:,2])
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ValueError: ndarray is not contiguous
    >>> sample.avg(a)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: Expected a 1-dimensional array
    >>> sample.avg(a[0])

    2.0
    >>>

----------
讨论
----------
将一个数组对象传给C函数可能是一个扩展函数做的最常见的事。
很多Python应用程序，从图像处理到科学计算，都是基于高性能的数组处理。
通过编写能接受并操作数组的代码，你可以编写很好的兼容这些应用程序的自定义代码，
而不是只能兼容你自己的代码。

代码的关键点在于 ``PyBuffer_GetBuffer()`` 函数。
给定一个任意的Python对象，它会试着去获取底层内存信息，它简单的抛出一个异常并返回-1.
传给 ``PyBuffer_GetBuffer()`` 的特殊标志给出了所需的内存缓冲类型。
例如，``PyBUF_ANY_CONTIGUOUS`` 表示是一个连续的内存区域。

对于数组、字节字符串和其他类似对象而言，一个 ``Py_buffer`` 结构体包含了所有底层内存的信息。
它包含一个指向内存地址、大小、元素大小、格式和其他细节的指针。下面是这个结构体的定义：

::

    typedef struct bufferinfo {
        void *buf;              /* Pointer to buffer memory */
        PyObject *obj;          /* Python object that is the owner */
        Py_ssize_t len;         /* Total size in bytes */
        Py_ssize_t itemsize;    /* Size in bytes of a single item */
        int readonly;           /* Read-only access flag */
        int ndim;               /* Number of dimensions */
        char *format;           /* struct code of a single item */
        Py_ssize_t *shape;      /* Array containing dimensions */
        Py_ssize_t *strides;    /* Array containing strides */
        Py_ssize_t *suboffsets; /* Array containing suboffsets */
    } Py_buffer;

本节中，我们只关注接受一个双精度浮点数数组作为参数。
要检查元素是否是一个双精度浮点数，只需验证 ``format`` 属性是不是字符串"d".
这个也是 ``struct`` 模块用来编码二进制数据的。
通常来讲，``format`` 可以是任何兼容 ``struct`` 模块的格式化字符串，
并且如果数组包含了C结构的话它可以包含多个值。
一旦我们已经确定了底层的缓存区信息，那只需要简单的将它传给C函数，然后会被当做是一个普通的C数组了。
实际上，我们不必担心是怎样的数组类型或者它是被什么库创建出来的。
这也是为什么这个函数能兼容 ``array`` 模块也能兼容 ``numpy`` 模块中的数组了。

在返回最终结果之前，底层的缓冲区视图必须使用 ``PyBuffer_Release()`` 释放掉。
之所以要这一步是为了能正确的管理对象的引用计数。

同样，本节也仅仅只是演示了接受数组的一个小的代码片段。
如果你真的要处理数组，你可能会碰到多维数据、大数据、不同的数据类型等等问题，
那么就得去学更高级的东西了。你需要参考官方文档来获取更多详细的细节。

如果你需要编写涉及到数组处理的多个扩展，那么通过Cython来实现会更容易下。参考15.11节。
==============================
15.4 在C扩展模块中操作隐形指针
==============================

----------
问题
----------
你有一个扩展模块需要处理C结构体中的指针，
但是你又不想暴露结构体中任何内部细节给Python。

----------
解决方案
----------
隐形结构体可以很容易的通过将它们包装在胶囊对象中来处理。
考虑我们例子代码中的下列C代码片段：

::

    typedef struct Point {
        double x,y;
    } Point;

    extern double distance(Point *p1, Point *p2);

下面是一个使用胶囊包装Point结构体和 ``distance()`` 函数的扩展代码实例：

::

    /* Destructor function for points */
    static void del_Point(PyObject *obj) {
      free(PyCapsule_GetPointer(obj,"Point"));
    }

    /* Utility functions */
    static Point *PyPoint_AsPoint(PyObject *obj) {
      return (Point *) PyCapsule_GetPointer(obj, "Point");
    }

    static PyObject *PyPoint_FromPoint(Point *p, int must_free) {
      return PyCapsule_New(p, "Point", must_free ? del_Point : NULL);
    }

    /* Create a new Point object */
    static PyObject *py_Point(PyObject *self, PyObject *args) {

      Point *p;
      double x,y;
      if (!PyArg_ParseTuple(args,"dd",&x,&y)) {
        return NULL;
      }
      p = (Point *) malloc(sizeof(Point));
      p->x = x;
      p->y = y;
      return PyPoint_FromPoint(p, 1);
    }

    static PyObject *py_distance(PyObject *self, PyObject *args) {
      Point *p1, *p2;
      PyObject *py_p1, *py_p2;
      double result;

      if (!PyArg_ParseTuple(args,"OO",&py_p1, &py_p2)) {
        return NULL;
      }
      if (!(p1 = PyPoint_AsPoint(py_p1))) {
        return NULL;
      }
      if (!(p2 = PyPoint_AsPoint(py_p2))) {
        return NULL;
      }
      result = distance(p1,p2);
      return Py_BuildValue("d", result);
    }

在Python中可以像下面这样来使用这些函数：

::

    >>> import sample
    >>> p1 = sample.Point(2,3)
    >>> p2 = sample.Point(4,5)
    >>> p1
    <capsule object "Point" at 0x1004ea330>
    >>> p2
    <capsule object "Point" at 0x1005d1db0>
    >>> sample.distance(p1,p2)
    2.8284271247461903
    >>>

----------
讨论
----------
胶囊和C指针类似。在内部，它们获取一个通用指针和一个名称，可以使用 ``PyCapsule_New()`` 函数很容易的被创建。
另外，一个可选的析构函数能被绑定到胶囊上，用来在胶囊对象被垃圾回收时释放底层的内存。

要提取胶囊中的指针，可使用 ``PyCapsule_GetPointer()`` 函数并指定名称。
如果提供的名称和胶囊不匹配或其他错误出现，那么就会抛出异常并返回NULL。

本节中，一对工具函数—— ``PyPoint_FromPoint()`` 和 ``PyPoint_AsPoint()``
被用来创建和从胶囊对象中提取Point实例。
在任何扩展函数中，我们会使用这些函数而不是直接使用胶囊对象。
这种设计使得我们可以很容易的应对将来对Point底下的包装的更改。
例如，如果你决定使用另外一个胶囊了，那么只需要更改这两个函数即可。

对于胶囊对象一个难点在于垃圾回收和内存管理。
``PyPoint_FromPoint()`` 函数接受一个 ``must_free`` 参数，
用来指定当胶囊被销毁时底层Point * 结构体是否应该被回收。
在某些C代码中，归属问题通常很难被处理（比如一个Point结构体被嵌入到一个被单独管理的大结构体中）。
程序员可以使用 ``extra`` 参数来控制，而不是单方面的决定垃圾回收。
要注意的是和现有胶囊有关的析构器能使用 ``PyCapsule_SetDestructor()`` 函数来更改。

对于涉及到结构体的C代码而言，使用胶囊是一个比较合理的解决方案。
例如，有时候你并不关心暴露结构体的内部信息或者将其转换成一个完整的扩展类型。
通过使用胶囊，你可以在它上面放一个轻量级的包装器，然后将它传给其他的扩展函数。

==============================
15.5 从扩展模块中定义和导出C的API
==============================

----------
问题
----------
你有一个C扩展模块，在内部定义了很多有用的函数，你想将它们导出为一个公共的C API供其他地方使用。
你想在其他扩展模块中使用这些函数，但是不知道怎样将它们链接起来，
并且通过C编译器/链接器来做看上去特别复杂（或者不可能做到）。

----------
解决方案
----------
本节主要问题是如何处理15.4小节中提到的Point对象。仔细回一下，在C代码中包含了如下这些工具函数：

::

    /* Destructor function for points */
    static void del_Point(PyObject *obj) {

      free(PyCapsule_GetPointer(obj,"Point"));
    }

    /* Utility functions */
    static Point *PyPoint_AsPoint(PyObject *obj) {
      return (Point *) PyCapsule_GetPointer(obj, "Point");
    }

    static PyObject *PyPoint_FromPoint(Point *p, int must_free) {
      return PyCapsule_New(p, "Point", must_free ? del_Point : NULL);
    }

现在的问题是怎样将 ``PyPoint_AsPoint()`` 和 ``Point_FromPoint()`` 函数作为API导出，
这样其他扩展模块能使用并链接它们，比如如果你有其他扩展也想使用包装的Point对象。

要解决这个问题，首先要为 ``sample`` 扩展写个新的头文件名叫 ``pysample.h`` ，如下：

::

    /* pysample.h */
    #include "Python.h"
    #include "sample.h"
    #ifdef __cplusplus
    extern "C" {
    #endif

    /* Public API Table */
    typedef struct {
      Point *(*aspoint)(PyObject *);
      PyObject *(*frompoint)(Point *, int);
    } _PointAPIMethods;

    #ifndef PYSAMPLE_MODULE
    /* Method table in external module */
    static _PointAPIMethods *_point_api = 0;

    /* Import the API table from sample */
    static int import_sample(void) {
      _point_api = (_PointAPIMethods *) PyCapsule_Import("sample._point_api",0);
      return (_point_api != NULL) ? 1 : 0;
    }

    /* Macros to implement the programming interface */
    #define PyPoint_AsPoint(obj) (_point_api->aspoint)(obj)
    #define PyPoint_FromPoint(obj) (_point_api->frompoint)(obj)
    #endif

    #ifdef __cplusplus
    }
    #endif

这里最重要的部分是函数指针表 ``_PointAPIMethods`` .
它会在导出模块时被初始化，然后导入模块时被查找到。
修改原始的扩展模块来填充表格并将它像下面这样导出：


::

    /* pysample.c */

    #include "Python.h"
    #define PYSAMPLE_MODULE
    #include "pysample.h"

    ...
    /* Destructor function for points */
    static void del_Point(PyObject *obj) {
      printf("Deleting point\n");
      free(PyCapsule_GetPointer(obj,"Point"));
    }

    /* Utility functions */
    static Point *PyPoint_AsPoint(PyObject *obj) {
      return (Point *) PyCapsule_GetPointer(obj, "Point");
    }

    static PyObject *PyPoint_FromPoint(Point *p, int free) {
      return PyCapsule_New(p, "Point", free ? del_Point : NULL);
    }

    static _PointAPIMethods _point_api = {
      PyPoint_AsPoint,
      PyPoint_FromPoint
    };
    ...

    /* Module initialization function */
    PyMODINIT_FUNC
    PyInit_sample(void) {
      PyObject *m;
      PyObject *py_point_api;

      m = PyModule_Create(&samplemodule);
      if (m == NULL)
        return NULL;

      /* Add the Point C API functions */
      py_point_api = PyCapsule_New((void *) &_point_api, "sample._point_api", NULL);
      if (py_point_api) {
        PyModule_AddObject(m, "_point_api", py_point_api);
      }
      return m;
    }

最后，下面是一个新的扩展模块例子，用来加载并使用这些API函数：

::

    /* ptexample.c */

    /* Include the header associated with the other module */
    #include "pysample.h"

    /* An extension function that uses the exported API */
    static PyObject *print_point(PyObject *self, PyObject *args) {
      PyObject *obj;
      Point *p;
      if (!PyArg_ParseTuple(args,"O", &obj)) {
        return NULL;
      }

      /* Note: This is defined in a different module */
      p = PyPoint_AsPoint(obj);
      if (!p) {
        return NULL;
      }
      printf("%f %f\n", p->x, p->y);
      return Py_BuildValue("");
    }

    static PyMethodDef PtExampleMethods[] = {
      {"print_point", print_point, METH_VARARGS, "output a point"},
      { NULL, NULL, 0, NULL}
    };

    static struct PyModuleDef ptexamplemodule = {
      PyModuleDef_HEAD_INIT,
      "ptexample",           /* name of module */
      "A module that imports an API",  /* Doc string (may be NULL) */
      -1,                 /* Size of per-interpreter state or -1 */
      PtExampleMethods       /* Method table */
    };

    /* Module initialization function */
    PyMODINIT_FUNC
    PyInit_ptexample(void) {
      PyObject *m;

      m = PyModule_Create(&ptexamplemodule);
      if (m == NULL)
        return NULL;

      /* Import sample, loading its API functions */
      if (!import_sample()) {
        return NULL;
      }

      return m;
    }

编译这个新模块时，你甚至不需要去考虑怎样将函数库或代码跟其他模块链接起来。
例如，你可以像下面这样创建一个简单的 ``setup.py`` 文件：

::

    # setup.py
    from distutils.core import setup, Extension

    setup(name='ptexample',
          ext_modules=[
            Extension('ptexample',
                      ['ptexample.c'],
                      include_dirs = [],  # May need pysample.h directory
                      )
            ]
    )

如果一切正常，你会发现你的新扩展函数能和定义在其他模块中的C API函数一起运行的很好。

::

    >>> import sample
    >>> p1 = sample.Point(2,3)
    >>> p1
    <capsule object "Point *" at 0x1004ea330>
    >>> import ptexample
    >>> ptexample.print_point(p1)
    2.000000 3.000000
    >>>

----------
讨论
----------
本节基于一个前提就是，胶囊对象能获取任何你想要的对象的指针。
这样的话，定义模块会填充一个函数指针的结构体，创建一个指向它的胶囊，并在一个模块级属性中保存这个胶囊，
例如 ``sample._point_api`` .

其他模块能够在导入时获取到这个属性并提取底层的指针。
事实上，Python提供了 ``PyCapsule_Import()`` 工具函数，为了完成所有的步骤。
你只需提供属性的名字即可（比如sample._point_api），然后他就会一次性找到胶囊对象并提取出指针来。

在将被导出函数变为其他模块中普通函数时，有一些C编程陷阱需要指出来。
在 ``pysample.h`` 文件中，一个 ``_point_api`` 指针被用来指向在导出模块中被初始化的方法表。
一个相关的函数 ``import_sample()`` 被用来指向胶囊导入并初始化这个指针。
这个函数必须在任何函数被使用之前被调用。通常来讲，它会在模块初始化时被调用到。
最后，C的预处理宏被定义，被用来通过方法表去分发这些API函数。
用户只需要使用这些原始函数名称即可，不需要通过宏去了解其他信息。

最后，还有一个重要的原因让你去使用这个技术来链接模块——它非常简单并且可以使得各个模块很清晰的解耦。
如果你不想使用本机的技术，那你就必须使用共享库的高级特性和动态加载器来链接模块。
例如，将一个普通的API函数放入一个共享库并确保所有扩展模块链接到那个共享库。
这种方法确实可行，但是它相对繁琐，特别是在大型系统中。
本节演示了如何通过Python的普通导入机制和仅仅几个胶囊调用来将多个模块链接起来的魔法。
对于模块的编译，你只需要定义头文件，而不需要考虑函数库的内部细节。

更多关于利用C API来构造扩展模块的信息可以参考
`Python的文档 <http://docs.python.org/3/extending/extending.html>`_
==============================
15.6 从C语言中调用Python代码
==============================

----------
问题
----------
你想在C中安全的执行某个Python调用并返回结果给C。
例如，你想在C语言中使用某个Python函数作为一个回调。

----------
解决方案
----------
在C语言中调用Python非常简单，不过设计到一些小窍门。
下面的C代码告诉你怎样安全的调用：

::

    #include <Python.h>

    /* Execute func(x,y) in the Python interpreter.  The
       arguments and return result of the function must
       be Python floats */

    double call_func(PyObject *func, double x, double y) {
      PyObject *args;
      PyObject *kwargs;
      PyObject *result = 0;
      double retval;

      /* Make sure we own the GIL */
      PyGILState_STATE state = PyGILState_Ensure();

      /* Verify that func is a proper callable */
      if (!PyCallable_Check(func)) {
        fprintf(stderr,"call_func: expected a callable\n");
        goto fail;
      }
      /* Build arguments */
      args = Py_BuildValue("(dd)", x, y);
      kwargs = NULL;

      /* Call the function */
      result = PyObject_Call(func, args, kwargs);
      Py_DECREF(args);
      Py_XDECREF(kwargs);

      /* Check for Python exceptions (if any) */
      if (PyErr_Occurred()) {
        PyErr_Print();
        goto fail;
      }

      /* Verify the result is a float object */
      if (!PyFloat_Check(result)) {
        fprintf(stderr,"call_func: callable didn't return a float\n");
        goto fail;
      }

      /* Create the return value */
      retval = PyFloat_AsDouble(result);
      Py_DECREF(result);

      /* Restore previous GIL state and return */
      PyGILState_Release(state);
      return retval;

    fail:
      Py_XDECREF(result);
      PyGILState_Release(state);
      abort();   // Change to something more appropriate
    }

要使用这个函数，你需要获取传递过来的某个已存在Python调用的引用。
有很多种方法可以让你这样做，
比如将一个可调用对象传给一个扩展模块或直接写C代码从已存在模块中提取出来。

下面是一个简单例子用来掩饰从一个嵌入的Python解释器中调用一个函数：

::

    #include <Python.h>

    /* Definition of call_func() same as above */
    ...

    /* Load a symbol from a module */
    PyObject *import_name(const char *modname, const char *symbol) {
      PyObject *u_name, *module;
      u_name = PyUnicode_FromString(modname);
      module = PyImport_Import(u_name);
      Py_DECREF(u_name);
      return PyObject_GetAttrString(module, symbol);
    }

    /* Simple embedding example */
    int main() {
      PyObject *pow_func;
      double x;

      Py_Initialize();
      /* Get a reference to the math.pow function */
      pow_func = import_name("math","pow");

      /* Call it using our call_func() code */
      for (x = 0.0; x < 10.0; x += 0.1) {
        printf("%0.2f %0.2f\n", x, call_func(pow_func,x,2.0));
      }
      /* Done */
      Py_DECREF(pow_func);
      Py_Finalize();
      return 0;
    }

要构建例子代码，你需要编译C并将它链接到Python解释器。
下面的Makefile可以教你怎样做（不过在你机器上面需要一些配置）。

::

    all::
            cc -g embed.c -I/usr/local/include/python3.3m \
              -L/usr/local/lib/python3.3/config-3.3m -lpython3.3m

编译并运行会产生类似下面的输出：

::

    0.00 0.00
    0.10 0.01
    0.20 0.04
    0.30 0.09
    0.40 0.16
    ...

下面是一个稍微不同的例子，展示了一个扩展函数，
它接受一个可调用对象和其他参数，并将它们传递给 ``call_func()`` 来做测试：

::

    /* Extension function for testing the C-Python callback */
    PyObject *py_call_func(PyObject *self, PyObject *args) {
      PyObject *func;

      double x, y, result;
      if (!PyArg_ParseTuple(args,"Odd", &func,&x,&y)) {
        return NULL;
      }
      result = call_func(func, x, y);
      return Py_BuildValue("d", result);
    }

使用这个扩展函数，你要像下面这样测试它：

::

    >>> import sample
    >>> def add(x,y):
    ...     return x+y
    ...
    >>> sample.call_func(add,3,4)
    7.0
    >>>

----------
讨论
----------
如果你在C语言中调用Python，要记住最重要的是C语言会是主体。
也就是说，C语言负责构造参数、调用Python函数、检查异常、检查类型、提取返回值等。

作为第一步，你必须先有一个表示你将要调用的Python可调用对象。
这可以是一个函数、类、方法、内置方法或其他任意实现了 ``__call__()`` 操作的东西。
为了确保是可调用的，可以像下面的代码这样利用 ``PyCallable_Check()`` 做检查：

::

    double call_func(PyObject *func, double x, double y) {
      ...
      /* Verify that func is a proper callable */
      if (!PyCallable_Check(func)) {
        fprintf(stderr,"call_func: expected a callable\n");
        goto fail;
      }
      ...

在C代码里处理错误你需要格外的小心。一般来讲，你不能仅仅抛出一个Python异常。
错误应该使用C代码方式来被处理。在这里，我们打算将对错误的控制传给一个叫 ``abort()`` 的错误处理器。
它会结束掉整个程序，在真实环境下面你应该要处理的更加优雅些（返回一个状态码）。
你要记住的是在这里C是主角，因此并没有跟抛出异常相对应的操作。
错误处理是你在编程时必须要考虑的事情。

调用一个函数相对来讲很简单——只需要使用 ``PyObject_Call()`` ，
传一个可调用对象给它、一个参数元组和一个可选的关键字字典。
要构建参数元组或字典，你可以使用 ``Py_BuildValue()`` ,如下：

::

    double call_func(PyObject *func, double x, double y) {
      PyObject *args;
      PyObject *kwargs;

      ...
      /* Build arguments */
      args = Py_BuildValue("(dd)", x, y);
      kwargs = NULL;

      /* Call the function */
      result = PyObject_Call(func, args, kwargs);
      Py_DECREF(args);
      Py_XDECREF(kwargs);
      ...

如果没有关键字参数，你可以传递NULL。当你要调用函数时，
需要确保使用了 ``Py_DECREF()`` 或者 ``Py_XDECREF()`` 清理参数。
第二个函数相对安全点，因为它允许传递NULL指针（直接忽略它），
这也是为什么我们使用它来清理可选的关键字参数。

调用万Python函数之后，你必须检查是否有异常发生。
``PyErr_Occurred()`` 函数可被用来做这件事。
对对于异常的处理就有点麻烦了，由于是用C语言写的，你没有像Python那么的异常机制。
因此，你必须要设置一个异常状态码，打印异常信息或其他相应处理。
在这里，我们选择了简单的 ``abort()`` 来处理。另外，传统C程序员可能会直接让程序奔溃。

::

      ...
      /* Check for Python exceptions (if any) */
      if (PyErr_Occurred()) {
        PyErr_Print();
        goto fail;
      }
      ...
      fail:
        PyGILState_Release(state);
        abort();

从调用Python函数的返回值中提取信息通常要进行类型检查和提取值。
要这样做的话，你必须使用Python对象层中的函数。
在这里我们使用了 ``PyFloat_Check()`` 和 ``PyFloat_AsDouble()`` 来检查和提取Python浮点数。

最后一个问题是对于Python全局锁的管理。
在C语言中访问Python的时候，你需要确保GIL被正确的获取和释放了。
不然的话，可能会导致解释器返回错误数据或者直接奔溃。
调用 ``PyGILState_Ensure()`` 和 ``PyGILState_Release()`` 可以确保一切都能正常。

::

    double call_func(PyObject *func, double x, double y) {
      ...
      double retval;

      /* Make sure we own the GIL */
      PyGILState_STATE state = PyGILState_Ensure();
      ...
      /* Code that uses Python C API functions */
      ...
      /* Restore previous GIL state and return */
      PyGILState_Release(state);
      return retval;

    fail:
      PyGILState_Release(state);
      abort();
    }

一旦返回，``PyGILState_Ensure()`` 可以确保调用线程独占Python解释器。
就算C代码运行于另外一个解释器不知道的线程也没事。
这时候，C代码可以自由的使用任何它想要的Python C-API 函数。
调用成功后，PyGILState_Release()被用来讲解释器恢复到原始状态。

要注意的是每一个 ``PyGILState_Ensure()``
调用必须跟着一个匹配的 ``PyGILState_Release()`` 调用——即便有错误发生。
在这里，我们使用一个 ``goto`` 语句看上去是个可怕的设计，
但是实际上我们使用它来讲控制权转移给一个普通的exit块来执行相应的操作。
在 ``fail:`` 标签后面的代码和Python的 ``fianl:`` 块的用途是一样的。

如果你使用所有这些约定来编写C代码，包括对GIL的管理、异常检查和错误检查，
你会发现从C语言中调用Python解释器是可靠的——就算再复杂的程序，用到了高级编程技巧比如多线程都没问题。

==============================
15.7 从C扩展中释放全局锁
==============================

----------
问题
----------
你想让C扩展代码和Python解释器中的其他进程一起正确的执行，
那么你就需要去释放并重新获取全局解释器锁（GIL）。

----------
解决方案
----------
在C扩展代码中，GIL可以通过在代码中插入下面这样的宏来释放和重新获取：

::

    #include "Python.h"
    ...

    PyObject *pyfunc(PyObject *self, PyObject *args) {
       ...
       Py_BEGIN_ALLOW_THREADS
       // Threaded C code.  Must not use Python API functions
       ...
       Py_END_ALLOW_THREADS
       ...
       return result;
    }

----------
讨论
----------
只有当你确保没有Python C API函数在C中执行的时候你才能安全的释放GIL。
GIL需要被释放的常见的场景是在计算密集型代码中需要在C数组上执行计算（比如在numpy中）
或者是要执行阻塞的I/O操作时（比如在一个文件描述符上读取或写入时）。

当GIL被释放后，其他Python线程才被允许在解释器中执行。
``Py_END_ALLOW_THREADS`` 宏会阻塞执行直到调用线程重新获取了GIL。

==============================
15.8 C和Python中的线程混用
==============================

----------
问题
----------
你有一个程序需要混合使用C、Python和线程，
有些线程是在C中创建的，超出了Python解释器的控制范围。
并且一些线程还使用了Python C API中的函数。

----------
解决方案
----------
如果你想将C、Python和线程混合在一起，你需要确保正确的初始化和管理Python的全局解释器锁（GIL）。
要想这样做，可以将下列代码放到你的C代码中并确保它在任何线程被创建之前被调用。

::

    #include <Python.h>
      ...
      if (!PyEval_ThreadsInitialized()) {
        PyEval_InitThreads();
      }
      ...

对于任何调用Python对象或Python C API的C代码，确保你首先已经正确地获取和释放了GIL。
这可以用 ``PyGILState_Ensure()`` 和 ``PyGILState_Release()`` 来做到，如下所示：

::

  ...
  /* Make sure we own the GIL */
  PyGILState_STATE state = PyGILState_Ensure();

  /* Use functions in the interpreter */
  ...
  /* Restore previous GIL state and return */
  PyGILState_Release(state);
  ...

每次调用 ``PyGILState_Ensure()`` 都要相应的调用 ``PyGILState_Release()`` .

----------
讨论
----------
在涉及到C和Python的高级程序中，很多事情一起做是很常见的——
可能是对C、Python、C线程、Python线程的混合使用。
只要你确保解释器被正确的初始化，并且涉及到解释器的C代码执行了正确的GIL管理，应该没什么问题。

要注意的是调用 ``PyGILState_Ensure()`` 并不会立刻抢占或中断解释器。
如果有其他代码正在执行，这个函数被中断知道那个执行代码释放掉GIL。
在内部，解释器会执行周期性的线程切换，因此如果其他线程在执行，
调用者最终还是可以运行的（尽管可能要先等一会）。
==============================
15.9 用WSIG包装C代码
==============================

----------
问题
----------
你想让你写的C代码作为一个C扩展模块来访问，想通过使用 `Swig包装生成器 <http://www.swig.org/>`_ 来完成。

----------
解决方案
----------
Swig通过解析C头文件并自动创建扩展代码来操作。
要使用它，你先要有一个C头文件。例如，我们示例的头文件如下：

::

    /* sample.h */

    #include <math.h>
    extern int gcd(int, int);
    extern int in_mandel(double x0, double y0, int n);
    extern int divide(int a, int b, int *remainder);
    extern double avg(double *a, int n);

    typedef struct Point {
        double x,y;
    } Point;

    extern double distance(Point *p1, Point *p2);

一旦你有了这个头文件，下一步就是编写一个Swig"接口"文件。
按照约定，这些文件以".i"后缀并且类似下面这样：

::

    // sample.i - Swig interface
    %module sample
    %{
    #include "sample.h"
    %}

    /* Customizations */
    %extend Point {
        /* Constructor for Point objects */
        Point(double x, double y) {
            Point *p = (Point *) malloc(sizeof(Point));
            p->x = x;
            p->y = y;
            return p;
       };
    };

    /* Map int *remainder as an output argument */
    %include typemaps.i
    %apply int *OUTPUT { int * remainder };

    /* Map the argument pattern (double *a, int n) to arrays */
    %typemap(in) (double *a, int n)(Py_buffer view) {
      view.obj = NULL;
      if (PyObject_GetBuffer($input, &view, PyBUF_ANY_CONTIGUOUS | PyBUF_FORMAT) == -1) {
        SWIG_fail;
      }
      if (strcmp(view.format,"d") != 0) {
        PyErr_SetString(PyExc_TypeError, "Expected an array of doubles");
        SWIG_fail;
      }
      $1 = (double *) view.buf;
      $2 = view.len / sizeof(double);
    }

    %typemap(freearg) (double *a, int n) {
      if (view$argnum.obj) {
        PyBuffer_Release(&view$argnum);
      }
    }

    /* C declarations to be included in the extension module */

    extern int gcd(int, int);
    extern int in_mandel(double x0, double y0, int n);
    extern int divide(int a, int b, int *remainder);
    extern double avg(double *a, int n);

    typedef struct Point {
        double x,y;
    } Point;

    extern double distance(Point *p1, Point *p2);

一旦你写好了接口文件，就可以在命令行工具中调用Swig了：

::

    bash % swig -python -py3 sample.i
    bash %

swig的输出就是两个文件，sample_wrap.c和sample.py。
后面的文件就是用户需要导入的。
而sample_wrap.c文件是需要被编译到名叫 ``_sample`` 的支持模块的C代码。
这个可以通过跟普通扩展模块一样的技术来完成。
例如，你创建了一个如下所示的 ``setup.py`` 文件：

.. code-block:: python

    # setup.py
    from distutils.core import setup, Extension

    setup(name='sample',
          py_modules=['sample.py'],
          ext_modules=[
            Extension('_sample',
                      ['sample_wrap.c'],
                      include_dirs = [],
                      define_macros = [],

                      undef_macros = [],
                      library_dirs = [],
                      libraries = ['sample']
                      )
            ]
    )

要编译和测试，在setup.py上执行python3，如下：

::

    bash % python3 setup.py build_ext --inplace
    running build_ext
    building '_sample' extension
    gcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes
    -I/usr/local/include/python3.3m -c sample_wrap.c
     -o build/temp.macosx-10.6-x86_64-3.3/sample_wrap.o
    sample_wrap.c: In function ‘SWIG_InitializeModule’:
    sample_wrap.c:3589: warning: statement with no effect
    gcc -bundle -undefined dynamic_lookup build/temp.macosx-10.6-x86_64-3.3/sample.o
     build/temp.macosx-10.6-x86_64-3.3/sample_wrap.o -o _sample.so -lsample
    bash %

如果一切正常的话，你会发现你就可以很方便的使用生成的C扩展模块了。例如：

::

    >>> import sample
    >>> sample.gcd(42,8)
    2
    >>> sample.divide(42,8)
    [5, 2]
    >>> p1 = sample.Point(2,3)
    >>> p2 = sample.Point(4,5)
    >>> sample.distance(p1,p2)
    2.8284271247461903
    >>> p1.x
    2.0
    >>> p1.y
    3.0
    >>> import array
    >>> a = array.array('d',[1,2,3])
    >>> sample.avg(a)
    2.0
    >>>

----------
讨论
----------
Swig是Python历史中构建扩展模块的最古老的工具之一。
Swig能自动化很多包装生成器的处理。

所有Swig接口都以类似下面这样的为开头：

::

    %module sample
    %{
    #include "sample.h"
    %}

这个仅仅只是声明了扩展模块的名称并指定了C头文件，
为了能让编译通过必须要包含这些头文件（位于 %{ 和 %} 的代码），
将它们之间复制粘贴到输出代码中，这也是你要放置所有包含文件和其他编译需要的定义的地方。

Swig接口的底下部分是一个C声明列表，你需要在扩展中包含它。
这通常从头文件中被复制。在我们的例子中，我们仅仅像下面这样直接粘贴在头文件中：

::

    %module sample
    %{
    #include "sample.h"
    %}
    ...
    extern int gcd(int, int);
    extern int in_mandel(double x0, double y0, int n);
    extern int divide(int a, int b, int *remainder);
    extern double avg(double *a, int n);

    typedef struct Point {
        double x,y;
    } Point;

    extern double distance(Point *p1, Point *p2);

有一点需要强调的是这些声明会告诉Swig你想要在Python模块中包含哪些东西。
通常你需要编辑这个声明列表或相应的修改下它。
例如，如果你不想某些声明被包含进来，你要将它从声明列表中移除掉。

使用Swig最复杂的地方是它能给C代码提供大量的自定义操作。
这个主题太大，这里无法展开，但是我们在本节还剩展示了一些自定义的东西。

第一个自定义是 ``%extend`` 指令允许方法被附加到已存在的结构体和类定义上。
我例子中，这个被用来添加一个Point结构体的构造器方法。
它可以让你像下面这样使用这个结构体：

::

    >>> p1 = sample.Point(2,3)
    >>>

如果略过的话，Point对象就必须以更加复杂的方式来被创建：

::

    >>> # Usage if %extend Point is omitted
    >>> p1 = sample.Point()
    >>> p1.x = 2.0
    >>> p1.y = 3

第二个自定义涉及到对 ``typemaps.i`` 库的引入和 ``%apply`` 指令，
它会指示Swig参数签名 ``int *remainder`` 要被当做是输出值。
这个实际上是一个模式匹配规则。
在接下来的所有声明中，任何时候只要碰上 ``int  *remainder`` ，他就会被作为输出。
这个自定义方法可以让 ``divide()`` 函数返回两个值。

::

    >>> sample.divide(42,8)
    [5, 2]
    >>>

最后一个涉及到 ``%typemap`` 指令的自定义可能是这里展示的最高级的特性了。
一个typemap就是一个在输入中特定参数模式的规则。
在本节中，一个typemap被定义为匹配参数模式 ``(double *a, int n)`` .
在typemap内部是一个C代码片段，它告诉Swig怎样将一个Python对象转换为相应的C参数。
本节代码使用了Python的缓存协议去匹配任何看上去类似双精度数组的输入参数
（比如NumPy数组、array模块创建的数组等），更多请参考15.3小节。

在typemap代码内部，$1和$2这样的变量替换会获取typemap模式的C参数值
（比如$1映射为 ``double *a`` ）。$input指向一个作为输入的 ``PyObject *`` 参数，
而 ``$argnum`` 就代表参数的个数。

编写和理解typemaps是使用Swig最基本的前提。
不仅是说代码更神秘，而且你需要理解Python C API和Swig和它交互的方式。
Swig文档有更多这方面的细节，可以参考下。

不过，如果你有大量的C代码需要被暴露为扩展模块。
Swig是一个非常强大的工具。关键点在于Swig是一个处理C声明的编译器，
通过强大的模式匹配和自定义组件，可以让你更改声明指定和类型处理方式。
更多信息请去查阅 `Swig网站 <http://www.swig.org/>`_ ，
还有 `特定于Python的相关文档 <http://www.swig.org/Doc2.0/Python.html>`_

==============================
15.10 用Cython包装C代码
==============================

----------
问题
----------
你想使用Cython来创建一个Python扩展模块，用来包装某个已存在的C函数库。

----------
解决方案
----------
使用Cython构建一个扩展模块看上去很手写扩展有些类似，
因为你需要创建很多包装函数。不过，跟前面不同的是，你不需要在C语言中做这些——代码看上去更像是Python。

作为准备，假设本章介绍部分的示例代码已经被编译到某个叫 ``libsample`` 的C函数库中了。
首先创建一个名叫 ``csample.pxd`` 的文件，如下所示：

::

    # csample.pxd
    #
    # Declarations of "external" C functions and structures

    cdef extern from "sample.h":
        int gcd(int, int)
        bint in_mandel(double, double, int)
        int divide(int, int, int *)
        double avg(double *, int) nogil

        ctypedef struct Point:
             double x
             double y

        double distance(Point *, Point *)

这个文件在Cython中的作用就跟C的头文件一样。
初始声明 ``cdef  extern  from  "sample.h"`` 指定了所学的C头文件。
接下来的声明都是来自于那个头文件。文件名是 ``csample.pxd`` ，而不是 ``sample.pxd`` ——这点很重要。

下一步，创建一个名为 ``sample.pyx`` 的问题。
该文件会定义包装器，用来桥接Python解释器到 ``csample.pxd`` 中声明的C代码。

::

    # sample.pyx

    # Import the low-level C declarations
    cimport csample

    # Import some functionality from Python and the C stdlib
    from cpython.pycapsule cimport *

    from libc.stdlib cimport malloc, free

    # Wrappers
    def gcd(unsigned int x, unsigned int y):
        return csample.gcd(x, y)

    def in_mandel(x, y, unsigned int n):
        return csample.in_mandel(x, y, n)

    def divide(x, y):
        cdef int rem
        quot = csample.divide(x, y, &rem)
        return quot, rem

    def avg(double[:] a):
        cdef:
            int sz
            double result

        sz = a.size
        with nogil:
            result = csample.avg(<double *> &a[0], sz)
        return result

    # Destructor for cleaning up Point objects
    cdef del_Point(object obj):
        pt = <csample.Point *> PyCapsule_GetPointer(obj,"Point")
        free(<void *> pt)

    # Create a Point object and return as a capsule
    def Point(double x,double y):
        cdef csample.Point *p
        p = <csample.Point *> malloc(sizeof(csample.Point))
        if p == NULL:
            raise MemoryError("No memory to make a Point")
        p.x = x
        p.y = y
        return PyCapsule_New(<void *>p,"Point",<PyCapsule_Destructor>del_Point)

    def distance(p1, p2):
        pt1 = <csample.Point *> PyCapsule_GetPointer(p1,"Point")
        pt2 = <csample.Point *> PyCapsule_GetPointer(p2,"Point")
        return csample.distance(pt1,pt2)

该文件更多的细节部分会在讨论部分详细展开。
最后，为了构建扩展模块，像下面这样创建一个 ``setup.py`` 文件：

.. code-block:: python

    from distutils.core import setup
    from distutils.extension import Extension
    from Cython.Distutils import build_ext

    ext_modules = [
        Extension('sample',

                  ['sample.pyx'],
                  libraries=['sample'],
                  library_dirs=['.'])]
    setup(
      name = 'Sample extension module',
      cmdclass = {'build_ext': build_ext},
      ext_modules = ext_modules
    )

要构建我们测试的目标模块，像下面这样做：

::

    bash % python3 setup.py build_ext --inplace
    running build_ext
    cythoning sample.pyx to sample.c
    building 'sample' extension
    gcc -fno-strict-aliasing -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes
     -I/usr/local/include/python3.3m -c sample.c
     -o build/temp.macosx-10.6-x86_64-3.3/sample.o
    gcc -bundle -undefined dynamic_lookup build/temp.macosx-10.6-x86_64-3.3/sample.o
      -L. -lsample -o sample.so
    bash %

如果一切顺利的话，你应该有了一个扩展模块 ``sample.so`` ，可在下面例子中使用：

::

    >>> import sample
    >>> sample.gcd(42,10)
    2
    >>> sample.in_mandel(1,1,400)
    False
    >>> sample.in_mandel(0,0,400)
    True
    >>> sample.divide(42,10)
    (4, 2)
    >>> import array
    >>> a = array.array('d',[1,2,3])
    >>> sample.avg(a)
    2.0
    >>> p1 = sample.Point(2,3)
    >>> p2 = sample.Point(4,5)
    >>> p1
    <capsule object "Point" at 0x1005d1e70>
    >>> p2
    <capsule object "Point" at 0x1005d1ea0>
    >>> sample.distance(p1,p2)
    2.8284271247461903
    >>>

----------
讨论
----------
本节包含了很多前面所讲的高级特性，包括数组操作、包装隐形指针和释放GIL。
每一部分都会逐个被讲述到，但是我们最好能复习一下前面几小节。
在顶层，使用Cython是基于C之上。.pxd文件仅仅只包含C定义（类似.h文件），
.pyx文件包含了实现（类似.c文件）。``cimport`` 语句被Cython用来导入.pxd文件中的定义。
它跟使用普通的加载Python模块的导入语句是不同的。

尽管 `.pxd` 文件包含了定义，但它们并不是用来自动创建扩展代码的。
因此，你还是要写包装函数。例如，就算 ``csample.pxd`` 文件声明了 ``int gcd(int, int)`` 函数，
你仍然需要在 ``sample.pyx`` 中为它写一个包装函数。例如：

.. code-block:: python

    cimport csample

    def gcd(unsigned int x, unsigned int y):
        return csample.gcd(x,y)

对于简单的函数，你并不需要去做太多的时。
Cython会生成包装代码来正确的转换参数和返回值。
绑定到属性上的C数据类型是可选的。不过，如果你包含了它们，你可以另外做一些错误检查。
例如，如果有人使用负数来调用这个函数，会抛出一个异常：

::

    >>> sample.gcd(-10,2)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "sample.pyx", line 7, in sample.gcd (sample.c:1284)
        def gcd(unsigned int x,unsigned int y):
    OverflowError: can't convert negative value to unsigned int
    >>>

如果你想对包装函数做另外的检查，只需要使用另外的包装代码。例如：

::

    def gcd(unsigned int x, unsigned int y):
        if x <= 0:
            raise ValueError("x must be > 0")
        if y <= 0:
            raise ValueError("y must be > 0")
        return csample.gcd(x,y)

在csample.pxd文件中的``in_mandel()`` 声明有个很有趣但是比较难理解的定义。
在这个文件中，函数被声明为然后一个bint而不是一个int。
它会让函数创建一个正确的Boolean值而不是简单的整数。
因此，返回值0表示False而1表示True。

在Cython包装器中，你可以选择声明C数据类型，也可以使用所有的常见Python对象。
对于 ``divide()`` 的包装器展示了这样一个例子，同时还有如何去处理一个指针参数。

::

    def divide(x,y):
        cdef int rem
        quot = csample.divide(x,y,&rem)
        return quot, rem

在这里，``rem`` 变量被显示的声明为一个C整型变量。
当它被传入 ``divide()`` 函数的时候，``&rem`` 创建一个跟C一样的指向它的指针。
``avg()`` 函数的代码演示了Cython更高级的特性。
首先 ``def avg(double[:] a)`` 声明了 ``avg()`` 接受一个一维的双精度内存视图。
最惊奇的部分是返回的结果函数可以接受任何兼容的数组对象，包括被numpy创建的。例如：

::

    >>> import array
    >>> a = array.array('d',[1,2,3])
    >>> import numpy
    >>> b = numpy.array([1., 2., 3.])
    >>> import sample
    >>> sample.avg(a)
    2.0
    >>> sample.avg(b)
    2.0
    >>>

在此包装器中，``a.size0`` 和 ``&a[0]`` 分别引用数组元素个数和底层指针。
语法 ``<double *> &a[0]`` 教你怎样将指针转换为不同的类型。
前提是C中的 ``avg()`` 接受一个正确类型的指针。
参考下一节关于Cython内存视图的更高级讲述。

除了处理通常的数组外，``avg()`` 的这个例子还展示了如何处理全局解释器锁。
语句 ``with nogil:`` 声明了一个不需要GIL就能执行的代码块。
在这个块中，不能有任何的普通Python对象——只能使用被声明为 ``cdef`` 的对象和函数。
另外，外部函数必须现实的声明它们能不依赖GIL就能执行。
因此，在csample.pxd文件中，``avg()`` 被声明为 ``double avg(double *, int) nogil`` .

对Point结构体的处理是一个挑战。本节使用胶囊对象将Point对象当做隐形指针来处理，这个在15.4小节介绍过。
要这样做的话，底层Cython代码稍微有点复杂。
首先，下面的导入被用来引入C函数库和Python C API中定义的函数：

::

    from cpython.pycapsule cimport *
    from libc.stdlib cimport malloc, free

函数 ``del_Point()`` 和 ``Point()`` 使用这个功能来创建一个胶囊对象，
它会包装一个 ``Point  *`` 指针。``cdef  del_Point()`` 将 ``del_Point()`` 声明为一个函数，
只能通过Cython访问，而不能从Python中访问。
因此，这个函数对外部是不可见的——它被用来当做一个回调函数来清理胶囊分配的内存。
函数调用比如 ``PyCapsule_New()`` 、``PyCapsule_GetPointer()``
直接来自Python C API并且以同样的方式被使用。

``distance`` 函数从 ``Point()`` 创建的胶囊对象中提取指针。
这里要注意的是你不需要担心异常处理。
如果一个错误的对象被传进来，``PyCapsule_GetPointer()`` 会抛出一个异常，
但是Cython已经知道怎么查找到它，并将它从 ``distance()`` 传递出去。

处理Point结构体一个缺点是它的实现是不可见的。
你不能访问任何属性来查看它的内部。
这里有另外一种方法去包装它，就是定义一个扩展类型，如下所示：

::

    # sample.pyx

    cimport csample
    from libc.stdlib cimport malloc, free
    ...

    cdef class Point:
        cdef csample.Point *_c_point
        def __cinit__(self, double x, double y):
            self._c_point = <csample.Point *> malloc(sizeof(csample.Point))
            self._c_point.x = x
            self._c_point.y = y

        def __dealloc__(self):
            free(self._c_point)

        property x:
            def __get__(self):
                return self._c_point.x
            def __set__(self, value):
                self._c_point.x = value

        property y:
            def __get__(self):
                return self._c_point.y
            def __set__(self, value):
                self._c_point.y = value

    def distance(Point p1, Point p2):
        return csample.distance(p1._c_point, p2._c_point)

在这里，cdif类 ``Point`` 将Point声明为一个扩展类型。
类属性 ``cdef csample.Point *_c_point`` 声明了一个实例变量，
拥有一个指向底层Point结构体的指针。
``__cinit__()`` 和 ``__dealloc__()`` 方法通过 ``malloc()`` 和 ``free()`` 创建并销毁底层C结构体。
x和y属性的声明让你获取和设置底层结构体的属性值。
``distance()`` 的包装器还可以被修改，使得它能接受 ``Point`` 扩展类型实例作为参数，
而传递底层指针给C函数。

做了这个改变后，你会发现操作Point对象就显得更加自然了：

::

    >>> import sample
    >>> p1 = sample.Point(2,3)
    >>> p2 = sample.Point(4,5)
    >>> p1
    <sample.Point object at 0x100447288>
    >>> p2
    <sample.Point object at 0x1004472a0>
    >>> p1.x
    2.0
    >>> p1.y
    3.0
    >>> sample.distance(p1,p2)
    2.8284271247461903
    >>>

本节已经演示了很多Cython的核心特性，你可以以此为基准来构建更多更高级的包装。
不过，你最好先去阅读下官方文档来了解更多信息。

接下来几节还会继续演示一些Cython的其他特性。
==============================
15.11 用Cython写高性能的数组操作
==============================

----------
问题
----------
你要写高性能的操作来自NumPy之类的数组计算函数。
你已经知道了Cython这样的工具会让它变得简单，但是并不确定该怎样去做。

----------
解决方案
----------
作为一个例子，下面的代码演示了一个Cython函数，用来修整一个简单的一维双精度浮点数数组中元素的值。

::

    # sample.pyx (Cython)

    cimport cython

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef clip(double[:] a, double min, double max, double[:] out):
        '''
        Clip the values in a to be between min and max. Result in out
        '''
        if min > max:
            raise ValueError("min must be <= max")
        if a.shape[0] != out.shape[0]:
            raise ValueError("input and output arrays must be the same size")
        for i in range(a.shape[0]):
            if a[i] < min:
                out[i] = min
            elif a[i] > max:
                out[i] = max
            else:
                out[i] = a[i]

要编译和构建这个扩展，你需要一个像下面这样的 ``setup.py`` 文件
（使用 ``python3 setup.py build_ext --inplace`` 来构建它）：

.. code-block:: python

    from distutils.core import setup
    from distutils.extension import Extension
    from Cython.Distutils import build_ext

    ext_modules = [
        Extension('sample',
                  ['sample.pyx'])
    ]

    setup(
      name = 'Sample app',
      cmdclass = {'build_ext': build_ext},
      ext_modules = ext_modules
    )

你会发现结果函数确实对数组进行的修正，并且可以适用于多种类型的数组对象。例如：

::

    >>> # array module example
    >>> import sample
    >>> import array
    >>> a = array.array('d',[1,-3,4,7,2,0])
    >>> a

    array('d', [1.0, -3.0, 4.0, 7.0, 2.0, 0.0])
    >>> sample.clip(a,1,4,a)
    >>> a
    array('d', [1.0, 1.0, 4.0, 4.0, 2.0, 1.0])

    >>> # numpy example
    >>> import numpy
    >>> b = numpy.random.uniform(-10,10,size=1000000)
    >>> b
    array([-9.55546017,  7.45599334,  0.69248932, ...,  0.69583148,
           -3.86290931,  2.37266888])
    >>> c = numpy.zeros_like(b)
    >>> c
    array([ 0.,  0.,  0., ...,  0.,  0.,  0.])
    >>> sample.clip(b,-5,5,c)
    >>> c
    array([-5.        ,  5.        ,  0.69248932, ...,  0.69583148,
           -3.86290931,  2.37266888])
    >>> min(c)
    -5.0
    >>> max(c)
    5.0
    >>>

你还会发现运行生成结果非常的快。
下面我们将本例和numpy中的已存在的 ``clip()`` 函数做一个性能对比：

::

    >>> timeit('numpy.clip(b,-5,5,c)','from __main__ import b,c,numpy',number=1000)
    8.093049556000551
    >>> timeit('sample.clip(b,-5,5,c)','from __main__ import b,c,sample',
    ...         number=1000)
    3.760528204000366
    >>>

正如你看到的，它要快很多——这是一个很有趣的结果，因为NumPy版本的核心代码还是用C语言写的。

----------
讨论
----------
本节利用了Cython类型的内存视图，极大的简化了数组的操作。
``cpdef clip()`` 声明了 ``clip()`` 同时为C级别函数以及Python级别函数。
在Cython中，这个是很重要的，因为它表示此函数调用要比其他Cython函数更加高效
（比如你想在另外一个不同的Cython函数中调用clip()）。

类型参数 ``double[:] a`` 和 ``double[:] out`` 声明这些参数为一维的双精度数组。
作为输入，它们会访问任何实现了内存视图接口的数组对象，这个在PEP 3118有详细定义。
包括了NumPy中的数组和内置的array库。

当你编写生成结果为数组的代码时，你应该遵循上面示例那样设置一个输出参数。
它会将创建输出数组的责任给调用者，不需要知道你操作的数组的具体细节
（它仅仅假设数组已经准备好了，只需要做一些小的检查比如确保数组大小是正确的）。
在像NumPy之类的库中，使用 ``numpy.zeros()`` 或 ``numpy.zeros_like()``
创建输出数组相对而言比较容易。另外，要创建未初始化数组，
你可以使用 ``numpy.empty()`` 或 ``numpy.empty_like()`` .
如果你想覆盖数组内容作为结果的话选择这两个会比较快点。

在你的函数实现中，你只需要简单的通过下标运算和数组查找（比如a[i],out[i]等）来编写代码操作数组。
Cython会负责为你生成高效的代码。

``clip()`` 定义之前的两个装饰器可以优化下性能。
``@cython.boundscheck(False)`` 省去了所有的数组越界检查，
当你知道下标访问不会越界的时候可以使用它。
``@cython.wraparound(False)`` 消除了相对数组尾部的负数下标的处理（类似Python列表）。
引入这两个装饰器可以极大的提升性能（测试这个例子的时候大概快了2.5倍）。

任何时候处理数组时，研究并改善底层算法同样可以极大的提示性能。
例如，考虑对 ``clip()`` 函数的如下修正，使用条件表达式：

::

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef clip(double[:] a, double min, double max, double[:] out):
        if min > max:
            raise ValueError("min must be <= max")
        if a.shape[0] != out.shape[0]:
            raise ValueError("input and output arrays must be the same size")
        for i in range(a.shape[0]):
            out[i] = (a[i] if a[i] < max else max) if a[i] > min else min

实际测试结果是，这个版本的代码运行速度要快50%以上（2.44秒对比之前使用 ``timeit()`` 测试的3.76秒）。

到这里为止，你可能想知道这种代码怎么能跟手写C语言PK呢？
例如，你可能写了如下的C函数并使用前面几节的技术来手写扩展：

::

    void clip(double *a, int n, double min, double max, double *out) {
      double x;
      for (; n >= 0; n--, a++, out++) {
        x = *a;

        *out = x > max ? max : (x < min ? min : x);
      }
    }

我们没有展示这个的扩展代码，但是试验之后，我们发现一个手写C扩展要比使用Cython版本的慢了大概10%。
最底下的一行比你想象的运行的快很多。

你可以对实例代码构建多个扩展。
对于某些数组操作，最好要释放GIL，这样多个线程能并行运行。
要这样做的话，需要修改代码，使用 ``with nogil:`` 语句：

::

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef clip(double[:] a, double min, double max, double[:] out):
        if min > max:
            raise ValueError("min must be <= max")
        if a.shape[0] != out.shape[0]:
            raise ValueError("input and output arrays must be the same size")
        with nogil:
            for i in range(a.shape[0]):
                out[i] = (a[i] if a[i] < max else max) if a[i] > min else min

如果你想写一个操作二维数组的版本，下面是可以参考下：

::

    @cython.boundscheck(False)
    @cython.wraparound(False)
    cpdef clip2d(double[:,:] a, double min, double max, double[:,:] out):
        if min > max:
            raise ValueError("min must be <= max")
        for n in range(a.ndim):
            if a.shape[n] != out.shape[n]:
                raise TypeError("a and out have different shapes")
        for i in range(a.shape[0]):
            for j in range(a.shape[1]):
                if a[i,j] < min:
                    out[i,j] = min
                elif a[i,j] > max:
                    out[i,j] = max
                else:
                    out[i,j] = a[i,j]

希望读者不要忘了本节所有代码都不会绑定到某个特定数组库（比如NumPy）上面。
这样代码就更有灵活性。
不过，要注意的是如果处理数组要涉及到多维数组、切片、偏移和其他因素的时候情况会变得复杂起来。
这些内容已经超出本节范围，更多信息请参考 `PEP 3118 <http://www.python.org/dev/peps/pep-3118>`_ ，
同时 `Cython文档中关于“类型内存视图” <http://docs.cython.org/src/userguide/memoryviews.html>`_
篇也值得一读。

==============================
15.12 将函数指针转换为可调用对象
==============================

----------
问题
----------
你已经获得了一个被编译函数的内存地址，想将它转换成一个Python可调用对象，
这样的话你就可以将它作为一个扩展函数使用了。

----------
解决方案
----------
``ctypes`` 模块可被用来创建包装任意内存地址的Python可调用对象。
下面的例子演示了怎样获取C函数的原始、底层地址，以及如何将其转换为一个可调用对象：

::

    >>> import ctypes
    >>> lib = ctypes.cdll.LoadLibrary(None)
    >>> # Get the address of sin() from the C math library
    >>> addr = ctypes.cast(lib.sin, ctypes.c_void_p).value
    >>> addr
    140735505915760

    >>> # Turn the address into a callable function
    >>> functype = ctypes.CFUNCTYPE(ctypes.c_double, ctypes.c_double)
    >>> func = functype(addr)
    >>> func
    <CFunctionType object at 0x1006816d0>

    >>> # Call the resulting function
    >>> func(2)
    0.9092974268256817
    >>> func(0)
    0.0
    >>>

----------
讨论
----------
要构建一个可调用对象，你首先需要创建一个 ``CFUNCTYPE`` 实例。
``CFUNCTYPE()`` 的第一个参数是返回类型。
接下来的参数是参数类型。一旦你定义了函数类型，你就能将它包装在一个整型内存地址上来创建一个可调用对象了。
生成的对象被当做普通的可通过 ``ctypes`` 访问的函数来使用。

本节看上去可能有点神秘，偏底层一点。
但是，但是它被广泛使用于各种高级代码生成技术比如即时编译，在LLVM函数库中可以看到。

例如，下面是一个使用 ``llvmpy`` 扩展的简单例子，用来构建一个小的聚集函数，获取它的函数指针，
并将其转换为一个Python可调用对象。

::

    >>> from llvm.core import Module, Function, Type, Builder
    >>> mod = Module.new('example')
    >>> f = Function.new(mod,Type.function(Type.double(), \
                         [Type.double(), Type.double()], False), 'foo')
    >>> block = f.append_basic_block('entry')
    >>> builder = Builder.new(block)
    >>> x2 = builder.fmul(f.args[0],f.args[0])
    >>> y2 = builder.fmul(f.args[1],f.args[1])
    >>> r = builder.fadd(x2,y2)
    >>> builder.ret(r)
    <llvm.core.Instruction object at 0x10078e990>
    >>> from llvm.ee import ExecutionEngine
    >>> engine = ExecutionEngine.new(mod)
    >>> ptr = engine.get_pointer_to_function(f)
    >>> ptr
    4325863440
    >>> foo = ctypes.CFUNCTYPE(ctypes.c_double, ctypes.c_double, ctypes.c_double)(ptr)

    >>> # Call the resulting function
    >>> foo(2,3)
    13.0
    >>> foo(4,5)
    41.0
    >>> foo(1,2)
    5.0
    >>>

并不是说在这个层面犯了任何错误就会导致Python解释器挂掉。
要记得的是你是在直接跟机器级别的内存地址和本地机器码打交道，而不是Python函数。
==============================
15.13 传递NULL结尾的字符串给C函数库
==============================

----------
问题
----------
你要写一个扩展模块，需要传递一个NULL结尾的字符串给C函数库。
不过，你不是很确定怎样使用Python的Unicode字符串去实现它。

----------
解决方案
----------
许多C函数库包含一些操作NULL结尾的字符串，被声明类型为 ``char *`` .
考虑如下的C函数，我们用来做演示和测试用的：

::

    void print_chars(char *s) {
        while (*s) {
            printf("%2x ", (unsigned char) *s);

            s++;
        }
        printf("\n");
    }

此函数会打印被传进来字符串的每个字符的十六进制表示，这样的话可以很容易的进行调试了。例如：

::

    print_chars("Hello");   // Outputs: 48 65 6c 6c 6f

对于在Python中调用这样的C函数，你有几种选择。
首先，你可以通过调用 ``PyArg_ParseTuple()`` 并指定”y“转换码来限制它只能操作字节，如下：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      char *s;

      if (!PyArg_ParseTuple(args, "y", &s)) {
        return NULL;
      }
      print_chars(s);
      Py_RETURN_NONE;
    }

结果函数的使用方法如下。仔细观察嵌入了NULL字节的字符串以及Unicode支持是怎样被拒绝的：

::

    >>> print_chars(b'Hello World')
    48 65 6c 6c 6f 20 57 6f 72 6c 64
    >>> print_chars(b'Hello\x00World')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: must be bytes without null bytes, not bytes
    >>> print_chars('Hello World')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: 'str' does not support the buffer interface
    >>>

如果你想传递Unicode字符串，在 ``PyArg_ParseTuple()`` 中使用”s“格式码，如下：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      char *s;

      if (!PyArg_ParseTuple(args, "s", &s)) {
        return NULL;
      }
      print_chars(s);
      Py_RETURN_NONE;
    }

当被使用的时候，它会自动将所有字符串转换为以NULL结尾的UTF-8编码。例如：

::

    >>> print_chars('Hello World')
    48 65 6c 6c 6f 20 57 6f 72 6c 64
    >>> print_chars('Spicy Jalape\u00f1o')  # Note: UTF-8 encoding
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
    >>> print_chars('Hello\x00World')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: must be str without null characters, not str
    >>> print_chars(b'Hello World')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: must be str, not bytes
    >>>

如果因为某些原因，你要直接使用 ``PyObject *`` 而不能使用 ``PyArg_ParseTuple()`` ，
下面的例子向你展示了怎样从字节和字符串对象中检查和提取一个合适的 ``char *`` 引用：

::

    /* Some Python Object (obtained somehow) */
    PyObject *obj;

    /* Conversion from bytes */
    {
       char *s;
       s = PyBytes_AsString(o);
       if (!s) {
          return NULL;   /* TypeError already raised */
       }
       print_chars(s);
    }

    /* Conversion to UTF-8 bytes from a string */
    {
       PyObject *bytes;
       char *s;
       if (!PyUnicode_Check(obj)) {
           PyErr_SetString(PyExc_TypeError, "Expected string");
           return NULL;
       }
       bytes = PyUnicode_AsUTF8String(obj);
       s = PyBytes_AsString(bytes);
       print_chars(s);
       Py_DECREF(bytes);
    }

前面两种转换都可以确保是NULL结尾的数据，
但是它们并不检查字符串中间是否嵌入了NULL字节。
因此，如果这个很重要的话，那你需要自己去做检查了。

----------
讨论
----------
如果可能的话，你应该避免去写一些依赖于NULL结尾的字符串，因为Python并没有这个需要。
最好结合使用一个指针和长度值来处理字符串。
不过，有时候你必须去处理C语言遗留代码时就没得选择了。

尽管很容易使用，但是很容易忽视的一个问题是在 ``PyArg_ParseTuple()``
中使用“s”格式化码会有内存损耗。
但你需要使用这种转换的时候，一个UTF-8字符串被创建并永久附加在原始字符串对象上面。
如果原始字符串包含非ASCII字符的话，就会导致字符串的尺寸增到一直到被垃圾回收。例如：

::

    >>> import sys
    >>> s = 'Spicy Jalape\u00f1o'
    >>> sys.getsizeof(s)
    87
    >>> print_chars(s)     # Passing string
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
    >>> sys.getsizeof(s)   # Notice increased size
    103
    >>>

如果你在乎这个内存的损耗，你最好重写你的C扩展代码，让它使用 ``PyUnicode_AsUTF8String()`` 函数。如下：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      PyObject *o, *bytes;
      char *s;

      if (!PyArg_ParseTuple(args, "U", &o)) {
        return NULL;
      }
      bytes = PyUnicode_AsUTF8String(o);
      s = PyBytes_AsString(bytes);
      print_chars(s);
      Py_DECREF(bytes);
      Py_RETURN_NONE;
    }

通过这个修改，一个UTF-8编码的字符串根据需要被创建，然后在使用过后被丢弃。下面是修订后的效果：

::

    >>> import sys
    >>> s = 'Spicy Jalape\u00f1o'
    >>> sys.getsizeof(s)
    87
    >>> print_chars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
    >>> sys.getsizeof(s)
    87
    >>>

如果你试着传递NULL结尾字符串给ctypes包装过的函数，
要注意的是ctypes只能允许传递字节，并且它不会检查中间嵌入的NULL字节。例如：

::

    >>> import ctypes
    >>> lib = ctypes.cdll.LoadLibrary("./libsample.so")
    >>> print_chars = lib.print_chars
    >>> print_chars.argtypes = (ctypes.c_char_p,)
    >>> print_chars(b'Hello World')
    48 65 6c 6c 6f 20 57 6f 72 6c 64
    >>> print_chars(b'Hello\x00World')
    48 65 6c 6c 6f
    >>> print_chars('Hello World')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    ctypes.ArgumentError: argument 1: <class 'TypeError'>: wrong type
    >>>

如果你想传递字符串而不是字节，你需要先执行手动的UTF-8编码。例如：

::

    >>> print_chars('Hello World'.encode('utf-8'))
    48 65 6c 6c 6f 20 57 6f 72 6c 64
    >>>

对于其他扩展工具（比如Swig、Cython），
在你使用它们传递字符串给C代码时要先好好学习相应的东西了。
==============================
15.14 传递Unicode字符串给C函数库
==============================

----------
问题
----------
你要写一个扩展模块，需要将一个Python字符串传递给C的某个库函数，但是这个函数不知道该怎么处理Unicode。

----------
解决方案
----------
这里我们需要考虑很多的问题，但是最主要的问题是现存的C函数库并不理解Python的原生Unicode表示。
因此，你的挑战是将Python字符串转换为一个能被C理解的形式。

为了演示的目的，下面有两个C函数，用来操作字符串数据并输出它来调试和测试。
一个使用形式为 ``char *, int`` 形式的字节，
而另一个使用形式为 ``wchar_t *, int`` 的宽字符形式：

::

    void print_chars(char *s, int len) {
      int n = 0;

      while (n < len) {
        printf("%2x ", (unsigned char) s[n]);
        n++;
      }
      printf("\n");
    }

    void print_wchars(wchar_t *s, int len) {
      int n = 0;
      while (n < len) {
        printf("%x ", s[n]);
        n++;
      }
      printf("\n");
    }

对于面向字节的函数 ``print_chars()`` ，你需要将Python字符串转换为一个合适的编码比如UTF-8.
下面是一个这样的扩展函数例子：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      char *s;
      Py_ssize_t  len;

      if (!PyArg_ParseTuple(args, "s#", &s, &len)) {
        return NULL;
      }
      print_chars(s, len);
      Py_RETURN_NONE;
    }

对于那些需要处理机器本地 ``wchar_t`` 类型的库函数，你可以像下面这样编写扩展代码：

::

    static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
      wchar_t *s;
      Py_ssize_t  len;

      if (!PyArg_ParseTuple(args, "u#", &s, &len)) {
        return NULL;
      }
      print_wchars(s,len);
      Py_RETURN_NONE;
    }

下面是一个交互会话来演示这个函数是如何工作的：

::

    >>> s = 'Spicy Jalape\u00f1o'
    >>> print_chars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
    >>> print_wchars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 f1 6f
    >>>

仔细观察这个面向字节的函数 ``print_chars()`` 是怎样接受UTF-8编码数据的，
以及 ``print_wchars()`` 是怎样接受Unicode编码值的

----------
讨论
----------
在继续本节之前，你应该首先学习你访问的C函数库的特征。
对于很多C函数库，通常传递字节而不是字符串会比较好些。要这样做，请使用如下的转换代码：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      char *s;
      Py_ssize_t  len;

      /* accepts bytes, bytearray, or other byte-like object */
      if (!PyArg_ParseTuple(args, "y#", &s, &len)) {
        return NULL;
      }
      print_chars(s, len);
      Py_RETURN_NONE;
    }

如果你仍然还是想要传递字符串，
你需要知道Python 3可使用一个合适的字符串表示，
它并不直接映射到使用标准类型 ``char *`` 或 ``wchar_t *`` （更多细节参考PEP 393）的C函数库。
因此，要在C中表示这个字符串数据，一些转换还是必须要的。
在 ``PyArg_ParseTuple()`` 中使用"s#" 和"u#"格式化码可以安全的执行这样的转换。

不过这种转换有个缺点就是它可能会导致原始字符串对象的尺寸增大。
一旦转换过后，会有一个转换数据的复制附加到原始字符串对象上面，之后可以被重用。
你可以观察下这种效果：

::

    >>> import sys
    >>> s = 'Spicy Jalape\u00f1o'
    >>> sys.getsizeof(s)
    87
    >>> print_chars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f
    >>> sys.getsizeof(s)
    103
    >>> print_wchars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 f1 6f
    >>> sys.getsizeof(s)
    163
    >>>

对于少量的字符串对象，可能没什么影响，
但是如果你需要在扩展中处理大量的文本，你可能想避免这个损耗了。
下面是一个修订版本可以避免这种内存损耗：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      PyObject *obj, *bytes;
      char *s;
      Py_ssize_t   len;

      if (!PyArg_ParseTuple(args, "U", &obj)) {
        return NULL;
      }
      bytes = PyUnicode_AsUTF8String(obj);
      PyBytes_AsStringAndSize(bytes, &s, &len);
      print_chars(s, len);
      Py_DECREF(bytes);
      Py_RETURN_NONE;
    }

而对 ``wchar_t`` 的处理时想要避免内存损耗就更加难办了。
在内部，Python使用最高效的表示来存储字符串。
例如，只包含ASCII的字符串被存储为字节数组，
而包含范围从U+0000到U+FFFF的字符的字符串使用双字节表示。
由于对于数据的表示形式不是单一的，你不能将内部数组转换为 ``wchar_t *`` 然后期望它能正确的工作。
你应该创建一个 ``wchar_t`` 数组并向其中复制文本。
``PyArg_ParseTuple()`` 的"u#"格式码可以帮助你高效的完成它（它将复制结果附加到字符串对象上）。

如果你想避免长时间内存损耗，你唯一的选择就是复制Unicode数据懂啊一个临时的数组，
将它传递给C函数，然后回收这个数组的内存。下面是一个可能的实现：

::

    static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
      PyObject *obj;
      wchar_t *s;
      Py_ssize_t len;

      if (!PyArg_ParseTuple(args, "U", &obj)) {
        return NULL;
      }
      if ((s = PyUnicode_AsWideCharString(obj, &len)) == NULL) {
        return NULL;
      }
      print_wchars(s, len);
      PyMem_Free(s);
      Py_RETURN_NONE;
    }

在这个实现中，``PyUnicode_AsWideCharString()`` 创建一个临时的wchar_t缓冲并复制数据进去。
这个缓冲被传递给C然后被释放掉。
但是我写这本书的时候，这里可能有个bug，后面的Python问题页有介绍。

如果你知道C函数库需要的字节编码并不是UTF-8，
你可以强制Python使用扩展码来执行正确的转换，就像下面这样：

::

    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      char *s = 0;
      int   len;
      if (!PyArg_ParseTuple(args, "es#", "encoding-name", &s, &len)) {
        return NULL;
      }
      print_chars(s, len);
      PyMem_Free(s);
      Py_RETURN_NONE;
    }

最后，如果你想直接处理Unicode字符串，下面的是例子，演示了底层操作访问：

::

    static PyObject *py_print_wchars(PyObject *self, PyObject *args) {
      PyObject *obj;
      int n, len;
      int kind;
      void *data;

      if (!PyArg_ParseTuple(args, "U", &obj)) {
        return NULL;
      }
      if (PyUnicode_READY(obj) < 0) {
        return NULL;
      }

      len = PyUnicode_GET_LENGTH(obj);
      kind = PyUnicode_KIND(obj);
      data = PyUnicode_DATA(obj);

      for (n = 0; n < len; n++) {
        Py_UCS4 ch = PyUnicode_READ(kind, data, n);
        printf("%x ", ch);
      }
      printf("\n");
      Py_RETURN_NONE;
    }

在这个代码中，``PyUnicode_KIND()`` 和 ``PyUnicode_DATA()``
这两个宏和Unicode的可变宽度存储有关，这个在PEP 393中有描述。
``kind`` 变量编码底层存储（8位、16位或32位）以及指向缓存的数据指针相关的信息。
在实际情况中，你并不需要知道任何跟这些值有关的东西，
只需要在提取字符的时候将它们传给 ``PyUnicode_READ()`` 宏。

还有最后几句：当从Python传递Unicode字符串给C的时候，你应该尽量简单点。
如果有UTF-8和宽字符两种选择，请选择UTF-8.
对UTF-8的支持更加普遍一些，也不容易犯错，解释器也能支持的更好些。
最后，确保你仔细阅读了 `关于处理Unicode的相关文档 <https://docs.python.org/3/c-api/unicode.html>`_
==============================
15.15 C字符串转换为Python字符串
==============================

----------
问题
----------
怎样将C中的字符串转换为Python字节或一个字符串对象？

----------
解决方案
----------
C字符串使用一对 ``char *`` 和 ``int`` 来表示，
你需要决定字符串到底是用一个原始字节字符串还是一个Unicode字符串来表示。
字节对象可以像下面这样使用 ``Py_BuildValue()`` 来构建：

::

    char *s;     /* Pointer to C string data */
    int   len;   /* Length of data */

    /* Make a bytes object */
    PyObject *obj = Py_BuildValue("y#", s, len);

如果你要创建一个Unicode字符串，并且你知道 ``s`` 指向了UTF-8编码的数据，可以使用下面的方式：

::

    PyObject *obj = Py_BuildValue("s#", s, len);

如果 ``s`` 使用其他编码方式，那么可以像下面使用 ``PyUnicode_Decode()`` 来构建一个字符串：

::

    PyObject *obj = PyUnicode_Decode(s, len, "encoding", "errors");

    /* Examples /*
    obj = PyUnicode_Decode(s, len, "latin-1", "strict");
    obj = PyUnicode_Decode(s, len, "ascii", "ignore");

如果你恰好有一个用 ``wchar_t *, len`` 对表示的宽字符串，
有几种选择性。首先你可以使用 ``Py_BuildValue()`` ：

::

    wchar_t *w;    /* Wide character string */
    int len;       /* Length */

    PyObject *obj = Py_BuildValue("u#", w, len);

另外，你还可以使用 ``PyUnicode_FromWideChar()`` :

::

    PyObject *obj = PyUnicode_FromWideChar(w, len);

对于宽字符串，并没有对字符数据进行解析——它被假定是原始Unicode编码指针，可以被直接转换成Python。

----------
讨论
----------
将C中的字符串转换为Python字符串遵循和I/O同样的原则。
也就是说，来自C中的数据必须根据一些解码器被显式的解码为一个字符串。
通常编码格式包括ASCII、Latin-1和UTF-8.
如果你并不确定编码方式或者数据是二进制的，你最好将字符串编码成字节。
当构造一个对象的时候，Python通常会复制你提供的字符串数据。
如果有必要的话，你需要在后面去释放C字符串。
同时，为了让程序更加健壮，你应该同时使用一个指针和一个大小值，
而不是依赖NULL结尾数据来创建字符串。

==============================
15.16 不确定编码格式的C字符串
==============================

----------
问题
----------
你要在C和Python直接来回转换字符串，但是C中的编码格式并不确定。
例如，可能C中的数据期望是UTF-8，但是并没有强制它必须是。
你想编写代码来以一种优雅的方式处理这些不合格数据，这样就不会让Python奔溃或者破坏进程中的字符串数据。

----------
解决方案
----------
下面是一些C的数据和一个函数来演示这个问题：

::

    /* Some dubious string data (malformed UTF-8) */
    const char *sdata = "Spicy Jalape\xc3\xb1o\xae";
    int slen = 16;

    /* Output character data */
    void print_chars(char *s, int len) {
      int n = 0;
      while (n < len) {
        printf("%2x ", (unsigned char) s[n]);
        n++;
      }
      printf("\n");
    }

在这个代码中，字符串 ``sdata`` 包含了UTF-8和不合格数据。
不过，如果用户在C中调用 ``print_chars(sdata, slen)`` ，它缺能正常工作。
现在假设你想将 ``sdata`` 的内容转换为一个Python字符串。
进一步假设你在后面还想通过一个扩展将那个字符串传个 ``print_chars()`` 函数。
下面是一种用来保护原始数据的方法，就算它编码有问题。

::

    /* Return the C string back to Python */
    static PyObject *py_retstr(PyObject *self, PyObject *args) {
      if (!PyArg_ParseTuple(args, "")) {
        return NULL;
      }
      return PyUnicode_Decode(sdata, slen, "utf-8", "surrogateescape");
    }

    /* Wrapper for the print_chars() function */
    static PyObject *py_print_chars(PyObject *self, PyObject *args) {
      PyObject *obj, *bytes;
      char *s = 0;
      Py_ssize_t   len;

      if (!PyArg_ParseTuple(args, "U", &obj)) {
        return NULL;
      }

      if ((bytes = PyUnicode_AsEncodedString(obj,"utf-8","surrogateescape"))
            == NULL) {
        return NULL;
      }
      PyBytes_AsStringAndSize(bytes, &s, &len);
      print_chars(s, len);
      Py_DECREF(bytes);
      Py_RETURN_NONE;
    }

如果你在Python中尝试这些函数，下面是运行效果：

::

    >>> s = retstr()
    >>> s
    'Spicy Jalapeño\udcae'
    >>> print_chars(s)
    53 70 69 63 79 20 4a 61 6c 61 70 65 c3 b1 6f ae
    >>>

仔细观察结果你会发现，不合格字符串被编码到一个Python字符串中，并且并没有产生错误，
并且当它被回传给C的时候，被转换为和之前原始C字符串一样的字节。

----------
讨论
----------
本节展示了在扩展模块中处理字符串时会配到的一个棘手又很恼火的问题。
也就是说，在扩展中的C字符串可能不会严格遵循Python所期望的Unicode编码/解码规则。
因此，很可能一些不合格C数据传递到Python中去。
一个很好的例子就是涉及到底层系统调用比如文件名这样的字符串。
例如，如果一个系统调用返回给解释器一个损坏的字符串，不能被正确解码的时候会怎样呢？

一般来讲，可以通过制定一些错误策略比如严格、忽略、替代或其他类似的来处理Unicode错误。
不过，这些策略的一个缺点是它们永久性破坏了原始字符串的内容。
例如，如果例子中的不合格数据使用这些策略之一解码，你会得到下面这样的结果：

::

    >>> raw = b'Spicy Jalape\xc3\xb1o\xae'
    >>> raw.decode('utf-8','ignore')
    'Spicy Jalapeño'
    >>> raw.decode('utf-8','replace')
    'Spicy Jalapeño?'
    >>>

``surrogateescape`` 错误处理策略会将所有不可解码字节转化为一个代理对的低位字节（\udcXX中XX是原始字节值）。
例如：

::

    >>> raw.decode('utf-8','surrogateescape')
    'Spicy Jalapeño\udcae'
    >>>

单独的低位代理字符比如 ``\udcae`` 在Unicode中是非法的。
因此，这个字符串就是一个非法表示。
实际上，如果你将它传个一个执行输出的函数，你会得到一个错误：

::

    >>> s = raw.decode('utf-8', 'surrogateescape')
    >>> print(s)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    UnicodeEncodeError: 'utf-8' codec can't encode character '\udcae'
    in position 14: surrogates not allowed
    >>>

然而，允许代理转换的关键点在于从C传给Python又回传给C的不合格字符串不会有任何数据丢失。
当这个字符串再次使用 ``surrogateescape`` 编码时，代理字符会转换回原始字节。例如：

::

    >>> s
    'Spicy Jalapeño\udcae'
    >>> s.encode('utf-8','surrogateescape')
    b'Spicy Jalape\xc3\xb1o\xae'
    >>>

作为一般准则，最好避免代理编码——如果你正确的使用了编码，那么你的代码就值得信赖。
不过，有时候确实会出现你并不能控制数据编码并且你又不能忽略或替换坏数据，因为其他函数可能会用到它。
那么就可以使用本节的技术了。

最后一点要注意的是，Python中许多面向系统的函数，特别是和文件名、环境变量和命令行参数相关的
都会使用代理编码。例如，如果你使用像 ``os.listdir()`` 这样的函数，
传入一个包含了不可解码文件名的目录的话，它会返回一个代理转换后的字符串。
参考5.15的相关章节。

`PEP 383 <https://www.python.org/dev/peps/pep-0383/>`_
中有更多关于本机提到的以及和surrogateescape错误处理相关的信息。
==============================
15.17 传递文件名给C扩展
==============================

----------
问题
----------
你需要向C库函数传递文件名，但是需要确保文件名根据系统期望的文件名编码方式编码过。

----------
解决方案
----------
写一个接受一个文件名为参数的扩展函数，如下这样：

::

    static PyObject *py_get_filename(PyObject *self, PyObject *args) {
      PyObject *bytes;
      char *filename;
      Py_ssize_t len;
      if (!PyArg_ParseTuple(args,"O&", PyUnicode_FSConverter, &bytes)) {
        return NULL;
      }
      PyBytes_AsStringAndSize(bytes, &filename, &len);
      /* Use filename */
      ...

      /* Cleanup and return */
      Py_DECREF(bytes)
      Py_RETURN_NONE;
    }

如果你已经有了一个 ``PyObject *`` ，希望将其转换成一个文件名，可以像下面这样做：

::

    PyObject *obj;    /* Object with the filename */
    PyObject *bytes;
    char *filename;
    Py_ssize_t len;

    bytes = PyUnicode_EncodeFSDefault(obj);
    PyBytes_AsStringAndSize(bytes, &filename, &len);
    /* Use filename */
    ...

    /* Cleanup */
    Py_DECREF(bytes);

    If you need to return a filename back to Python, use the following code:

    /* Turn a filename into a Python object */

    char *filename;       /* Already set */
    int   filename_len;   /* Already set */

    PyObject *obj = PyUnicode_DecodeFSDefaultAndSize(filename, filename_len);

----------
讨论
----------
以可移植方式来处理文件名是一个很棘手的问题，最后交由Python来处理。
如果你在扩展代码中使用本节的技术，文件名的处理方式和和Python中是一致的。
包括编码/界面字节，处理坏字符，代理转换和其他复杂情况。

==============================
15.18 传递已打开的文件给C扩展
==============================

----------
问题
----------
你在Python中有一个打开的文件对象，但是需要将它传给要使用这个文件的C扩展。

----------
解决方案
----------
要将一个文件转换为一个整型的文件描述符，使用 ``PyFile_FromFd()`` ，如下：

::

    PyObject *fobj;     /* File object (already obtained somehow) */
    int fd = PyObject_AsFileDescriptor(fobj);
    if (fd < 0) {
       return NULL;
    }

结果文件描述符是通过调用 ``fobj`` 中的 ``fileno()`` 方法获得的。
因此，任何以这种方式暴露给一个描述器的对象都适用（比如文件、套接字等）。
一旦你有了这个描述器，它就能被传递给多个低级的可处理文件的C函数。

如果你需要转换一个整型文件描述符为一个Python对象，适用下面的 ``PyFile_FromFd()`` :

::

    int fd;     /* Existing file descriptor (already open) */
    PyObject *fobj = PyFile_FromFd(fd, "filename","r",-1,NULL,NULL,NULL,1);

``PyFile_FromFd()`` 的参数对应内置的 ``open()`` 函数。
NULL表示编码、错误和换行参数使用默认值。

----------
讨论
----------
如果将Python中的文件对象传给C，有一些注意事项。
首先，Python通过 ``io`` 模块执行自己的I/O缓冲。
在传递任何类型的文件描述符给C之前，你都要首先在相应文件对象上刷新I/O缓冲。
不然的话，你会打乱文件系统上面的数据。

其次，你需要特别注意文件的归属者以及关闭文件的职责。
如果一个文件描述符被传给C，但是在Python中还在被使用着，你需要确保C没有意外的关闭它。
类似的，如果一个文件描述符被转换为一个Python文件对象，你需要清楚谁应该去关闭它。
``PyFile_FromFd()`` 的最后一个参数被设置成1，用来指出Python应该关闭这个文件。

如果你需要从C标准I/O库中使用如　``fdopen()`` 函数来创建不同类型的文件对象比如 ``FILE *`` 对象，
你需要特别小心了。这样做会在I/O堆栈中产生两个完全不同的I/O缓冲层
（一个是来自Python的 ``io`` 模块，另一个来自C的 ``stdio`` ）。
像C中的 ``fclose()`` 会关闭Python要使用的文件。
如果让你选的话，你应该会选择去构建一个扩展代码来处理底层的整型文件描述符，
而不是使用来自<stdio.h>的高层抽象功能。
==============================
15.19 从C语言中读取类文件对象
==============================

----------
问题
----------
你要写C扩展来读取来自任何Python类文件对象中的数据（比如普通文件、StringIO对象等）。

----------
解决方案
----------
要读取一个类文件对象的数据，你需要重复调用 ``read()`` 方法，然后正确的解码获得的数据。

下面是一个C扩展函数例子，仅仅只是读取一个类文件对象中的所有数据并将其输出到标准输出：

::

    #define CHUNK_SIZE 8192

    /* Consume a "file-like" object and write bytes to stdout */
    static PyObject *py_consume_file(PyObject *self, PyObject *args) {
      PyObject *obj;
      PyObject *read_meth;
      PyObject *result = NULL;
      PyObject *read_args;

      if (!PyArg_ParseTuple(args,"O", &obj)) {
        return NULL;
      }

      /* Get the read method of the passed object */
      if ((read_meth = PyObject_GetAttrString(obj, "read")) == NULL) {
        return NULL;
      }

      /* Build the argument list to read() */
      read_args = Py_BuildValue("(i)", CHUNK_SIZE);
      while (1) {
        PyObject *data;
        PyObject *enc_data;
        char *buf;
        Py_ssize_t len;

        /* Call read() */
        if ((data = PyObject_Call(read_meth, read_args, NULL)) == NULL) {
          goto final;
        }

        /* Check for EOF */
        if (PySequence_Length(data) == 0) {
          Py_DECREF(data);
          break;
        }

        /* Encode Unicode as Bytes for C */
        if ((enc_data=PyUnicode_AsEncodedString(data,"utf-8","strict"))==NULL) {
          Py_DECREF(data);
          goto final;
        }

        /* Extract underlying buffer data */
        PyBytes_AsStringAndSize(enc_data, &buf, &len);

        /* Write to stdout (replace with something more useful) */
        write(1, buf, len);

        /* Cleanup */
        Py_DECREF(enc_data);
        Py_DECREF(data);
      }
      result = Py_BuildValue("");

     final:
      /* Cleanup */
      Py_DECREF(read_meth);
      Py_DECREF(read_args);
      return result;
    }

要测试这个代码，先构造一个类文件对象比如一个StringIO实例，然后传递进来：

::

    >>> import io
    >>> f = io.StringIO('Hello\nWorld\n')
    >>> import sample
    >>> sample.consume_file(f)
    Hello
    World
    >>>

----------
讨论
----------
和普通系统文件不同的是，一个类文件对象并不需要使用低级文件描述符来构建。
因此，你不能使用普通的C库函数来访问它。
你需要使用Python的C API来像普通文件类似的那样操作类文件对象。

在我们的解决方案中，``read()`` 方法从被传递的对象中提取出来。
一个参数列表被构建然后不断的被传给 ``PyObject_Call()`` 来调用这个方法。
要检查文件末尾（EOF），使用了 ``PySequence_Length()`` 来查看是否返回对象长度为0.

对于所有的I/O操作，你需要关注底层的编码格式，还有字节和Unicode之前的区别。
本节演示了如何以文本模式读取一个文件并将结果文本解码为一个字节编码，这样在C中就可以使用它了。
如果你想以二进制模式读取文件，只需要修改一点点即可，例如：
::

    ...
    /* Call read() */
    if ((data = PyObject_Call(read_meth, read_args, NULL)) == NULL) {
      goto final;
    }

    /* Check for EOF */
    if (PySequence_Length(data) == 0) {
      Py_DECREF(data);
      break;
    }
    if (!PyBytes_Check(data)) {
      Py_DECREF(data);
      PyErr_SetString(PyExc_IOError, "File must be in binary mode");
      goto final;
    }

    /* Extract underlying buffer data */
    PyBytes_AsStringAndSize(data, &buf, &len);
    ...

本节最难的地方在于如何进行正确的内存管理。
当处理 ``PyObject *`` 变量的时候，需要注意管理引用计数以及在不需要的变量的时候清理它们的值。
对 ``Py_DECREF()`` 的调用就是来做这个的。

本节代码以一种通用方式编写，因此他也能适用于其他的文件操作，比如写文件。
例如，要写数据，只需要获取类文件对象的 ``write()`` 方法，将数据转换为合适的Python对象
（字节或Unicode），然后调用该方法将输入写入到文件。

最后，尽管类文件对象通常还提供其他方法（比如readline(), read_info()），
我们最好只使用基本的 ``read()`` 和 ``write()`` 方法。
在写C扩展的时候，能简单就尽量简单。
==============================
15.20 处理C语言中的可迭代对象
==============================

----------
问题
----------
你想写C扩展代码处理来自任何可迭代对象如列表、元组、文件或生成器中的元素。

----------
解决方案
----------
下面是一个C扩展函数例子，演示了怎样处理可迭代对象中的元素：

::

    static PyObject *py_consume_iterable(PyObject *self, PyObject *args) {
      PyObject *obj;
      PyObject *iter;
      PyObject *item;

      if (!PyArg_ParseTuple(args, "O", &obj)) {
        return NULL;
      }
      if ((iter = PyObject_GetIter(obj)) == NULL) {
        return NULL;
      }
      while ((item = PyIter_Next(iter)) != NULL) {
        /* Use item */
        ...
        Py_DECREF(item);
      }

      Py_DECREF(iter);
      return Py_BuildValue("");
    }

----------
讨论
----------
本节中的代码和Python中对应代码类似。
``PyObject_GetIter()`` 的调用和调用 ``iter()`` 一样可获得一个迭代器。
``PyIter_Next()`` 函数调用 ``next`` 方法返回下一个元素或NULL(如果没有元素了)。
要注意正确的内存管理—— ``Py_DECREF()`` 需要同时在产生的元素和迭代器对象本身上同时被调用，
以避免出现内存泄露。
==============================
15.21 诊断分段错误
==============================

----------
问题
----------
解释器因为某个分段错误、总线错误、访问越界或其他致命错误而突然间奔溃。
你想获得Python堆栈信息，从而找出在发生错误的时候你的程序运行点。

----------
解决方案
----------
``faulthandler`` 模块能被用来帮你解决这个问题。
在你的程序中引入下列代码：

.. code-block:: python

    import faulthandler
    faulthandler.enable()

另外还可以像下面这样使用 ``-Xfaulthandler`` 来运行Python：

::

    bash % python3 -Xfaulthandler program.py

最后，你可以设置 ``PYTHONFAULTHANDLER`` 环境变量。
开启faulthandler后，在C扩展中的致命错误会导致一个Python错误堆栈被打印出来。例如：

::

    Fatal Python error: Segmentation fault

    Current thread 0x00007fff71106cc0:
      File "example.py", line 6 in foo
      File "example.py", line 10 in bar
      File "example.py", line 14 in spam
      File "example.py", line 19 in <module>
    Segmentation fault

尽管这个并不能告诉你C代码中哪里出错了，但是至少能告诉你Python里面哪里有错。

----------
讨论
----------
faulthandler会在Python代码执行出错的时候向你展示跟踪信息。
至少，它会告诉你出错时被调用的最顶级扩展函数是哪个。
在pdb和其他Python调试器的帮助下，你就能追根溯源找到错误所在的位置了。

faulthandler不会告诉你任何C语言中的错误信息。
因此，你需要使用传统的C调试器，比如gdb。
不过，在faulthandler追踪信息可以让你去判断从哪里着手。
还要注意的是在C中某些类型的错误可能不太容易恢复。
例如，如果一个C扩展丢弃了程序堆栈信息，它会让faulthandler不可用，
那么你也得不到任何输出（除了程序奔溃外）。


