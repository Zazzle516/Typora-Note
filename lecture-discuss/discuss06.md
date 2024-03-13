# Nonlocal, Midterm Review

## Nonlocal

### 1.2

```python
def memory(n):
    def apply_function(f):
        nonlocal n
        n = f(n)
        return n
    return apply_function

f = memory(10)
print(f(lambda x: x * 2))
print(f(lambda x: x - 7))
print(f(lambda x: x > 5))
```



## Midterm Review

