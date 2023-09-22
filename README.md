# Heterogeneity-Aware Operator Placement for Stream Processing Systems at the Edge

## Design Document

This project involves offloading processing at the edge to pre-process the data for stream processing. It uses Raspberry 
Pi as edge systems where operators are offloaded to compensate for the latency due to limited bandwidth.

<br/>

### Problem statement

The project aims to enhance stream processing systems, where resources are limited and latency requirements are stringent. This is accomplished by offloading operators to edge systems that are located near the data source. The goal is to optimize a streaming application that ingests data streams from multiple sources on different locations to the cloud via a Wide Area Network (WAN) with limited bandwidth and high latency. Existing stream processing systems, such as Flink, are mainly designed for data center servers with homogeneous hardware resources and cannot automatically offload tasks to the edge.

<br/>

### Proposed solution

The application would benefit greatly if data streams were preprocessed at the edge, i.e., near the data source, to reduce the data traffic and latency over the WAN. This can be achieved if edge systems are smart enough to pre-process data streams, filter out data, or automatically offload tasks to edges efficiently.

There can be two approaches: one is to identify the tasks that can be offloaded to edge systems using performance matrices, and the other is to implement a dynamic mechanism to automatically predict the data stream flow, compute the metrics, and decide which tasks can be offloaded to increase efficiency.

We will first extract the following **performance metrics**:
* `backPressureTimeMsPerSecond`: time that subtask spent being back pressured
* `idleTimeMsPerSecond`: time that subtask spent waiting for something to process
* `busyTimeMsPerSecond`: time that subtask was busy doing some actual work.
* `numRecordsOutPerSecond`: The number of records this operator/task emits per second.

Then we will be able to calculate the maximum output rate an instance can sustain for every task
The above can be calculated using the given equation i.e.

`trueOutputProcessingRate` = `numRecordsOutPerSecond`/(`busyTimeMsPerSecond`/1000)

The `trueOutputProcessingRate` denotes how many records a subtask can process per unit of useful time.
A higher `trueOutputProcessingRate` indicates higher efficiency.


The performance metrics provided by the Flink will be used to decide which tasks can be offloaded to edges.
The tasks that can be offloaded to edges will be offloaded to edges of the Flink cluster and the rest of the tasks 
will be executed on the server.

To offload the tasks, we will modify the scheduler in Flink to use custom scheduling and decide which operators can be offloaded and modify the scheduler configurations accordingly. This will take the performance metrics into account to make that decision. We have some python helper scripts [permutation.py](https://cs551-gitlab.bu.edu/cs551/spring23/student-resources/flink-placement/-/blob/main/scripts/permutation.py) to generate `schedulercfg` file for placement plan for a job and cluster. Our solution will generate schedule policies based on the query. 

To make the decision of offloading operators to edge systems, we would be developing a cost model. This cost model will incorporate various metrics into account like the ratio of  `numRecordsInPerSecond` and `numRecordsOutPerSecond`, the `trueOutputProcessingRate`, and is the operator stateful or stateless. After using our cost model to make the decision of whether we should move any operator to some other TaskManager, we would try to offload that operator dynamically without the need to restart the job. This could be done by pause that TaskManager, and then using our own strategy to reconfigure the operator placement, if not feasible we would use Flink's default strategy to restart the job with the new placement. For this to work, we have to make Flink compatible with heterogenous resources. Right now, Flink assumes all the TaskManagers have equal resources and are similarly configured (homogenous), we have to modify its behaviour to work with TaskManagers with different resources (heterogenous).


<br/>

### Expectations

Our goal is to implement a prototype that can offload tasks to edge systems to reduce the data traffic and latency overhead caused by the limited bandwidth of WAN. We aim to achieve this by implementing a heterogeneity-aware operator placement algorithm/system that can efficiently offload tasks to edge systems. With this, we expect to use edge resources as efficiently as possible, minimize usage of resources on the server, and improve the system's efficiency without sacrificing performance, thereby improving latency problems over the WAN. We expect to improve the latency performance with our system compared to if all the operators are running on the server resource. We expect to see performance gains if we are able to offload potentially resource cost saving stateless operators (determined by our cost model) to our edge systems compared to if all those operators are running in a server machine / server TaskManager.

<br/>

### Experimental Plan

We plan to use the following experimental setup to evaluate our solution:

- Cloud Server / Local Server
- Raspberry Pi 4B
- Flink
- Python
- Java

For the scheduling algorithm, we will start with the following pipeline:
1. Initially run all tasks on the Server side (except the source) and collect the performance metrics from Flink.
2. Calculate the `trueOutputProcessingRate` for each operator.
3. Select lightweight operators and place them on available slots at the edge.

We are planning to build this prototype using Flink, a stream processing system, and Raspberry Pi, through some cloud service (later). The initial prototype we would try to optimize queries from [Nexmark Benchmark](https://cs551-gitlab.bu.edu/cs551/spring23/student-resources/flink-placement/-/tree/main/example).
We will use some open source data sets initially to test our prototype. We will then identify which tasks are suitable for
offloading to edge systems. 

The following experimental setup will be used as the starting point:
- The parallelism of the operators will be fixed. Slot sharing and operator chaining will be disabled so that each subtask uses only one slot.
- The number of slots per TaskManager will be set to the number of CPU cores on that worker, for example, 4 slots per TaskManager, if there are 4 TaskManagers we would have 16 slots, because we are using Raspberry Pi 4B, which has a quad core CPU, therefore 1 slot per CPU core.
- For the first experiment, 1 Raspberry Pi and 1 Server node will be used. The source operator will be placed on the Raspberry Pi to simulate data collection at the edge. The heavy computation will be placed on the Server node. The network bandwidth will be limited to simulate a WAN.
- We will deploy our scripts to change placement, deploy Flink cluster and jobs, and collect metrics from Flink using a Python library like [flink-rest-client](https://pypi.org/project/flink-rest-client/).
- We are going to compare different scheduling policies and compare performance.


<br/>

### Success Indicators
Since the goal of this project is to reduce the data traffic and latency overhead caused by the limited bandwidth of WAN, the solution should be able to prove and demonstrate that this approach outperforms the normal approach.

Major milestones:

1. **Make Flink compatible with heterogenous resources.** 
Being able to configure Flink with TaskManagers running on systems with different configurations would be indicative of achieving this milestone. As of right now, all the TaskManagers running on Flink are assumed to have same configuration and resources (homogenous), making it compatible with heterogenous resources is what we will try to acheive here.
2. **Implement a cost model to determine operators which can be offloaded to TaskManagers on edge systems**
We will be implementing a basic cost model which takes multiple metrics from the Flink's environment, running job and the TaskManagers into account to determine a new placement of operators to improve the performance. Implementating a basic cost model which improves gives out a new placement to improve performance would mark this milestone as complete.
3. **Changing the placement configuration based on cost model output**
Changing the placement strategy of a running job in Flink based on the what our cost model determines. We will try to apply the new placement without the need for restarting the job, but if that is not feasible, we will be using the Flink's default behaviour to apply the new placement of operators. Being able to apply the new placement one way or the other would mark this milestone as complete.

Plan of action for the project:

1. Install Flink on Raspberry Pis (Task manager) and the server computer (Task manager and Job manager), and configure them as a Flink cluster.
2. Conduct initial experiments with Source operators on the RPis and heavy operators on the server machine.
3. Collect various metrics from RPis and the server.
4. Identify operators to offload, configure the scheduler to deploy them and optimize for predetermined queries.
5. Experiment with our solution, optimize it, modify and run more experiments. Optimize and repeat.
6. Brainstorm and work on improvements and future works such as dynamic operator offloading, dynamic deployment of operators, and so on.

Our project aims at improving the performance of a Flink job, by improving the placement of operators based on the available resources. It especially aims to tackle the latency overhead introduced over WAN due to limited bandwidth, by placing operators chosed by our algorithm at the edge systems near the source. Therefore, our solution should make these Flink jobs run faster, reduce performance lags due latency introduced from limited bandwidth and reduce resource requirements on server systems. Achieving these success indicators and milestones, would make this project a success. 

Project can be declared a success if we are able to make Flink run with heterogenous hardware, determine the operators which can be offloaded to edge systems, safely and efficiently offload those determined operators and improve the performance of the running Flink job with these steps.

<br/>

### Task assignment

- Atul Lal - Responsible for analyzing performance metrics and predicting scheduling configurations: Atul has been assigned this task because of his strong analytical and problem-solving skills, as well as his ability to work with complex data sets and models.

- Dhruv Toshniwal - Handle metrics collection: Dhruv has been assigned this task because of his programming skills and experience with software development best practices, as well as his ability to work with metrics and debugging code.

- Nishil Agrawal - Experimental setup design and installation of Flink on Raspberry Pi: Nishil has been assigned this task because of his experience with hardware setup and configuration, as well as his knowledge of distributed systems.

- Prateek Jain - Take care of Flink cluster setup, networking management, and bandwidth allocation: Prateek has been assigned this task because of his knowledge of networking protocols and configurations, as well as his experience with network management.

- Shubham Kaushik - Design the scheduler algorithm and handle deployment: Shubham has been assigned this task because of his experience with distributed systems and cluster management, as well as his knowledge of scheduling algorithms and techniques.

**Independent tasks done in parallel:**
- Firstly, our goal is to make Flink compatible with heterogeneous resources by configuring it with TaskManagers running on systems with different configurations. This will be done in parallel with calculating the performance metric, flink cluster setup, bandwidth allocation and other networking setup.
- Secondly, we are tasked with implementing a cost model that considers various metrics from the Flink environment, running job, and TaskManagers to determine which operators can be offloaded to TaskManagers on edge systems to improve performance.
- Finally, we need to change the placement configuration of a running job in Flink based on the output of the cost model.

**Task dependency:**
- The implementation of the cost model is dependent on the accurate calculation of various metrics from the Flink environment, running job, and TaskManagers. These metrics include processing power, available memory, and network bandwidth, among others. Therefore, before implementing the cost model, we need to ensure that the metrics are being calculated accurately and reliably. Once we have verified the accuracy of these metrics, we can proceed with the design and implementation of the cost model. By using the cost model to determine the most efficient placement of operators, we hope to improve the performance of Flink jobs, particularly in edge computing scenarios where resources are limited.
- If we are not able to apply the new placement of operators on task managers without restarting the job, then we will work on designing a mechanism to apply the new placement of operators on the task managers in real-time.
<br/>

### References

1. [Project document](https://piazza.com/class_profile/get_resource/lciuqdxe8rx4ik/lddjblzoy163jk)

2. [Xinwei Fu, Talha Ghaffar, James C. Davis, and Dongyoon Lee. 2019. Edgewise: a
better stream processing engine for the edge. In Proceedings of the 2019 USENIX
Conference on Usenix Annual Technical Conference (USENIX ATC '19). USENIX
Association, USA, 929–945.](https://www.usenix.org/system/files/atc19-fu.pdf)

3. [Apache Flink Metrics](https://nightlies.apache.org/flink/flink-docs-master/docs/ops/metrics/)

4. [Flink placement scripts](https://cs551-gitlab.bu.edu/cs551/spring23/student-resources/flink-placement/-/blob/main/scripts/README.md)

5. [Kalavri, V., Liagouris, J., Hoffmann, M., Dimitrova, D., Forshaw, M., & Roscoe, T. (2018, October). Three steps is all you need: fast, accurate, automatic scaling decisions for distributed streaming dataflows. 13th USENIX Symposium on Operating Systems Design and Implementation (OSDI 18), 783–798.](https://www.usenix.org/system/files/osdi18-kalavri.pdf)

6. [Flink placement project (Team 4)](https://cs551-gitlab.bu.edu/cs551/spring23/group-4/flink-placement)