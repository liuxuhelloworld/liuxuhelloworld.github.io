# What is AI?
human vs rational? thought vs behavior?
- acting humanly, the Turing test approach
- thinking humanly, the cognitive modeling approach
- thinking rationally, the laws of thought approach
- acting rationally, the rational agent approach

The rational agent approach to AI has two advantages over the other approaches. First, it is more general than the laws of thought approach because correct inference is just one of several possible mechanisms for achieving rationality. Second, it is more amenable to scientific development. The standard of rationality is mathematically well defined and completely general. We can often work back from this specification to derive agent designs that provably achieve it, something that is largely impossible if the goal is to imitate human behavior or thought processes.

The rational agent approach to AI has prevailed throughout most of the field's history. In the early decades, rational agents were built on logical foundations and formed definite plans to achieve specific goals. Later, methods based on probability theory and machine learning allowed the creation of agents that could make decisions under uncertainty to attain the best expected outcome. In a nutshell, AI has focused on the study and construction of agents that do the right thing. What counts as the right thing is defined by the objective that we provide to the agent. This general paradigm is so pervasive that we might call is the **standard model**. 

We need to make one important refinement to the standard model to account for the fact that perfect rationality, always taking the exactly optimal action, is not feasible in complex environments. The computational demands are just too high. However, perfect rationality often remains a good starting point for theoretical analysis.

The standard model has been a useful guide for AI research since its inception, but it is probably not the right model in the long run. The reason is that the standard model assumes that we will supply a fully specified objective to the machine. For an artificially defined task such as chess or shortest-path computation, the task comes with an objective built in, so the standard model is applicable. As we move into the real world, however, it becomes more and more difficult to specify the objective completely and correctly. It is particularly problematic in the general area of human-robot interaction, of which the self-driving car is one example.

The problem of achieving agreement between our true preferences and the objective we put into the machine is called the **value alignment problem**: the values or objectives put into the machine must be aligned with those of the human.

# The History of AI
One quick way to summarize the milestones in AI history is to list the Turing Award winners: Marvin Minsky (1969) and John McCarthy (1971) for defining the foundations of the field based on representation and reasoning; Ed Feigenbaum and Raj Reddy (1994) for developing expert systems that encode human knowledge to solve real-world problems; Judea Pearl (2011) for developing probabilistic reasoning techniques that deal with uncertainty in a principled manner; and finally Yoshua Bengio, Geoffrey Hinton, and Yann LeCun (2019) for making deep learning (multilayer neural networks) a critical part of modern computing.

- the inception of artificial intelligence (1943-1956)
- early enthusiasm, great expectations (1952-1969)
- a dose of reality (1966-1973)
- expert systems (1969-1986)
- the return of neural networks (1986-present)
- probabilistic reasoning and machine learning (1987-present)
- big data (2001-present)
- deep learning (2011-present)