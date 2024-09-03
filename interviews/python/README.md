### Шпаргалка по Python

## Область видимости

**Что выведет следующий код?**
```python
x = 10

def sum_func(x):
    x += 5

def first_func():
    global x
    sum_func(x)

first_func()
print(x)
```
**Ответ:** 10  
*Пояснение:* при попытке изменить переменную x в sum_func() будет создана новая локальная переменная.

## Декораторы

Декораторы это просто функции, которые возвращают функции, которые имеют дополнительный функционал внутри себя, прежде чем вызвать нужну нам функцию.

```python
def dec(func):
    def inner(*args, **kwargs):
        print('call func')
        return func(*args, **kwargs)
    return inner

def some_func(x):
    return x * x

some_func = dec(some_func)
some_func(2)
# >>> call func
# >>> 4
```

Эта запись эквивалентна коду снизу:
```python
def dec(func):
    def inner(*args, **kwargs):
        print('call func')
        return func(*args, **kwargs)
    return inner

@dec
def some_func(x):
    return x * x

some_func(2)
# >>> call func
# >>> 4
```

В декораторы можно так же передавать аргументы.
```python
def dec(handle):
    def decorator(func):
        def inner(*args, **kwargs):
            print(func.__name__, args, kwargs, file=handle)
            return func(*args, **kwargs)
        return inner
    return decorator

@dec(sys.stderr)
def some_func(x):
    return x * x
```

Одной функции можно задавать несколько декораторов. Причём *порядок* **имеет** значение! Можно считать это композицией, сверху вниз.
```python
def square(func):
    return lambda x: func(x * x)

def addsome(func):
    return lambda x: func(x + 42)

@square
@addsome
def identity(x):
    return x

identity(2) # >>> 46; 
            # square(addsome(identity))
            # square(2) -> inner(2 * 2) -> addsome(4) -> inner(4 + 42) -> func(46)


@addsome
@square
def identity(x):
    return x

identity(2) # >>> 1936
```
