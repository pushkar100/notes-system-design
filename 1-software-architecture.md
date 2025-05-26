# Software architecture

[toc]

## Who is a software architect

> Developers know what CAN be done

> Architects know what **SHOULD** be done

An architect makes sure the software design implements the requirements of the system. He is less interested in the implementation details such as the best loops to use, performing I/O efficiently, libraries to use to perform an operation, and so on.

An architect looks at things from a **MACRO** level. His role is more about infusing the technology with a requirement.

Baseline requirements:
1. **Fast** 
2. **Secure**
3. **Reliable**
4. **Easy to maintain**

Fast can mean different things in different settings. Example, satellite telemetry being fast means much more than a regular information system telemetry of a end user. Similarly, reliability of a mission critical system is not the same as that of a customer support chatbot that needs to work only during core office hours.

## Architects must code!

**Architects must code** but not in the traditional sense i.e they must not be a developer on the projects they have architected with tasks assigned to them

What kind of coding must architects do? **Proof of concept / feasibility coding**. Examples:
* Testing out if a dependency injection package can work for solution that would need that
* Whether RDBMS or NoSQL works best. For this, one should have a small POC where they try out both

The following aspects are important to consider:
1. **Architect's trustworthiness**: If you provide un-implementable approaches to developers then they cannot follow through then that is is bad (Ex: Choosing a DB solution that does not fit)
2. **Support the developers**: While developers implements the project, developers should be able to unblock them if needed, for which they need to know how to code and what they are building and if the direction of the implementation matches the architecture
3. **Respect**: When you are hands-on in the way described above and are spending time discussing actively with the development team, your respect increases and people listen to what you have to say instead of seeing you as arrogant, aloof, and so on

Career path to an architect depends on **experience**. Different companies have different paths. For example, developer to architect in small companies where new developers get hired, senior developer to architect, tech lead to junior architect, analyst to architect, and so on.

## An architect's mindset 

### Understand the business

**An architect must understand the business** before working on the architecture. What are the business' strengths and weaknesses, who are its competitors. We cannot solely have a developer's mindset; they do not need to know much about the business. The reason is that only when you understand the biz can you focus on what to build and how to architect it.

### Goals of a system

An architect must find out the **goals** of a system. Goals are not requirements which essentially define what a system should do.

1. Goals are ***not*** what a "system should do"
2. **Goals describe what effect a system has on the organisation**
	* These are usually described by the client (but not always as sometimes the client does not know what system they actually need to solve their problems)

Examples of goals:
* HR system: Goal is to streamline the recruitment process and hire more people faster
* Mobile flash sales: Generate quick revenue stream and attract investors

### Listen to your client's clients

Sometimes, when developing systems, it is important to know who your client's clients are and tailor the system to meed their needs.

As an example, your client might work with users who are viewing telemetry data on a dashboard. If the network is down, it is common to show an error message on the screen. However, the end users would like to know the most recent data but not actively modify it. Hence, you introduce a caching layer - this makes them happy! This is what listening to your client's clients means

**Note**: Client's client need not be an external user. It can be another team in the same organisation that includes your client's colleagues, and so on. Be sure to identify them correctly!

### Use the right language

> Always keep in mind what is the thing that really matters to the person you are talking to.

Examples: 
```
Talking to a project manager
----------------------------
Cares about: Project success
Avoid: "This is the latest and greatest pattern, 
and we'll be the first to test it out! 
We can write a blog post about it later.
Use: "This new technology can help us write code twice as fast,
so we can cut our schedule and budget accordingly"
```

```
Talking to a team leader
------------------------
Cares about: Programming
Use: "Have you heard about the latest angular versioin? 
We are going to use it!"
```

```
Talking to a CEO
----------------------------
Cares about: Financial bottom line
Avoid: Technical buzzwords
Use: "The architecture I have designed will ensure 
the continuity of business, and will be able to cope with 
the high loads expected during Black Friday sales."
```

**Bottom line**: Try to be in the shoes of the person you are talking to, not yours, and then show him how your work contributes to his interests

## The architecture process

**TL;DR**
1. Understand the system requirements
2. Understand the non-functional requirements
3. Map the components
4. Select the technology stack
5. Design the architecture
6. Write the architecture document
7. Support the team

### Understand the system requirements

* System requirements = **What the system should do** 
* System requirements are usually set after figuring out the system's goals.

This process is usually done by a system analyst or a product manager. 

Your responsibility as an architect:
1. Understand the requirements in the initial meeting
2. Set dates for subsequent meetings on the same subject for deeper understanding/clarity

### Understand the non-functional requirements

* NFR define the **technical and service level attributes**
	* Ex: Number of users, volume, load, performance
* This is not always known to the analyst, PM, or client
* NFRs are **more important to architects than regular requirements**
	* So many architectural elements can be affected by NFRs
	* Do not start architecting until you know exactly the NFRs

### Map the components

Represents the **tasks** of the system

Two goals:
1. Understand the system functionality
2. Communicate your understanding to the client

This is a **non-technical exercise**. We **identify software units and define the capabilities** they must possess.

### Select the technology stack

This process is about **selecting the platform** on which the system will be based.

There are three technologies to consider:
1. Frontend (Ex: React)
2. Backend (Ex: Python, Java Springboot)
3. Data store (Ex: PostgreSQL or Firestore (NoSQL, serverless))

Selection must be very **rational**. A wrong selection can lead to the failure of the entire system!

### Design the architecture

Heart of your work. Glue all the information from the steps earlier and design a system that is fast, reliable, secure, and easy to maintain (i.e the four baseline requirements of a system)

This process gives us the **building blocks** for the architecture but it is not formalised yet!

### Write the architecture document

* The document describes the whole process above that you have been through
* Gives the developers and management a full picture of the system you is going to be built

Good architecture document:
1. Describes the process and architecture
2. Must be relevant for all participants (CEO, CIO, Developer, ...) (Find great value in it)
	* Maximise its value

### Support the team

* Writing the document is not the end of the architect's role
* An architect must support the developers implementing it
	* There will be dilemma's regarding it - Help resolve it
	* Architecture may change - Your involvement and guidance is needed

### Additional steps

It is important to know who is going to be involved in which steps.
For example, clients and analysts need to provide their inputs in the non-functional requirements gathering phase. Similarly, developers can be made a part of the architecture discussions or planning early on. This provides faster feedback on the feasibility of it and they become ambassadors for the project.

## Requirements

Requirement is basically a user wanting to "achieve something". Software helps him to achieve this "something".

Essentially, whatever the **user needs becomes requirements**.

Examples:
* I want to apply filters on my photos
* I need to communicate with my friends easily

These **requirements are never left in such high level** as in the example above. During the development cycle, they become more detailed!

### Two types of requirements

1. **Functional requirements** (***"What should the system do?***")
	* Business flows
	* Business services
	* User interfaces

2. **Non-functional requirements** (***What should the system deal with?***)
	* Performance
	* Load
	* Data volume
	* Concurrent users
	* SLA (Service Level Agreements)

Both the types of requirements are important. We must not architect a system without knowing exactly each of them.

For an architect, the *non-functional requirements are extremely crucial* though! For example, imagine developing a simple CRUD app with a simple REST API only to learn that the client will be sending hundreds of megabytes of data in the request! Such huge amounts of data will no longer work well with a simple REST API. We need a more complex system!

## Application types

Decided after requirements are set!

Common types:
1. **Web apps**
	* Returns HTML and the page functions with a bit of JS and is styled with CSS
	* Best suited for: UI, user initiated actions, large scale, short, focused actions, request response based
2. **Web APIs**
	* Ex: `/v1/users/17`
	* Returns data not HTML, very accessible
	* Best suited for: Data retrieval and storage, client initiated actions, large scale, short, focused actions, request response based
3. **Mobile**
	* Runs on mobile, works usually with web APIs
	* Best suited for: UI, Front end for web API, location 
4. **Console**
	* No fancy UI
	* Requires technical knowledge
	* Limited interaction
	* Best suited for: Long running processes or short focused actions
5. **Service**
	* No UI, managed by a service level manager
	* Best suited for: Long running processes like monitoring, processing files, etc
6. **Desktop**
	* Run mainly on the PC (Ex: word processing)
	* Great UI, might connect to the cloud (not very popular now)
	* Architects might never get to work on this!

Other types of applications: `*-as-a-service` applications. For example, AWS has Lambda which is a function-as-a-service offering. Typically, such apps are called serverless.

* Application type should be **set early**
* Application type for a system **can be more than one**. Ex: Web app + Web API

## Selecting a technology stack

Super important decision. Why?
1. It is **irreversible**
2. It can be an emotional decision. This must be avoided. **Be rational** all the time!

A rational decision is:
* Made with a clear mind
* Heavily documented 
* A group effort (Just because you are an architect, decisions cannot be taken alone!)

Three main categories of technology:
1. Frontend
2. Backend
3. Data store

### General considerations

1. **Can perform the task**
2. **Community**: Best way to find out is to go to stackoverflow and see how many tags that technology has got and how recently
3. **Popularity**: Go to Google trends to compare visually the tools against each other and their popularity growth

### Backend technology

Server for web apps and web APIs

Common options currently in the market:
1. .NET
2. Java
3. NodeJS
4. PHP
5. Python

Other mentions: Go and Rust

| Tool | App types | Type system | Cross platform | Community | Performance | Learning curve |
|--|--|--|--|--|--|--|
| .NET | All | Static | No | Large | Ok | Long |
| .NET core | Web apps & APIs, console, service | Static | Yes | Medium and growing rapidly | Great | Long |
| Java | All | Static | Yes | Huge | Ok | Long |
| NodeJS | Web Apps & APIs | Dynamic | Yes | Large | Great | Medium |
| PHP | Web Apps & APIs | Dynamic | Yes | Large | Ok | Medium |
| Python | All | Dynamic | Yes | Huge | Ok | Short |

* NodeJS: Good for I/O
* Java: Good for multi-threading
* PHP: Was good but it is not designed well (Newer versions try to fix this)
* Python: Universal language. Used extensively in ML. Slower performance 

### Frontend technology

#### For Web apps:
1. Angular
	1. Full blown framework
	2. Long learning curve
2. React
	1. UI-centric library
	2. Short learning curve

#### For mobile apps:
1. Native apps (Swift with Objective-C/Android with Java): full access to OS features and exceptional UX
2. Hybrid: Thin wrapper around HTML, JS, CSS. Limited access to OS features and inferior UX
3. Cross platform (React Native/Flutter): Always catching up with the latest native versions and has a good UX but with limitations

### Data store

Common options:
* RDBMS (which typically uses SQL)
* NoSQL

#### SQL 
1. Stores data in tables
2. Tables have concrete set of columns

#### SQL database transactions
1. Transactions are atomic
2. ACID
	1. Atomicity
	2. Consistency
	3. Isolation
	4. Durability

* *Good for structured data*
* *Data amount is not huge*

#### NoSQL
1. Emphasis on scale and performance
2. Schema-less
3. Data usually stored in JSON format

#### NoSQL transactions
1. Eventual consistency
2. Data can be temporarily inconsistent

* *Good for un-structured data*
* *Data amount is huge!*

## System quality attribute

Quality attributes describe **what technical capabilities fulfil the non-functional requirements**.

For example, NFR: "System must work under heavy load, but should not waste money on unused resources". Quality attribute for the above will be "scalability".

```
NFR (What the system should deal with)
|
|
-> Quality attributes (What technical capabilities of the system address the NFR?)
|
|
-> Design in the architecture
```

Common quality attributes:
1. Scalability
2. Manageability
3. Modularity
4. Extensibility
5. Testability

### Scalability

* **Definition**: Adding more computing resources without any interruption
* **Keywords**: Load
* **Example scenario**: System works fine for regular usage but we need to ramp up for a spike during Black Friday sales

Two types of scalability:
* **Scale up** (vertical): Make changes to non-scalable parts of the code and add more power to the VM (Ex: More CPU power, memory, storage, ...)
* **Scale out** (horizontal): No code changes. Add more VMs without increasing its CPU cores or memory. Notify the load balancer about this VM. 

***Scale out is better than*** scale up because:
1. **Redundancy** (or single point of failure): More VMs protect against a single VM crashing rendering the whole system broken
2. **Limitations**: There are practical limits to how much CPU power or memory can be increased in a system!

### Manageability

* **Definition**: Know what is going on in an application and take action at any given time
* **Keywords**: Monitoring, alerting, error reports, etc
* **Example scenario**: A mass of requests are received and the response time starts to degrade. A monitoring system records this degradation and the operator can then act upon it (Ex: add more VMs)

A manageable application will constantly report its status to a monitoring agent. This agent reports it a management console (Ex: dashboard) 

**Thumb rule**: *How to know if your system is manageable?*
* If the end user is the one reporting problems in your applications, it is not manageable
* If the system itself reports problems that have occurred or can occur and we can act on it, the system is manageable

### Modularity

* **Definition**: A system that is built from *small, well-defined* building blocks that can be changed or replaced without affecting the whole system
* **Keywords**: Single responsibility, modules, independent components, interface, etc
* **Example scenario**: Application to read data from an API and save it in the database

A modular application is one where there are independent components that when changed do not affect other components. For example, if we have a single module to read data from an API and to save it in the DB then a change in the API would mean changing this module i.e not only is read from API being affected, so is write to DB. In a modular system, there will be two components for this example: (1) Read API data and (2) Save data to DB. Hence, if the API is changed, only the component reading it needs to change.

### Extensibility

* **Definition**: A system whose functionality can be extended without modifying existing code. *Future enhancements become easier!*
* **Keywords**: Dependency injection, factory pattern, strategy pattern, inheritance, composition, plugins, etc
* **Example scenario**: Reading data from a new file format, CSV, in addition to the already supported XML and JSON

When a system is not extensible, we need to modify existing code. Example: Adding a case to a switch-case or an if-else statement
```java
switch (format) {
	case "xml":
		...
	case "json":
		...
	case "csv": 
		// NEW ADDITION
		// HARDER TO MAKE CHANGES
}
```

A system that is extensible can have a factory or a builder. Example: Adding a new formatter class instead of an if-else clause!
```java
String formatQuery(string format, string data) {
	Iformatter formatter = GetFormatter(format); // format = "csv", "json", ...
	return formatter.format(data);
} // This code stays unchanged if a formatter is created for a new (future) format
```

### Testability

Types of testing:
1. Manual
2. Unit
3. Integration

Testability only applies to unit and integration testing since this is what is automated!

**unit test example**
```js
function add(x, y) {
	return x + y
}

test("add", () => {
	// Arrange
	const x = 5
	const y = 10
	const expectedResult = 15

	// Act
	const actualResult = add(x, y)
	
	// Assert
	expect(actualResult).toEqual(expectedResult)
}
```

**Integration test example**
Combines multiple units that work together. Ex: Multiple units working together to finally write some data to the database. We test what is finally written to the database

* **Definition**: A system whose functionality can be extended without modifying existing code. *Future enhancements become easier!*
* **Keywords**: Single responsibility principle (SRP), Independent methods, modules
* **Example scenario**: Reading data from a new file format, CSV, in addition to the already supported XML and JSON

***It is difficult to test (especially unit test) if modules are not independent! SRP matters!***

Example: If add two numbers also checks if the numbers are positive then there is a problem: We have a *tough time isolating* the add functionality from the positive integer check functionality! *Solution: Make each check in separate methods*.

Example:
```js
// Before:
function add(x, y) {
	if (x >= 0 && y >= 0) {
		return x + y
	}
}
// --> Testing add(): Need to test add functionality + positive integer check

// After:
function checkPositiveIntegers(...args) {
	return args.filter(integer => integer < 0).length > 0
}
function add(x, y) {
	if (checkPositiveIntegers(x, y)) {
		return x + y
	}
}
// --> Testing checkPositiveIntegers(): Need to test only positive integers
// --> Testing add(): Need to test only addition
```
