---
title: Does Multi-Token Teacher Forcing Work?
tags:
date: 2026-04-21
---
Circuit discovery is powerful: it allows us to identify the components that influence an output the most, given a certain input, and a metric of causality. 

However, despite its strengths, circuit discovery is hobbled by its design setup, which only measures causal effects over the very next token. But in order to say meaningful things about how something affects model generation, we have to somehow quantify its effects over a trajectory of words. Putting it another way, it seems rather contrived to limit the extent of our knowledge about language models to what it can express in the next token. 

Let's think about some ways we can overcome this issue. 

---
###### The naive method: teacher-forcing
The simplest thing to do would be to keep extending the single-token counterfactual patching paradigm _ad infinitum_. A single token at position $i$ gives us the indirect effect of edge $(u,v)$ on a certain importance metric $m$, over a set of inputs $x=\{x_1, x_2 \cdots x_i\}$: 

$$IE(u,v) = m(x^*|\text{do}(u,v)^* \leftarrow (u,v)) - m(x^*)$$

Fitting in the next token, the first generated token $y_1 = x_{i+1}$, means that we include it in the set of inputs, thus extending it: $x'=\{x_1, x_2 \cdots x_i, y_1\}$. We then calculate the edge's indirect effect over this new set $x'$.

Let's take a step back. What are we doing? Multi-token generation zooms in on the possibility of generating $y_1$ and $y_2 | y_1$ and $y_3 | (y_2, y_1)$... it's like a series of AND gates. 

Through teacher-forced counterfactual discovery, what we have is actually a series of branching points where edge $(u, v)$ could make a difference: 
• at token $x_i^*$, could it have steered the conversation away from corrupt continuation $y_1^*$ to clean continuation $y_1$?
• at token $y_1^*$, given that token $y_1^*$ was *already generated*, could it have steered the conversation away from corrupt continuation $y_2^*$ to clean continuation $y_2$?
.....

In my mind, this looks like a tree with a main branch, where we measure the "impact" a counterfactual edge continuously siphons from the tree, over many steps. 

![[Pasted image 20260421172320.png]]



