# ROS 2 Communication Architecture

## Phase 1: Node Initialization & Discovery

### 1. ROS 2 Node Initialization
ROS 2 nodes are instantiated within an **RTPS (Real-Time Publish-Subscribe) communication model**, leveraging **Data Distribution Service (DDS)** as the underlying middleware. Each node belongs to a **ROS Domain**, defined by a **Domain ID**, ensuring logical isolation of distributed systems.

- **Publisher Node:** Registers with DDS to publish messages on a specific **ROS Topic**.
- **Subscriber Node:** Registers interest in receiving messages from the **ROS Topic**.
- **Topic:** A named **message queue abstraction** that facilitates **many-to-many** communication.

By default, **UDP multicast** is used for **peer discovery**. Nodes exchange **participant discovery messages** via the **Simple Discovery Protocol (SDP)** with the following announcements:
- **Publisher:** "I am a Publisher on Topic X"
- **Subscriber:** "I am a Subscriber looking for Topic X"

### 2. Discovery Mechanism & Matching Criteria
DDS performs **Participant Discovery**, followed by **Endpoint Discovery**, ensuring that publishers and subscribers are correctly matched based on:

1. **Topic Name:** Both endpoints must register under the same topic name.
2. **QoS Policies:**  
   - **Reliability QoS:** Determines whether messages are **Best-Effort** or **Reliable**.  
   - **Durability QoS:** Configures whether late-joining subscribers can receive past messages.  
   - **Liveliness QoS:** Ensures active node presence via periodic heartbeats.  
3. **Domain ID:** Defines the ROS 2 namespace, preventing cross-communication with other networks.

If a **publisher-subscriber pair matches**, the middleware determines the optimal transport mechanism:
- **Intra-Process Communication (IPC):** When both nodes exist within the same process, leveraging **zero-copy message passing**.
- **Inter-Process Communication (IPC):** When nodes exist in separate processes, using **Shared Memory Transport (SHM)** or **DDS transport (UDP/TCP)**.
- **Inter-Machine Communication:** If nodes are on different machines, messages are **serialized and transmitted over RTPS (Real-Time Publish-Subscribe) packets** via **UDP/TCP**.

### 3. Handling Network Restrictions
If **UDP multicast is restricted** (due to firewalls, VLAN segmentation, or separate networks), ROS 2 offers a **Discovery Server** as a centralized **peer registry**. Instead of **broadcasting UDP multicast messages**, nodes **register** with a **Discovery Server**, which manages participant lookup and route determination.

---

## Phase 2: Communication Mechanisms

### 1. Intra-Process Communication (IPC) - Zero-Copy Optimization
To eliminate **network overhead**, ROS 2 uses an **intra-process manager** to handle message passing. Instead of duplicating messages across multiple subscribers, a **meta-message** is sent, significantly reducing memory overhead.

#### Step-by-Step Execution:
1. **Publisher Sends a Message**  
   - The message is forwarded to the **IntraProcessManager**, which handles message routing.
2. **Ring Buffer Storage for Message Retention**  
   - Each **publisher maintains a circular buffer (ring buffer)** for temporary message storage.  
   - This enables **constant-time (O(1)) read/write** operations with **automatic overwriting** of older messages.
3. **Meta-Message Transmission**  
   - A **meta-message** (containing a **pointer to the ring buffer** and an **index offset**) is created.  
   - This **meta-message** is propagated to the middleware layer instead of the full message.
4. **Subscriber Retrieval of Message Data**  
   - The subscriber **extracts the pointer and index** from the meta-message.  
   - It fetches the message from **local shared memory**, eliminating redundant data copies.

---

### 2. Inter-Process Communication (IPC) - Shared Memory vs. DDS Transport
If publisher and subscriber nodes exist in **separate processes**, ROS 2 checks for **Shared Memory Transport (SHM)** support.

- **Shared Memory Transport (SHM) Enabled:**  
  - The message is written to a **memory-mapped file (MMF)** in the OS kernel space.  
  - The subscriber accesses the message **without serialization or context switching**.

- **Shared Memory Transport (SHM) Disabled:**  
  - The message undergoes **serialization** (typically via **CBOR, CDR, or XCDR encoding**).  
  - The serialized payload is transmitted via **DDS transport (UDP/TCP)**.

SHM is significantly faster than DDS transport because it eliminates **CPU context switching and unnecessary data movement**.

---

### 3. Inter-Machine Communication - RTPS Protocol Execution
When nodes exist on **separate machines**, message transport occurs over **DDS RTPS (Real-Time Publish-Subscribe) packets** via **UDP/TCP**.

#### Step-by-Step Execution:
1. **Publisher Serializes Message**  
   - The message is converted into an **RTPS packet**, conforming to **IDL (Interface Definition Language) serialization rules**.
   - Encapsulation occurs based on **XCDR (Extensible CDR) encoding**.

2. **Packet Transmission via Transport Layer**  
   - The RTPS packet is **fragmented and sent over UDP Multicast (default)** or **Unicast (if Discovery Server is used)**.

3. **Firewall & Network Addressing Mechanism**  
   - The message must pass **firewall policies** and comply with network transport configurations.
   - **ROS 2 dynamically calculates UDP port assignments** based on:
     ```bash
     Base_Port = 7400 + (Domain_ID * 250) + (Participant_ID * 2)
     ```
   - This ensures non-overlapping **UDP port allocation** for each participant.

4. **Subscriber Receives, Deserializes & Processes Message**  
   - The RTPS packet is **reassembled**, and the message is **deserialized**.
   - The subscriber validates **RTPS sequence numbers** to ensure **in-order message delivery**.
