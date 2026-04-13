<img width="1360" height="742" alt="project2-preview" src="https://github.com/user-attachments/assets/bc7e23f2-d173-4844-8f67-55be11bda865" />





# 3D E-Commerce Platform · Connection Explanations

---

## Connection Explanations

**Solid arrow** = main traffic flow, data travels this path on every user request  
**Dashed arrow** = supporting, async, monitoring, or optional traffic  

---

## Section 1 — Edge Layer Connections

These connections form the outermost path a user request takes before entering any AWS infrastructure. They happen before the VPC is involved.

---

### Connection 1 — User → Route 53  
FROM Global Users → TO Route 53 · DNS lookup  

**Why:**  
When a user types the shop address into their browser, their device asks Route 53 where to find it. Route 53 uses latency-based routing to respond with the IP address of the nearest and fastest endpoint — CloudFront in this case. This is the very first step of every user interaction.

**Line type:**  
Solid — every request starts here  

---

### Connection 2 — Route 53 → CloudFront  
FROM Route 53 → TO CloudFront CDN · Directs request to edge  

**Why:**  
Route 53 resolves the domain name to a CloudFront distribution endpoint. CloudFront then takes over as the entry point for all content delivery. The user's browser now sends its HTTP request to CloudFront.

**Line type:**  
Solid — every user request flows through this  

---

### Connection 3 — CloudFront → Shield Standard  
FROM CloudFront CDN → TO Shield Standard · DDoS monitoring (passive)  

**Why:**  
This is not a traffic routing connection. Shield Standard is a passive protection layer that monitors traffic patterns and blocks volumetric DDoS attacks. Traffic does not pass through Shield.

**Line type:**  
Dashed — passive monitoring only  

---

### Connection 4 — CloudFront → AWS WAF  
FROM CloudFront CDN → TO AWS WAF · Content inspection  

**Why:**  
WAF actively inspects every HTTP/HTTPS request. It detects SQL injection, XSS, bot traffic, and rate abuse. Malicious requests are blocked before entering the VPC.

**Line type:**  
Solid — every request is inspected  

---

## Section 2 — Content Delivery Connections

These connections describe how static content and 3D assets reach the user.

---

### Connection 5 — WAF → Internet Gateway  
FROM AWS WAF → TO Internet Gateway · API requests  

**Why:**  
After WAF approves a request, it forwards it to the Internet Gateway — the entry point into the VPC. Only API requests follow this path.

**Line type:**  
Solid — API traffic enters the VPC  

---

### Connection 6 — CloudFront → S3 (3D Assets bucket)  
FROM CloudFront CDN → TO S3 — 3D Assets · Fetches assets (cache miss)  

**Why:**  
CloudFront serves cached assets if available. Otherwise, it fetches from S3 using a private connection and then caches the result at the edge.

**Line type:**  
Solid — main asset delivery path  

---

## Section 3 — Public Subnet Networking Connections

---

### Connection 7 — Internet Gateway → App Load Balancer  
FROM Internet Gateway → TO App Load Balancer · Traffic distribution  

**Why:**  
The Internet Gateway passes incoming requests to the ALB, which distributes traffic to EC2 instances.

**Line type:**  
Solid  

---

### Connection 8 — App Load Balancer → Cognito  
FROM App Load Balancer → TO Cognito · Validates token  

**Why:**  
The ALB validates JWT tokens with Cognito before forwarding requests. Invalid requests are rejected immediately.

**Line type:**  
Solid  

---

### Connection 9 — EC2 → NAT Gateway  
FROM EC2 → TO NAT Gateway · Outbound traffic  

**Why:**  
Private EC2 instances use NAT Gateway to access external services (e.g., APIs, updates) without being publicly exposed.

**Line type:**  
Dashed  

---

### Connection 10 — NAT Gateway → Internet Gateway  
FROM NAT Gateway → TO Internet Gateway · Internet access  

**Why:**  
NAT forwards outbound traffic to the internet and returns responses back to EC2.

**Line type:**  
Dashed  

---

## Section 4 — Application Layer Connections

---

### Connection 11–13 — ALB → EC2 (Multi-AZ)  
FROM App Load Balancer → TO EC2 (3 AZs) · Request distribution  

**Why:**  
The ALB distributes traffic across multiple availability zones for high availability and fault tolerance.

**Line type:**  
Solid  

---

### Connection 14 — EC2 → Cognito  
FROM EC2 → TO Cognito · Token verification  

**Why:**  
EC2 sometimes directly verifies tokens when additional identity data is required.

**Line type:**  
Dashed  

---

### Connection 15 — EC2 → Lambda  
FROM EC2 → TO Lambda · Event trigger  

**Why:**  
Triggers asynchronous processes such as notifications and background jobs.

**Line type:**  
Dashed  

---

### Connection 16 — EC2 → DynamoDB  
FROM EC2 → TO DynamoDB · Catalog / cart  

**Why:**  
Handles product browsing, cart updates, and session data with low latency.

**Line type:**  
Solid  

---

### Connection 17 — EC2 → RDS Aurora  
FROM EC2 → TO RDS Aurora · Orders / payments  

**Why:**  
Handles transactional operations with ACID consistency during checkout.

**Line type:**  
Solid  

---

### Connection 18 — EC2 → CloudWatch  
FROM EC2 → TO CloudWatch · Metrics and logs  

**Why:**  
Provides monitoring, logging, and auto-scaling triggers.

**Line type:**  
Dashed  

---

## Section 5 — Event-Driven Task Connections

---

### Connection 19 — Lambda → SNS  
FROM Lambda → TO SNS · Event publishing  

**Why:**  
Publishes events for fan-out to multiple services.

**Line type:**  
Solid  

---

### Connection 20 — SNS → SES  
FROM SNS → TO SES · Email delivery  

**Why:**  
Sends order confirmation emails.

**Line type:**  
Solid  

---

### Connection 21 — S3 → Lambda  
FROM S3 → TO Lambda · Upload trigger  

**Why:**  
Triggers processing when new assets are uploaded.

**Line type:**  
Solid  

---

### Connection 22 — Lambda → DynamoDB  
FROM Lambda → TO DynamoDB · Update records  

**Why:**  
Updates product data after asset processing.

**Line type:**  
Dashed  

---

## Section 6 — Data Layer Connections

---

### Connection 23 — RDS Aurora → Standby  
FROM RDS Primary → TO RDS Standby · Replication  

**Why:**  
Ensures high availability with synchronous replication.

**Line type:**  
Dashed  

---

### Connection 24 — EC2 → S3  
FROM EC2 → TO S3 · Upload  

**Why:**  
Handles merchant uploads of 3D assets.

**Line type:**  
Solid  

---

### Connection 25 — S3 → EC2  
FROM S3 → TO EC2 · Deployment  

**Why:**  
Used during deployment and startup.

**Line type:**  
Dashed  
