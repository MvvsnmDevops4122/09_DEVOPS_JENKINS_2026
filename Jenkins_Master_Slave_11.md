# Jenkins Master–Slave architecture (Controller–Agent Architecture)

In Jenkins, the Master–Slave architecture (now commonly referred to as the Controller–Agent architecture) is a distributed system designed to handle high-volume build, test, and deployment workloads across multiple machines.

* Master-slave architecture is used to improve the performance.

<img width="879" height="462" alt="image" src="https://github.com/user-attachments/assets/070af77d-fc8e-4ddb-9c9c-29b46062eff5" />

---

## Jenkins Controller (Master)

* This is the main Jenkins server.
* Handles job scheduling, build result reporting, UI rendering, and coordination of agents.

👉 (“rendering” means displaying information on the UI)

---

## Slave Nodes (Agents)

* These are remote machines connected to the master.
* Execute build jobs as instructed by the master.

---

## Communication

* The master and agents communicate over TCP/IP using the Jenkins remoting protocol.

---

## How It Works

### Job Assignment

* The master schedules jobs and assigns them to available and suitable slave nodes based on labels, load, and node capabilities.

### Work Execution

* Each agent performs the actual build/test tasks independently, then send reports results back.

### Monitoring

* The master tracks each node’s status (online/offline), job progress, and logs.

---

# Benefits

1. **Scalability**

   * Add or remove slave nodes based on workload.

2. **Fault Tolerance**

   * If one node fails, job runs on another node.

3. **Resource Optimization**

   * Use all machines efficiently.

4. **Parallel Execution**

   * Run multiple builds at the same time.

5. **Cross-Platform Builds**

   * Run jobs on different OS (Windows, Linux, etc.).

---

## IQ Questions

**Q:** Where the jobs information will stored either slave or master?
**Ans:** All the jobs information stored in the master

**Q:** where the source code will be store?
**Ans:** Slave machine

**Q:** How many slave machines in your project?
**Ans:** 5 or 6

---

# Creating a Slave (Agent) Node in Jenkins UI

## Prerequisites

Before configuring Jenkins:

* Create a New EC2 Linux Instance (Slave Node)

* Security Group: Allow SSH (port 22) and communication with the Jenkins master

* Key Pair: Download .pem file (you’ll need this to connect via SSH)

* Create the Working Directory
  Example: `/home/ec2-user/node1`

---

# Jenkins Slave Setup Steps

## 1. Add New Node in Jenkins

* Go to: Jenkins Dashboard → Manage Jenkins → Manage Nodes (or)

* Open Jenkins UI → Click on Build Executor Status

* Click **+New Node**

* Name: Node1 (or your preferred name)

* Select **Permanent Agent** → Click Create

---

## 2. Configure Node Settings

* Name your agent node (e.g., Node1)

* Number of Executors: 1 (or based on EC2 CPU cores)

* Depends on CPU cores

* Example: 2 CPUs → set executors = 2

* One executor ≈ one thread

* Remote Root Directory: `/home/ec2-user/Node1`

* This is the working directory where Jenkins runs jobs

---

## 3. Assign Labels

* Labels are like name tags for nodes

* Helps Jenkins decide where to run jobs

* Usage:

  * Select → **Use this node as much as possible**

👉 Meaning:

* Jenkins uses this node whenever it's free
* Runs jobs if label matches

---

## 4. Launch Method Configuration

* Select: **Launch agents via SSH**

* Host: EC2 IP (e.g., 13.30.90.5)

Click → Add → Jenkins

* Kind: **SSH Username with private key**

* Username: `ec2-user`

* Private Key: Paste `.pem` file content

👉 Paste full key:

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

* Host Key Verification:
  Select → **Manually trusted key verification strategy**

---

# Node1 Setup and Troubleshooting

* Node shows **Offline** → Usually Java not installed

* Check logs from Build Executor Status

## Recommended Java

* Java 11 → Recommended
* Java 8 → Supported

## Install Java & Git

```bash
sudo yum install git -y
sudo yum install java-11-openjdk-devel -y
java -version
```

* The slave node in a Linux environment is responsible for executing the remote remoting.jar file, maintaining the connection with the master.

* This .jar handles all communication, job execution, and log transfer.

* ava -jar remoting.jar

* After Java is installed, Jenkins will detect the node and change status to Online.

*  Now the node is ready and connected to the master. We have assigned one executor to Node1.


```bash
java -jar remoting.jar
```

* After installation → Node becomes **Online**

---

# By Default – Builds Run on Master

* Jenkins runs builds on master by default

## Restrict Master Builds

1. Open Job Configuration
2. Enable: **Restrict where this project can be run**
3. Enter label (e.g., Nodes)
4. Use label name (not node name)
5. Save

---

# Understanding Labels

* Labels match jobs to nodes
* Node1 + Node2 can share same label

👉 Only label matters, not node name

---

# Execution Behavior

* Node1 has 1 executor

* If 2 jobs triggered:

  * Job1 → Running
  * Job2 → Queue

👉 Increase executors to run parallel jobs

---

## Increase Executors

* Go to: Manage Jenkins → Nodes → Node1 → Configure
* Change executors: 1 → 2
* Save

---

# Adding Node2 with Same Label

* Create Node2 same as Node1
* Assign same label: **Nodes**

👉 Jenkins distributes jobs automatically

---

# Parallel Execution Example

* Node1 → 2 executors
* Node2 → 1 executor

👉 5 jobs triggered:

1. Job1 → Node1
2. Job2 → Node1
3. Job3 → Node2
4. Job4 → Queue
5. Job5 → Queue

---

# Pipeline with Labels

* Instead of using agent any, you can specify a label to make sure the pipeline runs only on nodes that match the label.

* You have already configured:

• Node1 with label: Nodes  
• Node2 with label: Nodes  
• Any job that specifies the label Nodes can run on either Node1 or Node2.  
• Jenkins will distribute the jobs based on node availability and executor count. 

<img width="1010" height="529" alt="image" src="https://github.com/user-attachments/assets/d83c177a-afff-4373-a2b5-be52f04d46d8" />

```groovy
agent any
```

Use:

```groovy
agent { label 'Nodes' }
```

👉 Jobs run only on labeled nodes

---

# Jenkins Job and Source Code Storage

• Job Information:  

All job configurations, build history, and related metadata are stored on the Master (Controller) node.  

• Source Code:  

The actual source code and build workspace are downloaded and used on the Agent (formerly Slave) node during job execution.  

Jenkins Master–Slave Architecture enhances scalability and efficiency by distributing build tasks between a master node and multiple slave nodes. It ensures parallel execution, resource optimization, and faster build times through intelligent job assignment and load balancing.

* Improves performance
* Enables parallel execution
* Optimizes resources
* Provides scalability
* Distributes workload efficiently
