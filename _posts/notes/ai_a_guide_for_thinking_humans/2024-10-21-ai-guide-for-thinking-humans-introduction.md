---
title: Introduction
---

# The Dartmouth Summer
Most people in articifial intelligence trace the field's official founding to a small workshop in 1956 at Dartmouth College organized by a young mathematician named John McCarthy. The term artificial intelligence was McCarthy's invention.

There were lots of interesting discussions but not a lot of coherence. However, the Dartmouth summer of AI did produce a few very important outcomes. The field itself was named, and its general goals were outlined. The soon-to-be "big four" pionners of the field, McCarthy, Minsky, Allen Newell, and Herbert Simon met and did some planning for the future. And for whatever reason, these four came out of the meeting with tremendous optimism for the field.

# Definitions and Methods
The field of AI has focused on two efforts: one scientific and one practical. On the scientific side, AI researchers are investigating the mechanisms of biological intelligence by trying to embed it in computers. On the practical side, AI proponents simply want to create computer programs that perform tasks as well as or better than humans, without worrying about whether these programs are actually thinking in the way humans think. It's hard to define AI, and this may be a good thing. The lack of a precise, universally accepted definition of AI probably has helped the field to grow, blossom, and advance at an ever-accelerating pace.

AI is a field that includes a broad set of approaches, with the goal of creating machines with intelligence. **Deep learning** is one such approach. Deep learning is itself one method among many in the field of **machine learning**, a subfield of AI in which machines "learn" from data or from their own "experiences". But since 2010s, deep learning has risen to become the dominant AI paradigm.

# Symbolic AI
A symbolic AI program's knowledge consists of words or phrases (the "symbols"), typically understandable to a human, along with rules by which the program can combine and process these symbols in order to perform its assigned task.

Advocates of the symbolic approach to AI argued that to attain intelligence in computers, it would not be necessary to build programs that mimic the brain. Instead, general intelligence can be captured entirely by the right kind of symbol-processing program. Symbolic AI dominated the AI field for its first three decades, most notably in the form of expert systems, in which human experts devised rules for computer programs to use in tasks such as medical diagnosis and legal decision-making.

By the mid-1980s, expert systems, symbolic AI approaches that rely on humans to create rules that reflect expert knowledge of a particular domain, were increasingly revealing themselves to be brittle: that is, error-prone and often unable to generalize or adapt when presented with new situations. In short, after a cycle of grand promises, immense funding, and media hype, symbolic AI was facing yet another AI winter.

# Subsymbolic AI
Subsymbolic approaches to AI took inspiration from neuroscience and sought to capture the sometimes-unconscious thought processes underlying what some have called fast perception, such as recognizing faces or identifying spoken words. Subsymbolic AI programs do not contain the kind of human-understandable language. Instead, a subsymbolic program is essentially a stack of equations, a thicket of often hard-to-interpret operations on numbers.

An early example of a subsymbolic, brain-inspired AI program was the perceptron, invented in the late 1950s by the psychologist Frank Rosenblatt. The perceptron was an important milestone in AI and was the influential great-grandparent of modern AI's most successful tool, **deep neural networks**.

Rosenblatt's invention of perceptrons was inspired by the way in which neurons process information. A neuron is a cell in the brain that receives electrical or chemical input from other neurons that connect to it. Roughly speaking, a neuron sums up all the inputs it receives from other neurons, and if the total sum reaches a certain threshold level, the neuron fires. Importantly, different connections from other neurons to a given neuron have different strengths; in calculating the sum of its inputs, the given neuron gives more weight to inputs from stronger connections than inputs from weaker connections. Neuroscientists believe that adjustments to the strength of connections between neurons is a key part of how learning takes place in the brain.

In short, a perceptron is a simple program that makes a yes-or-no decision based on whether the sum of its weighted inputs meets a threshold value.

How can we determine the correct weights and threshold for a given task? Rosenblatt's idea was that the perceptrons should learn on its own via conditioning. The perceptron should be trained on examples, it should be rewarded when it fires correctly and punished when it errs. This form of conditioning is now known in AI as **supervised learning**. During training, the learning system is given an example, it produces an output, and it is then given a "supervision signal", which tells how much the system's output differs from the correct output. The system then uses this signal to adjust its weights and threshold.

Supervised learning typically requires a large set of positive examples and negative examples. Each example is labeled by a human with its category. This label will be used as the supervision signal. Some of the positive and negative examples are used to train the system; these are called the training set. The remainder, the test set, is used to evaluate the system's performance after it has been trained, to see how well it has learned to answer correctly in general, not just on the training examples.

Rosenblatt and others showed that networks of perceptrons could learn to perform relatively simple perceptual tasks; moreover, Rosenblatt proved mathematically that for a certain class of tasks, perceptrons with sufficient training could, in principle, learn to perform these tasks without error.

The fact that a perceptron's knowledge consists of a set of numbers, namely, the weights and threshold it has learned, means that it is hard to uncover the rules the perceptron is using in performing its recognition task. It is not easy to translate these numbers into rules that are understandable by humans. The situation gets much worse with modern neural networks that have millions of weights.

Minsky and Papert pointed out that if a perceptron is augmented by adding a layer of simulated neurons, the types of problems that the device can solve is, in principle, much broader. A perceptron with such an added layer is called a multilayer neural network. Such networks form the foundations of much of modern AI.

# AI spring and AI winter
There is a repeating cycle of bubbles and crashes in the field of AI. The two-part cycle goes like this. Phase 1: News ideas create a lot of optimism in the research community. Results of imminent AI breakthroughs are promised, and often hyped in the news media. Money pours in from government funders and venture capitalists for both academic reserach and commercial startups. Phase 2: The promised breakthroughs don't occur, or are much less impressive than promised. Government funding and venture capital dry up. Start-up companies fold, and AI reasrch slows. This pattern became familiar to the AI community: "AI spring", followed by overpromising and media hype, followed by "AI winter". This has happend, to various degrees, in cycles of five to ten years.

Marvin Minsky pointed out that in fact AI research had uncovered a paradox: "Easy things are hard." The original goals of AI, computers that could converse with us in natural language, describe what they saw through their camera eyes, learn new concepts after seeing only a few examples, are things that young children can easily do, but surprisingly, these "easy things" have turned out to be harder for AI to achieve than diagnosing complex diseases, beating human champions at chess and Go, and solving complex algebraic problems.

# Multilayer Neural Networks
A multilayer network can have multiple layers of hidden units, networks that have more than one layer of hidden units are called deep networks.

Unlike in a perceptron, a unit here doesn't simply "fire" or "not fire" (that is, produce 1 or 0) based on a threshold; instead, each unit uses its sum to compute a number between 0 and 1 that is called the unit's "activation." If the sum that a unit computes is low, the unit's activation is close to 0; if the sum is high, the activation is close to 1.

In general, it's hard to know ahead of time how many layers of hidden units are needed, or how many hidden units should be included in a layer, for a network to perform well on a given task. Most neural network researchers use a form of trial and error to find the best settings.

There is a general learning algorithm for multilayer neural networks, which is called **back-propagation**. As its name implies, back-propagation is a way to take an error observed at the output units and to "propagate" the blame for that error backward so as to assign proper blame to each of the weights in the network. This allows back-propagation to determine how much to change each weight in order to reduce the error. Learning in neural networks simply consists of gradually modifying the weights on connections so that each output's error gets as close to 0 as possible on all training examples.

Back-propagation will work (in principle at least) no matter how many inputs, hidden units, or output units your neural network has. While there is no mathematical guarantee that back-propagation will settle on the correct weights for a network, in practice it has worked very well on many tasks that are too hard for simple peceptrons.

# Symbolic vs Subsymbolic
Why not just use symbolic systems for tasks that require high-level language-like descriptions and logical reasoning, and use subsymbolic systems for the low-level perceptual tasks such as recognizing faces and voices? To some extent, this is what has been done in AI, with very little connection between the two areas. Each of these approaches has had important successes in narrow areas but has serious limitations in achieving the original goals of AI. While there have been some attempts to construct hybird systems that integrate symbolic and subsymbolic methods, none have yet led to any striking success.

# Machine Learning
Inspired by statistics and probability theory, AI researchers developed numerous algorithms that enable computers to learn from data, and the filed of machine learning became its own independent subdiscipline of AI, intentionally separate from symbolic AI. Machine-learning researchers disparagingly referred to symbolic AI methods as good old-fashioned AI, or GOFAI, and roundly rejected them.

Over the next two decades, machine learning had its own cycles of optimism, government funding, start-ups, and overpromising, followed by the inevitable winters. Training neural networks and similar methods to solve real-world problems could be glacially slow, and often didn't work very well, given the limited amount of data and computer power available at the time.

# Narrow and General, Weak and Strong
The terms narrow and weak are used to constrast with strong, human-level, general, or full-blown AI (sometimes called AGI, or artificial general intelligence). General AI might have been the original goal of the field, but achieving it has turned out to be much harder than expected. A pile of narrow intelligences will never add up to a general intelligence. General intelligence isn't about the number of abilities, but about the integration between those abilities.

Given the rapidly increasing pile of narrow intelligences, how long will it be before someone figures out how to integrate them and produce all of the broad, deep, and subtle features of human intelligence?

In the AI research community there is considerable controversy over what human-level AI would entail? How can we know if we have succeeded in building such a "thinking machine"? Would such a system be required to have consciousness or self-awareness in the ways humans do? Would it need to understand things in the same way a human understands them? Given that we're talking about a machine here, would we be more correct to say it is "simulating thought", or could we say it is truly thinking?

# Could Machines Think?
Could mahchines think? Is there a difference between "simulating a mind" and "literally having a mind"?

Turing suggested the following:"The question, 'Can machines think?' should be replaced by 'Are there imaginable digital computers which would do well in the imitation game?'" In other words, if a computer is sufficiently humanlike to be indistinguishable from humans, aside from its physical appearance or what it sounds like (or smells like or feels like, for that matter), why shouldn't we consider it to actually think? Why should we require an entity to be created out of a particular kind of material (for example, biological cells) to grant it "thinking" status? This is now called the **Turing test**.

# The Singularity
Ray Kurzweil is best known for his futurist prognostications, most notably the idea of the Singularity:"a future period during which the pace of technological change will be so rapid, its impact so deep, that human life will be irreversibly transformed." Kruzweil uses the term singularity in the sense of "a unique event with ... singular implications"; in particular, "an event capable of rupturing the fabric of human history." For kruzweil, this singular event is the point in time when AI exceeds human intelligence.

Kurzweil's ideas were spurred by the mathematician I.J.Good's speculations on the potential of an intelligence explosion:"Let an ultraintelligent machine be defined as a machine that can far surpass all the intellectual activities of any man however clever. Since the design of machines is one of these intellectual activities, an ultraintelligent machine could design even better machines; there would then unquestionably be an 'intelligence explosion,' and the intelligence of man would be left far behind."

Kurzweil bases all of his predictions on the idea of "exponential progress" in many areas of science and technology, especially computers.

Responses to Kurzweil are often one of two extremes: enthusiatic embrace or dismissive skepticism.