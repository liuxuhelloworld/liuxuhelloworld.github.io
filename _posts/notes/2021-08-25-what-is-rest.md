---
title: What Is REST
---

In the early days of web APIs, people spent a lot of time and effort figuring out how to implement the features of previous-generation distributed technologies like CORBA (Common Object Request Broker Architecture) and DCOM (Distributed Component Object Model) on top of HTTP. This led to technologies like SOAP (Simple Object Access Protocol) and WSDL (Web Services Description Language). Experience showed that these technologies were more complex, heavyweight, and brittle than was useful for most web APIs. The idea that replaced SOAP and WSDL was that you could use HTTP more directly with much less technology layered on top.

REST (Representational State Transfer) is the name that has been given to the architectural style of HTTP itself, described by one of the leading authors of the HTTP specifications. HTTP is the reality, REST is a set of design ideas that shaped it. The importance of REST is that it helps us understand how to think about HTTP and its use.

There are a few web APIs that are designed to use only HTTP concepts and no more. The majority of modern APIs uses some subset of the concepts from HTTP, blended with some concepts from other computing technologies. For example, many web APIs are defined in terms of *endpoints* that have *parameters*. *Endpoint* and *parameter* are not terms or concepts that are native to HTTP or REST, they are concepts carried over from Remote Procedure Call (RPC) and related technologies. The term *RESTful* has emerged for web APIs that use more of the native concepts and technologies of HTTP than antecedent technologies did, but also blended in other concepts. Many good APIs have been designed this way, and in fact, this blended style is probably the most common API style in use. HTTP and REST are precisely defined, but the term RESTful is not, it is one of those "I know it when I see it" sort of concepts.

A REST API focuses on the underlying entities of the problem domain it exposes, rather than a set of functions that manipulate those entities.

A significant part of the value of basing your API design on HTTP and REST comes from the uniformity that it brings to your API. In REST, this idea is called the uniform interface constraint.