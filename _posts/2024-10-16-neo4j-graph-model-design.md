As with any database, the data model that you design is important in determining the logic of your queries and the structure of data in storage.

## write your queries first
Knowing the kinds of questions and queries you want to ask for your data is a great way to determining the structure of your data model.

## property vs relationship
One of the earliest decisions you may encounter is whether to model something as a property on a node or as a relationship to a separate node. For example, consider whether the genre of a movie should be a property or a relationship? The best option highly depends on the types of queries you intend to run against your data.

## time-bound data and versioning
One way to model time-specific data and relationships in by including data in the relationship type.