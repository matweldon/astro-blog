---
date: "2025-10-10"
title: "AI Accelerator Week 3"
pubDate: 10 October 2025
description: Deep learning and Pytorch
slug: week-3
---

Week three of the AI accelerator introduced the concepts of deep learning, brought to life by some fun browser-based demos and practical exercises with Pytorch. The course curriculum covered the basic architecture of neural networks, the loss function, gradients, backpropagation and gradient descent. We didn't have much time to cover the material so self-study was essential. I think I would have really struggled to grasp all these concepts for the first time in the time available.

Rather than give a blow-by-blow account of the week I'll just cover the bits that I thought were interesting or new. I ended up spending a couple of hours on Friday going on a productive tangent, diving into the computational workings of the Pytorch package, so most of this post is about that. Most of the article assumes familiarity with the basics of deep learning.

### Interactive demos

One of the tools the course leader used really effectively was interactive browser-based demos. I especially enjoyed playing around with the Tensorflow [neural network playground](https://playground.tensorflow.org/) which gives a good intuition into neural network architectures. I got a good intuition from making the neural network 'too small' so that it was not able to fit the data, and then gradually building up the representative capacity by adding nodes and layers. Another neural network running in the browser, although less interactive, was the Stanford [Deep Learning for Computer Vision](https://cs231n.stanford.edu/) course website.

To allow us to experiment with optimisation algorithms, the course leader used an open-source project by [Emilien Dupont](https://emiliendupont.github.io/2018/01/24/optimization-visualization/) which shows visually how gradient descent and related algorithms find the minimum of a loss function.


### Pytorch: simple linear regression

I have some superficial experience with Pytorch, but I've always found the API a bit impenetrable, so I decided to do some simple experiments to get some intuition into how it works under the hood. I think this is something that distinguishes my learning preferences from others; I've worked with talented colleagues who are happy to see a new package and say "right, what's the most impressive thing I can build with this?" Whereas I'm quite cautious about building with a new tool until I've taken it back to basics to understand how it works.

For example, with Pytorch, typically we'll see a snippet like:

```python
for data, target in dataloader:
    optimizer.zero_grad()
    output = model(data) 
    loss = loss_fn(output, target) 
    loss.backward() 
    optimizer.step()
```

If you've used Pytorch before you'll have seen a similar training loop: load the data a batch at a time; zero the gradients ready to recompute them; get the predicted output by inference from the model and the data; compare the prediction to the target with the loss function; compute the gradients of the parameters with respect to the loss ($\nabla_{\theta} L$) through back-propagation; and finally update the parameters a step.

But I see this snippet, and I have so many questions! Why does the optimiser have to zero all the gradients at the start of each iteration? Why aren't they just overwritten by default? If the model object contains all of the differentiable parameters, why is `.backward()` called on the loss and not on the model object? For that matter, why is `.zero_grad()` and `.step()` called on the optimizer and not on the model parameters? What is all this spooky action at a distance?

This is where the flexible course structure was very handy, because I was able to have a long conversation with an LLM about this code snippet, and gradually the picture became clearer. Through explanation, examples, and giving me code to run, the LLM gradually helped me to see what my naive non-computer-science brain didn't understand -- the unseen complex web of memory references extending like a mycelial network under the surface of my code.

First, I asked the LLM to show me the most basic example I could imagine: a simple linear regression.

```python
import torch
torch.manual_seed(0)

# Generate the data
a_true = 1.5
b_true = -2.0
x = torch.randn(10)
y = a_true + b_true * x + 0.1 * torch.randn(10)

# Scalar parameters
a = torch.randn(1, requires_grad=True)
b = torch.randn(1, requires_grad=True)
```

Note that every piece of data, including the fixed data, needs to be a `torch.tensor` otherwise the whole system won't work. the argument `requires_grad=True` marks a tensor as a trainable parameter that needs gradients to be computed.

```python
learning_rate = 0.01
for step in range(90):
    y_hat = a + b * x

    # loss := mean squared error
    loss = ((y_hat - y) ** 2).mean()

    loss.backward()

    with torch.no_grad():
        # Step down the gradient
        a -= learning_rate * a.grad
        b -= learning_rate * b.grad

        # Zero the gradient for the next run
        a.grad.zero_()
        b.grad.zero_()

    if step % 20 == 0:
        print(f"Step {step}: Loss={loss.item():.4f},",
        f"a={a.item():.4f}, b={b.item():.4f}")

# Step 0: Loss=17.6487, a=0.4925, b=1.1871
# Step 20: Loss=0.6128, a=1.4560, b=-1.3277
# Step 40: Loss=0.0345, a=1.5238, b=-1.8259
# Step 60: Loss=0.0074, a=1.4996, b=-1.9357
# Step 80: Loss=0.0052, a=1.4823, b=-1.9634
```

There's a lot going on here. Here, the gradient attributes on the parameters are zeroed explicitly which was more intuitive to me, but still didn't explain why the gradients don't just overwrite by default. The LLM explained why the `with torch.no_grad():` block was needed. Because Pytorch normally records any mathematical operations on tensors that might contribute to the gradients, you have to specifically tell Pytorch not to do this when iterating over the parameters. 

At this stage I'm beginning to see how the flow of computations for backpropagation works.

Basically, every piece of data in a Pytorch system -- including all of the parameters, the `data` and `target` (i.e. $X$ and $y$), the `output` (i.e. $\hat{y}$) and the loss -- is a `torch.tensor` object, a multi-dimensional array of numbers with additional attributes. When instantiated in code, each tensor records the mathematical operations (addition, multiplication, matrix multiplication etc.) that link it to other tensors, and saves a list of pointers that point backwards to the data objects it was made from. The LLM explained that these pointers form a huge computational graph that the gradient backpropagation traverses to accumulate (sum) gradients in the parameters, which are the leaf nodes of the graph. The `.backward()` method is just called on the loss because the loss is the root node of the graph. In fact, the `.backward()` method can be called on any scalar tensor object.

Then I had an idea: last week I built a decision tree classifier, and what I learned from that experience is that LLMs are very good at writing recursive functions to traverse graphs. Why don't I just get the LLM to visualise this computational graph for me?

### Traversing the computational graph

First, I asked for a function to traverse the graph and build up the information in a data structure:

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
```

In this function we can see that each tensor object has an attribute `.grad_fn`, except the tensors which are themselves parameters, which have no `.grad_fn` but have a `.grad` parameter. Each `.grad_fn` has a `.next_functions` list that contains the pointers to the next objects in the graph.

Then I asked for a function to print the result. I had the bright idea to print it like a file directory structure -- it just seemed like a clear way to present it, YMMV:

```python
def pretty_print_graph(node, prefix='', 
    is_last=True, show_memloc=False):

    name, children = node
    connector = '└── ' if is_last else '├── '

    if show_memloc:
        formatted_name = f"{name[0]}: {name[1]}"
    else:
        formatted_name = name[0]

    print(prefix + connector + formatted_name)

    if children:
        child_prefix = prefix + ('    ' if is_last else '│   ')
        for idx, child in enumerate(children):
            is_child_last = (idx == len(children) - 1)
            pretty_print_graph(child, child_prefix, 
                        is_child_last, show_memloc)
```

Both of these functions are actually the final iterations after I asked for some extra features. Let's apply it to our linear regression example:
```python
pretty_print_graph(traverse_graph(loss.grad_fn))
# └── MeanBackward0
#     └── PowBackward0
#         └── SubBackward0
#             └── AddBackward0
#                 ├── AccumulateGrad
#                 └── MulBackward0
#                     └── AccumulateGrad
```

In this case (but not in general! See later) the graph is a tree, and ends in two leaves called `AccumulateGrad` that represent the parameters `a` and `b` accumulating the gradients. Notice that the `MeanBackward`, `PowBackward` and `SubBackward` are all different operations in the `loss` variable. This implies that there are `torch.tensor` objects representing not only the final variable, but also all of the sub-computations that contributed to it.

Let's create a more complex example:

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
loss = torch.sum(h)
```

Here we have four 2 x 2 matrix-valued parameters (`a`,`b`,`c`,`d`) one matrix datapoint `x` (with `requires_grad=False` to denote it as fixed data), and several intermediate matrices, with addition, componentwise multiplication `*` and matrix multiplication `@`. Let's see what this looks like:

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

Interesting! But when we count up the leaf nodes we find that there are more than the four we expected. This is because some of the parameters enter the graph more than once. There's no way to print the names of variables associated with the functions (Pytorch doesn't care about the variables -- it only cares about the objects in memory, some of which aren't even bound to variables) but we can print a shortened hex string for the memory location. This makes it clear that some of the leaves of the graph are pointing to the same place.

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
# Variable names added by me
```

A neural network gradient graph is not, in general, a tree -- which is intuitive when we think of vast, densely-connected networks. So printing it like this can mislead us if we don't know better. But... this gives us some insight into the final piece of the puzzle -- the need for `.zero_grad()`! 

The back-propagation algorithm also traverses the graph in a very similar way to our python function -- it too sees something that looks like a tree. What happens when the algorithm visits the same parameter leaf node twice on different branches? It can't see the whole graph -- it doesn't know it already visited that node -- so how does it avoid getting the wrong answer? It just adds the gradient to whatever it finds there! That finally explains why gradients accumulate by default, and have to be explicitly zeroed after each iteration!

For example, the gradient of `h` with respect to parameter `a` is the sum of its gradient via both branches. It looks like this:

```python
loss.backward()
a.grad
# tensor([[2., 2.],
#         [2., 2.]])
```

The twos are there because the parameter is just linked to `loss` by addition, which would just be a matrix of ones, but it appears twice in the computational graph.

At the end of this week, I still haven't used Pytorch to fine-tune any billion-parameter language models, or train any vision models, or even classify digits. But at this point I feel like I have drunk deep from its well of secrets, and I am suffused by a sense of connectedness to the mycelial tendrils of the graph itself. I am a master of the gradients. Why would I want to spoil this feeling by actually training a model on data?