---
title: Learning to Play
---

# Reinforcement Learning
Operant conditioning inspired an important machine-learning approach called **reinforcement learning**. Reinforcement learning contrasts with the supervised-learning method: in its purest form, reinforcement learning requires no labeled training examples. Instead, an agent, the learning program, performs actions in an environment (usually a computer simulation) and occasionally receives rewards from the environment. These intermittent rewards are the only feedback the agent uses for learning.

While reinforcement learning has been part of the AI toolbox for decades, it has long been overshadowed by neural networks and other supervised-learning methods. This changed in 2016 when reinforcement learning played a central role in a stunning and momentous achievement in AI: a program that learned to beat the best humans at the complex game of Go.

The promise of reinforcement learning is that the agent can learn flexible strategies on its own simply by performing actions in the world and occasionally receiving rewards (that is, reinforcement) without humans having to manually write rules or directly teach the agent every possible circumstance.

Reinforcement learning occurs by having the agent take actions over a series of learning episodes, each of which consists of some number of iterations. At each iteration, the agent determines its current state and choose an action to take. If the agent receives a reward, it then learns something.

A crucial notion in reinforcement learning is that of the value of performing a particular action in a given state. The value of action A in state S is a number reflecting the agent's current prediction of how much reward it will eventually obtain if, when in state S, it performs action A, and then continues performing high-value actions. The goal of reinforcement learning is for the agent to learn values that are good predictions of upcoming rewards (assuming that the agent keeps doing the right thing after taking the action in question). Usually, the process of learning the values of particular actions in a given state typically takes many steps of trial and error.