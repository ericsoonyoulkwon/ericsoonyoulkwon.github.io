# Fibonacci sequence

#### Category: Practice Algorithms

---

### Definition: [Fibonacci sequence – Wikipedia](https://en.wikipedia.org/wiki/Fibonacci_sequence)

Defining a function to return the n-th Fibonacci numbers,
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
 
mx = int(input("How many Fibonacci numbers do you need? "))
for i in range(1,mx+1):
    print(n_fibonacci(i))
```
To prevent the input from automatically being printed along with the result, I added a string in front of the input so that I could make the result and input separated.

Depending on the computational power available to you, enter the number of Fibonacci numbers you want to see.
I will enter 20 as an example.

Result

```python
How many Fibonacci numbers do you need? 20
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
```
