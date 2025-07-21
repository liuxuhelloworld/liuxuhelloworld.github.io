Neo4j is a *native graph database*, which means that it implements a true graph model all the way down to the storage level.

A Neo4j graph database stores nodes and relationships instead of tables or documents.

Nodes are the entities in the graph. 
- nodes can be tagged with *labels*, representing their different roles in your domain (for example, **Person**).
- nodes can hold any number of key-value paris, or *properties* (for example, **name**). 
- node labels may also attach metadata (such as index or constraint information) to certain nodes.

Relationships provide directed, named connections between two node entities (for example, *Person* **LOVES** *Person*). 
- relationships always have a direction, a type, a start node, and an end node, and they can have properties, just like nodes. 
- nodes can have any number or type of relationships without sacrificing performance. 
- although relationships are always *directed*, they can be navigated efficiently in any direction.

A label is a named graph construct that is used to group nodes into sets. All nodes labeled with the same label belongs to the same set. A node may be labeled with any number of labels, including none, making labels an optional addition to the graph.