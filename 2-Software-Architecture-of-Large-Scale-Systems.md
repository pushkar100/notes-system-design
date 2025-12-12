# 💻 Software Architecture of Large Scale Systems

[Course link](https://www.udemy.com/course/software-architecture-design-of-modern-large-scale-systems)

- [Software Architecture of Large Scale Systems](#software-architecture-of-large-scale-systems)
   * [Definition of software architecture](#definition-of-software-architecture)
   * [Levels of abstraction](#levels-of-abstraction)x
   * [Advantages of distributed services](#advantages-of-distributed-services)
   * [Software architecture position in the SDLC](#software-architecture-position-in-the-sdlc)
   * [Challenges of software architecture](#challenges-of-software-architecture)
   * [Gathering requirements](#gathering-requirements)
      + [Importance of gathering requirements](#importance-of-gathering-requirements)
      + [Types of requirements](#types-of-requirements)
         - [Functional requirements](#functional-requirements)
         - [Non-Functional requirements](#non-functional-requirements)
         - [Limitations and boundaries](#limitations-and-boundaries)
      + [Right way to gather functional requirements](#right-way-to-gather-functional-requirements)
         - [Formal way of gathering requirements:](#formal-way-of-gathering-requirements)
         - [Steps to gather requirements:](#steps-to-gather-requirements)
      + [More on non-functional requirements](#more-on-non-functional-requirements)
         - [Non-functional requirements need to be proven](#non-functional-requirements-need-to-be-proven)
         - [Last consideration is feasibility](#last-consideration-is-feasibility)
      + [System constraints ](#system-constraints)
         - [Technical constraints](#technical-constraints)
         - [Business constraints](#business-constraints)
         - [Legal/Regulatory constraints](#legalregulatory-constraints)
         - [Accepting vs pushing back on constraints](#accepting-vs-pushing-back-on-constraints)
   * [Performance](#performance)
      + [Response time](#response-time)
      + [Measuring response time distribution](#measuring-response-time-distribution)
         - [Tail latency in response times](#tail-latency-in-response-times)
      + [Throughput](#throughput)
      + [Identifying performance degradation](#identifying-performance-degradation)
   * [Scalability](#scalability)
      + [Understanding load](#understanding-load)
      + [Intuition for a scalable design](#intuition-for-a-scalable-design)
      + [Note on linear scalability](#note-on-linear-scalability)
      + [Types of scalability](#types-of-scalability)
         - [Vertical scalability](#vertical-scalability)
         - [Horizontal scalability](#horizontal-scalability)
         - [Pros and cons of each scalability choice](#pros-and-cons-of-each-scalability-choice)
         - [Team / Organisational scalability](#team-organisational-scalability)
   * [Availability](#availability)
      + [Formula of availability](#formula-of-availability)
      + [Alternative formula of availability](#alternative-formula-of-availability)
      + [Myth of complete availability](#myth-of-complete-availability)
      + [The 9s of availability](#the-9s-of-availability)
      + [Fault tolerance](#fault-tolerance)
         - [Failure prevention](#failure-prevention)
         - [Types of redundancy](#types-of-redundancy)
         - [Strategies for redundancy](#strategies-for-redundancy)
         - [Failure detection](#failure-detection)
         - [Failure recovery](#failure-recovery)
      + [SLAs SLOs and SLIs](#slas-slos-and-slis)
         - [SLA or Service Level Agreement](#sla-or-service-level-agreement)
         - [SLOs or Service Level Objectives](#slos-or-service-level-objectives)
         - [SLIs or Service Level Indicators](#slis-or-service-level-indicators)
         - [Important considerations](#important-considerations)
   * [APIs](#apis)
      + [Categories of APIs](#categories-of-apis)
      + [API benefits](#api-benefits)
      + [API best practices](#api-best-practices)
   * [Remote Procedure Calls](#remote-procedure-calls)
      + [RESTful APIs](#restful-apis)
         - [REST API quality attributes (Non-functional requirements)](#rest-api-quality-attributes-non-functional-requirements)
         - [REST API endpoints](#rest-api-endpoints)
         - [Best practices for resources](#best-practices-for-resources)
         - [CRUD operations via HTTP verbs](#crud-operations-via-http-verbs)
         - [REST API step by step process](#rest-api-step-by-step-process)
      + [Why is gRPC popular?](#why-is-grpc-popular)
   * [Architectural blocks of large scale systems](#architectural-blocks-of-large-scale-systems)
      + [Load balancer](#load-balancer)
         - [Benefit of using a load balancer](#benefit-of-using-a-load-balancer)
         - [Quality attributes of a load balancer](#quality-attributes-of-a-load-balancer)
         - [Types of load balancers](#types-of-load-balancers)
         - [DNS load balancer](#dns-load-balancer)
         - [Hardware and software load balancers](#hardware-and-software-load-balancers)
         - [Global Server Load Balancing](#global-server-load-balancing)
         - [Common load balancer tools](#common-load-balancer-tools)
      + [Message brokers](#message-brokers)
         - [Quality attributes of message brokers](#quality-attributes-of-message-brokers)
         - [Popular message broker tools](#popular-message-broker-tools)
      + [API gateways](#api-gateways)
         - [Popular API gateways](#popular-api-gateways)
         - [API Gateway vs Load Balancer](#api-gateway-vs-load-balancer)
      + [Content Delivery Networks](#content-delivery-networks)
         - [Popular CDN providers](#popular-cdn-providers)

## 💡 Definition of software architecture

> Software Architecture of a system is a high-level description of the **system's structure**, its **different components**, and **how those components communicate with each other** to fulfil the system's requirements and constraints.

* High-level: 
	* It means it is an abstraction showing the important components
	* Hides implementation details
	* Programming languages or technologies are not a part of Software architecture but implementation
	* Decisions about technologies should be delayed to the very end of our design

* Components are "black boxes" defined by their *behaviour* and *APIs*
	* They themselves maybe complex systems described by their own architecture diagrams (recursive when needed)

* System's requirements
	* What is **should do**

* System constraints
	* What it **should not do**

<a name="levels-of-abstraction"></a>
## 📈 Levels of abstraction

We can talk about software architecture in terms of:
1. Classes / Structs (Low-level)
2. Modules / packages / libraries
3. Services (processes / group of processes) (High-level)

**Services** are what we focus on when explaining the *architecture of large-scale systems* i.e at a high-level. 

Services are potentially placed in a **distributed** network

<a name="advantages-of-distributed-services"></a>
## 🌐 Advantages of distributed services

1. Handle a large amount of requests
2. Process and store very large amounts of data
3. Serve many users every day

Example applications: Ride sharing, Video on Demand (VOD), online video games, investment services and banks, etc.

<a name="software-architecture-position-in-the-sdlc"></a>
## 🔁 Software architecture position in the SDLC

Software development lifecycle consists of the following states (loop):
1. Design
2. Implementation
3. Testing
4. Deployment
