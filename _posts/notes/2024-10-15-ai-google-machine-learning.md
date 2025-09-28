---
title: Machine Learning
---

In basic terms, ML is the process of training a piece of software, called a model, to make useful predictions or generate content from data.

ML systems fall into one or more of the following categories based on how they learn to make predictions or generate content:
- supervised learning
- unsupervised learning
- reinforcement learning
- generative AI

# Supervised Learning
Supervised learning models can make predictions after seeing lots of data with the correct answers and then discovering the connections between the elements in the data that produce the correct answers. These ML systems are supervised in the sense that a human gives the ML system data with the known correct results.

Two of the most common use cases for supervised learning are regression and classification. A regression model predicts a numeric value. Classification models predict the likelihood that something belongs to a category.

Supervised learning is based on the following core concepts:
- data
- model
- training
- evaluating
- inference

## data
Data comes in the form of words and numbers stored in tables, or as the values of pixels and waveforms captured in images and audio files. We store related data in datasets. Datasets are made up of individual **examples** that contain **features** and a **label**. 

Features are the values that a supervised model uses to predict the label. The label is the answer, or the value we want the model to predict. In a weather model that predicts rainfall, the features could be *latitude*, *longitude*, *temperature*, *humidity*, *cloud coverage*, *wind direction*, and *atmospheric pressure*. The label would be *rainfall amount*.

Examples that contain both features and a label are called **labeled examples**.

A dataset is characterized by its size and diversity. Size indicates the number of examples. Diversity indicates the range those examples cover. Good datasets are both large and highly diverse.

A dataset can also be characterized by the number of its features. Datasets with more features can help a model discover additional patterns and make better predictions. However, datasets with more features don't always produce models that make better predictions because some features might have no causal relationship to the label.

## model
In supervised learning, a model is the complex collection of numbers that define the mathematical relationship from specific input feature patterns to specific output label values. The model discovers these patterns through training.

# Unsupervised Learning
Unsupervised learning models make predictions by being given data that does not contain any correct answers. An unsupervised learning model's goal is to identify meaningful patterns among the data. In other words, the model has no hints on how to categorize each piece of data, but instead it must infer its own rules.

A commonly used unsupervised learning model employs a technique called clustering. Clustering differs from classification because the categories aren't defined by you.

# Reinforcement Learning
Reinforcement learning models make predictions by getting rewards or penalties based on actions performed within an environment. A reinforcement learning system generates a policy that defines the best strategy for getting the most rewards. Reinforcement learning is used to train robots to perform tasks, like walking around a room, and software programs like AlphaGo to play the game of Go.

# Generative AI
Generative AI is a class of models that creates content from user input. Generative AI can take a variety of inputs and create a variety of outputs, like text, images, audio and video. At a high-level, generative models learn patterns in data with the goal to produce new but similar data. To produce unique and creative outputs, generative models are initially trained using an unsupervised approach, where the model learns to mimic the data it's trained on. The model is sometimes trained further using supervised or reinforcement learning on specific data related to tasks the model might be asked to perform.