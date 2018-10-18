===============================
1.1 解压序列赋值给多个变量
===============================

----------
问题
----------
现在有一个包含 N 个元素的元组或者是序列，怎样将它里面的值解压后同时赋值给 N 个变量？

----------
解决方案
----------
任何的序列（或者是可迭代对象）可以通过一个简单的赋值语句解压并赋值给多个变量。
唯一的前提就是变量的数量必须跟序列元素的数量是一样的。

代码示例：

.. code-block:: python

    >>> p = (4, 5)
    >>> x, y = p
    >>> x
    4
    >>> y
    5
    >>>
    >>> data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
    >>> name, shares, price, date = data
    >>> name
    'ACME'
    >>> date
    (2012, 12, 21)
    >>> name, shares, price, (year, mon, day) = data
    >>> name
    'ACME'
    >>> year
    2012
    >>> mon
    12
    >>> day
    21
    >>>

如果变量个数和序列元素的个数不匹配，会产生一个异常。

代码示例：

.. code-block:: python

    >>> p = (4, 5)
    >>> x, y, z = p
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    ValueError: need more than 2 values to unpack
    >>>

----------
讨论
----------
实际上，这种解压赋值可以用在任何可迭代对象上面，而不仅仅是列表或者元组。
包括字符串，文件对象，迭代器和生成器。

代码示例：

.. code-block:: python

    >>> s = 'Hello'
    >>> a, b, c, d, e = s
    >>> a
    'H'
    >>> b
    'e'
    >>> e
    'o'
    >>>

有时候，你可能只想解压一部分，丢弃其他的值。对于这种情况 Python 并没有提供特殊的语法。
但是你可以使用任意变量名去占位，到时候丢掉这些变量就行了。

代码示例：

.. code-block:: python

    >>> data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
    >>> _, shares, price, _ = data
    >>> shares
    50
    >>> price
    91.1
    >>>

你必须保证你选用的那些占位变量名在其他地方没被使用到。
================================
1.2 解压可迭代对象赋值给多个变量
================================

----------
问题
----------
如果一个可迭代对象的元素个数超过变量个数时，会抛出一个 ``ValueError`` 。
那么怎样才能从这个可迭代对象中解压出 N 个元素出来？

----------
解决方案
----------
Python 的星号表达式可以用来解决这个问题。比如，你在学习一门课程，在学期末的时候，
你想统计下家庭作业的平均成绩，但是排除掉第一个和最后一个分数。如果只有四个分数，你可能就直接去简单的手动赋值，
但如果有 24 个呢？这时候星号表达式就派上用场了：

.. code-block:: python

    def drop_first_last(grades):
        first, *middle, last = grades
        return avg(middle)

另外一种情况，假设你现在有一些用户的记录列表，每条记录包含一个名字、邮件，接着就是不确定数量的电话号码。
你可以像下面这样分解这些记录：

.. code-block:: python

    >>> record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212')
    >>> name, email, *phone_numbers = record
    >>> name
    'Dave'
    >>> email
    'dave@example.com'
    >>> phone_numbers
    ['773-555-1212', '847-555-1212']
    >>>
值得注意的是上面解压出的 ``phone_numbers`` 变量永远都是列表类型，不管解压的电话号码数量是多少（包括 0 个）。
所以，任何使用到 ``phone_numbers`` 变量的代码就不需要做多余的类型检查去确认它是否是列表类型了。

星号表达式也能用在列表的开始部分。比如，你有一个公司前 8 个月销售数据的序列，
但是你想看下最近一个月数据和前面 7 个月的平均值的对比。你可以这样做：

.. code-block:: python

    *trailing_qtrs, current_qtr = sales_record
    trailing_avg = sum(trailing_qtrs) / len(trailing_qtrs)
    return avg_comparison(trailing_avg, current_qtr)

下面是在 Python 解释器中执行的结果：

.. code-block:: python

    >>> *trailing, current = [10, 8, 7, 1, 9, 5, 10, 3]
    >>> trailing
    [10, 8, 7, 1, 9, 5, 10]
    >>> current
    3

----------
讨论
----------
扩展的迭代解压语法是专门为解压不确定个数或任意个数元素的可迭代对象而设计的。
通常，这些可迭代对象的元素结构有确定的规则（比如第 1 个元素后面都是电话号码），
星号表达式让开发人员可以很容易的利用这些规则来解压出元素来。
而不是通过一些比较复杂的手段去获取这些关联的元素值。

值得注意的是，星号表达式在迭代元素为可变长元组的序列时是很有用的。
比如，下面是一个带有标签的元组序列：

.. code-block:: python

    records = [
        ('foo', 1, 2),
        ('bar', 'hello'),
        ('foo', 3, 4),
    ]

    def do_foo(x, y):
        print('foo', x, y)

    def do_bar(s):
        print('bar', s)

    for tag, *args in records:
        if tag == 'foo':
            do_foo(*args)
        elif tag == 'bar':
            do_bar(*args)

星号解压语法在字符串操作的时候也会很有用，比如字符串的分割。

代码示例：

.. code-block:: python

    >>> line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
    >>> uname, *fields, homedir, sh = line.split(':')
    >>> uname
    'nobody'
    >>> homedir
    '/var/empty'
    >>> sh
    '/usr/bin/false'
    >>>

有时候，你想解压一些元素后丢弃它们，你不能简单就使用 ``*`` ，
但是你可以使用一个普通的废弃名称，比如 ``_`` 或者 ``ign`` （ignore）。

代码示例：

.. code-block:: python

    >>> record = ('ACME', 50, 123.45, (12, 18, 2012))
    >>> name, *_, (*_, year) = record
    >>> name
    'ACME'
    >>> year
    2012
    >>>

在很多函数式语言中，星号解压语法跟列表处理有许多相似之处。比如，如果你有一个列表，
你可以很容易的将它分割成前后两部分：

.. code-block:: python

    >>> items = [1, 10, 7, 4, 5, 9]
    >>> head, *tail = items
    >>> head
    1
    >>> tail
    [10, 7, 4, 5, 9]
    >>>

如果你够聪明的话，还能用这种分割语法去巧妙的实现递归算法。比如：

.. code-block:: python

    >>> def sum(items):
    ...     head, *tail = items
    ...     return head + sum(tail) if tail else head
    ...
    >>> sum(items)
    36
    >>>

然后，由于语言层面的限制，递归并不是 Python 擅长的。
因此，最后那个递归演示仅仅是个好奇的探索罢了，对这个不要太认真了。
================================
1.3 保留最后 N 个元素
================================

----------
问题
----------
在迭代操作或者其他操作的时候，怎样只保留最后有限几个元素的历史记录？

----------
解决方案
----------
保留有限历史记录正是 ``collections.deque`` 大显身手的时候。比如，下面的代码在多行上面做简单的文本匹配，
并返回匹配所在行的最后N行：

.. code-block:: python

    from collections import deque


    def search(lines, pattern, history=5):
        previous_lines = deque(maxlen=history)
        for line in lines:
            if pattern in line:
                yield line, previous_lines
            previous_lines.append(line)

    # Example use on a file
    if __name__ == '__main__':
        with open(r'../../cookbook/somefile.txt') as f:
            for line, prevlines in search(f, 'python', 5):
                for pline in prevlines:
                    print(pline, end='')
                print(line, end='')
                print('-' * 20)

----------
讨论
----------
我们在写查询元素的代码时，通常会使用包含 ``yield`` 表达式的生成器函数，也就是我们上面示例代码中的那样。
这样可以将搜索过程代码和使用搜索结果代码解耦。如果你还不清楚什么是生成器，请参看 4.3 节。

使用 ``deque(maxlen=N)`` 构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候，
最老的元素会自动被移除掉。

代码示例：

.. code-block:: python

    >>> q = deque(maxlen=3)
    >>> q.append(1)
    >>> q.append(2)
    >>> q.append(3)
    >>> q
    deque([1, 2, 3], maxlen=3)
    >>> q.append(4)
    >>> q
    deque([2, 3, 4], maxlen=3)
    >>> q.append(5)
    >>> q
    deque([3, 4, 5], maxlen=3)

尽管你也可以手动在一个列表上实现这一的操作（比如增加、删除等等）。但是这里的队列方案会更加优雅并且运行得更快些。

更一般的， ``deque`` 类可以被用在任何你只需要一个简单队列数据结构的场合。
如果你不设置最大队列大小，那么就会得到一个无限大小队列，你可以在队列的两端执行添加和弹出元素的操作。

代码示例：

.. code-block:: python

    >>> q = deque()
    >>> q.append(1)
    >>> q.append(2)
    >>> q.append(3)
    >>> q
    deque([1, 2, 3])
    >>> q.appendleft(4)
    >>> q
    deque([4, 1, 2, 3])
    >>> q.pop()
    3
    >>> q
    deque([4, 1, 2])
    >>> q.popleft()
    4

在队列两端插入或删除元素时间复杂度都是 ``O(1)`` ，区别于列表，在列表的开头插入或删除元素的时间复杂度为 ``O(N)`` 。
================================
1.4 查找最大或最小的 N 个元素
================================

----------
问题
----------
怎样从一个集合中获得最大或者最小的 N 个元素列表？

----------
解决方案
----------
heapq 模块有两个函数：``nlargest()`` 和 ``nsmallest()`` 可以完美解决这个问题。

.. code-block:: python

    import heapq
    nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
    print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
    print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]

两个函数都能接受一个关键字参数，用于更复杂的数据结构中：

.. code-block:: python

    portfolio = [
        {'name': 'IBM', 'shares': 100, 'price': 91.1},
        {'name': 'AAPL', 'shares': 50, 'price': 543.22},
        {'name': 'FB', 'shares': 200, 'price': 21.09},
        {'name': 'HPQ', 'shares': 35, 'price': 31.75},
        {'name': 'YHOO', 'shares': 45, 'price': 16.35},
        {'name': 'ACME', 'shares': 75, 'price': 115.65}
    ]
    cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
    expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])

译者注：上面代码在对每个元素进行对比的时候，会以 ``price`` 的值进行比较。

----------
讨论
----------
如果你想在一个集合中查找最小或最大的 N 个元素，并且 N 小于集合元素数量，那么这些函数提供了很好的性能。
因为在底层实现里面，首先会先将集合数据进行堆排序后放入一个列表中：

.. code-block:: python

    >>> nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
    >>> import heapq
    >>> heap = list(nums)
    >>> heapq.heapify(heap)
    >>> heap
    [-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8]
    >>>

堆数据结构最重要的特征是 ``heap[0]`` 永远是最小的元素。并且剩余的元素可以很容易的通过调用 ``heapq.heappop()`` 方法得到，
该方法会先将第一个元素弹出来，然后用下一个最小的元素来取代被弹出元素（这种操作时间复杂度仅仅是 O(log N)，N 是堆大小）。
比如，如果想要查找最小的 3 个元素，你可以这样做：

.. code-block:: python

    >>> heapq.heappop(heap)
    -4
    >>> heapq.heappop(heap)
    1
    >>> heapq.heappop(heap)
    2

当要查找的元素个数相对比较小的时候，函数 ``nlargest()`` 和 ``nsmallest()`` 是很合适的。
如果你仅仅想查找唯一的最小或最大（N=1）的元素的话，那么使用 ``min()`` 和 ``max()`` 函数会更快些。
类似的，如果 N 的大小和集合大小接近的时候，通常先排序这个集合然后再使用切片操作会更快点
（ ``sorted(items)[:N]`` 或者是 ``sorted(items)[-N:]`` ）。
需要在正确场合使用函数 ``nlargest()`` 和 ``nsmallest()`` 才能发挥它们的优势
（如果 N 快接近集合大小了，那么使用排序操作会更好些）。

尽管你没有必要一定使用这里的方法，但是堆数据结构的实现是一个很有趣并且值得你深入学习的东西。
基本上只要是数据结构和算法书籍里面都会有提及到。
``heapq`` 模块的官方文档里面也详细的介绍了堆数据结构底层的实现细节。
================================
1.5 实现一个优先级队列
================================

----------
问题
----------
怎样实现一个按优先级排序的队列？ 并且在这个队列上面每次 pop 操作总是返回优先级最高的那个元素

----------
解决方案
----------
下面的类利用 ``heapq`` 模块实现了一个简单的优先级队列：

.. code-block:: python

    import heapq

    class PriorityQueue:
        def __init__(self):
            self._queue = []
            self._index = 0

        def push(self, item, priority):
            heapq.heappush(self._queue, (-priority, self._index, item))
            self._index += 1

        def pop(self):
            return heapq.heappop(self._queue)[-1]


下面是它的使用方式：

.. code-block:: python

    >>> class Item:
    ...     def __init__(self, name):
    ...         self.name = name
    ...     def __repr__(self):
    ...         return 'Item({!r})'.format(self.name)
    ...
    >>> q = PriorityQueue()
    >>> q.push(Item('foo'), 1)
    >>> q.push(Item('bar'), 5)
    >>> q.push(Item('spam'), 4)
    >>> q.push(Item('grok'), 1)
    >>> q.pop()
    Item('bar')
    >>> q.pop()
    Item('spam')
    >>> q.pop()
    Item('foo')
    >>> q.pop()
    Item('grok')
    >>>

仔细观察可以发现，第一个 ``pop()`` 操作返回优先级最高的元素。
另外注意到如果两个有着相同优先级的元素（ ``foo`` 和 ``grok`` ），pop 操作按照它们被插入到队列的顺序返回的。

----------
讨论
----------
这一小节我们主要关注 ``heapq`` 模块的使用。
函数 ``heapq.heappush()`` 和 ``heapq.heappop()`` 分别在队列 ``_queue`` 上插入和删除第一个元素，
并且队列 ``_queue`` 保证第一个元素拥有最高优先级（ 1.4 节已经讨论过这个问题）。
``heappop()`` 函数总是返回"最小的"的元素，这就是保证队列pop操作返回正确元素的关键。
另外，由于 push 和 pop 操作时间复杂度为 O(log N)，其中 N 是堆的大小，因此就算是 N 很大的时候它们运行速度也依旧很快。

在上面代码中，队列包含了一个 ``(-priority, index, item)`` 的元组。
优先级为负数的目的是使得元素按照优先级从高到低排序。
这个跟普通的按优先级从低到高排序的堆排序恰巧相反。

``index`` 变量的作用是保证同等优先级元素的正确排序。
通过保存一个不断增加的 ``index`` 下标变量，可以确保元素按照它们插入的顺序排序。
而且， ``index`` 变量也在相同优先级元素比较的时候起到重要作用。

为了阐明这些，先假定 ``Item`` 实例是不支持排序的：

.. code-block:: python

    >>> a = Item('foo')
    >>> b = Item('bar')
    >>> a < b
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: unorderable types: Item() < Item()
    >>>

如果你使用元组 ``(priority, item)`` ，只要两个元素的优先级不同就能比较。
但是如果两个元素优先级一样的话，那么比较操作就会跟之前一样出错：

.. code-block:: python

    >>> a = (1, Item('foo'))
    >>> b = (5, Item('bar'))
    >>> a < b
    True
    >>> c = (1, Item('grok'))
    >>> a < c
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: unorderable types: Item() < Item()
    >>>

通过引入另外的 ``index`` 变量组成三元组 ``(priority, index, item)`` ，就能很好的避免上面的错误，
因为不可能有两个元素有相同的 ``index`` 值。Python 在做元组比较时候，如果前面的比较已经可以确定结果了，
后面的比较操作就不会发生了：

.. code-block:: python

    >>> a = (1, 0, Item('foo'))
    >>> b = (5, 1, Item('bar'))
    >>> c = (1, 2, Item('grok'))
    >>> a < b
    True
    >>> a < c
    True
    >>>

如果你想在多个线程中使用同一个队列，那么你需要增加适当的锁和信号量机制。
可以查看 12.3 小节的例子演示是怎样做的。

``heapq`` 模块的官方文档有更详细的例子程序以及对于堆理论及其实现的详细说明。
================================
1.6 字典中的键映射多个值
================================

----------
问题
----------
怎样实现一个键对应多个值的字典（也叫 ``multidict``）？

----------
解决方案
----------
一个字典就是一个键对应一个单值的映射。如果你想要一个键映射多个值，那么你就需要将这多个值放到另外的容器中，
比如列表或者集合里面。比如，你可以像下面这样构造这样的字典：

.. code-block:: python

    d = {
        'a' : [1, 2, 3],
        'b' : [4, 5]
    }
    e = {
        'a' : {1, 2, 3},
        'b' : {4, 5}
    }

选择使用列表还是集合取决于你的实际需求。如果你想保持元素的插入顺序就应该使用列表，
如果想去掉重复元素就使用集合（并且不关心元素的顺序问题）。

你可以很方便的使用 ``collections`` 模块中的 ``defaultdict`` 来构造这样的字典。
``defaultdict`` 的一个特征是它会自动初始化每个 ``key`` 刚开始对应的值，所以你只需要关注添加元素操作了。比如：

.. code-block:: python

    from collections import defaultdict

    d = defaultdict(list)
    d['a'].append(1)
    d['a'].append(2)
    d['b'].append(4)

    d = defaultdict(set)
    d['a'].add(1)
    d['a'].add(2)
    d['b'].add(4)

需要注意的是， ``defaultdict`` 会自动为将要访问的键（就算目前字典中并不存在这样的键）创建映射实体。
如果你并不需要这样的特性，你可以在一个普通的字典上使用 ``setdefault()`` 方法来代替。比如：

.. code-block:: python

    d = {} # 一个普通的字典
    d.setdefault('a', []).append(1)
    d.setdefault('a', []).append(2)
    d.setdefault('b', []).append(4)

但是很多程序员觉得 ``setdefault()`` 用起来有点别扭。因为每次调用都得创建一个新的初始值的实例（例子程序中的空列表 ``[]`` ）。

----------
讨论
----------
一般来讲，创建一个多值映射字典是很简单的。但是，如果你选择自己实现的话，那么对于值的初始化可能会有点麻烦，
你可能会像下面这样来实现：

.. code-block:: python

    d = {}
    for key, value in pairs:
        if key not in d:
            d[key] = []
        d[key].append(value)

如果使用 ``defaultdict`` 的话代码就更加简洁了：

.. code-block:: python

    d = defaultdict(list)
    for key, value in pairs:
        d[key].append(value)

这一小节所讨论的问题跟数据处理中的记录归类问题有大的关联。可以参考 1.15 小节的例子。
================================
1.7 字典排序
================================

----------
问题
----------
你想创建一个字典，并且在迭代或序列化这个字典的时候能够控制元素的顺序。

----------
解决方案
----------
为了能控制一个字典中元素的顺序，你可以使用 ``collections`` 模块中的 ``OrderedDict`` 类。
在迭代操作的时候它会保持元素被插入时的顺序，示例如下：

.. code-block:: python

    from collections import OrderedDict

    d = OrderedDict()
    d['foo'] = 1
    d['bar'] = 2
    d['spam'] = 3
    d['grok'] = 4
    # Outputs "foo 1", "bar 2", "spam 3", "grok 4"
    for key in d:
        print(key, d[key])

当你想要构建一个将来需要序列化或编码成其他格式的映射的时候， ``OrderedDict`` 是非常有用的。
比如，你想精确控制以 JSON 编码后字段的顺序，你可以先使用 ``OrderedDict`` 来构建这样的数据：

.. code-block:: python

    >>> import json
    >>> json.dumps(d)
    '{"foo": 1, "bar": 2, "spam": 3, "grok": 4}'
    >>>

----------
讨论
----------
``OrderedDict`` 内部维护着一个根据键插入顺序排序的双向链表。每次当一个新的元素插入进来的时候，
它会被放到链表的尾部。对于一个已经存在的键的重复赋值不会改变键的顺序。

需要注意的是，一个 ``OrderedDict`` 的大小是一个普通字典的两倍，因为它内部维护着另外一个链表。
所以如果你要构建一个需要大量 ``OrderedDict`` 实例的数据结构的时候（比如读取 100,000 行 CSV 数据到一个 ``OrderedDict`` 列表中去），
那么你就得仔细权衡一下是否使用 ``OrderedDict`` 带来的好处要大过额外内存消耗的影响。
================================
1.8 字典的运算
================================

----------
问题
----------
怎样在数据字典中执行一些计算操作（比如求最小值、最大值、排序等等）？

----------
解决方案
----------
考虑下面的股票名和价格映射字典：

.. code-block:: python

    prices = {
        'ACME': 45.23,
        'AAPL': 612.78,
        'IBM': 205.55,
        'HPQ': 37.20,
        'FB': 10.75
    }

为了对字典值执行计算操作，通常需要使用 ``zip()`` 函数先将键和值反转过来。
比如，下面是查找最小和最大股票价格和股票值的代码：

.. code-block:: python

    min_price = min(zip(prices.values(), prices.keys()))
    # min_price is (10.75, 'FB')
    max_price = max(zip(prices.values(), prices.keys()))
    # max_price is (612.78, 'AAPL')

类似的，可以使用 ``zip()`` 和 ``sorted()`` 函数来排列字典数据：

.. code-block:: python

    prices_sorted = sorted(zip(prices.values(), prices.keys()))
    # prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
    #                   (45.23, 'ACME'), (205.55, 'IBM'),
    #                   (612.78, 'AAPL')]

执行这些计算的时候，需要注意的是 ``zip()`` 函数创建的是一个只能访问一次的迭代器。
比如，下面的代码就会产生错误：

.. code-block:: python

    prices_and_names = zip(prices.values(), prices.keys())
    print(min(prices_and_names)) # OK
    print(max(prices_and_names)) # ValueError: max() arg is an empty sequence

----------
讨论
----------
如果你在一个字典上执行普通的数学运算，你会发现它们仅仅作用于键，而不是值。比如：

.. code-block:: python

    min(prices) # Returns 'AAPL'
    max(prices) # Returns 'IBM'

这个结果并不是你想要的，因为你想要在字典的值集合上执行这些计算。
或许你会尝试着使用字典的 ``values()`` 方法来解决这个问题：

.. code-block:: python

    min(prices.values()) # Returns 10.75
    max(prices.values()) # Returns 612.78

不幸的是，通常这个结果同样也不是你想要的。
你可能还想要知道对应的键的信息（比如那种股票价格是最低的？）。

你可以在 ``min()`` 和 ``max()`` 函数中提供 ``key`` 函数参数来获取最小值或最大值对应的键的信息。比如：

.. code-block:: python

    min(prices, key=lambda k: prices[k]) # Returns 'FB'
    max(prices, key=lambda k: prices[k]) # Returns 'AAPL'

但是，如果还想要得到最小值，你又得执行一次查找操作。比如：

.. code-block:: python

    min_value = prices[min(prices, key=lambda k: prices[k])]

前面的 ``zip()`` 函数方案通过将字典"反转"为 (值，键) 元组序列来解决了上述问题。
当比较两个元组的时候，值会先进行比较，然后才是键。
这样的话你就能通过一条简单的语句就能很轻松的实现在字典上的求最值和排序操作了。

需要注意的是在计算操作中使用到了 (值，键) 对。当多个实体拥有相同的值的时候，键会决定返回结果。
比如，在执行 ``min()`` 和 ``max()`` 操作的时候，如果恰巧最小或最大值有重复的，那么拥有最小或最大键的实体会返回：

.. code-block:: python

    >>> prices = { 'AAA' : 45.23, 'ZZZ': 45.23 }
    >>> min(zip(prices.values(), prices.keys()))
    (45.23, 'AAA')
    >>> max(zip(prices.values(), prices.keys()))
    (45.23, 'ZZZ')
    >>>
=============================
1.9 查找两字典的相同点
=============================

----------
问题
----------
怎样在两个字典中寻寻找相同点（比如相同的键、相同的值等等）？

----------
解决方案
----------
考虑下面两个字典：

.. code-block:: python

    a = {
        'x' : 1,
        'y' : 2,
        'z' : 3
    }

    b = {
        'w' : 10,
        'x' : 11,
        'y' : 2
    }

为了寻找两个字典的相同点，可以简单的在两字典的 ``keys()`` 或者 ``items()`` 方法返回结果上执行集合操作。比如：

.. code-block:: python

    # Find keys in common
    a.keys() & b.keys() # { 'x', 'y' }
    # Find keys in a that are not in b
    a.keys() - b.keys() # { 'z' }
    # Find (key,value) pairs in common
    a.items() & b.items() # { ('y', 2) }

这些操作也可以用于修改或者过滤字典元素。
比如，假如你想以现有字典构造一个排除几个指定键的新字典。
下面利用字典推导来实现这样的需求：

.. code-block:: python

    # Make a new dictionary with certain keys removed
    c = {key:a[key] for key in a.keys() - {'z', 'w'}}
    # c is {'x': 1, 'y': 2}

----------
讨论
----------
一个字典就是一个键集合与值集合的映射关系。
字典的 ``keys()`` 方法返回一个展现键集合的键视图对象。
键视图的一个很少被了解的特性就是它们也支持集合操作，比如集合并、交、差运算。
所以，如果你想对集合的键执行一些普通的集合操作，可以直接使用键视图对象而不用先将它们转换成一个 set。

字典的 ``items()`` 方法返回一个包含 (键，值) 对的元素视图对象。
这个对象同样也支持集合操作，并且可以被用来查找两个字典有哪些相同的键值对。

尽管字典的 ``values()`` 方法也是类似，但是它并不支持这里介绍的集合操作。
某种程度上是因为值视图不能保证所有的值互不相同，这样会导致某些集合操作会出现问题。
不过，如果你硬要在值上面执行这些集合操作的话，你可以先将值集合转换成 set，然后再执行集合运算就行了。
=============================
1.10 删除序列相同元素并保持顺序
=============================

----------
问题
----------
怎样在一个序列上面保持元素顺序的同时消除重复的值？

----------
解决方案
----------
如果序列上的值都是 ``hashable`` 类型，那么可以很简单的利用集合或者生成器来解决这个问题。比如：

.. code-block:: python

    def dedupe(items):
        seen = set()
        for item in items:
            if item not in seen:
                yield item
                seen.add(item)
下面是使用上述函数的例子：

.. code-block:: python

    >>> a = [1, 5, 2, 1, 9, 1, 5, 10]
    >>> list(dedupe(a))
    [1, 5, 2, 9, 10]
    >>>
这个方法仅仅在序列中元素为 ``hashable`` 的时候才管用。
如果你想消除元素不可哈希（比如 ``dict`` 类型）的序列中重复元素的话，你需要将上述代码稍微改变一下，就像这样：

.. code-block:: python

    def dedupe(items, key=None):
        seen = set()
        for item in items:
            val = item if key is None else key(item)
            if val not in seen:
                yield item
                seen.add(val)

这里的key参数指定了一个函数，将序列元素转换成 ``hashable`` 类型。下面是它的用法示例：

.. code-block:: python

    >>> a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
    >>> list(dedupe(a, key=lambda d: (d['x'],d['y'])))
    [{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
    >>> list(dedupe(a, key=lambda d: d['x']))
    [{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
    >>>

如果你想基于单个字段、属性或者某个更大的数据结构来消除重复元素，第二种方案同样可以胜任。

----------
讨论
----------
如果你仅仅就是想消除重复元素，通常可以简单的构造一个集合。比如：

.. code-block:: python

    >>> a
    [1, 5, 2, 1, 9, 1, 5, 10]
    >>> set(a)
    {1, 2, 10, 5, 9}
    >>>

然而，这种方法不能维护元素的顺序，生成的结果中的元素位置被打乱。而上面的方法可以避免这种情况。

在本节中我们使用了生成器函数让我们的函数更加通用，不仅仅是局限于列表处理。
比如，如果如果你想读取一个文件，消除重复行，你可以很容易像这样做：

.. code-block:: python

    with open(somefile,'r') as f:
    for line in dedupe(f):
        ...

上述key函数参数模仿了 ``sorted()`` , ``min()`` 和 ``max()`` 等内置函数的相似功能。
可以参考 1.8 和 1.13 小节了解更多。
================================
1.11 命名切片
================================

----------
问题
----------
如果你的程序包含了大量无法直视的硬编码切片，并且你想清理一下代码。

----------
解决方案
----------
假定你要从一个记录（比如文件或其他类似格式）中的某些固定位置提取字段：

.. code-block:: python

    ######    0123456789012345678901234567890123456789012345678901234567890'
    record = '....................100 .......513.25 ..........'
    cost = int(record[20:23]) * float(record[31:37])

与其那样写，为什么不想这样命名切片呢：

.. code-block:: python

    SHARES = slice(20, 23)
    PRICE = slice(31, 37)
    cost = int(record[SHARES]) * float(record[PRICE])

在这个版本中，你避免了使用大量难以理解的硬编码下标。这使得你的代码更加清晰可读。

----------
讨论
----------
一般来讲，代码中如果出现大量的硬编码下标会使得代码的可读性和可维护性大大降低。
比如，如果你回过来看看一年前你写的代码，你会摸着脑袋想那时候自己到底想干嘛啊。
这是一个很简单的解决方案，它让你更加清晰的表达代码的目的。

内置的 ``slice()`` 函数创建了一个切片对象。所有使用切片的地方都可以使用切片对象。比如：

.. code-block:: python

    >>> items = [0, 1, 2, 3, 4, 5, 6]
    >>> a = slice(2, 4)
    >>> items[2:4]
    [2, 3]
    >>> items[a]
    [2, 3]
    >>> items[a] = [10,11]
    >>> items
    [0, 1, 10, 11, 4, 5, 6]
    >>> del items[a]
    >>> items
    [0, 1, 4, 5, 6]

如果你有一个切片对象a，你可以分别调用它的 ``a.start`` , ``a.stop`` , ``a.step`` 属性来获取更多的信息。比如：

.. code-block:: python

    >>> a = slice(5, 50, 2)
    >>> a.start
    5
    >>> a.stop
    50
    >>> a.step
    2
    >>>

另外，你还可以通过调用切片的 ``indices(size)`` 方法将它映射到一个已知大小的序列上。
这个方法返回一个三元组 ``(start, stop, step)`` ，所有的值都会被缩小，直到适合这个已知序列的边界为止。
这样，使用的时就不会出现 ``IndexError`` 异常。比如：

.. code-block:: python

    >>> s = 'HelloWorld'
    >>> a.indices(len(s))
    (5, 10, 2)
    >>> for i in range(*a.indices(len(s))):
    ...     print(s[i])
    ...
    W
    r
    d
    >>>
================================
1.12 序列中出现次数最多的元素
================================

----------
问题
----------
怎样找出一个序列中出现次数最多的元素呢？

----------
解决方案
----------
``collections.Counter`` 类就是专门为这类问题而设计的，
它甚至有一个有用的 ``most_common()`` 方法直接给了你答案。

为了演示，先假设你有一个单词列表并且想找出哪个单词出现频率最高。你可以这样做：

.. code-block:: python

    words = [
        'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
        'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
        'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
        'my', 'eyes', "you're", 'under'
    ]
    from collections import Counter
    word_counts = Counter(words)
    # 出现频率最高的3个单词
    top_three = word_counts.most_common(3)
    print(top_three)
    # Outputs [('eyes', 8), ('the', 5), ('look', 4)]

----------
讨论
----------
作为输入， ``Counter`` 对象可以接受任意的由可哈希（``hashable``）元素构成的序列对象。
在底层实现上，一个 ``Counter`` 对象就是一个字典，将元素映射到它出现的次数上。比如：

.. code-block:: python

    >>> word_counts['not']
    1
    >>> word_counts['eyes']
    8
    >>>

如果你想手动增加计数，可以简单的用加法：

.. code-block:: python

    >>> morewords = ['why','are','you','not','looking','in','my','eyes']
    >>> for word in morewords:
    ...     word_counts[word] += 1
    ...
    >>> word_counts['eyes']
    9
    >>>

或者你可以使用 ``update()`` 方法：

.. code-block:: python

    >>> word_counts.update(morewords)
    >>>

``Counter`` 实例一个鲜为人知的特性是它们可以很容易的跟数学运算操作相结合。比如：

.. code-block:: python

    >>> a = Counter(words)
    >>> b = Counter(morewords)
    >>> a
    Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,
    "you're": 1, "don't": 1, 'under': 1, 'not': 1})
    >>> b
    Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,
    'my': 1, 'why': 1})
    >>> # Combine counts
    >>> c = a + b
    >>> c
    Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,
    'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,
    'looking': 1, 'are': 1, 'under': 1, 'you': 1})
    >>> # Subtract counts
    >>> d = a - b
    >>> d
    Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,
    "you're": 1, "don't": 1, 'under': 1})
    >>>

毫无疑问， ``Counter`` 对象在几乎所有需要制表或者计数数据的场合是非常有用的工具。
在解决这类问题的时候你应该优先选择它，而不是手动的利用字典去实现。
====================================
1.13 通过某个关键字排序一个字典列表
====================================

----------
问题
----------
你有一个字典列表，你想根据某个或某几个字典字段来排序这个列表。

----------
解决方案
----------
通过使用 ``operator`` 模块的 ``itemgetter`` 函数，可以非常容易的排序这样的数据结构。
假设你从数据库中检索出来网站会员信息列表，并且以下列的数据结构返回：

.. code-block:: python

    rows = [
        {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
        {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
        {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
        {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
    ]

根据任意的字典字段来排序输入结果行是很容易实现的，代码示例：

.. code-block:: python

    from operator import itemgetter
    rows_by_fname = sorted(rows, key=itemgetter('fname'))
    rows_by_uid = sorted(rows, key=itemgetter('uid'))
    print(rows_by_fname)
    print(rows_by_uid)

代码的输出如下：

.. code-block:: python

    [{'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
    {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
    {'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
    {'fname': 'John', 'uid': 1001, 'lname': 'Cleese'}]
    [{'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
    {'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
    {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'},
    {'fname': 'Big', 'uid': 1004, 'lname': 'Jones'}]

``itemgetter()`` 函数也支持多个 keys，比如下面的代码

.. code-block:: python

    rows_by_lfname = sorted(rows, key=itemgetter('lname','fname'))
    print(rows_by_lfname)

会产生如下的输出：

.. code-block:: python

    [{'fname': 'David', 'uid': 1002, 'lname': 'Beazley'},
    {'fname': 'John', 'uid': 1001, 'lname': 'Cleese'},
    {'fname': 'Big', 'uid': 1004, 'lname': 'Jones'},
    {'fname': 'Brian', 'uid': 1003, 'lname': 'Jones'}]

----------
讨论
----------
在上面例子中， ``rows`` 被传递给接受一个关键字参数的 ``sorted()`` 内置函数。
这个参数是 ``callable`` 类型，并且从 ``rows`` 中接受一个单一元素，然后返回被用来排序的值。
``itemgetter()`` 函数就是负责创建这个 ``callable`` 对象的。

``operator.itemgetter()`` 函数有一个被 ``rows`` 中的记录用来查找值的索引参数。可以是一个字典键名称，
一个整形值或者任何能够传入一个对象的 ``__getitem__()`` 方法的值。
如果你传入多个索引参数给 ``itemgetter()`` ，它生成的 ``callable`` 对象会返回一个包含所有元素值的元组，
并且 ``sorted()`` 函数会根据这个元组中元素顺序去排序。
但你想要同时在几个字段上面进行排序（比如通过姓和名来排序，也就是例子中的那样）的时候这种方法是很有用的。

``itemgetter()`` 有时候也可以用 ``lambda`` 表达式代替，比如：

.. code-block:: python

    rows_by_fname = sorted(rows, key=lambda r: r['fname'])
    rows_by_lfname = sorted(rows, key=lambda r: (r['lname'],r['fname']))

这种方案也不错。但是，使用 ``itemgetter()`` 方式会运行的稍微快点。因此，如果你对性能要求比较高的话就使用 ``itemgetter()`` 方式。

最后，不要忘了这节中展示的技术也同样适用于 ``min()`` 和 ``max()`` 等函数。比如：

.. code-block:: python

    >>> min(rows, key=itemgetter('uid'))
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001}
    >>> max(rows, key=itemgetter('uid'))
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
    >>>
================================
1.14 排序不支持原生比较的对象
================================

----------
问题
----------
你想排序类型相同的对象，但是他们不支持原生的比较操作。

----------
解决方案
----------
内置的 ``sorted()`` 函数有一个关键字参数 ``key`` ，可以传入一个 ``callable`` 对象给它，
这个 ``callable`` 对象对每个传入的对象返回一个值，这个值会被 ``sorted`` 用来排序这些对象。
比如，如果你在应用程序里面有一个 ``User`` 实例序列，并且你希望通过他们的 ``user_id`` 属性进行排序，
你可以提供一个以 ``User`` 实例作为输入并输出对应 ``user_id`` 值的 ``callable`` 对象。比如：

.. code-block:: python

    class User:
        def __init__(self, user_id):
            self.user_id = user_id

        def __repr__(self):
            return 'User({})'.format(self.user_id)


    def sort_notcompare():
        users = [User(23), User(3), User(99)]
        print(users)
        print(sorted(users, key=lambda u: u.user_id))

另外一种方式是使用 ``operator.attrgetter()`` 来代替 lambda 函数：

.. code-block:: python

    >>> from operator import attrgetter
    >>> sorted(users, key=attrgetter('user_id'))
    [User(3), User(23), User(99)]
    >>>

----------
讨论
----------
选择使用 lambda 函数或者是 ``attrgetter()`` 可能取决于个人喜好。
但是， ``attrgetter()`` 函数通常会运行的快点，并且还能同时允许多个字段进行比较。
这个跟 ``operator.itemgetter()`` 函数作用于字典类型很类似（参考1.13小节）。
例如，如果 ``User`` 实例还有一个 ``first_name`` 和 ``last_name`` 属性，那么可以向下面这样排序：

.. code-block:: python

    by_name = sorted(users, key=attrgetter('last_name', 'first_name'))

同样需要注意的是，这一小节用到的技术同样适用于像 ``min()`` 和 ``max()`` 之类的函数。比如：

.. code-block:: python

    >>> min(users, key=attrgetter('user_id'))
    User(3)
    >>> max(users, key=attrgetter('user_id'))
    User(99)
    >>>
================================
1.15 通过某个字段将记录分组
================================

----------
问题
----------
你有一个字典或者实例的序列，然后你想根据某个特定的字段比如 ``date`` 来分组迭代访问。

----------
解决方案
----------
``itertools.groupby()`` 函数对于这样的数据分组操作非常实用。
为了演示，假设你已经有了下列的字典列表：

.. code-block:: python

    rows = [
        {'address': '5412 N CLARK', 'date': '07/01/2012'},
        {'address': '5148 N CLARK', 'date': '07/04/2012'},
        {'address': '5800 E 58TH', 'date': '07/02/2012'},
        {'address': '2122 N CLARK', 'date': '07/03/2012'},
        {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
        {'address': '1060 W ADDISON', 'date': '07/02/2012'},
        {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
        {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
    ]

现在假设你想在按 date 分组后的数据块上进行迭代。为了这样做，你首先需要按照指定的字段(这里就是 ``date`` )排序，
然后调用 ``itertools.groupby()`` 函数：

.. code-block:: python

    from operator import itemgetter
    from itertools import groupby

    # Sort by the desired field first
    rows.sort(key=itemgetter('date'))
    # Iterate in groups
    for date, items in groupby(rows, key=itemgetter('date')):
        print(date)
        for i in items:
            print(' ', i)

运行结果：

.. code-block:: python

    07/01/2012
      {'date': '07/01/2012', 'address': '5412 N CLARK'}
      {'date': '07/01/2012', 'address': '4801 N BROADWAY'}
    07/02/2012
      {'date': '07/02/2012', 'address': '5800 E 58TH'}
      {'date': '07/02/2012', 'address': '5645 N RAVENSWOOD'}
      {'date': '07/02/2012', 'address': '1060 W ADDISON'}
    07/03/2012
      {'date': '07/03/2012', 'address': '2122 N CLARK'}
    07/04/2012
      {'date': '07/04/2012', 'address': '5148 N CLARK'}
      {'date': '07/04/2012', 'address': '1039 W GRANVILLE'}

----------
讨论
----------
``groupby()`` 函数扫描整个序列并且查找连续相同值（或者根据指定 key 函数返回值相同）的元素序列。
在每次迭代的时候，它会返回一个值和一个迭代器对象，
这个迭代器对象可以生成元素值全部等于上面那个值的组中所有对象。

一个非常重要的准备步骤是要根据指定的字段将数据排序。
因为 ``groupby()`` 仅仅检查连续的元素，如果事先并没有排序完成的话，分组函数将得不到想要的结果。

如果你仅仅只是想根据 ``date`` 字段将数据分组到一个大的数据结构中去，并且允许随机访问，
那么你最好使用 ``defaultdict()`` 来构建一个多值字典，关于多值字典已经在 1.6 小节有过详细的介绍。比如：

.. code-block:: python

    from collections import defaultdict
    rows_by_date = defaultdict(list)
    for row in rows:
        rows_by_date[row['date']].append(row)

这样的话你可以很轻松的就能对每个指定日期访问对应的记录：

.. code-block:: python

    >>> for r in rows_by_date['07/01/2012']:
    ... print(r)
    ...
    {'date': '07/01/2012', 'address': '5412 N CLARK'}
    {'date': '07/01/2012', 'address': '4801 N BROADWAY'}
    >>>

在上面这个例子中，我们没有必要先将记录排序。因此，如果对内存占用不是很关心，
这种方式会比先排序然后再通过 ``groupby()`` 函数迭代的方式运行得快一些。
================================
1.16 过滤序列元素
================================

----------
问题
----------
你有一个数据序列，想利用一些规则从中提取出需要的值或者是缩短序列

----------
解决方案
----------
最简单的过滤序列元素的方法就是使用列表推导。比如：

.. code-block:: python

    >>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
    >>> [n for n in mylist if n > 0]
    [1, 4, 10, 2, 3]
    >>> [n for n in mylist if n < 0]
    [-5, -7, -1]
    >>>

使用列表推导的一个潜在缺陷就是如果输入非常大的时候会产生一个非常大的结果集，占用大量内存。
如果你对内存比较敏感，那么你可以使用生成器表达式迭代产生过滤的元素。比如：

.. code-block:: python

    >>> pos = (n for n in mylist if n > 0)
    >>> pos
    <generator object <genexpr> at 0x1006a0eb0>
    >>> for x in pos:
    ... print(x)
    ...
    1
    4
    10
    2
    3
    >>>

有时候，过滤规则比较复杂，不能简单的在列表推导或者生成器表达式中表达出来。
比如，假设过滤的时候需要处理一些异常或者其他复杂情况。这时候你可以将过滤代码放到一个函数中，
然后使用内建的 ``filter()`` 函数。示例如下：

.. code-block:: python

    values = ['1', '2', '-3', '-', '4', 'N/A', '5']
    def is_int(val):
        try:
            x = int(val)
            return True
        except ValueError:
            return False
    ivals = list(filter(is_int, values))
    print(ivals)
    # Outputs ['1', '2', '-3', '4', '5']
``filter()`` 函数创建了一个迭代器，因此如果你想得到一个列表的话，就得像示例那样使用 ``list()`` 去转换。

----------
讨论
----------
列表推导和生成器表达式通常情况下是过滤数据最简单的方式。
其实它们还能在过滤的时候转换数据。比如：

.. code-block:: python

    >>> mylist = [1, 4, -5, 10, -7, 2, 3, -1]
    >>> import math
    >>> [math.sqrt(n) for n in mylist if n > 0]
    [1.0, 2.0, 3.1622776601683795, 1.4142135623730951, 1.7320508075688772]
    >>>
过滤操作的一个变种就是将不符合条件的值用新的值代替，而不是丢弃它们。
比如，在一列数据中你可能不仅想找到正数，而且还想将不是正数的数替换成指定的数。
通过将过滤条件放到条件表达式中去，可以很容易的解决这个问题，就像这样：

.. code-block:: python

    >>> clip_neg = [n if n > 0 else 0 for n in mylist]
    >>> clip_neg
    [1, 4, 0, 10, 0, 2, 3, 0]
    >>> clip_pos = [n if n < 0 else 0 for n in mylist]
    >>> clip_pos
    [0, 0, -5, 0, -7, 0, 0, -1]
    >>>
另外一个值得关注的过滤工具就是 ``itertools.compress()`` ，
它以一个 ``iterable`` 对象和一个相对应的 ``Boolean`` 选择器序列作为输入参数。
然后输出 ``iterable`` 对象中对应选择器为 ``True`` 的元素。
当你需要用另外一个相关联的序列来过滤某个序列的时候，这个函数是非常有用的。
比如，假如现在你有下面两列数据：

.. code-block:: python

    addresses = [
        '5412 N CLARK',
        '5148 N CLARK',
        '5800 E 58TH',
        '2122 N CLARK',
        '5645 N RAVENSWOOD',
        '1060 W ADDISON',
        '4801 N BROADWAY',
        '1039 W GRANVILLE',
    ]
    counts = [ 0, 3, 10, 4, 1, 7, 6, 1]

现在你想将那些对应 ``count`` 值大于5的地址全部输出，那么你可以这样做：

.. code-block:: python

    >>> from itertools import compress
    >>> more5 = [n > 5 for n in counts]
    >>> more5
    [False, False, True, False, False, True, True, False]
    >>> list(compress(addresses, more5))
    ['5800 E 58TH', '1060 W ADDISON', '4801 N BROADWAY']
    >>>
这里的关键点在于先创建一个 ``Boolean`` 序列，指示哪些元素符合条件。
然后 ``compress()`` 函数根据这个序列去选择输出对应位置为 ``True`` 的元素。

和 ``filter()`` 函数类似， ``compress()`` 也是返回的一个迭代器。因此，如果你需要得到一个列表，
那么你需要使用 ``list()`` 来将结果转换为列表类型。
================================
1.17 从字典中提取子集
================================

----------
问题
----------
你想构造一个字典，它是另外一个字典的子集。

----------
解决方案
----------
最简单的方式是使用字典推导。比如：

.. code-block:: python

    prices = {
        'ACME': 45.23,
        'AAPL': 612.78,
        'IBM': 205.55,
        'HPQ': 37.20,
        'FB': 10.75
    }
    # Make a dictionary of all prices over 200
    p1 = {key: value for key, value in prices.items() if value > 200}
    # Make a dictionary of tech stocks
    tech_names = {'AAPL', 'IBM', 'HPQ', 'MSFT'}
    p2 = {key: value for key, value in prices.items() if key in tech_names}

----------
讨论
----------
大多数情况下字典推导能做到的，通过创建一个元组序列然后把它传给 ``dict()`` 函数也能实现。比如：

.. code-block:: python

    p1 = dict((key, value) for key, value in prices.items() if value > 200)

但是，字典推导方式表意更清晰，并且实际上也会运行的更快些
（在这个例子中，实际测试几乎比 ``dict()`` 函数方式快整整一倍）。

有时候完成同一件事会有多种方式。比如，第二个例子程序也可以像这样重写：

.. code-block:: python

    # Make a dictionary of tech stocks
    tech_names = { 'AAPL', 'IBM', 'HPQ', 'MSFT' }
    p2 = { key:prices[key] for key in prices.keys() & tech_names }

但是，运行时间测试结果显示这种方案大概比第一种方案慢 1.6 倍。
如果对程序运行性能要求比较高的话，需要花点时间去做计时测试。
关于更多计时和性能测试，可以参考 14.13 小节。
================================
1.18 映射名称到序列元素
================================

----------
问题
----------
你有一段通过下标访问列表或者元组中元素的代码，但是这样有时候会使得你的代码难以阅读，
于是你想通过名称来访问元素。

----------
解决方案
----------
``collections.namedtuple()`` 函数通过使用一个普通的元组对象来帮你解决这个问题。
这个函数实际上是一个返回 Python 中标准元组类型子类的一个工厂方法。
你需要传递一个类型名和你需要的字段给它，然后它就会返回一个类，你可以初始化这个类，为你定义的字段传递值等。
代码示例：

.. code-block:: python

    >>> from collections import namedtuple
    >>> Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
    >>> sub = Subscriber('jonesy@example.com', '2012-10-19')
    >>> sub
    Subscriber(addr='jonesy@example.com', joined='2012-10-19')
    >>> sub.addr
    'jonesy@example.com'
    >>> sub.joined
    '2012-10-19'
    >>>

尽管 ``namedtuple`` 的实例看起来像一个普通的类实例，但是它跟元组类型是可交换的，支持所有的普通元组操作，比如索引和解压。
比如：

.. code-block:: python

    >>> len(sub)
    2
    >>> addr, joined = sub
    >>> addr
    'jonesy@example.com'
    >>> joined
    '2012-10-19'
    >>>

命名元组的一个主要用途是将你的代码从下标操作中解脱出来。
因此，如果你从数据库调用中返回了一个很大的元组列表，通过下标去操作其中的元素，
当你在表中添加了新的列的时候你的代码可能就会出错了。但是如果你使用了命名元组，那么就不会有这样的顾虑。

为了说明清楚，下面是使用普通元组的代码：

.. code-block:: python

    def compute_cost(records):
        total = 0.0
        for rec in records:
            total += rec[1] * rec[2]
        return total

下标操作通常会让代码表意不清晰，并且非常依赖记录的结构。
下面是使用命名元组的版本：

.. code-block:: python

    from collections import namedtuple

    Stock = namedtuple('Stock', ['name', 'shares', 'price'])
    def compute_cost(records):
        total = 0.0
        for rec in records:
            s = Stock(*rec)
            total += s.shares * s.price
        return total

----------
讨论
----------
命名元组另一个用途就是作为字典的替代，因为字典存储需要更多的内存空间。
如果你需要构建一个非常大的包含字典的数据结构，那么使用命名元组会更加高效。
但是需要注意的是，不像字典那样，一个命名元组是不可更改的。比如：

.. code-block:: python

    >>> s = Stock('ACME', 100, 123.45)
    >>> s
    Stock(name='ACME', shares=100, price=123.45)
    >>> s.shares = 75
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    AttributeError: can't set attribute
    >>>

如果你真的需要改变属性的值，那么可以使用命名元组实例的 ``_replace()`` 方法，
它会创建一个全新的命名元组并将对应的字段用新的值取代。比如：

.. code-block:: python

    >>> s = s._replace(shares=75)
    >>> s
    Stock(name='ACME', shares=75, price=123.45)
    >>>

``_replace()`` 方法还有一个很有用的特性就是当你的命名元组拥有可选或者缺失字段时候，
它是一个非常方便的填充数据的方法。
你可以先创建一个包含缺省值的原型元组，然后使用 ``_replace()`` 方法创建新的值被更新过的实例。比如：

.. code-block:: python

    from collections import namedtuple

    Stock = namedtuple('Stock', ['name', 'shares', 'price', 'date', 'time'])

    # Create a prototype instance
    stock_prototype = Stock('', 0, 0.0, None, None)

    # Function to convert a dictionary to a Stock
    def dict_to_stock(s):
        return stock_prototype._replace(**s)

下面是它的使用方法：

.. code-block:: python

    >>> a = {'name': 'ACME', 'shares': 100, 'price': 123.45}
    >>> dict_to_stock(a)
    Stock(name='ACME', shares=100, price=123.45, date=None, time=None)
    >>> b = {'name': 'ACME', 'shares': 100, 'price': 123.45, 'date': '12/17/2012'}
    >>> dict_to_stock(b)
    Stock(name='ACME', shares=100, price=123.45, date='12/17/2012', time=None)
    >>>

最后要说的是，如果你的目标是定义一个需要更新很多实例属性的高效数据结构，那么命名元组并不是你的最佳选择。
这时候你应该考虑定义一个包含 ``__slots__`` 方法的类（参考8.4小节）。
================================
1.19 转换并同时计算数据
================================

----------
问题
----------
你需要在数据序列上执行聚集函数（比如 ``sum()`` , ``min()`` , ``max()`` ），
但是首先你需要先转换或者过滤数据

----------
解决方案
----------
一个非常优雅的方式去结合数据计算与转换就是使用一个生成器表达式参数。
比如，如果你想计算平方和，可以像下面这样做：

.. code-block:: python

    nums = [1, 2, 3, 4, 5]
    s = sum(x * x for x in nums)

下面是更多的例子：

.. code-block:: python

    # Determine if any .py files exist in a directory
    import os
    files = os.listdir('dirname')
    if any(name.endswith('.py') for name in files):
        print('There be python!')
    else:
        print('Sorry, no python.')
    # Output a tuple as CSV
    s = ('ACME', 50, 123.45)
    print(','.join(str(x) for x in s))
    # Data reduction across fields of a data structure
    portfolio = [
        {'name':'GOOG', 'shares': 50},
        {'name':'YHOO', 'shares': 75},
        {'name':'AOL', 'shares': 20},
        {'name':'SCOX', 'shares': 65}
    ]
    min_shares = min(s['shares'] for s in portfolio)

----------
讨论
----------
上面的示例向你演示了当生成器表达式作为一个单独参数传递给函数时候的巧妙语法（你并不需要多加一个括号）。
比如，下面这些语句是等效的：

.. code-block:: python

    s = sum((x * x for x in nums)) # 显示的传递一个生成器表达式对象
    s = sum(x * x for x in nums) # 更加优雅的实现方式，省略了括号

使用一个生成器表达式作为参数会比先创建一个临时列表更加高效和优雅。
比如，如果你不使用生成器表达式的话，你可能会考虑使用下面的实现方式：

.. code-block:: python

    nums = [1, 2, 3, 4, 5]
    s = sum([x * x for x in nums])

这种方式同样可以达到想要的效果，但是它会多一个步骤，先创建一个额外的列表。
对于小型列表可能没什么关系，但是如果元素数量非常大的时候，
它会创建一个巨大的仅仅被使用一次就被丢弃的临时数据结构。而生成器方案会以迭代的方式转换数据，因此更省内存。

在使用一些聚集函数比如 ``min()`` 和 ``max()`` 的时候你可能更加倾向于使用生成器版本，
它们接受的一个 key 关键字参数或许对你很有帮助。
比如，在上面的证券例子中，你可能会考虑下面的实现版本：

.. code-block:: python

    # Original: Returns 20
    min_shares = min(s['shares'] for s in portfolio)
    # Alternative: Returns {'name': 'AOL', 'shares': 20}
    min_shares = min(portfolio, key=lambda s: s['shares'])
============================
1.20 合并多个字典或映射
============================

----------
问题
----------
现在有多个字典或者映射，你想将它们从逻辑上合并为一个单一的映射后执行某些操作，
比如查找值或者检查某些键是否存在。

----------
解决方案
----------
假如你有如下两个字典:

.. code-block:: python

    a = {'x': 1, 'z': 3 }
    b = {'y': 2, 'z': 4 }

现在假设你必须在两个字典中执行查找操作（比如先从 ``a`` 中找，如果找不到再在 ``b`` 中找）。
一个非常简单的解决方案就是使用 ``collections`` 模块中的 ``ChainMap`` 类。比如：

.. code-block:: python

    from collections import ChainMap
    c = ChainMap(a,b)
    print(c['x']) # Outputs 1 (from a)
    print(c['y']) # Outputs 2 (from b)
    print(c['z']) # Outputs 3 (from a)

----------
讨论
----------
一个 ``ChainMap`` 接受多个字典并将它们在逻辑上变为一个字典。
然后，这些字典并不是真的合并在一起了， ``ChainMap`` 类只是在内部创建了一个容纳这些字典的列表
并重新定义了一些常见的字典操作来遍历这个列表。大部分字典操作都是可以正常使用的，比如：

.. code-block:: python

    >>> len(c)
    3
    >>> list(c.keys())
    ['x', 'y', 'z']
    >>> list(c.values())
    [1, 2, 3]
    >>>

如果出现重复键，那么第一次出现的映射值会被返回。
因此，例子程序中的 ``c['z']`` 总是会返回字典 ``a`` 中对应的值，而不是 ``b`` 中对应的值。

对于字典的更新或删除操作总是影响的是列表中第一个字典。比如：

.. code-block:: python

    >>> c['z'] = 10
    >>> c['w'] = 40
    >>> del c['x']
    >>> a
    {'w': 40, 'z': 10}
    >>> del c['y']
    Traceback (most recent call last):
    ...
    KeyError: "Key not found in the first mapping: 'y'"
    >>>

``ChainMap`` 对于编程语言中的作用范围变量（比如 ``globals`` , ``locals`` 等）是非常有用的。
事实上，有一些方法可以使它变得简单：

.. code-block:: python

    >>> values = ChainMap()
    >>> values['x'] = 1
    >>> # Add a new mapping
    >>> values = values.new_child()
    >>> values['x'] = 2
    >>> # Add a new mapping
    >>> values = values.new_child()
    >>> values['x'] = 3
    >>> values
    ChainMap({'x': 3}, {'x': 2}, {'x': 1})
    >>> values['x']
    3
    >>> # Discard last mapping
    >>> values = values.parents
    >>> values['x']
    2
    >>> # Discard last mapping
    >>> values = values.parents
    >>> values['x']
    1
    >>> values
    ChainMap({'x': 1})
    >>>

作为 ``ChainMap`` 的替代，你可能会考虑使用 ``update()`` 方法将两个字典合并。比如：

.. code-block:: python

    >>> a = {'x': 1, 'z': 3 }
    >>> b = {'y': 2, 'z': 4 }
    >>> merged = dict(b)
    >>> merged.update(a)
    >>> merged['x']
    1
    >>> merged['y']
    2
    >>> merged['z']
    3
    >>>

这样也能行得通，但是它需要你创建一个完全不同的字典对象（或者是破坏现有字典结构）。
同时，如果原字典做了更新，这种改变不会反应到新的合并字典中去。比如：

.. code-block:: python

    >>> a['x'] = 13
    >>> merged['x']
    1

``ChainMap`` 使用原来的字典，它自己不创建新的字典。所以它并不会产生上面所说的结果，比如：

.. code-block:: python

    >>> a = {'x': 1, 'z': 3 }
    >>> b = {'y': 2, 'z': 4 }
    >>> merged = ChainMap(a, b)
    >>> merged['x']
    1
    >>> a['x'] = 42
    >>> merged['x'] # Notice change to merged dicts
    42
    >>>
=========================
2.1 使用多个界定符分割字符串
=========================

----------
问题
----------
你需要将一个字符串分割为多个字段，但是分隔符(还有周围的空格)并不是固定的。

----------
解决方案
----------
``string`` 对象的 ``split()`` 方法只适应于非常简单的字符串分割情形，
它并不允许有多个分隔符或者是分隔符周围不确定的空格。
当你需要更加灵活的切割字符串的时候，最好使用 ``re.split()`` 方法：

.. code-block:: python

    >>> line = 'asdf fjdk; afed, fjek,asdf, foo'
    >>> import re
    >>> re.split(r'[;,\s]\s*', line)
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']

----------
讨论
----------
函数 ``re.split()`` 是非常实用的，因为它允许你为分隔符指定多个正则模式。
比如，在上面的例子中，分隔符可以是逗号，分号或者是空格，并且后面紧跟着任意个的空格。
只要这个模式被找到，那么匹配的分隔符两边的实体都会被当成是结果中的元素返回。
返回结果为一个字段列表，这个跟 ``str.split()`` 返回值类型是一样的。

当你使用 ``re.split()`` 函数时候，需要特别注意的是正则表达式中是否包含一个括号捕获分组。
如果使用了捕获分组，那么被匹配的文本也将出现在结果列表中。比如，观察一下这段代码运行后的结果：

.. code-block:: python

    >>> fields = re.split(r'(;|,|\s)\s*', line)
    >>> fields
    ['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
    >>>

获取分割字符在某些情况下也是有用的。
比如，你可能想保留分割字符串，用来在后面重新构造一个新的输出字符串：

.. code-block:: python

    >>> values = fields[::2]
    >>> delimiters = fields[1::2] + ['']
    >>> values
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
    >>> delimiters
    [' ', ';', ',', ',', ',', '']
    >>> # Reform the line using the same delimiters
    >>> ''.join(v+d for v,d in zip(values, delimiters))
    'asdf fjdk;afed,fjek,asdf,foo'
    >>>

如果你不想保留分割字符串到结果列表中去，但仍然需要使用到括号来分组正则表达式的话，
确保你的分组是非捕获分组，形如 ``(?:...)`` 。比如：

.. code-block:: python

    >>> re.split(r'(?:,|;|\s)\s*', line)
    ['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
    >>>

======================
2.2 字符串开头或结尾匹配
======================

----------
问题
----------
你需要通过指定的文本模式去检查字符串的开头或者结尾，比如文件名后缀，URL Scheme等等。

----------
解决方案
----------
检查字符串开头或结尾的一个简单方法是使用 ``str.startswith()`` 或者是 ``str.endswith()`` 方法。比如：

.. code-block:: python

    >>> filename = 'spam.txt'
    >>> filename.endswith('.txt')
    True
    >>> filename.startswith('file:')
    False
    >>> url = 'http://www.python.org'
    >>> url.startswith('http:')
    True
    >>>

如果你想检查多种匹配可能，只需要将所有的匹配项放入到一个元组中去，
然后传给 ``startswith()`` 或者 ``endswith()`` 方法：

.. code-block:: python

    >>> import os
    >>> filenames = os.listdir('.')
    >>> filenames
    [ 'Makefile', 'foo.c', 'bar.py', 'spam.c', 'spam.h' ]
    >>> [name for name in filenames if name.endswith(('.c', '.h')) ]
    ['foo.c', 'spam.c', 'spam.h'
    >>> any(name.endswith('.py') for name in filenames)
    True
    >>>

下面是另一个例子：

.. code-block:: python

    from urllib.request import urlopen

    def read_data(name):
        if name.startswith(('http:', 'https:', 'ftp:')):
            return urlopen(name).read()
        else:
            with open(name) as f:
                return f.read()

奇怪的是，这个方法中必须要输入一个元组作为参数。
如果你恰巧有一个 ``list`` 或者 ``set`` 类型的选择项，
要确保传递参数前先调用 ``tuple()`` 将其转换为元组类型。比如：

.. code-block:: python

    >>> choices = ['http:', 'ftp:']
    >>> url = 'http://www.python.org'
    >>> url.startswith(choices)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: startswith first arg must be str or a tuple of str, not list
    >>> url.startswith(tuple(choices))
    True
    >>>

----------
讨论
----------
``startswith()`` 和 ``endswith()`` 方法提供了一个非常方便的方式去做字符串开头和结尾的检查。
类似的操作也可以使用切片来实现，但是代码看起来没有那么优雅。比如：

.. code-block:: python

    >>> filename = 'spam.txt'
    >>> filename[-4:] == '.txt'
    True
    >>> url = 'http://www.python.org'
    >>> url[:5] == 'http:' or url[:6] == 'https:' or url[:4] == 'ftp:'
    True
    >>>

你可以能还想使用正则表达式去实现，比如：

.. code-block:: python

    >>> import re
    >>> url = 'http://www.python.org'
    >>> re.match('http:|https:|ftp:', url)
    <_sre.SRE_Match object at 0x101253098>
    >>>

这种方式也行得通，但是对于简单的匹配实在是有点小材大用了，本节中的方法更加简单并且运行会更快些。

最后提一下，当和其他操作比如普通数据聚合相结合的时候 ``startswith()`` 和 ``endswith()`` 方法是很不错的。
比如，下面这个语句检查某个文件夹中是否存在指定的文件类型：

.. code-block:: python

    if any(name.endswith(('.c', '.h')) for name in listdir(dirname)):
    ...

========================
2.3 用Shell通配符匹配字符串
========================

----------
问题
----------
你想使用 **Unix Shell** 中常用的通配符(比如 ``*.py`` , ``Dat[0-9]*.csv`` 等)去匹配文本字符串

----------
解决方案
----------
``fnmatch`` 模块提供了两个函数—— ``fnmatch()`` 和 ``fnmatchcase()`` ，可以用来实现这样的匹配。用法如下：

.. code-block:: python

    >>> from fnmatch import fnmatch, fnmatchcase
    >>> fnmatch('foo.txt', '*.txt')
    True
    >>> fnmatch('foo.txt', '?oo.txt')
    True
    >>> fnmatch('Dat45.csv', 'Dat[0-9]*')
    True
    >>> names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
    >>> [name for name in names if fnmatch(name, 'Dat*.csv')]
    ['Dat1.csv', 'Dat2.csv']
    >>>

``fnmatch()`` 函数使用底层操作系统的大小写敏感规则(不同的系统是不一样的)来匹配模式。比如：

.. code-block:: python

    >>> # On OS X (Mac)
    >>> fnmatch('foo.txt', '*.TXT')
    False
    >>> # On Windows
    >>> fnmatch('foo.txt', '*.TXT')
    True
    >>>

如果你对这个区别很在意，可以使用 ``fnmatchcase()`` 来代替。它完全使用你的模式大小写匹配。比如：

.. code-block:: python

    >>> fnmatchcase('foo.txt', '*.TXT')
    False
    >>>

这两个函数通常会被忽略的一个特性是在处理非文件名的字符串时候它们也是很有用的。
比如，假设你有一个街道地址的列表数据：

.. code-block:: python

    addresses = [
        '5412 N CLARK ST',
        '1060 W ADDISON ST',
        '1039 W GRANVILLE AVE',
        '2122 N CLARK ST',
        '4802 N BROADWAY',
    ]

你可以像这样写列表推导：

.. code-block:: python

    >>> from fnmatch import fnmatchcase
    >>> [addr for addr in addresses if fnmatchcase(addr, '* ST')]
    ['5412 N CLARK ST', '1060 W ADDISON ST', '2122 N CLARK ST']
    >>> [addr for addr in addresses if fnmatchcase(addr, '54[0-9][0-9] *CLARK*')]
    ['5412 N CLARK ST']
    >>>

----------
讨论
----------
``fnmatch()`` 函数匹配能力介于简单的字符串方法和强大的正则表达式之间。
如果在数据处理操作中只需要简单的通配符就能完成的时候，这通常是一个比较合理的方案。

如果你的代码需要做文件名的匹配，最好使用 ``glob`` 模块。参考5.13小节。

========================
2.4 字符串匹配和搜索
========================

----------
问题
----------
你想匹配或者搜索特定模式的文本

----------
解决方案
----------
如果你想匹配的是字面字符串，那么你通常只需要调用基本字符串方法就行，
比如 ``str.find()`` , ``str.endswith()`` , ``str.startswith()`` 或者类似的方法：

.. code-block:: python

    >>> text = 'yeah, but no, but yeah, but no, but yeah'
    >>> # Exact match
    >>> text == 'yeah'
    False
    >>> # Match at start or end
    >>> text.startswith('yeah')
    True
    >>> text.endswith('no')
    False
    >>> # Search for the location of the first occurrence
    >>> text.find('no')
    10
    >>>

对于复杂的匹配需要使用正则表达式和 ``re`` 模块。
为了解释正则表达式的基本原理，假设你想匹配数字格式的日期字符串比如 ``11/27/2012`` ，你可以这样做：

.. code-block:: python

    >>> text1 = '11/27/2012'
    >>> text2 = 'Nov 27, 2012'
    >>>
    >>> import re
    >>> # Simple matching: \d+ means match one or more digits
    >>> if re.match(r'\d+/\d+/\d+', text1):
    ... print('yes')
    ... else:
    ... print('no')
    ...
    yes
    >>> if re.match(r'\d+/\d+/\d+', text2):
    ... print('yes')
    ... else:
    ... print('no')
    ...
    no
    >>>

如果你想使用同一个模式去做多次匹配，你应该先将模式字符串预编译为模式对象。比如：

.. code-block:: python

    >>> datepat = re.compile(r'\d+/\d+/\d+')
    >>> if datepat.match(text1):
    ... print('yes')
    ... else:
    ... print('no')
    ...
    yes
    >>> if datepat.match(text2):
    ... print('yes')
    ... else:
    ... print('no')
    ...
    no
    >>>

``match()`` 总是从字符串开始去匹配，如果你想查找字符串任意部分的模式出现位置，
使用 ``findall()`` 方法去代替。比如：

.. code-block:: python

    >>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> datepat.findall(text)
    ['11/27/2012', '3/13/2013']
    >>>

在定义正则式的时候，通常会利用括号去捕获分组。比如：

.. code-block:: python

    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
    >>>

捕获分组可以使得后面的处理更加简单，因为可以分别将每个组的内容提取出来。比如：

.. code-block:: python

    >>> m = datepat.match('11/27/2012')
    >>> m
    <_sre.SRE_Match object at 0x1005d2750>
    >>> # Extract the contents of each group
    >>> m.group(0)
    '11/27/2012'
    >>> m.group(1)
    '11'
    >>> m.group(2)
    '27'
    >>> m.group(3)
    '2012'
    >>> m.groups()
    ('11', '27', '2012')
    >>> month, day, year = m.groups()
    >>>
    >>> # Find all matches (notice splitting into tuples)
    >>> text
    'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> datepat.findall(text)
    [('11', '27', '2012'), ('3', '13', '2013')]
    >>> for month, day, year in datepat.findall(text):
    ... print('{}-{}-{}'.format(year, month, day))
    ...
    2012-11-27
    2013-3-13
    >>>

``findall()`` 方法会搜索文本并以列表形式返回所有的匹配。
如果你想以迭代方式返回匹配，可以使用 ``finditer()`` 方法来代替，比如：

.. code-block:: python

    >>> for m in datepat.finditer(text):
    ... print(m.groups())
    ...
    ('11', '27', '2012')
    ('3', '13', '2013')
    >>>

----------
讨论
----------
关于正则表达式理论的教程已经超出了本书的范围。
不过，这一节阐述了使用re模块进行匹配和搜索文本的最基本方法。
核心步骤就是先使用 ``re.compile()`` 编译正则表达式字符串，
然后使用 ``match()`` , ``findall()`` 或者 ``finditer()`` 等方法。

当写正则式字符串的时候，相对普遍的做法是使用原始字符串比如 ``r'(\d+)/(\d+)/(\d+)'`` 。
这种字符串将不去解析反斜杠，这在正则表达式中是很有用的。
如果不这样做的话，你必须使用两个反斜杠，类似 ``'(\\d+)/(\\d+)/(\\d+)'`` 。

需要注意的是 ``match()`` 方法仅仅检查字符串的开始部分。它的匹配结果有可能并不是你期望的那样。比如：

.. code-block:: python

    >>> m = datepat.match('11/27/2012abcdef')
    >>> m
    <_sre.SRE_Match object at 0x1005d27e8>
    >>> m.group()
    '11/27/2012'
    >>>

如果你想精确匹配，确保你的正则表达式以$结尾，就像这么这样：

.. code-block:: python

    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)$')
    >>> datepat.match('11/27/2012abcdef')
    >>> datepat.match('11/27/2012')
    <_sre.SRE_Match object at 0x1005d2750>
    >>>

最后，如果你仅仅是做一次简单的文本匹配/搜索操作的话，可以略过编译部分，直接使用 ``re`` 模块级别的函数。比如：

.. code-block:: python

    >>> re.findall(r'(\d+)/(\d+)/(\d+)', text)
    [('11', '27', '2012'), ('3', '13', '2013')]
    >>>

但是需要注意的是，如果你打算做大量的匹配和搜索操作的话，最好先编译正则表达式，然后再重复使用它。
模块级别的函数会将最近编译过的模式缓存起来，因此并不会消耗太多的性能，
但是如果使用预编译模式的话，你将会减少查找和一些额外的处理损耗。
========================
2.5 字符串搜索和替换
========================

----------
问题
----------
你想在字符串中搜索和匹配指定的文本模式

----------
解决方案
----------
对于简单的字面模式，直接使用 ``str.replace()`` 方法即可，比如：

.. code-block:: python

    >>> text = 'yeah, but no, but yeah, but no, but yeah'
    >>> text.replace('yeah', 'yep')
    'yep, but no, but yep, but no, but yep'
    >>>

对于复杂的模式，请使用 ``re`` 模块中的 ``sub()`` 函数。
为了说明这个，假设你想将形式为 ``11/27/2012`` 的日期字符串改成 ``2012-11-27`` 。示例如下：

.. code-block:: python

    >>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    >>> import re
    >>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
    'Today is 2012-11-27. PyCon starts 2013-3-13.'
    >>>

``sub()`` 函数中的第一个参数是被匹配的模式，第二个参数是替换模式。反斜杠数字比如 ``\3`` 指向前面模式的捕获组号。

如果你打算用相同的模式做多次替换，考虑先编译它来提升性能。比如：

.. code-block:: python

    >>> import re
    >>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
    >>> datepat.sub(r'\3-\1-\2', text)
    'Today is 2012-11-27. PyCon starts 2013-3-13.'
    >>>

对于更加复杂的替换，可以传递一个替换回调函数来代替，比如：

.. code-block:: python

    >>> from calendar import month_abbr
    >>> def change_date(m):
    ... mon_name = month_abbr[int(m.group(1))]
    ... return '{} {} {}'.format(m.group(2), mon_name, m.group(3))
    ...
    >>> datepat.sub(change_date, text)
    'Today is 27 Nov 2012. PyCon starts 13 Mar 2013.'
    >>>

一个替换回调函数的参数是一个 ``match`` 对象，也就是 ``match()`` 或者 ``find()`` 返回的对象。
使用 ``group()`` 方法来提取特定的匹配部分。回调函数最后返回替换字符串。

如果除了替换后的结果外，你还想知道有多少替换发生了，可以使用 ``re.subn()`` 来代替。比如：

..  code-block:: python

    >>> newtext, n = datepat.subn(r'\3-\1-\2', text)
    >>> newtext
    'Today is 2012-11-27. PyCon starts 2013-3-13.'
    >>> n
    2
    >>>

----------
讨论
----------
关于正则表达式搜索和替换，上面演示的 ``sub()`` 方法基本已经涵盖了所有。
其实最难的部分就是编写正则表达式模式，这个最好是留给读者自己去练习了。

===========================
2.6 字符串忽略大小写的搜索替换
===========================

----------
问题
----------
你需要以忽略大小写的方式搜索与替换文本字符串

----------
解决方案
----------
为了在文本操作时忽略大小写，你需要在使用 ``re`` 模块的时候给这些操作提供 ``re.IGNORECASE`` 标志参数。比如：

.. code-block:: python

    >>> text = 'UPPER PYTHON, lower python, Mixed Python'
    >>> re.findall('python', text, flags=re.IGNORECASE)
    ['PYTHON', 'python', 'Python']
    >>> re.sub('python', 'snake', text, flags=re.IGNORECASE)
    'UPPER snake, lower snake, Mixed snake'
    >>>

最后的那个例子揭示了一个小缺陷，替换字符串并不会自动跟被匹配字符串的大小写保持一致。
为了修复这个，你可能需要一个辅助函数，就像下面的这样：

.. code-block:: python

    def matchcase(word):
        def replace(m):
            text = m.group()
            if text.isupper():
                return word.upper()
            elif text.islower():
                return word.lower()
            elif text[0].isupper():
                return word.capitalize()
            else:
                return word
        return replace

下面是使用上述函数的方法：

.. code-block:: python

    >>> re.sub('python', matchcase('snake'), text, flags=re.IGNORECASE)
    'UPPER SNAKE, lower snake, Mixed Snake'
    >>>

译者注： ``matchcase('snake')`` 返回了一个回调函数(参数必须是 ``match`` 对象)，前面一节提到过，
``sub()`` 函数除了接受替换字符串外，还能接受一个回调函数。

----------
讨论
----------
对于一般的忽略大小写的匹配操作，简单的传递一个 ``re.IGNORECASE`` 标志参数就已经足够了。
但是需要注意的是，这个对于某些需要大小写转换的Unicode匹配可能还不够，
参考2.10小节了解更多细节。
========================
2.7 最短匹配模式
========================

----------
问题
----------
你正在试着用正则表达式匹配某个文本模式，但是它找到的是模式的最长可能匹配。
而你想修改它变成查找最短的可能匹配。

----------
解决方案
----------
这个问题一般出现在需要匹配一对分隔符之间的文本的时候(比如引号包含的字符串)。
为了说明清楚，考虑如下的例子：

.. code-block:: python

    >>> str_pat = re.compile(r'"(.*)"')
    >>> text1 = 'Computer says "no."'
    >>> str_pat.findall(text1)
    ['no.']
    >>> text2 = 'Computer says "no." Phone says "yes."'
    >>> str_pat.findall(text2)
    ['no." Phone says "yes.']
    >>>

在这个例子中，模式 ``r'\"(.*)\"'`` 的意图是匹配被双引号包含的文本。
但是在正则表达式中*操作符是贪婪的，因此匹配操作会查找最长的可能匹配。
于是在第二个例子中搜索 ``text2`` 的时候返回结果并不是我们想要的。

为了修正这个问题，可以在模式中的*操作符后面加上?修饰符，就像这样：

.. code-block:: python

    >>> str_pat = re.compile(r'"(.*?)"')
    >>> str_pat.findall(text2)
    ['no.', 'yes.']
    >>>

这样就使得匹配变成非贪婪模式，从而得到最短的匹配，也就是我们想要的结果。

----------
讨论
----------
这一节展示了在写包含点(.)字符的正则表达式的时候遇到的一些常见问题。
在一个模式字符串中，点(.)匹配除了换行外的任何字符。
然而，如果你将点(.)号放在开始与结束符(比如引号)之间的时候，那么匹配操作会查找符合模式的最长可能匹配。
这样通常会导致很多中间的被开始与结束符包含的文本被忽略掉，并最终被包含在匹配结果字符串中返回。
通过在 ``*`` 或者 ``+`` 这样的操作符后面添加一个 ``?`` 可以强制匹配算法改成寻找最短的可能匹配。

========================
2.8 多行匹配模式
========================

----------
问题
----------
你正在试着使用正则表达式去匹配一大块的文本，而你需要跨越多行去匹配。

----------
解决方案
----------
这个问题很典型的出现在当你用点(.)去匹配任意字符的时候，忘记了点(.)不能匹配换行符的事实。
比如，假设你想试着去匹配C语言分割的注释：

.. code-block:: python

    >>> comment = re.compile(r'/\*(.*?)\*/')
    >>> text1 = '/* this is a comment */'
    >>> text2 = '''/* this is a
    ... multiline comment */
    ... '''
    >>>
    >>> comment.findall(text1)
    [' this is a comment ']
    >>> comment.findall(text2)
    []
    >>>

为了修正这个问题，你可以修改模式字符串，增加对换行的支持。比如：

.. code-block:: python

    >>> comment = re.compile(r'/\*((?:.|\n)*?)\*/')
    >>> comment.findall(text2)
    [' this is a\n multiline comment ']
    >>>

在这个模式中， ``(?:.|\n)`` 指定了一个非捕获组
(也就是它定义了一个仅仅用来做匹配，而不能通过单独捕获或者编号的组)。

----------
讨论
----------
``re.compile()`` 函数接受一个标志参数叫 ``re.DOTALL`` ，在这里非常有用。
它可以让正则表达式中的点(.)匹配包括换行符在内的任意字符。比如：

.. code-block:: python

    >>> comment = re.compile(r'/\*(.*?)\*/', re.DOTALL)
    >>> comment.findall(text2)
    [' this is a\n multiline comment ']

对于简单的情况使用 ``re.DOTALL`` 标记参数工作的很好，
但是如果模式非常复杂或者是为了构造字符串令牌而将多个模式合并起来(2.18节有详细描述)，
这时候使用这个标记参数就可能出现一些问题。
如果让你选择的话，最好还是定义自己的正则表达式模式，这样它可以在不需要额外的标记参数下也能工作的很好。

===========================
2.9 将Unicode文本标准化
===========================

----------
问题
----------
你正在处理Unicode字符串，需要确保所有字符串在底层有相同的表示。

----------
解决方案
----------
在Unicode中，某些字符能够用多个合法的编码表示。为了说明，考虑下面的这个例子：

.. code-block:: python

    >>> s1 = 'Spicy Jalape\u00f1o'
    >>> s2 = 'Spicy Jalapen\u0303o'
    >>> s1
    'Spicy Jalapeño'
    >>> s2
    'Spicy Jalapeño'
    >>> s1 == s2
    False
    >>> len(s1)
    14
    >>> len(s2)
    15
    >>>

这里的文本"Spicy Jalapeño"使用了两种形式来表示。
第一种使用整体字符"ñ"(U+00F1)，第二种使用拉丁字母"n"后面跟一个"~"的组合字符(U+0303)。

在需要比较字符串的程序中使用字符的多种表示会产生问题。
为了修正这个问题，你可以使用unicodedata模块先将文本标准化：

.. code-block:: python

    >>> import unicodedata
    >>> t1 = unicodedata.normalize('NFC', s1)
    >>> t2 = unicodedata.normalize('NFC', s2)
    >>> t1 == t2
    True
    >>> print(ascii(t1))
    'Spicy Jalape\xf1o'
    >>> t3 = unicodedata.normalize('NFD', s1)
    >>> t4 = unicodedata.normalize('NFD', s2)
    >>> t3 == t4
    True
    >>> print(ascii(t3))
    'Spicy Jalapen\u0303o'
    >>>

``normalize()`` 第一个参数指定字符串标准化的方式。
NFC表示字符应该是整体组成(比如可能的话就使用单一编码)，而NFD表示字符应该分解为多个组合字符表示。

Python同样支持扩展的标准化形式NFKC和NFKD，它们在处理某些字符的时候增加了额外的兼容特性。比如：

.. code-block:: python

    >>> s = '\ufb01' # A single character
    >>> s
    'ﬁ'
    >>> unicodedata.normalize('NFD', s)
    'ﬁ'
    # Notice how the combined letters are broken apart here
    >>> unicodedata.normalize('NFKD', s)
    'fi'
    >>> unicodedata.normalize('NFKC', s)
    'fi'
    >>>

----------
讨论
----------
标准化对于任何需要以一致的方式处理Unicode文本的程序都是非常重要的。
当处理来自用户输入的字符串而你很难去控制编码的时候尤其如此。

在清理和过滤文本的时候字符的标准化也是很重要的。
比如，假设你想清除掉一些文本上面的变音符的时候(可能是为了搜索和匹配)：

.. code-block:: python

    >>> t1 = unicodedata.normalize('NFD', s1)
    >>> ''.join(c for c in t1 if not unicodedata.combining(c))
    'Spicy Jalapeno'
    >>>

最后一个例子展示了 ``unicodedata`` 模块的另一个重要方面，也就是测试字符类的工具函数。
``combining()`` 函数可以测试一个字符是否为和音字符。
在这个模块中还有其他函数用于查找字符类别，测试是否为数字字符等等。

Unicode显然是一个很大的主题。如果想更深入的了解关于标准化方面的信息，
请看考 `Unicode官网中关于这部分的说明 <http://www.unicode.org/faq/normalization.html>`_
Ned Batchelder在 `他的网站 <http://nedbatchelder.com/text/unipain.html>`_
上对Python的Unicode处理问题也有一个很好的介绍。

===========================
2.10 在正则式中使用Unicode
===========================

----------
问题
----------
你正在使用正则表达式处理文本，但是关注的是Unicode字符处理。

----------
解决方案
----------
默认情况下 ``re`` 模块已经对一些Unicode字符类有了基本的支持。
比如， ``\\d`` 已经匹配任意的unicode数字字符了：

.. code-block:: python

    >>> import re
    >>> num = re.compile('\d+')
    >>> # ASCII digits
    >>> num.match('123')
    <_sre.SRE_Match object at 0x1007d9ed0>
    >>> # Arabic digits
    >>> num.match('\u0661\u0662\u0663')
    <_sre.SRE_Match object at 0x101234030>
    >>>

如果你想在模式中包含指定的Unicode字符，你可以使用Unicode字符对应的转义序列(比如 ``\uFFF`` 或者 ``\UFFFFFFF`` )。
比如，下面是一个匹配几个不同阿拉伯编码页面中所有字符的正则表达式：

.. code-block:: python

    >>> arabic = re.compile('[\u0600-\u06ff\u0750-\u077f\u08a0-\u08ff]+')
    >>>

当执行匹配和搜索操作的时候，最好是先标准化并且清理所有文本为标准化格式(参考2.9小节)。
但是同样也应该注意一些特殊情况，比如在忽略大小写匹配和大小写转换时的行为。

.. code-block:: python

    >>> pat = re.compile('stra\u00dfe', re.IGNORECASE)
    >>> s = 'straße'
    >>> pat.match(s) # Matches
    <_sre.SRE_Match object at 0x10069d370>
    >>> pat.match(s.upper()) # Doesn't match
    >>> s.upper() # Case folds
    'STRASSE'
    >>>

----------
讨论
----------
混合使用Unicode和正则表达式通常会让你抓狂。
如果你真的打算这样做的话，最好考虑下安装第三方正则式库，
它们会为Unicode的大小写转换和其他大量有趣特性提供全面的支持，包括模糊匹配。

============================
2.11 删除字符串中不需要的字符
============================

----------
问题
----------
你想去掉文本字符串开头，结尾或者中间不想要的字符，比如空白。

----------
解决方案
----------
``strip()`` 方法能用于删除开始或结尾的字符。 ``lstrip()`` 和 ``rstrip()`` 分别从左和从右执行删除操作。
默认情况下，这些方法会去除空白字符，但是你也可以指定其他字符。比如：

.. code-block:: python

    >>> # Whitespace stripping
    >>> s = ' hello world \n'
    >>> s.strip()
    'hello world'
    >>> s.lstrip()
    'hello world \n'
    >>> s.rstrip()
    ' hello world'
    >>>
    >>> # Character stripping
    >>> t = '-----hello====='
    >>> t.lstrip('-')
    'hello====='
    >>> t.strip('-=')
    'hello'
    >>>

----------
讨论
----------
这些 ``strip()`` 方法在读取和清理数据以备后续处理的时候是经常会被用到的。
比如，你可以用它们来去掉空格，引号和完成其他任务。

但是需要注意的是去除操作不会对字符串的中间的文本产生任何影响。比如：

.. code-block:: python

    >>> s = ' hello     world \n'
    >>> s = s.strip()
    >>> s
    'hello     world'
    >>>

如果你想处理中间的空格，那么你需要求助其他技术。比如使用 ``replace()`` 方法或者是用正则表达式替换。示例如下：

.. code-block:: python

    >>> s.replace(' ', '')
    'helloworld'
    >>> import re
    >>> re.sub('\s+', ' ', s)
    'hello world'
    >>>

通常情况下你想将字符串 ``strip`` 操作和其他迭代操作相结合，比如从文件中读取多行数据。
如果是这样的话，那么生成器表达式就可以大显身手了。比如：

.. code-block:: python

    with open(filename) as f:
        lines = (line.strip() for line in f)
        for line in lines:
            print(line)

在这里，表达式 ``lines = (line.strip() for line in f)`` 执行数据转换操作。
这种方式非常高效，因为它不需要预先读取所有数据放到一个临时的列表中去。
它仅仅只是创建一个生成器，并且每次返回行之前会先执行 ``strip`` 操作。

对于更高阶的strip，你可能需要使用 ``translate()`` 方法。请参阅下一节了解更多关于字符串清理的内容。
============================
2.12 审查清理文本字符串
============================

----------
问题
----------
一些无聊的幼稚黑客在你的网站页面表单中输入文本"pýtĥöñ"，然后你想将这些字符清理掉。

----------
解决方案
----------
文本清理问题会涉及到包括文本解析与数据处理等一系列问题。
在非常简单的情形下，你可能会选择使用字符串函数(比如 ``str.upper()`` 和 ``str.lower()`` )将文本转为标准格式。
使用 ``str.replace()`` 或者 ``re.sub()`` 的简单替换操作能删除或者改变指定的字符序列。
你同样还可以使用2.9小节的 ``unicodedata.normalize()`` 函数将unicode文本标准化。

然后，有时候你可能还想在清理操作上更进一步。比如，你可能想消除整个区间上的字符或者去除变音符。
为了这样做，你可以使用经常会被忽视的 ``str.translate()`` 方法。
为了演示，假设你现在有下面这个凌乱的字符串：

.. code-block:: python

    >>> s = 'pýtĥöñ\fis\tawesome\r\n'
    >>> s
    'pýtĥöñ\x0cis\tawesome\r\n'
    >>>

第一步是清理空白字符。为了这样做，先创建一个小的转换表格然后使用 ``translate()`` 方法：

.. code-block:: python

    >>> remap = {
    ...     ord('\t') : ' ',
    ...     ord('\f') : ' ',
    ...     ord('\r') : None # Deleted
    ... }
    >>> a = s.translate(remap)
    >>> a
    'pýtĥöñ is awesome\n'
    >>>

正如你看的那样，空白字符 ``\t`` 和 ``\f`` 已经被重新映射到一个空格。回车字符\r直接被删除。

你可以以这个表格为基础进一步构建更大的表格。比如，让我们删除所有的和音符：

.. code-block:: python

    >>> import unicodedata
    >>> import sys
    >>> cmb_chrs = dict.fromkeys(c for c in range(sys.maxunicode)
    ...                         if unicodedata.combining(chr(c)))
    ...
    >>> b = unicodedata.normalize('NFD', a)
    >>> b
    'pýtĥöñ is awesome\n'
    >>> b.translate(cmb_chrs)
    'python is awesome\n'
    >>>

上面例子中，通过使用 ``dict.fromkeys()`` 方法构造一个字典，每个Unicode和音符作为键，对应的值全部为 ``None`` 。

然后使用 ``unicodedata.normalize()`` 将原始输入标准化为分解形式字符。
然后再调用 ``translate`` 函数删除所有重音符。
同样的技术也可以被用来删除其他类型的字符(比如控制字符等)。

作为另一个例子，这里构造一个将所有Unicode数字字符映射到对应的ASCII字符上的表格：

.. code-block:: python

    >>> digitmap = { c: ord('0') + unicodedata.digit(chr(c))
    ...         for c in range(sys.maxunicode)
    ...         if unicodedata.category(chr(c)) == 'Nd' }
    ...
    >>> len(digitmap)
    460
    >>> # Arabic digits
    >>> x = '\u0661\u0662\u0663'
    >>> x.translate(digitmap)
    '123'
    >>>

另一种清理文本的技术涉及到I/O解码与编码函数。这里的思路是先对文本做一些初步的清理，
然后再结合 ``encode()`` 或者 ``decode()`` 操作来清除或修改它。比如：

.. code-block:: python

    >>> a
    'pýtĥöñ is awesome\n'
    >>> b = unicodedata.normalize('NFD', a)
    >>> b.encode('ascii', 'ignore').decode('ascii')
    'python is awesome\n'
    >>>

这里的标准化操作将原来的文本分解为单独的和音符。接下来的ASCII编码/解码只是简单的一下子丢弃掉那些字符。
当然，这种方法仅仅只在最后的目标就是获取到文本对应ACSII表示的时候生效。

----------
讨论
----------
文本字符清理一个最主要的问题应该是运行的性能。一般来讲，代码越简单运行越快。
对于简单的替换操作， ``str.replace()`` 方法通常是最快的，甚至在你需要多次调用的时候。
比如，为了清理空白字符，你可以这样做：

.. code-block:: python

    def clean_spaces(s):
        s = s.replace('\r', '')
        s = s.replace('\t', ' ')
        s = s.replace('\f', ' ')
        return s

如果你去测试的话，你就会发现这种方式会比使用 ``translate()`` 或者正则表达式要快很多。

另一方面，如果你需要执行任何复杂字符对字符的重新映射或者删除操作的话， ``tanslate()`` 方法会非常的快。

从大的方面来讲，对于你的应用程序来说性能是你不得不去自己研究的东西。
不幸的是，我们不可能给你建议一个特定的技术，使它能够适应所有的情况。
因此实际情况中需要你自己去尝试不同的方法并评估它。

尽管这一节集中讨论的是文本，但是类似的技术也可以适用于字节，包括简单的替换，转换和正则表达式。
============================
2.13 字符串对齐
============================

----------
问题
----------
你想通过某种对齐方式来格式化字符串

----------
解决方案
----------
对于基本的字符串对齐操作，可以使用字符串的 ``ljust()`` , ``rjust()`` 和 ``center()`` 方法。比如：

.. code-block:: python

    >>> text = 'Hello World'
    >>> text.ljust(20)
    'Hello World         '
    >>> text.rjust(20)
    '         Hello World'
    >>> text.center(20)
    '    Hello World     '
    >>>
所有这些方法都能接受一个可选的填充字符。比如：

.. code-block:: python

    >>> text.rjust(20,'=')
    '=========Hello World'
    >>> text.center(20,'*')
    '****Hello World*****'
    >>>

函数 ``format()`` 同样可以用来很容易的对齐字符串。
你要做的就是使用 ``<,>`` 或者 ``^`` 字符后面紧跟一个指定的宽度。比如：

.. code-block:: python

    >>> format(text, '>20')
    '         Hello World'
    >>> format(text, '<20')
    'Hello World         '
    >>> format(text, '^20')
    '    Hello World     '
    >>>

如果你想指定一个非空格的填充字符，将它写到对齐字符的前面即可：

.. code-block:: python

    >>> format(text, '=>20s')
    '=========Hello World'
    >>> format(text, '*^20s')
    '****Hello World*****'
    >>>

当格式化多个值的时候，这些格式代码也可以被用在 ``format()`` 方法中。比如：

.. code-block:: python

    >>> '{:>10s} {:>10s}'.format('Hello', 'World')
    '     Hello      World'
    >>>

``format()`` 函数的一个好处是它不仅适用于字符串。它可以用来格式化任何值，使得它非常的通用。
比如，你可以用它来格式化数字：

.. code-block:: python

    >>> x = 1.2345
    >>> format(x, '>10')
    '    1.2345'
    >>> format(x, '^10.2f')
    '   1.23   '
    >>>

----------
讨论
----------
在老的代码中，你经常会看到被用来格式化文本的 ``%`` 操作符。比如：

.. code-block:: python

    >>> '%-20s' % text
    'Hello World         '
    >>> '%20s' % text
    '         Hello World'
    >>>

但是，在新版本代码中，你应该优先选择 ``format()`` 函数或者方法。
``format()`` 要比 ``%`` 操作符的功能更为强大。
并且 ``format()`` 也比使用 ``ljust()`` , ``rjust()`` 或 ``center()`` 方法更通用，
因为它可以用来格式化任意对象，而不仅仅是字符串。

如果想要完全了解 ``format()`` 函数的有用特性，
请参考 `在线Python文档 <https://docs.python.org/3/library/string.html#formatspec>`_

============================
2.14 合并拼接字符串
============================

----------
问题
----------
你想将几个小的字符串合并为一个大的字符串

----------
解决方案
----------
如果你想要合并的字符串是在一个序列或者 ``iterable`` 中，那么最快的方式就是使用 ``join()`` 方法。比如：

.. code-block:: python

    >>> parts = ['Is', 'Chicago', 'Not', 'Chicago?']
    >>> ' '.join(parts)
    'Is Chicago Not Chicago?'
    >>> ','.join(parts)
    'Is,Chicago,Not,Chicago?'
    >>> ''.join(parts)
    'IsChicagoNotChicago?'
    >>>

初看起来，这种语法看上去会比较怪，但是 ``join()`` 被指定为字符串的一个方法。
这样做的部分原因是你想去连接的对象可能来自各种不同的数据序列(比如列表，元组，字典，文件，集合或生成器等)，
如果在所有这些对象上都定义一个 ``join()`` 方法明显是冗余的。
因此你只需要指定你想要的分割字符串并调用他的 ``join()`` 方法去将文本片段组合起来。

如果你仅仅只是合并少数几个字符串，使用加号(+)通常已经足够了：

.. code-block:: python

    >>> a = 'Is Chicago'
    >>> b = 'Not Chicago?'
    >>> a + ' ' + b
    'Is Chicago Not Chicago?'
    >>>

加号(+)操作符在作为一些复杂字符串格式化的替代方案的时候通常也工作的很好，比如：

.. code-block:: python

    >>> print('{} {}'.format(a,b))
    Is Chicago Not Chicago?
    >>> print(a + ' ' + b)
    Is Chicago Not Chicago?
    >>>

如果你想在源码中将两个字面字符串合并起来，你只需要简单的将它们放到一起，不需要用加号(+)。比如：

.. code-block:: python

    >>> a = 'Hello' 'World'
    >>> a
    'HelloWorld'
    >>>

----------
讨论
----------
字符串合并可能看上去并不需要用一整节来讨论。
但是不应该小看这个问题，程序员通常在字符串格式化的时候因为选择不当而给应用程序带来严重性能损失。

最重要的需要引起注意的是，当我们使用加号(+)操作符去连接大量的字符串的时候是非常低效率的，
因为加号连接会引起内存复制以及垃圾回收操作。
特别的，你永远都不应像下面这样写字符串连接代码：

.. code-block:: python

    s = ''
    for p in parts:
        s += p

这种写法会比使用 ``join()`` 方法运行的要慢一些，因为每一次执行+=操作的时候会创建一个新的字符串对象。
你最好是先收集所有的字符串片段然后再将它们连接起来。

一个相对比较聪明的技巧是利用生成器表达式(参考1.19小节)转换数据为字符串的同时合并字符串，比如：

.. code-block:: python

    >>> data = ['ACME', 50, 91.1]
    >>> ','.join(str(d) for d in data)
    'ACME,50,91.1'
    >>>

同样还得注意不必要的字符串连接操作。有时候程序员在没有必要做连接操作的时候仍然多此一举。比如在打印的时候：

.. code-block:: python

    print(a + ':' + b + ':' + c) # Ugly
    print(':'.join([a, b, c])) # Still ugly
    print(a, b, c, sep=':') # Better

当混合使用I/O操作和字符串连接操作的时候，有时候需要仔细研究你的程序。
比如，考虑下面的两端代码片段：

.. code-block:: python

    # Version 1 (string concatenation)
    f.write(chunk1 + chunk2)

    # Version 2 (separate I/O operations)
    f.write(chunk1)
    f.write(chunk2)

如果两个字符串很小，那么第一个版本性能会更好些，因为I/O系统调用天生就慢。
另外一方面，如果两个字符串很大，那么第二个版本可能会更加高效，
因为它避免了创建一个很大的临时结果并且要复制大量的内存块数据。
还是那句话，有时候是需要根据你的应用程序特点来决定应该使用哪种方案。

最后谈一下，如果你准备编写构建大量小字符串的输出代码，
你最好考虑下使用生成器函数，利用yield语句产生输出片段。比如：

.. code-block:: python

    def sample():
        yield 'Is'
        yield 'Chicago'
        yield 'Not'
        yield 'Chicago?'

这种方法一个有趣的方面是它并没有对输出片段到底要怎样组织做出假设。
例如，你可以简单的使用 ``join()`` 方法将这些片段合并起来：

.. code-block:: python

    text = ''.join(sample())

或者你也可以将字符串片段重定向到I/O：

.. code-block:: python

    for part in sample():
        f.write(part)

再或者你还可以写出一些结合I/O操作的混合方案：

.. code-block:: python

    def combine(source, maxsize):
        parts = []
        size = 0
        for part in source:
            parts.append(part)
            size += len(part)
            if size > maxsize:
                yield ''.join(parts)
                parts = []
                size = 0
        yield ''.join(parts)

    # 结合文件操作
    with open('filename', 'w') as f:
        for part in combine(sample(), 32768):
            f.write(part)

这里的关键点在于原始的生成器函数并不需要知道使用细节，它只负责生成字符串片段就行了。
============================
2.15 字符串中插入变量
============================

----------
问题
----------
你想创建一个内嵌变量的字符串，变量被它的值所表示的字符串替换掉。

----------
解决方案
----------
Python并没有对在字符串中简单替换变量值提供直接的支持。
但是通过使用字符串的 ``format()`` 方法来解决这个问题。比如：

.. code-block:: python

    >>> s = '{name} has {n} messages.'
    >>> s.format(name='Guido', n=37)
    'Guido has 37 messages.'
    >>>

或者，如果要被替换的变量能在变量域中找到，
那么你可以结合使用 ``format_map()`` 和 ``vars()`` 。就像下面这样：

.. code-block:: python

    >>> name = 'Guido'
    >>> n = 37
    >>> s.format_map(vars())
    'Guido has 37 messages.'
    >>>

``vars()`` 还有一个有意思的特性就是它也适用于对象实例。比如：

.. code-block:: python

    >>> class Info:
    ...     def __init__(self, name, n):
    ...         self.name = name
    ...         self.n = n
    ...
    >>> a = Info('Guido',37)
    >>> s.format_map(vars(a))
    'Guido has 37 messages.'
    >>>

``format`` 和 ``format_map()`` 的一个缺陷就是它们并不能很好的处理变量缺失的情况，比如：

.. code-block:: python

    >>> s.format(name='Guido')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'n'
    >>>

一种避免这种错误的方法是另外定义一个含有 ``__missing__()`` 方法的字典对象，就像下面这样：

.. code-block:: python

    class safesub(dict):
    """防止key找不到"""
    def __missing__(self, key):
        return '{' + key + '}'

现在你可以利用这个类包装输入后传递给 ``format_map()`` ：

.. code-block:: python

    >>> del n # Make sure n is undefined
    >>> s.format_map(safesub(vars()))
    'Guido has {n} messages.'
    >>>

如果你发现自己在代码中频繁的执行这些步骤，你可以将变量替换步骤用一个工具函数封装起来。就像下面这样：

.. code-block:: python

    import sys

    def sub(text):
        return text.format_map(safesub(sys._getframe(1).f_locals))

现在你可以像下面这样写了：

.. code-block:: python

    >>> name = 'Guido'
    >>> n = 37
    >>> print(sub('Hello {name}'))
    Hello Guido
    >>> print(sub('You have {n} messages.'))
    You have 37 messages.
    >>> print(sub('Your favorite color is {color}'))
    Your favorite color is {color}
    >>>

----------
讨论
----------
多年以来由于Python缺乏对变量替换的内置支持而导致了各种不同的解决方案。
作为本节中展示的一个可能的解决方案，你可以有时候会看到像下面这样的字符串格式化代码：

.. code-block:: python

    >>> name = 'Guido'
    >>> n = 37
    >>> '%(name) has %(n) messages.' % vars()
    'Guido has 37 messages.'
    >>>

你可能还会看到字符串模板的使用：

.. code-block:: python

    >>> import string
    >>> s = string.Template('$name has $n messages.')
    >>> s.substitute(vars())
    'Guido has 37 messages.'
    >>>

然而， ``format()`` 和 ``format_map()`` 相比较上面这些方案而已更加先进，因此应该被优先选择。
使用 ``format()`` 方法还有一个好处就是你可以获得对字符串格式化的所有支持(对齐，填充，数字格式化等待)，
而这些特性是使用像模板字符串之类的方案不可能获得的。

本机还部分介绍了一些高级特性。映射或者字典类中鲜为人知的 ``__missing__()`` 方法可以让你定义如何处理缺失的值。
在 ``SafeSub`` 类中，这个方法被定义为对缺失的值返回一个占位符。
你可以发现缺失的值会出现在结果字符串中(在调试的时候可能很有用)，而不是产生一个 ``KeyError`` 异常。

``sub()`` 函数使用 ``sys._getframe(1)`` 返回调用者的栈帧。可以从中访问属性 ``f_locals`` 来获得局部变量。
毫无疑问绝大部分情况下在代码中去直接操作栈帧应该是不推荐的。
但是，对于像字符串替换工具函数而言它是非常有用的。
另外，值得注意的是 ``f_locals`` 是一个复制调用函数的本地变量的字典。
尽管你可以改变 ``f_locals`` 的内容，但是这个修改对于后面的变量访问没有任何影响。
所以，虽说访问一个栈帧看上去很邪恶，但是对它的任何操作不会覆盖和改变调用者本地变量的值。
==========================
2.16 以指定列宽格式化字符串
==========================

----------
问题
----------
你有一些长字符串，想以指定的列宽将它们重新格式化。

----------
解决方案
----------
使用 ``textwrap`` 模块来格式化字符串的输出。比如，假如你有下列的长字符串：

.. code-block:: python

    s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
    the eyes, not around the eyes, don't look around the eyes, \
    look into my eyes, you're under."

下面演示使用 ``textwrap`` 格式化字符串的多种方式：

.. code-block:: python

    >>> import textwrap
    >>> print(textwrap.fill(s, 70))
    Look into my eyes, look into my eyes, the eyes, the eyes, the eyes,
    not around the eyes, don't look around the eyes, look into my eyes,
    you're under.

    >>> print(textwrap.fill(s, 40))
    Look into my eyes, look into my eyes,
    the eyes, the eyes, the eyes, not around
    the eyes, don't look around the eyes,
    look into my eyes, you're under.

    >>> print(textwrap.fill(s, 40, initial_indent='    '))
        Look into my eyes, look into my
    eyes, the eyes, the eyes, the eyes, not
    around the eyes, don't look around the
    eyes, look into my eyes, you're under.

    >>> print(textwrap.fill(s, 40, subsequent_indent='    '))
    Look into my eyes, look into my eyes,
        the eyes, the eyes, the eyes, not
        around the eyes, don't look around
        the eyes, look into my eyes, you're
        under.

----------
讨论
----------
``textwrap`` 模块对于字符串打印是非常有用的，特别是当你希望输出自动匹配终端大小的时候。
你可以使用 ``os.get_terminal_size()`` 方法来获取终端的大小尺寸。比如：

.. code-block:: python

    >>> import os
    >>> os.get_terminal_size().columns
    80
    >>>

``fill()`` 方法接受一些其他可选参数来控制tab，语句结尾等。
参阅 `textwrap.TextWrapper文档`_ 获取更多内容。

.. _textwrap.TextWrapper文档:
    https://docs.python.org/3.6/library/textwrap.html#textwrap.TextWrapper

============================
2.17 在字符串中处理html和xml
============================

----------
问题
----------
你想将HTML或者XML实体如 ``&entity;`` 或 ``&#code;`` 替换为对应的文本。
再者，你需要转换文本中特定的字符(比如<, >, 或 &)。

----------
解决方案
----------
如果你想替换文本字符串中的 '<' 或者 '>' ，使用 ``html.escape()`` 函数可以很容易的完成。比如：

.. code-block:: python

    >>> s = 'Elements are written as "<tag>text</tag>".'
    >>> import html
    >>> print(s)
    Elements are written as "<tag>text</tag>".
    >>> print(html.escape(s))
    Elements are written as &quot;&lt;tag&gt;text&lt;/tag&gt;&quot;.

    >>> # Disable escaping of quotes
    >>> print(html.escape(s, quote=False))
    Elements are written as "&lt;tag&gt;text&lt;/tag&gt;".
    >>>

如果你正在处理的是ASCII文本，并且想将非ASCII文本对应的编码实体嵌入进去，
可以给某些I/O函数传递参数 ``errors='xmlcharrefreplace'`` 来达到这个目。比如：

.. code-block:: python

    >>> s = 'Spicy Jalapeño'
    >>> s.encode('ascii', errors='xmlcharrefreplace')
    b'Spicy Jalape&#241;o'
    >>>

为了替换文本中的编码实体，你需要使用另外一种方法。
如果你正在处理HTML或者XML文本，试着先使用一个合适的HTML或者XML解析器。
通常情况下，这些工具会自动替换这些编码值，你无需担心。

有时候，如果你接收到了一些含有编码值的原始文本，需要手动去做替换，
通常你只需要使用HTML或者XML解析器的一些相关工具函数/方法即可。比如：

.. code-block:: python

    >>> s = 'Spicy &quot;Jalape&#241;o&quot.'
    >>> from html.parser import HTMLParser
    >>> p = HTMLParser()
    >>> p.unescape(s)
    'Spicy "Jalapeño".'
    >>>
    >>> t = 'The prompt is &gt;&gt;&gt;'
    >>> from xml.sax.saxutils import unescape
    >>> unescape(t)
    'The prompt is >>>'
    >>>

----------
讨论
----------
在生成HTML或者XML文本的时候，如果正确的转换特殊标记字符是一个很容易被忽视的细节。
特别是当你使用 ``print()`` 函数或者其他字符串格式化来产生输出的时候。
使用像 ``html.escape()`` 的工具函数可以很容易的解决这类问题。

如果你想以其他方式处理文本，还有一些其他的工具函数比如 ``xml.sax.saxutils.unescapge()`` 可以帮助你。
然而，你应该先调研清楚怎样使用一个合适的解析器。
比如，如果你在处理HTML或XML文本，
使用某个解析模块比如 ``html.parse`` 或 ``xml.etree.ElementTree`` 已经帮你自动处理了相关的替换细节。

============================
2.18 字符串令牌解析
============================

----------
问题
----------
你有一个字符串，想从左至右将其解析为一个令牌流。

----------
解决方案
----------
假如你有下面这样一个文本字符串：

.. code-block:: python

    text = 'foo = 23 + 42 * 10'

为了令牌化字符串，你不仅需要匹配模式，还得指定模式的类型。
比如，你可能想将字符串像下面这样转换为序列对：

.. code-block:: python

    tokens = [('NAME', 'foo'), ('EQ','='), ('NUM', '23'), ('PLUS','+'),
              ('NUM', '42'), ('TIMES', '*'), ('NUM', '10')]

为了执行这样的切分，第一步就是像下面这样利用命名捕获组的正则表达式来定义所有可能的令牌，包括空格：

.. code-block:: python

    import re
    NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
    NUM = r'(?P<NUM>\d+)'
    PLUS = r'(?P<PLUS>\+)'
    TIMES = r'(?P<TIMES>\*)'
    EQ = r'(?P<EQ>=)'
    WS = r'(?P<WS>\s+)'

    master_pat = re.compile('|'.join([NAME, NUM, PLUS, TIMES, EQ, WS]))

在上面的模式中， ``?P<TOKENNAME>`` 用于给一个模式命名，供后面使用。

下一步，为了令牌化，使用模式对象很少被人知道的 ``scanner()`` 方法。
这个方法会创建一个 ``scanner`` 对象，
在这个对象上不断的调用 ``match()`` 方法会一步步的扫描目标文本，每步一个匹配。
下面是演示一个 ``scanner`` 对象如何工作的交互式例子：

.. code-block:: python

    >>> scanner = master_pat.scanner('foo = 42')
    >>> scanner.match()
    <_sre.SRE_Match object at 0x100677738>
    >>> _.lastgroup, _.group()
    ('NAME', 'foo')
    >>> scanner.match()
    <_sre.SRE_Match object at 0x100677738>
    >>> _.lastgroup, _.group()
    ('WS', ' ')
    >>> scanner.match()
    <_sre.SRE_Match object at 0x100677738>
    >>> _.lastgroup, _.group()
    ('EQ', '=')
    >>> scanner.match()
    <_sre.SRE_Match object at 0x100677738>
    >>> _.lastgroup, _.group()
    ('WS', ' ')
    >>> scanner.match()
    <_sre.SRE_Match object at 0x100677738>
    >>> _.lastgroup, _.group()
    ('NUM', '42')
    >>> scanner.match()
    >>>

实际使用这种技术的时候，可以很容易的像下面这样将上述代码打包到一个生成器中：

.. code-block:: python

    def generate_tokens(pat, text):
        Token = namedtuple('Token', ['type', 'value'])
        scanner = pat.scanner(text)
        for m in iter(scanner.match, None):
            yield Token(m.lastgroup, m.group())

    # Example use
    for tok in generate_tokens(master_pat, 'foo = 42'):
        print(tok)
    # Produces output
    # Token(type='NAME', value='foo')
    # Token(type='WS', value=' ')
    # Token(type='EQ', value='=')
    # Token(type='WS', value=' ')
    # Token(type='NUM', value='42')

如果你想过滤令牌流，你可以定义更多的生成器函数或者使用一个生成器表达式。
比如，下面演示怎样过滤所有的空白令牌：

.. code-block:: python

    tokens = (tok for tok in generate_tokens(master_pat, text)
              if tok.type != 'WS')
    for tok in tokens:
        print(tok)

----------
讨论
----------
通常来讲令牌化是很多高级文本解析与处理的第一步。
为了使用上面的扫描方法，你需要记住这里一些重要的几点。
第一点就是你必须确认你使用正则表达式指定了所有输入中可能出现的文本序列。
如果有任何不可匹配的文本出现了，扫描就会直接停止。这也是为什么上面例子中必须指定空白字符令牌的原因。

令牌的顺序也是有影响的。 ``re`` 模块会按照指定好的顺序去做匹配。
因此，如果一个模式恰好是另一个更长模式的子字符串，那么你需要确定长模式写在前面。比如：

.. code-block:: python

    LT = r'(?P<LT><)'
    LE = r'(?P<LE><=)'
    EQ = r'(?P<EQ>=)'

    master_pat = re.compile('|'.join([LE, LT, EQ])) # Correct
    # master_pat = re.compile('|'.join([LT, LE, EQ])) # Incorrect

第二个模式是错的，因为它会将文本<=匹配为令牌LT紧跟着EQ，而不是单独的令牌LE，这个并不是我们想要的结果。

最后，你需要留意下子字符串形式的模式。比如，假设你有如下两个模式：

.. code-block:: python

    PRINT = r'(?P<PRINT>print)'
    NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'

    master_pat = re.compile('|'.join([PRINT, NAME]))

    for tok in generate_tokens(master_pat, 'printer'):
        print(tok)

    # Outputs :
    # Token(type='PRINT', value='print')
    # Token(type='NAME', value='er')

关于更高阶的令牌化技术，你可能需要查看 `PyParsing <http://pyparsing.wikispaces.com/>`_
或者 `PLY <http://www.dabeaz.com/ply/index.html>`_ 包。
一个调用PLY的例子在下一节会有演示。


============================
2.19 实现一个简单的递归下降分析器
============================

----------
问题
----------
你想根据一组语法规则解析文本并执行命令，或者构造一个代表输入的抽象语法树。
如果语法非常简单，你可以不去使用一些框架，而是自己写这个解析器。

----------
解决方案
----------
在这个问题中，我们集中讨论根据特殊语法去解析文本的问题。
为了这样做，你首先要以BNF或者EBNF形式指定一个标准语法。
比如，一个简单数学表达式语法可能像下面这样：

.. code-block:: python

    expr ::= expr + term
        |   expr - term
        |   term

    term ::= term * factor
        |   term / factor
        |   factor

    factor ::= ( expr )
        |   NUM

或者，以EBNF形式：

.. code-block:: python

    expr ::= term { (+|-) term }*

    term ::= factor { (*|/) factor }*

    factor ::= ( expr )
        |   NUM

在EBNF中，被包含在 ``{...}*`` 中的规则是可选的。*代表0次或多次重复(跟正则表达式中意义是一样的)。

现在，如果你对BNF的工作机制还不是很明白的话，就把它当做是一组左右符号可相互替换的规则。
一般来讲，解析的原理就是你利用BNF完成多个替换和扩展以匹配输入文本和语法规则。
为了演示，假设你正在解析形如 ``3 + 4 * 5`` 的表达式。
这个表达式先要通过使用2.18节中介绍的技术分解为一组令牌流。
结果可能是像下列这样的令牌序列：

.. code-block:: python

    NUM + NUM * NUM

在此基础上， 解析动作会试着去通过替换操作匹配语法到输入令牌：

.. code-block:: python

    expr
    expr ::= term { (+|-) term }*
    expr ::= factor { (*|/) factor }* { (+|-) term }*
    expr ::= NUM { (*|/) factor }* { (+|-) term }*
    expr ::= NUM { (+|-) term }*
    expr ::= NUM + term { (+|-) term }*
    expr ::= NUM + factor { (*|/) factor }* { (+|-) term }*
    expr ::= NUM + NUM { (*|/) factor}* { (+|-) term }*
    expr ::= NUM + NUM * factor { (*|/) factor }* { (+|-) term }*
    expr ::= NUM + NUM * NUM { (*|/) factor }* { (+|-) term }*
    expr ::= NUM + NUM * NUM { (+|-) term }*
    expr ::= NUM + NUM * NUM

下面所有的解析步骤可能需要花点时间弄明白，但是它们原理都是查找输入并试着去匹配语法规则。
第一个输入令牌是NUM，因此替换首先会匹配那个部分。
一旦匹配成功，就会进入下一个令牌+，以此类推。
当已经确定不能匹配下一个令牌的时候，右边的部分(比如 ``{ (*/) factor }*`` )就会被清理掉。
在一个成功的解析中，整个右边部分会完全展开来匹配输入令牌流。

有了前面的知识背景，下面我们举一个简单示例来展示如何构建一个递归下降表达式求值程序：

.. code-block:: python

    #!/usr/bin/env python
    # -*- encoding: utf-8 -*-
    """
    Topic: 下降解析器
    Desc :
    """
    import re
    import collections

    # Token specification
    NUM = r'(?P<NUM>\d+)'
    PLUS = r'(?P<PLUS>\+)'
    MINUS = r'(?P<MINUS>-)'
    TIMES = r'(?P<TIMES>\*)'
    DIVIDE = r'(?P<DIVIDE>/)'
    LPAREN = r'(?P<LPAREN>\()'
    RPAREN = r'(?P<RPAREN>\))'
    WS = r'(?P<WS>\s+)'

    master_pat = re.compile('|'.join([NUM, PLUS, MINUS, TIMES,
                                      DIVIDE, LPAREN, RPAREN, WS]))
    # Tokenizer
    Token = collections.namedtuple('Token', ['type', 'value'])


    def generate_tokens(text):
        scanner = master_pat.scanner(text)
        for m in iter(scanner.match, None):
            tok = Token(m.lastgroup, m.group())
            if tok.type != 'WS':
                yield tok


    # Parser
    class ExpressionEvaluator:
        '''
        Implementation of a recursive descent parser. Each method
        implements a single grammar rule. Use the ._accept() method
        to test and accept the current lookahead token. Use the ._expect()
        method to exactly match and discard the next token on on the input
        (or raise a SyntaxError if it doesn't match).
        '''

        def parse(self, text):
            self.tokens = generate_tokens(text)
            self.tok = None  # Last symbol consumed
            self.nexttok = None  # Next symbol tokenized
            self._advance()  # Load first lookahead token
            return self.expr()

        def _advance(self):
            'Advance one token ahead'
            self.tok, self.nexttok = self.nexttok, next(self.tokens, None)

        def _accept(self, toktype):
            'Test and consume the next token if it matches toktype'
            if self.nexttok and self.nexttok.type == toktype:
                self._advance()
                return True
            else:
                return False

        def _expect(self, toktype):
            'Consume next token if it matches toktype or raise SyntaxError'
            if not self._accept(toktype):
                raise SyntaxError('Expected ' + toktype)

        # Grammar rules follow
        def expr(self):
            "expression ::= term { ('+'|'-') term }*"
            exprval = self.term()
            while self._accept('PLUS') or self._accept('MINUS'):
                op = self.tok.type
                right = self.term()
                if op == 'PLUS':
                    exprval += right
                elif op == 'MINUS':
                    exprval -= right
            return exprval

        def term(self):
            "term ::= factor { ('*'|'/') factor }*"
            termval = self.factor()
            while self._accept('TIMES') or self._accept('DIVIDE'):
                op = self.tok.type
                right = self.factor()
                if op == 'TIMES':
                    termval *= right
                elif op == 'DIVIDE':
                    termval /= right
            return termval

        def factor(self):
            "factor ::= NUM | ( expr )"
            if self._accept('NUM'):
                return int(self.tok.value)
            elif self._accept('LPAREN'):
                exprval = self.expr()
                self._expect('RPAREN')
                return exprval
            else:
                raise SyntaxError('Expected NUMBER or LPAREN')


    def descent_parser():
        e = ExpressionEvaluator()
        print(e.parse('2'))
        print(e.parse('2 + 3'))
        print(e.parse('2 + 3 * 4'))
        print(e.parse('2 + (3 + 4) * 5'))
        # print(e.parse('2 + (3 + * 4)'))
        # Traceback (most recent call last):
        #    File "<stdin>", line 1, in <module>
        #    File "exprparse.py", line 40, in parse
        #    return self.expr()
        #    File "exprparse.py", line 67, in expr
        #    right = self.term()
        #    File "exprparse.py", line 77, in term
        #    termval = self.factor()
        #    File "exprparse.py", line 93, in factor
        #    exprval = self.expr()
        #    File "exprparse.py", line 67, in expr
        #    right = self.term()
        #    File "exprparse.py", line 77, in term
        #    termval = self.factor()
        #    File "exprparse.py", line 97, in factor
        #    raise SyntaxError("Expected NUMBER or LPAREN")
        #    SyntaxError: Expected NUMBER or LPAREN


    if __name__ == '__main__':
        descent_parser()

----------
讨论
----------
文本解析是一个很大的主题， 一般会占用学生学习编译课程时刚开始的三周时间。
如果你在找寻关于语法，解析算法等相关的背景知识的话，你应该去看一下编译器书籍。
很显然，关于这方面的内容太多，不可能在这里全部展开。

尽管如此，编写一个递归下降解析器的整体思路是比较简单的。
开始的时候，你先获得所有的语法规则，然后将其转换为一个函数或者方法。
因此如果你的语法类似这样：

.. code-block:: python

    expr ::= term { ('+'|'-') term }*

    term ::= factor { ('*'|'/') factor }*

    factor ::= '(' expr ')'
        | NUM

你应该首先将它们转换成一组像下面这样的方法：

.. code-block:: python

    class ExpressionEvaluator:
        ...
        def expr(self):
        ...
        def term(self):
        ...
        def factor(self):
        ...

每个方法要完成的任务很简单 - 它必须从左至右遍历语法规则的每一部分，处理每个令牌。
从某种意义上讲，方法的目的就是要么处理完语法规则，要么产生一个语法错误。
为了这样做，需采用下面的这些实现方法：

-   如果规则中的下个符号是另外一个语法规则的名字(比如term或factor)，就简单的调用同名的方法即可。
    这就是该算法中"下降"的由来 - 控制下降到另一个语法规则中去。
    有时候规则会调用已经执行的方法(比如，在 ``factor ::= '('expr ')'`` 中对expr的调用)。
    这就是算法中"递归"的由来。
-   如果规则中下一个符号是个特殊符号(比如()，你得查找下一个令牌并确认是一个精确匹配)。
    如果不匹配，就产生一个语法错误。这一节中的 ``_expect()`` 方法就是用来做这一步的。
-   如果规则中下一个符号为一些可能的选择项(比如 + 或 -)，
    你必须对每一种可能情况检查下一个令牌，只有当它匹配一个的时候才能继续。
    这也是本节示例中 ``_accept()`` 方法的目的。
    它相当于_expect()方法的弱化版本，因为如果一个匹配找到了它会继续，
    但是如果没找到，它不会产生错误而是回滚(允许后续的检查继续进行)。
-   对于有重复部分的规则(比如在规则表达式 ``::= term { ('+'|'-') term }*`` 中)，
    重复动作通过一个while循环来实现。
    循环主体会收集或处理所有的重复元素直到没有其他元素可以找到。
-   一旦整个语法规则处理完成，每个方法会返回某种结果给调用者。
    这就是在解析过程中值是怎样累加的原理。
    比如，在表达式求值程序中，返回值代表表达式解析后的部分结果。
    最后所有值会在最顶层的语法规则方法中合并起来。

尽管向你演示的是一个简单的例子，递归下降解析器可以用来实现非常复杂的解析。
比如，Python语言本身就是通过一个递归下降解析器去解释的。
如果你对此感兴趣，你可以通过查看Python源码文件Grammar/Grammar来研究下底层语法机制。
看完你会发现，通过手动方式去实现一个解析器其实会有很多的局限和不足之处。

其中一个局限就是它们不能被用于包含任何左递归的语法规则中。比如，假如你需要翻译下面这样一个规则：

.. code-block:: python

    items ::= items ',' item
        | item

为了这样做，你可能会像下面这样使用 ``items()`` 方法：

.. code-block:: python

    def items(self):
        itemsval = self.items()
        if itemsval and self._accept(','):
            itemsval.append(self.item())
        else:
            itemsval = [ self.item() ]

唯一的问题是这个方法根本不能工作，事实上，它会产生一个无限递归错误。

关于语法规则本身你可能也会碰到一些棘手的问题。
比如，你可能想知道下面这个简单扼语法是否表述得当：

.. code-block:: python

    expr ::= factor { ('+'|'-'|'*'|'/') factor }*

    factor ::= '(' expression ')'
        | NUM

这个语法看上去没啥问题，但是它却不能察觉到标准四则运算中的运算符优先级。
比如，表达式 ``"3 + 4 * 5"`` 会得到35而不是期望的23.
分开使用"expr"和"term"规则可以让它正确的工作。

对于复杂的语法，你最好是选择某个解析工具比如PyParsing或者是PLY。
下面是使用PLY来重写表达式求值程序的代码：

.. code-block:: python

    from ply.lex import lex
    from ply.yacc import yacc

    # Token list
    tokens = [ 'NUM', 'PLUS', 'MINUS', 'TIMES', 'DIVIDE', 'LPAREN', 'RPAREN' ]
    # Ignored characters
    t_ignore = ' \t\n'
    # Token specifications (as regexs)
    t_PLUS = r'\+'
    t_MINUS = r'-'
    t_TIMES = r'\*'
    t_DIVIDE = r'/'
    t_LPAREN = r'\('
    t_RPAREN = r'\)'

    # Token processing functions
    def t_NUM(t):
        r'\d+'
        t.value = int(t.value)
        return t

    # Error handler
    def t_error(t):
        print('Bad character: {!r}'.format(t.value[0]))
        t.skip(1)

    # Build the lexer
    lexer = lex()

    # Grammar rules and handler functions
    def p_expr(p):
        '''
        expr : expr PLUS term
            | expr MINUS term
        '''
        if p[2] == '+':
            p[0] = p[1] + p[3]
        elif p[2] == '-':
            p[0] = p[1] - p[3]


    def p_expr_term(p):
        '''
        expr : term
        '''
        p[0] = p[1]


    def p_term(p):
        '''
        term : term TIMES factor
        | term DIVIDE factor
        '''
        if p[2] == '*':
            p[0] = p[1] * p[3]
        elif p[2] == '/':
            p[0] = p[1] / p[3]

    def p_term_factor(p):
        '''
        term : factor
        '''
        p[0] = p[1]

    def p_factor(p):
        '''
        factor : NUM
        '''
        p[0] = p[1]

    def p_factor_group(p):
        '''
        factor : LPAREN expr RPAREN
        '''
        p[0] = p[2]

    def p_error(p):
        print('Syntax error')

    parser = yacc()

这个程序中，所有代码都位于一个比较高的层次。你只需要为令牌写正则表达式和规则匹配时的高阶处理函数即可。
而实际的运行解析器，接受令牌等等底层动作已经被库函数实现了。

下面是一个怎样使用得到的解析对象的例子：

.. code-block:: python

    >>> parser.parse('2')
    2
    >>> parser.parse('2+3')
    5
    >>> parser.parse('2+(3+4)*5')
    37
    >>>

如果你想在你的编程过程中来点挑战和刺激，编写解析器和编译器是个不错的选择。
再次，一本编译器的书籍会包含很多底层的理论知识。不过很多好的资源也可以在网上找到。
Python自己的ast模块也值得去看一下。

============================
2.20 字节字符串上的字符串操作
============================

----------
问题
----------
你想在字节字符串上执行普通的文本操作(比如移除，搜索和替换)。

----------
解决方案
----------
字节字符串同样也支持大部分和文本字符串一样的内置操作。比如：

.. code-block:: python

    >>> data = b'Hello World'
    >>> data[0:5]
    b'Hello'
    >>> data.startswith(b'Hello')
    True
    >>> data.split()
    [b'Hello', b'World']
    >>> data.replace(b'Hello', b'Hello Cruel')
    b'Hello Cruel World'
    >>>

这些操作同样也适用于字节数组。比如：

.. code-block:: python

    >>> data = bytearray(b'Hello World')
    >>> data[0:5]
    bytearray(b'Hello')
    >>> data.startswith(b'Hello')
    True
    >>> data.split()
    [bytearray(b'Hello'), bytearray(b'World')]
    >>> data.replace(b'Hello', b'Hello Cruel')
    bytearray(b'Hello Cruel World')
    >>>

你可以使用正则表达式匹配字节字符串，但是正则表达式本身必须也是字节串。比如：

.. code-block:: python

    >>>
    >>> data = b'FOO:BAR,SPAM'
    >>> import re
    >>> re.split('[:,]',data)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/local/lib/python3.3/re.py", line 191, in split
    return _compile(pattern, flags).split(string, maxsplit)
    TypeError: can't use a string pattern on a bytes-like object
    >>> re.split(b'[:,]',data) # Notice: pattern as bytes
    [b'FOO', b'BAR', b'SPAM']
    >>>

----------
讨论
----------
大多数情况下，在文本字符串上的操作均可用于字节字符串。
然而，这里也有一些需要注意的不同点。首先，字节字符串的索引操作返回整数而不是单独字符。比如：

.. code-block:: python

    >>> a = 'Hello World' # Text string
    >>> a[0]
    'H'
    >>> a[1]
    'e'
    >>> b = b'Hello World' # Byte string
    >>> b[0]
    72
    >>> b[1]
    101
    >>>
这种语义上的区别会对于处理面向字节的字符数据有影响。

第二点，字节字符串不会提供一个美观的字符串表示，也不能很好的打印出来，除非它们先被解码为一个文本字符串。比如：

.. code-block:: python

    >>> s = b'Hello World'
    >>> print(s)
    b'Hello World' # Observe b'...'
    >>> print(s.decode('ascii'))
    Hello World
    >>>

类似的，也不存在任何适用于字节字符串的格式化操作：

.. code-block:: python

    >>> b'%10s %10d %10.2f' % (b'ACME', 100, 490.1)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: unsupported operand type(s) for %: 'bytes' and 'tuple'
    >>> b'{} {} {}'.format(b'ACME', 100, 490.1)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    AttributeError: 'bytes' object has no attribute 'format'
    >>>

如果你想格式化字节字符串，你得先使用标准的文本字符串，然后将其编码为字节字符串。比如：

.. code-block:: python

    >>> '{:10s} {:10d} {:10.2f}'.format('ACME', 100, 490.1).encode('ascii')
    b'ACME 100 490.10'
    >>>

最后需要注意的是，使用字节字符串可能会改变一些操作的语义，特别是那些跟文件系统有关的操作。
比如，如果你使用一个编码为字节的文件名，而不是一个普通的文本字符串，会禁用文件名的编码/解码。比如：

.. code-block:: python

    >>> # Write a UTF-8 filename
    >>> with open('jalape\xf1o.txt', 'w') as f:
    ...     f.write('spicy')
    ...
    >>> # Get a directory listing
    >>> import os
    >>> os.listdir('.') # Text string (names are decoded)
    ['jalapeño.txt']
    >>> os.listdir(b'.') # Byte string (names left as bytes)
    [b'jalapen\xcc\x83o.txt']
    >>>

注意例子中的最后部分给目录名传递一个字节字符串是怎样导致结果中文件名以未解码字节返回的。
在目录中的文件名包含原始的UTF-8编码。
参考5.15小节获取更多文件名相关的内容。

最后提一点，一些程序员为了提升程序执行的速度会倾向于使用字节字符串而不是文本字符串。
尽管操作字节字符串确实会比文本更加高效(因为处理文本固有的Unicode相关开销)。
这样做通常会导致非常杂乱的代码。你会经常发现字节字符串并不能和Python的其他部分工作的很好，
并且你还得手动处理所有的编码/解码操作。
坦白讲，如果你在处理文本的话，就直接在程序中使用普通的文本字符串而不是字节字符串。不做死就不会死！

========================
3.1 数字的四舍五入
========================

----------
问题
----------
你想对浮点数执行指定精度的舍入运算。

----------
解决方案
----------
对于简单的舍入运算，使用内置的 ``round(value, ndigits)`` 函数即可。比如：

.. code-block:: python

    >>> round(1.23, 1)
    1.2
    >>> round(1.27, 1)
    1.3
    >>> round(-1.27, 1)
    -1.3
    >>> round(1.25361,3)
    1.254
    >>>

当一个值刚好在两个边界的中间的时候， ``round`` 函数返回离它最近的偶数。
也就是说，对1.5或者2.5的舍入运算都会得到2。

传给 ``round()`` 函数的 ``ndigits`` 参数可以是负数，这种情况下，
舍入运算会作用在十位、百位、千位等上面。比如：

.. code-block:: python

    >>> a = 1627731
    >>> round(a, -1)
    1627730
    >>> round(a, -2)
    1627700
    >>> round(a, -3)
    1628000
    >>>

----------
讨论
----------
不要将舍入和格式化输出搞混淆了。
如果你的目的只是简单的输出一定宽度的数，你不需要使用 ``round()`` 函数。
而仅仅只需要在格式化的时候指定精度即可。比如：

.. code-block:: python

    >>> x = 1.23456
    >>> format(x, '0.2f')
    '1.23'
    >>> format(x, '0.3f')
    '1.235'
    >>> 'value is {:0.3f}'.format(x)
    'value is 1.235'
    >>>

同样，不要试着去舍入浮点值来"修正"表面上看起来正确的问题。比如，你可能倾向于这样做：

.. code-block:: python

    >>> a = 2.1
    >>> b = 4.2
    >>> c = a + b
    >>> c
    6.300000000000001
    >>> c = round(c, 2) # "Fix" result (???)
    >>> c
    6.3
    >>>

对于大多数使用到浮点的程序，没有必要也不推荐这样做。
尽管在计算的时候会有一点点小的误差，但是这些小的误差是能被理解与容忍的。
如果不能允许这样的小误差(比如涉及到金融领域)，那么就得考虑使用 ``decimal`` 模块了，下一节我们会详细讨论。
========================
3.2 执行精确的浮点数运算
========================

----------
问题
----------
你需要对浮点数执行精确的计算操作，并且不希望有任何小误差的出现。

----------
解决方案
----------
浮点数的一个普遍问题是它们并不能精确的表示十进制数。
并且，即使是最简单的数学运算也会产生小的误差，比如：

.. code-block:: python

    >>> a = 4.2
    >>> b = 2.1
    >>> a + b
    6.300000000000001
    >>> (a + b) == 6.3
    False
    >>>

这些错误是由底层CPU和IEEE 754标准通过自己的浮点单位去执行算术时的特征。
由于Python的浮点数据类型使用底层表示存储数据，因此你没办法去避免这样的误差。

如果你想更加精确(并能容忍一定的性能损耗)，你可以使用 ``decimal`` 模块：

.. code-block:: python

    >>> from decimal import Decimal
    >>> a = Decimal('4.2')
    >>> b = Decimal('2.1')
    >>> a + b
    Decimal('6.3')
    >>> print(a + b)
    6.3
    >>> (a + b) == Decimal('6.3')
    True

初看起来，上面的代码好像有点奇怪，比如我们用字符串来表示数字。
然而， ``Decimal`` 对象会像普通浮点数一样的工作(支持所有的常用数学运算)。
如果你打印它们或者在字符串格式化函数中使用它们，看起来跟普通数字没什么两样。

``decimal`` 模块的一个主要特征是允许你控制计算的每一方面，包括数字位数和四舍五入运算。
为了这样做，你先得创建一个本地上下文并更改它的设置，比如：

.. code-block:: python

    >>> from decimal import localcontext
    >>> a = Decimal('1.3')
    >>> b = Decimal('1.7')
    >>> print(a / b)
    0.7647058823529411764705882353
    >>> with localcontext() as ctx:
    ...     ctx.prec = 3
    ...     print(a / b)
    ...
    0.765
    >>> with localcontext() as ctx:
    ...     ctx.prec = 50
    ...     print(a / b)
    ...
    0.76470588235294117647058823529411764705882352941176
    >>>

----------
讨论
----------
``decimal`` 模块实现了IBM的"通用小数运算规范"。不用说，有很多的配置选项这本书没有提到。

Python新手会倾向于使用 ``decimal`` 模块来处理浮点数的精确运算。
然而，先理解你的应用程序目的是非常重要的。
如果你是在做科学计算或工程领域的计算、电脑绘图，或者是科学领域的大多数运算，
那么使用普通的浮点类型是比较普遍的做法。
其中一个原因是，在真实世界中很少会要求精确到普通浮点数能提供的17位精度。
因此，计算过程中的那么一点点的误差是被允许的。
第二点就是，原生的浮点数计算要快的多-有时候你在执行大量运算的时候速度也是非常重要的。

即便如此，你却不能完全忽略误差。数学家花了大量时间去研究各类算法，有些处理误差会比其他方法更好。
你也得注意下减法删除以及大数和小数的加分运算所带来的影响。比如：

.. code-block:: python

    >>> nums = [1.23e+18, 1, -1.23e+18]
    >>> sum(nums) # Notice how 1 disappears
    0.0
    >>>

上面的错误可以利用 ``math.fsum()`` 所提供的更精确计算能力来解决：

.. code-block:: python

    >>> import math
    >>> math.fsum(nums)
    1.0
    >>>

然而，对于其他的算法，你应该仔细研究它并理解它的误差产生来源。

总的来说， ``decimal`` 模块主要用在涉及到金融的领域。
在这类程序中，哪怕是一点小小的误差在计算过程中蔓延都是不允许的。
因此， ``decimal`` 模块为解决这类问题提供了方法。
当Python和数据库打交道的时候也通常会遇到 ``Decimal`` 对象，并且，通常也是在处理金融数据的时候。
============================
3.3 数字的格式化输出
============================

----------
问题
----------
你需要将数字格式化后输出，并控制数字的位数、对齐、千位分隔符和其他的细节。

----------
解决方案
----------
格式化输出单个数字的时候，可以使用内置的 ``format()`` 函数，比如：

.. code-block:: python

    >>> x = 1234.56789

    >>> # Two decimal places of accuracy
    >>> format(x, '0.2f')
    '1234.57'

    >>> # Right justified in 10 chars, one-digit accuracy
    >>> format(x, '>10.1f')
    '    1234.6'

    >>> # Left justified
    >>> format(x, '<10.1f')
    '1234.6    '

    >>> # Centered
    >>> format(x, '^10.1f')
    '  1234.6  '

    >>> # Inclusion of thousands separator
    >>> format(x, ',')
    '1,234.56789'
    >>> format(x, '0,.1f')
    '1,234.6'
    >>>

如果你想使用指数记法，将f改成e或者E(取决于指数输出的大小写形式)。比如：

.. code-block:: python

    >>> format(x, 'e')
    '1.234568e+03'
    >>> format(x, '0.2E')
    '1.23E+03'
    >>>

同时指定宽度和精度的一般形式是 ``'[<>^]?width[,]?(.digits)?'`` ，
其中 ``width`` 和 ``digits`` 为整数，？代表可选部分。
同样的格式也被用在字符串的 ``format()`` 方法中。比如：

.. code-block:: python

    >>> 'The value is {:0,.2f}'.format(x)
    'The value is 1,234.57'
    >>>

----------
讨论
----------
数字格式化输出通常是比较简单的。上面演示的技术同时适用于浮点数和 ``decimal`` 模块中的 ``Decimal`` 数字对象。

当指定数字的位数后，结果值会根据 ``round()`` 函数同样的规则进行四舍五入后返回。比如：

.. code-block:: python

    >>> x
    1234.56789
    >>> format(x, '0.1f')
    '1234.6'
    >>> format(-x, '0.1f')
    '-1234.6'
    >>>

包含千位符的格式化跟本地化没有关系。
如果你需要根据地区来显示千位符，你需要自己去调查下 ``locale`` 模块中的函数了。
你同样也可以使用字符串的 ``translate()`` 方法来交换千位符。比如：

.. code-block:: python

    >>> swap_separators = { ord('.'):',', ord(','):'.' }
    >>> format(x, ',').translate(swap_separators)
    '1.234,56789'
    >>>

在很多Python代码中会看到使用%来格式化数字的，比如：

.. code-block:: python

    >>> '%0.2f' % x
    '1234.57'
    >>> '%10.1f' % x
    '    1234.6'
    >>> '%-10.1f' % x
    '1234.6    '
    >>>

这种格式化方法也是可行的，不过比更加先进的 ``format()`` 要差一点。
比如，在使用%操作符格式化数字的时候，一些特性(添加千位符)并不能被支持。

============================
3.4 二八十六进制整数
============================

----------
问题
----------
你需要转换或者输出使用二进制，八进制或十六进制表示的整数。

----------
解决方案
----------
为了将整数转换为二进制、八进制或十六进制的文本串，
可以分别使用 ``bin()`` , ``oct()`` 或 ``hex()`` 函数：

.. code-block:: python

    >>> x = 1234
    >>> bin(x)
    '0b10011010010'
    >>> oct(x)
    '0o2322'
    >>> hex(x)
    '0x4d2'
    >>>

另外，如果你不想输出 ``0b`` , ``0o`` 或者 ``0x`` 的前缀的话，可以使用 ``format()`` 函数。比如：

.. code-block:: python

    >>> format(x, 'b')
    '10011010010'
    >>> format(x, 'o')
    '2322'
    >>> format(x, 'x')
    '4d2'
    >>>

整数是有符号的，所以如果你在处理负数的话，输出结果会包含一个负号。比如：

.. code-block:: python

    >>> x = -1234
    >>> format(x, 'b')
    '-10011010010'
    >>> format(x, 'x')
    '-4d2'
    >>>

如果你想产生一个无符号值，你需要增加一个指示最大位长度的值。比如为了显示32位的值，可以像下面这样写：

.. code-block:: python

    >>> x = -1234
    >>> format(2**32 + x, 'b')
    '11111111111111111111101100101110'
    >>> format(2**32 + x, 'x')
    'fffffb2e'
    >>>

为了以不同的进制转换整数字符串，简单的使用带有进制的 ``int()`` 函数即可：

.. code-block:: python

    >>> int('4d2', 16)
    1234
    >>> int('10011010010', 2)
    1234
    >>>

----------
讨论
----------
大多数情况下处理二进制、八进制和十六进制整数是很简单的。
只要记住这些转换属于整数和其对应的文本表示之间的转换即可。永远只有一种整数类型。

最后，使用八进制的程序员有一点需要注意下。
Python指定八进制数的语法跟其他语言稍有不同。比如，如果你像下面这样指定八进制，会出现语法错误：

.. code-block:: python

    >>> import os
    >>> os.chmod('script.py', 0755)
        File "<stdin>", line 1
            os.chmod('script.py', 0755)
                                ^
    SyntaxError: invalid token
    >>>

需确保八进制数的前缀是 ``0o`` ，就像下面这样：

.. code-block:: python

    >>> os.chmod('script.py', 0o755)
    >>>
==========================
3.5 字节到大整数的打包与解包
==========================

----------
问题
----------
你有一个字节字符串并想将它解压成一个整数。或者，你需要将一个大整数转换为一个字节字符串。

----------
解决方案
----------
假设你的程序需要处理一个拥有128位长的16个元素的字节字符串。比如：

.. code-block:: python

    data = b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'

为了将bytes解析为整数，使用 ``int.from_bytes()`` 方法，并像下面这样指定字节顺序：

.. code-block:: python

    >>> len(data)
    16
    >>> int.from_bytes(data, 'little')
    69120565665751139577663547927094891008
    >>> int.from_bytes(data, 'big')
    94522842520747284487117727783387188
    >>>

为了将一个大整数转换为一个字节字符串，使用 ``int.to_bytes()`` 方法，并像下面这样指定字节数和字节顺序：

.. code-block:: python

    >>> x = 94522842520747284487117727783387188
    >>> x.to_bytes(16, 'big')
    b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'
    >>> x.to_bytes(16, 'little')
    b'4\x00#\x00\x01\xef\xcd\x00\xab\x90x\x00V4\x12\x00'
    >>>

----------
讨论
----------
大整数和字节字符串之间的转换操作并不常见。
然而，在一些应用领域有时候也会出现，比如密码学或者网络。
例如，IPv6网络地址使用一个128位的整数表示。
如果你要从一个数据记录中提取这样的值的时候，你就会面对这样的问题。

作为一种替代方案，你可能想使用6.11小节中所介绍的 ``struct`` 模块来解压字节。
这样也行得通，不过利用 ``struct`` 模块来解压对于整数的大小是有限制的。
因此，你可能想解压多个字节串并将结果合并为最终的结果，就像下面这样：

.. code-block:: python

    >>> data
    b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'
    >>> import struct
    >>> hi, lo = struct.unpack('>QQ', data)
    >>> (hi << 64) + lo
    94522842520747284487117727783387188
    >>>

字节顺序规则(little或big)仅仅指定了构建整数时的字节的低位高位排列方式。
我们从下面精心构造的16进制数的表示中可以很容易的看出来：

.. code-block:: python

    >>> x = 0x01020304
    >>> x.to_bytes(4, 'big')
    b'\x01\x02\x03\x04'
    >>> x.to_bytes(4, 'little')
    b'\x04\x03\x02\x01'
    >>>

如果你试着将一个整数打包为字节字符串，那么它就不合适了，你会得到一个错误。
如果需要的话，你可以使用 ``int.bit_length()`` 方法来决定需要多少字节位来存储这个值。

.. code-block:: python

    >>> x = 523 ** 23
    >>> x
    335381300113661875107536852714019056160355655333978849017944067
    >>> x.to_bytes(16, 'little')
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    OverflowError: int too big to convert
    >>> x.bit_length()
    208
    >>> nbytes, rem = divmod(x.bit_length(), 8)
    >>> if rem:
    ... nbytes += 1
    ...
    >>>
    >>> x.to_bytes(nbytes, 'little')
    b'\x03X\xf1\x82iT\x96\xac\xc7c\x16\xf3\xb9\xcf...\xd0'
    >>>

============================
3.6 复数的数学运算
============================

----------
问题
----------
你写的最新的网络认证方案代码遇到了一个难题，并且你唯一的解决办法就是使用复数空间。
再或者是你仅仅需要使用复数来执行一些计算操作。

----------
解决方案
----------
复数可以用使用函数 ``complex(real, imag)`` 或者是带有后缀j的浮点数来指定。比如：

.. code-block:: python

    >>> a = complex(2, 4)
    >>> b = 3 - 5j
    >>> a
    (2+4j)
    >>> b
    (3-5j)
    >>>

对应的实部、虚部和共轭复数可以很容易的获取。就像下面这样：

.. code-block:: python

    >>> a.real
    2.0
    >>> a.imag
    4.0
    >>> a.conjugate()
    (2-4j)
    >>>

另外，所有常见的数学运算都可以工作：

.. code-block:: python

    >>> a + b
    (5-1j)
    >>> a * b
    (26+2j)
    >>> a / b
    (-0.4117647058823529+0.6470588235294118j)
    >>> abs(a)
    4.47213595499958
    >>>

如果要执行其他的复数函数比如正弦、余弦或平方根，使用 ``cmath`` 模块：

.. code-block:: python

    >>> import cmath
    >>> cmath.sin(a)
    (24.83130584894638-11.356612711218174j)
    >>> cmath.cos(a)
    (-11.36423470640106-24.814651485634187j)
    >>> cmath.exp(a)
    (-4.829809383269385-5.5920560936409816j)
    >>>

----------
讨论
----------
Python中大部分与数学相关的模块都能处理复数。
比如如果你使用 ``numpy`` ，可以很容易的构造一个复数数组并在这个数组上执行各种操作：

.. code-block:: python

    >>> import numpy as np
    >>> a = np.array([2+3j, 4+5j, 6-7j, 8+9j])
    >>> a
    array([ 2.+3.j, 4.+5.j, 6.-7.j, 8.+9.j])
    >>> a + 2
    array([ 4.+3.j, 6.+5.j, 8.-7.j, 10.+9.j])
    >>> np.sin(a)
    array([ 9.15449915 -4.16890696j, -56.16227422 -48.50245524j,
            -153.20827755-526.47684926j, 4008.42651446-589.49948373j])
    >>>

Python的标准数学函数确实情况下并不能产生复数值，因此你的代码中不可能会出现复数返回值。比如：

.. code-block:: python

    >>> import math
    >>> math.sqrt(-1)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ValueError: math domain error
    >>>

如果你想生成一个复数返回结果，你必须显示的使用 ``cmath`` 模块，或者在某个支持复数的库中声明复数类型的使用。比如：

.. code-block:: python

    >>> import cmath
    >>> cmath.sqrt(-1)
    1j
    >>>

============================
3.7 无穷大与NaN
============================

----------
问题
----------
你想创建或测试正无穷、负无穷或NaN(非数字)的浮点数。

----------
解决方案
----------
Python并没有特殊的语法来表示这些特殊的浮点值，但是可以使用 ``float()`` 来创建它们。比如：

.. code-block:: python

    >>> a = float('inf')
    >>> b = float('-inf')
    >>> c = float('nan')
    >>> a
    inf
    >>> b
    -inf
    >>> c
    nan
    >>>

为了测试这些值的存在，使用 ``math.isinf()`` 和 ``math.isnan()`` 函数。比如：

.. code-block:: python

    >>> math.isinf(a)
    True
    >>> math.isnan(c)
    True
    >>>

----------
讨论
----------
想了解更多这些特殊浮点值的信息，可以参考IEEE 754规范。
然而，也有一些地方需要你特别注意，特别是跟比较和操作符相关的时候。

无穷大数在执行数学计算的时候会传播，比如：

.. code-block:: python

    >>> a = float('inf')
    >>> a + 45
    inf
    >>> a * 10
    inf
    >>> 10 / a
    0.0
    >>>

但是有些操作时未定义的并会返回一个NaN结果。比如：

.. code-block:: python

    >>> a = float('inf')
    >>> a/a
    nan
    >>> b = float('-inf')
    >>> a + b
    nan
    >>>

NaN值会在所有操作中传播，而不会产生异常。比如：

.. code-block:: python

    >>> c = float('nan')
    >>> c + 23
    nan
    >>> c / 2
    nan
    >>> c * 2
    nan
    >>> math.sqrt(c)
    nan
    >>>

NaN值的一个特别的地方时它们之间的比较操作总是返回False。比如：

.. code-block:: python

    >>> c = float('nan')
    >>> d = float('nan')
    >>> c == d
    False
    >>> c is d
    False
    >>>

由于这个原因，测试一个NaN值得唯一安全的方法就是使用 ``math.isnan()`` ，也就是上面演示的那样。

有时候程序员想改变Python默认行为，在返回无穷大或NaN结果的操作中抛出异常。
``fpectl`` 模块可以用来改变这种行为，但是它在标准的Python构建中并没有被启用，它是平台相关的，
并且针对的是专家级程序员。可以参考在线的Python文档获取更多的细节。

============================
3.8 分数运算
============================

----------
问题
----------
你进入时间机器，突然发现你正在做小学家庭作业，并涉及到分数计算问题。
或者你可能需要写代码去计算在你的木工工厂中的测量值。

----------
解决方案
----------
``fractions`` 模块可以被用来执行包含分数的数学运算。比如：

.. code-block:: python

    >>> from fractions import Fraction
    >>> a = Fraction(5, 4)
    >>> b = Fraction(7, 16)
    >>> print(a + b)
    27/16
    >>> print(a * b)
    35/64

    >>> # Getting numerator/denominator
    >>> c = a * b
    >>> c.numerator
    35
    >>> c.denominator
    64

    >>> # Converting to a float
    >>> float(c)
    0.546875

    >>> # Limiting the denominator of a value
    >>> print(c.limit_denominator(8))
    4/7

    >>> # Converting a float to a fraction
    >>> x = 3.75
    >>> y = Fraction(*x.as_integer_ratio())
    >>> y
    Fraction(15, 4)
    >>>

----------
讨论
----------
在大多数程序中一般不会出现分数的计算问题，但是有时候还是需要用到的。
比如，在一个允许接受分数形式的测试单位并以分数形式执行运算的程序中，
直接使用分数可以减少手动转换为小数或浮点数的工作。

========================
3.9 大型数组运算
========================

----------
问题
----------
你需要在大数据集(比如数组或网格)上面执行计算。

----------
解决方案
----------
涉及到数组的重量级运算操作，可以使用 ``NumPy`` 库。
``NumPy`` 的一个主要特征是它会给Python提供一个数组对象，相比标准的Python列表而已更适合用来做数学运算。
下面是一个简单的小例子，向你展示标准列表对象和 ``NumPy`` 数组对象之间的差别：

.. code-block:: python

    >>> # Python lists
    >>> x = [1, 2, 3, 4]
    >>> y = [5, 6, 7, 8]
    >>> x * 2
    [1, 2, 3, 4, 1, 2, 3, 4]
    >>> x + 10
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: can only concatenate list (not "int") to list
    >>> x + y
    [1, 2, 3, 4, 5, 6, 7, 8]

    >>> # Numpy arrays
    >>> import numpy as np
    >>> ax = np.array([1, 2, 3, 4])
    >>> ay = np.array([5, 6, 7, 8])
    >>> ax * 2
    array([2, 4, 6, 8])
    >>> ax + 10
    array([11, 12, 13, 14])
    >>> ax + ay
    array([ 6, 8, 10, 12])
    >>> ax * ay
    array([ 5, 12, 21, 32])
    >>>

正如所见，两种方案中数组的基本数学运算结果并不相同。
特别的， ``NumPy`` 中的标量运算(比如 ``ax * 2`` 或 ``ax + 10`` )会作用在每一个元素上。
另外，当两个操作数都是数组的时候执行元素对等位置计算，并最终生成一个新的数组。

对整个数组中所有元素同时执行数学运算可以使得作用在整个数组上的函数运算简单而又快速。
比如，如果你想计算多项式的值，可以这样做：

.. code-block:: python

    >>> def f(x):
    ... return 3*x**2 - 2*x + 7
    ...
    >>> f(ax)
    array([ 8, 15, 28, 47])
    >>>

``NumPy`` 还为数组操作提供了大量的通用函数，这些函数可以作为 ``math`` 模块中类似函数的替代。比如：

.. code-block:: python

    >>> np.sqrt(ax)
    array([ 1. , 1.41421356, 1.73205081, 2. ])
    >>> np.cos(ax)
    array([ 0.54030231, -0.41614684, -0.9899925 , -0.65364362])
    >>>

使用这些通用函数要比循环数组并使用 ``math`` 模块中的函数执行计算要快的多。
因此，只要有可能的话尽量选择 ``NumPy`` 的数组方案。

底层实现中， ``NumPy`` 数组使用了C或者Fortran语言的机制分配内存。
也就是说，它们是一个非常大的连续的并由同类型数据组成的内存区域。
所以，你可以构造一个比普通Python列表大的多的数组。
比如，如果你想构造一个10,000*10,000的浮点数二维网格，很轻松：

.. code-block:: python

    >>> grid = np.zeros(shape=(10000,10000), dtype=float)
    >>> grid
        array([[ 0., 0., 0., ..., 0., 0., 0.],
        [ 0., 0., 0., ..., 0., 0., 0.],
        [ 0., 0., 0., ..., 0., 0., 0.],
        ...,
        [ 0., 0., 0., ..., 0., 0., 0.],
        [ 0., 0., 0., ..., 0., 0., 0.],
        [ 0., 0., 0., ..., 0., 0., 0.]])
    >>>

所有的普通操作还是会同时作用在所有元素上：

.. code-block:: python

    >>> grid += 10
    >>> grid
    array([[ 10., 10., 10., ..., 10., 10., 10.],
        [ 10., 10., 10., ..., 10., 10., 10.],
        [ 10., 10., 10., ..., 10., 10., 10.],
        ...,
        [ 10., 10., 10., ..., 10., 10., 10.],
        [ 10., 10., 10., ..., 10., 10., 10.],
        [ 10., 10., 10., ..., 10., 10., 10.]])
    >>> np.sin(grid)
    array([[-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111],
        [-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111],
        [-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111],
        ...,
        [-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111],
        [-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111],
        [-0.54402111, -0.54402111, -0.54402111, ..., -0.54402111,
            -0.54402111, -0.54402111]])
    >>>

关于 ``NumPy`` 有一点需要特别的主意，那就是它扩展Python列表的索引功能 - 特别是对于多维数组。
为了说明清楚，先构造一个简单的二维数组并试着做些试验：

.. code-block:: python

    >>> a = np.array([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]])
    >>> a
    array([[ 1, 2, 3, 4],
    [ 5, 6, 7, 8],
    [ 9, 10, 11, 12]])

    >>> # Select row 1
    >>> a[1]
    array([5, 6, 7, 8])

    >>> # Select column 1
    >>> a[:,1]
    array([ 2, 6, 10])

    >>> # Select a subregion and change it
    >>> a[1:3, 1:3]
    array([[ 6, 7],
            [10, 11]])
    >>> a[1:3, 1:3] += 10
    >>> a
    array([[ 1, 2, 3, 4],
            [ 5, 16, 17, 8],
            [ 9, 20, 21, 12]])

    >>> # Broadcast a row vector across an operation on all rows
    >>> a + [100, 101, 102, 103]
    array([[101, 103, 105, 107],
            [105, 117, 119, 111],
            [109, 121, 123, 115]])
    >>> a
    array([[ 1, 2, 3, 4],
            [ 5, 16, 17, 8],
            [ 9, 20, 21, 12]])

    >>> # Conditional assignment on an array
    >>> np.where(a < 10, a, 10)
    array([[ 1, 2, 3, 4],
            [ 5, 10, 10, 8],
            [ 9, 10, 10, 10]])
    >>>

----------
讨论
----------
``NumPy`` 是Python领域中很多科学与工程库的基础，同时也是被广泛使用的最大最复杂的模块。
即便如此，在刚开始的时候通过一些简单的例子和玩具程序也能帮我们完成一些有趣的事情。

通常我们导入 ``NumPy`` 模块的时候会使用语句 ``import numpy as np`` 。
这样的话你就不用再你的程序里面一遍遍的敲入 ``numpy`` ，只需要输入 ``np`` 就行了，节省了不少时间。

如果想获取更多的信息，你当然得去 ``NumPy`` 官网逛逛了，网址是： http://www.numpy.org

============================
3.10 矩阵与线性代数运算
============================

----------
问题
----------
你需要执行矩阵和线性代数运算，比如矩阵乘法、寻找行列式、求解线性方程组等等。

----------
解决方案
----------
 ``NumPy`` 库有一个矩阵对象可以用来解决这个问题。
矩阵类似于3.9小节中数组对象，但是遵循线性代数的计算规则。下面的一个例子展示了矩阵的一些基本特性：

.. code-block:: python

    >>> import numpy as np
    >>> m = np.matrix([[1,-2,3],[0,4,5],[7,8,-9]])
    >>> m
    matrix([[ 1, -2, 3],
            [ 0, 4, 5],
            [ 7, 8, -9]])

    >>> # Return transpose
    >>> m.T
    matrix([[ 1, 0, 7],
            [-2, 4, 8],
            [ 3, 5, -9]])

    >>> # Return inverse
    >>> m.I
    matrix([[ 0.33043478, -0.02608696, 0.09565217],
            [-0.15217391, 0.13043478, 0.02173913],
            [ 0.12173913, 0.09565217, -0.0173913 ]])

    >>> # Create a vector and multiply
    >>> v = np.matrix([[2],[3],[4]])
    >>> v
    matrix([[2],
            [3],
            [4]])
    >>> m * v
    matrix([[ 8],
            [32],
            [ 2]])
    >>>

可以在 ``numpy.linalg`` 子包中找到更多的操作函数，比如：

.. code-block:: python

    >>> import numpy.linalg

    >>> # Determinant
    >>> numpy.linalg.det(m)
    -229.99999999999983

    >>> # Eigenvalues
    >>> numpy.linalg.eigvals(m)
    array([-13.11474312, 2.75956154, 6.35518158])

    >>> # Solve for x in mx = v
    >>> x = numpy.linalg.solve(m, v)
    >>> x
    matrix([[ 0.96521739],
            [ 0.17391304],
            [ 0.46086957]])
    >>> m * x
    matrix([[ 2.],
            [ 3.],
            [ 4.]])
    >>> v
    matrix([[2],
            [3],
            [4]])
    >>>

----------
讨论
----------
很显然线性代数是个非常大的主题，已经超出了本书能讨论的范围。
但是，如果你需要操作数组和向量的话， ``NumPy`` 是一个不错的入口点。
可以访问 ``NumPy`` 官网 http://www.numpy.org 获取更多信息。

============================
3.11 随机选择
============================

----------
问题
----------
你想从一个序列中随机抽取若干元素，或者想生成几个随机数。

----------
解决方案
----------
``random`` 模块有大量的函数用来产生随机数和随机选择元素。
比如，要想从一个序列中随机的抽取一个元素，可以使用 ``random.choice()`` ：

.. code-block:: python

    >>> import random
    >>> values = [1, 2, 3, 4, 5, 6]
    >>> random.choice(values)
    2
    >>> random.choice(values)
    3
    >>> random.choice(values)
    1
    >>> random.choice(values)
    4
    >>> random.choice(values)
    6
    >>>

为了提取出N个不同元素的样本用来做进一步的操作，可以使用 ``random.sample()`` ：

.. code-block:: python

    >>> random.sample(values, 2)
    [6, 2]
    >>> random.sample(values, 2)
    [4, 3]
    >>> random.sample(values, 3)
    [4, 3, 1]
    >>> random.sample(values, 3)
    [5, 4, 1]
    >>>

如果你仅仅只是想打乱序列中元素的顺序，可以使用 ``random.shuffle()`` ：

.. code-block:: python

    >>> random.shuffle(values)
    >>> values
    [2, 4, 6, 5, 3, 1]
    >>> random.shuffle(values)
    >>> values
    [3, 5, 2, 1, 6, 4]
    >>>

生成随机整数，请使用 ``random.randint()`` ：

.. code-block:: python

    >>> random.randint(0,10)
    2
    >>> random.randint(0,10)
    5
    >>> random.randint(0,10)
    0
    >>> random.randint(0,10)
    7
    >>> random.randint(0,10)
    10
    >>> random.randint(0,10)
    3
    >>>

为了生成0到1范围内均匀分布的浮点数，使用 ``random.random()`` ：

.. code-block:: python

    >>> random.random()
    0.9406677561675867
    >>> random.random()
    0.133129581343897
    >>> random.random()
    0.4144991136919316
    >>>

如果要获取N位随机位(二进制)的整数，使用 ``random.getrandbits()`` ：

.. code-block:: python

    >>> random.getrandbits(200)
    335837000776573622800628485064121869519521710558559406913275
    >>>

----------
讨论
----------
``random`` 模块使用 *Mersenne Twister* 算法来计算生成随机数。这是一个确定性算法，
但是你可以通过 ``random.seed()`` 函数修改初始化种子。比如：

.. code-block:: python

    random.seed() # Seed based on system time or os.urandom()
    random.seed(12345) # Seed based on integer given
    random.seed(b'bytedata') # Seed based on byte data

除了上述介绍的功能，random模块还包含基于均匀分布、高斯分布和其他分布的随机数生成函数。
比如， ``random.uniform()`` 计算均匀分布随机数， ``random.gauss()`` 计算正态分布随机数。
对于其他的分布情况请参考在线文档。

在 ``random`` 模块中的函数不应该用在和密码学相关的程序中。
如果你确实需要类似的功能，可以使用ssl模块中相应的函数。
比如， ``ssl.RAND_bytes()`` 可以用来生成一个安全的随机字节序列。
============================
3.12 基本的日期与时间转换
============================

----------
问题
----------
你需要执行简单的时间转换，比如天到秒，小时到分钟等的转换。

----------
解决方案
----------
为了执行不同时间单位的转换和计算，请使用 ``datetime`` 模块。
比如，为了表示一个时间段，可以创建一个 ``timedelta`` 实例，就像下面这样：

.. code-block:: python

    >>> from datetime import timedelta
    >>> a = timedelta(days=2, hours=6)
    >>> b = timedelta(hours=4.5)
    >>> c = a + b
    >>> c.days
    2
    >>> c.seconds
    37800
    >>> c.seconds / 3600
    10.5
    >>> c.total_seconds() / 3600
    58.5
    >>>

如果你想表示指定的日期和时间，先创建一个 ``datetime`` 实例然后使用标准的数学运算来操作它们。比如：

.. code-block:: python

    >>> from datetime import datetime
    >>> a = datetime(2012, 9, 23)
    >>> print(a + timedelta(days=10))
    2012-10-03 00:00:00
    >>>
    >>> b = datetime(2012, 12, 21)
    >>> d = b - a
    >>> d.days
    89
    >>> now = datetime.today()
    >>> print(now)
    2012-12-21 14:54:43.094063
    >>> print(now + timedelta(minutes=10))
    2012-12-21 15:04:43.094063
    >>>

在计算的时候，需要注意的是 ``datetime`` 会自动处理闰年。比如：

.. code-block:: python

    >>> a = datetime(2012, 3, 1)
    >>> b = datetime(2012, 2, 28)
    >>> a - b
    datetime.timedelta(2)
    >>> (a - b).days
    2
    >>> c = datetime(2013, 3, 1)
    >>> d = datetime(2013, 2, 28)
    >>> (c - d).days
    1
    >>>

----------
讨论
----------
对大多数基本的日期和时间处理问题， ``datetime`` 模块已经足够了。
如果你需要执行更加复杂的日期操作，比如处理时区，模糊时间范围，节假日计算等等，
可以考虑使用 `dateutil模块 <http://pypi.python.org/pypi/python-dateutil>`_

许多类似的时间计算可以使用 ``dateutil.relativedelta()`` 函数代替。
但是，有一点需要注意的就是，它会在处理月份(还有它们的天数差距)的时候填充间隙。看例子最清楚：

.. code-block:: python

    >>> a = datetime(2012, 9, 23)
    >>> a + timedelta(months=1)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: 'months' is an invalid keyword argument for this function
    >>>
    >>> from dateutil.relativedelta import relativedelta
    >>> a + relativedelta(months=+1)
    datetime.datetime(2012, 10, 23, 0, 0)
    >>> a + relativedelta(months=+4)
    datetime.datetime(2013, 1, 23, 0, 0)
    >>>
    >>> # Time between two dates
    >>> b = datetime(2012, 12, 21)
    >>> d = b - a
    >>> d
    datetime.timedelta(89)
    >>> d = relativedelta(b, a)
    >>> d
    relativedelta(months=+2, days=+28)
    >>> d.months
    2
    >>> d.days
    28
    >>>
============================
3.13 计算最后一个周五的日期
============================

----------
问题
----------
你需要查找星期中某一天最后出现的日期，比如星期五。

----------
解决方案
----------
Python的 ``datetime`` 模块中有工具函数和类可以帮助你执行这样的计算。
下面是对类似这样的问题的一个通用解决方案：

.. code-block:: python

    #!/usr/bin/env python
    # -*- encoding: utf-8 -*-
    """
    Topic: 最后的周五
    Desc :
    """
    from datetime import datetime, timedelta

    weekdays = ['Monday', 'Tuesday', 'Wednesday', 'Thursday',
                'Friday', 'Saturday', 'Sunday']


    def get_previous_byday(dayname, start_date=None):
        if start_date is None:
            start_date = datetime.today()
        day_num = start_date.weekday()
        day_num_target = weekdays.index(dayname)
        days_ago = (7 + day_num - day_num_target) % 7
        if days_ago == 0:
            days_ago = 7
        target_date = start_date - timedelta(days=days_ago)
        return target_date

在交互式解释器中使用如下：

.. code-block:: python

    >>> datetime.today() # For reference
    datetime.datetime(2012, 8, 28, 22, 4, 30, 263076)
    >>> get_previous_byday('Monday')
    datetime.datetime(2012, 8, 27, 22, 3, 57, 29045)
    >>> get_previous_byday('Tuesday') # Previous week, not today
    datetime.datetime(2012, 8, 21, 22, 4, 12, 629771)
    >>> get_previous_byday('Friday')
    datetime.datetime(2012, 8, 24, 22, 5, 9, 911393)
    >>>

可选的 ``start_date`` 参数可以由另外一个 ``datetime`` 实例来提供。比如：

.. code-block:: python

    >>> get_previous_byday('Sunday', datetime(2012, 12, 21))
    datetime.datetime(2012, 12, 16, 0, 0)
    >>>

----------
讨论
----------
上面的算法原理是这样的：先将开始日期和目标日期映射到星期数组的位置上(星期一索引为0)，
然后通过模运算计算出目标日期要经过多少天才能到达开始日期。然后用开始日期减去那个时间差即得到结果日期。

如果你要像这样执行大量的日期计算的话，你最好安装第三方包 ``python-dateutil`` 来代替。
比如，下面是是使用 ``dateutil`` 模块中的 ``relativedelta()`` 函数执行同样的计算：

.. code-block:: python

    >>> from datetime import datetime
    >>> from dateutil.relativedelta import relativedelta
    >>> from dateutil.rrule import *
    >>> d = datetime.now()
    >>> print(d)
    2012-12-23 16:31:52.718111

    >>> # Next Friday
    >>> print(d + relativedelta(weekday=FR))
    2012-12-28 16:31:52.718111
    >>>

    >>> # Last Friday
    >>> print(d + relativedelta(weekday=FR(-1)))
    2012-12-21 16:31:52.718111
    >>>

==========================
3.14 计算当前月份的日期范围
==========================

----------
问题
----------
你的代码需要在当前月份中循环每一天，想找到一个计算这个日期范围的高效方法。

----------
解决方案
----------
在这样的日期上循环并需要事先构造一个包含所有日期的列表。
你可以先计算出开始日期和结束日期，
然后在你步进的时候使用 ``datetime.timedelta`` 对象递增这个日期变量即可。

下面是一个接受任意 ``datetime`` 对象并返回一个由当前月份开始日和下个月开始日组成的元组对象。

.. code-block:: python

    from datetime import datetime, date, timedelta
    import calendar

    def get_month_range(start_date=None):
        if start_date is None:
            start_date = date.today().replace(day=1)
        _, days_in_month = calendar.monthrange(start_date.year, start_date.month)
        end_date = start_date + timedelta(days=days_in_month)
        return (start_date, end_date)

有了这个就可以很容易的在返回的日期范围上面做循环操作了：

.. code-block:: python

    >>> a_day = timedelta(days=1)
    >>> first_day, last_day = get_month_range()
    >>> while first_day < last_day:
    ...     print(first_day)
    ...     first_day += a_day
    ...
    2012-08-01
    2012-08-02
    2012-08-03
    2012-08-04
    2012-08-05
    2012-08-06
    2012-08-07
    2012-08-08
    2012-08-09
    #... and so on...

----------
讨论
----------
上面的代码先计算出一个对应月份第一天的日期。
一个快速的方法就是使用 ``date`` 或 ``datetime`` 对象的 ``replace()`` 方法简单的将 ``days`` 属性设置成1即可。
``replace()`` 方法一个好处就是它会创建和你开始传入对象类型相同的对象。
所以，如果输入参数是一个 ``date`` 实例，那么结果也是一个 ``date`` 实例。
同样的，如果输入是一个 ``datetime`` 实例，那么你得到的就是一个 ``datetime`` 实例。

然后，使用 ``calendar.monthrange()`` 函数来找出该月的总天数。
任何时候只要你想获得日历信息，那么 ``calendar`` 模块就非常有用了。
``monthrange()`` 函数会返回包含星期和该月天数的元组。

一旦该月的天数已知了，那么结束日期就可以通过在开始日期上面加上这个天数获得。
有个需要注意的是结束日期并不包含在这个日期范围内(事实上它是下个月的开始日期)。
这个和Python的 ``slice`` 与 ``range`` 操作行为保持一致，同样也不包含结尾。

为了在日期范围上循环，要使用到标准的数学和比较操作。
比如，可以利用 ``timedelta`` 实例来递增日期，小于号<用来检查一个日期是否在结束日期之前。

理想情况下，如果能为日期迭代创建一个同内置的 ``range()`` 函数一样的函数就好了。
幸运的是，可以使用一个生成器来很容易的实现这个目标：

.. code-block:: python

    def date_range(start, stop, step):
        while start < stop:
            yield start
            start += step

下面是使用这个生成器的例子：

.. code-block:: python

    >>> for d in date_range(datetime(2012, 9, 1), datetime(2012,10,1),
                            timedelta(hours=6)):
    ...     print(d)
    ...
    2012-09-01 00:00:00
    2012-09-01 06:00:00
    2012-09-01 12:00:00
    2012-09-01 18:00:00
    2012-09-02 00:00:00
    2012-09-02 06:00:00
    ...
    >>>

这种实现之所以这么简单，还得归功于Python中的日期和时间能够使用标准的数学和比较操作符来进行运算。

============================
3.15 字符串转换为日期
============================

----------
问题
----------
你的应用程序接受字符串格式的输入，但是你想将它们转换为 ``datetime`` 对象以便在上面执行非字符串操作。

----------
解决方案
----------
使用Python的标准模块 ``datetime`` 可以很容易的解决这个问题。比如：

.. code-block:: python

    >>> from datetime import datetime
    >>> text = '2012-09-20'
    >>> y = datetime.strptime(text, '%Y-%m-%d')
    >>> z = datetime.now()
    >>> diff = z - y
    >>> diff
    datetime.timedelta(3, 77824, 177393)
    >>>

----------
讨论
----------
``datetime.strptime()`` 方法支持很多的格式化代码，
比如 ``%Y`` 代表4位数年份， ``%m`` 代表两位数月份。
还有一点值得注意的是这些格式化占位符也可以反过来使用，将日期输出为指定的格式字符串形式。

比如，假设你的代码中生成了一个 ``datetime`` 对象，
你想将它格式化为漂亮易读形式后放在自动生成的信件或者报告的顶部：

.. code-block:: python

    >>> z
    datetime.datetime(2012, 9, 23, 21, 37, 4, 177393)
    >>> nice_z = datetime.strftime(z, '%A %B %d, %Y')
    >>> nice_z
    'Sunday September 23, 2012'
    >>>

还有一点需要注意的是， ``strptime()`` 的性能要比你想象中的差很多，
因为它是使用纯Python实现，并且必须处理所有的系统本地设置。
如果你要在代码中需要解析大量的日期并且已经知道了日期字符串的确切格式，可以自己实现一套解析方案来获取更好的性能。
比如，如果你已经知道所以日期格式是 ``YYYY-MM-DD`` ，你可以像下面这样实现一个解析函数：

.. code-block:: python

    from datetime import datetime
    def parse_ymd(s):
        year_s, mon_s, day_s = s.split('-')
        return datetime(int(year_s), int(mon_s), int(day_s))

实际测试中，这个函数比 ``datetime.strptime()`` 快7倍多。
如果你要处理大量的涉及到日期的数据的话，那么最好考虑下这个方案！
============================
3.16 结合时区的日期操作
============================

----------
问题
----------
你有一个安排在2012年12月21日早上9:30的电话会议，地点在芝加哥。
而你的朋友在印度的班加罗尔，那么他应该在当地时间几点参加这个会议呢？


----------
解决方案
----------
对几乎所有涉及到时区的问题，你都应该使用 ``pytz`` 模块。这个包提供了Olson时区数据库，
它是时区信息的事实上的标准，在很多语言和操作系统里面都可以找到。

``pytz`` 模块一个主要用途是将 ``datetime`` 库创建的简单日期对象本地化。
比如，下面如何表示一个芝加哥时间的示例：

.. code-block:: python

    >>> from datetime import datetime
    >>> from pytz import timezone
    >>> d = datetime(2012, 12, 21, 9, 30, 0)
    >>> print(d)
    2012-12-21 09:30:00
    >>>

    >>> # Localize the date for Chicago
    >>> central = timezone('US/Central')
    >>> loc_d = central.localize(d)
    >>> print(loc_d)
    2012-12-21 09:30:00-06:00
    >>>

一旦日期被本地化了， 它就可以转换为其他时区的时间了。
为了得到班加罗尔对应的时间，你可以这样做：

.. code-block:: python

    >>> # Convert to Bangalore time
    >>> bang_d = loc_d.astimezone(timezone('Asia/Kolkata'))
    >>> print(bang_d)
    2012-12-21 21:00:00+05:30
    >>>

如果你打算在本地化日期上执行计算，你需要特别注意夏令时转换和其他细节。
比如，在2013年，美国标准夏令时时间开始于本地时间3月13日凌晨2:00(在那时，时间向前跳过一小时)。
如果你正在执行本地计算，你会得到一个错误。比如：

.. code-block:: python

    >>> d = datetime(2013, 3, 10, 1, 45)
    >>> loc_d = central.localize(d)
    >>> print(loc_d)
    2013-03-10 01:45:00-06:00
    >>> later = loc_d + timedelta(minutes=30)
    >>> print(later)
    2013-03-10 02:15:00-06:00 # WRONG! WRONG!
    >>>

结果错误是因为它并没有考虑在本地时间中有一小时的跳跃。
为了修正这个错误，可以使用时区对象 ``normalize()`` 方法。比如：

.. code-block:: python

    >>> from datetime import timedelta
    >>> later = central.normalize(loc_d + timedelta(minutes=30))
    >>> print(later)
    2013-03-10 03:15:00-05:00
    >>>

----------
讨论
----------
为了不让你被这些东东弄的晕头转向，处理本地化日期的通常的策略先将所有日期转换为UTC时间，
并用它来执行所有的中间存储和操作。比如：

.. code-block:: python

    >>> print(loc_d)
    2013-03-10 01:45:00-06:00
    >>> utc_d = loc_d.astimezone(pytz.utc)
    >>> print(utc_d)
    2013-03-10 07:45:00+00:00
    >>>

一旦转换为UTC，你就不用去担心跟夏令时相关的问题了。
因此，你可以跟之前一样放心的执行常见的日期计算。
当你想将输出变为本地时间的时候，使用合适的时区去转换下就行了。比如：

.. code-block:: python

    >>> later_utc = utc_d + timedelta(minutes=30)
    >>> print(later_utc.astimezone(central))
    2013-03-10 03:15:00-05:00
    >>>

当涉及到时区操作的时候，有个问题就是我们如何得到时区的名称。
比如，在这个例子中，我们如何知道“Asia/Kolkata”就是印度对应的时区名呢？
为了查找，可以使用ISO 3166国家代码作为关键字去查阅字典 ``pytz.country_timezones`` 。比如：

.. code-block:: python

    >>> pytz.country_timezones['IN']
    ['Asia/Kolkata']
    >>>

注：当你阅读到这里的时候，有可能 ``pytz`` 模块已经不再建议使用了，因为PEP431提出了更先进的时区支持。
但是这里谈到的很多问题还是有参考价值的(比如使用UTC日期的建议等)。
========================
4.1 手动遍历迭代器
========================

----------
问题
----------
你想遍历一个可迭代对象中的所有元素，但是却不想使用for循环。

----------
解决方案
----------
为了手动的遍历可迭代对象，使用 ``next()`` 函数并在代码中捕获 ``StopIteration`` 异常。
比如，下面的例子手动读取一个文件中的所有行：

.. code-block:: python

    def manual_iter():
        with open('/etc/passwd') as f:
            try:
                while True:
                    line = next(f)
                    print(line, end='')
            except StopIteration:
                pass

通常来讲， ``StopIteration`` 用来指示迭代的结尾。
然而，如果你手动使用上面演示的 ``next()`` 函数的话，你还可以通过返回一个指定值来标记结尾，比如 ``None`` 。
下面是示例：

.. code-block:: python

    with open('/etc/passwd') as f:
        while True:
            line = next(f, None)
            if line is None:
                break
            print(line, end='')

----------
讨论
----------
大多数情况下，我们会使用 ``for`` 循环语句用来遍历一个可迭代对象。
但是，偶尔也需要对迭代做更加精确的控制，这时候了解底层迭代机制就显得尤为重要了。

下面的交互示例向我们演示了迭代期间所发生的基本细节：

.. code-block:: python

    >>> items = [1, 2, 3]
    >>> # Get the iterator
    >>> it = iter(items) # Invokes items.__iter__()
    >>> # Run the iterator
    >>> next(it) # Invokes it.__next__()
    1
    >>> next(it)
    2
    >>> next(it)
    3
    >>> next(it)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    StopIteration
    >>>

本章接下来几小节会更深入的讲解迭代相关技术，前提是你先要理解基本的迭代协议机制。
所以确保你已经把这章的内容牢牢记在心中。

============================
4.2 代理迭代
============================

----------
问题
----------
你构建了一个自定义容器对象，里面包含有列表、元组或其他可迭代对象。
你想直接在你的这个新容器对象上执行迭代操作。

----------
解决方案
----------
实际上你只需要定义一个 ``__iter__()`` 方法，将迭代操作代理到容器内部的对象上去。比如：

.. code-block:: python

    class Node:
        def __init__(self, value):
            self._value = value
            self._children = []

        def __repr__(self):
            return 'Node({!r})'.format(self._value)

        def add_child(self, node):
            self._children.append(node)

        def __iter__(self):
            return iter(self._children)

    # Example
    if __name__ == '__main__':
        root = Node(0)
        child1 = Node(1)
        child2 = Node(2)
        root.add_child(child1)
        root.add_child(child2)
        # Outputs Node(1), Node(2)
        for ch in root:
            print(ch)

在上面代码中， ``__iter__()`` 方法只是简单的将迭代请求传递给内部的 ``_children`` 属性。

----------
讨论
----------
Python的迭代器协议需要 ``__iter__()`` 方法返回一个实现了 ``__next__()`` 方法的迭代器对象。
如果你只是迭代遍历其他容器的内容，你无须担心底层是怎样实现的。你所要做的只是传递迭代请求既可。

这里的 ``iter()`` 函数的使用简化了代码，
``iter(s)`` 只是简单的通过调用 ``s.__iter__()`` 方法来返回对应的迭代器对象，
就跟 ``len(s)`` 会调用 ``s.__len__()`` 原理是一样的。

============================
4.3 使用生成器创建新的迭代模式
============================

----------
问题
----------
你想实现一个自定义迭代模式，跟普通的内置函数比如 ``range()`` , ``reversed()`` 不一样。

----------
解决方案
----------
如果你想实现一种新的迭代模式，使用一个生成器函数来定义它。
下面是一个生产某个范围内浮点数的生成器：

.. code-block:: python

    def frange(start, stop, increment):
        x = start
        while x < stop:
            yield x
            x += increment

为了使用这个函数，
你可以用for循环迭代它或者使用其他接受一个可迭代对象的函数(比如 ``sum()`` , ``list()`` 等)。示例如下：

.. code-block:: python

    >>> for n in frange(0, 4, 0.5):
    ...     print(n)
    ...
    0
    0.5
    1.0
    1.5
    2.0
    2.5
    3.0
    3.5
    >>> list(frange(0, 1, 0.125))
    [0, 0.125, 0.25, 0.375, 0.5, 0.625, 0.75, 0.875]
    >>>

----------
讨论
----------
一个函数中需要有一个 ``yield`` 语句即可将其转换为一个生成器。
跟普通函数不同的是，生成器只能用于迭代操作。
下面是一个实验，向你展示这样的函数底层工作机制：

.. code-block:: python

    >>> def countdown(n):
    ...     print('Starting to count from', n)
    ...     while n > 0:
    ...         yield n
    ...         n -= 1
    ...     print('Done!')
    ...

    >>> # Create the generator, notice no output appears
    >>> c = countdown(3)
    >>> c
    <generator object countdown at 0x1006a0af0>

    >>> # Run to first yield and emit a value
    >>> next(c)
    Starting to count from 3
    3

    >>> # Run to the next yield
    >>> next(c)
    2

    >>> # Run to next yield
    >>> next(c)
    1

    >>> # Run to next yield (iteration stops)
    >>> next(c)
    Done!
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    StopIteration
    >>>

一个生成器函数主要特征是它只会回应在迭代中使用到的 *next* 操作。
一旦生成器函数返回退出，迭代终止。我们在迭代中通常使用的for语句会自动处理这些细节，所以你无需担心。

============================
4.4 实现迭代器协议
============================

----------
问题
----------
你想构建一个能支持迭代操作的自定义对象，并希望找到一个能实现迭代协议的简单方法。

----------
解决方案
----------
目前为止，在一个对象上实现迭代最简单的方式是使用一个生成器函数。
在4.2小节中，使用Node类来表示树形数据结构。你可能想实现一个以深度优先方式遍历树形节点的生成器。
下面是代码示例：

.. code-block:: python

    class Node:
        def __init__(self, value):
            self._value = value
            self._children = []

        def __repr__(self):
            return 'Node({!r})'.format(self._value)

        def add_child(self, node):
            self._children.append(node)

        def __iter__(self):
            return iter(self._children)

        def depth_first(self):
            yield self
            for c in self:
                yield from c.depth_first()

    # Example
    if __name__ == '__main__':
        root = Node(0)
        child1 = Node(1)
        child2 = Node(2)
        root.add_child(child1)
        root.add_child(child2)
        child1.add_child(Node(3))
        child1.add_child(Node(4))
        child2.add_child(Node(5))

        for ch in root.depth_first():
            print(ch)
        # Outputs Node(0), Node(1), Node(3), Node(4), Node(2), Node(5)

在这段代码中，``depth_first()`` 方法简单直观。
它首先返回自己本身并迭代每一个子节点并
通过调用子节点的 ``depth_first()`` 方法(使用 ``yield from`` 语句)返回对应元素。

----------
讨论
----------
Python的迭代协议要求一个 ``__iter__()`` 方法返回一个特殊的迭代器对象，
这个迭代器对象实现了 ``__next__()`` 方法并通过 ``StopIteration`` 异常标识迭代的完成。
但是，实现这些通常会比较繁琐。
下面我们演示下这种方式，如何使用一个关联迭代器类重新实现 ``depth_first()`` 方法：


.. code-block:: python

    class Node2:
        def __init__(self, value):
            self._value = value
            self._children = []

        def __repr__(self):
            return 'Node({!r})'.format(self._value)

        def add_child(self, node):
            self._children.append(node)

        def __iter__(self):
            return iter(self._children)

        def depth_first(self):
            return DepthFirstIterator(self)


    class DepthFirstIterator(object):
        '''
        Depth-first traversal
        '''

        def __init__(self, start_node):
            self._node = start_node
            self._children_iter = None
            self._child_iter = None

        def __iter__(self):
            return self

        def __next__(self):
            # Return myself if just started; create an iterator for children
            if self._children_iter is None:
                self._children_iter = iter(self._node)
                return self._node
            # If processing a child, return its next item
            elif self._child_iter:
                try:
                    nextchild = next(self._child_iter)
                    return nextchild
                except StopIteration:
                    self._child_iter = None
                    return next(self)
            # Advance to the next child and start its iteration
            else:
                self._child_iter = next(self._children_iter).depth_first()
                return next(self)

``DepthFirstIterator`` 类和上面使用生成器的版本工作原理类似，
但是它写起来很繁琐，因为迭代器必须在迭代处理过程中维护大量的状态信息。
坦白来讲，没人愿意写这么晦涩的代码。将你的迭代器定义为一个生成器后一切迎刃而解。

============================
4.5 反向迭代
============================

----------
问题
----------
你想反方向迭代一个序列

----------
解决方案
----------
使用内置的 ``reversed()`` 函数，比如：

.. code-block:: python

    >>> a = [1, 2, 3, 4]
    >>> for x in reversed(a):
    ...     print(x)
    ...
    4
    3
    2
    1

反向迭代仅仅当对象的大小可预先确定或者对象实现了 ``__reversed__()`` 的特殊方法时才能生效。
如果两者都不符合，那你必须先将对象转换为一个列表才行，比如：

.. code-block:: python

    # Print a file backwards
    f = open('somefile')
    for line in reversed(list(f)):
        print(line, end='')

要注意的是如果可迭代对象元素很多的话，将其预先转换为一个列表要消耗大量的内存。

----------
讨论
----------
很多程序员并不知道可以通过在自定义类上实现 ``__reversed__()`` 方法来实现反向迭代。比如：

.. code-block:: python

    class Countdown:
        def __init__(self, start):
            self.start = start

        # Forward iterator
        def __iter__(self):
            n = self.start
            while n > 0:
                yield n
                n -= 1

        # Reverse iterator
        def __reversed__(self):
            n = 1
            while n <= self.start:
                yield n
                n += 1

    for rr in reversed(Countdown(30)):
        print(rr)
    for rr in Countdown(30):
        print(rr)

定义一个反向迭代器可以使得代码非常的高效，
因为它不再需要将数据填充到一个列表中然后再去反向迭代这个列表。
==========================
4.6 带有外部状态的生成器函数
==========================

----------
问题
----------
你想定义一个生成器函数，但是它会调用某个你想暴露给用户使用的外部状态值。

----------
解决方案
----------
如果你想让你的生成器暴露外部状态给用户，
别忘了你可以简单的将它实现为一个类，然后把生成器函数放到 ``__iter__()`` 方法中过去。比如：

.. code-block:: python

    from collections import deque

    class linehistory:
        def __init__(self, lines, histlen=3):
            self.lines = lines
            self.history = deque(maxlen=histlen)

        def __iter__(self):
            for lineno, line in enumerate(self.lines, 1):
                self.history.append((lineno, line))
                yield line

        def clear(self):
            self.history.clear()

为了使用这个类，你可以将它当做是一个普通的生成器函数。
然而，由于可以创建一个实例对象，于是你可以访问内部属性值，
比如 ``history`` 属性或者是 ``clear()`` 方法。代码示例如下：

.. code-block:: python

    with open('somefile.txt') as f:
        lines = linehistory(f)
        for line in lines:
            if 'python' in line:
                for lineno, hline in lines.history:
                    print('{}:{}'.format(lineno, hline), end='')

----------
讨论
----------
关于生成器，很容易掉进函数无所不能的陷阱。
如果生成器函数需要跟你的程序其他部分打交道的话(比如暴露属性值，允许通过方法调用来控制等等)，
可能会导致你的代码异常的复杂。
如果是这种情况的话，可以考虑使用上面介绍的定义类的方式。
在 ``__iter__()`` 方法中定义你的生成器不会改变你任何的算法逻辑。
由于它是类的一部分，所以允许你定义各种属性和方法来供用户使用。

一个需要注意的小地方是，如果你在迭代操作时不使用for循环语句，那么你得先调用 ``iter()`` 函数。比如：

.. code-block:: python

    >>> f = open('somefile.txt')
    >>> lines = linehistory(f)
    >>> next(lines)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: 'linehistory' object is not an iterator

    >>> # Call iter() first, then start iterating
    >>> it = iter(lines)
    >>> next(it)
    'hello world\n'
    >>> next(it)
    'this is a test\n'
    >>>
============================
4.7 迭代器切片
============================

----------
问题
----------
你想得到一个由迭代器生成的切片对象，但是标准切片操作并不能做到。

----------
解决方案
----------
函数 ``itertools.islice()`` 正好适用于在迭代器和生成器上做切片操作。比如：

.. code-block:: python

    >>> def count(n):
    ...     while True:
    ...         yield n
    ...         n += 1
    ...
    >>> c = count(0)
    >>> c[10:20]
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: 'generator' object is not subscriptable

    >>> # Now using islice()
    >>> import itertools
    >>> for x in itertools.islice(c, 10, 20):
    ...     print(x)
    ...
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    >>>

----------
讨论
----------
迭代器和生成器不能使用标准的切片操作，因为它们的长度事先我们并不知道(并且也没有实现索引)。
函数 ``islice()`` 返回一个可以生成指定元素的迭代器，它通过遍历并丢弃直到切片开始索引位置的所有元素。
然后才开始一个个的返回元素，并直到切片结束索引位置。

这里要着重强调的一点是 ``islice()`` 会消耗掉传入的迭代器中的数据。
必须考虑到迭代器是不可逆的这个事实。
所以如果你需要之后再次访问这个迭代器的话，那你就得先将它里面的数据放入一个列表中。
============================
4.8 跳过可迭代对象的开始部分
============================

----------
问题
----------
你想遍历一个可迭代对象，但是它开始的某些元素你并不感兴趣，想跳过它们。

----------
解决方案
----------
``itertools`` 模块中有一些函数可以完成这个任务。
首先介绍的是 ``itertools.dropwhile()`` 函数。使用时，你给它传递一个函数对象和一个可迭代对象。
它会返回一个迭代器对象，丢弃原有序列中直到函数返回Flase之前的所有元素，然后返回后面所有元素。

为了演示，假定你在读取一个开始部分是几行注释的源文件。比如：

.. code-block:: python

    >>> with open('/etc/passwd') as f:
    ... for line in f:
    ...     print(line, end='')
    ...
    ##
    # User Database
    #
    # Note that this file is consulted directly only when the system is running
    # in single-user mode. At other times, this information is provided by
    # Open Directory.
    ...
    ##
    nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
    root:*:0:0:System Administrator:/var/root:/bin/sh
    ...
    >>>

如果你想跳过开始部分的注释行的话，可以这样做：

.. code-block:: python

    >>> from itertools import dropwhile
    >>> with open('/etc/passwd') as f:
    ...     for line in dropwhile(lambda line: line.startswith('#'), f):
    ...         print(line, end='')
    ...
    nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
    root:*:0:0:System Administrator:/var/root:/bin/sh
    ...
    >>>

这个例子是基于根据某个测试函数跳过开始的元素。
如果你已经明确知道了要跳过的元素的个数的话，那么可以使用 ``itertools.islice()`` 来代替。比如：

.. code-block:: python

    >>> from itertools import islice
    >>> items = ['a', 'b', 'c', 1, 4, 10, 15]
    >>> for x in islice(items, 3, None):
    ...     print(x)
    ...
    1
    4
    10
    15
    >>>

在这个例子中， ``islice()`` 函数最后那个 ``None`` 参数指定了你要获取从第3个到最后的所有元素，
如果 ``None`` 和3的位置对调，意思就是仅仅获取前三个元素恰恰相反，
(这个跟切片的相反操作 ``[3:]`` 和 ``[:3]`` 原理是一样的)。

----------
讨论
----------
函数 ``dropwhile()`` 和 ``islice()`` 其实就是两个帮助函数，为的就是避免写出下面这种冗余代码：

.. code-block:: python

    with open('/etc/passwd') as f:
        # Skip over initial comments
        while True:
            line = next(f, '')
            if not line.startswith('#'):
                break

        # Process remaining lines
        while line:
            # Replace with useful processing
            print(line, end='')
            line = next(f, None)

跳过一个可迭代对象的开始部分跟通常的过滤是不同的。
比如，上述代码的第一个部分可能会这样重写：

.. code-block:: python

    with open('/etc/passwd') as f:
        lines = (line for line in f if not line.startswith('#'))
        for line in lines:
            print(line, end='')

这样写确实可以跳过开始部分的注释行，但是同样也会跳过文件中其他所有的注释行。
换句话讲，我们的解决方案是仅仅跳过开始部分满足测试条件的行，在那以后，所有的元素不再进行测试和过滤了。

最后需要着重强调的一点是，本节的方案适用于所有可迭代对象，包括那些事先不能确定大小的，
比如生成器，文件及其类似的对象。
============================
4.9 排列组合的迭代
============================

----------
问题
----------
你想迭代遍历一个集合中元素的所有可能的排列或组合

----------
解决方案
----------
itertools模块提供了三个函数来解决这类问题。
其中一个是 ``itertools.permutations()`` ，
它接受一个集合并产生一个元组序列，每个元组由集合中所有元素的一个可能排列组成。
也就是说通过打乱集合中元素排列顺序生成一个元组，比如：

.. code-block:: python

    >>> items = ['a', 'b', 'c']
    >>> from itertools import permutations
    >>> for p in permutations(items):
    ...     print(p)
    ...
    ('a', 'b', 'c')
    ('a', 'c', 'b')
    ('b', 'a', 'c')
    ('b', 'c', 'a')
    ('c', 'a', 'b')
    ('c', 'b', 'a')
    >>>

如果你想得到指定长度的所有排列，你可以传递一个可选的长度参数。就像这样：

.. code-block:: python

    >>> for p in permutations(items, 2):
    ...     print(p)
    ...
    ('a', 'b')
    ('a', 'c')
    ('b', 'a')
    ('b', 'c')
    ('c', 'a')
    ('c', 'b')
    >>>

使用 ``itertools.combinations()`` 可得到输入集合中元素的所有的组合。比如：

.. code-block:: python

    >>> from itertools import combinations
    >>> for c in combinations(items, 3):
    ...     print(c)
    ...
    ('a', 'b', 'c')

    >>> for c in combinations(items, 2):
    ...     print(c)
    ...
    ('a', 'b')
    ('a', 'c')
    ('b', 'c')

    >>> for c in combinations(items, 1):
    ...     print(c)
    ...
    ('a',)
    ('b',)
    ('c',)
    >>>

对于 ``combinations()`` 来讲，元素的顺序已经不重要了。
也就是说，组合 ``('a', 'b')`` 跟 ``('b', 'a')`` 其实是一样的(最终只会输出其中一个)。

在计算组合的时候，一旦元素被选取就会从候选中剔除掉(比如如果元素'a'已经被选取了，那么接下来就不会再考虑它了)。
而函数 ``itertools.combinations_with_replacement()`` 允许同一个元素被选择多次，比如：

.. code-block:: python

    >>> for c in combinations_with_replacement(items, 3):
    ...     print(c)
    ...
    ('a', 'a', 'a')
    ('a', 'a', 'b')
    ('a', 'a', 'c')
    ('a', 'b', 'b')
    ('a', 'b', 'c')
    ('a', 'c', 'c')
    ('b', 'b', 'b')
    ('b', 'b', 'c')
    ('b', 'c', 'c')
    ('c', 'c', 'c')
    >>>

----------
讨论
----------
这一小节我们向你展示的仅仅是 ``itertools`` 模块的一部分功能。
尽管你也可以自己手动实现排列组合算法，但是这样做得要花点脑力。
当我们碰到看上去有些复杂的迭代问题时，最好可以先去看看itertools模块。
如果这个问题很普遍，那么很有可能会在里面找到解决方案！

========================
4.10 序列上索引值迭代
========================

----------
问题
----------
你想在迭代一个序列的同时跟踪正在被处理的元素索引。

----------
解决方案
----------
内置的 ``enumerate()`` 函数可以很好的解决这个问题：

.. code-block:: python

    >>> my_list = ['a', 'b', 'c']
    >>> for idx, val in enumerate(my_list):
    ...     print(idx, val)
    ...
    0 a
    1 b
    2 c

为了按传统行号输出(行号从1开始)，你可以传递一个开始参数：

.. code-block:: python

    >>> my_list = ['a', 'b', 'c']
    >>> for idx, val in enumerate(my_list, 1):
    ...     print(idx, val)
    ...
    1 a
    2 b
    3 c

这种情况在你遍历文件时想在错误消息中使用行号定位时候非常有用：

.. code-block:: python

    def parse_data(filename):
        with open(filename, 'rt') as f:
            for lineno, line in enumerate(f, 1):
                fields = line.split()
                try:
                    count = int(fields[1])
                    ...
                except ValueError as e:
                    print('Line {}: Parse error: {}'.format(lineno, e))

``enumerate()`` 对于跟踪某些值在列表中出现的位置是很有用的。
所以，如果你想将一个文件中出现的单词映射到它出现的行号上去，可以很容易的利用 ``enumerate()`` 来完成：

.. code-block:: python

    word_summary = defaultdict(list)

    with open('myfile.txt', 'r') as f:
        lines = f.readlines()

    for idx, line in enumerate(lines):
        # Create a list of words in current line
        words = [w.strip().lower() for w in line.split()]
        for word in words:
            word_summary[word].append(idx)

如果你处理完文件后打印 ``word_summary`` ，会发现它是一个字典(准确来讲是一个 ``defaultdict`` )，
对于每个单词有一个 ``key`` ，每个 ``key`` 对应的值是一个由这个单词出现的行号组成的列表。
如果某个单词在一行中出现过两次，那么这个行号也会出现两次，
同时也可以作为文本的一个简单统计。

----------
讨论
----------
当你想额外定义一个计数变量的时候，使用 ``enumerate()`` 函数会更加简单。你可能会像下面这样写代码：

.. code-block:: python

    lineno = 1
    for line in f:
        # Process line
        ...
        lineno += 1

但是如果使用 ``enumerate()`` 函数来代替就显得更加优雅了：

.. code-block:: python

    for lineno, line in enumerate(f):
        # Process line
        ...

``enumerate()`` 函数返回的是一个 ``enumerate`` 对象实例，
它是一个迭代器，返回连续的包含一个计数和一个值的元组，
元组中的值通过在传入序列上调用 ``next()`` 返回。

还有一点可能并不很重要，但是也值得注意，
有时候当你在一个已经解压后的元组序列上使用 ``enumerate()`` 函数时很容易调入陷阱。
你得像下面正确的方式这样写：

.. code-block:: python

    data = [ (1, 2), (3, 4), (5, 6), (7, 8) ]

    # Correct!
    for n, (x, y) in enumerate(data):
        ...
    # Error!
    for n, x, y in enumerate(data):
        ...


============================
4.11 同时迭代多个序列
============================

----------
问题
----------
你想同时迭代多个序列，每次分别从一个序列中取一个元素。

----------
解决方案
----------
为了同时迭代多个序列，使用 ``zip()`` 函数。比如：

.. code-block:: python

    >>> xpts = [1, 5, 4, 2, 10, 7]
    >>> ypts = [101, 78, 37, 15, 62, 99]
    >>> for x, y in zip(xpts, ypts):
    ...     print(x,y)
    ...
    1 101
    5 78
    4 37
    2 15
    10 62
    7 99
    >>>

``zip(a, b)`` 会生成一个可返回元组 ``(x, y)`` 的迭代器，其中x来自a，y来自b。
一旦其中某个序列到底结尾，迭代宣告结束。
因此迭代长度跟参数中最短序列长度一致。

.. code-block:: python

    >>> a = [1, 2, 3]
    >>> b = ['w', 'x', 'y', 'z']
    >>> for i in zip(a,b):
    ...     print(i)
    ...
    (1, 'w')
    (2, 'x')
    (3, 'y')
    >>>

如果这个不是你想要的效果，那么还可以使用 ``itertools.zip_longest()`` 函数来代替。比如：

.. code-block:: python

    >>> from itertools import zip_longest
    >>> for i in zip_longest(a,b):
    ...     print(i)
    ...
    (1, 'w')
    (2, 'x')
    (3, 'y')
    (None, 'z')

    >>> for i in zip_longest(a, b, fillvalue=0):
    ...     print(i)
    ...
    (1, 'w')
    (2, 'x')
    (3, 'y')
    (0, 'z')
    >>>

----------
讨论
----------
当你想成对处理数据的时候 ``zip()`` 函数是很有用的。
比如，假设你头列表和一个值列表，就像下面这样：

.. code-block:: python

    headers = ['name', 'shares', 'price']
    values = ['ACME', 100, 490.1]

使用zip()可以让你将它们打包并生成一个字典：

.. code-block:: python

    s = dict(zip(headers,values))

或者你也可以像下面这样产生输出：

.. code-block:: python

    for name, val in zip(headers, values):
        print(name, '=', val)

虽然不常见，但是 ``zip()`` 可以接受多于两个的序列的参数。
这时候所生成的结果元组中元素个数跟输入序列个数一样。比如;

.. code-block:: python

    >>> a = [1, 2, 3]
    >>> b = [10, 11, 12]
    >>> c = ['x','y','z']
    >>> for i in zip(a, b, c):
    ...     print(i)
    ...
    (1, 10, 'x')
    (2, 11, 'y')
    (3, 12, 'z')
    >>>

最后强调一点就是， ``zip()`` 会创建一个迭代器来作为结果返回。
如果你需要将结对的值存储在列表中，要使用 ``list()`` 函数。比如：

.. code-block:: python

    >>> zip(a, b)
    <zip object at 0x1007001b8>
    >>> list(zip(a, b))
    [(1, 10), (2, 11), (3, 12)]
    >>>
============================
4.12 不同集合上元素的迭代
============================

----------
问题
----------
你想在多个对象执行相同的操作，但是这些对象在不同的容器中，你希望代码在不失可读性的情况下避免写重复的循环。

----------
解决方案
----------
``itertools.chain()`` 方法可以用来简化这个任务。
它接受一个可迭代对象列表作为输入，并返回一个迭代器，有效的屏蔽掉在多个容器中迭代细节。
为了演示清楚，考虑下面这个例子：

.. code-block:: python

    >>> from itertools import chain
    >>> a = [1, 2, 3, 4]
    >>> b = ['x', 'y', 'z']
    >>> for x in chain(a, b):
    ... print(x)
    ...
    1
    2
    3
    4
    x
    y
    z
    >>>

使用 ``chain()`` 的一个常见场景是当你想对不同的集合中所有元素执行某些操作的时候。比如：

.. code-block:: python

    # Various working sets of items
    active_items = set()
    inactive_items = set()

    # Iterate over all items
    for item in chain(active_items, inactive_items):
        # Process item

这种解决方案要比像下面这样使用两个单独的循环更加优雅，

.. code-block:: python

    for item in active_items:
        # Process item
        ...

    for item in inactive_items:
        # Process item
        ...

----------
讨论
----------
``itertools.chain()`` 接受一个或多个可迭代对象作为输入参数。
然后创建一个迭代器，依次连续的返回每个可迭代对象中的元素。
这种方式要比先将序列合并再迭代要高效的多。比如：

.. code-block:: python

    # Inefficent
    for x in a + b:
        ...

    # Better
    for x in chain(a, b):
        ...

第一种方案中， ``a + b`` 操作会创建一个全新的序列并要求a和b的类型一致。
``chian()`` 不会有这一步，所以如果输入序列非常大的时候会很省内存。
并且当可迭代对象类型不一样的时候 ``chain()`` 同样可以很好的工作。

============================
4.13 创建数据处理管道
============================

----------
问题
----------
你想以数据管道(类似Unix管道)的方式迭代处理数据。
比如，你有个大量的数据需要处理，但是不能将它们一次性放入内存中。

----------
解决方案
----------
生成器函数是一个实现管道机制的好办法。
为了演示，假定你要处理一个非常大的日志文件目录：

.. code-block:: python

    foo/
        access-log-012007.gz
        access-log-022007.gz
        access-log-032007.gz
        ...
        access-log-012008
    bar/
        access-log-092007.bz2
        ...
        access-log-022008

假设每个日志文件包含这样的数据：

.. code-block:: python

    124.115.6.12 - - [10/Jul/2012:00:18:50 -0500] "GET /robots.txt ..." 200 71
    210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /ply/ ..." 200 11875
    210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] "GET /favicon.ico ..." 404 369
    61.135.216.105 - - [10/Jul/2012:00:20:04 -0500] "GET /blog/atom.xml ..." 304 -
    ...

为了处理这些文件，你可以定义一个由多个执行特定任务独立任务的简单生成器函数组成的容器。就像这样：

.. code-block:: python

    import os
    import fnmatch
    import gzip
    import bz2
    import re

    def gen_find(filepat, top):
        '''
        Find all filenames in a directory tree that match a shell wildcard pattern
        '''
        for path, dirlist, filelist in os.walk(top):
            for name in fnmatch.filter(filelist, filepat):
                yield os.path.join(path,name)

    def gen_opener(filenames):
        '''
        Open a sequence of filenames one at a time producing a file object.
        The file is closed immediately when proceeding to the next iteration.
        '''
        for filename in filenames:
            if filename.endswith('.gz'):
                f = gzip.open(filename, 'rt')
            elif filename.endswith('.bz2'):
                f = bz2.open(filename, 'rt')
            else:
                f = open(filename, 'rt')
            yield f
            f.close()

    def gen_concatenate(iterators):
        '''
        Chain a sequence of iterators together into a single sequence.
        '''
        for it in iterators:
            yield from it

    def gen_grep(pattern, lines):
        '''
        Look for a regex pattern in a sequence of lines
        '''
        pat = re.compile(pattern)
        for line in lines:
            if pat.search(line):
                yield line

现在你可以很容易的将这些函数连起来创建一个处理管道。
比如，为了查找包含单词python的所有日志行，你可以这样做：

.. code-block:: python

    lognames = gen_find('access-log*', 'www')
    files = gen_opener(lognames)
    lines = gen_concatenate(files)
    pylines = gen_grep('(?i)python', lines)
    for line in pylines:
        print(line)

如果将来的时候你想扩展管道，你甚至可以在生成器表达式中包装数据。
比如，下面这个版本计算出传输的字节数并计算其总和。

.. code-block:: python

    lognames = gen_find('access-log*', 'www')
    files = gen_opener(lognames)
    lines = gen_concatenate(files)
    pylines = gen_grep('(?i)python', lines)
    bytecolumn = (line.rsplit(None,1)[1] for line in pylines)
    bytes = (int(x) for x in bytecolumn if x != '-')
    print('Total', sum(bytes))

----------
讨论
----------
以管道方式处理数据可以用来解决各类其他问题，包括解析，读取实时数据，定时轮询等。

为了理解上述代码，重点是要明白 ``yield`` 语句作为数据的生产者而 ``for`` 循环语句作为数据的消费者。
当这些生成器被连在一起后，每个 ``yield`` 会将一个单独的数据元素传递给迭代处理管道的下一阶段。
在例子最后部分， ``sum()`` 函数是最终的程序驱动者，每次从生成器管道中提取出一个元素。

这种方式一个非常好的特点是每个生成器函数很小并且都是独立的。这样的话就很容易编写和维护它们了。
很多时候，这些函数如果比较通用的话可以在其他场景重复使用。
并且最终将这些组件组合起来的代码看上去非常简单，也很容易理解。

使用这种方式的内存效率也不得不提。上述代码即便是在一个超大型文件目录中也能工作的很好。
事实上，由于使用了迭代方式处理，代码运行过程中只需要很小很小的内存。

在调用 ``gen_concatenate()`` 函数的时候你可能会有些不太明白。
这个函数的目的是将输入序列拼接成一个很长的行序列。
``itertools.chain()`` 函数同样有类似的功能，但是它需要将所有可迭代对象最为参数传入。
在上面这个例子中，你可能会写类似这样的语句 ``lines = itertools.chain(*files)`` ，
这将导致 ``gen_opener()`` 生成器被提前全部消费掉。
但由于 ``gen_opener()`` 生成器每次生成一个打开过的文件，
等到下一个迭代步骤时文件就关闭了，因此 ``chain()`` 在这里不能这样使用。
上面的方案可以避免这种情况。

``gen_concatenate()`` 函数中出现过 ``yield from`` 语句，它将 ``yield`` 操作代理到父生成器上去。
语句 ``yield from it`` 简单的返回生成器 ``it`` 所产生的所有值。
关于这个我们在4.14小节会有更进一步的描述。

最后还有一点需要注意的是，管道方式并不是万能的。
有时候你想立即处理所有数据。
然而，即便是这种情况，使用生成器管道也可以将这类问题从逻辑上变为工作流的处理方式。

*David Beazley* 在他的
`Generator Tricks for Systems Programmers <http://www.dabeaz.com/generators/>`_
教程中对于这种技术有非常深入的讲解。可以参考这个教程获取更多的信息。

============================
4.14 展开嵌套的序列
============================

----------
问题
----------
你想将一个多层嵌套的序列展开成一个单层列表

----------
解决方案
----------
可以写一个包含 ``yield from`` 语句的递归生成器来轻松解决这个问题。比如：

.. code-block:: python

    from collections import Iterable

    def flatten(items, ignore_types=(str, bytes)):
        for x in items:
            if isinstance(x, Iterable) and not isinstance(x, ignore_types):
                yield from flatten(x)
            else:
                yield x

    items = [1, 2, [3, 4, [5, 6], 7], 8]
    # Produces 1 2 3 4 5 6 7 8
    for x in flatten(items):
        print(x)

在上面代码中， ``isinstance(x, Iterable)`` 检查某个元素是否是可迭代的。
如果是的话， ``yield from`` 就会返回所有子例程的值。最终返回结果就是一个没有嵌套的简单序列了。

额外的参数 ``ignore_types`` 和检测语句 ``isinstance(x, ignore_types)``
用来将字符串和字节排除在可迭代对象外，防止将它们再展开成单个的字符。
这样的话字符串数组就能最终返回我们所期望的结果了。比如：

.. code-block:: python

    >>> items = ['Dave', 'Paula', ['Thomas', 'Lewis']]
    >>> for x in flatten(items):
    ...     print(x)
    ...
    Dave
    Paula
    Thomas
    Lewis
    >>>

----------
讨论
----------
语句 ``yield from`` 在你想在生成器中调用其他生成器作为子例程的时候非常有用。
如果你不使用它的话，那么就必须写额外的 ``for`` 循环了。比如：

.. code-block:: python

    def flatten(items, ignore_types=(str, bytes)):
        for x in items:
            if isinstance(x, Iterable) and not isinstance(x, ignore_types):
                for i in flatten(x):
                    yield i
            else:
                yield x

尽管只改了一点点，但是 ``yield from`` 语句看上去感觉更好，并且也使得代码更简洁清爽。

之前提到的对于字符串和字节的额外检查是为了防止将它们再展开成单个字符。
如果还有其他你不想展开的类型，修改参数 ``ignore_types`` 即可。

最后要注意的一点是， ``yield from`` 在涉及到基于协程和生成器的并发编程中扮演着更加重要的角色。
可以参考12.12小节查看另外一个例子。

==============================
4.15 顺序迭代合并后的排序迭代对象
==============================

----------
问题
----------
你有一系列排序序列，想将它们合并后得到一个排序序列并在上面迭代遍历。

----------
解决方案
----------
``heapq.merge()`` 函数可以帮你解决这个问题。比如：

.. code-block:: python

    >>> import heapq
    >>> a = [1, 4, 7, 10]
    >>> b = [2, 5, 6, 11]
    >>> for c in heapq.merge(a, b):
    ...     print(c)
    ...
    1
    2
    4
    5
    6
    7
    10
    11


----------
讨论
----------
``heapq.merge`` 可迭代特性意味着它不会立马读取所有序列。
这就意味着你可以在非常长的序列中使用它，而不会有太大的开销。
比如，下面是一个例子来演示如何合并两个排序文件：

.. code-block:: python

    with open('sorted_file_1', 'rt') as file1, \
        open('sorted_file_2', 'rt') as file2, \
        open('merged_file', 'wt') as outf:

        for line in heapq.merge(file1, file2):
            outf.write(line)

有一点要强调的是 ``heapq.merge()`` 需要所有输入序列必须是排过序的。
特别的，它并不会预先读取所有数据到堆栈中或者预先排序，也不会对输入做任何的排序检测。
它仅仅是检查所有序列的开始部分并返回最小的那个，这个过程一直会持续直到所有输入序列中的元素都被遍历完。
==========================
4.16 迭代器代替while无限循环
==========================

----------
问题
----------
你在代码中使用 ``while`` 循环来迭代处理数据，因为它需要调用某个函数或者和一般迭代模式不同的测试条件。
能不能用迭代器来重写这个循环呢？

----------
解决方案
----------
一个常见的IO操作程序可能会想下面这样：

.. code-block:: python

    CHUNKSIZE = 8192

    def reader(s):
        while True:
            data = s.recv(CHUNKSIZE)
            if data == b'':
                break
            process_data(data)

这种代码通常可以使用 ``iter()`` 来代替，如下所示：

.. code-block:: python

    def reader2(s):
        for chunk in iter(lambda: s.recv(CHUNKSIZE), b''):
            pass
            # process_data(data)


如果你怀疑它到底能不能正常工作，可以试验下一个简单的例子。比如：

.. code-block:: python

    >>> import sys
    >>> f = open('/etc/passwd')
    >>> for chunk in iter(lambda: f.read(10), ''):
    ...     n = sys.stdout.write(chunk)
    ...
    nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
    root:*:0:0:System Administrator:/var/root:/bin/sh
    daemon:*:1:1:System Services:/var/root:/usr/bin/false
    _uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
    ...
    >>>

----------
讨论
----------
``iter`` 函数一个鲜为人知的特性是它接受一个可选的 ``callable`` 对象和一个标记(结尾)值作为输入参数。
当以这种方式使用的时候，它会创建一个迭代器， 这个迭代器会不断调用 ``callable`` 对象直到返回值和标记值相等为止。

这种特殊的方法对于一些特定的会被重复调用的函数很有效果，比如涉及到I/O调用的函数。
举例来讲，如果你想从套接字或文件中以数据块的方式读取数据，通常你得要不断重复的执行 ``read()`` 或 ``recv()`` ，
并在后面紧跟一个文件结尾测试来决定是否终止。这节中的方案使用一个简单的 ``iter()`` 调用就可以将两者结合起来了。
其中 ``lambda`` 函数参数是为了创建一个无参的 ``callable`` 对象，并为 ``recv`` 或 ``read()`` 方法提供了 ``size`` 参数。
============================
5.1 读写文本数据
============================

----------
问题
----------
你需要读写各种不同编码的文本数据，比如ASCII，UTF-8或UTF-16编码等。

----------
解决方案
----------
使用带有 ``rt`` 模式的 ``open()`` 函数读取文本文件。如下所示：

.. code-block:: python

    # Read the entire file as a single string
    with open('somefile.txt', 'rt') as f:
        data = f.read()

    # Iterate over the lines of the file
    with open('somefile.txt', 'rt') as f:
        for line in f:
            # process line
            ...

类似的，为了写入一个文本文件，使用带有 ``wt`` 模式的 ``open()`` 函数，
如果之前文件内容存在则清除并覆盖掉。如下所示：

.. code-block:: python

    # Write chunks of text data
    with open('somefile.txt', 'wt') as f:
        f.write(text1)
        f.write(text2)
        ...

    # Redirected print statement
    with open('somefile.txt', 'wt') as f:
        print(line1, file=f)
        print(line2, file=f)
        ...

如果是在已存在文件中添加内容，使用模式为 ``at`` 的 ``open()`` 函数。

文件的读写操作默认使用系统编码，可以通过调用 ``sys.getdefaultencoding()`` 来得到。
在大多数机器上面都是utf-8编码。如果你已经知道你要读写的文本是其他编码方式，
那么可以通过传递一个可选的 ``encoding`` 参数给open()函数。如下所示：

.. code-block:: python

    with open('somefile.txt', 'rt', encoding='latin-1') as f:
        ...

Python支持非常多的文本编码。几个常见的编码是ascii, latin-1, utf-8和utf-16。
在web应用程序中通常都使用的是UTF-8。
ascii对应从U+0000到U+007F范围内的7位字符。
latin-1是字节0-255到U+0000至U+00FF范围内Unicode字符的直接映射。
当读取一个未知编码的文本时使用latin-1编码永远不会产生解码错误。
使用latin-1编码读取一个文件的时候也许不能产生完全正确的文本解码数据，
但是它也能从中提取出足够多的有用数据。同时，如果你之后将数据回写回去，原先的数据还是会保留的。

----------
讨论
----------
读写文本文件一般来讲是比较简单的。但是也几点是需要注意的。
首先，在例子程序中的with语句给被使用到的文件创建了一个上下文环境，
但 ``with`` 控制块结束时，文件会自动关闭。你也可以不使用 ``with`` 语句，但是这时候你就必须记得手动关闭文件：

.. code-block:: python

    f = open('somefile.txt', 'rt')
    data = f.read()
    f.close()

另外一个问题是关于换行符的识别问题，在Unix和Windows中是不一样的(分别是 ``\n`` 和 ``\r\n`` )。
默认情况下，Python会以统一模式处理换行符。
这种模式下，在读取文本的时候，Python可以识别所有的普通换行符并将其转换为单个 ``\n`` 字符。
类似的，在输出时会将换行符 ``\n`` 转换为系统默认的换行符。
如果你不希望这种默认的处理方式，可以给 ``open()`` 函数传入参数 ``newline=''`` ，就像下面这样：

.. code-block:: python

    # Read with disabled newline translation
    with open('somefile.txt', 'rt', newline='') as f:
        ...

为了说明两者之间的差异，下面我在Unix机器上面读取一个Windows上面的文本文件，里面的内容是 ``hello world!\r\n`` ：

.. code-block:: python

    >>> # Newline translation enabled (the default)
    >>> f = open('hello.txt', 'rt')
    >>> f.read()
    'hello world!\n'

    >>> # Newline translation disabled
    >>> g = open('hello.txt', 'rt', newline='')
    >>> g.read()
    'hello world!\r\n'
    >>>

最后一个问题就是文本文件中可能出现的编码错误。
但你读取或者写入一个文本文件时，你可能会遇到一个编码或者解码错误。比如：

.. code-block:: python

    >>> f = open('sample.txt', 'rt', encoding='ascii')
    >>> f.read()
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "/usr/local/lib/python3.3/encodings/ascii.py", line 26, in decode
            return codecs.ascii_decode(input, self.errors)[0]
    UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position
    12: ordinal not in range(128)
    >>>

如果出现这个错误，通常表示你读取文本时指定的编码不正确。
你最好仔细阅读说明并确认你的文件编码是正确的(比如使用UTF-8而不是Latin-1编码或其他)。
如果编码错误还是存在的话，你可以给 ``open()`` 函数传递一个可选的 ``errors`` 参数来处理这些错误。
下面是一些处理常见错误的方法：

.. code-block:: python

    >>> # Replace bad chars with Unicode U+fffd replacement char
    >>> f = open('sample.txt', 'rt', encoding='ascii', errors='replace')
    >>> f.read()
    'Spicy Jalape?o!'
    >>> # Ignore bad chars entirely
    >>> g = open('sample.txt', 'rt', encoding='ascii', errors='ignore')
    >>> g.read()
    'Spicy Jalapeo!'
    >>>

如果你经常使用 ``errors`` 参数来处理编码错误，可能会让你的生活变得很糟糕。
对于文本处理的首要原则是确保你总是使用的是正确编码。当模棱两可的时候，就使用默认的设置(通常都是UTF-8)。

============================
5.2 打印输出至文件中
============================

----------
问题
----------
你想将 ``print()`` 函数的输出重定向到一个文件中去。

----------
解决方案
----------
在 ``print()`` 函数中指定 ``file`` 关键字参数，像下面这样：

.. code-block:: python

    with open('d:/work/test.txt', 'wt') as f:
        print('Hello World!', file=f)

----------
讨论
----------
关于输出重定向到文件中就这些了。但是有一点要注意的就是文件必须是以文本模式打开。
如果文件是二进制模式的话，打印就会出错。

==============================
5.3 使用其他分隔符或行终止符打印
==============================

----------
问题
----------
你想使用 ``print()`` 函数输出数据，但是想改变默认的分隔符或者行尾符。

----------
解决方案
----------
可以使用在 ``print()`` 函数中使用 ``sep`` 和 ``end`` 关键字参数，以你想要的方式输出。比如：

.. code-block:: python

    >>> print('ACME', 50, 91.5)
    ACME 50 91.5
    >>> print('ACME', 50, 91.5, sep=',')
    ACME,50,91.5
    >>> print('ACME', 50, 91.5, sep=',', end='!!\n')
    ACME,50,91.5!!
    >>>

使用 ``end`` 参数也可以在输出中禁止换行。比如：

.. code-block:: python

    >>> for i in range(5):
    ...     print(i)
    ...
    0
    1
    2
    3
    4
    >>> for i in range(5):
    ...     print(i, end=' ')
    ...
    0 1 2 3 4 >>>

----------
讨论
----------
当你想使用非空格分隔符来输出数据的时候，给 ``print()`` 函数传递一个 ``sep`` 参数是最简单的方案。
有时候你会看到一些程序员会使用 ``str.join()`` 来完成同样的事情。比如：

.. code-block:: python

    >>> print(','.join(('ACME','50','91.5')))
    ACME,50,91.5
    >>>

``str.join()`` 的问题在于它仅仅适用于字符串。这意味着你通常需要执行另外一些转换才能让它正常工作。比如：

.. code-block:: python

    >>> row = ('ACME', 50, 91.5)
    >>> print(','.join(row))
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: sequence item 1: expected str instance, int found
    >>> print(','.join(str(x) for x in row))
    ACME,50,91.5
    >>>

你当然可以不用那么麻烦，只需要像下面这样写：

.. code-block:: python

    >>> print(*row, sep=',')
    ACME,50,91.5
    >>>

==============================
5.4 读写字节数据
==============================

----------
问题
----------
你想读写二进制文件，比如图片，声音文件等等。

----------
解决方案
----------
使用模式为 ``rb`` 或 ``wb`` 的 ``open()`` 函数来读取或写入二进制数据。比如：

.. code-block:: python

    # Read the entire file as a single byte string
    with open('somefile.bin', 'rb') as f:
        data = f.read()

    # Write binary data to a file
    with open('somefile.bin', 'wb') as f:
        f.write(b'Hello World')

在读取二进制数据时，需要指明的是所有返回的数据都是字节字符串格式的，而不是文本字符串。
类似的，在写入的时候，必须保证参数是以字节形式对外暴露数据的对象(比如字节字符串，字节数组对象等)。

----------
讨论
----------
在读取二进制数据的时候，字节字符串和文本字符串的语义差异可能会导致一个潜在的陷阱。
特别需要注意的是，索引和迭代动作返回的是字节的值而不是字节字符串。比如：

.. code-block:: python

    >>> # Text string
    >>> t = 'Hello World'
    >>> t[0]
    'H'
    >>> for c in t:
    ...     print(c)
    ...
    H
    e
    l
    l
    o
    ...
    >>> # Byte string
    >>> b = b'Hello World'
    >>> b[0]
    72
    >>> for c in b:
    ...     print(c)
    ...
    72
    101
    108
    108
    111
    ...
    >>>

如果你想从二进制模式的文件中读取或写入文本数据，必须确保要进行解码和编码操作。比如：

.. code-block:: python

    with open('somefile.bin', 'rb') as f:
        data = f.read(16)
        text = data.decode('utf-8')

    with open('somefile.bin', 'wb') as f:
        text = 'Hello World'
        f.write(text.encode('utf-8'))

二进制I/O还有一个鲜为人知的特性就是数组和C结构体类型能直接被写入，而不需要中间转换为自己对象。比如：

.. code-block:: python

    import array
    nums = array.array('i', [1, 2, 3, 4])
    with open('data.bin','wb') as f:
        f.write(nums)

这个适用于任何实现了被称之为"缓冲接口"的对象，这种对象会直接暴露其底层的内存缓冲区给能处理它的操作。
二进制数据的写入就是这类操作之一。

很多对象还允许通过使用文件对象的 ``readinto()`` 方法直接读取二进制数据到其底层的内存中去。比如：

.. code-block:: python

    >>> import array
    >>> a = array.array('i', [0, 0, 0, 0, 0, 0, 0, 0])
    >>> with open('data.bin', 'rb') as f:
    ...     f.readinto(a)
    ...
    16
    >>> a
    array('i', [1, 2, 3, 4, 0, 0, 0, 0])
    >>>

但是使用这种技术的时候需要格外小心，因为它通常具有平台相关性，并且可能会依赖字长和字节顺序(高位优先和低位优先)。
可以查看5.9小节中另外一个读取二进制数据到可修改缓冲区的例子。

==========================
5.5 文件不存在才能写入
==========================

----------
问题
----------
你想像一个文件中写入数据，但是前提必须是这个文件在文件系统上不存在。
也就是不允许覆盖已存在的文件内容。

----------
解决方案
----------
可以在 ``open()`` 函数中使用 ``x`` 模式来代替 ``w`` 模式的方法来解决这个问题。比如：

.. code-block:: python

    >>> with open('somefile', 'wt') as f:
    ...     f.write('Hello\n')
    ...
    >>> with open('somefile', 'xt') as f:
    ...     f.write('Hello\n')
    ...
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    FileExistsError: [Errno 17] File exists: 'somefile'
    >>>

如果文件是二进制的，使用 ``xb`` 来代替 ``xt``

----------
讨论
----------
这一小节演示了在写文件时通常会遇到的一个问题的完美解决方案(不小心覆盖一个已存在的文件)。
一个替代方案是先测试这个文件是否存在，像下面这样：

.. code-block:: python

    >>> import os
    >>> if not os.path.exists('somefile'):
    ...     with open('somefile', 'wt') as f:
    ...         f.write('Hello\n')
    ... else:
    ...     print('File already exists!')
    ...
    File already exists!
    >>>

显而易见，使用x文件模式更加简单。要注意的是x模式是一个Python3对 ``open()`` 函数特有的扩展。
在Python的旧版本或者是Python实现的底层C函数库中都是没有这个模式的。
==============================
5.6 字符串的I/O操作
==============================

----------
问题
----------
你想使用操作类文件对象的程序来操作文本或二进制字符串。

----------
解决方案
----------
使用 ``io.StringIO()`` 和 ``io.BytesIO()`` 类来创建类文件对象操作字符串数据。比如：

.. code-block:: python

    >>> s = io.StringIO()
    >>> s.write('Hello World\n')
    12
    >>> print('This is a test', file=s)
    15
    >>> # Get all of the data written so far
    >>> s.getvalue()
    'Hello World\nThis is a test\n'
    >>>

    >>> # Wrap a file interface around an existing string
    >>> s = io.StringIO('Hello\nWorld\n')
    >>> s.read(4)
    'Hell'
    >>> s.read()
    'o\nWorld\n'
    >>>

``io.StringIO`` 只能用于文本。如果你要操作二进制数据，要使用 ``io.BytesIO`` 类来代替。比如：

.. code-block:: python

    >>> s = io.BytesIO()
    >>> s.write(b'binary data')
    >>> s.getvalue()
    b'binary data'
    >>>

----------
讨论
----------
当你想模拟一个普通的文件的时候 ``StringIO`` 和 ``BytesIO`` 类是很有用的。
比如，在单元测试中，你可以使用 ``StringIO`` 来创建一个包含测试数据的类文件对象，
这个对象可以被传给某个参数为普通文件对象的函数。

需要注意的是， ``StringIO`` 和 ``BytesIO`` 实例并没有正确的整数类型的文件描述符。
因此，它们不能在那些需要使用真实的系统级文件如文件，管道或者是套接字的程序中使用。

=========================
5.7 读写压缩文件
=========================

----------
问题
----------
你想读写一个gzip或bz2格式的压缩文件。

----------
解决方案
----------
``gzip`` 和 ``bz2`` 模块可以很容易的处理这些文件。
两个模块都为 ``open()`` 函数提供了另外的实现来解决这个问题。
比如，为了以文本形式读取压缩文件，可以这样做：

.. code-block:: python

    # gzip compression
    import gzip
    with gzip.open('somefile.gz', 'rt') as f:
        text = f.read()

    # bz2 compression
    import bz2
    with bz2.open('somefile.bz2', 'rt') as f:
        text = f.read()

类似的，为了写入压缩数据，可以这样做：

.. code-block:: python

    # gzip compression
    import gzip
    with gzip.open('somefile.gz', 'wt') as f:
        f.write(text)

    # bz2 compression
    import bz2
    with bz2.open('somefile.bz2', 'wt') as f:
        f.write(text)

如上，所有的I/O操作都使用文本模式并执行Unicode的编码/解码。
类似的，如果你想操作二进制数据，使用 ``rb`` 或者 ``wb`` 文件模式即可。

----------
讨论
----------
大部分情况下读写压缩数据都是很简单的。但是要注意的是选择一个正确的文件模式是非常重要的。
如果你不指定模式，那么默认的就是二进制模式，如果这时候程序想要接受的是文本数据，那么就会出错。
``gzip.open()`` 和 ``bz2.open()`` 接受跟内置的 ``open()`` 函数一样的参数，
包括 ``encoding``，``errors``，``newline`` 等等。

当写入压缩数据时，可以使用 ``compresslevel`` 这个可选的关键字参数来指定一个压缩级别。比如：

.. code-block:: python

    with gzip.open('somefile.gz', 'wt', compresslevel=5) as f:
        f.write(text)

默认的等级是9，也是最高的压缩等级。等级越低性能越好，但是数据压缩程度也越低。

最后一点， ``gzip.open()`` 和 ``bz2.open()`` 还有一个很少被知道的特性，
它们可以作用在一个已存在并以二进制模式打开的文件上。比如，下面代码是可行的：

.. code-block:: python

    import gzip
    f = open('somefile.gz', 'rb')
    with gzip.open(f, 'rt') as g:
        text = g.read()

这样就允许 ``gzip`` 和 ``bz2`` 模块可以工作在许多类文件对象上，比如套接字，管道和内存中文件等。

==============================
5.8 固定大小记录的文件迭代
==============================

----------
问题
----------
你想在一个固定长度记录或者数据块的集合上迭代，而不是在一个文件中一行一行的迭代。

----------
解决方案
----------
通过下面这个小技巧使用 ``iter`` 和 ``functools.partial()`` 函数：

.. code-block:: python

    from functools import partial

    RECORD_SIZE = 32

    with open('somefile.data', 'rb') as f:
        records = iter(partial(f.read, RECORD_SIZE), b'')
        for r in records:
            ...

这个例子中的 ``records`` 对象是一个可迭代对象，它会不断的产生固定大小的数据块，直到文件末尾。
要注意的是如果总记录大小不是块大小的整数倍的话，最后一个返回元素的字节数会比期望值少。

----------
讨论
----------
``iter()`` 函数有一个鲜为人知的特性就是，如果你给它传递一个可调用对象和一个标记值，它会创建一个迭代器。
这个迭代器会一直调用传入的可调用对象直到它返回标记值为止，这时候迭代终止。

在例子中， ``functools.partial`` 用来创建一个每次被调用时从文件中读取固定数目字节的可调用对象。
标记值 ``b''`` 就是当到达文件结尾时的返回值。

最后再提一点，上面的例子中的文件时以二进制模式打开的。
如果是读取固定大小的记录，这通常是最普遍的情况。
而对于文本文件，一行一行的读取(默认的迭代行为)更普遍点。

==============================
5.9 读取二进制数据到可变缓冲区中
==============================

----------
问题
----------
你想直接读取二进制数据到一个可变缓冲区中，而不需要做任何的中间复制操作。
或者你想原地修改数据并将它写回到一个文件中去。

----------
解决方案
----------
为了读取数据到一个可变数组中，使用文件对象的 ``readinto()`` 方法。比如：

.. code-block:: python

    import os.path

    def read_into_buffer(filename):
        buf = bytearray(os.path.getsize(filename))
        with open(filename, 'rb') as f:
            f.readinto(buf)
        return buf

下面是一个演示这个函数使用方法的例子：

.. code-block:: python

    >>> # Write a sample file
    >>> with open('sample.bin', 'wb') as f:
    ...     f.write(b'Hello World')
    ...
    >>> buf = read_into_buffer('sample.bin')
    >>> buf
    bytearray(b'Hello World')
    >>> buf[0:5] = b'Hello'
    >>> buf
    bytearray(b'Hello World')
    >>> with open('newsample.bin', 'wb') as f:
    ...     f.write(buf)
    ...
    11
    >>>

----------
讨论
----------
文件对象的 ``readinto()`` 方法能被用来为预先分配内存的数组填充数据，甚至包括由 ``array`` 模块或 ``numpy`` 库创建的数组。
和普通 ``read()`` 方法不同的是， ``readinto()`` 填充已存在的缓冲区而不是为新对象重新分配内存再返回它们。
因此，你可以使用它来避免大量的内存分配操作。
比如，如果你读取一个由相同大小的记录组成的二进制文件时，你可以像下面这样写：

.. code-block:: python

    record_size = 32 # Size of each record (adjust value)

    buf = bytearray(record_size)
    with open('somefile', 'rb') as f:
        while True:
            n = f.readinto(buf)
            if n < record_size:
                break
            # Use the contents of buf
            ...

另外有一个有趣特性就是 ``memoryview`` ，
它可以通过零复制的方式对已存在的缓冲区执行切片操作，甚至还能修改它的内容。比如：

.. code-block:: python

    >>> buf
    bytearray(b'Hello World')
    >>> m1 = memoryview(buf)
    >>> m2 = m1[-5:]
    >>> m2
    <memory at 0x100681390>
    >>> m2[:] = b'WORLD'
    >>> buf
    bytearray(b'Hello WORLD')
    >>>

使用 ``f.readinto()`` 时需要注意的是，你必须检查它的返回值，也就是实际读取的字节数。

如果字节数小于缓冲区大小，表明数据被截断或者被破坏了(比如你期望每次读取指定数量的字节)。

最后，留心观察其他函数库和模块中和 ``into`` 相关的函数(比如 ``recv_into()`` ， ``pack_into()`` 等)。
Python的很多其他部分已经能支持直接的I/O或数据访问操作，这些操作可被用来填充或修改数组和缓冲区内容。

关于解析二进制结构和 ``memoryviews`` 使用方法的更高级例子，请参考6.12小节。



==============================
5.10 内存映射的二进制文件
==============================

----------
问题
----------
你想内存映射一个二进制文件到一个可变字节数组中，目的可能是为了随机访问它的内容或者是原地做些修改。

----------
解决方案
----------
使用 ``mmap`` 模块来内存映射文件。
下面是一个工具函数，向你演示了如何打开一个文件并以一种便捷方式内存映射这个文件。

.. code-block:: python

    import os
    import mmap

    def memory_map(filename, access=mmap.ACCESS_WRITE):
        size = os.path.getsize(filename)
        fd = os.open(filename, os.O_RDWR)
        return mmap.mmap(fd, size, access=access)

为了使用这个函数，你需要有一个已创建并且内容不为空的文件。
下面是一个例子，教你怎样初始创建一个文件并将其内容扩充到指定大小：

.. code-block:: python

    >>> size = 1000000
    >>> with open('data', 'wb') as f:
    ...     f.seek(size-1)
    ...     f.write(b'\x00')
    ...
    >>>

下面是一个利用 ``memory_map()`` 函数类内存映射文件内容的例子：

.. code-block:: python

    >>> m = memory_map('data')
    >>> len(m)
    1000000
    >>> m[0:10]
    b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
    >>> m[0]
    0
    >>> # Reassign a slice
    >>> m[0:11] = b'Hello World'
    >>> m.close()

    >>> # Verify that changes were made
    >>> with open('data', 'rb') as f:
    ... print(f.read(11))
    ...
    b'Hello World'
    >>>

``mmap()`` 返回的 ``mmap`` 对象同样也可以作为一个上下文管理器来使用，
这时候底层的文件会被自动关闭。比如：

.. code-block:: python

    >>> with memory_map('data') as m:
    ...     print(len(m))
    ...     print(m[0:10])
    ...
    1000000
    b'Hello World'
    >>> m.closed
    True
    >>>

默认情况下， ``memeory_map()`` 函数打开的文件同时支持读和写操作。
任何的修改内容都会复制回原来的文件中。
如果需要只读的访问模式，可以给参数 ``access`` 赋值为 ``mmap.ACCESS_READ`` 。比如：

.. code-block:: python

    m = memory_map(filename, mmap.ACCESS_READ)

如果你想在本地修改数据，但是又不想将修改写回到原始文件中，可以使用 ``mmap.ACCESS_COPY`` ：

.. code-block:: python

    m = memory_map(filename, mmap.ACCESS_COPY)

----------
讨论
----------
为了随机访问文件的内容，使用 ``mmap`` 将文件映射到内存中是一个高效和优雅的方法。
例如，你无需打开一个文件并执行大量的 ``seek()`` ， ``read()`` ， ``write()`` 调用，
只需要简单的映射文件并使用切片操作访问数据即可。

一般来讲， ``mmap()`` 所暴露的内存看上去就是一个二进制数组对象。
但是，你可以使用一个内存视图来解析其中的数据。比如：

.. code-block:: python

    >>> m = memory_map('data')
    >>> # Memoryview of unsigned integers
    >>> v = memoryview(m).cast('I')
    >>> v[0] = 7
    >>> m[0:4]
    b'\x07\x00\x00\x00'
    >>> m[0:4] = b'\x07\x01\x00\x00'
    >>> v[0]
    263
    >>>

需要强调的一点是，内存映射一个文件并不会导致整个文件被读取到内存中。
也就是说，文件并没有被复制到内存缓存或数组中。相反，操作系统仅仅为文件内容保留了一段虚拟内存。
当你访问文件的不同区域时，这些区域的内容才根据需要被读取并映射到内存区域中。
而那些从没被访问到的部分还是留在磁盘上。所有这些过程是透明的，在幕后完成！

如果多个Python解释器内存映射同一个文件，得到的 ``mmap`` 对象能够被用来在解释器直接交换数据。
也就是说，所有解释器都能同时读写数据，并且其中一个解释器所做的修改会自动呈现在其他解释器中。
很明显，这里需要考虑同步的问题。但是这种方法有时候可以用来在管道或套接字间传递数据。

这一小节中函数尽量写得很通用，同时适用于Unix和Windows平台。
要注意的是使用 ``mmap()`` 函数时会在底层有一些平台的差异性。
另外，还有一些选项可以用来创建匿名的内存映射区域。
如果你对这个感兴趣，确保你仔细研读了Python文档中
`这方面的内容 <http://docs.python.org/3/library/mmap.html>`_ 。


==============================
5.11 文件路径名的操作
==============================

----------
问题
----------
你需要使用路径名来获取文件名，目录名，绝对路径等等。

----------
解决方案
----------
使用 ``os.path`` 模块中的函数来操作路径名。
下面是一个交互式例子来演示一些关键的特性：

.. code-block:: python

    >>> import os
    >>> path = '/Users/beazley/Data/data.csv'

    >>> # Get the last component of the path
    >>> os.path.basename(path)
    'data.csv'

    >>> # Get the directory name
    >>> os.path.dirname(path)
    '/Users/beazley/Data'

    >>> # Join path components together
    >>> os.path.join('tmp', 'data', os.path.basename(path))
    'tmp/data/data.csv'

    >>> # Expand the user's home directory
    >>> path = '~/Data/data.csv'
    >>> os.path.expanduser(path)
    '/Users/beazley/Data/data.csv'

    >>> # Split the file extension
    >>> os.path.splitext(path)
    ('~/Data/data', '.csv')
    >>>

----------
讨论
----------
对于任何的文件名的操作，你都应该使用 ``os.path`` 模块，而不是使用标准字符串操作来构造自己的代码。
特别是为了可移植性考虑的时候更应如此，
因为 ``os.path`` 模块知道Unix和Windows系统之间的差异并且能够可靠地处理类似 ``Data/data.csv``
和 ``Data\data.csv`` 这样的文件名。
其次，你真的不应该浪费时间去重复造轮子。通常最好是直接使用已经为你准备好的功能。

要注意的是 ``os.path`` 还有更多的功能在这里并没有列举出来。
可以查阅官方文档来获取更多与文件测试，符号链接等相关的函数说明。

==============================
5.12 测试文件是否存在
==============================

----------
问题
----------
你想测试一个文件或目录是否存在。

----------
解决方案
----------
使用 ``os.path`` 模块来测试一个文件或目录是否存在。比如：

.. code-block:: python

    >>> import os
    >>> os.path.exists('/etc/passwd')
    True
    >>> os.path.exists('/tmp/spam')
    False
    >>>

你还能进一步测试这个文件时什么类型的。
在下面这些测试中，如果测试的文件不存在的时候，结果都会返回False：

.. code-block:: python

    >>> # Is a regular file
    >>> os.path.isfile('/etc/passwd')
    True

    >>> # Is a directory
    >>> os.path.isdir('/etc/passwd')
    False

    >>> # Is a symbolic link
    >>> os.path.islink('/usr/local/bin/python3')
    True

    >>> # Get the file linked to
    >>> os.path.realpath('/usr/local/bin/python3')
    '/usr/local/bin/python3.3'
    >>>

如果你还想获取元数据(比如文件大小或者是修改日期)，也可以使用 ``os.path`` 模块来解决：

.. code-block:: python

    >>> os.path.getsize('/etc/passwd')
    3669
    >>> os.path.getmtime('/etc/passwd')
    1272478234.0
    >>> import time
    >>> time.ctime(os.path.getmtime('/etc/passwd'))
    'Wed Apr 28 13:10:34 2010'
    >>>

----------
讨论
----------
使用 ``os.path`` 来进行文件测试是很简单的。
在写这些脚本时，可能唯一需要注意的就是你需要考虑文件权限的问题，特别是在获取元数据时候。比如：

.. code-block:: python

    >>> os.path.getsize('/Users/guido/Desktop/foo.txt')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "/usr/local/lib/python3.3/genericpath.py", line 49, in getsize
            return os.stat(filename).st_size
    PermissionError: [Errno 13] Permission denied: '/Users/guido/Desktop/foo.txt'
    >>>

==============================
5.13 获取文件夹中的文件列表
==============================

----------
问题
----------
你想获取文件系统中某个目录下的所有文件列表。

----------
解决方案
----------
使用 ``os.listdir()`` 函数来获取某个目录中的文件列表：

.. code-block:: python

    import os
    names = os.listdir('somedir')

结果会返回目录中所有文件列表，包括所有文件，子目录，符号链接等等。
如果你需要通过某种方式过滤数据，可以考虑结合 ``os.path`` 库中的一些函数来使用列表推导。比如：

.. code-block:: python

    import os.path

    # Get all regular files
    names = [name for name in os.listdir('somedir')
            if os.path.isfile(os.path.join('somedir', name))]

    # Get all dirs
    dirnames = [name for name in os.listdir('somedir')
            if os.path.isdir(os.path.join('somedir', name))]

字符串的 ``startswith()`` 和 ``endswith()`` 方法对于过滤一个目录的内容也是很有用的。比如：

.. code-block:: python

    pyfiles = [name for name in os.listdir('somedir')
                if name.endswith('.py')]

对于文件名的匹配，你可能会考虑使用 ``glob`` 或 ``fnmatch`` 模块。比如：

.. code-block:: python

    import glob
    pyfiles = glob.glob('somedir/*.py')

    from fnmatch import fnmatch
    pyfiles = [name for name in os.listdir('somedir')
                if fnmatch(name, '*.py')]

----------
讨论
----------
获取目录中的列表是很容易的，但是其返回结果只是目录中实体名列表而已。
如果你还想获取其他的元信息，比如文件大小，修改时间等等，
你或许还需要使用到 ``os.path`` 模块中的函数或着 ``os.stat()`` 函数来收集数据。比如：

.. code-block:: python

    # Example of getting a directory listing

    import os
    import os.path
    import glob

    pyfiles = glob.glob('*.py')

    # Get file sizes and modification dates
    name_sz_date = [(name, os.path.getsize(name), os.path.getmtime(name))
                    for name in pyfiles]
    for name, size, mtime in name_sz_date:
        print(name, size, mtime)

    # Alternative: Get file metadata
    file_metadata = [(name, os.stat(name)) for name in pyfiles]
    for name, meta in file_metadata:
        print(name, meta.st_size, meta.st_mtime)

最后还有一点要注意的就是，有时候在处理文件名编码问题时候可能会出现一些问题。
通常来讲，函数 ``os.listdir()`` 返回的实体列表会根据系统默认的文件名编码来解码。
但是有时候也会碰到一些不能正常解码的文件名。
关于文件名的处理问题，在5.14和5.15小节有更详细的讲解。

==============================
5.14 忽略文件名编码
==============================

----------
问题
----------
你想使用原始文件名执行文件的I/O操作，也就是说文件名并没有经过系统默认编码去解码或编码过。

----------
解决方案
----------
默认情况下，所有的文件名都会根据 ``sys.getfilesystemencoding()`` 返回的文本编码来编码或解码。比如：

.. code-block:: python

    >>> sys.getfilesystemencoding()
    'utf-8'
    >>>

如果因为某种原因你想忽略这种编码，可以使用一个原始字节字符串来指定一个文件名即可。比如：

.. code-block:: python

    >>> # Wrte a file using a unicode filename
    >>> with open('jalape\xf1o.txt', 'w') as f:
    ...     f.write('Spicy!')
    ...
    6
    >>> # Directory listing (decoded)
    >>> import os
    >>> os.listdir('.')
    ['jalapeño.txt']

    >>> # Directory listing (raw)
    >>> os.listdir(b'.') # Note: byte string
    [b'jalapen\xcc\x83o.txt']

    >>> # Open file with raw filename
    >>> with open(b'jalapen\xcc\x83o.txt') as f:
    ...     print(f.read())
    ...
    Spicy!
    >>>

正如你所见，在最后两个操作中，当你给文件相关函数如 ``open()`` 和 ``os.listdir()``
传递字节字符串时，文件名的处理方式会稍有不同。

----------
讨论
----------
通常来讲，你不需要担心文件名的编码和解码，普通的文件名操作应该就没问题了。
但是，有些操作系统允许用户通过偶然或恶意方式去创建名字不符合默认编码的文件。
这些文件名可能会神秘地中断那些需要处理大量文件的Python程序。

读取目录并通过原始未解码方式处理文件名可以有效的避免这样的问题，
尽管这样会带来一定的编程难度。

关于打印不可解码的文件名，请参考5.15小节。
==========================
5.15 打印不合法的文件名
==========================

----------
问题
----------
你的程序获取了一个目录中的文件名列表，但是当它试着去打印文件名的时候程序崩溃，
出现了 ``UnicodeEncodeError`` 异常和一条奇怪的消息—— ``surrogates not allowed`` 。

----------
解决方案
----------
当打印未知的文件名时，使用下面的方法可以避免这样的错误：

.. code-block:: python

    def bad_filename(filename):
        return repr(filename)[1:-1]

    try:
        print(filename)
    except UnicodeEncodeError:
        print(bad_filename(filename))

----------
讨论
----------
这一小节讨论的是在编写必须处理文件系统的程序时一个不太常见但又很棘手的问题。
默认情况下，Python假定所有文件名都已经根据 ``sys.getfilesystemencoding()`` 的值编码过了。
但是，有一些文件系统并没有强制要求这样做，因此允许创建文件名没有正确编码的文件。
这种情况不太常见，但是总会有些用户冒险这样做或者是无意之中这样做了(
可能是在一个有缺陷的代码中给 ``open()`` 函数传递了一个不合规范的文件名)。

当执行类似 ``os.listdir()`` 这样的函数时，这些不合规范的文件名就会让Python陷入困境。
一方面，它不能仅仅只是丢弃这些不合格的名字。而另一方面，它又不能将这些文件名转换为正确的文本字符串。
Python对这个问题的解决方案是从文件名中获取未解码的字节值比如 ``\xhh``
并将它映射成Unicode字符 ``\udchh`` 表示的所谓的"代理编码"。
下面一个例子演示了当一个不合格目录列表中含有一个文件名为bäd.txt(使用Latin-1而不是UTF-8编码)时的样子：

.. code-block:: python

    >>> import os
    >>> files = os.listdir('.')
    >>> files
    ['spam.py', 'b\udce4d.txt', 'foo.txt']
    >>>

如果你有代码需要操作文件名或者将文件名传递给 ``open()`` 这样的函数，一切都能正常工作。
只有当你想要输出文件名时才会碰到些麻烦(比如打印输出到屏幕或日志文件等)。
特别的，当你想打印上面的文件名列表时，你的程序就会崩溃：

.. code-block:: python

    >>> for name in files:
    ...     print(name)
    ...
    spam.py
    Traceback (most recent call last):
        File "<stdin>", line 2, in <module>
    UnicodeEncodeError: 'utf-8' codec can't encode character '\udce4' in
    position 1: surrogates not allowed
    >>>

程序崩溃的原因就是字符 ``\udce4`` 是一个非法的Unicode字符。
它其实是一个被称为代理字符对的双字符组合的后半部分。
由于缺少了前半部分，因此它是个非法的Unicode。
所以，唯一能成功输出的方法就是当遇到不合法文件名时采取相应的补救措施。
比如可以将上述代码修改如下：

.. code-block:: python

    >>> for name in files:
    ... try:
    ...     print(name)
    ... except UnicodeEncodeError:
    ...     print(bad_filename(name))
    ...
    spam.py
    b\udce4d.txt
    foo.txt
    >>>

在 ``bad_filename()`` 函数中怎样处置取决于你自己。
另外一个选择就是通过某种方式重新编码，示例如下：

.. code-block:: python

    def bad_filename(filename):
        temp = filename.encode(sys.getfilesystemencoding(), errors='surrogateescape')
        return temp.decode('latin-1')

译者注::

    surrogateescape:
    这种是Python在绝大部分面向OS的API中所使用的错误处理器，
    它能以一种优雅的方式处理由操作系统提供的数据的编码问题。
    在解码出错时会将出错字节存储到一个很少被使用到的Unicode编码范围内。
    在编码时将那些隐藏值又还原回原先解码失败的字节序列。
    它不仅对于OS API非常有用，也能很容易的处理其他情况下的编码错误。

使用这个版本产生的输出如下：

.. code-block:: python

    >>> for name in files:
    ...     try:
    ...         print(name)
    ...     except UnicodeEncodeError:
    ...         print(bad_filename(name))
    ...
    spam.py
    bäd.txt
    foo.txt
    >>>

这一小节主题可能会被大部分读者所忽略。但是如果你在编写依赖文件名和文件系统的关键任务程序时，
就必须得考虑到这个。否则你可能会在某个周末被叫到办公室去调试一些令人费解的错误。

==============================
5.16 增加或改变已打开文件的编码
==============================

----------
问题
----------
你想在不关闭一个已打开的文件前提下增加或改变它的Unicode编码。

----------
解决方案
----------
如果你想给一个以二进制模式打开的文件添加Unicode编码/解码方式，
可以使用 ``io.TextIOWrapper()`` 对象包装它。比如：

.. code-block:: python

    import urllib.request
    import io

    u = urllib.request.urlopen('http://www.python.org')
    f = io.TextIOWrapper(u, encoding='utf-8')
    text = f.read()

如果你想修改一个已经打开的文本模式的文件的编码方式，可以先使用 ``detach()`` 方法移除掉已存在的文本编码层，
并使用新的编码方式代替。下面是一个在 ``sys.stdout`` 上修改编码方式的例子：

.. code-block:: python

    >>> import sys
    >>> sys.stdout.encoding
    'UTF-8'
    >>> sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding='latin-1')
    >>> sys.stdout.encoding
    'latin-1'
    >>>

这样做可能会中断你的终端，这里仅仅是为了演示而已。

----------
讨论
----------
I/O系统由一系列的层次构建而成。你可以试着运行下面这个操作一个文本文件的例子来查看这种层次：

.. code-block:: python

    >>> f = open('sample.txt','w')
    >>> f
    <_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
    >>> f.buffer
    <_io.BufferedWriter name='sample.txt'>
    >>> f.buffer.raw
    <_io.FileIO name='sample.txt' mode='wb'>
    >>>

在这个例子中，``io.TextIOWrapper`` 是一个编码和解码Unicode的文本处理层，
``io.BufferedWriter`` 是一个处理二进制数据的带缓冲的I/O层，
``io.FileIO`` 是一个表示操作系统底层文件描述符的原始文件。
增加或改变文本编码会涉及增加或改变最上面的 ``io.TextIOWrapper`` 层。

一般来讲，像上面例子这样通过访问属性值来直接操作不同的层是很不安全的。
例如，如果你试着使用下面这样的技术改变编码看看会发生什么：

.. code-block:: python

    >>> f
    <_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
    >>> f = io.TextIOWrapper(f.buffer, encoding='latin-1')
    >>> f
    <_io.TextIOWrapper name='sample.txt' encoding='latin-1'>
    >>> f.write('Hello')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ValueError: I/O operation on closed file.
    >>>

结果出错了，因为f的原始值已经被破坏了并关闭了底层的文件。

``detach()`` 方法会断开文件的最顶层并返回第二层，之后最顶层就没什么用了。例如：

.. code-block:: python

    >>> f = open('sample.txt', 'w')
    >>> f
    <_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
    >>> b = f.detach()
    >>> b
    <_io.BufferedWriter name='sample.txt'>
    >>> f.write('hello')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    ValueError: underlying buffer has been detached
    >>>

一旦断开最顶层后，你就可以给返回结果添加一个新的最顶层。比如：

.. code-block:: python

    >>> f = io.TextIOWrapper(b, encoding='latin-1')
    >>> f
    <_io.TextIOWrapper name='sample.txt' encoding='latin-1'>
    >>>

尽管已经向你演示了改变编码的方法，
但是你还可以利用这种技术来改变文件行处理、错误机制以及文件处理的其他方面。例如：

.. code-block:: python

    >>> sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding='ascii',
    ...                             errors='xmlcharrefreplace')
    >>> print('Jalape\u00f1o')
    Jalape&#241;o
    >>>

注意下最后输出中的非ASCII字符 ``ñ`` 是如何被 ``&#241;`` 取代的。
==============================
5.17 将字节写入文本文件
==============================

----------
问题
----------
你想在文本模式打开的文件中写入原始的字节数据。

----------
解决方案
----------
将字节数据直接写入文件的缓冲区即可，例如：

.. code-block:: python

    >>> import sys
    >>> sys.stdout.write(b'Hello\n')
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
    TypeError: must be str, not bytes
    >>> sys.stdout.buffer.write(b'Hello\n')
    Hello
    5
    >>>

类似的，能够通过读取文本文件的 ``buffer`` 属性来读取二进制数据。

----------
讨论
----------
I/O系统以层级结构的形式构建而成。
文本文件是通过在一个拥有缓冲的二进制模式文件上增加一个Unicode编码/解码层来创建。
``buffer`` 属性指向对应的底层文件。如果你直接访问它的话就会绕过文本编码/解码层。

本小节例子展示的 ``sys.stdout`` 可能看起来有点特殊。
默认情况下，``sys.stdout`` 总是以文本模式打开的。
但是如果你在写一个需要打印二进制数据到标准输出的脚本的话，你可以使用上面演示的技术来绕过文本编码层。

==============================
5.18 将文件描述符包装成文件对象
==============================

----------
问题
----------
你有一个对应于操作系统上一个已打开的I/O通道(比如文件、管道、套接字等)的整型文件描述符，
你想将它包装成一个更高层的Python文件对象。

----------
解决方案
----------
一个文件描述符和一个打开的普通文件是不一样的。
文件描述符仅仅是一个由操作系统指定的整数，用来指代某个系统的I/O通道。
如果你碰巧有这么一个文件描述符，你可以通过使用 ``open()`` 函数来将其包装为一个Python的文件对象。
你仅仅只需要使用这个整数值的文件描述符作为第一个参数来代替文件名即可。例如：

.. code-block:: python

    # Open a low-level file descriptor
    import os
    fd = os.open('somefile.txt', os.O_WRONLY | os.O_CREAT)

    # Turn into a proper file
    f = open(fd, 'wt')
    f.write('hello world\n')
    f.close()

当高层的文件对象被关闭或者破坏的时候，底层的文件描述符也会被关闭。
如果这个并不是你想要的结果，你可以给 ``open()`` 函数传递一个可选的 ``colsefd=False`` 。比如：

.. code-block:: python

    # Create a file object, but don't close underlying fd when done
    f = open(fd, 'wt', closefd=False)
    ...

----------
讨论
----------
在Unix系统中，这种包装文件描述符的技术可以很方便的将一个类文件接口作用于一个以不同方式打开的I/O通道上，
如管道、套接字等。举例来讲，下面是一个操作管道的例子：

.. code-block:: python

    from socket import socket, AF_INET, SOCK_STREAM

    def echo_client(client_sock, addr):
        print('Got connection from', addr)

        # Make text-mode file wrappers for socket reading/writing
        client_in = open(client_sock.fileno(), 'rt', encoding='latin-1',
                    closefd=False)

        client_out = open(client_sock.fileno(), 'wt', encoding='latin-1',
                    closefd=False)

        # Echo lines back to the client using file I/O
        for line in client_in:
            client_out.write(line)
            client_out.flush()

        client_sock.close()

    def echo_server(address):
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(address)
        sock.listen(1)
        while True:
            client, addr = sock.accept()
            echo_client(client, addr)

需要重点强调的一点是，上面的例子仅仅是为了演示内置的 ``open()`` 函数的一个特性，并且也只适用于基于Unix的系统。
如果你想将一个类文件接口作用在一个套接字并希望你的代码可以跨平台，请使用套接字对象的 ``makefile()`` 方法。
但是如果不考虑可移植性的话，那上面的解决方案会比使用 ``makefile()`` 性能更好一点。

你也可以使用这种技术来构造一个别名，允许以不同于第一次打开文件的方式使用它。
例如，下面演示如何创建一个文件对象，它允许你输出二进制数据到标准输出(通常以文本模式打开)：

.. code-block:: python

    import sys
    # Create a binary-mode file for stdout
    bstdout = open(sys.stdout.fileno(), 'wb', closefd=False)
    bstdout.write(b'Hello World\n')
    bstdout.flush()

尽管可以将一个已存在的文件描述符包装成一个正常的文件对象，
但是要注意的是并不是所有的文件模式都被支持，并且某些类型的文件描述符可能会有副作用
(特别是涉及到错误处理、文件结尾条件等等的时候)。
在不同的操作系统上这种行为也是不一样，特别的，上面的例子都不能在非Unix系统上运行。
我说了这么多，意思就是让你充分测试自己的实现代码，确保它能按照期望工作。
==============================
5.19 创建临时文件和文件夹
==============================

----------
问题
----------
你需要在程序执行时创建一个临时文件或目录，并希望使用完之后可以自动销毁掉。

----------
解决方案
----------
``tempfile`` 模块中有很多的函数可以完成这任务。
为了创建一个匿名的临时文件，可以使用 ``tempfile.TemporaryFile`` ：

.. code-block:: python

    from tempfile import TemporaryFile

    with TemporaryFile('w+t') as f:
        # Read/write to the file
        f.write('Hello World\n')
        f.write('Testing\n')

        # Seek back to beginning and read the data
        f.seek(0)
        data = f.read()

    # Temporary file is destroyed

或者，如果你喜欢，你还可以像这样使用临时文件：

.. code-block:: python

    f = TemporaryFile('w+t')
    # Use the temporary file
    ...
    f.close()
    # File is destroyed

``TemporaryFile()`` 的第一个参数是文件模式，通常来讲文本模式使用 ``w+t`` ，二进制模式使用 ``w+b`` 。
这个模式同时支持读和写操作，在这里是很有用的，因为当你关闭文件去改变模式的时候，文件实际上已经不存在了。
``TemporaryFile()`` 另外还支持跟内置的 ``open()`` 函数一样的参数。比如：

.. code-block:: python

    with TemporaryFile('w+t', encoding='utf-8', errors='ignore') as f:
        ...

在大多数Unix系统上，通过 ``TemporaryFile()`` 创建的文件都是匿名的，甚至连目录都没有。
如果你想打破这个限制，可以使用 ``NamedTemporaryFile()`` 来代替。比如：

.. code-block:: python

    from tempfile import NamedTemporaryFile

    with NamedTemporaryFile('w+t') as f:
        print('filename is:', f.name)
        ...

    # File automatically destroyed

这里，被打开文件的 ``f.name`` 属性包含了该临时文件的文件名。
当你需要将文件名传递给其他代码来打开这个文件的时候，这个就很有用了。
和 ``TemporaryFile()`` 一样，结果文件关闭时会被自动删除掉。
如果你不想这么做，可以传递一个关键字参数 ``delete=False`` 即可。比如：

.. code-block:: python

    with NamedTemporaryFile('w+t', delete=False) as f:
        print('filename is:', f.name)
        ...

为了创建一个临时目录，可以使用 ``tempfile.TemporaryDirectory()`` 。比如：

.. code-block:: python

    from tempfile import TemporaryDirectory

    with TemporaryDirectory() as dirname:
        print('dirname is:', dirname)
        # Use the directory
        ...
    # Directory and all contents destroyed

----------
讨论
----------
``TemporaryFile()`` 、``NamedTemporaryFile()`` 和 ``TemporaryDirectory()`` 函数
应该是处理临时文件目录的最简单的方式了，因为它们会自动处理所有的创建和清理步骤。
在一个更低的级别，你可以使用 ``mkstemp()`` 和 ``mkdtemp()`` 来创建临时文件和目录。比如：

.. code-block:: python

    >>> import tempfile
    >>> tempfile.mkstemp()
    (3, '/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-/tmp7fefhv')
    >>> tempfile.mkdtemp()
    '/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-/tmp5wvcv6'
    >>>

但是，这些函数并不会做进一步的管理了。
例如，函数 ``mkstemp()`` 仅仅就返回一个原始的OS文件描述符，你需要自己将它转换为一个真正的文件对象。
同样你还需要自己清理这些文件。

通常来讲，临时文件在系统默认的位置被创建，比如 ``/var/tmp`` 或类似的地方。
为了获取真实的位置，可以使用 ``tempfile.gettempdir()`` 函数。比如：

.. code-block:: python

    >>> tempfile.gettempdir()
    '/var/folders/7W/7WZl5sfZEF0pljrEB1UMWE+++TI/-Tmp-'
    >>>

所有和临时文件相关的函数都允许你通过使用关键字参数
``prefix`` 、``suffix`` 和 ``dir`` 来自定义目录以及命名规则。比如：

.. code-block:: python

    >>> f = NamedTemporaryFile(prefix='mytemp', suffix='.txt', dir='/tmp')
    >>> f.name
    '/tmp/mytemp8ee899.txt'
    >>>

最后还有一点，尽可能以最安全的方式使用 ``tempfile`` 模块来创建临时文件。
包括仅给当前用户授权访问以及在文件创建过程中采取措施避免竞态条件。
要注意的是不同的平台可能会不一样。因此你最好阅读
`官方文档 <https://docs.python.org/3/library/tempfile.html>`_ 来了解更多的细节。



==============================
5.20 与串行端口的数据通信
==============================

----------
问题
----------
你想通过串行端口读写数据，典型场景就是和一些硬件设备打交道(比如一个机器人或传感器)。

----------
解决方案
----------
尽管你可以通过使用Python内置的I/O模块来完成这个任务，但对于串行通信最好的选择是使用
`pySerial包 <http://pyserial.sourceforge.net/>`_ 。
这个包的使用非常简单，先安装pySerial，使用类似下面这样的代码就能很容易的打开一个串行端口：

.. code-block:: python

    import serial
    ser = serial.Serial('/dev/tty.usbmodem641', # Device name varies
                        baudrate=9600,
                        bytesize=8,
                        parity='N',
                        stopbits=1)

设备名对于不同的设备和操作系统是不一样的。
比如，在Windows系统上，你可以使用0, 1等表示的一个设备来打开通信端口"COM0"和"COM1"。
一旦端口打开，那就可以使用 ``read()``，``readline()`` 和 ``write()`` 函数读写数据了。例如：

.. code-block:: python

    ser.write(b'G1 X50 Y50\r\n')
    resp = ser.readline()

大多数情况下，简单的串口通信从此变得十分简单。

----------
讨论
----------
尽管表面上看起来很简单，其实串口通信有时候也是挺麻烦的。
推荐你使用第三方包如 ``pySerial`` 的一个原因是它提供了对高级特性的支持
(比如超时，控制流，缓冲区刷新，握手协议等等)。举个例子，如果你想启用 ``RTS-CTS`` 握手协议，
你只需要给 ``Serial()`` 传递一个 ``rtscts=True`` 的参数即可。
其官方文档非常完善，因此我在这里极力推荐这个包。

时刻记住所有涉及到串口的I/O都是二进制模式的。因此，确保你的代码使用的是字节而不是文本
(或有时候执行文本的编码/解码操作)。
另外当你需要创建二进制编码的指令或数据包的时候，``struct`` 模块也是非常有用的。

==============================
5.21 序列化Python对象
==============================

----------
问题
----------
你需要将一个Python对象序列化为一个字节流，以便将它保存到一个文件、存储到数据库或者通过网络传输它。

----------
解决方案
----------
对于序列化最普遍的做法就是使用 ``pickle`` 模块。为了将一个对象保存到一个文件中，可以这样做：

.. code-block:: python

    import pickle

    data = ... # Some Python object
    f = open('somefile', 'wb')
    pickle.dump(data, f)

为了将一个对象转储为一个字符串，可以使用 ``pickle.dumps()`` ：

.. code-block:: python

    s = pickle.dumps(data)

为了从字节流中恢复一个对象，使用 ``pickle.load()`` 或 ``pickle.loads()`` 函数。比如：

.. code-block:: python

    # Restore from a file
    f = open('somefile', 'rb')
    data = pickle.load(f)

    # Restore from a string
    data = pickle.loads(s)

----------
讨论
----------
对于大多数应用程序来讲，``dump()`` 和 ``load()`` 函数的使用就是你有效使用 ``pickle`` 模块所需的全部了。
它可适用于绝大部分Python数据类型和用户自定义类的对象实例。
如果你碰到某个库可以让你在数据库中保存/恢复Python对象或者是通过网络传输对象的话，
那么很有可能这个库的底层就使用了 ``pickle`` 模块。

``pickle`` 是一种Python特有的自描述的数据编码。
通过自描述，被序列化后的数据包含每个对象开始和结束以及它的类型信息。
因此，你无需担心对象记录的定义，它总是能工作。
举个例子，如果要处理多个对象，你可以这样做：

.. code-block:: python

    >>> import pickle
    >>> f = open('somedata', 'wb')
    >>> pickle.dump([1, 2, 3, 4], f)
    >>> pickle.dump('hello', f)
    >>> pickle.dump({'Apple', 'Pear', 'Banana'}, f)
    >>> f.close()
    >>> f = open('somedata', 'rb')
    >>> pickle.load(f)
    [1, 2, 3, 4]
    >>> pickle.load(f)
    'hello'
    >>> pickle.load(f)
    {'Apple', 'Pear', 'Banana'}
    >>>

你还能序列化函数，类，还有接口，但是结果数据仅仅将它们的名称编码成对应的代码对象。例如：

.. code-block:: python

    >>> import math
    >>> import pickle.
    >>> pickle.dumps(math.cos)
    b'\x80\x03cmath\ncos\nq\x00.'
    >>>

当数据反序列化回来的时候，会先假定所有的源数据时可用的。
模块、类和函数会自动按需导入进来。对于Python数据被不同机器上的解析器所共享的应用程序而言，
数据的保存可能会有问题，因为所有的机器都必须访问同一个源代码。

注 ::

    千万不要对不信任的数据使用pickle.load()。
    pickle在加载时有一个副作用就是它会自动加载相应模块并构造实例对象。
    但是某个坏人如果知道pickle的工作原理，
    他就可以创建一个恶意的数据导致Python执行随意指定的系统命令。
    因此，一定要保证pickle只在相互之间可以认证对方的解析器的内部使用。

有些类型的对象是不能被序列化的。这些通常是那些依赖外部系统状态的对象，
比如打开的文件，网络连接，线程，进程，栈帧等等。
用户自定义类可以通过提供 ``__getstate__()`` 和 ``__setstate__()`` 方法来绕过这些限制。
如果定义了这两个方法，``pickle.dump()`` 就会调用 ``__getstate__()`` 获取序列化的对象。
类似的，``__setstate__()`` 在反序列化时被调用。为了演示这个工作原理，
下面是一个在内部定义了一个线程但仍然可以序列化和反序列化的类：

.. code-block:: python

    # countdown.py
    import time
    import threading

    class Countdown:
        def __init__(self, n):
            self.n = n
            self.thr = threading.Thread(target=self.run)
            self.thr.daemon = True
            self.thr.start()

        def run(self):
            while self.n > 0:
                print('T-minus', self.n)
                self.n -= 1
                time.sleep(5)

        def __getstate__(self):
            return self.n

        def __setstate__(self, n):
            self.__init__(n)

试着运行下面的序列化试验代码：

.. code-block:: python

    >>> import countdown
    >>> c = countdown.Countdown(30)
    >>> T-minus 30
    T-minus 29
    T-minus 28
    ...

    >>> # After a few moments
    >>> f = open('cstate.p', 'wb')
    >>> import pickle
    >>> pickle.dump(c, f)
    >>> f.close()


然后退出Python解析器并重启后再试验下：

.. code-block:: python

    >>> f = open('cstate.p', 'rb')
    >>> pickle.load(f)
    countdown.Countdown object at 0x10069e2d0>
    T-minus 19
    T-minus 18
    ...

你可以看到线程又奇迹般的重生了，从你第一次序列化它的地方又恢复过来。

``pickle`` 对于大型的数据结构比如使用 ``array`` 或 ``numpy``
模块创建的二进制数组效率并不是一个高效的编码方式。
如果你需要移动大量的数组数据，你最好是先在一个文件中将其保存为数组数据块或使用更高级的标准编码方式如HDF5
(需要第三方库的支持)。

由于 ``pickle`` 是Python特有的并且附着在源码上，所有如果需要长期存储数据的时候不应该选用它。
例如，如果源码变动了，你所有的存储数据可能会被破坏并且变得不可读取。
坦白来讲，对于在数据库和存档文件中存储数据时，你最好使用更加标准的数据编码格式如XML，CSV或JSON。
这些编码格式更标准，可以被不同的语言支持，并且也能很好的适应源码变更。

最后一点要注意的是  ``pickle`` 有大量的配置选项和一些棘手的问题。
对于最常见的使用场景，你不需要去担心这个，但是如果你要在一个重要的程序中使用pickle去做序列化的话，
最好去查阅一下 `官方文档 <https://docs.python.org/3/library/pickle.html>`_ 。
============================
6.1 读写CSV数据
============================

----------
问题
----------
你想读写一个CSV格式的文件。

----------
解决方案
----------
对于大多数的CSV格式的数据读写问题，都可以使用 ``csv`` 库。
例如：假设你在一个名叫stocks.csv文件中有一些股票市场数据，就像这样：

.. code-block:: python

    Symbol,Price,Date,Time,Change,Volume
    "AA",39.48,"6/11/2007","9:36am",-0.18,181800
    "AIG",71.38,"6/11/2007","9:36am",-0.15,195500
    "AXP",62.58,"6/11/2007","9:36am",-0.46,935000
    "BA",98.31,"6/11/2007","9:36am",+0.12,104800
    "C",53.08,"6/11/2007","9:36am",-0.25,360900
    "CAT",78.29,"6/11/2007","9:36am",-0.23,225400

下面向你展示如何将这些数据读取为一个元组的序列：

.. code-block:: python

    import csv
    with open('stocks.csv') as f:
        f_csv = csv.reader(f)
        headers = next(f_csv)
        for row in f_csv:
            # Process row
            ...

在上面的代码中， ``row`` 会是一个列表。因此，为了访问某个字段，你需要使用下标，如 ``row[0]`` 访问Symbol， ``row[4]`` 访问Change。

由于这种下标访问通常会引起混淆，你可以考虑使用命名元组。例如：

.. code-block:: python

    from collections import namedtuple
    with open('stock.csv') as f:
        f_csv = csv.reader(f)
        headings = next(f_csv)
        Row = namedtuple('Row', headings)
        for r in f_csv:
            row = Row(*r)
            # Process row
            ...

它允许你使用列名如 ``row.Symbol`` 和 ``row.Change`` 代替下标访问。
需要注意的是这个只有在列名是合法的Python标识符的时候才生效。如果不是的话，
你可能需要修改下原始的列名(如将非标识符字符替换成下划线之类的)。

另外一个选择就是将数据读取到一个字典序列中去。可以这样做：

.. code-block:: python

    import csv
    with open('stocks.csv') as f:
        f_csv = csv.DictReader(f)
        for row in f_csv:
            # process row
            ...

在这个版本中，你可以使用列名去访问每一行的数据了。比如，``row['Symbol']`` 或者 ``row['Change']``

为了写入CSV数据，你仍然可以使用csv模块，不过这时候先创建一个 ``writer`` 对象。例如:

.. code-block:: python

    headers = ['Symbol','Price','Date','Time','Change','Volume']
    rows = [('AA', 39.48, '6/11/2007', '9:36am', -0.18, 181800),
             ('AIG', 71.38, '6/11/2007', '9:36am', -0.15, 195500),
             ('AXP', 62.58, '6/11/2007', '9:36am', -0.46, 935000),
           ]

    with open('stocks.csv','w') as f:
        f_csv = csv.writer(f)
        f_csv.writerow(headers)
        f_csv.writerows(rows)

如果你有一个字典序列的数据，可以像这样做：

.. code-block:: python

    headers = ['Symbol', 'Price', 'Date', 'Time', 'Change', 'Volume']
    rows = [{'Symbol':'AA', 'Price':39.48, 'Date':'6/11/2007',
            'Time':'9:36am', 'Change':-0.18, 'Volume':181800},
            {'Symbol':'AIG', 'Price': 71.38, 'Date':'6/11/2007',
            'Time':'9:36am', 'Change':-0.15, 'Volume': 195500},
            {'Symbol':'AXP', 'Price': 62.58, 'Date':'6/11/2007',
            'Time':'9:36am', 'Change':-0.46, 'Volume': 935000},
            ]

    with open('stocks.csv','w') as f:
        f_csv = csv.DictWriter(f, headers)
        f_csv.writeheader()
        f_csv.writerows(rows)

----------
讨论
----------
你应该总是优先选择csv模块分割或解析CSV数据。例如，你可能会像编写类似下面这样的代码：

.. code-block:: python

    with open('stocks.csv') as f:
    for line in f:
        row = line.split(',')
        # process row
        ...

使用这种方式的一个缺点就是你仍然需要去处理一些棘手的细节问题。
比如，如果某些字段值被引号包围，你不得不去除这些引号。
另外，如果一个被引号包围的字段碰巧含有一个逗号，那么程序就会因为产生一个错误大小的行而出错。

默认情况下，``csv`` 库可识别Microsoft Excel所使用的CSV编码规则。
这或许也是最常见的形式，并且也会给你带来最好的兼容性。
然而，如果你查看csv的文档，就会发现有很多种方法将它应用到其他编码格式上(如修改分割字符等)。
例如，如果你想读取以tab分割的数据，可以这样做：

.. code-block:: python

    # Example of reading tab-separated values
    with open('stock.tsv') as f:
        f_tsv = csv.reader(f, delimiter='\t')
        for row in f_tsv:
            # Process row
            ...

如果你正在读取CSV数据并将它们转换为命名元组，需要注意对列名进行合法性认证。
例如，一个CSV格式文件有一个包含非法标识符的列头行，类似下面这样：

.. code-block:: text

    Street Address,Num-Premises,Latitude,Longitude 5412 N CLARK,10,41.980262,-87.668452

这样最终会导致在创建一个命名元组时产生一个 ``ValueError`` 异常而失败。
为了解决这问题，你可能不得不先去修正列标题。
例如，可以像下面这样在非法标识符上使用一个正则表达式替换：

.. code-block:: python

    import re
    with open('stock.csv') as f:
        f_csv = csv.reader(f)
        headers = [ re.sub('[^a-zA-Z_]', '_', h) for h in next(f_csv) ]
        Row = namedtuple('Row', headers)
        for r in f_csv:
            row = Row(*r)
            # Process row
            ...

还有重要的一点需要强调的是，csv产生的数据都是字符串类型的，它不会做任何其他类型的转换。
如果你需要做这样的类型转换，你必须自己手动去实现。
下面是一个在CSV数据上执行其他类型转换的例子：

.. code-block:: python

    col_types = [str, float, str, str, float, int]
    with open('stocks.csv') as f:
        f_csv = csv.reader(f)
        headers = next(f_csv)
        for row in f_csv:
            # Apply conversions to the row items
            row = tuple(convert(value) for convert, value in zip(col_types, row))
            ...

另外，下面是一个转换字典中特定字段的例子：

.. code-block:: python

    print('Reading as dicts with type conversion')
    field_types = [ ('Price', float),
                    ('Change', float),
                    ('Volume', int) ]

    with open('stocks.csv') as f:
        for row in csv.DictReader(f):
            row.update((key, conversion(row[key]))
                    for key, conversion in field_types)
            print(row)

通常来讲，你可能并不想过多去考虑这些转换问题。
在实际情况中，CSV文件都或多或少有些缺失的数据，被破坏的数据以及其它一些让转换失败的问题。
因此，除非你的数据确实有保障是准确无误的，否则你必须考虑这些问题(你可能需要增加合适的错误处理机制)。

最后，如果你读取CSV数据的目的是做数据分析和统计的话，
你可能需要看一看 ``Pandas`` 包。``Pandas`` 包含了一个非常方便的函数叫 ``pandas.read_csv()`` ，
它可以加载CSV数据到一个 ``DataFrame`` 对象中去。
然后利用这个对象你就可以生成各种形式的统计、过滤数据以及执行其他高级操作了。
在6.13小节中会有这样一个例子。
============================
6.2 读写JSON数据
============================

----------
问题
----------
你想读写JSON(JavaScript Object Notation)编码格式的数据。

----------
解决方案
----------
``json`` 模块提供了一种很简单的方式来编码和解码JSON数据。
其中两个主要的函数是 ``json.dumps()`` 和 ``json.loads()`` ，
要比其他序列化函数库如pickle的接口少得多。
下面演示如何将一个Python数据结构转换为JSON：

.. code-block:: python

    import json

    data = {
        'name' : 'ACME',
        'shares' : 100,
        'price' : 542.23
    }

    json_str = json.dumps(data)

下面演示如何将一个JSON编码的字符串转换回一个Python数据结构：

.. code-block:: python

    data = json.loads(json_str)

如果你要处理的是文件而不是字符串，你可以使用 ``json.dump()`` 和 ``json.load()`` 来编码和解码JSON数据。例如：

.. code-block:: python

    # Writing JSON data
    with open('data.json', 'w') as f:
        json.dump(data, f)

    # Reading data back
    with open('data.json', 'r') as f:
        data = json.load(f)

----------
讨论
----------
JSON编码支持的基本数据类型为 ``None`` ， ``bool`` ， ``int`` ， ``float`` 和 ``str`` ，
以及包含这些类型数据的lists，tuples和dictionaries。
对于dictionaries，keys需要是字符串类型(字典中任何非字符串类型的key在编码时会先转换为字符串)。
为了遵循JSON规范，你应该只编码Python的lists和dictionaries。
而且，在web应用程序中，顶层对象被编码为一个字典是一个标准做法。

JSON编码的格式对于Python语法而已几乎是完全一样的，除了一些小的差异之外。
比如，True会被映射为true，False被映射为false，而None会被映射为null。
下面是一个例子，演示了编码后的字符串效果：

.. code-block:: python

    >>> json.dumps(False)
    'false'
    >>> d = {'a': True,
    ...     'b': 'Hello',
    ...     'c': None}
    >>> json.dumps(d)
    '{"b": "Hello", "c": null, "a": true}'
    >>>

如果你试着去检查JSON解码后的数据，你通常很难通过简单的打印来确定它的结构，
特别是当数据的嵌套结构层次很深或者包含大量的字段时。
为了解决这个问题，可以考虑使用pprint模块的 ``pprint()`` 函数来代替普通的 ``print()`` 函数。
它会按照key的字母顺序并以一种更加美观的方式输出。
下面是一个演示如何漂亮的打印输出Twitter上搜索结果的例子：

.. code-block:: python

    >>> from urllib.request import urlopen
    >>> import json
    >>> u = urlopen('http://search.twitter.com/search.json?q=python&rpp=5')
    >>> resp = json.loads(u.read().decode('utf-8'))
    >>> from pprint import pprint
    >>> pprint(resp)
    {'completed_in': 0.074,
    'max_id': 264043230692245504,
    'max_id_str': '264043230692245504',
    'next_page': '?page=2&max_id=264043230692245504&q=python&rpp=5',
    'page': 1,
    'query': 'python',
    'refresh_url': '?since_id=264043230692245504&q=python',
    'results': [{'created_at': 'Thu, 01 Nov 2012 16:36:26 +0000',
                'from_user': ...
                },
                {'created_at': 'Thu, 01 Nov 2012 16:36:14 +0000',
                'from_user': ...
                },
                {'created_at': 'Thu, 01 Nov 2012 16:36:13 +0000',
                'from_user': ...
                },
                {'created_at': 'Thu, 01 Nov 2012 16:36:07 +0000',
                'from_user': ...
                }
                {'created_at': 'Thu, 01 Nov 2012 16:36:04 +0000',
                'from_user': ...
                }],
    'results_per_page': 5,
    'since_id': 0,
    'since_id_str': '0'}
    >>>

一般来讲，JSON解码会根据提供的数据创建dicts或lists。
如果你想要创建其他类型的对象，可以给 ``json.loads()`` 传递object_pairs_hook或object_hook参数。
例如，下面是演示如何解码JSON数据并在一个OrderedDict中保留其顺序的例子：

.. code-block:: python

    >>> s = '{"name": "ACME", "shares": 50, "price": 490.1}'
    >>> from collections import OrderedDict
    >>> data = json.loads(s, object_pairs_hook=OrderedDict)
    >>> data
    OrderedDict([('name', 'ACME'), ('shares', 50), ('price', 490.1)])
    >>>

下面是如何将一个JSON字典转换为一个Python对象例子：

.. code-block:: python

    >>> class JSONObject:
    ...     def __init__(self, d):
    ...         self.__dict__ = d
    ...
    >>>
    >>> data = json.loads(s, object_hook=JSONObject)
    >>> data.name
    'ACME'
    >>> data.shares
    50
    >>> data.price
    490.1
    >>>

最后一个例子中，JSON解码后的字典作为一个单个参数传递给 ``__init__()`` 。
然后，你就可以随心所欲的使用它了，比如作为一个实例字典来直接使用它。

在编码JSON的时候，还有一些选项很有用。
如果你想获得漂亮的格式化字符串后输出，可以使用 ``json.dumps()`` 的indent参数。
它会使得输出和pprint()函数效果类似。比如：

.. code-block:: python

    >>> print(json.dumps(data))
    {"price": 542.23, "name": "ACME", "shares": 100}
    >>> print(json.dumps(data, indent=4))
    {
        "price": 542.23,
        "name": "ACME",
        "shares": 100
    }
    >>>

对象实例通常并不是JSON可序列化的。例如：

.. code-block:: python

    >>> class Point:
    ...     def __init__(self, x, y):
    ...         self.x = x
    ...         self.y = y
    ...
    >>> p = Point(2, 3)
    >>> json.dumps(p)
    Traceback (most recent call last):
        File "<stdin>", line 1, in <module>
        File "/usr/local/lib/python3.3/json/__init__.py", line 226, in dumps
            return _default_encoder.encode(obj)
        File "/usr/local/lib/python3.3/json/encoder.py", line 187, in encode
            chunks = self.iterencode(o, _one_shot=True)
        File "/usr/local/lib/python3.3/json/encoder.py", line 245, in iterencode
            return _iterencode(o, 0)
        File "/usr/local/lib/python3.3/json/encoder.py", line 169, in default
            raise TypeError(repr(o) + " is not JSON serializable")
    TypeError: <__main__.Point object at 0x1006f2650> is not JSON serializable
    >>>

如果你想序列化对象实例，你可以提供一个函数，它的输入是一个实例，返回一个可序列化的字典。例如：

.. code-block:: python

    def serialize_instance(obj):
        d = { '__classname__' : type(obj).__name__ }
        d.update(vars(obj))
        return d

如果你想反过来获取这个实例，可以这样做：

.. code-block:: python

    # Dictionary mapping names to known classes
    classes = {
        'Point' : Point
    }

    def unserialize_object(d):
        clsname = d.pop('__classname__', None)
        if clsname:
            cls = classes[clsname]
            obj = cls.__new__(cls) # Make instance without calling __init__
            for key, value in d.items():
                setattr(obj, key, value)
            return obj
        else:
            return d

下面是如何使用这些函数的例子：

.. code-block:: python

    >>> p = Point(2,3)
    >>> s = json.dumps(p, default=serialize_instance)
    >>> s
    '{"__classname__": "Point", "y": 3, "x": 2}'
    >>> a = json.loads(s, object_hook=unserialize_object)
    >>> a
    <__main__.Point object at 0x1017577d0>
    >>> a.x
    2
    >>> a.y
    3
    >>>

``json`` 模块还有很多其他选项来控制更低级别的数字、特殊值如NaN等的解析。
可以参考官方文档获取更多细节。

============================
6.3 解析简单的XML数据
============================

----------
问题
----------
你想从一个简单的XML文档中提取数据。

----------
解决方案
----------
可以使用 ``xml.etree.ElementTree`` 模块从简单的XML文档中提取数据。
为了演示，假设你想解析Planet Python上的RSS源。下面是相应的代码：

.. code-block:: python

    from urllib.request import urlopen
    from xml.etree.ElementTree import parse

    # Download the RSS feed and parse it
    u = urlopen('http://planet.python.org/rss20.xml')
    doc = parse(u)

    # Extract and output tags of interest
    for item in doc.iterfind('channel/item'):
        title = item.findtext('title')
        date = item.findtext('pubDate')
        link = item.findtext('link')

        print(title)
        print(date)
        print(link)
        print()

运行上面的代码，输出结果类似这样：

.. code-block:: python

    Steve Holden: Python for Data Analysis
    Mon, 19 Nov 2012 02:13:51 +0000
    http://holdenweb.blogspot.com/2012/11/python-for-data-analysis.html

    Vasudev Ram: The Python Data model (for v2 and v3)
    Sun, 18 Nov 2012 22:06:47 +0000
    http://jugad2.blogspot.com/2012/11/the-python-data-model.html

    Python Diary: Been playing around with Object Databases
    Sun, 18 Nov 2012 20:40:29 +0000
    http://www.pythondiary.com/blog/Nov.18,2012/been-...-object-databases.html

    Vasudev Ram: Wakari, Scientific Python in the cloud
    Sun, 18 Nov 2012 20:19:41 +0000
    http://jugad2.blogspot.com/2012/11/wakari-scientific-python-in-cloud.html

    Jesse Jiryu Davis: Toro: synchronization primitives for Tornado coroutines
    Sun, 18 Nov 2012 20:17:49 +0000
    http://feedproxy.google.com/~r/EmptysquarePython/~3/_DOZT2Kd0hQ/

很显然，如果你想做进一步的处理，你需要替换 ``print()`` 语句来完成其他有趣的事。

----------
讨论
----------
在很多应用程序中处理XML编码格式的数据是很常见的。
不仅因为XML在Internet上面已经被广泛应用于数据交换，
同时它也是一种存储应用程序数据的常用格式(比如字处理，音乐库等)。
接下来的讨论会先假定读者已经对XML基础比较熟悉了。

在很多情况下，当使用XML来仅仅存储数据的时候，对应的文档结构非常紧凑并且直观。
例如，上面例子中的RSS订阅源类似于下面的格式：

.. code-block:: python

    <?xml version="1.0"?>
    <rss version="2.0" xmlns:dc="http://purl.org/dc/elements/1.1/">
        <channel>
            <title>Planet Python</title>
            <link>http://planet.python.org/</link>
            <language>en</language>
            <description>Planet Python - http://planet.python.org/</description>
            <item>
                <title>Steve Holden: Python for Data Analysis</title>
                <guid>http://holdenweb.blogspot.com/...-data-analysis.html</guid>
                <link>http://holdenweb.blogspot.com/...-data-analysis.html</link>
                <description>...</description>
                <pubDate>Mon, 19 Nov 2012 02:13:51 +0000</pubDate>
            </item>
            <item>
                <title>Vasudev Ram: The Python Data model (for v2 and v3)</title>
                <guid>http://jugad2.blogspot.com/...-data-model.html</guid>
                <link>http://jugad2.blogspot.com/...-data-model.html</link>
                <description>...</description>
                <pubDate>Sun, 18 Nov 2012 22:06:47 +0000</pubDate>
            </item>
            <item>
                <title>Python Diary: Been playing around with Object Databases</title>
                <guid>http://www.pythondiary.com/...-object-databases.html</guid>
                <link>http://www.pythondiary.com/...-object-databases.html</link>
                <description>...</description>
                <pubDate>Sun, 18 Nov 2012 20:40:29 +0000</pubDate>
            </item>
            ...
        </channel>
    </rss>

``xml.etree.ElementTree.parse()`` 函数解析整个XML文档并将其转换成一个文档对象。
然后，你就能使用 ``find()`` 、``iterfind()`` 和 ``findtext()`` 等方法来搜索特定的XML元素了。
这些函数的参数就是某个指定的标签名，例如 ``channel/item`` 或 ``title`` 。

每次指定某个标签时，你需要遍历整个文档结构。每次搜索操作会从一个起始元素开始进行。
同样，每次操作所指定的标签名也是起始元素的相对路径。
例如，执行 ``doc.iterfind('channel/item')`` 来搜索所有在 ``channel`` 元素下面的 ``item`` 元素。
``doc`` 代表文档的最顶层(也就是第一级的 ``rss`` 元素)。
然后接下来的调用 ``item.findtext()`` 会从已找到的 ``item`` 元素位置开始搜索。

``ElementTree`` 模块中的每个元素有一些重要的属性和方法，在解析的时候非常有用。
``tag`` 属性包含了标签的名字，``text`` 属性包含了内部的文本，而 ``get()`` 方法能获取属性值。例如：

.. code-block:: python

    >>> doc
    <xml.etree.ElementTree.ElementTree object at 0x101339510>
    >>> e = doc.find('channel/title')
    >>> e
    <Element 'title' at 0x10135b310>
    >>> e.tag
    'title'
    >>> e.text
    'Planet Python'
    >>> e.get('some_attribute')
    >>>

有一点要强调的是 ``xml.etree.ElementTree`` 并不是XML解析的唯一方法。
对于更高级的应用程序，你需要考虑使用 ``lxml`` 。
它使用了和ElementTree同样的编程接口，因此上面的例子同样也适用于lxml。
你只需要将刚开始的import语句换成 ``from lxml.etree import parse`` 就行了。
``lxml`` 完全遵循XML标准，并且速度也非常快，同时还支持验证，XSLT，和XPath等特性。
============================
6.4 增量式解析大型XML文件
============================

----------
问题
----------
你想使用尽可能少的内存从一个超大的XML文档中提取数据。

----------
解决方案
----------
任何时候只要你遇到增量式的数据处理时，第一时间就应该想到迭代器和生成器。
下面是一个很简单的函数，只使用很少的内存就能增量式的处理一个大型XML文件：

.. code-block:: python

    from xml.etree.ElementTree import iterparse

    def parse_and_remove(filename, path):
        path_parts = path.split('/')
        doc = iterparse(filename, ('start', 'end'))
        # Skip the root element
        next(doc)

        tag_stack = []
        elem_stack = []
        for event, elem in doc:
            if event == 'start':
                tag_stack.append(elem.tag)
                elem_stack.append(elem)
            elif event == 'end':
                if tag_stack == path_parts:
                    yield elem
                    elem_stack[-2].remove(elem)
                try:
                    tag_stack.pop()
                    elem_stack.pop()
                except IndexError:
                    pass

为了测试这个函数，你需要先有一个大型的XML文件。
通常你可以在政府网站或公共数据网站上找到这样的文件。
例如，你可以下载XML格式的芝加哥城市道路坑洼数据库。
在写这本书的时候，下载文件已经包含超过100,000行数据，编码格式类似于下面这样：

.. code-block:: xml

    <response>
        <row>
            <row ...>
                <creation_date>2012-11-18T00:00:00</creation_date>
                <status>Completed</status>
                <completion_date>2012-11-18T00:00:00</completion_date>
                <service_request_number>12-01906549</service_request_number>
                <type_of_service_request>Pot Hole in Street</type_of_service_request>
                <current_activity>Final Outcome</current_activity>
                <most_recent_action>CDOT Street Cut ... Outcome</most_recent_action>
                <street_address>4714 S TALMAN AVE</street_address>
                <zip>60632</zip>
                <x_coordinate>1159494.68618856</x_coordinate>
                <y_coordinate>1873313.83503384</y_coordinate>
                <ward>14</ward>
                <police_district>9</police_district>
                <community_area>58</community_area>
                <latitude>41.808090232127896</latitude>
                <longitude>-87.69053684711305</longitude>
                <location latitude="41.808090232127896"
                longitude="-87.69053684711305" />
            </row>
            <row ...>
                <creation_date>2012-11-18T00:00:00</creation_date>
                <status>Completed</status>
                <completion_date>2012-11-18T00:00:00</completion_date>
                <service_request_number>12-01906695</service_request_number>
                <type_of_service_request>Pot Hole in Street</type_of_service_request>
                <current_activity>Final Outcome</current_activity>
                <most_recent_action>CDOT Street Cut ... Outcome</most_recent_action>
                <street_address>3510 W NORTH AVE</street_address>
                <zip>60647</zip>
                <x_coordinate>1152732.14127696</x_coordinate>
                <y_coordinate>1910409.38979075</y_coordinate>
                <ward>26</ward>
                <police_district>14</police_district>
                <community_area>23</community_area>
                <latitude>41.91002084292946</latitude>
                <longitude>-87.71435952353961</longitude>
                <location latitude="41.91002084292946"
                longitude="-87.71435952353961" />
            </row>
        </row>
    </response>

假设你想写一个脚本来按照坑洼报告数量排列邮编号码。你可以像这样做：

.. code-block:: python

    from xml.etree.ElementTree import parse
    from collections import Counter

    potholes_by_zip = Counter()

    doc = parse('potholes.xml')
    for pothole in doc.iterfind('row/row'):
        potholes_by_zip[pothole.findtext('zip')] += 1
    for zipcode, num in potholes_by_zip.most_common():
        print(zipcode, num)

这个脚本唯一的问题是它会先将整个XML文件加载到内存中然后解析。
在我的机器上，为了运行这个程序需要用到450MB左右的内存空间。
如果使用如下代码，程序只需要修改一点点：

.. code-block:: python

    from collections import Counter

    potholes_by_zip = Counter()

    data = parse_and_remove('potholes.xml', 'row/row')
    for pothole in data:
        potholes_by_zip[pothole.findtext('zip')] += 1
    for zipcode, num in potholes_by_zip.most_common():
        print(zipcode, num)

结果是：这个版本的代码运行时只需要7MB的内存--大大节约了内存资源。

----------
讨论
----------
这一节的技术会依赖 ``ElementTree`` 模块中的两个核心功能。
第一，``iterparse()`` 方法允许对XML文档进行增量操作。
使用时，你需要提供文件名和一个包含下面一种或多种类型的事件列表：
``start`` , ``end``, ``start-ns`` 和 ``end-ns`` 。
由 ``iterparse()`` 创建的迭代器会产生形如 ``(event, elem)`` 的元组，
其中 ``event`` 是上述事件列表中的某一个，而 ``elem`` 是相应的XML元素。例如：

.. code-block:: python

    >>> data = iterparse('potholes.xml',('start','end'))
    >>> next(data)
    ('start', <Element 'response' at 0x100771d60>)
    >>> next(data)
    ('start', <Element 'row' at 0x100771e68>)
    >>> next(data)
    ('start', <Element 'row' at 0x100771fc8>)
    >>> next(data)
    ('start', <Element 'creation_date' at 0x100771f18>)
    >>> next(data)
    ('end', <Element 'creation_date' at 0x100771f18>)
    >>> next(data)
    ('start', <Element 'status' at 0x1006a7f18>)
    >>> next(data)
    ('end', <Element 'status' at 0x1006a7f18>)
    >>>

``start`` 事件在某个元素第一次被创建并且还没有被插入其他数据(如子元素)时被创建。
而 ``end`` 事件在某个元素已经完成时被创建。
尽管没有在例子中演示， ``start-ns`` 和 ``end-ns`` 事件被用来处理XML文档命名空间的声明。

这本节例子中， ``start`` 和 ``end`` 事件被用来管理元素和标签栈。
栈代表了文档被解析时的层次结构，
还被用来判断某个元素是否匹配传给函数 ``parse_and_remove()`` 的路径。
如果匹配，就利用 ``yield`` 语句向调用者返回这个元素。

在 ``yield`` 之后的下面这个语句才是使得程序占用极少内存的ElementTree的核心特性：

.. code-block:: python

    elem_stack[-2].remove(elem)

这个语句使得之前由 ``yield`` 产生的元素从它的父节点中删除掉。
假设已经没有其它的地方引用这个元素了，那么这个元素就被销毁并回收内存。

对节点的迭代式解析和删除的最终效果就是一个在文档上高效的增量式清扫过程。
文档树结构从始自终没被完整的创建过。尽管如此，还是能通过上述简单的方式来处理这个XML数据。

这种方案的主要缺陷就是它的运行性能了。
我自己测试的结果是，读取整个文档到内存中的版本的运行速度差不多是增量式处理版本的两倍快。
但是它却使用了超过后者60倍的内存。
因此，如果你更关心内存使用量的话，那么增量式的版本完胜。
============================
6.5 将字典转换为XML
============================

----------
问题
----------
你想使用一个Python字典存储数据，并将它转换成XML格式。

----------
解决方案
----------
尽管 ``xml.etree.ElementTree`` 库通常用来做解析工作，其实它也可以创建XML文档。
例如，考虑如下这个函数：

.. code-block:: python

    from xml.etree.ElementTree import Element

    def dict_to_xml(tag, d):
    '''
    Turn a simple dict of key/value pairs into XML
    '''
    elem = Element(tag)
    for key, val in d.items():
        child = Element(key)
        child.text = str(val)
        elem.append(child)
    return elem

下面是一个使用例子：

.. code-block:: python

    >>> s = { 'name': 'GOOG', 'shares': 100, 'price':490.1 }
    >>> e = dict_to_xml('stock', s)
    >>> e
    <Element 'stock' at 0x1004b64c8>
    >>>

转换结果是一个 ``Element`` 实例。对于I/O操作，使用 ``xml.etree.ElementTree`` 中的 ``tostring()``
函数很容易就能将它转换成一个字节字符串。例如：

.. code-block:: python

    >>> from xml.etree.ElementTree import tostring
    >>> tostring(e)
    b'<stock><price>490.1</price><shares>100</shares><name>GOOG</name></stock>'
    >>>

如果你想给某个元素添加属性值，可以使用 ``set()`` 方法：

.. code-block:: python

    >>> e.set('_id','1234')
    >>> tostring(e)
    b'<stock _id="1234"><price>490.1</price><shares>100</shares><name>GOOG</name>
    </stock>'
    >>>

如果你还想保持元素的顺序，可以考虑构造一个 ``OrderedDict`` 来代替一个普通的字典。请参考1.7小节。

----------
讨论
----------
当创建XML的时候，你被限制只能构造字符串类型的值。例如：

.. code-block:: python

    def dict_to_xml_str(tag, d):
        '''
        Turn a simple dict of key/value pairs into XML
        '''
        parts = ['<{}>'.format(tag)]
        for key, val in d.items():
            parts.append('<{0}>{1}</{0}>'.format(key,val))
        parts.append('</{}>'.format(tag))
        return ''.join(parts)

问题是如果你手动的去构造的时候可能会碰到一些麻烦。例如，当字典的值中包含一些特殊字符的时候会怎样呢？

.. code-block:: python

    >>> d = { 'name' : '<spam>' }

    >>> # String creation
    >>> dict_to_xml_str('item',d)
    '<item><name><spam></name></item>'

    >>> # Proper XML creation
    >>> e = dict_to_xml('item',d)
    >>> tostring(e)
    b'<item><name>&lt;spam&gt;</name></item>'
    >>>

注意到程序的后面那个例子中，字符 '<' 和 '>' 被替换成了 ``&lt;`` 和 ``&gt;``

下面仅供参考，如果你需要手动去转换这些字符，
可以使用 ``xml.sax.saxutils`` 中的 ``escape()``  和 ``unescape()`` 函数。例如：

.. code-block:: python

    >>> from xml.sax.saxutils import escape, unescape
    >>> escape('<spam>')
    '&lt;spam&gt;'
    >>> unescape(_)
    '<spam>'
    >>>

除了能创建正确的输出外，还有另外一个原因推荐你创建 ``Element`` 实例而不是字符串，
那就是使用字符串组合构造一个更大的文档并不是那么容易。
而 ``Element`` 实例可以不用考虑解析XML文本的情况下通过多种方式被处理。
也就是说，你可以在一个高级数据结构上完成你所有的操作，并在最后以字符串的形式将其输出。
============================
6.6 解析和修改XML
============================

----------
问题
----------
你想读取一个XML文档，对它最一些修改，然后将结果写回XML文档。

----------
解决方案
----------
使用 ``xml.etree.ElementTree`` 模块可以很容易的处理这些任务。
第一步是以通常的方式来解析这个文档。例如，假设你有一个名为 ``pred.xml`` 的文档，类似下面这样：

.. code-block:: xml

    <?xml version="1.0"?>
    <stop>
        <id>14791</id>
        <nm>Clark &amp; Balmoral</nm>
        <sri>
            <rt>22</rt>
            <d>North Bound</d>
            <dd>North Bound</dd>
        </sri>
        <cr>22</cr>
        <pre>
            <pt>5 MIN</pt>
            <fd>Howard</fd>
            <v>1378</v>
            <rn>22</rn>
        </pre>
        <pre>
            <pt>15 MIN</pt>
            <fd>Howard</fd>
            <v>1867</v>
            <rn>22</rn>
        </pre>
    </stop>

下面是一个利用 ``ElementTree`` 来读取这个文档并对它做一些修改的例子：

.. code-block:: python

    >>> from xml.etree.ElementTree import parse, Element
    >>> doc = parse('pred.xml')
    >>> root = doc.getroot()
    >>> root
    <Element 'stop' at 0x100770cb0>

    >>> # Remove a few elements
    >>> root.remove(root.find('sri'))
    >>> root.remove(root.find('cr'))
    >>> # Insert a new element after <nm>...</nm>
    >>> root.getchildren().index(root.find('nm'))
    1
    >>> e = Element('spam')
    >>> e.text = 'This is a test'
    >>> root.insert(2, e)

    >>> # Write back to a file
    >>> doc.write('newpred.xml', xml_declaration=True)
    >>>

处理结果是一个像下面这样新的XML文件：

.. code-block:: xml

    <?xml version='1.0' encoding='us-ascii'?>
    <stop>
        <id>14791</id>
        <nm>Clark &amp; Balmoral</nm>
        <spam>This is a test</spam>
        <pre>
            <pt>5 MIN</pt>
            <fd>Howard</fd>
            <v>1378</v>
            <rn>22</rn>
        </pre>
        <pre>
            <pt>15 MIN</pt>
            <fd>Howard</fd>
            <v>1867</v>
            <rn>22</rn>
        </pre>
    </stop>

----------
讨论
----------
修改一个XML文档结构是很容易的，但是你必须牢记的是所有的修改都是针对父节点元素，
将它作为一个列表来处理。例如，如果你删除某个元素，通过调用父节点的 ``remove()`` 方法从它的直接父节点中删除。
如果你插入或增加新的元素，你同样使用父节点元素的 ``insert()`` 和 ``append()`` 方法。
还能对元素使用索引和切片操作，比如 ``element[i]`` 或 ``element[i:j]``

如果你需要创建新的元素，可以使用本节方案中演示的 ``Element`` 类。我们在6.5小节已经详细讨论过了。
============================
6.7 利用命名空间解析XML文档
============================

----------
问题
----------
你想解析某个XML文档，文档中使用了XML命名空间。

----------
解决方案
----------
考虑下面这个使用了命名空间的文档：

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <top>
        <author>David Beazley</author>
        <content>
            <html xmlns="http://www.w3.org/1999/xhtml">
                <head>
                    <title>Hello World</title>
                </head>
                <body>
                    <h1>Hello World!</h1>
                </body>
            </html>
        </content>
    </top>

如果你解析这个文档并执行普通的查询，你会发现这个并不是那么容易，因为所有步骤都变得相当的繁琐。

.. code-block:: python

    >>> # Some queries that work
    >>> doc.findtext('author')
    'David Beazley'
    >>> doc.find('content')
    <Element 'content' at 0x100776ec0>
    >>> # A query involving a namespace (doesn't work)
    >>> doc.find('content/html')
    >>> # Works if fully qualified
    >>> doc.find('content/{http://www.w3.org/1999/xhtml}html')
    <Element '{http://www.w3.org/1999/xhtml}html' at 0x1007767e0>
    >>> # Doesn't work
    >>> doc.findtext('content/{http://www.w3.org/1999/xhtml}html/head/title')
    >>> # Fully qualified
    >>> doc.findtext('content/{http://www.w3.org/1999/xhtml}html/'
    ... '{http://www.w3.org/1999/xhtml}head/{http://www.w3.org/1999/xhtml}title')
    'Hello World'
    >>>

你可以通过将命名空间处理逻辑包装为一个工具类来简化这个过程：

.. code-block:: python

    class XMLNamespaces:
        def __init__(self, **kwargs):
            self.namespaces = {}
            for name, uri in kwargs.items():
                self.register(name, uri)
        def register(self, name, uri):
            self.namespaces[name] = '{'+uri+'}'
        def __call__(self, path):
            return path.format_map(self.namespaces)

通过下面的方式使用这个类：

.. code-block:: python

    >>> ns = XMLNamespaces(html='http://www.w3.org/1999/xhtml')
    >>> doc.find(ns('content/{html}html'))
    <Element '{http://www.w3.org/1999/xhtml}html' at 0x1007767e0>
    >>> doc.findtext(ns('content/{html}html/{html}head/{html}title'))
    'Hello World'
    >>>

----------
讨论
----------
解析含有命名空间的XML文档会比较繁琐。
上面的 ``XMLNamespaces`` 仅仅是允许你使用缩略名代替完整的URI将其变得稍微简洁一点。

很不幸的是，在基本的 ``ElementTree`` 解析中没有任何途径获取命名空间的信息。
但是，如果你使用 ``iterparse()`` 函数的话就可以获取更多关于命名空间处理范围的信息。例如：

.. code-block:: python

    >>> from xml.etree.ElementTree import iterparse
    >>> for evt, elem in iterparse('ns2.xml', ('end', 'start-ns', 'end-ns')):
    ... print(evt, elem)
    ...
    end <Element 'author' at 0x10110de10>
    start-ns ('', 'http://www.w3.org/1999/xhtml')
    end <Element '{http://www.w3.org/1999/xhtml}title' at 0x1011131b0>
    end <Element '{http://www.w3.org/1999/xhtml}head' at 0x1011130a8>
    end <Element '{http://www.w3.org/1999/xhtml}h1' at 0x101113310>
    end <Element '{http://www.w3.org/1999/xhtml}body' at 0x101113260>
    end <Element '{http://www.w3.org/1999/xhtml}html' at 0x10110df70>
    end-ns None
    end <Element 'content' at 0x10110de68>
    end <Element 'top' at 0x10110dd60>
    >>> elem # This is the topmost element
    <Element 'top' at 0x10110dd60>
    >>>

最后一点，如果你要处理的XML文本除了要使用到其他高级XML特性外，还要使用到命名空间，
建议你最好是使用 ``lxml`` 函数库来代替 ``ElementTree`` 。
例如，``lxml`` 对利用DTD验证文档、更好的XPath支持和一些其他高级XML特性等都提供了更好的支持。
这一小节其实只是教你如何让XML解析稍微简单一点。

============================
6.8 与关系型数据库的交互
============================

----------
问题
----------
你想在关系型数据库中查询、增加或删除记录。

----------
解决方案
----------
Python中表示多行数据的标准方式是一个由元组构成的序列。例如：

.. code-block:: python

    stocks = [
        ('GOOG', 100, 490.1),
        ('AAPL', 50, 545.75),
        ('FB', 150, 7.45),
        ('HPQ', 75, 33.2),
    ]

依据PEP249，通过这种形式提供数据，
可以很容易的使用Python标准数据库API和关系型数据库进行交互。
所有数据库上的操作都通过SQL查询语句来完成。每一行输入输出数据用一个元组来表示。

为了演示说明，你可以使用Python标准库中的 ``sqlite3`` 模块。
如果你使用的是一个不同的数据库(比如MySql、Postgresql或者ODBC)，
还得安装相应的第三方模块来提供支持。
不过相应的编程接口几乎都是一样的，除了一点点细微差别外。

第一步是连接到数据库。通常你要执行 ``connect()`` 函数，
给它提供一些数据库名、主机、用户名、密码和其他必要的一些参数。例如：

.. code-block:: python

    >>> import sqlite3
    >>> db = sqlite3.connect('database.db')
    >>>

为了处理数据，下一步你需要创建一个游标。
一旦你有了游标，那么你就可以执行SQL查询语句了。比如：

.. code-block:: python

    >>> c = db.cursor()
    >>> c.execute('create table portfolio (symbol text, shares integer, price real)')
    <sqlite3.Cursor object at 0x10067a730>
    >>> db.commit()
    >>>

为了向数据库表中插入多条记录，使用类似下面这样的语句：

.. code-block:: python

    >>> c.executemany('insert into portfolio values (?,?,?)', stocks)
    <sqlite3.Cursor object at 0x10067a730>
    >>> db.commit()
    >>>

为了执行某个查询，使用像下面这样的语句：

.. code-block:: python

    >>> for row in db.execute('select * from portfolio'):
    ...     print(row)
    ...
    ('GOOG', 100, 490.1)
    ('AAPL', 50, 545.75)
    ('FB', 150, 7.45)
    ('HPQ', 75, 33.2)
    >>>

如果你想接受用户输入作为参数来执行查询操作，必须确保你使用下面这样的占位符``?``来进行引用参数：

.. code-block:: python

    >>> min_price = 100
    >>> for row in db.execute('select * from portfolio where price >= ?',
                              (min_price,)):
    ...     print(row)
    ...
    ('GOOG', 100, 490.1)
    ('AAPL', 50, 545.75)
    >>>

----------
讨论
----------
在比较低的级别上和数据库交互是非常简单的。
你只需提供SQL语句并调用相应的模块就可以更新或提取数据了。
虽说如此，还是有一些比较棘手的细节问题需要你逐个列出去解决。

一个难点是数据库中的数据和Python类型直接的映射。
对于日期类型，通常可以使用 ``datetime`` 模块中的 ``datetime`` 实例，
或者可能是 ``time`` 模块中的系统时间戳。
对于数字类型，特别是使用到小数的金融数据，可以用 ``decimal`` 模块中的 ``Decimal`` 实例来表示。
不幸的是，对于不同的数据库而言具体映射规则是不一样的，你必须参考相应的文档。

另外一个更加复杂的问题就是SQL语句字符串的构造。
你千万不要使用Python字符串格式化操作符(如%)或者 ``.format()`` 方法来创建这样的字符串。
如果传递给这些格式化操作符的值来自于用户的输入，那么你的程序就很有可能遭受SQL注入攻击(参考 http://xkcd.com/327 )。
查询语句中的通配符 ``?`` 指示后台数据库使用它自己的字符串替换机制，这样更加的安全。

不幸的是，不同的数据库后台对于通配符的使用是不一样的。大部分模块使用 ``?`` 或 ``%s`` ，
还有其他一些使用了不同的符号，比如:0或:1来指示参数。
同样的，你还是得去参考你使用的数据库模块相应的文档。
一个数据库模块的 ``paramstyle`` 属性包含了参数引用风格的信息。

对于简单的数据库数据的读写问题，使用数据库API通常非常简单。
如果你要处理更加复杂的问题，建议你使用更加高级的接口，比如一个对象关系映射ORM所提供的接口。
类似 ``SQLAlchemy`` 这样的库允许你使用Python类来表示一个数据库表，
并且能在隐藏底层SQL的情况下实现各种数据库的操作。
============================
6.9 编码和解码十六进制数
============================

----------
问题
----------
你想将一个十六进制字符串解码成一个字节字符串或者将一个字节字符串编码成一个十六进制字符串。

----------
解决方案
----------
如果你只是简单的解码或编码一个十六进制的原始字符串，可以使用　``binascii`` 模块。例如：

.. code-block:: python

    >>> # Initial byte string
    >>> s = b'hello'
    >>> # Encode as hex
    >>> import binascii
    >>> h = binascii.b2a_hex(s)
    >>> h
    b'68656c6c6f'
    >>> # Decode back to bytes
    >>> binascii.a2b_hex(h)
    b'hello'
    >>>

类似的功能同样可以在 ``base64`` 模块中找到。例如：

.. code-block:: python

    >>> import base64
    >>> h = base64.b16encode(s)
    >>> h
    b'68656C6C6F'
    >>> base64.b16decode(h)
    b'hello'
    >>>

----------
讨论
----------
大部分情况下，通过使用上述的函数来转换十六进制是很简单的。
上面两种技术的主要不同在于大小写的处理。
函数 ``base64.b16decode()`` 和 ``base64.b16encode()`` 只能操作大写形式的十六进制字母，
而 ``binascii`` 模块中的函数大小写都能处理。

还有一点需要注意的是编码函数所产生的输出总是一个字节字符串。
如果想强制以Unicode形式输出，你需要增加一个额外的界面步骤。例如：

.. code-block:: python

    >>> h = base64.b16encode(s)
    >>> print(h)
    b'68656C6C6F'
    >>> print(h.decode('ascii'))
    68656C6C6F
    >>>

在解码十六进制数时，函数 ``b16decode()`` 和 ``a2b_hex()`` 可以接受字节或unicode字符串。
但是，unicode字符串必须仅仅只包含ASCII编码的十六进制数。
============================
6.10 编码解码Base64数据
============================

----------
问题
----------
你需要使用Base64格式解码或编码二进制数据。

----------
解决方案
----------
``base64`` 模块中有两个函数 ``b64encode()`` and ``b64decode()`` 可以帮你解决这个问题。例如;

.. code-block:: python

    >>> # Some byte data
    >>> s = b'hello'
    >>> import base64

    >>> # Encode as Base64
    >>> a = base64.b64encode(s)
    >>> a
    b'aGVsbG8='

    >>> # Decode from Base64
    >>> base64.b64decode(a)
    b'hello'
    >>>

----------
讨论
----------
Base64编码仅仅用于面向字节的数据比如字节字符串和字节数组。
此外，编码处理的输出结果总是一个字节字符串。
如果你想混合使用Base64编码的数据和Unicode文本，你必须添加一个额外的解码步骤。例如：

.. code-block:: python

    >>> a = base64.b64encode(s).decode('ascii')
    >>> a
    'aGVsbG8='
    >>>

当解码Base64的时候，字节字符串和Unicode文本都可以作为参数。
但是，Unicode字符串只能包含ASCII字符。
============================
6.11 读写二进制数组数据
============================

----------
问题
----------
你想读写一个二进制数组的结构化数据到Python元组中。

----------
解决方案
----------
可以使用 ``struct`` 模块处理二进制数据。
下面是一段示例代码将一个Python元组列表写入一个二进制文件，并使用 ``struct`` 将每个元组编码为一个结构体。

.. code-block:: python

    from struct import Struct
    def write_records(records, format, f):
        '''
        Write a sequence of tuples to a binary file of structures.
        '''
        record_struct = Struct(format)
        for r in records:
            f.write(record_struct.pack(*r))

    # Example
    if __name__ == '__main__':
        records = [ (1, 2.3, 4.5),
                    (6, 7.8, 9.0),
                    (12, 13.4, 56.7) ]
        with open('data.b', 'wb') as f:
            write_records(records, '<idd', f)

有很多种方法来读取这个文件并返回一个元组列表。
首先，如果你打算以块的形式增量读取文件，你可以这样做：

.. code-block:: python

    from struct import Struct

    def read_records(format, f):
        record_struct = Struct(format)
        chunks = iter(lambda: f.read(record_struct.size), b'')
        return (record_struct.unpack(chunk) for chunk in chunks)

    # Example
    if __name__ == '__main__':
        with open('data.b','rb') as f:
            for rec in read_records('<idd', f):
                # Process rec
                ...

如果你想将整个文件一次性读取到一个字节字符串中，然后在分片解析。那么你可以这样做：

.. code-block:: python

    from struct import Struct

    def unpack_records(format, data):
        record_struct = Struct(format)
        return (record_struct.unpack_from(data, offset)
                for offset in range(0, len(data), record_struct.size))

    # Example
    if __name__ == '__main__':
        with open('data.b', 'rb') as f:
            data = f.read()
        for rec in unpack_records('<idd', data):
            # Process rec
            ...

两种情况下的结果都是一个可返回用来创建该文件的原始元组的可迭代对象。

----------
讨论
----------
对于需要编码和解码二进制数据的程序而言，通常会使用 ``struct`` 模块。
为了声明一个新的结构体，只需要像这样创建一个 ``Struct`` 实例即可：

.. code-block:: python

    # Little endian 32-bit integer, two double precision floats
    record_struct = Struct('<idd')

结构体通常会使用一些结构码值i, d, f等
[参考 `Python文档 <https://docs.python.org/3/library/struct.html>`_ ]。
这些代码分别代表某个特定的二进制数据类型如32位整数，64位浮点数，32位浮点数等。
第一个字符 ``<`` 指定了字节顺序。在这个例子中，它表示"低位在前"。
更改这个字符为 ``>`` 表示高位在前，或者是 ``!`` 表示网络字节顺序。

产生的 ``Struct`` 实例有很多属性和方法用来操作相应类型的结构。
``size`` 属性包含了结构的字节数，这在I/O操作时非常有用。
``pack()`` 和 ``unpack()`` 方法被用来打包和解包数据。比如：

.. code-block:: python

    >>> from struct import Struct
    >>> record_struct = Struct('<idd')
    >>> record_struct.size
    20
    >>> record_struct.pack(1, 2.0, 3.0)
    b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00@\x00\x00\x00\x00\x00\x00\x08@'
    >>> record_struct.unpack(_)
    (1, 2.0, 3.0)
    >>>

有时候你还会看到 ``pack()`` 和 ``unpack()`` 操作以模块级别函数被调用，类似下面这样：

.. code-block:: python

    >>> import struct
    >>> struct.pack('<idd', 1, 2.0, 3.0)
    b'\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00@\x00\x00\x00\x00\x00\x00\x08@'
    >>> struct.unpack('<idd', _)
    (1, 2.0, 3.0)
    >>>

这样可以工作，但是感觉没有实例方法那么优雅，特别是在你代码中同样的结构出现在多个地方的时候。
通过创建一个 ``Struct`` 实例，格式代码只会指定一次并且所有的操作被集中处理。
这样一来代码维护就变得更加简单了(因为你只需要改变一处代码即可)。

读取二进制结构的代码要用到一些非常有趣而优美的编程技巧。
在函数　``read_records`` 中，``iter()`` 被用来创建一个返回固定大小数据块的迭代器，参考5.8小节。
这个迭代器会不断的调用一个用户提供的可调用对象(比如 ``lambda: f.read(record_struct.size)`` )，
直到它返回一个特殊的值(如b'')，这时候迭代停止。例如：

.. code-block:: python

    >>> f = open('data.b', 'rb')
    >>> chunks = iter(lambda: f.read(20), b'')
    >>> chunks
    <callable_iterator object at 0x10069e6d0>
    >>> for chk in chunks:
    ... print(chk)
    ...
    b'\x01\x00\x00\x00ffffff\x02@\x00\x00\x00\x00\x00\x00\x12@'
    b'\x06\x00\x00\x00333333\x1f@\x00\x00\x00\x00\x00\x00"@'
    b'\x0c\x00\x00\x00\xcd\xcc\xcc\xcc\xcc\xcc*@\x9a\x99\x99\x99\x99YL@'
    >>>

如你所见，创建一个可迭代对象的一个原因是它能允许使用一个生成器推导来创建记录。
如果你不使用这种技术，那么代码可能会像下面这样：

.. code-block:: python

    def read_records(format, f):
        record_struct = Struct(format)
        while True:
            chk = f.read(record_struct.size)
            if chk == b'':
                break
            yield record_struct.unpack(chk)

在函数 ``unpack_records()`` 中使用了另外一种方法 ``unpack_from()`` 。
``unpack_from()`` 对于从一个大型二进制数组中提取二进制数据非常有用，
因为它不会产生任何的临时对象或者进行内存复制操作。
你只需要给它一个字节字符串(或数组)和一个字节偏移量，它会从那个位置开始直接解包数据。

如果你使用 ``unpack()`` 来代替 ``unpack_from()`` ，
你需要修改代码来构造大量的小的切片以及进行偏移量的计算。比如：

.. code-block:: python

    def unpack_records(format, data):
        record_struct = Struct(format)
        return (record_struct.unpack(data[offset:offset + record_struct.size])
                for offset in range(0, len(data), record_struct.size))

这种方案除了代码看上去很复杂外，还得做很多额外的工作，因为它执行了大量的偏移量计算，
复制数据以及构造小的切片对象。
如果你准备从读取到的一个大型字节字符串中解包大量的结构体的话，``unpack_from()`` 会表现的更出色。

在解包的时候，``collections`` 模块中的命名元组对象或许是你想要用到的。
它可以让你给返回元组设置属性名称。例如：

.. code-block:: python

    from collections import namedtuple

    Record = namedtuple('Record', ['kind','x','y'])

    with open('data.p', 'rb') as f:
        records = (Record(*r) for r in read_records('<idd', f))

    for r in records:
        print(r.kind, r.x, r.y)

如果你的程序需要处理大量的二进制数据，你最好使用 ``numpy`` 模块。
例如，你可以将一个二进制数据读取到一个结构化数组中而不是一个元组列表中。就像下面这样：

.. code-block:: python

    >>> import numpy as np
    >>> f = open('data.b', 'rb')
    >>> records = np.fromfile(f, dtype='<i,<d,<d')
    >>> records
    array([(1, 2.3, 4.5), (6, 7.8, 9.0), (12, 13.4, 56.7)],
    dtype=[('f0', '<i4'), ('f1', '<f8'), ('f2', '<f8')])
    >>> records[0]
    (1, 2.3, 4.5)
    >>> records[1]
    (6, 7.8, 9.0)
    >>>

最后提一点，如果你需要从已知的文件格式(如图片格式，图形文件，HDF5等)中读取二进制数据时，
先检查看看Python是不是已经提供了现存的模块。因为不到万不得已没有必要去重复造轮子。
============================
6.12 读取嵌套和可变长二进制数据
============================

----------
问题
----------
你需要读取包含嵌套或者可变长记录集合的复杂二进制格式的数据。这些数据可能包含图片、视频、电子地图文件等。

----------
解决方案
----------
``struct`` 模块可被用来编码/解码几乎所有类型的二进制的数据结构。为了解释清楚这种数据，假设你用下面的Python数据结构
来表示一个组成一系列多边形的点的集合：

.. code-block:: python

    polys = [
        [ (1.0, 2.5), (3.5, 4.0), (2.5, 1.5) ],
        [ (7.0, 1.2), (5.1, 3.0), (0.5, 7.5), (0.8, 9.0) ],
        [ (3.4, 6.3), (1.2, 0.5), (4.6, 9.2) ],
    ]

现在假设这个数据被编码到一个以下列头部开始的二进制文件中去了：

.. code-block:: python

    +------+--------+------------------------------------+
    |Byte  | Type   |  Description                       |
    +======+========+====================================+
    |0     | int    |  文件代码（0x1234，小端）          |
    +------+--------+------------------------------------+
    |4     | double |  x 的最小值（小端）                |
    +------+--------+------------------------------------+
    |12    | double |  y 的最小值（小端）                |
    +------+--------+------------------------------------+
    |20    | double |  x 的最大值（小端）                |
    +------+--------+------------------------------------+
    |28    | double |  y 的最大值（小端）                |
    +------+--------+------------------------------------+
    |36    | int    |  三角形数量（小端）                |
    +------+--------+------------------------------------+

紧跟着头部是一系列的多边形记录，编码格式如下：

.. code-block:: python

    +------+--------+-------------------------------------------+
    |Byte  | Type   |  Description                              |
    +======+========+===========================================+
    |0     | int    |  记录长度（N字节）                        |
    +------+--------+-------------------------------------------+
    |4-N   | Points |  (X,Y) 坐标，以浮点数表示                 |
    +------+--------+-------------------------------------------+

为了写这样的文件，你可以使用如下的Python代码：

.. code-block:: python

    import struct
    import itertools

    def write_polys(filename, polys):
        # Determine bounding box
        flattened = list(itertools.chain(*polys))
        min_x = min(x for x, y in flattened)
        max_x = max(x for x, y in flattened)
        min_y = min(y for x, y in flattened)
        max_y = max(y for x, y in flattened)
        with open(filename, 'wb') as f:
            f.write(struct.pack('<iddddi', 0x1234,
                                min_x, min_y,
                                max_x, max_y,
                                len(polys)))
            for poly in polys:
                size = len(poly) * struct.calcsize('<dd')
                f.write(struct.pack('<i', size + 4))
                for pt in poly:
                    f.write(struct.pack('<dd', *pt))

将数据读取回来的时候，可以利用函数 ``struct.unpack()`` ，代码很相似，基本就是上面写操作的逆序。如下：

.. code-block:: python

    def read_polys(filename):
        with open(filename, 'rb') as f:
            # Read the header
            header = f.read(40)
            file_code, min_x, min_y, max_x, max_y, num_polys = \
                struct.unpack('<iddddi', header)
            polys = []
            for n in range(num_polys):
                pbytes, = struct.unpack('<i', f.read(4))
                poly = []
                for m in range(pbytes // 16):
                    pt = struct.unpack('<dd', f.read(16))
                    poly.append(pt)
                polys.append(poly)
        return polys

尽管这个代码可以工作，但是里面混杂了很多读取、解包数据结构和其他细节的代码。如果用这样的代码来处理真实的数据文件，
那未免也太繁杂了点。因此很显然应该有另一种解决方法可以简化这些步骤，让程序员只关注自最重要的事情。

在本小节接下来的部分，我会逐步演示一个更加优秀的解析字节数据的方案。
目标是可以给程序员提供一个高级的文件格式化方法，并简化读取和解包数据的细节。但是我要先提醒你，
本小节接下来的部分代码应该是整本书中最复杂最高级的例子，使用了大量的面向对象编程和元编程技术。
一定要仔细的阅读我们的讨论部分，另外也要参考下其他章节内容。

首先，当读取字节数据的时候，通常在文件开始部分会包含文件头和其他的数据结构。
尽管struct模块可以解包这些数据到一个元组中去，另外一种表示这种信息的方式就是使用一个类。
就像下面这样：

.. code-block:: python

    import struct

    class StructField:
        '''
        Descriptor representing a simple structure field
        '''
        def __init__(self, format, offset):
            self.format = format
            self.offset = offset
        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                r = struct.unpack_from(self.format, instance._buffer, self.offset)
                return r[0] if len(r) == 1 else r

    class Structure:
        def __init__(self, bytedata):
            self._buffer = memoryview(bytedata)

这里我们使用了一个描述器来表示每个结构字段，每个描述器包含一个结构兼容格式的代码以及一个字节偏移量，
存储在内部的内存缓冲中。在 ``__get__()`` 方法中，``struct.unpack_from()``
函数被用来从缓冲中解包一个值，省去了额外的分片或复制操作步骤。

``Structure`` 类就是一个基础类，接受字节数据并存储在内部的内存缓冲中，并被 ``StructField`` 描述器使用。
这里使用了 ``memoryview()`` ，我们会在后面详细讲解它是用来干嘛的。

使用这个代码，你现在就能定义一个高层次的结构对象来表示上面表格信息所期望的文件格式。例如：

.. code-block:: python

    class PolyHeader(Structure):
        file_code = StructField('<i', 0)
        min_x = StructField('<d', 4)
        min_y = StructField('<d', 12)
        max_x = StructField('<d', 20)
        max_y = StructField('<d', 28)
        num_polys = StructField('<i', 36)

下面的例子利用这个类来读取之前我们写入的多边形数据的头部数据：

.. code-block:: python

    >>> f = open('polys.bin', 'rb')
    >>> phead = PolyHeader(f.read(40))
    >>> phead.file_code == 0x1234
    True
    >>> phead.min_x
    0.5
    >>> phead.min_y
    0.5
    >>> phead.max_x
    7.0
    >>> phead.max_y
    9.2
    >>> phead.num_polys
    3
    >>>

这个很有趣，不过这种方式还是有一些烦人的地方。首先，尽管你获得了一个类接口的便利，
但是这个代码还是有点臃肿，还需要使用者指定很多底层的细节(比如重复使用 ``StructField`` ，指定偏移量等)。
另外，返回的结果类同样确实一些便利的方法来计算结构的总数。

任何时候只要你遇到了像这样冗余的类定义，你应该考虑下使用类装饰器或元类。
元类有一个特性就是它能够被用来填充许多低层的实现细节，从而释放使用者的负担。
下面我来举个例子，使用元类稍微改造下我们的 ``Structure`` 类：

.. code-block:: python

    class StructureMeta(type):
        '''
        Metaclass that automatically creates StructField descriptors
        '''
        def __init__(self, clsname, bases, clsdict):
            fields = getattr(self, '_fields_', [])
            byte_order = ''
            offset = 0
            for format, fieldname in fields:
                if format.startswith(('<','>','!','@')):
                    byte_order = format[0]
                    format = format[1:]
                format = byte_order + format
                setattr(self, fieldname, StructField(format, offset))
                offset += struct.calcsize(format)
            setattr(self, 'struct_size', offset)

    class Structure(metaclass=StructureMeta):
        def __init__(self, bytedata):
            self._buffer = bytedata

        @classmethod
        def from_file(cls, f):
            return cls(f.read(cls.struct_size))

使用新的 ``Structure`` 类，你可以像下面这样定义一个结构：

.. code-block:: python

    class PolyHeader(Structure):
        _fields_ = [
            ('<i', 'file_code'),
            ('d', 'min_x'),
            ('d', 'min_y'),
            ('d', 'max_x'),
            ('d', 'max_y'),
            ('i', 'num_polys')
        ]

正如你所见，这样写就简单多了。我们添加的类方法 ``from_file()``
让我们在不需要知道任何数据的大小和结构的情况下就能轻松的从文件中读取数据。比如：

.. code-block:: python

    >>> f = open('polys.bin', 'rb')
    >>> phead = PolyHeader.from_file(f)
    >>> phead.file_code == 0x1234
    True
    >>> phead.min_x
    0.5
    >>> phead.min_y
    0.5
    >>> phead.max_x
    7.0
    >>> phead.max_y
    9.2
    >>> phead.num_polys
    3
    >>>

一旦你开始使用了元类，你就可以让它变得更加智能。例如，假设你还想支持嵌套的字节结构，
下面是对前面元类的一个小的改进，提供了一个新的辅助描述器来达到想要的效果：

.. code-block:: python

    class NestedStruct:
        '''
        Descriptor representing a nested structure
        '''
        def __init__(self, name, struct_type, offset):
            self.name = name
            self.struct_type = struct_type
            self.offset = offset

        def __get__(self, instance, cls):
            if instance is None:
                return self
            else:
                data = instance._buffer[self.offset:
                                self.offset+self.struct_type.struct_size]
                result = self.struct_type(data)
                # Save resulting structure back on instance to avoid
                # further recomputation of this step
                setattr(instance, self.name, result)
                return result

    class StructureMeta(type):
        '''
        Metaclass that automatically creates StructField descriptors
        '''
        def __init__(self, clsname, bases, clsdict):
            fields = getattr(self, '_fields_', [])
            byte_order = ''
            offset = 0
            for format, fieldname in fields:
                if isinstance(format, StructureMeta):
                    setattr(self, fieldname,
                            NestedStruct(fieldname, format, offset))
                    offset += format.struct_size
                else:
                    if format.startswith(('<','>','!','@')):
                        byte_order = format[0]
                        format = format[1:]
                    format = byte_order + format
                    setattr(self, fieldname, StructField(format, offset))
                    offset += struct.calcsize(format)
            setattr(self, 'struct_size', offset)

在这段代码中，``NestedStruct`` 描述器被用来叠加另外一个定义在某个内存区域上的结构。
它通过将原始内存缓冲进行切片操作后实例化给定的结构类型。由于底层的内存缓冲区是通过一个内存视图初始化的，
所以这种切片操作不会引发任何的额外的内存复制。相反，它仅仅就是之前的内存的一个叠加而已。
另外，为了防止重复实例化，通过使用和8.10小节同样的技术，描述器保存了该实例中的内部结构对象。

使用这个新的修正版，你就可以像下面这样编写：

.. code-block:: python

    class Point(Structure):
        _fields_ = [
            ('<d', 'x'),
            ('d', 'y')
        ]

    class PolyHeader(Structure):
        _fields_ = [
            ('<i', 'file_code'),
            (Point, 'min'), # nested struct
            (Point, 'max'), # nested struct
            ('i', 'num_polys')
        ]

令人惊讶的是，它也能按照预期的正常工作，我们实际操作下：

.. code-block:: python

    >>> f = open('polys.bin', 'rb')
    >>> phead = PolyHeader.from_file(f)
    >>> phead.file_code == 0x1234
    True
    >>> phead.min # Nested structure
    <__main__.Point object at 0x1006a48d0>
    >>> phead.min.x
    0.5
    >>> phead.min.y
    0.5
    >>> phead.max.x
    7.0
    >>> phead.max.y
    9.2
    >>> phead.num_polys
    3
    >>>

到目前为止，一个处理定长记录的框架已经写好了。但是如果组件记录是变长的呢？
比如，多边形文件包含变长的部分。

一种方案是写一个类来表示字节数据，同时写一个工具函数来通过多少方式解析内容。跟6.11小节的代码很类似：

.. code-block:: python

    class SizedRecord:
        def __init__(self, bytedata):
            self._buffer = memoryview(bytedata)

        @classmethod
        def from_file(cls, f, size_fmt, includes_size=True):
            sz_nbytes = struct.calcsize(size_fmt)
            sz_bytes = f.read(sz_nbytes)
            sz, = struct.unpack(size_fmt, sz_bytes)
            buf = f.read(sz - includes_size * sz_nbytes)
            return cls(buf)

        def iter_as(self, code):
            if isinstance(code, str):
                s = struct.Struct(code)
                for off in range(0, len(self._buffer), s.size):
                    yield s.unpack_from(self._buffer, off)
            elif isinstance(code, StructureMeta):
                size = code.struct_size
                for off in range(0, len(self._buffer), size):
                    data = self._buffer[off:off+size]
                    yield code(data)

类方法 ``SizedRecord.from_file()`` 是一个工具，用来从一个文件中读取带大小前缀的数据块，
这也是很多文件格式常用的方式。作为输入，它接受一个包含大小编码的结构格式编码，并且也是自己形式。
可选的 ``includes_size`` 参数指定了字节数是否包含头部大小。
下面是一个例子教你怎样使用从多边形文件中读取单独的多边形数据：

.. code-block:: python

    >>> f = open('polys.bin', 'rb')
    >>> phead = PolyHeader.from_file(f)
    >>> phead.num_polys
    3
    >>> polydata = [ SizedRecord.from_file(f, '<i')
    ...             for n in range(phead.num_polys) ]
    >>> polydata
    [<__main__.SizedRecord object at 0x1006a4d50>,
    <__main__.SizedRecord object at 0x1006a4f50>,
    <__main__.SizedRecord object at 0x10070da90>]
    >>>

可以看出，``SizedRecord`` 实例的内容还没有被解析出来。
可以使用 ``iter_as()`` 方法来达到目的，这个方法接受一个结构格式化编码或者是 ``Structure`` 类作为输入。
这样子可以很灵活的去解析数据，例如：

.. code-block:: python

    >>> for n, poly in enumerate(polydata):
    ...     print('Polygon', n)
    ...     for p in poly.iter_as('<dd'):
    ...         print(p)
    ...
    Polygon 0
    (1.0, 2.5)
    (3.5, 4.0)
    (2.5, 1.5)
    Polygon 1
    (7.0, 1.2)
    (5.1, 3.0)
    (0.5, 7.5)
    (0.8, 9.0)
    Polygon 2
    (3.4, 6.3)
    (1.2, 0.5)
    (4.6, 9.2)
    >>>

    >>> for n, poly in enumerate(polydata):
    ...     print('Polygon', n)
    ...     for p in poly.iter_as(Point):
    ...         print(p.x, p.y)
    ...
    Polygon 0
    1.0 2.5
    3.5 4.0
    2.5 1.5
    Polygon 1
    7.0 1.2
    5.1 3.0
    0.5 7.5
    0.8 9.0
    Polygon 2
    3.4 6.3
    1.2 0.5
    4.6 9.2
    >>>

将所有这些结合起来，下面是一个 ``read_polys()`` 函数的另外一个修正版：

.. code-block:: python

    class Point(Structure):
        _fields_ = [
            ('<d', 'x'),
            ('d', 'y')
        ]

    class PolyHeader(Structure):
        _fields_ = [
            ('<i', 'file_code'),
            (Point, 'min'),
            (Point, 'max'),
            ('i', 'num_polys')
        ]

    def read_polys(filename):
        polys = []
        with open(filename, 'rb') as f:
            phead = PolyHeader.from_file(f)
            for n in range(phead.num_polys):
                rec = SizedRecord.from_file(f, '<i')
                poly = [ (p.x, p.y) for p in rec.iter_as(Point) ]
                polys.append(poly)
        return polys

----------
讨论
----------
这一节向你展示了许多高级的编程技术，包括描述器，延迟计算，元类，类变量和内存视图。
然而，它们都为了同一个特定的目标服务。

上面的实现的一个主要特征是它是基于懒解包的思想。当一个 ``Structure`` 实例被创建时，
``__init__()`` 仅仅只是创建一个字节数据的内存视图，没有做其他任何事。
特别的，这时候并没有任何的解包或者其他与结构相关的操作发生。
这样做的一个动机是你可能仅仅只对一个字节记录的某一小部分感兴趣。我们只需要解包你需要访问的部分，而不是整个文件。

为了实现懒解包和打包，需要使用 ``StructField`` 描述器类。
用户在 ``_fields_`` 中列出来的每个属性都会被转化成一个 ``StructField`` 描述器，
它将相关结构格式码和偏移值保存到存储缓存中。元类 ``StructureMeta`` 在多个结构类被定义时自动创建了这些描述器。
我们使用元类的一个主要原因是它使得用户非常方便的通过一个高层描述就能指定结构格式，而无需考虑低层的细节问题。

``StructureMeta`` 的一个很微妙的地方就是它会固定字节数据顺序。
也就是说，如果任意的属性指定了一个字节顺序(<表示低位优先 或者 >表示高位优先)，
那后面所有字段的顺序都以这个顺序为准。这么做可以帮助避免额外输入，但是在定义的中间我们仍然可能切换顺序的。
比如，你可能有一些比较复杂的结构，就像下面这样：

.. code-block:: python

    class ShapeFile(Structure):
        _fields_ = [ ('>i', 'file_code'), # Big endian
            ('20s', 'unused'),
            ('i', 'file_length'),
            ('<i', 'version'), # Little endian
            ('i', 'shape_type'),
            ('d', 'min_x'),
            ('d', 'min_y'),
            ('d', 'max_x'),
            ('d', 'max_y'),
            ('d', 'min_z'),
            ('d', 'max_z'),
            ('d', 'min_m'),
            ('d', 'max_m') ]

之前我们提到过，``memoryview()`` 的使用可以帮助我们避免内存的复制。
当结构存在嵌套的时候，``memoryviews`` 可以叠加同一内存区域上定义的机构的不同部分。
这个特性比较微妙，但是它关注的是内存视图与普通字节数组的切片操作行为。
如果你在一个字节字符串或字节数组上执行切片操作，你通常会得到一个数据的拷贝。
而内存视图切片不是这样的，它仅仅是在已存在的内存上面叠加而已。因此，这种方式更加高效。

还有很多相关的章节可以帮助我们扩展这里讨论的方案。
参考8.13小节使用描述器构建一个类型系统。
8.10小节有更多关于延迟计算属性值的讨论，并且跟NestedStruct描述器的实现也有关。
9.19小节有一个使用元类来初始化类成员的例子，和 ``StructureMeta`` 类非常相似。
Python的 ``ctypes`` 源码同样也很有趣，它提供了对定义数据结构、数据结构嵌套这些相似功能的支持。
============================
6.13 数据的累加与统计操作
============================

----------
问题
----------
你需要处理一个很大的数据集并需要计算数据总和或其他统计量。

----------
解决方案
----------
对于任何涉及到统计、时间序列以及其他相关技术的数据分析问题，都可以考虑使用 `Pandas库 <http://pandas.pydata.org/>`_ 。

为了让你先体验下，下面是一个使用Pandas来分析芝加哥城市的
`老鼠和啮齿类动物数据库 <https://data.cityofchicago.org/Service-Requests/311-Service-Requests-Rodent-Baiting/97t6-zrhs>`_ 的例子。
在我写这篇文章的时候，这个数据库是一个拥有大概74,000行数据的CSV文件。

.. code-block:: python

    >>> import pandas

    >>> # Read a CSV file, skipping last line
    >>> rats = pandas.read_csv('rats.csv', skip_footer=1)
    >>> rats
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 74055 entries, 0 to 74054
    Data columns:
    Creation Date 74055 non-null values
    Status 74055 non-null values
    Completion Date 72154 non-null values
    Service Request Number 74055 non-null values
    Type of Service Request 74055 non-null values
    Number of Premises Baited 65804 non-null values
    Number of Premises with Garbage 65600 non-null values
    Number of Premises with Rats 65752 non-null values
    Current Activity 66041 non-null values
    Most Recent Action 66023 non-null values
    Street Address 74055 non-null values
    ZIP Code 73584 non-null values
    X Coordinate 74043 non-null values
    Y Coordinate 74043 non-null values
    Ward 74044 non-null values
    Police District 74044 non-null values
    Community Area 74044 non-null values
    Latitude 74043 non-null values
    Longitude 74043 non-null values
    Location 74043 non-null values
    dtypes: float64(11), object(9)

    >>> # Investigate range of values for a certain field
    >>> rats['Current Activity'].unique()
    array([nan, Dispatch Crew, Request Sanitation Inspector], dtype=object)
    >>> # Filter the data
    >>> crew_dispatched = rats[rats['Current Activity'] == 'Dispatch Crew']
    >>> len(crew_dispatched)
    65676
    >>>

    >>> # Find 10 most rat-infested ZIP codes in Chicago
    >>> crew_dispatched['ZIP Code'].value_counts()[:10]
    60647 3837
    60618 3530
    60614 3284
    60629 3251
    60636 2801
    60657 2465
    60641 2238
    60609 2206
    60651 2152
    60632 2071
    >>>

    >>> # Group by completion date
    >>> dates = crew_dispatched.groupby('Completion Date')
    <pandas.core.groupby.DataFrameGroupBy object at 0x10d0a2a10>
    >>> len(dates)
    472
    >>>

    >>> # Determine counts on each day
    >>> date_counts = dates.size()
    >>> date_counts[0:10]
    Completion Date
    01/03/2011 4
    01/03/2012 125
    01/04/2011 54
    01/04/2012 38
    01/05/2011 78
    01/05/2012 100
    01/06/2011 100
    01/06/2012 58
    01/07/2011 1
    01/09/2012 12
    >>>

    >>> # Sort the counts
    >>> date_counts.sort()
    >>> date_counts[-10:]
    Completion Date
    10/12/2012 313
    10/21/2011 314
    09/20/2011 316
    10/26/2011 319
    02/22/2011 325
    10/26/2012 333
    03/17/2011 336
    10/13/2011 378
    10/14/2011 391
    10/07/2011 457
    >>>
嗯，看样子2011年10月7日对老鼠们来说是个很忙碌的日子啊！^_^

----------
讨论
----------
Pandas是一个拥有很多特性的大型函数库，我在这里不可能介绍完。
但是只要你需要去分析大型数据集合、对数据分组、计算各种统计量或其他类似任务的话，这个函数库真的值得你去看一看。

============================
7.1 可接受任意数量参数的函数
============================

----------
问题
----------
你想构造一个可接受任意数量参数的函数。

----------
解决方案
----------
为了能让一个函数接受任意数量的位置参数，可以使用一个*参数。例如：

.. code-block:: python

    def avg(first, *rest):
        return (first + sum(rest)) / (1 + len(rest))

    # Sample use
    avg(1, 2) # 1.5
    avg(1, 2, 3, 4) # 2.5

在这个例子中，rest是由所有其他位置参数组成的元组。然后我们在代码中把它当成了一个序列来进行后续的计算。

为了接受任意数量的关键字参数，使用一个以**开头的参数。比如：

.. code-block:: python

    import html

    def make_element(name, value, **attrs):
        keyvals = [' %s="%s"' % item for item in attrs.items()]
        attr_str = ''.join(keyvals)
        element = '<{name}{attrs}>{value}</{name}>'.format(
                    name=name,
                    attrs=attr_str,
                    value=html.escape(value))
        return element

    # Example
    # Creates '<item size="large" quantity="6">Albatross</item>'
    make_element('item', 'Albatross', size='large', quantity=6)

    # Creates '<p>&lt;spam&gt;</p>'
    make_element('p', '<spam>')

在这里，attrs是一个包含所有被传入进来的关键字参数的字典。

如果你还希望某个函数能同时接受任意数量的位置参数和关键字参数，可以同时使用*和**。比如：

.. code-block:: python

    def anyargs(*args, **kwargs):
        print(args) # A tuple
        print(kwargs) # A dict

使用这个函数时，所有位置参数会被放到args元组中，所有关键字参数会被放到字典kwargs中。

----------
讨论
----------
一个*参数只能出现在函数定义中最后一个位置参数后面，而 **参数只能出现在最后一个参数。
有一点要注意的是，在*参数后面仍然可以定义其他参数。

.. code-block:: python

    def a(x, *args, y):
        pass

    def b(x, *args, y, **kwargs):
        pass

这种参数就是我们所说的强制关键字参数，在后面7.2小节还会详细讲解到。
============================
7.2 只接受关键字参数的函数
============================

----------
问题
----------
你希望函数的某些参数强制使用关键字参数传递

----------
解决方案
----------
将强制关键字参数放到某个*参数或者单个*后面就能达到这种效果。比如：

.. code-block:: python

    def recv(maxsize, *, block):
        'Receives a message'
        pass

    recv(1024, True) # TypeError
    recv(1024, block=True) # Ok

利用这种技术，我们还能在接受任意多个位置参数的函数中指定关键字参数。比如：

.. code-block:: python

    def minimum(*values, clip=None):
        m = min(values)
        if clip is not None:
            m = clip if clip > m else m
        return m

    minimum(1, 5, 2, -5, 10) # Returns -5
    minimum(1, 5, 2, -5, 10, clip=0) # Returns 0

----------
讨论
----------
很多情况下，使用强制关键字参数会比使用位置参数表意更加清晰，程序也更加具有可读性。
例如，考虑下如下一个函数调用：

.. code-block:: python

    msg = recv(1024, False)

如果调用者对recv函数并不是很熟悉，那他肯定不明白那个False参数到底来干嘛用的。
但是，如果代码变成下面这样子的话就清楚多了：

.. code-block:: python

    msg = recv(1024, block=False)

另外，使用强制关键字参数也会比使用**kwargs参数更好，因为在使用函数help的时候输出也会更容易理解：

.. code-block:: python

    >>> help(recv)
    Help on function recv in module __main__:
    recv(maxsize, *, block)
        Receives a message

强制关键字参数在一些更高级场合同样也很有用。
例如，它们可以被用来在使用*args和**kwargs参数作为输入的函数中插入参数，9.11小节有一个这样的例子。
============================
7.3 给函数参数增加元信息
============================

----------
问题
----------
你写好了一个函数，然后想为这个函数的参数增加一些额外的信息，这样的话其他使用者就能清楚的知道这个函数应该怎么使用。

----------
解决方案
----------
使用函数参数注解是一个很好的办法，它能提示程序员应该怎样正确使用这个函数。
例如，下面有一个被注解了的函数：

.. code-block:: python

    def add(x:int, y:int) -> int:
        return x + y

python解释器不会对这些注解添加任何的语义。它们不会被类型检查，运行时跟没有加注解之前的效果也没有任何差距。
然而，对于那些阅读源码的人来讲就很有帮助啦。第三方工具和框架可能会对这些注解添加语义。同时它们也会出现在文档中。

.. code-block:: python

    >>> help(add)
    Help on function add in module __main__:
    add(x: int, y: int) -> int
    >>>

尽管你可以使用任意类型的对象给函数添加注解(例如数字，字符串，对象实例等等)，不过通常来讲使用类或者字符串会比较好点。

----------
讨论
----------
函数注解只存储在函数的 ``__annotations__`` 属性中。例如：

.. code-block:: python

    >>> add.__annotations__
    {'y': <class 'int'>, 'return': <class 'int'>, 'x': <class 'int'>}

尽管注解的使用方法可能有很多种，但是它们的主要用途还是文档。
因为python并没有类型声明，通常来讲仅仅通过阅读源码很难知道应该传递什么样的参数给这个函数。
这时候使用注解就能给程序员更多的提示，让他们可以正确的使用函数。

参考9.20小节的一个更加高级的例子，演示了如何利用注解来实现多分派(比如重载函数)。
============================
7.4 返回多个值的函数
============================

----------
问题
----------
你希望构造一个可以返回多个值的函数

----------
解决方案
----------
为了能返回多个值，函数直接return一个元组就行了。例如：

.. code-block:: python

    >>> def myfun():
    ... return 1, 2, 3
    ...
    >>> a, b, c = myfun()
    >>> a
    1
    >>> b
    2
    >>> c
    3

----------
讨论
----------
尽管myfun()看上去返回了多个值，实际上是先创建了一个元组然后返回的。
这个语法看上去比较奇怪，实际上我们使用的是逗号来生成一个元组，而不是用括号。比如下面的：

.. code-block:: python

    >>> a = (1, 2) # With parentheses
    >>> a
    (1, 2)
    >>> b = 1, 2 # Without parentheses
    >>> b
    (1, 2)
    >>>

当我们调用返回一个元组的函数的时候 ，通常我们会将结果赋值给多个变量，就像上面的那样。
其实这就是1.1小节中我们所说的元组解包。返回结果也可以赋值给单个变量，
这时候这个变量值就是函数返回的那个元组本身了：

.. code-block:: python

    >>> x = myfun()
    >>> x
    (1, 2, 3)
    >>>
============================
7.5 定义有默认参数的函数
============================

----------
问题
----------
你想定义一个函数或者方法，它的一个或多个参数是可选的并且有一个默认值。

----------
解决方案
----------
定义一个有可选参数的函数是非常简单的，直接在函数定义中给参数指定一个默认值，并放到参数列表最后就行了。例如：

.. code-block:: python

    def spam(a, b=42):
        print(a, b)

    spam(1) # Ok. a=1, b=42
    spam(1, 2) # Ok. a=1, b=2

如果默认参数是一个可修改的容器比如一个列表、集合或者字典，可以使用None作为默认值，就像下面这样：

.. code-block:: python

    # Using a list as a default value
    def spam(a, b=None):
        if b is None:
            b = []
        ...

如果你并不想提供一个默认值，而是想仅仅测试下某个默认参数是不是有传递进来，可以像下面这样写：

.. code-block:: python

    _no_value = object()

    def spam(a, b=_no_value):
        if b is _no_value:
            print('No b value supplied')
        ...

我们测试下这个函数：

.. code-block:: python

    >>> spam(1)
    No b value supplied
    >>> spam(1, 2) # b = 2
    >>> spam(1, None) # b = None
    >>>

仔细观察可以发现到传递一个None值和不传值两种情况是有差别的。

----------
讨论
----------
定义带默认值参数的函数是很简单的，但绝不仅仅只是这个，还有一些东西在这里也深入讨论下。

首先，默认参数的值仅仅在函数定义的时候赋值一次。试着运行下面这个例子：

.. code-block:: python

    >>> x = 42
    >>> def spam(a, b=x):
    ...     print(a, b)
    ...
    >>> spam(1)
    1 42
    >>> x = 23 # Has no effect
    >>> spam(1)
    1 42
    >>>

注意到当我们改变x的值的时候对默认参数值并没有影响，这是因为在函数定义的时候就已经确定了它的默认值了。

其次，默认参数的值应该是不可变的对象，比如None、True、False、数字或字符串。
特别的，千万不要像下面这样写代码：

.. code-block:: python

    def spam(a, b=[]): # NO!
        ...
如果你这么做了，当默认值在其他地方被修改后你将会遇到各种麻烦。这些修改会影响到下次调用这个函数时的默认值。比如：

.. code-block:: python

    >>> def spam(a, b=[]):
    ...     print(b)
    ...     return b
    ...
    >>> x = spam(1)
    >>> x
    []
    >>> x.append(99)
    >>> x.append('Yow!')
    >>> x
    [99, 'Yow!']
    >>> spam(1) # Modified list gets returned!
    [99, 'Yow!']
    >>>

这种结果应该不是你想要的。为了避免这种情况的发生，最好是将默认值设为None，
然后在函数里面检查它，前面的例子就是这样做的。

在测试None值时使用 ``is`` 操作符是很重要的，也是这种方案的关键点。
有时候大家会犯下下面这样的错误：

.. code-block:: python

    def spam(a, b=None):
        if not b: # NO! Use 'b is None' instead
            b = []
        ...

这么写的问题在于尽管None值确实是被当成False，
但是还有其他的对象(比如长度为0的字符串、列表、元组、字典等)都会被当做False。
因此，上面的代码会误将一些其他输入也当成是没有输入。比如：

.. code-block:: python

    >>> spam(1) # OK
    >>> x = []
    >>> spam(1, x) # Silent error. x value overwritten by default
    >>> spam(1, 0) # Silent error. 0 ignored
    >>> spam(1, '') # Silent error. '' ignored
    >>>

最后一个问题比较微妙，那就是一个函数需要测试某个可选参数是否被使用者传递进来。
这时候需要小心的是你不能用某个默认值比如None、
0或者False值来测试用户提供的值(因为这些值都是合法的值，是可能被用户传递进来的)。
因此，你需要其他的解决方案了。

为了解决这个问题，你可以创建一个独一无二的私有对象实例，就像上面的_no_value变量那样。
在函数里面，你可以通过检查被传递参数值跟这个实例是否一样来判断。
这里的思路是用户不可能去传递这个_no_value实例作为输入。
因此，这里通过检查这个值就能确定某个参数是否被传递进来了。

这里对 ``object()`` 的使用看上去有点不太常见。``object`` 是python中所有类的基类。
你可以创建 ``object`` 类的实例，但是这些实例没什么实际用处，因为它并没有任何有用的方法，
也没有任何实例数据(因为它没有任何的实例字典，你甚至都不能设置任何属性值)。
你唯一能做的就是测试同一性。这个刚好符合我的要求，因为我在函数中就只是需要一个同一性的测试而已。
============================
7.6 定义匿名或内联函数
============================

----------
问题
----------
你想为 ``sort()`` 操作创建一个很短的回调函数，但又不想用 ``def`` 去写一个单行函数，
而是希望通过某个快捷方式以内联方式来创建这个函数。

----------
解决方案
----------
当一些函数很简单，仅仅只是计算一个表达式的值的时候，就可以使用lambda表达式来代替了。比如：

.. code-block:: python

    >>> add = lambda x, y: x + y
    >>> add(2,3)
    5
    >>> add('hello', 'world')
    'helloworld'
    >>>

这里使用的lambda表达式跟下面的效果是一样的：

.. code-block:: python

    >>> def add(x, y):
    ...     return x + y
    ...
    >>> add(2,3)
    5
    >>>
lambda表达式典型的使用场景是排序或数据reduce等：

.. code-block:: python

    >>> names = ['David Beazley', 'Brian Jones',
    ...         'Raymond Hettinger', 'Ned Batchelder']
    >>> sorted(names, key=lambda name: name.split()[-1].lower())
    ['Ned Batchelder', 'David Beazley', 'Raymond Hettinger', 'Brian Jones']
    >>>

----------
讨论
----------
尽管lambda表达式允许你定义简单函数，但是它的使用是有限制的。
你只能指定单个表达式，它的值就是最后的返回值。也就是说不能包含其他的语言特性了，
包括多个语句、条件表达式、迭代以及异常处理等等。

你可以不使用lambda表达式就能编写大部分python代码。
但是，当有人编写大量计算表达式值的短小函数或者需要用户提供回调函数的程序的时候，
你就会看到lambda表达式的身影了。
============================
7.7 匿名函数捕获变量值
============================

----------
问题
----------
你用lambda定义了一个匿名函数，并想在定义时捕获到某些变量的值。

----------
解决方案
----------
先看下下面代码的效果：

.. code-block:: python

    >>> x = 10
    >>> a = lambda y: x + y
    >>> x = 20
    >>> b = lambda y: x + y
    >>>
现在我问你，a(10)和b(10)返回的结果是什么？如果你认为结果是20和30，那么你就错了：

.. code-block:: python

    >>> a(10)
    30
    >>> b(10)
    30
    >>>

这其中的奥妙在于lambda表达式中的x是一个自由变量，
在运行时绑定值，而不是定义时就绑定，这跟函数的默认值参数定义是不同的。
因此，在调用这个lambda表达式的时候，x的值是执行时的值。例如：

.. code-block:: python

    >>> x = 15
    >>> a(10)
    25
    >>> x = 3
    >>> a(10)
    13
    >>>

如果你想让某个匿名函数在定义时就捕获到值，可以将那个参数值定义成默认参数即可，就像下面这样：

.. code-block:: python

    >>> x = 10
    >>> a = lambda y, x=x: x + y
    >>> x = 20
    >>> b = lambda y, x=x: x + y
    >>> a(10)
    20
    >>> b(10)
    30
    >>>

----------
讨论
----------
在这里列出来的问题是新手很容易犯的错误，有些新手可能会不恰当的使用lambda表达式。
比如，通过在一个循环或列表推导中创建一个lambda表达式列表，并期望函数能在定义时就记住每次的迭代值。例如：

.. code-block:: python

    >>> funcs = [lambda x: x+n for n in range(5)]
    >>> for f in funcs:
    ... print(f(0))
    ...
    4
    4
    4
    4
    4
    >>>

但是实际效果是运行是n的值为迭代的最后一个值。现在我们用另一种方式修改一下：

.. code-block:: python

    >>> funcs = [lambda x, n=n: x+n for n in range(5)]
    >>> for f in funcs:
    ... print(f(0))
    ...
    0
    1
    2
    3
    4
    >>>

通过使用函数默认值参数形式，lambda函数在定义时就能绑定到值。
============================
7.8 减少可调用对象的参数个数
============================

----------
问题
----------
你有一个被其他python代码使用的callable对象，可能是一个回调函数或者是一个处理器，
但是它的参数太多了，导致调用时出错。

----------
解决方案
----------
如果需要减少某个函数的参数个数，你可以使用 ``functools.partial()`` 。
``partial()`` 函数允许你给一个或多个参数设置固定的值，减少接下来被调用时的参数个数。
为了演示清楚，假设你有下面这样的函数：

.. code-block:: python

    def spam(a, b, c, d):
        print(a, b, c, d)

现在我们使用 ``partial()`` 函数来固定某些参数值：

.. code-block:: python

    >>> from functools import partial
    >>> s1 = partial(spam, 1) # a = 1
    >>> s1(2, 3, 4)
    1 2 3 4
    >>> s1(4, 5, 6)
    1 4 5 6
    >>> s2 = partial(spam, d=42) # d = 42
    >>> s2(1, 2, 3)
    1 2 3 42
    >>> s2(4, 5, 5)
    4 5 5 42
    >>> s3 = partial(spam, 1, 2, d=42) # a = 1, b = 2, d = 42
    >>> s3(3)
    1 2 3 42
    >>> s3(4)
    1 2 4 42
    >>> s3(5)
    1 2 5 42
    >>>

可以看出 ``partial()`` 固定某些参数并返回一个新的callable对象。这个新的callable接受未赋值的参数，
然后跟之前已经赋值过的参数合并起来，最后将所有参数传递给原始函数。

----------
讨论
----------
本节要解决的问题是让原本不兼容的代码可以一起工作。下面我会列举一系列的例子。

第一个例子是，假设你有一个点的列表来表示(x,y)坐标元组。
你可以使用下面的函数来计算两点之间的距离：

.. code-block:: python

    points = [ (1, 2), (3, 4), (5, 6), (7, 8) ]

    import math
    def distance(p1, p2):
        x1, y1 = p1
        x2, y2 = p2
        return math.hypot(x2 - x1, y2 - y1)

现在假设你想以某个点为基点，根据点和基点之间的距离来排序所有的这些点。
列表的 ``sort()`` 方法接受一个关键字参数来自定义排序逻辑，
但是它只能接受一个单个参数的函数(distance()很明显是不符合条件的)。
现在我们可以通过使用 ``partial()`` 来解决这个问题：

.. code-block:: python

    >>> pt = (4, 3)
    >>> points.sort(key=partial(distance,pt))
    >>> points
    [(3, 4), (1, 2), (5, 6), (7, 8)]
    >>>

更进一步，``partial()`` 通常被用来微调其他库函数所使用的回调函数的参数。
例如，下面是一段代码，使用 ``multiprocessing`` 来异步计算一个结果值，
然后这个值被传递给一个接受一个result值和一个可选logging参数的回调函数：

.. code-block:: python

    def output_result(result, log=None):
        if log is not None:
            log.debug('Got: %r', result)

    # A sample function
    def add(x, y):
        return x + y

    if __name__ == '__main__':
        import logging
        from multiprocessing import Pool
        from functools import partial

        logging.basicConfig(level=logging.DEBUG)
        log = logging.getLogger('test')

        p = Pool()
        p.apply_async(add, (3, 4), callback=partial(output_result, log=log))
        p.close()
        p.join()

当给 ``apply_async()`` 提供回调函数时，通过使用 ``partial()`` 传递额外的 ``logging`` 参数。
而 ``multiprocessing`` 对这些一无所知——它仅仅只是使用单个值来调用回调函数。

作为一个类似的例子，考虑下编写网络服务器的问题，``socketserver`` 模块让它变得很容易。
下面是个简单的echo服务器：

.. code-block:: python

    from socketserver import StreamRequestHandler, TCPServer

    class EchoHandler(StreamRequestHandler):
        def handle(self):
            for line in self.rfile:
                self.wfile.write(b'GOT:' + line)

    serv = TCPServer(('', 15000), EchoHandler)
    serv.serve_forever()

不过，假设你想给EchoHandler增加一个可以接受其他配置选项的 ``__init__`` 方法。比如：

.. code-block:: python

    class EchoHandler(StreamRequestHandler):
        # ack is added keyword-only argument. *args, **kwargs are
        # any normal parameters supplied (which are passed on)
        def __init__(self, *args, ack, **kwargs):
            self.ack = ack
            super().__init__(*args, **kwargs)

        def handle(self):
            for line in self.rfile:
                self.wfile.write(self.ack + line)

这么修改后，我们就不需要显式地在TCPServer类中添加前缀了。
但是你再次运行程序后会报类似下面的错误：

.. code-block:: python

    Exception happened during processing of request from ('127.0.0.1', 59834)
    Traceback (most recent call last):
    ...
    TypeError: __init__() missing 1 required keyword-only argument: 'ack'

初看起来好像很难修正这个错误，除了修改 ``socketserver`` 模块源代码或者使用某些奇怪的方法之外。
但是，如果使用 ``partial()`` 就能很轻松的解决——给它传递 ``ack`` 参数的值来初始化即可，如下：

.. code-block:: python

    from functools import partial
    serv = TCPServer(('', 15000), partial(EchoHandler, ack=b'RECEIVED:'))
    serv.serve_forever()

在这个例子中，``__init__()`` 方法中的ack参数声明方式看上去很有趣，其实就是声明ack为一个强制关键字参数。
关于强制关键字参数问题我们在7.2小节我们已经讨论过了，读者可以再去回顾一下。

很多时候 ``partial()`` 能实现的效果，lambda表达式也能实现。比如，之前的几个例子可以使用下面这样的表达式：

.. code-block:: python

    points.sort(key=lambda p: distance(pt, p))
    p.apply_async(add, (3, 4), callback=lambda result: output_result(result,log))
    serv = TCPServer(('', 15000),
            lambda *args, **kwargs: EchoHandler(*args, ack=b'RECEIVED:', **kwargs))

这样写也能实现同样的效果，不过相比而已会显得比较臃肿，对于阅读代码的人来讲也更加难懂。
这时候使用 ``partial()`` 可以更加直观的表达你的意图(给某些参数预先赋值)。
============================
7.9 将单方法的类转换为函数
============================

----------
问题
----------
你有一个除 ``__init__()`` 方法外只定义了一个方法的类。为了简化代码，你想将它转换成一个函数。

----------
解决方案
----------
大多数情况下，可以使用闭包来将单个方法的类转换成函数。
举个例子，下面示例中的类允许使用者根据某个模板方案来获取到URL链接地址。

.. code-block:: python

    from urllib.request import urlopen

    class UrlTemplate:
        def __init__(self, template):
            self.template = template

        def open(self, **kwargs):
            return urlopen(self.template.format_map(kwargs))

    # Example use. Download stock data from yahoo
    yahoo = UrlTemplate('http://finance.yahoo.com/d/quotes.csv?s={names}&f={fields}')
    for line in yahoo.open(names='IBM,AAPL,FB', fields='sl1c1v'):
        print(line.decode('utf-8'))

这个类可以被一个更简单的函数来代替：

.. code-block:: python

    def urltemplate(template):
        def opener(**kwargs):
            return urlopen(template.format_map(kwargs))
        return opener

    # Example use
    yahoo = urltemplate('http://finance.yahoo.com/d/quotes.csv?s={names}&f={fields}')
    for line in yahoo(names='IBM,AAPL,FB', fields='sl1c1v'):
        print(line.decode('utf-8'))

----------
讨论
----------
大部分情况下，你拥有一个单方法类的原因是需要存储某些额外的状态来给方法使用。
比如，定义UrlTemplate类的唯一目的就是先在某个地方存储模板值，以便将来可以在open()方法中使用。

使用一个内部函数或者闭包的方案通常会更优雅一些。简单来讲，一个闭包就是一个函数，
只不过在函数内部带上了一个额外的变量环境。闭包关键特点就是它会记住自己被定义时的环境。
因此，在我们的解决方案中，``opener()`` 函数记住了 ``template`` 参数的值，并在接下来的调用中使用它。

任何时候只要你碰到需要给某个函数增加额外的状态信息的问题，都可以考虑使用闭包。
相比将你的函数转换成一个类而言，闭包通常是一种更加简洁和优雅的方案。
============================
7.10 带额外状态信息的回调函数
============================

----------
问题
----------
你的代码中需要依赖到回调函数的使用(比如事件处理器、等待后台任务完成后的回调等)，
并且你还需要让回调函数拥有额外的状态值，以便在它的内部使用到。

----------
解决方案
----------
这一小节主要讨论的是那些出现在很多函数库和框架中的回调函数的使用——特别是跟异步处理有关的。
为了演示与测试，我们先定义如下一个需要调用回调函数的函数：

.. code-block:: python

    def apply_async(func, args, *, callback):
        # Compute the result
        result = func(*args)

        # Invoke the callback with the result
        callback(result)

实际上，这段代码可以做任何更高级的处理，包括线程、进程和定时器，但是这些都不是我们要关心的。
我们仅仅只需要关注回调函数的调用。下面是一个演示怎样使用上述代码的例子：

.. code-block:: python

    >>> def print_result(result):
    ...     print('Got:', result)
    ...
    >>> def add(x, y):
    ...     return x + y
    ...
    >>> apply_async(add, (2, 3), callback=print_result)
    Got: 5
    >>> apply_async(add, ('hello', 'world'), callback=print_result)
    Got: helloworld
    >>>

注意到 ``print_result()`` 函数仅仅只接受一个参数 ``result`` 。不能再传入其他信息。
而当你想让回调函数访问其他变量或者特定环境的变量值的时候就会遇到麻烦。

为了让回调函数访问外部信息，一种方法是使用一个绑定方法来代替一个简单函数。
比如，下面这个类会保存一个内部序列号，每次接收到一个 ``result`` 的时候序列号加1：

.. code-block:: python

    class ResultHandler:

        def __init__(self):
            self.sequence = 0

        def handler(self, result):
            self.sequence += 1
            print('[{}] Got: {}'.format(self.sequence, result))

使用这个类的时候，你先创建一个类的实例，然后用它的 ``handler()`` 绑定方法来做为回调函数：

.. code-block:: python

    >>> r = ResultHandler()
    >>> apply_async(add, (2, 3), callback=r.handler)
    [1] Got: 5
    >>> apply_async(add, ('hello', 'world'), callback=r.handler)
    [2] Got: helloworld
    >>>

第二种方式，作为类的替代，可以使用一个闭包捕获状态值，例如：

.. code-block:: python

    def make_handler():
        sequence = 0
        def handler(result):
            nonlocal sequence
            sequence += 1
            print('[{}] Got: {}'.format(sequence, result))
        return handler

下面是使用闭包方式的一个例子：

.. code-block:: python

    >>> handler = make_handler()
    >>> apply_async(add, (2, 3), callback=handler)
    [1] Got: 5
    >>> apply_async(add, ('hello', 'world'), callback=handler)
    [2] Got: helloworld
    >>>

还有另外一个更高级的方法，可以使用协程来完成同样的事情：

.. code-block:: python

    def make_handler():
        sequence = 0
        while True:
            result = yield
            sequence += 1
            print('[{}] Got: {}'.format(sequence, result))

对于协程，你需要使用它的 ``send()`` 方法作为回调函数，如下所示：

.. code-block:: python

    >>> handler = make_handler()
    >>> next(handler) # Advance to the yield
    >>> apply_async(add, (2, 3), callback=handler.send)
    [1] Got: 5
    >>> apply_async(add, ('hello', 'world'), callback=handler.send)
    [2] Got: helloworld
    >>>

----------
讨论
----------
基于回调函数的软件通常都有可能变得非常复杂。一部分原因是回调函数通常会跟请求执行代码断开。
因此，请求执行和处理结果之间的执行环境实际上已经丢失了。如果你想让回调函数连续执行多步操作，
那你就必须去解决如何保存和恢复相关的状态信息了。

至少有两种主要方式来捕获和保存状态信息，你可以在一个对象实例(通过一个绑定方法)或者在一个闭包中保存它。
两种方式相比，闭包或许是更加轻量级和自然一点，因为它们可以很简单的通过函数来构造。
它们还能自动捕获所有被使用到的变量。因此，你无需去担心如何去存储额外的状态信息(代码中自动判定)。

如果使用闭包，你需要注意对那些可修改变量的操作。在上面的方案中，
``nonlocal`` 声明语句用来指示接下来的变量会在回调函数中被修改。如果没有这个声明，代码会报错。

而使用一个协程来作为一个回调函数就更有趣了，它跟闭包方法密切相关。
某种意义上来讲，它显得更加简洁，因为总共就一个函数而已。
并且，你可以很自由的修改变量而无需去使用 ``nonlocal`` 声明。
这种方式唯一缺点就是相对于其他Python技术而言或许比较难以理解。
另外还有一些比较难懂的部分，比如使用之前需要调用 ``next()`` ，实际使用时这个步骤很容易被忘记。
尽管如此，协程还有其他用处，比如作为一个内联回调函数的定义(下一节会讲到)。

如果你仅仅只需要给回调函数传递额外的值的话，还有一种使用 ``partial()`` 的方式也很有用。
在没有使用 ``partial()`` 的时候，你可能经常看到下面这种使用lambda表达式的复杂代码：

.. code-block:: python

    >>> apply_async(add, (2, 3), callback=lambda r: handler(r, seq))
    [1] Got: 5
    >>>

可以参考7.8小节的几个示例，教你如何使用 ``partial()`` 来更改参数签名来简化上述代码。
============================
7.11 内联回调函数
============================

----------
问题
----------
当你编写使用回调函数的代码的时候，担心很多小函数的扩张可能会弄乱程序控制流。
你希望找到某个方法来让代码看上去更像是一个普通的执行序列。

----------
解决方案
----------
通过使用生成器和协程可以使得回调函数内联在某个函数中。
为了演示说明，假设你有如下所示的一个执行某种计算任务然后调用一个回调函数的函数(参考7.10小节)：

.. code-block:: python

    def apply_async(func, args, *, callback):
        # Compute the result
        result = func(*args)

        # Invoke the callback with the result
        callback(result)

接下来让我们看一下下面的代码，它包含了一个 ``Async`` 类和一个 ``inlined_async`` 装饰器：

.. code-block:: python

    from queue import Queue
    from functools import wraps

    class Async:
        def __init__(self, func, args):
            self.func = func
            self.args = args

    def inlined_async(func):
        @wraps(func)
        def wrapper(*args):
            f = func(*args)
            result_queue = Queue()
            result_queue.put(None)
            while True:
                result = result_queue.get()
                try:
                    a = f.send(result)
                    apply_async(a.func, a.args, callback=result_queue.put)
                except StopIteration:
                    break
        return wrapper

这两个代码片段允许你使用 ``yield`` 语句内联回调步骤。比如：

.. code-block:: python

    def add(x, y):
        return x + y

    @inlined_async
    def test():
        r = yield Async(add, (2, 3))
        print(r)
        r = yield Async(add, ('hello', 'world'))
        print(r)
        for n in range(10):
            r = yield Async(add, (n, n))
            print(r)
        print('Goodbye')

如果你调用 ``test()`` ，你会得到类似如下的输出：

.. code-block:: python

    5
    helloworld
    0
    2
    4
    6
    8
    10
    12
    14
    16
    18
    Goodbye

你会发现，除了那个特别的装饰器和 ``yield`` 语句外，其他地方并没有出现任何的回调函数(其实是在后台定义的)。

----------
讨论
----------
本小节会实实在在的测试你关于回调函数、生成器和控制流的知识。

首先，在需要使用到回调的代码中，关键点在于当前计算工作会挂起并在将来的某个时候重启(比如异步执行)。
当计算重启时，回调函数被调用来继续处理结果。``apply_async()`` 函数演示了执行回调的实际逻辑，
尽管实际情况中它可能会更加复杂(包括线程、进程、事件处理器等等)。

计算的暂停与重启思路跟生成器函数的执行模型不谋而合。
具体来讲，``yield`` 操作会使一个生成器函数产生一个值并暂停。
接下来调用生成器的 ``__next__()`` 或 ``send()`` 方法又会让它从暂停处继续执行。

根据这个思路，这一小节的核心就在 ``inline_async()`` 装饰器函数中了。
关键点就是，装饰器会逐步遍历生成器函数的所有 ``yield`` 语句，每一次一个。
为了这样做，刚开始的时候创建了一个 ``result`` 队列并向里面放入一个 ``None`` 值。
然后开始一个循环操作，从队列中取出结果值并发送给生成器，它会持续到下一个 ``yield`` 语句，
在这里一个 ``Async`` 的实例被接受到。然后循环开始检查函数和参数，并开始进行异步计算 ``apply_async()`` 。
然而，这个计算有个最诡异部分是它并没有使用一个普通的回调函数，而是用队列的 ``put()`` 方法来回调。

这时候，是时候详细解释下到底发生了什么了。主循环立即返回顶部并在队列上执行 ``get()`` 操作。
如果数据存在，它一定是 ``put()`` 回调存放的结果。如果没有数据，那么先暂停操作并等待结果的到来。
这个具体怎样实现是由 ``apply_async()`` 函数来决定的。
如果你不相信会有这么神奇的事情，你可以使用 ``multiprocessing`` 库来试一下，
在单独的进程中执行异步计算操作，如下所示：

.. code-block:: python

    if __name__ == '__main__':
        import multiprocessing
        pool = multiprocessing.Pool()
        apply_async = pool.apply_async

        # Run the test function
        test()

实际上你会发现这个真的就是这样的，但是要解释清楚具体的控制流得需要点时间了。

将复杂的控制流隐藏到生成器函数背后的例子在标准库和第三方包中都能看到。
比如，在 ``contextlib`` 中的 ``@contextmanager`` 装饰器使用了一个令人费解的技巧，
通过一个 ``yield`` 语句将进入和离开上下文管理器粘合在一起。
另外非常流行的 ``Twisted`` 包中也包含了非常类似的内联回调。
============================
7.12 访问闭包中定义的变量
============================

----------
问题
----------
你想要扩展函数中的某个闭包，允许它能访问和修改函数的内部变量。

----------
解决方案
----------
通常来讲，闭包的内部变量对于外界来讲是完全隐藏的。
但是，你可以通过编写访问函数并将其作为函数属性绑定到闭包上来实现这个目的。例如：

.. code-block:: python

    def sample():
        n = 0
        # Closure function
        def func():
            print('n=', n)

        # Accessor methods for n
        def get_n():
            return n

        def set_n(value):
            nonlocal n
            n = value

        # Attach as function attributes
        func.get_n = get_n
        func.set_n = set_n
        return func

下面是使用的例子:

.. code-block:: python

    >>> f = sample()
    >>> f()
    n= 0
    >>> f.set_n(10)
    >>> f()
    n= 10
    >>> f.get_n()
    10
    >>>

----------
讨论
----------
为了说明清楚它如何工作的，有两点需要解释一下。首先，``nonlocal`` 声明可以让我们编写函数来修改内部变量的值。
其次，函数属性允许我们用一种很简单的方式将访问方法绑定到闭包函数上，这个跟实例方法很像(尽管并没有定义任何类)。

还可以进一步的扩展，让闭包模拟类的实例。你要做的仅仅是复制上面的内部函数到一个字典实例中并返回它即可。例如：

.. code-block:: python

    import sys
    class ClosureInstance:
        def __init__(self, locals=None):
            if locals is None:
                locals = sys._getframe(1).f_locals

            # Update instance dictionary with callables
            self.__dict__.update((key,value) for key, value in locals.items()
                                if callable(value) )
        # Redirect special methods
        def __len__(self):
            return self.__dict__['__len__']()

    # Example use
    def Stack():
        items = []
        def push(item):
            items.append(item)

        def pop():
            return items.pop()

        def __len__():
            return len(items)

        return ClosureInstance()

下面是一个交互式会话来演示它是如何工作的：

.. code-block:: python

    >>> s = Stack()
    >>> s
    <__main__.ClosureInstance object at 0x10069ed10>
    >>> s.push(10)
    >>> s.push(20)
    >>> s.push('Hello')
    >>> len(s)
    3
    >>> s.pop()
    'Hello'
    >>> s.pop()
    20
    >>> s.pop()
    10
    >>>

有趣的是，这个代码运行起来会比一个普通的类定义要快很多。你可能会像下面这样测试它跟一个类的性能对比：

.. code-block:: python

    class Stack2:
        def __init__(self):
            self.items = []

        def push(self, item):
            self.items.append(item)

        def pop(self):
            return self.items.pop()

        def __len__(self):
            return len(self.items)

如果这样做，你会得到类似如下的结果：

.. code-block:: python

    >>> from timeit import timeit
    >>> # Test involving closures
    >>> s = Stack()
    >>> timeit('s.push(1);s.pop()', 'from __main__ import s')
    0.9874754269840196
    >>> # Test involving a class
    >>> s = Stack2()
    >>> timeit('s.push(1);s.pop()', 'from __main__ import s')
    1.0707052160287276
    >>>

结果显示，闭包的方案运行起来要快大概8%，大部分原因是因为对实例变量的简化访问，
闭包更快是因为不会涉及到额外的self变量。

Raymond Hettinger对于这个问题设计出了更加难以理解的改进方案。不过，你得考虑下是否真的需要在你代码中这样做，
而且它只是真实类的一个奇怪的替换而已，例如，类的主要特性如继承、属性、描述器或类方法都是不能用的。
并且你要做一些其他的工作才能让一些特殊方法生效(比如上面 ``ClosureInstance`` 中重写过的 ``__len__()`` 实现。)

最后，你可能还会让其他阅读你代码的人感到疑惑，为什么它看起来不像一个普通的类定义呢？
(当然，他们也想知道为什么它运行起来会更快)。尽管如此，这对于怎样访问闭包的内部变量也不失为一个有趣的例子。

总体上讲，在配置的时候给闭包添加方法会有更多的实用功能，
比如你需要重置内部状态、刷新缓冲区、清除缓存或其他的反馈机制的时候。
