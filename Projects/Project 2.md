<img width="1360" height="742" alt="project2-preview" src="https://github.com/user-attachments/assets/bc7e23f2-d173-4844-8f67-55be11bda865" />



# 3D E-Commerce Platform — AWS Architecture

## Connection Explanations

**Legend:**
- Solid arrow = Main traffic flow (every user request)
- Dashed arrow = Supporting / async / monitoring traffic

---

## Section 1 — Edge Layer Connections

These connections happen before traffic enters the VPC.

### 1. User → Route 53
- Type: Solid  
- Flow: Global Users → Route 53 (DNS lookup)  
- Why: Resolves domain using latency-based routing to the nearest CloudFront endpoint.

### 2. Route 53 → CloudFront
- Type: Solid  
- Flow: Route 53 → CloudFront CDN  
- Why: Hands off DNS resolution to CloudFront for content delivery.

### 3. CloudFront → Shield Standard
- Type: Dashed (passive)  
- Flow: CloudFront → Shield  
- Why: Provides DDoS protection through passive monitoring.

### 4. CloudFront → AWS WAF
- Type: Solid  
- Flow: CloudFront → WAF  
- Why: Inspects requests for malicious patterns such as SQL injection, XSS, and bots.

---

## Section 2 — Content Delivery

### 5. WAF → Internet Gateway
- Type: Solid  
- Flow: WAF → Internet Gateway  
- Why: Sends approved API traffic into the VPC.

### 6. CloudFront → S3 (3D Assets)
- Type: Solid  
- Flow: CloudFront → S3  
- Why: Fetches assets on cache miss and caches them at the edge.

---

## Section 3 — Public Subnet Networking

### 7. Internet Gateway → Application Load Balancer
- Type: Solid  
- Flow: Internet Gateway → ALB  
- Why: Entry point into the VPC; ALB distributes traffic to EC2 instances.

### 8. Application Load Balancer → Cognito
- Type: Solid  
- Flow: ALB → Cognito  
- Why: Validates JWT tokens before forwarding requests.

### 9. EC2 → NAT Gateway
- Type: Dashed  
- Flow: EC2 → NAT Gateway  
- Why: Enables outbound internet access from private subnet instances.

### 10. NAT Gateway → Internet Gateway
- Type: Dashed  
- Flow: NAT Gateway → Internet Gateway  
- Why: Sends outbound traffic to the internet and returns responses.

---

## Section 4 — Application Layer

### 11–13. ALB → EC2 (Multi-AZ)
- Type: Solid  
- Flow: ALB → EC2 (across 3 Availability Zones)  
- Why: Distributes traffic for high availability and fault tolerance.

### 14. EC2 → Cognito
- Type: Dashed  
- Flow: EC2 → Cognito  
- Why: Retrieves additional identity data when required.

### 15. EC2 → Lambda
- Type: Dashed  
- Flow: EC2 → Lambda  
- Why: Triggers asynchronous processing tasks.

### 16. EC2 → DynamoDB
- Type: Solid  
- Flow: EC2 → DynamoDB  
- Why: Handles product catalog, cart, and session data.

### 17. EC2 → RDS Aurora
- Type: Solid  
- Flow: EC2 → RDS Aurora  
- Why: Handles transactional data such as orders and payments.

### 18. EC2 → CloudWatch
- Type: Dashed  
- Flow: EC2 → CloudWatch  
- Why: Sends metrics and logs for monitoring and auto-scaling.

---

## Section 5 — Event-Driven Tasks

### 19. Lambda → SNS
- Type: Solid  
- Flow: Lambda → SNS  
- Why: Publishes events for fan-out notifications.

### 20. SNS → SES
- Type: Solid  
- Flow: SNS → SES  
- Why: Sends order confirmation emails.

### 21. S3 → Lambda
- Type: Solid  
- Flow: S3 → Lambda  
- Why: Triggers processing when new assets are uploaded.

### 22. Lambda → DynamoDB
- Type: Dashed  
- Flow: Lambda → DynamoDB  
- Why: Updates product metadata after processing.

---

## Section 6 — Data Layer

### 23. RDS Aurora → Standby
- Type: Dashed  
- Flow: Primary → Standby  
- Why: Provides synchronous replication for high availability.

### 24. EC2 → S3 (Uploads)
- Type: Solid  
- Flow: EC2 → S3  
- Why: Handles merchant uploads of 3D assets.

### 25. S3 → EC2 (Frontend Bundle)
- Type: Dashed  
- Flow: S3 → EC2  
- Why: Used during deployment and application startup.

---

## Summary

This architecture provides:
- Low-latency global content delivery
- Strong security and request filtering
- Scalable and fault-tolerant compute layer
- Efficient data storage for both real-time and transactional workloads
- Decoupled event-driven processing


