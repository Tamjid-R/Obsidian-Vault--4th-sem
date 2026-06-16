# Data Centre Networks

A **Data Centre Network (DCN)** is a specialized, high-capacity infrastructure that interconnects hundreds or thousands of servers within a data centre and connects them to the broader Internet.

## Technical Details
Data centre networks are designed to handle massive amounts of internal traffic (East-West traffic) and traffic between the data centre and the outside world (North-South traffic).

- **Hierarchy:** 
    - **Servers:** Thousands of individual computers (hosts) that store data and run applications.
    - **Top-of-Rack (ToR) Switches:** Switches located at the top of each rack that connect all servers in that rack.
    - **Aggregation/Core Switches:** High-speed switches that interconnect the ToR switches.
- **Redundancy:** Multiple paths between servers are common to ensure high availability and prevent bottlenecks.
- **Virtualization:** Heavily used to allow multiple virtual machines to share the same physical hardware while appearing as distinct hosts on the network.
- **Interconnection:** Large content providers (like Google, Microsoft, Amazon) build their own private data centre networks and connect them to the Internet, often bypassing Tier-1 ISPs to reduce latency.

---

## Feynman Explanation
Imagine a **Data Centre** is like a **Giant Library** that holds all the information in the world.
- The **Servers** are the bookshelves where the books (data) are stored.
- The **Data Centre Network** is the system of hallways, carts, and librarians that move the books around.
- Inside the library, the hallways are extremely wide and organized so that librarians can grab a book from any shelf and bring it to another shelf instantly (**Internal Network**). 
- There is also a front door where people from the city can come in to request a book (**Internet Connection**). 
- Because this library is so big and busy, the hallways (the network) have to be incredibly fast and well-managed so that thousands of people can get their books at the same time without bumping into each other.

## Examples
- **Google Data Centres:** Interconnect thousands of servers to handle search queries, YouTube videos, and Gmail.
- **Amazon Web Services (AWS):** Provides the infrastructure for millions of websites by networking servers across the globe.
- **Internal Synchronization:** When you save a file to the cloud, the data centre network moves that file across several servers to make copies (backups) instantly.

---

Q. What is a data centre network?
ANS: A data centre network is a high-performance infrastructure that interconnects large numbers of servers (often tens of thousands) within a single facility. It facilitates high-bandwidth communication between servers (for distributed processing and storage) and provides a gateway for these servers to communicate with external users over the Internet.

---
*See also:* [[Access Networks]], [[Chapter 1 Summary]]
