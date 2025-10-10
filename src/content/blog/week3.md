---
date: "2025-10-10"
title: "AI Accelerator Week 3"
pubDate: 10 October 2025
description: Deep learning and Pytorch
slug: week-3
---


* loss, gradients, optimisation
* Some cool demos
* Pytorch:
  - Linear regression example
  - Why `no_grad` is needed (because any torch.tensor remembers any functions it's associated with otherwise)
  - understanding how the computational graph works
  - More cool recursive functions
  - Why grad accumulates (because the graph is not necessarily a tree)

```python
a = torch.randn(1, requires_grad=True)
b = torch.randn(1, requires_grad=True)
lr = 0.01
for step in range(100):
    y_pred = a + b * x
    loss = ((y_pred - y) ** 2).mean()

    # (a,b) -> y_pred -> loss forms a DAG
    # Backpropagation traverses the graph in reverse, building up gradients
    loss.backward()

    with torch.no_grad():
        a -= lr * a.grad
        b -= lr * b.grad
        a.grad.zero_()
        b.grad.zero_()

    if step % 10 == 0:
        print(f"Step {step}: Loss={loss.item():.4f}, a={a.item():.4f}, b={b.item():.4f}")
```

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
# Fixed matrix 'data'
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