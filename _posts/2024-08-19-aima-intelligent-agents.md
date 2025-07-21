An **agent** is anything that can be viewed as perceiving its **environment** through **sensors** and acting upon that environment through **actuators**.

An agent's **percept sequence** is the complete history of everything the agent has ever perceived. In general, an agent's choice of action at any given instant can depend on its built-in knowledge and on the entire percept sequence observed to date, but not on anything it hasn't perceived. Mathematicelly speaking, we say that an agent's behavior is described by the **agent function** that maps any given percept sequence to an action.

Internally, the agent function for an artificial agent will be implemented by an **agent program**. It is important to keep these two ideas distinct. The agent function is an abstract mathematical description; the agent program is a concrete implementation, running within some physical system.

## rational agent
A **rational agent** is one that does the right thing. What does it mean to do the right thing? 

Moral philosophy has developed several different notions of the "right thing", but AI has generally stuck to one notion called **consequentialism**: we evaluate an agent's behavior by its consequences. The notion of desirability is captured by a **performance measure** that evaluates any given sequence of environment states. As a general rule, it is better to design performance measures according to what one actually wants to be achieved in the environment, rather than according to how one thinks the agent should behave.

What is rational at any given time depends on four thing:
- the performance measure that defines the criterion of success
- the agent's prior knowledge of the environment
- the actions that the agent can perform
- the agent's percept sequence to date

So a rational agent is: for each possible percept sequence, a rational agent should select an action that is expected to maximize its performance measure, given the evidence provided by the percept sequence and whatever built-in knowledge the agent has.

We need to be careful to distinguish between rationality and omniscience. Our definition of rationality does not require omniscience, because the rational choice depends only on the percept sequence to date. Doing actions in order to modify future percepts, sometimes called **information gathering**, is an important part of rationality.

Our definition requires a rational agent not only to gather information but also to learn as much as possible from what it perceives. The agent's initial configuration could reflect some prior knowledge of the environment, but as the agent gains experience this may be modified and augmented. A rational agent should be autonomous, it should learn what it can do to compensate for partial or incorrect prior knowledge.

## task environment
Task environments come in a variety of flavors, and the nature of the task environment directly affects the appropriate design for the agent program.

PEAS (Performance, Environment, Actuators, Sensors)

Task environment dimensions:
- fully observable vs. partially observable
- single-agent vs. multiagent
- deterministic vs. nondeterministic
- episodic vs. sequential
- static vs. dynamic
- discrete vs. continuous
- known vs. unknown

The hardest case is *partially observable, multiagent, nondeterministic, sequential, dynamic, continuous, and unknown*.