
This framework glosses over the recommended step-by-step phases I should go over during the interview. 

### 1. Requirements. 

**Functional Requirements**: These are the core features of the system and it should be the very first thing I discuss with my interviewer. 

- We gotta ensure we keep this list compact and targeted. We only want to *identify and prioritize the top 3* requirements of the system. Having a very long list of requirements will actually hurt us more than do us good. 

- It is encouraged to probe my interviewer (ask clarifying questions like "Does the system need to do X?)

- FRs as we know are typically written down with the following format:
	- "Users/Clients should be able to ..."


**Non-Functional Requirements**: Statements about the quality of the system that are important to the users. 

- NFRs as we know are typically written down with the following format:
	- The system should be able to ..."

- If possible, these NFRs should also be quantified and specific as to where it should be located. 
	- E.g. It is better for us to put "the system's search function should be low latency (500ms)" rather than "the system should be low latency".

- Some important NFRs worth considering (only pick 3-5):
	- CAP Theorem, Environment Constraints, Scalability, Latency, Durability, Security, Fault Tolerance, Compliance.

>*NOTE*: Uneccessary to perform exact calculations (DAU, QPS, etc.), rough estimations will be fine.

### 2. Core Entities

The core entities that my APIs will communicate with and that my system will persist in the data model. 

Start will a small, core list of the absolute core neccessary entities in the system. Once we reach Step 3, we can start to have a better idea of the columns / fields for each entity as well. 

- E.g. Twitter's core entities would be:
	- User
	- Tweet
	- Follow

- Guiding questions to help us identify our core entities:
	- Who are the actors in the system and are they overlapping?
	- What are the nouns / resources neccessary to satisfy our FRs?



**What's the difference between entities and components?**
 
- Entities are things that our system stores (these become our DB tables / collections). 
	- Example: User, Video, Follow, Like

- Components are pieces of infrastructure that does something. They can process, store, serve, or move data.
	- Example: API Server, Cache, CDN, Database, Message Queue. 


### 3. API 

Which API Protocol should we use? --> REST (Most common), GraphQL, RPC, WebSockets?

Define the endpoints.


### 4. Data Flow (Optional)

This step is only neccessary if our system involves a long sequence of actions (e.g. data 
processing systems). 

- E.g. A web crawler:
	1. Fetch seed URLs.
	2. Parse HTML
	3. Extract URLs
	4. Store Data
	5. Repeat.



### 5. High-Level Design

Drawing boxes and arrows to represent the different components (basic building blocks: caches, databases, servers, etc.) in our system and how they interact. 

This is typically done using `Excalidraw`, but be sure to confirm with interviewer beforehand.

- Important to not overthink. We want to have a design that --> Satisfies our APIs --> Satisfies our FRs. **TIP**: Go 1 by 1 through our API endpoints and build our design sequentially to satisfy each one. 

- Important to NOT layer on complexity early at all. We need to **focus on a relatively simple design that meets the core FRs**.
	- In the next section (Deep Dives), we layer on complexity to satisfy the **NFRs**. 
	- It is however natural to identify areas where we can add some complexity (adding caches / message queues) in this step, but it's important to just take note of it during this step and only in the Deep Dives section do we actually draw it up. 

- As we are drawing, make sure to be explaining our thought process to the interviewer. As well as the data flow and state changes (basically explain the whole API request and response flow). 
	- When the API request actually hits our DB / Persistence layer, this is when **we start writing down the relevant columns / fields (DO NOT WRITE DOWN ALL FIELDS, JUST WRITE DOWN NECCESARY ONES)** for each entity. Write it down right next to the DB as shown below:

![[TwitterStepOne.png]]


### 6. Deep Dives

Expanding the high level design such that it meets the:

- NFRs
- Any edge cases
- Identify & address any issues and bottlenecks
- Improving the design based on probes from interviewer (listen attentively)

