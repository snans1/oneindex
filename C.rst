
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
