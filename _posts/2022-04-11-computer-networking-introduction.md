The Internet is a computer network that interconnects billions of computing devices throughout the world. In Internet jargon, all of these devices are called **hosts** or **end systems**.

Hosts are connected together by a network of **communication links** and **packet switches**. There are many types of **communication links**, which are made up of different types of physical media. When one host has data to send to another host, the sending host segments the data and adds header bytes to each segment. The resulting packages of information, known as **packets** in the jargon of computer networks, are then sent through the network to the destination host, where they are reassembled into the original data. A **packet switch** takes a packet arriving on one of its incoming communication links and forwards that packet on one of its outgoing communication links. Packet switches come in many shapes and flavors, but the two most prominent types in today's Internet are **routers** and **link-layer switches**. Link-layer switches are typically used in access networks, while routers are typically used in the network core.

Hosts, packet switches, and other pieces of the Internet run **protocols** that control the sending and receiving of information within the Internet. A **protocol** defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event. The **Transmission Control Protocol (TCP)** and the **Internet Protocol (IP)** are two of the most important protocols in the Internet.

Internet standards are developed by the Internet Engineering Task Force (IEFT). The IEFT standards documents are called **requests for comments (RFCs)**.

The Internet can also be seen as *an infrastructure that provides services to applications*. The applications are said to be **distributed applications**, since they involve multiple hosts that exchange data with each other. How does one program running on one host instruct the Internet to deliver data to another program running on another host? Hosts attached to the Internet provide a **socket interface** that specifies how a program running on one host asks the Internet infrastructure to deliver data to a specific destination program running on another host. This Internet socket interface is a set of rules that the sending program must follow so that the Internet can deliver the data to the destination program.

**Access network**, the network that physically connects an host to the first router on a path from the host to any other distant host. The two most prevalent types of broadband residental access are **digital subscriber line (DSL)** and **cable**.