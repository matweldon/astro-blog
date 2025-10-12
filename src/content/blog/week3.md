---
date: "2025-10-10"
title: "AI Accelerator Week 3"
pubDate: 10 October 2025
description: Deep learning and Pytorch
slug: week-3
---


* loss, gradients, optimisation
* Some cool demos
 - https://cs231n.stanford.edu/
 - https://emiliendupont.github.io/2018/01/24/optimization-visualization/
 - https://playground.tensorflow.org/
* Pytorch:
  - Linear regression example
  - Why `no_grad` is needed (because any torch.tensor remembers any functions it's associated with otherwise)
  - understanding how the computational graph works
  - More cool recursive functions
  - Why grad accumulates (because the graph is not necessarily a tree)

Week 3 of the AI accelerator introduced the concepts of deep learning, brought to life by some fun browser-based demos and practical exercises with Pytorch. The course curriculum covered the basic architecture of neural networks, the loss function, gradients, backpropagation and gradient descent. We didn't have much time to cover the material so self-study was essential. I think I would have really struggled to grasp all these concepts for the first time in the time available, but the course leader clearly explained how everything fit together.

### Interactive demos

One of the tools the course leader used really effectively was interactive browser-based demos. I especially enjoyed playing around with the Tensorflow [neural network playground](https://playground.tensorflow.org/) which gives a good intuition into neural network architectures. I got a good intuition from making the neural network 'too small' so that it was not able to fit the data, and then gradually building up the representative capacity by adding nodes and layers. Another neural network running in the browser, although less interactive, was the Stanford [Deep Learning for Computer Vision](https://cs231n.stanford.edu/) course website.

To allow us to experiment with optimisation algorithms, the course leader used an open-source project by [Emilien Dupont](https://emiliendupont.github.io/2018/01/24/optimization-visualization/) which shows visually how gradient descent and related algorithms find the minimum of a loss function.


### Back to basics - simple linear regression

After covering the concepts we were introduced to the Pytorch package. I have some experience training deep learning models, but deep learning libraries, Pytorch in particular, still hold some mysteries for me.







```python
# Scalar parameters
a = torch.randn(1, requires_grad=True)
b = torch.randn(1, requires_grad=True)

learning_rate = 0.01
for step in range(100):
    y_pred = a + b * x

    # loss := mean squared error
    loss = ((y_pred - y) ** 2).mean()

    # (a,b) -> y_pred -> loss forms a DAG
    # Backpropagation traverses the graph in reverse,
    # calculating the gradients in (a,b)
    loss.backward()

    with torch.no_grad():
        # Step down the gradient
        a -= learning_rate * a.grad
        b -= learning_rate * b.grad

        # Zero the gradient for the next run
        a.grad.zero_()
        b.grad.zero_()

    if step % 10 == 0:
        print(f"Step {step}: Loss={loss.item():.4f}, a={a.item():.4f}, b={b.item():.4f}")
```

### The computational graph

```python
def traverse_graph(grad_fn):
   if grad_fn is None:
       return None
   grad_fn_name = type(grad_fn).__name__
   memloc = hex(id(grad_fn))[-5:]
   children = []
   for next_func, _ in grad_fn.next_functions:
       if next_func is not None:
           children.append(traverse_graph(next_func))
   return ((grad_fn_name, memloc), children)

def pretty_print_graph(node, prefix='', is_last=True, show_memloc=False):
    """
    Recursively prints the computation graph in a tree-like structure.
    Expects ((name, memloc), [children]) as node format.
    """
    name, children = node
    connector = '└── ' if is_last else '├── '
    if show_memloc:
        formatted_name = f"{name[0]}: {name[1]}"
    else:
        formatted_name = name[0]
    print(prefix + connector + formatted_name)
    if children:
        # Prepare the prefix for child nodes:
        child_prefix = prefix + ('    ' if is_last else '│   ')
        for idx, child in enumerate(children):
            is_child_last = (idx == len(children) - 1)
            pretty_print_graph(child, child_prefix, is_child_last, show_memloc)
```

```python
import torch
# Fixed 2*2 matrix 'data'
x = torch.randn((2,2), requires_grad=False)

# Matrix 'parameters'
a = torch.randn((2,2), requires_grad=True)
b = torch.randn((2,2), requires_grad=True)
c = torch.randn((2,2), requires_grad=True)
d = torch.randn((2,2), requires_grad=True)

e = a + b @ x
f = c + d*x
g = e + f @ x + d
h = a + g
```

```python
pretty_print_graph(traverse_graph(h.grad_fn))
# └── AddBackward0
#     ├── AccumulateGrad
#     └── AddBackward0
#         ├── AddBackward0
#         │   ├── AddBackward0
#         │   │   ├── AccumulateGrad
#         │   │   └── MmBackward0
#         │   │       └── AccumulateGrad
#         │   └── MmBackward0
#         │       └── AddBackward0
#         │           ├── AccumulateGrad
#         │           └── MulBackward0
#         │               └── AccumulateGrad
#         └── AccumulateGrad
```

```python
pretty_print_graph(traverse_graph(h.grad_fn),show_memloc=True)
# └── AddBackward0: acd60
#     ├── AccumulateGrad: 3e5c0                      # 'a'
#     └── AddBackward0: 3fc70
#         ├── AddBackward0: 33d30
#         │   ├── AddBackward0: 335e0
#         │   │   ├── AccumulateGrad: 3e5c0          # 'a'
#         │   │   └── MmBackward0: 32620
#         │   │       └── AccumulateGrad: 32800      # 'b'
#         │   └── MmBackward0: 33250
#         │       └── AddBackward0: 318d0
#         │           ├── AccumulateGrad: 311b0      # 'c'
#         │           └── MulBackward0: 33280
#         │               └── AccumulateGrad: 32fb0  # 'd'
#         └── AccumulateGrad: 32fb0                  # 'd'
```