# Resources: The Training Loop

---

## The best single resource

**Andrej Karpathy — "The spelled-out intro to neural networks and backpropagation: building micrograd"**
https://www.youtube.com/watch?v=VMj-3S1tku0
A 2.5-hour hands-on session where Karpathy builds a working autograd engine (automatic differentiation / backpropagation) from scratch in ~150 lines of Python. The most thorough, accessible explanation of backprop that exists. If you watch one resource from this module, make it this one. No prerequisites beyond basic Python.

**Karpathy's micrograd repo (the code from the video)**
https://github.com/karpathy/micrograd
The complete source code. Readable in an afternoon — the core engine is under 100 lines.

---

## Visual explainers

**3Blue1Brown — "Backpropagation calculus" (Neural Networks, Chapter 4)**
https://www.youtube.com/watch?v=tIeHLnjs5U8
A 10-minute visual walkthrough of how gradients flow backwards through a network. The animations make the chain rule intuitive without requiring calculus fluency.

**3Blue1Brown — "What is backpropagation really doing?" (Chapter 3)**
https://www.youtube.com/watch?v=Ilg3gGewQ5U
The conceptual motivation before the math. Explains why gradient descent works by visualising how nudging parameters affects the loss.

---

## Written explainers

**"Yes you should understand backprop" — Andrej Karpathy**
https://karpathy.medium.com/yes-you-should-understand-backprop-e2f06eab496b
A short essay arguing why understanding backprop matters even for practitioners who use autodiff frameworks. A good framing for why this module exists.

**"A Step-by-Step Backpropagation Example" — Matt Mazur**
https://mattmazur.com/2015/03/17/a-step-by-step-backpropagation-example/
A worked numerical example walking through backprop on a tiny network by hand. Useful for building intuition about what the numbers actually do.

---

## Deeper: the full Neural Networks: Zero to Hero course

**Andrej Karpathy — Neural Networks: Zero to Hero (full playlist)**
https://karpathy.ai/zero-to-hero.html
The complete lecture series that includes micrograd plus language models. The training loop, backpropagation, and optimisation videos are among the best ML education content available anywhere. Free. Strongly recommended if you want to go from concepts to implementation.

---

## Reference

**fast.ai — Practical Deep Learning for Coders (Lesson 1 & 2)**
https://course.fast.ai/
Lessons 1–2 cover the full training loop in practice using PyTorch, including batches, epochs, learning rate schedules, and validation. Code-first, but concepts are explained before implementation.
