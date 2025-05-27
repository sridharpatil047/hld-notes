# Introduction to High-Level Design (HLD) | `#Day1`

## 📘 What is HLD?

- HLD focuses on **designing scalable and distributed systems**.
- Unlike DSA (Coding + Problem Solving) and LLD (Writing good structured, reusable, extensible code), HLD is about:
    - Structuring servers and components
    - Supporting millions of users
    - Making system scale horizontally

> ❗ Designing for 1,000 users is different from designing for 1M or 100M users.

---

## 🎯 Learning Goals

- Understand fundamental system design concepts.
- Solve 7–8 common case studies (e.g., URL Shortener, Social Media, etc.)
- Motivation: Be confident about system design, not threatened by LLMs like ChatGPT.

---

## 🧠 Real-World Example: **Delicious (2004)**

- Problem: Bookmarks lost when changing browsers/machines.
- Solution: Centralized bookmarking tool — Login and save bookmarks accessible from any device.
- Architecture:
    - Client: Any browser
    - Server: A single desktop acting as server

---

## ✅ Steps to Make Website Live

### Step 1: Buy Static Public IP

- **Public IP**: Accessible from the Internet
- **Private IP**: Only accessible within intranet
- **Static IP**: Fixed IP for servers (expensive, assigned by ISP)
- **Dynamic IP**: Temporary IP (cheap, suitable for clients)

> ISPs use **NAT (Network Address Translation)** to manage IP allocation

---

### Step 2: Buy Domain Name

- Domain purchased from registrars like GoDaddy.
- Browsers access websites using **IP addresses**, not domain names.

---

### Step 3: Map Domain to IP

- **ICANN**: Non-profit managing domain names and TLDs (.com, .org, etc.)
- Mapping process:
    - Registrar sends domain ownership to ICANN.
    - Owner sends IP address to registrar.
    - Registrar updates ICANN with IP mapping.
    - Now browsers can resolve domain → IP.

---

## ❌ Problems with ICANN

### 1. Latency
- ICANN servers are in Europe → Higher latency for other regions (e.g., India).

### 2. Traffic Bottleneck
- Every domain resolution request hits ICANN → High traffic load.

### 3. Single Point of Failure (SPOF)
- ICANN downtime = Internet-wide DNS failures.

---

## ✅ Solutions to ICANN Issues

### Key Concepts

- **Sharding**: Split domain responsibilities by TLD.
- **Replication**: Multiple copies of TLD servers to avoid SPOF.
- **Caching**: Store frequently used domain-IP mappings closer to users.

### DNS Caching Strategy

- Cache only frequently accessed domains in each region.
- First-time domain lookup → Slower.
- Cached domain → Faster access.

#### Neighborhood DNS

- Maintained by ISPs (e.g., Airtel, Jio)
- Also by companies like Google, Facebook to improve access speed.

---

## 🌍 DNS Hierarchical System

- **TLD Servers**: Managed and replicated by ICANN
- **Geographical Caches**: Maintained by ISPs, Google, etc.
    - Continent → Country → City → Neighborhood
- Use **MFU (Most Frequently Used)** cache replacement policy.

---

## 🔁 Recursive DNS Lookup

1. Check Browser Cache
2. Check OS Cache
3. Check Neighborhood DNS
4. Check ISP/Root DNS
5. ICANN TLD → Authoritative DNS

> DNS caches attach a **TTL** (Time-To-Live). After expiry, new mapping is fetched.

---

## 🌐 How Internet Works (Flow)

1. User types `www.delicio.us` in browser.
2. Recursive DNS lookup fetches IP.
3. Browser connects to IP.
4. Server responds.

---

## 🧩 DNS Server Types
![how-dns-works](diagrams/how-dns-works.png)
- **Recursive DNS (Resolver)**:
    - First server to receive query from user.
    - Checks cache or queries others to resolve domain name.

- **Root DNS Servers**:
    - Provide list of TLD servers (.com, .org, etc.)

- **TLD DNS Servers**:
    - Direct to authoritative servers for specific domain extensions.

- **Authoritative DNS Servers**:
    - Final source of truth. Holds actual domain-IP records.

---

## 🌐 DNS Resolution Example (`https://xyz.com`)

1. Browser → Recursive DNS (via ISP)
2. If not in cache → Request sent to Root DNS
3. Root DNS → Returns TLD (.com) server
4. TLD Server → Returns Authoritative server for xyz.com
5. Authoritative DNS → Returns IP address
6. Resolver caches & returns IP → Browser connects

---
---

# APIs and Scaling Challenges – delicio.us

## 🧩 Exposed APIs

### Authentication
```java
authenticate(user_id, password, meta_data)
// Returns access token (e.g., JWT)
```

### Bookmark Operations
```java
createBookmark(access_token, name, url, timestamp)
getBookmarks(access_token, noOfBookmarks, offset)
```

---

## 💥 Challenges with Desktop-based Setup

### Limitations at Scale
- Works well for a few hundred/thousand users.
- Fails at scale (1M–100M users), especially with concurrency.

### Issues
- **Single Point of Failure**: If desktop crashes or restarts, service goes down.
- **Resource Bottlenecks**:
  - Compute
  - Memory
  - Disk
  - Network bandwidth

---

## 🛠️ Solutions

### ✅ Addressing Failures and Bottlenecks

| Problem                 | Solution       |
|------------------------|----------------|
| Single Point of Failure | **Replication** |
| Resource Bottleneck     | **Sharding**    |

#### Types of Sharding
- **CPU / RAM Bottleneck** → Horizontal scaling of App Servers
- **Disk Bottleneck** → Horizontal scaling of Database Servers

---

## 🧱 Scaling Types

### 🔼 Vertical Scaling
- **Definition**: Upgrade to a more powerful machine.
- **Cluster**: A group of servers running the same code.

#### 🔻 Drawbacks
- Doesn't eliminate single point of failure.
- Expensive (due to high-end hardware and R&D).
- Hard to estimate future capacity.
- No elasticity for fluctuating traffic (e.g., during festivals).

---

### ➕ Horizontal Scaling (Distributed Computing)
- **Definition**: Add more machines instead of upgrading one.
- **Works by**: Running multiple low-cost servers in parallel.

#### ✅ Advantages
- No single point of failure.
- Cost-effective (uses commodity hardware).
- Easier to scale incrementally.
- Elastic: Can scale up/down based on traffic.

#### 🔻 Disadvantages
- Requires **code restructuring**.
- Need to implement:
  - Load balancing logic
  - Data distribution logic
  - Inter-service communication
- **Consistency issues** arise (multiple data sources).
- More architectural complexity.

---

### 🔍 Note
> In CAP Theorem, Vertical Scaling aligns more with **CA (Consistency + Availability)** model.
