## What is computing?
“Computing is the process of using computer technology to complete a given goal-oriented task. Computing may encompass the design and development of software and hardware systems for a broad range of purposes”

## From distributed computing to cloud
1. Centralized Computing: mainframe computers
2. Distributed Computing
3. Cluster Computing 
4. Grid Computing : preliminary idea of cloud, it was used when the virtualization technology was not mature enough
5. Utility Computing
6. Cloud Computing 

## Distributed Computing

*“A distributed system is a collection of autonomous computers that are interconnected with each other and cooperate, thereby sharing resources such as printers and databases”*

The network acts as a glue among the resources
- Several nodes are connected by the network and share resources

- **Distributed problems** are solved more easily using the means of distributed computing
- In **Computing intensive problems** communications is limited (High Throughput Computing)
- **Data Intensive problems**: computing task deal with a large amount or large size of data

Distributed computing allows for “***scavenging***”(Or *cycle stealing*), this means that, by integrating the computers into a distributed system, the excess computing power can be made available to other users or applications

### Features of Distributed Computing: 
- Fault tolerance
- Resource sharing 
- Load Sharing and Balance
- Scalability
- Performance
### DD Architecture
Interconnect processes running on different CPUs with some sort of communication system. The most famous approaches are the following:
- **Client-server**: The server owns the resources, the client uses them 
- **3-Tier architecture**
- **P2P**: responsibilities are uniformly divided among all machines, known as peers that serves both as client and servers
### Cluster Computing 
- A computer cluster is a group of linked computers, working together closely so that in many respects they form a single computer. 
- The components of a cluster are commonly, but not always, connected to each other through fast local area networks.

There are different kinds of clusters:
- **High Availability Clusters**: In the fail-over configuration, services run in one computing node while the other waits to take over during outages. It is mainly used to add failure resiliency.
- **Network Load Balancing Clusters**:  operate by distributing a workload evenly over multible backend nodes
- HPC Clusters
![[Pasted image 20230929101517.png]]

Remark that the high majority of computers in the TOP500 ranking are clusters

Cluster computing is:
- Cost-effective
- Resilient
- Multi user
- You can cluster togheter several computers with different architectures 
- Scalable

## HPC vs HTC
LInk:: https://www.geeksforgeeks.org/difference-between-high-performance-computing-and-high-throughput-computing/
Link:: [High-throughput computing - Wikipedia](https://en.wikipedia.org/wiki/High-throughput_computing)

HPC is defined as the type of computing that makes use of multiple computer processors in order to perform complex computations parallelly.

HTC is defined as a type of computing that parallelly executes a large number of simple and computationally independent tasks.

In HTC you may expect that its job is isolated, this is not true in the HPC case

In HTC:
- divide the problem up into smaller independent parts;
- get system to process as many of these small parts as possible in parallel
- combine the partial results produced by the system to give the overall result

Obv, data should be partionable in small independent parts, and the partitioning overhead should not be too big

## (Data reduction) Pipeline
A pipeline is a set of processes and tools used to collect raw data from multiple sources, analyse it and present the results in an understandable format.

![[Pasted image 20230929102811.png]]

#### Challenges
- **Computing challenge**
- **Memory challenge**: a huge dataset cannot be loaded on the memory of a single node, it has to be splitted and loaded on different nodes memory  
- **Data challenge**: The amount of collected data is so big that moving it from one location to another is expensive
---
# Lecture 2

## Grid computing 
A GRID computer is a collection of geographically separated resources **(CLUSTERS)** connected by a network a software layer, often called middleware, which transforms a collection of independent resources into a single, coherent, virtual machine

- Every cluster has a scheduler
- There is a metascheduler, you submit your job to it, it accepst your request, finds the best (based on several metrics) cluster to process your job and send it the job. Then the job interact with the scheduler of that cluster
- This is useful for HTC, so for low I/O tasks

Low level services:
High Level Services: 

##### Virtual organization
The idea is that:
- I have a very complex problem to solve
- Me and other actors share computational resources and form a grid computing network, then everybody can use the resources or a fraction of them
- These virtual organizations allows for scavenging or cycle stealing 

#### GRID limitations
- Very rigid environment: all the clusters all over the world are supposed to have the same OS and versions of low level services. This makes the system not easy to scale
- Licensing problems accros different domains
- High level of complexity to use resources efficiently

Due to these limitations, the GRID computing has lost appeal and has been substituted by cloud computing 

## Utility computing
- Pure theoretical concept, CC is the implementation
- It is a service provisioning model in which a service provider makes computing resources and infrastructure available to customers and charges them for specific usage rather than a flat rate
- Based on the concept of outsourcing 

#### Utility computing concepts
- When you are starting to use an utility computing source you sign a Service Level Agreement about which kind of services, security, etc... will be offered
- Pay-per-use Pricing Business Model

## Edge Computing
- Related to the concept of cloud
- A network of micro data centers embedded in the instruments/sensors that store or process critical data locally and push received data to a centralized data center or repository of cloud storage.
- So your coupling data acquisition with the first step of analysis
- This reduces traffic in the central repository
- Data transfer has a cost

## Cloud Computing
It implements Utility Computing as princing model

### EXTRA: difference between cloud and utility computing
- Utility computing is a billing model based on the actual usage of computing resources, while cloud computing is a broader concept that includes the delivery of computing services over the internet. 
- Cloud computing offers a wide range of services, including infrastructure, platforms, and applications, whereas utility computing primarily focuses on consumption-based billing. 
- Cloud computing can include utility computing as one of its billing models, but it goes beyond by providing a complete ecosystem of computing services.

### NIST definition of Cloud
Cloud computing is a model for enabling convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

### Cloud computing features:
- Cloud computing is the on-demand availability of computer system resources, especially data storage (“cloud storage”) and computing power, without direct active management by the user. 
- Clouds may be limited to a single organization (private/enterprise cloud), or be available to many organizations (public cloud), maybe a mixture of the two (hybrid cloud); 
- Cloud implements a pay-as-you-go model based on the concept of infinite resources availability;
- Cloud is tightly coupled to Virtualization and Containerization, Microservicing and Composability

### Cloud computing base concepts:
- **Abstraction**: Cloud computing abstracts the details of system implementation from users and developers.  
- **Virtualization**: Cloud computing virtualizes systems by pooling and sharing resources. 
### NIST model view
![[Pasted image 20231006102009.png]]
Let's anayse in bottom-up order
### Service Attributes
- **On-demand**
	- computing resources are made available to the user on an "as needed" basis
	- it overcomes the common challenge that enterprises encountered of not being able to meet unpredictable, fluctuating computing demands efficiently
- **Broad Network Access**
	- You can login to the resources remotely throught a wide area network 
- **Multi-tenancy**
	- Thanks to virtualization, computing and storage resources can be partitioned such that a certain group can exploit only that partition
	- Provider’s computing resources are pooled to serve multiple consumers, allocated and deallocated as needed. Tenants are Isolated one to each other.
- **Rapid Elasticity**
	- We can rapidly add resources by scale up or scale out systems.
		- **Scaling up** (or vertical scaling) is adding more resources (CPU, memory... )
		- **Scaling out** means keeping single resource and increase the number of resources
- **Measured service**: Metering capability of service/resource abstractions in terms of storage, processing, bandwidth, active user accounts etc.
### Vertical and horizontal scaling
**Scaling up** (or vertical scaling) is adding more resources—like CPU, memory, and disk—to increase more compute power and storage capacity.
**Scaling out** (or horizontal scaling) keep single resource and increase the number of resources.
### Cloud Service Models
**Infrastracture as a Service**: IaaS provides virtual machines, virtual storage, virtual infrastructure, and other hardware assets as resources that clients can provision
**Platform as a Service**: PaaS provides virtual machines, operating systems, applications, services, development frameworks, transactions, and control structures.
**Software as a Service**: SaaS is a complete operating environment with applications, management, and the user interface.

**Container Platform** as a Service is a middleware between PaaS and IaaS
![[Pasted image 20231020093054.png]]
## Cloud Deployment Models
**Public/commercial cloud**: The public cloud infrastructure is available for public use alternatively for a large industry group and is owned by an organization selling cloud services. 
- You pay, you use it
**Private cloud**: The private cloud infrastructure is operated for the exclusive use of an organization. The cloud may be managed by that organization or a third party. Private clouds may be either on- or off-premises. 
**Hybrid cloud**: A hybrid cloud combines multiple clouds (commercial, community of public) where those clouds retain their unique identities, but are bound together as a unit. 
**Community cloud**: A community cloud is one where the cloud has been organized to serve a common function or purpose
- Like ths scientific community 
## Cloud Architecture
The cloud creates a system where resources can be pooled and partitioned as needed
- Virtualization is necessary to cloud computing
Cloud architecture can couple software running on virtualized hardware in multiple locations to provide an on- demand service to user-facing hardware and software
### Composability
Each component used during your development process is pluggable and can be replaced, scaled, and consistently improved to help you meet your business needs.
When establishing composable architecture, each component of your development stack and microservices should be able to communicate with one another — regardless of differences in language or code.

**Modular**: : It is a self-contained and independent unit that is cooperative, reusable, and replaceable.
**Stateless**: A transaction is executed without regard to other transactions or requests.
### Micro-service
Independent and self-contained units that perform a given task.
- This will avoid the *dependency hell*
Micro-service approach involves:
- **Isolate tasks** - a task can range from just a simple function (i.e. serve a file to download) to complex computer programs (i.e. classify images using a neural network).
- **Standard Access** - Microservices are interacted with using a well-defined interface, usually a REST API over HTTP, but a Secure Shell protocol (SSH) is a perfectly viable interface as well.
- **Stateless operations**
## Cloud Infrastructure
Most large Infrastructure as a Service (IaaS) providers rely on virtual machine technology to deliver servers that can run applications. Virtual servers described in terms of a machine image or instance have characteristics that often can be described in terms of real servers delivering a certain number of microprocessor (CPU) cycles, memory access, and network bandwidth to customers
![[Pasted image 20231020095408.png]]
- The dark blocks show the portion of the cloud computing stack that is defined as the “server.
- The VMM component is the Virtual Machine Monitor, also called a hypervisor. This is the low-level software that allows different operating systems to run in their own memory space and manages I/O for the virtual machines.
## Cloud Platoforms
A platform in the cloud is a software layer that is used to create higher levels of service.
Platforms represent nearly the full cloud software stack, missing only the presentation layer that represents the user interface.
![[Pasted image 20231020095750.png]]
- What separates a platform from a virtual appliance is that the software that is installed is constructed from components and services and controlled through the API that the platform provider publishes
- Just as a virtual appliance may expose itself to users through an API, so too an application built in the cloud using a platform service would encapsulate the service through its own API. Users would then interact with the platform, consuming services through that API, leaving the platform to manage and scale the service appropriately.

**Virtual appliances** are software installed on virtual servers
- A virtual appliance is a platform instance. Therefore, virtual appliances occupy the middle of the cloud computing stack
- you can use the appliances as the basis for assembling more complex services, the appliance being one of your standardized components
- virtual appliances are also much larger than the application themselves would be because they are usually bundled with the operating system on which they are meant to run
# Virtualization
“Virtualization uses software to create an abstraction layer over computer hardware that allows the hardware elements of a single computer—processors, memory, storage and more—to be divided into multiple virtual computers, commonly called virtual machines (VMs). Each VM runs its own operating system (OS) and behaves like an independent computer, even though it is running on just a portion of the actual underlying computer hardware.”
![[Pasted image 20231020101644.png]]
The limit is the hardware, but whithin these limits you can build your virtual machine however you want

---
### Example
If you install several services on the same machine you have the dependency hell because there could be several versions of the same library and, in case of update, there would be a lot of dependency problems
![[Pasted image 20231020102057.png]]
This doesn't happen if you use several virtual machines

---
**Virtual Machine** is the software simulation of a computer. It is able to run an Operating Systems and applications interacting with the virtualized abstracted resources, not with the physical resources, of the actual host computer.
**Hypervisor** (or **Virtual Machine Monitor**) is a software tool installed on the physical host system to provide the thin software layer of abstraction that decouples the OS from the physical bare-metal. It allows to split a computer in different separate environment, the Virtual Machines, distributing them the computer resources
## Virtualization components
![[Pasted image 20231020102254.png]]
- **Host machine** is the actual machine on which the virtualization takes place
- **Guest machine** is the virtual machine
- **Hypervisor** or **VM Manager** is the software or firmware that creates a virtualization layer on the host hardware
## Virtual Machines
- Virtual Machine is a virtual computing system 
- It has tightly isolated software with an operating system and applications inside 
- Each Virtual Machine in a host is independent 
- In a single physical server can be put multiple VMs enabling the run of multiple OSes and Applications
## VMs and Virtual Appliances
An application and its operating system packaged together for a virtualized environment
## Virtual Machines features
- **Consolidation** - Run different OSes on the same physical host Partition physical resources between VMs
- **Isolation** - Fault and security isolation at the hardware level Preserve performance with advanced resource control
- **Encapsulation** - Save the VM state to files VMs can be moved and copied moving and copying files (migration)
- **Hardware Independence** - VMs can be copied, moved or migrated to different physical servers
## Hypervisors
![[Pasted image 20231020103021.png]]
### Type 1 Hypervisor
- VMM manages all hardware resources and supports the execution of VMs
	- It should be allowed to manage all the drivers 
	- It is a sort of OS
The problem with this kind of Hypervisor is that there are a lot of different drivers in the world, therefore, the Hypervisor should have the same capabilities of an OS in menaging drivers' bugs 
 
Therefore, the idea was to have a level 1 Hypervisor and a guest privileged Virtual Machine (with an OS) with full privileged access to hardware to take care of drivers
![[Pasted image 20231020103455.png]]
### Type 2 Hypervisor
Hosted hypervisors are designed to run within a traditional operating system. A hosted hypervisor owns all HW. VMM module provides HW interfaces to VMs and manage VM context switch.
- The OS provides all the drivers to interact with the hardware
![[Pasted image 20231020104415.png]]
## Hardware Protection levels
Commodity hardware has more than two protection levels. (×86 architecture has four protection levels called rings).
- **Ring 3**: has the least level of privilege, so this is where the applications would reside.
- ...
- **Ring 0**: has the highest privilege and can access all the resources and execute all hardware-supported instructions (OS level).
![[Pasted image 20231020161931.png]]
## Hardware protection levels for a VM
## Type 1 Hypervisor
![[Pasted image 20231020162603.png]]
### Virtualization types:
- **Full Virtualization**: the hypervisor provides complete hardware abstraction creating simulated hardware devices. The guest OS don’t know (or care) about the presence of a hypervisor and issue commands to what it thinks is actual hardware.
- **Paravirtualization**: para means partial. The guest OS is aware that it is a guest, it recognizes the presence of a hypervisor, and it has drivers to issue some commands, mainly I/O operations, directly to the host OS, more efficiently than inside a virtual environment. The guest OS must be modified
	- VirtualBox knows that it is a guest OS
- **Hardware assisted virtualization**: is a type of full virtualization where the microprocessor architecture has special instructions to aid the virtualization of the hardware. These hardware extensions help the hypervisor tackle complex tasks at the processor level rather than through software emulation
	- Today, the high majority of processors offer this option
### Memory virtualization
![[Pasted image 20231020163431.png]]
Every layer abstracts a continuous stream of memory and then maps it to the lower layer
- The only mapping that is taken care by the hardware is Host Physiscal-Host Vortual
- All the others layers are managed at the software level

To solve this problem the **Shadow Page Table** has been introduced
- Establishes a shortcut to directly manage the mapping from GVA to HPA.
- It works the same way virtual address is mapped to physical address
![[Pasted image 20231020163925.png]]
## Memory paravirtualization
- Guests are aware of the virtualization 
- No longer strict requirement on continuous physical memory starting at 0
- Guests are modified to explicitely registers page tables with VMM
## Device Virtualization
### Passthrough
VMM configure the access permission to a device so that the GuestVM has direct access to the device (VMMbypass
- VM can directly access th edevice bypassing the hypervisor
- Device sharing $\rightarrow$ Overhead
- VMs and HW must have the exact type of device
	- Coupling
### Hypervisor direct mode
VMM intercept all the device access requests and emulate device operations.
- Latency in device access due to emulation
- VMM is extremely complex has it must support all devices
- VMM exposed to bugs in drivers and drivers
### Split device driver model
Device operations are split between front end driver in guest VM and backend driver in a service VM (dom0) that hosts the actual regular driver for the device.
- Eliminate emulation overhead
- Backend reduce the VMM exposure to bugs and improve security
- Only for paravirtualization guests
- Requires a modification in GuestVM front end device
## HW virtualization support
Hardware specifications to Hypervisors that reduced the overhead of VMM operations and greatly improve the speed and abilities of the VMM.
## HW emulator support
Creating an environment that imitates the properties of one system onto another. mimics the qualities and logic of one processor to run in another platform. Run an OS or software in any other system. Guest Operators need a translation.
- An emulator converts the needed architecture CPU instructions and successfully runs it on another architecture
## Emulation and virtualization
![[Pasted image 20231021170059.png]]
In summary, emulation is about mimicking the behavior of one system on another, typically involving higher-level abstraction, while virtualization creates isolated environments (virtual machines) that share the underlying hardware resources, operating at a lower level of abstraction. Virtualization is more efficient and versatile for running multiple operating systems simultaneously on a single host, while emulation is useful for running software or systems on non-native platforms, often with performance overhead.
## Network virtualization
![[Pasted image 20231021170419.png]]

- Support of multiple logical networks running on a common physical substrate
## Disk virtualization  
Disk virtualization, also known as storage virtualization, is a technology that abstracts and centralizes the management of physical storage resources to create more flexible and efficient storage solutions.

---
# What will we do?
- [ ] setup a cluster in several fashions and measure its performances
	- [ ] virtual machines
	- [ ] containers
	- [ ] kubernetes
- [ ] write notes about lecture 04
- [ ] write notes about lecture 05
- [ ] write notes about the Docker short course