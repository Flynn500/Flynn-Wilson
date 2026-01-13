# Flynn-Wilson
I am in my final year of a Computer Science degree at AUT. The following projects showcase what I have learned in and outside of university. 

## Dubious
Dubious is a Python library for propagating uncertainty through mathematical expressions using Monte Carlo simulation, allowing uncertain values to behave like normal numerical objects. You can use `Uncertain()` objects as a wrapper for distributions, apply numerical methods, and calculate statistics about the resulting uncertainty. 

I had the personal constraint for this project of using minimal external dependencies. Other than a library for fast array computation (originally numpy), I wanted to implement as much myself as is possible.

An example:
```python
#two non-Gaussian marginals correlated using copula
from dubious.distributions import Beta, LogNormal
from dubious.core import Uncertain, Context

ctx = Context()

conv = Uncertain(Beta(20,80), ctx=ctx)
traffic = Uncertain(LogNormal(8.0,0.4), ctx=ctx)

conv.corr(traffic, 0.7)

sales = conv * traffic

print(f"Mean: {sales.mean()},")
print(f"p10, p90: {sales.quantile(0.1)}, {sales.quantile(0.9)}")
```
output: `Mean: 682.9556148632868,
p10, p90: 282.49895472166844, 1195.2049066217655`

It works by creating a graph of operations, evaluating said graph with Monte Carlo simulations, correlation being implemented with Gaussian copula. I got the idea by watching a engineering friend struggle with the concept and I wanted to attempt a project that may have some real world use. I figured for this domain this method of uncertainty propagation be suitable for many use cases.

At the time of writing this, there are a few libraries who do similar things in different ways. I've aimed for simplicity, I wanted to have a simple API that gives results quickly with minimal boilerplate. The distributions as numerical objects model does that well. While other libraries might be better for more rigorous uncertainty propagation, dubious is a good option for exploratory analysis given its ease of use.

I learned a lot about writing code that's easy for other people to use in this project. Many of my prior projects were done alone, without the intention of someone else interacting with the code. For this library, I had to think about what would be most intuitive to use and how I could document that finer details of the codebase so that you can effectively use it without knowing the exact implementation. Sphinx was an excellent tool to achieve this, every time I implemented a function I'd write a docstring explaining what it does, and what limitations the user needs to be aware of.

This type of project isn't feasible in vanilla python, to meet performance requirements we have to offload the array based computation to other libraries, initially this was numpy before I created substratum (my next project) as a drop in replacement.

For more about dubious see: https://github.com/Flynn500/dubious/

## Substratum
Substratum is another library, this time made in rust. Substratum is a Rust-based numerical computation library with Python bindings, originally built to support Dubious but it's also designed to stand as its own project. After porting it to dubious, the two libraries stand by themselves, no external dependencies required. It is named as such as it is the underlying "substrate" of my original library.

I developed substratum as I became interested in how high performance libraries like numpy worked under the hood after developing dubious. When using python, you frequently find yourself using "magic" libraries like numpy to compute things at large scales. This project taught me how to take advantage of lower level languages to achieve these performance boosts myself, and bring them back into python to keep things simple to use. I chose rust for this project simply because I had read the book in my second year of university, and pyo3 provides a simple way to create python bindings.

An example of matrix multiplication:

```python
import substratum as ss

a = ss.Array([2, 2], [1.0, 2.0, 3.0, 4.0])
b = ss.Array([2, 2], [5.0, 6.0, 7.0, 8.0])

print(f"a @ b = {(a @ b).tolist()}")
```
output: `a @ b = [19.0, 22.0, 43.0, 50.0]`

Substratum supports the operations needed for dubious, so these are the ones I focused on implementing:
- Broadcast operations
- trig methods
- Statistical methods (mean, var, quantile)
- matrix methods and constructors
- Cholesky and eigen decomposition

The library is less performant than that which it sought to replace (numpy), but the performance is absolutely good enough for my use case. And it was never meant to be a numpy replacement, rather a way to deepen my understanding of numerical computation and the python ecosystem as a whole. I find python an excellent language to work with (other than managing dependencies), the only issue I sometimes run into is performance. After this project, I can see myself picking up C, C++ or Rust as a tool to fill in these gaps where computational speed really matters.

For more see: https://github.com/Flynn500/substratum

