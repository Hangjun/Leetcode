From [mitbbs](http://www.mitbbs.com/article_t/JobHunting/32230525.html) for Quantcast

Also seen from CC150

CC150 uses two stacks. [Stackoverflow](http://stackoverflow.com/questions/2168803/how-to-sort-a-stack-using-only-push-pop-top-isempty-isfull/3510086#3510086) uses system stack, using recursion

```python
def sort_by_two_stacks(s1):
    s2 = []
    while True:
        while not s2 or s2[-1] < s1[-1]:
            s2.append(s1.pop())
            if not s1:
                return s2
        tmp = s1.pop()
        while s2 and s2[-1] > tmp:
            s1.append(s2.pop())
        s1.append(tmp)
    
print sort_by_two_stacks([5,3,8,4,6,9,7])
```

```python
def sort(s):
    if len(s) > 0:
        last = s.pop()
        sort(s)
        insert(last, s)

def insert(num, s):
    if len(s) == 0:
        s.append(num)
        return

    if num < s[-1]:
        last = s.pop()
        insert(num, s)
        s.append(last)
    else:
        s.append(num)

s = [5,3,8,4,6,9,7]

sort(s)

print s
```
