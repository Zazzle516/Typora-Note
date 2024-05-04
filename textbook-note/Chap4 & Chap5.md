# Parallel & Generator

## Chap4

这节没什么好说的，都在 408 学过了，有也基本是伪代码



## Chap5

这节的内容在之前那个相关的 Iterator 的练习已经见过了，只能说练习和教科书对应的不太好

```python
def all_pairs(s):
    for item1 in s:
        for item2 in s:
            yield (item1, item2)

class LetterIterable(object):
    def __iter__(self):
        current = 'a'
        while current < 'd':
            yield current
            current = chr(ord(current) + 1)

letters = LetterIterable()

letter_generator = all_pairs(letters)

print(letter_generator.__next__())
print(letter_generator.__next__())
print(letter_generator.__next__())
print(letter_generator.__next__())
```



### Stream

```python
class Stream(object):
    """
    A lazyily computed recursive list
    """
    def __init__(self, first, compute_rest, empty=False):
        self.first = first
        self._compute_rest = compute_rest
        self.empty = empty
        self._rest = None
        self._computed = False

    @property
    # 让方法像属性一样访问
    def rest(self):
        """
        Return the rest of the stream, computing it if necessary
        """
        assert not self.empty, 'Empty stream has no rest'

        if not self._computed:
            self._rest = self._compute_rest()
            self._computed = True
        return self._rest
    
    def __repr__(self):
        if self.empty:
            return '<Empty Stream>'
        return 'Stream({0}, <compute_rest>)'.format(repr(self.first))
    
Stream.empty = Stream(None, None, True)

def make_integer_stream(first = 1):
    def compute_rest():
        return make_integer_stream(first + 1)
    return Stream(first, compute_rest)

def map_stream(fn, s):
    if s.empty:
        return s
    def compute_rest():
        return map_stream(fn, s.rest)
    return Stream(fn(s.first), compute_rest)

def filter_stream(fn, s):
    if s.empty:
        return s
    def compute_rest():
        return filter_stream(fn, s.rest)
    if fn(s.first):
        # 持续找下一个元素 直到 fn(s.first) is True
        return Stream(s.first, compute_rest)
    return compute_rest()

def truncate_stream(s, k):
    if s.empty or k == 0:
        return Stream.empty
    def compute_rest():
        return truncate_stream(s.rest, k - 1)
    return Stream(s.first, compute_rest)

def stream_to_list(s):
    r = []
    while not s.empty:
        r.append(s.first)
        s = s.rest
    return r

######################## testcase ########################

s = Stream(1, lambda: Stream(2 + 3, lambda: Stream.empty))

print(s.first)
print(s.rest.first)
print(s.rest.rest)

s = make_integer_stream(3)
m = map_stream(lambda x: x * x, s)
print(stream_to_list(truncate_stream(m, 5)))

def primes(pos_stream):
    def not_divible(x):
        return x % pos_stream.first != 0
    
    def compute_rest():
        return primes(filter_stream(not_divible, pos_stream.rest))
    
    return Stream(pos_stream.first, compute_rest)

p1 = primes(make_integer_stream(2))

print(stream_to_list(truncate_stream(p1, 7)))
```

