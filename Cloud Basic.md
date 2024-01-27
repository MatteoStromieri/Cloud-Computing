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
# Lecture 2
## Grid computing 
A GRID computer is a collection of geographically separated resources **(CLUSTERS)** connected by a network software layer, often called middleware, which transforms a collection of independent resources into a single, coherent, virtual machine
- Every cluster has a scheduler
- There is a metascheduler, you submit your job to it, it accepst your request, finds the best (based on several metrics) cluster to process your job and send it the job. Then the job interact with the scheduler of that cluster
- This is useful for HTC, so for low I/O tasks
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
- So you are coupling data acquisition with the first step of analysis
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
**Scaling out** (or horizontal scaling) link togheter multiple resources.
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
Each component used during your development process is pluggable and can be replaced, scaled, and consistently improved to help your business needs.
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
- What separates a platform from a virtual appliance is the API. The software that is installed is constructed from components and services and controlled through the API that the platform provider publishes
- Just as a virtual appliance may expose itself to users through an API, so too an application built in the cloud using a platform service would encapsulate the service through its own API. Users would then interact with the platform, consuming services through that API, leaving the platform to manage and scale the service appropriately.

**Virtual appliances** are software installed on virtual servers
- A virtual appliance is a platform instance. Therefore, virtual appliances occupy the middle of the cloud computing stack
- you can use the appliances as the basis for assembling more complex services, the appliance being one of your standardized components
- virtual appliances are also much larger than the application themselves would be because they are usually bundled with the operating system on which they are meant to run
# Virtualization
“Virtualization uses software to create an abstraction layer over computer hardware that allows the hardware elements of a single computer—processors, memory, storage and more—to be divided into multiple virtual computers, commonly called virtual machines (VMs). Each VM runs its own operating system (OS) and behaves like an independent computer, even though it is running on just a portion of the actual underlying computer hardware.”

The limit is the hardware, but whithin these limits you can build your virtual machine however you want
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
When a service is deployed, both the application and its operating system packaged are together for a virtualized environment in order to turn it up and make it run without having to face dependencies and so on
## Virtual Machines features
- **Consolidation** 
	- Run different OSes on the same physical host 
	- Partition physical resources among VMs
- **Isolation** 
	- Fault and security isolation at the hardware level  
- **Encapsulation** 
	- Save the VM state to files 
	- VMs can be moved and copied moving and copying files (migration)
- **Hardware Independence** 
	- VMs can be copied, moved or migrated to different physical servers
## Hypervisors
![[Pasted image 20231209170710.png]]
### Type 1 Hypervisor
- VMM (Hypervisor) manages all hardware resources and supports the execution of VMs
	- It should be allowed to manage all the drivers 
	- It is a sort of OS
The problem with this kind of Hypervisor is that there are a lot of different drivers in the world, therefore, the Hypervisor should have the same capabilities of an OS in menaging drivers' bugs 
 
Therefore, the idea was to have a level 1 Hypervisor and a guest privileged Virtual Machine (with an OS) with full privileged access to hardware to take care of drivers
### Type 2 Hypervisor
Hosted hypervisors are designed to run within a traditional operating system. A hosted hypervisor owns all HW. VMM module provides HW interfaces to VMs and manage VM context switch.
- The OS provides all the drivers to interact with the hardware
## Hardware Protection levels
Commodity hardware has more than two protection levels. (×86 architecture has four protection levels called rings).
- **Ring 3**: has the least level of privilege, so this is where the applications reside.
- **Ring 0**: has the highest privilege and can access all the resources and execute all hardware-supported instructions (OS level).
## Basic Virtualization Technique for x86 architecture
In order to do the privileged actions, the guest operating system will generate a trap that it will pass on to the hypervisor. This acts as a system call, and the hypervisor will then make the call on the guest operating system’s behalf. Hypervisor will decide what needs to be done as a result. When a trap is present, the hypervisor checks to see if the operation is legal before terminating the virtual machine. If the operation is legal, the guest operating system’s functionality, which anticipates being executed at hardware speed by hardware, will be emulated. In case of illegal operation the hypervisor will shut down the virtual machine.
>[!NOTE]
>Thus, all of the privilege instructions are likewise carried out at the guest operating system level with the aid of the hypervisor without significant efficiency loss.
## Type 1 Hypervisor
### Virtualization types:
- **Full Virtualization**: the hypervisor provides complete hardware abstraction creating simulated hardware devices. The guest OS don’t know (or care) about the presence of a hypervisor and issue commands to what it thinks is actual hardware.
- **Paravirtualization**: para means partial. The guest OS is aware that it is a guest, it recognizes the presence of a hypervisor, and it has drivers to issue some commands, mainly I/O operations, directly to the host OS, more efficiently than inside a virtual environment. The guest OS must be modified
	- VirtualBox knows that it is a guest OS
- **Hardware assisted virtualization**: is a type of full virtualization where the microprocessor architecture has special instructions to aid the virtualization of the hardware. These hardware extensions help the hypervisor tackle complex tasks at the processor level rather than through software emulation
	- Today, the high majority of processors offer this option
### Memory virtualization
![[Pasted image 20231020163431.png]]
Every layer abstracts a continuous stream of memory that and then maps it to the lower layer
- The guest physical memory starts from address 0
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
Currently, due to the devices’ great degree of variation, all of them lack the standard specification interfaces and behaviour. So, there isn’t really a universal standard that all gadgets will adhere to.

As a result, device virtualization is not as straightforward as it is for the CPU and memory, as we have seen while employing those techniques.
### Passthrough
VMM configure the access permission to a device so that the GuestVM has direct access to the device
- VM can directly access th edevice bypassing the hypervisor
- Device sharing $\rightarrow$ Overhead
- VMs and HW must have the exact type of device
	- Coupling
### Hypervisor direct mode
VMM intercept all the device access requests and emulate device operations.
- VM decoupled with physical device
- Latency in device access due to emulation
- VMM is extremely complex as it must support all devices
- VMM exposed to bugs in drivers and drivers
### Split device driver model
One device driver runs inside the guest Virtual Machine (aka domU) and communicates with another corresponding device driver inside the control domain Virtual Machine (aka dom0) which is managed by the Hypervisor (Both in the case of type I and II). This pair of codesigned device drivers function together, and so can be considered to be a single "split" driver.

Device operations are split between front end driver in guest VM and backend driver in a service VM (dom0) that hosts the actual regular driver for the device.
- The backend driver is the same as the real device driver
- The front-end driver has to be modified to communicate with the hypervisor $\rightarrow$ limited to paravirtualized guests
- Eliminate emulation overhead when accessing available devices
## HW virtualization support
Hardware specifications to Hypervisors that reduced the overhead of VMM operations and greatly improve the speed and abilities of the VMM.
## Emulation
Creating an environment that imitates the properties of one system onto another. mimics the qualities and logic of one processor to run in another platform. 
- Run an OS or software in any other system. 
- Guest Operators need a translation.
	- An emulator converts the needed architecture CPU instructions and successfully runs it on another architecture

Emulation brings higher overhead but has its perks too. It is highly inexpensive, easy to access, and helps us run the programs that have become obsolete in the available system
## Emulation and virtualization
![[Pasted image 20231021170059.png]]
In summary, emulation is about mimicking the behavior of one system on another, typically involving higher-level abstraction, while virtualization creates isolated environments (virtual machines) that share the underlying hardware resources, operating at a lower level of abstraction. Virtualization is more efficient and versatile for running multiple operating systems simultaneously on a single host, while emulation is useful for running software or systems on non-native platforms, often with performance overhead.
## Network virtualization
- Support of multiple logical networks running on a common physical substrate
## Disk virtualization  
Disk virtualization, also known as storage virtualization, is a technology that abstracts and centralizes the management of physical storage resources to create more flexible and efficient storage solutions.

**Fixed size disk**: When you create a fixed-size virtual hard disk, space is reserved beforehand. Suppose you created a VHDX of 200 GB, then 200GB space will be reserved for you on the hard disk of the host.

**Dynamically expanding disk**: When you create a dynamically expanding virtual hard disk, initially, just a small space is reserved for you on the host. Suppose you create a 200GB virtual hard disk for your VM, although your VM will look as if it has a 200GB virtual hard disk, but only a small amount of space will be taken up from the hard disk of the host. When the VM starts writing data, the VHD will grow automatically.
- In a dynamically expanding virtual disk, your hard disk space is not wasted but your virtual disk may be fragmented

**Differencing disk**:his image type saves all changes within the VHD to a child image, with the option to undo the changes or to merge the changes into the VHD. This type of image allows cloning of VHDs.

**Pass-through disk image**: This kind is linked to a physical hard drive or to one of its partitions
### Disk virtualization implementation

Disk image files reside on the host system and are seen by the guest systems as hard disks of a certain geometry. When a guest OS reads from or writes to a hard disk, Oracle VM VirtualBox redirects the request to the image file. VirtualBox uses **Virtual Disk Image** (.vdi) file format.
- portable and supported by other hypervisors
- Fixed size or dynamically expanding size

Virtual hard disks (**VHDs**) are disk image file (.vhd) formats that have similar functionalities to a physical hard drive and are designed primarily for use with Hyper-V virtual machines.

**VMDK** (short for Virtual Machine Disk) is a file format that describes containers for virtual hard disk drives to be used in virtual machines like VMware Workstation or VirtualBox.
- It can be both dynamic or fixed size
# Cloud Service Models
Consider the average cloud architecture
![[Pasted image 20231208175321.png]]
In this lecture we are going to inspect the most famous service models
## Traditional service models
The most famous service models are **IAAS**, **PAAS** and **SAAS**
![[Pasted image 20231208175644.png]]
>[!Cloud infrastructure]
> An infrastructure to provide users with the most flexible way to allocate computational power and storage space (and any other IT services)
### IAAS
Infrastructure as a service (IaaS) refers to online services that provide high-level APIs used to dereference various low-level details of underlying network infrastructure like physical computing resources, location, data partitioning, scaling, security, backup etc.

A hypervisor runs the virtual machines as guests. Pools of hypervisors within the cloud operational system can support large numbers of virtual machines and the ability to scale services up and down according to customers' varying requirements.

More concisely, IaaS is a cloud computing service model in which hardware is virtualized in the cloud
#### Architectural elements
The fundamental unit of virtualized client in an IaaS deployment is called a **workload**. A workload simulates the ability of a certain type of real or physical server to do an amount of work. In other words it is the amount of resources allocated to a client to do an amount of work.

Workloads support a certain number of users, at which point you exceed the load that the instance sizing allows. When you reach the limit of the largest virtual machine instance possible, you must make a copy or clone of the instance to support additional users. A group of users within a particular instance is called a **pod** (Point of Delivery).

Pods are aggregated into pools within an IaaS region or site called an **availability zone**. In very large cloud computing networks, when systems fail, they fail on a pod-by-pod basis, and often on a zone-by-zone basis

When cloud computing infrastructure isolates user clouds from each other so the management system is incapable of interoperating with other private clouds, it creates an information **silo**.

>[!Availability zones in Azure]
>Many Azure regions provide _availability zones_, which are separated groups of datacenters within a region. Availability zones are close enough to have low-latency connections to other availability zones.
### PAAS
**Platform as a service** (**PaaS**) or **application platform as a service** (**aPaaS**) or platform-based service is a category of [cloud computing services](https://en.wikipedia.org/wiki/Cloud_computing#Service_models "Cloud computing") that allows customers to provision, instantiate, run, and manage a modular bundle comprising a [computing platform](https://en.wikipedia.org/wiki/Computing_platform "Computing platform") and one or more applications, without the complexity of building and maintaining the infrastructure typically associated with developing and launching the application(s), and to allow developers to create, develop, and package such [software](https://en.wikipedia.org/wiki/Software "Software") bundles.

Paas provides a framework for:
- **Application development**: A PaaS platform either provides the means to use programs you create in a supported language or offers a visual development environment that writes the code for you.
- **Collaboration**: Many PaaS systems are set up to allow multiple individuals to work on the same projects.
- **Data management**: Tools are provided for accessing and using data in a data store.
- **Instrumentation, performance, and testing**: Tools are available for measuring your applications and optimizing their performance.
- **Storage**: Data can be stored in either the PaaS vendor’s service or accessed from a third- party storage service.
- **Transaction management**: Many PaaS systems provide services such as transaction managers or brokerage service for maintaining transaction integrity

PaaS systems exist to allow you to create software that can be hosted as SaaS systems. In fact, PaaS cloud are designed to allow in developing robust, scalable, and hopefully portable applications; indeed, they are provided with a set of tools to support the development to build different types of applications designed to work togheter

>[!In practice]
>You just deploy your application and the service figures out what to do with it.
>
>A platform as a service should handle scaling seamlessly for you so you can just focus on your website and the code running it.
## Modern models
There is a set of new emerging models in the marked. Here we list some of them:
- Application platform (Paas)
- Function platform (Faas)
- Analytics platform (Aaas)

They exploit some of the most currently trending technologies like containers, HPC platforms (GPU, FPGAs...), accekerated libraries, ML tools, parallel file systems, data lake, etc...
# Data Cloud and Cloud Security
Cloud storage is a service model in which data is maintained, managed and backed up remotely and made available to users over a network (typically the Internet)

Features of Cloud Storage:
- improve scalability
- improve security
- simplify storage management
- on demand access
- unstructured data

Traditionally, data can be store inside a **block storage** or a **file storage**
## Block storage
- You have an hard drive connected to your machine with a cable
- storage and computing system are highly coupled, therefore concurrent reads and writes are not supported
- must be formatted by the computing node
- complex to expand
- expensive
## File storage
- Data are abstracted into files and directories
- Network shared
- High throughput 
- scale out capabilities 
- Multi-tiered architecture
- Expensive
### Distributed File System
- File system that is shared by many distributed clients 
- The resources (file and dir) on a particular machine are local to itself
- Resources on other machines are remote 

The server interface is the normal set of create, read, write... on files
- This interface is used by the clients to perform operations on the resources managed by the server
#### Properties of a distributed file system
**Transparency:**
- Location: a client cannot tell where a file is located
- Migration: a file can transparently move to another server
- Replication: multiple copies of a file may exist
- Concurrency: multiple clients access the same file

**Flexibility**:
- Servers may be added or replaced 
- Support for multiple file system types

**Dependability**:
- Consistency: conflicts with replication & concurrency
- Security: users may have different access rights on clients sharing files & network transmission
- Fault tolerance: server crash, availability of files

**Performance**:
- Requests may be distributed across servers
- Multiple servers allow higher storage capacity

**Caching:**
- Reduce network traffic by retaining recently accessed disk blocks in a cache, so that repeated accesses to the same information can be handled locally
- If required data is not already cached, a copy of data is brought from the server to the user and into the cache
- Files are identified with one master copy residing at the server machine
- Copies of (parts of) the file are scattered in different caches, this causes the cache consistency problem which can be solved with several techniques

Ideally, the client would perceive remote files like local ones.

Configuration and implementation may vary:
- Servers may run on dedicated machines
- Servers and clients can be on the same machines
## The Google File System
Motivation:
- One single distributed file system
- Store big data reliably 
- Allow parallel processing of big data

Assumptions:
- Inexpensive components that often fail 
- Large files (million of files 100+MB) 
- Large streaming reads and small random reads (500Mb/s read/write load) 
- Large sequential writes 
- Multiple users append to the same file 
- High bandwidth is more important than low latency

To accomplish this we can't fully implement the POSIX interface. Therefore, to mitigate this problem, a similar interface has been implemented.

Definitions:
- **Snapshot**: low cost copy of a whole file with copy-on-write operation 
- **Record append**: Atomic append operation
### GFS design overview
- Files split in fixed size chunks of 64 MByte 
- Chunks stored on chunk servers 
- Chunks replicated on multiple chunk servers 
- GFS master manages name space 
- Clients interact with master to get chunk handles 
- Clients interact with chunk servers for reads and writes No explicit caching
>[!Note]
>Each file in Google File System consists of many chunk and each chunk is identified by a 64 bits chunk handle.

**Master server**: 
- Single master 
- Keep metadata 
- accept requests on metadata 
- Most management activities

**Chunk servers**: 
- Multiple 
- Keep chunks of data
- Accept requests on chunk data 
![[Pasted image 20231208193822.png]]
## Object Storage
Here we don't use the POSIX standard to access resources but the HTTP verbs

- An object storage exposes a set of REST apis to acess its resources
	- They can be used to obtain only a portion of a certain object
- Data are not split into chunks but distributed

Assume that we have several seververs full of disks, then we have a master server (called manager) that manages the resources and the access to them. The manager manages:
- The ACL (Access Control List)
- Load balance
- multiple copies of the same resource
	- high throughput
	- when a file is closed in the manager, copies on the servers are overwritten
### Objects
An object is a logical unit of storage:
- ID
- Data
- Metadata: data that describes the Object (i.e. the data)
	- They can be changed
	- They can be indexed
	- We can make queries on objects based on metadata
- Other attributes

They usually have file-like methods: open, close, read, write
### Common features
- Objects have a 64bit unique ID
- Objects live on a flat namespace
	- You can mimic a file system but */mydata/obs/...* is just a label
- Every request has three components
	- HTTP verb
	- Authentication information
	- SURL (Storage URL)
	- Metadata (Optional)
## Cloud Security
### General considerations
**Security boundary**: separating the client’s and vendor’s responsibilities in terms of privacy.

We have to consider the privacy at different levels:
- Network level
- Host level
	- Hypervisor and Virtual Machine security
		- VM images
		- Vulnerable services
		- ssh private keys
- Application level

On a cloud is necessary to do
- auditing: you have to know what happens to your data
- data integrity
- Recovery
- Regulatory compliance

Before approaching the cloud
- Determine which resources (data, services, or applications) you are planning to move to the cloud.
- Determine the sensitivity of the resources
- Determine the risk associated with the particular cloud type for a resource
- Take into account the particular cloud service model that you will be using
- If you have selected a particular cloud service provider, you need to evaluate its system to understand how data is transferred, where it is stored, and how to move data both in and out of the cloud

**General and important concept**: The platform security level is reduced to the security level of the most vulnerable application running on the platform. (isolate with VM and containers)

OS:
- Implements minimal security
- Applications with special privileges that perform security- related functions are called trusted applications. Only such applications should be allowed the lowest level of privileges required to perform their functions.
- An OS poorly isolates one application from another, and once an application is compromised, the entire physical platform and all applications running on it can be affected.

Data:
- Identify the security boundary separating the client’s and vendor’s responsibilities 
- Determine the sensitivity of the data at risk 
- Data should be transferred and stored in an encrypted format. 
- Separate clients from direct access to shared cloud storage

Key mechanisms to protect data:
- Access control
- Logging
- Authentication
- Authorization

**Data Segregation and Isolation** is a very good practice:   
- Isolate data from direct client access creating a layered access to the data.
- Data segregation based on tenants
![[Pasted image 20231208201049.png]]
# Containers short course
Containers bring basically the same benefits of virtual machines but require a fraction of their resources.
## Solutions spectrum for the dependency hell problem
![[Pasted image 20231209131237.png]]
### Proper requirements
Carefully keep track of what libraries/OS features are used in development and report them on the documentation, for each release
- Prone to human error
### Virtual environments
Work in a reproducible environment where libraries are the same for developers and for users. Each release has a virtual environment definition.

Requires the user to set up and activate its own environment, and works only with some libraries.

- Not a comprehensive solution (You can not always create a virtual environment) and prone to human error
### Statically linked binaries
- Only works for compiled languages
- big binaries

Overall, this is a good solution
### Virtual machines with hardware emulation
- [Overengineering](https://en.wikipedia.org/wiki/Overengineering)
### Virtual machines
The pros are
- doesn't touch the main OS
- quick to setup
- allows to quickly  test a given software

Cons:
- Need to download a (big) pre-built, trusted image (no “source” code);
- Requires pre-allocating dedicated memory at startup, and an entire boot
- Not suitable for much more than just giving the software a try;

... but we are on the right path. We want this kind of insulation
### Containerization
Containers are lightweight, standalone, executable packages of software that include everything needed to run an application
- code
- runtime
- system tools
- system libraries
- etc...
Containers allow to reliably move and distribute software from one computing environment to another, without the burden of VMs.
#### Main concepts
The idea of containers is to insulate a single process from your Operating System
- Let it live in its own space, including its own network;
- Let it have its own File System with its own libraries;
- Allow to natively access hardware without virtualization;
- Avoid booting an entire Virtual machine and to pre-allocate dedicated memory

The following image depicts the differences between Containers and VMs
![[Pasted image 20231209132735.png]]
- Docker is a container engine
- containers share kernel OS, file system....
	- The file system is layered, if multiple containers use the same layer they can share it

In order to run a container you have to
1) get or build a container image (generic source code for a container)
2) run the image (this is your container)

The following is the command used to run an image
```bash
docker run my_image
```
>[!NOTE]
>If docker doesn't find the image locally will automatically look online

The following is an output example when docker can't find the image locally
```bash 
$ docker run hello-world 

Unable to find image 'hello-world:latest' locally 
latest: Pulling from library/hello-world 
2db29710123e: Pull complete 
Digest:sha256:6d60b42fdd5a0aa8a718b5f2eab139868bb4fa9a03c9fe1a59ed4944318 
Status: Downloaded newer image for hello-world:latest 

Hello from Docker! 
This message shows that your installation appears to be working correctly.
```
We can use the ```--entrypoint``` option to overwrite the entrypoint of the image that we are trying to run
```bash
docker run --entrypoint /bin/bash -it python:3.8
```
**Volumes** are the tools used to share files between different containers in Docker
```bash
docker run -it -v $HOME:/data python:3.8
```
In the above example we are runninga container and we are mounting the *home* directory into the */data* directory of our container.
- A volume can be both read-only or read-write

>[!WARNING]
>[Docker containers](https://www.geeksforgeeks.org/containerization-using-docker/) can only communicate with other containers on the same Docker network and run by default in isolation from the host and external networks. By publishing a [container’s network](https://www.geeksforgeeks.org/basics-of-docker-networking/) service to the host or external network through [port](https://www.geeksforgeeks.org/docker-managing-ports/) mapping, you can make it reachable from other networked devices.

If we want to access a server from within the container we have to exploit one function of Docker called **port mapping**
```bash
docker run -p 9001:8888 jupyter/tensorflow-notebook:tensorflow-2.4.1
```
In this case we are publishing the port 8888 from within the container to port 9001 on the host node.

Note that in this case the port is available to every user in the world, you can use certain options to make is accessible only to localhost or certain IP addresses.
>[!SHARING CONTAINERS]
>If you want to share a container you can use the **registries**. They are like a github for software containers.
>
> When Docker can't find an image locally, it looks for it over the registries
## Docker
Docker is the de facto containerization standard

Some features:
- Incremental file system
- Plenty of software on DockerHub
- Native on linux
	- if your machine runs windows it will install a lightweight linux VM to run on it

>[!Defintion]
>In computer science, native software or data-formats are **those that were designed to run on a particular operating system.** 

- Relies on a system daemon to manage container
- Running containers are seen as (micro)services
- Containers have an IP address by default 
	- You can make containers communicate with each others using these IPs
- Extensive support for networking between containers
- Requires a **privileged user**
	- You have to be root or member of the Docker group
- **Isolation**
	- Filesystem at runtime: completely isolated by default, use volumes to bind folders
	- Network: isolated within the Docker engine, use --net-host to use the host network or -p to share ports

Some useful commands
- **docker build**: Build a container 
- **docker pull**: Pull a container (from a registry) 
- **docker run**: Run a container (and execute the default command, or a custom one) 
- **docker ps**: List running containers 
- **docker exec**: Run a command in a running container 
- **docker stop**: Stop a running container 
- **docker rm**: Remove a container

For every Docker image there are several version, each of them is identified by a **TAG**. For every  (Image,Tag) couple there are **several version** depending on which architecture they are supposed to be ran. 
- If you don't specify the tag you will get the latest version
- If you don't specify the digest you will get the version compatible with your architecture

Every Docker image with a certain tag and architecture is identified by a **digest** which is the hash of the manifest of that image
- So, since on DockerHub there are several images with the same name, the only way to be sure that you are downloading the right one is to add the digest in your command

>[!NOTE]
>An **image** is a “file” from which you can run a container while a **container** is an “entity” run from an image

To run a shell inside the container 
```bash
docker run -it gcc:5.4 bash
```
>[!PIDS]
>The system process with pid=1 inside the container has not pid 1 outside. Indeed, everything that runs inside a container is mapped into something else on the main OS 
### GCC example
```bash
$ docker run -it gcc:5.4 bash
root@b9c1414bab3d:/#
```
As you can see, the *bash* process is the root inside the container
```bash
root@b9c1414bab3d:/# ls 
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```
List running processes
```bash
root@b9c1414bab3d:/# ps -ef 
UID PID PPID C STIME TTY TIME CMD 
root 1 0 1 13:54 pts/0 00:00:00 bash 
root 8 1 0 13:54 pts/0 00:00:00 ps -ef
```
Get your container IP address 
```bash
root@b9c1414bab3d:/# ip addr show dev eth0 
[...] 
inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0 
[...]
```
>[!NOTE]
>When you exit a container, you lose every change to the container File System
### Dockerfile
The **Dockerfile** is a file that containes a set of instructions that allows me to build a container
- The Dockerfile is what defines a Docker Container. Think about it as its source code
- When you build it, it generates a Docker Image. When you run a Docker Image, this “becomes” a Docker Container, as mentioned before

The following is a simple example of Dockerfile
```dockerfile
FROM <base image>
RUN <a setup command>
COPY <source file/folder> <dest file/folder in the containers>
RUN <another setup command>
```
Assuming that you have a Dockerfile called Test, you can build it using
```bash
docker build Test -t testcontainer
```
Then, if you want to run the corresponding container
```bash
docker run testcontainer /opt/test.bin
```
And you can also share it
```bash
docker save testcontainer > testcontainer.tar
docker load < testcontainer.tar
docker tag testcontainer sarusso/testcontainer
docker push sarusso/testcontainer
```
when I push, I only load the layers (lines in the Dockerfile) that I added to an image that is already existing in DockerHub
### Versioning in Docker
In the Docker ecosystem Dockerfile layer and every Dockerfile/Image is versioned using hashes
- A tag is a friendly name for an hash
	- When you write gcc:5.4, “5.4” for the gcc Docker container is actually saying that the tag is “gcc:5.4"
- For practical use, also the short hashes are allowed (and commonly used), which are the first 12 characters (of the Hash) for Docker. If by chance two hashes in the system starts with the same short hash, you will be required to enter one more character or the full hash.

>[!WHERE DO YOU SAVE YOUR DOCKERFILES?]
>On a versioning system like git

- Docker allows to have everything up and running, including dependencies etc. with a single command. 
- This command trigger a build with a given set of dependencies (the ones you wrote to install in the Dockerfile) 
- Over time, you will probably make changes in your Dockerfiles and in your code. 
- If you use a versioning system, you can jump back in time to a particular **version/hash**, build it, and it will run exactly as it was running at that time 
- For managing multiple container versions simultaneously, you can use **tags**

# What will we do?
- [ ] setup a cluster in several fashions and measure its performances
	- [ ] virtual machines
	- [ ] containers
	- [ ] kubernetes
- [x] write notes about lecture 04
- [x] write notes about lecture 05
- [x] write notes about the Docker short course