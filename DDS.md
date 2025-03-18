# **DDS in ROS 2**

## **1. Introduction to DDS in ROS 2**
ROS 2 utilizes the **Data Distribution Service (DDS)** as its middleware for inter-node communication. DDS is a **decentralized** publish-subscribe communication protocol that replaces the **centralized ROS Master** used in ROS 1. This transition enables **dynamic peer-to-peer discovery** and enhances scalability.

### **Key Features of DDS in ROS 2:**
- **Automatic Discovery:** Nodes dynamically detect and communicate with each other without requiring a master node.
- **Quality of Service (QoS):** Offers fine-grained control over message reliability, durability, and deadline enforcement.
- **Transport Flexibility:** Supports **UDP multicast, unicast, and TCP**, allowing for adaptability based on network requirements.
- **Data-Centric Architecture:** Prioritizes shared data structures over direct message-passing, enhancing efficiency.

## **2. Transport Protocols in ROS 2**
DDS supports two primary transport protocols:
- **UDP (User Datagram Protocol):** A low-latency, connectionless protocol that does not guarantee message delivery.
- **TCP (Transmission Control Protocol):** A reliable, connection-oriented protocol ensuring guaranteed delivery.

### **2.1 UDP Multicast in ROS 2**
Upon initialization, a node sends a **discovery message** via UDP multicast, allowing other DDS participants to detect it.

- **Message Type:** Discovery message containing node metadata.
- **Destination:** A predefined **multicast address** where all DDS nodes listen.

#### **Configuring UDP Multicast in Fast DDS**
```xml
<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <participant name="MyParticipant" domainId="42">
        <rtps>
            <useMulticast>true</useMulticast>
        </rtps>
    </participant>
</profiles>
```
This configuration enables **UDP multicast discovery** for a participant within **Domain ID 42**.

## **3. Discovery Mechanism in DDS**
The DDS discovery process enables nodes to dynamically locate each other, eliminating the need for a central master node.

### **3.1 Steps in the DDS Discovery Process:**
1. **Node A starts** → Sends a **discovery message** via UDP multicast to predefined group addresses.
2. **Other nodes receive this message** → Respond with their details (IP address, GUID, topic list, QoS settings).
3. **Node A updates its peer list** → Now aware of active nodes and their capabilities.
4. **Communication is established** → Nodes exchange data over topics.

### **3.2 DDS Simple vs. Server Discovery Modes**
- **Simple Discovery:** Default mechanism using **UDP multicast**.
- **Discovery Server Mode:** A **centralized discovery server** manages node registration (useful for large-scale deployments).

#### **Why Are Topics Needed in DDS?**
Despite nodes discovering each other dynamically, topics enable **structured publish-subscribe communication**, avoiding direct IP-based messaging. This ensures:
- **Asynchronous communication** (nodes don’t need to be online simultaneously).
- **Scalability** (multiple publishers/subscribers share the same topic efficiently).

## **4. QoS (Quality of Service) in DDS**
DDS provides extensive QoS policies to control communication behavior, ensuring reliability and performance.

### **4.1 Key QoS Policies:**
| QoS Policy        | Description |
|------------------|-------------|
| **Reliability**   | Best-Effort vs. Reliable delivery. |
| **Durability**    | Determines message persistence when new subscribers join. |
| **History**       | Controls the number of stored messages. |
| **Deadline**      | Enforces expected message frequency. |
| **Lifespan**      | Defines message expiration time. |

### **4.2 Transient-Local QoS: Message Persistence**
With **transient-local durability**, messages remain stored **locally on the publisher** until they are delivered to subscribers.

```xml
<durability>
    <kind>TRANSIENT_LOCAL</kind>
</durability>
```
This configuration ensures that **late-joining subscribers** receive the latest published data.

## **5. ROS 2 Domain ID**
Each ROS 2 network operates within a **domain**, uniquely identified by a **Domain ID**.

- **Default value:** **42** (configurable as needed).
- **Purpose:** Prevents interference between multiple ROS 2 networks on the same physical infrastructure.

```bash
export ROS_DOMAIN_ID=42          # Set the ROS 2 Domain ID
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp  # Use Fast DDS as the middleware
```

## **6. Fast DDS Discovery Server**
For large-scale networks, a **Discovery Server** can replace UDP multicast, reducing network congestion and improving efficiency.

### **6.1 Configuring a Discovery Server in Fast DDS**
```xml
<participant>
    <discovery>
        <simple>
            <discoveryServer>
                <guidPrefix>01.0f.2c.00.00.01</guidPrefix>
                <listeningPort>11811</listeningPort>
            </discoveryServer>
        </simple>
    </discovery>
</participant>
```
This setup configures a **central discovery server** to streamline participant registration and minimize multicast traffic.

By leveraging DDS, ROS 2 achieves enhanced flexibility, reliability, and efficiency for distributed robotic systems.

