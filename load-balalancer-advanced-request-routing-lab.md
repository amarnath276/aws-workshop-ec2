# Load Balanced Architecture with Advanced Request Routing

This lab guide walks you through deploying a simple website hosted on Amazon EC2, fronted by an Application Load Balancer (ALB). You’ll configure both **host-based** and **path-based routing** rules to direct traffic using the host header or URL path.

---

## Overview

With **host-based routing**, you can route traffic to multiple domains using a single ALB. **Path-based routing** (also known as URL-based routing) allows you to forward requests to different targets based on the request path.

---

## 🧾 Requirements

- AWS Free Tier Account
- Basic knowledge of the AWS Management Console
- A registered domain name in Route 53 (see instructions in your setup documentation)

---

## 🔗 Resources

Download the ZIP file: `advanced-request-routing-code.zip`

---

## 🧪 Exercise Overview

1. Create EC2 Instances (Red & Blue)
2. Configure Path-based Routing
3. Configure Host-based Routing
4. Clean Up Resources

---

## Exercise 1 – Create Red and Blue EC2 Instances

### Task 1 – Create an S3 Bucket & Upload Code

1. Go to **S3 Console** → Click **Create bucket**
2. Name the bucket something unique like `arr-bucket-123456`
3. Scroll and click **Create bucket**
4. Upload all course files (except user data and permissions files)

---

### Task 2 – Create a Security Group

1. Go to the **EC2 Console**
2. Under **Network & Security**, click **Security Groups** → **Create Security Group**
3. Name: `WebsiteSG`
4. Inbound Rules:
   - Type: HTTP
   - Source: Anywhere (0.0.0.0/0)
5. Click **Create security group**

---

### Task 3 – Create IAM Role and Launch Red Instance

1. Go to **IAM Console** → **Policies** → Click **Create policy**
2. Choose **JSON** and paste content from `bucket-permissions.json`, replacing `YOUR-BUCKET-ARN`
3. Name the policy: `S3-ARR-Policy`
4. Go to **Roles** → **Create role**
   - Use case: EC2
   - Attach policy: `S3-ARR-Policy`
   - Name: `S3-ARR-Role`

#### Launch the Red EC2 Instance

1. In EC2 Console → Click **Launch instance**
2. Name: `Red`
3. AMI: Amazon Linux 2 (Free Tier eligible)
4. Instance Type: `t2.micro`
5. Key Pair: Proceed without a key
6. Network Settings: Use `WebsiteSG` and subnet `us-east-1a`
7. Expand **Advanced details**:
   - IAM instance profile: `S3-ARR-Role`
   - User data: Use content from `user-data-red`, replacing `YOUR-BUCKET-NAME`
8. Launch instance

Repeat the steps to launch the **Blue** instance using `user-data-blue` and subnet `us-east-1b`.

---

## Exercise 2 – Path-Based Routing

### Task 1 – Create Target Groups

1. Go to **Target Groups** → Click **Create target group**
2. Create:
   - **Red** target group with health check path `/red/index.html`
   - **Blue** target group with health check path `/blue/index.html`
3. Register the respective EC2 instance to each target group

---

### Task 2 – Create Application Load Balancer

1. Go to **Load Balancers** → Click **Create Load Balancer**
2. Choose **Application Load Balancer**
3. Name: `LabLoadBalancer`
4. Scheme: Internet-facing
5. Subnets: `us-east-1a`, `us-east-1b`
6. Security Group: `WebsiteSG`
7. Listener: HTTP (port 80)
8. Click **Create Load Balancer**

Once the ALB is **Active**, configure listener rules:

#### Add Path-Based Rules

1. Go to **Listeners and rules** → Select Listener → Click **Manage rules**
2. Add Rule:
   - Condition: Path is `/red*`
   - Action: Forward to **Red** target group
   - Priority: 1
3. Add another rule:
   - Condition: Path is `/blue*`
   - Action: Forward to **Blue** target group
   - Priority: 2

Test using ALB DNS name + `/red` or `/blue`

---

## Exercise 3 – Host-Based Routing

### Task 1 – Update Listener Rules

1. Go to Listener → Click **Edit Rules**
2. Delete existing path-based rules
3. Add new rules:

#### Red Subdomain

- Condition: Host is `red.yourdomain.com`
- Action: Forward to **Red** target group
- Priority: 1

#### Blue Subdomain

- Condition: Host is `blue.yourdomain.com`
- Action: Forward to **Blue** target group
- Priority: 2

---

### Task 2 – Configure Route 53 DNS Records

1. Open **Route 53** → Go to your **Hosted Zone**
2. Click **Create record** for each:

#### Blue Record

- Name: `blue`
- Type: A (Alias)
- Route to: Application Load Balancer in N. Virginia
- Target: Select your ALB

#### Red Record

- Name: `red`
- Same configuration as above

#### Apex Domain (Optional)

- Leave name field blank
- Type: A (Alias)
- Point to same ALB

Test with:

- `http://red.yourdomain.com`
- `http://blue.yourdomain.com`

---

## Exercise 4 – Clean Up Resources

- Delete the Application Load Balancer
- Delete Target Groups
- Delete EC2 Instances
- Delete S3 Bucket
- Delete Security Groups
- Delete Route 53 Records

---

> ⚠️ **Note:** Some resources (e.g., EC2, S3, ALB) may incur charges. Always clean up after completing labs.
