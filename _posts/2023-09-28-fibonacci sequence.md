# Fibonacci sequence

#### Category: Practice Algorithms

---

### Definition: [Fibonacci sequence – Wikipedia](https://en.wikipedia.org/wiki/Fibonacci_sequence)

defining a function to return the Fibonacci numbers
and returning the results depending on the maximum iterations you want to perform.

```python
# n-th element of the Fibonacci sequence
 
def n_fibonacci (n):
    if n == 1:
        n_th = 0
    elif n == 2:
        n_th = 1
    else:
        t = 3
        while t <= n:
            n_th = n_fibonacci (t-2) + n_fibonacci (t-1)
            t += 1
    return n_th
 
mx = int(input())
for i in range(1,mx+1):
    print(n_fibonacci(i))
```

Depending on the computational power available to you, enter the number of Fibonacci numbers you want to see.
I will enter 999 as an example.

```python
999
```

Result

```python
0
2
3
4
5
6
7
8
9
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
20
21
22
23
24
25
26
27
28
0
1
1
2
3
5
8
13
21
34
55
89
144
233
377
610
987
1597
2584
4181
6765
10946
17711
28657
46368
.
.
.
```
