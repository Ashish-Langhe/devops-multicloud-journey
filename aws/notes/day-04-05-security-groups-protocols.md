# Day 04-05 | Security Groups, Protocols, TCP/UDP

**Dates:** 15th April – 16th April  
**Topics:** Firewall, Security Groups, Protocols, HTTP, HTTPS, TCP, UDP, Ports

---

## 🔥 What is a Firewall?

- A firewall = **Security Layer**
- Allows only **authorized requests** and **blocks unauthorized** ones
- In AWS → Firewall = **Security Group (SG)**

---

## 🛡️ Security Group (SG)

- SG works at the **server (EC2 instance) level**
- We define **rules** that allow or deny traffic
- SG rule contains: **Source, Protocol, Port, Destination**

### Two Types of Rules

| Rule Type | Direction | Purpose |
|-----------|-----------|---------|
| **Inbound** | Incoming → Server | Controls who can access the server |
| **Outbound** | Server → Outside | Controls what server can send out |

### SG Rule Structure
```
Source (IP)  →  Protocol  →  Port  →  Destination (Server IP)
```

### Example
```
source: laptop IP (174.1.2.3)
  → port 3000 → IRCTC app
  → port 5000 → Flipkart app
  → port 6000 → Amazon app
```

---

## 🔌 What is a Port?

- A port = **logical endpoint inside a server**
- Multiple applications can run on a **single server** using different ports
- **Total port range: 0 – 65535**

### Common Ports
| Port | Use |
|------|-----|
| 22 | SSH (Linux server login) |
| 3389 | RDP (Windows server login) |
| 80 | HTTP (web traffic) |
| 443 | HTTPS (secure web traffic) |

> 💡 **To SSH into a Linux server**, SG must allow port **22**  
> SG checks if port 22 is open — it does NOT validate the key

---

## 🌐 What is a Protocol?

- Protocol = **Set of rules for data communication**
- Defines **how data is transferred** between source and destination
- Example URL breakdown:
  ```
  http://192.3.4.5:5000
    ↑        ↑       ↑
  protocol  server  port
  ```

### HTTP vs HTTPS

| Protocol | Full Form | Data Transfer | Security |
|----------|-----------|---------------|----------|
| **HTTP** | Hyper Text Transfer Protocol | Plain text | ❌ Not secure |
| **HTTPS** | Hyper Text Transfer Protocol Secure | Encrypted | ✅ Secure |

```
HTTP:  sender → 134617281719 → receiver (plain text, anyone can read)
HTTPS: sender → encrypted → receiver → decrypted (safe)
```

---

## 📡 TCP vs UDP

### TCP (Transmission Control Protocol)
- **Establishes connection** between source and destination first
- Only transfers data **after connection is verified**
- Used by HTTP/HTTPS
- **No data loss** — reliable
- Slightly slower (due to connection setup)

### UDP (User Datagram Protocol)
- **Does NOT establish or verify** any connection
- Data is sent directly — no handshake
- **Faster** — no setup time needed
- **Higher chance of data loss** — no dedicated connection
- Used for: video streaming, gaming, DNS

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Established first | No connection |
| Data Loss | No | Possible |
| Speed | Slower | Faster |
| Use Case | HTTP, SSH, file transfer | Video stream, gaming |

---

## 📌 Key Takeaways

- Security Group = Firewall at **server level**
- Always **whitelist only required ports** — don't open all ports
- `0.0.0.0/0` as source = **allows all IPs** (any IP can connect)
- For SSH: allow port 22, source = your specific IP (not 0.0.0.0/0 in production)
- HTTP = plain text, HTTPS = encrypted — always prefer HTTPS in production
