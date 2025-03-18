# Comprehensive Guide to QoS in ROS 2

## 1. Introduction to QoS in ROS 2
Quality of Service (QoS) in ROS 2 defines how messages are sent, received, and managed in a distributed system. Unlike ROS 1, which had a basic publish-subscribe model, ROS 2 uses DDS (Data Distribution Service), allowing users to fine-tune message delivery behavior.

QoS settings determine **reliability, durability, timing constraints, liveliness, network priority**, and **real-time behavior** of ROS 2 nodes.

### 1.1 Why QoS Matters?
QoS is essential for:
- **Reliable communication** in robotic systems.
- **Low-latency, high-priority data transmission** for real-time control.
- **Efficient use of network resources** in distributed applications.
- **Fault detection and recovery** in mission-critical tasks.

## 2. Core QoS Policies in ROS 2

### 2.1 Reliability QoS
Reliability determines how messages are delivered between nodes.

- **BEST_EFFORT**: Messages are sent but may be lost (faster, less overhead).
- **RELIABLE**: Messages are guaranteed to be delivered (slower but ensures delivery).

#### Example: Setting Reliability QoS
```python
from rclpy.qos import QoSProfile, ReliabilityPolicy

qos_profile = QoSProfile(reliability=ReliabilityPolicy.RELIABLE)
```
 **Use case**: Use RELIABLE for critical messages like **robot control commands**.

---

### 2.2 Durability QoS
Durability controls whether a subscriber receives past messages when it connects.

- **VOLATILE**: Only receives new messages (default behavior).
- **TRANSIENT_LOCAL**: Receives past messages from an active publisher.

#### Example: Setting Transient Local QoS
```python
from rclpy.qos import DurabilityPolicy

qos_profile = QoSProfile(durability=DurabilityPolicy.TRANSIENT_LOCAL)
```
 **Use case**: Use **TRANSIENT_LOCAL** for sensor data that should be available to late-joining subscribers.

---

### 2.3 History QoS
Controls how many messages are stored in a queue before discarding older ones.

- **KEEP_LAST (N)**: Stores the last N messages.
- **KEEP_ALL**: Stores all messages (not recommended for high-frequency topics).

#### Example: Setting History QoS
```python
from rclpy.qos import HistoryPolicy

qos_profile = QoSProfile(history=HistoryPolicy.KEEP_LAST, depth=10)
```
 **Use case**: Keep **10 most recent** sensor readings.

---

### 2.4 Deadline QoS (Timing Constraint for Messages)
If a subscriber expects messages at fixed intervals, but a publisher fails to send them within that time, the subscriber can detect a missed deadline and trigger a fallback mechanism.

#### Example: Detecting Missed Deadlines
```python
from rclpy.qos import Deadline

def deadline_missed_callback(event):
    print("Message deadline missed! Taking action...")

qos_profile = QoSProfile(deadline=Deadline(duration=1.0))  # 1 second
subscription.set_on_deadline_missed(deadline_missed_callback)
```
**Use case**: Emergency **robot stop** if commands aren’t received on time.

---

### 2.5 Lifespan QoS (Discarding Old Messages)
Some messages become useless after a certain time (e.g., sensor data, camera frames). Lifespan QoS ensures outdated messages aren’t delivered.

#### Example: Configuring Lifespan QoS
```python
from rclpy.qos import Lifespan

qos_profile = QoSProfile(lifespan=Lifespan(duration=2.0))  # 2-second lifespan
```
**Use case**: Discard **old camera frames** after 2 seconds.

---

### 2.6 Liveliness QoS (Ensuring Node Activity)
Liveliness ensures a node is still running by periodically announcing its presence. If it fails to do so, it is assumed dead.

Two types of liveliness:

Automatic: ROS 2 handles liveliness in the background.
Manual by Topic: The node must explicitly publish to remain active.

#### Example: Setting Liveliness QoS
```python
from rclpy.qos import LivelinessPolicy, Liveliness

qos_profile = QoSProfile(
    liveliness=LivelinessPolicy.MANUAL_BY_TOPIC, 
    lease_duration=5.0
)
```
**Use case**: If a **teleoperation node stops responding**, the robot stops moving.

---

### 2.7 Latency Budget QoS (Message Urgency)
Hints to the transport layer about how urgent a message is. Lower values request faster delivery,but it doesn't guarantee it.

#### Example: Assigning Latency Budget
```python
from rclpy.qos import LatencyBudget

qos_profile = QoSProfile(latency_budget=LatencyBudget(duration=0.05))  # 50ms
```
**Use case**: Optimize bandwidth in **swarm robotics**.

---

### 2.8 Transport Priority QoS (Prioritizing Critical Data)
Determines the importance of messages over the network.

#### Example: Assigning Transport Priority
```xml
<dds>
    <transport_priority>
        <value>10</value>
    </transport_priority>
</dds>
```
**Use case**: Emergency stop commands **should be transmitted first**.

---

### 2.9 Ownership QoS (Publisher Control Over Topics)
When multiple publishers send messages on the same topic, Ownership QoS determines which one has priority.

#### Example: Assigning Exclusive Ownership
```xml
<dds>
    <ownership>
        <kind>EXCLUSIVE</kind>
        <strength>100</strength>
    </ownership>
</dds>
```
**Use case**: A **backup controller takes over** when the primary fails.

---
Mastering QoS in ROS 2 allows for **high-performance, real-time, and fault-tolerant robotic systems**. Properly tuning these settings ensures **efficient network usage**, **low-latency communication**, and **robust fault detection**.

---
