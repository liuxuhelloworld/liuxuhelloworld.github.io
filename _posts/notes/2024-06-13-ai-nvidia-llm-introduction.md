---
title: LLM Introduction
---

A large language model is a type of artificial intelligence system that is capable of generating human-like text based on the patterns and relationships it learns from vast amounts of data. Large language models use a machine learning technique called deep learning to analyze and process large sets of data, such as books, articles, and web pages.

Large language models (LLMs) are deep learning algorithms that can recognize, extract, summarize, predict, and generate text based on knowledge gained during traning on very large datasets.

# Evolution of Large Language Models
AI systems were historically about processing and analyzing data, not generating it. They were more oriented toward perceiving and understanding the world around us rather than on generating new information. This distinction marks the main difference between *Perceptive* and *Generative* AI, with the latter becoming increasingly prevalent since 2020, or after companies started adopting transformer models and developing increasingly more robust LLMs at a large scale.

The advent of large language models further fueled a revolutionary paradigm shift in the way NLP models are designed, trained, and used.
- pre-transformers NLP, it was mainly marked by models that relied on human-crafted rules rather than machine learning algorithms to perform NLP tasks
- transformers NLP, it was set in motion by the rise of the transformer architecture in 2017
- LLM NLP, it was mainly initiated by the launch of OpenAI's GPT-3 in 2020

The switch from one methodology to another was largely driven by relevant technological and methodological advancements, such as the advent of neural networks, attention mechanisms, and transformers and developments in the field of unsupervised and self-supervised learning.

# Neural Networks
Neural networks (NN) are machine learning algorithms loosely modeled after the human brain. Like the biological human brain, artificial neural networks consist of neurons, also called nodes, that are responsible for all model functions, from processing input to generating output. 

The neurons are further organized into layers, vertically stacked components of NNs that perform specific tasks related to input and output sequences.

When LLMs were first developed, they were based on simpler NN architectures with fewer layers, mainly recurrent neural networks (RNNs) and long short-term memory networks (LSTMs). Unlike other neural networks, RNNs and LSTMs could take into account the context, position, and relationships between words even if they were far apart in a data sequence. Simply put, this meant they could memorize and consider past data when generating output, which resulted in more accurate solutions to many NLP tasks, especially sentiment analysis and text classification.

The biggest advantage that neural networks like RNNs and LSTMs had over traditional, rule-based systems was that they were capable of learning on their own with little to no human involvement. They analyze data to create their own rules, rather than learn the rules first and apply them to data later. This is also known as representation learning and is inspired by human learning processes. Representations, or features, are hidden patterns that neural networks can extract from data.

The ability to extract representations is very much dependent on the number of neurons and layers comprising a network. The more neurons neural networks have, the more complex representations they can extract. That's why, today, most LLMs use deep learning neural networks with multiple hidden layers and thus, a higher number of neurons.

# Transformers
Transformers were first introduced in 2017 in a paper titled "Attention is All You Need." 

Attention mechanisms solved the problem of inadequate context handling by allowing models to selectively pay attention to certain parts of the input while processing it. Instead of having to capture the entire context at once, models could now focus on the most important tokens regarding a specific task. Attention mechanisms would first calculate weights for each word in our input. Attention weights represent the importance of each token, so the more weight a token is assigned, the more important it's deemed. The model would then use these weights to dynamically emphasize or downplay each word as it generates output. So, by enabling models to handle context more effectively, attention mechanisms allowed them to generate more accurate outputs than models based on RNNs and LSTMs. Simultaneously, this new approach to data-processing also allowed transformer-based models to generate outputs more quickly than RNNs and LSTMs models. RNNs and LSTMs need more time to generate output because they process input sequentially. Transformers, on the other hand, process data in parallel, which means they read all input tokens at once instead of processing one at a time. It also means they are able to perform NLP tasks faster than RNNs and LSTMs.

# Unsupervised and Self-Supervised Learning
Unsupervised learning refers to machine learning algorithms finding patterns in unlabeled datasets with no human intervention. Unsupervised learning models use feedback loops to learn and improve their performance. This involves getting feedback on whether a prediction or classification was right or wrong, which the model uses to guide its future decisions. 

Self-supervised learning models don't have feedback loops but rather use supervisory signals to get feedback during training. Self-supervised learning is currently the dominant approach to pre-training large language models and is often recommended to enterprises that want to build their own.

Both unsupervised and self-supervised learning techniques have one key advantage over supervised learning: they rely on the model to create labels and extract features on it own, rather than demanding human intervention. This helps companies train models without time-consuming data lebelling processs or providing human feedback on the model's outputs.

# GPT vs BERT
GPT (Generative Pre-trained Transformer) and BERT (Bidirectional Encoder Representations from Transformers) are both highly advanced and widely used NLP models. However, they differ in their architectures and use cases.

GPT is a generative model that is trained to predict the next word in a sentence given the previous words. This pre-training enables GPT to generate coherent and fluent sentences from scratch, making it ideal for language generation tasks such as text completion, summarization, and question answering.

In distinction, BERT is a discriminative model that is trained to classify sentences or tokens into differnet categories such as sentiment analysis, named entity recognition, and text classification. It is a bidirectional model that considers both the left and right contexts of a sentence to understand the meaning of a word, making it highly effective for tasks such as sentiment analysis and question answering.

In terms of architecture, GPT uses a unidirectional transformer, whereas BERT uses a bidirectional transformer. This means that GPT can only consider the left context of a word when making predictions, while BERT considers both the left and right contexts.

# How to Evaluate LLMs
Evaluating the performance of LLMs is not a straightforward task, and it requires a careful analysis of different factors such as training data, model size, and speed of inference.

The most crucial element in evaluating LLMs is the quality and quantity of the training data used. The training data should be diverse and representative of the target language and domain to ensure that the LLM can learn and generalize language patterns effectively.

Another important factor is the size of the model. Generally, larger models have better performance, but they also require more computational resources to train and run. Therefore, researchers often use a trade-off between model size and performance, depending on the specific task and resources available. It is also worth noting that larger models tend to be more prone to overfitting, which can lead to poor generalization performance on new data.

Speed of inference must also be used in evaluation, especially when deploying LLMs in real-world applications. Faster inference time is desirable as it enables the LLM to process large amounts of data in a timely and efficient manner. Several techniques, such as pruning, quantization, and distillation, have been proposed to reduce the size and imporve the speed of LLMs.

To evaluate the performance of LLMs, researchers often use benchmarks, which are standardized datasets and evaluation metrics for a particular language-related task. Benchmarks enable fair comparisons between different models and methods and help identify the strenghs and weaknesses of LLMs. Common benchmarks include GLUE (General Language Understanding Evaluation), SuperGLUE, and CoQA (Conversational Question Answering).

# Traditional NLP Tasks
- machine translation, translations of text from one language to another
- text summarization, generation of summaries of texts
- sentiment analysis, analysis of text sentiment (e.g., positive, negative, and neutral)
- text classification, classification of texts into categories
- text generation, generation of texts
- question answering, answering questions
- part-of-speech tagging, assignment of part-of-speech tags, like "noun" or "verb", to words or tokens
- named entity recognition, identification and classification of named entities in text, such as people, organizations, and locations