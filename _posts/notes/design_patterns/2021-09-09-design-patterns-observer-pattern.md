The observer pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically. 

![observer-1](/assets/images/design_patterns/observer-1.jpg)

如果你理解现实生活中的报纸订阅和退订服务，那么就很好理解observer pattern.

The observer pattern provides an object design where subjects and observers are **loosely coupled**. When two objects are loosely coupled, they can interact, but they have very little knowledge of each other. The only thing the subject knows about an observer is that it implements a certain interface (the Observer interface). Because the only thing the subject depends on is a list of objects that implement the Observer interface, we can add new observers whenever we want. We never need to modify the subject to add new types of observers.

subject如何将新的信息给到observers呢？有两种方式：
- push，即subject通知observers有新信息，同时将新信息推送给observers
- pull，即subject通知observers有新信息，但只是通知，由observers自己向subject拉取新信息

push和pull各有优劣，哪一种方式更合适需要依据具体问题来权衡。

应用示例：
![observer-2](/assets/images/design_patterns/observer-2.jpg)