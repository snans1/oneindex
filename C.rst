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
