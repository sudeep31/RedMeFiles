# 🚀 Complete AWS Full-Stack Deployment Guide

## 📋 **What You'll Deploy**

A modern, loosely-coupled web application with:

- **Frontend:** Angular app on S3 + CloudFront
- **Backend:** Node.js API on ECS Fargate + API Gateway
- **Database:** MongoDB Atlas (AWS Marketplace)
- **Caching:** ElastiCache Redis
- **Security:** WAF + SSL certificates
- **Monitoring:** CloudWatch comprehensive setup
- **Domain:** Custom domain with Route 53

---

## 🎯 **Architecture Overview**

Your application will follow this structure:

**Frontend Flow:**
Custom Domain → Route 53 → CloudFront → S3 (Angular Build)

**Backend Flow:**
api.yourdomain.com → Route 53 → Application Load Balancer → ECS Fargate → MongoDB Atlas

**Caching Layer:**
ECS Fargate ↔ ElastiCache Redis ↔ MongoDB Atlas

---

## 🏗️ **Detailed Architecture Diagram**

````mermaid
graph TB
    %% External Users
    User[End Users - Web Browsers]

    %% DNS and Domain Management
    subgraph DNS["DNS and Domain Layer"]
        Domain[yourdomain.com - Custom Domain]
        R53[Route 53 DNS Service<br/>A Record yourdomain.com to CloudFront<br/>CNAME www.yourdomain.com to CloudFront<br/>A Record api.yourdomain.com to ALB]
        ACM[Certificate Manager SSL/TLS<br/>*.yourdomain.com<br/>yourdomain.com<br/>api.yourdomain.com]
    end

    %% Frontend Infrastructure
    subgraph Frontend["Frontend Layer - Region us-east-1"]
        CF[CloudFront Global CDN<br/>Custom Domain yourdomain.com<br/>SSL Certificate from ACM<br/>Cache Policy CachingOptimized<br/>Origin S3 Static Website]
        S3[S3 Bucket Static Hosting<br/>Bucket yourdomain-frontend<br/>Static Website Hosting Enabled<br/>Angular Build Files<br/>Public Read Policy]
    end

    %% Security Layer
    subgraph Security["Security and Monitoring"]
        WAF[AWS WAF Web Application Firewall<br/>Rate Limiting<br/>SQL Injection Protection<br/>XSS Protection]
        CW[CloudWatch Monitoring<br/>ECS Container Insights<br/>ALB Access Logs<br/>CloudFront Logs<br/>Custom Metrics and Alarms]
    end

    %% Backend Infrastructure
    subgraph Backend["Backend Layer - VPC 10.0.0.0/16"]

        %% Load Balancing
        subgraph PublicSubnets["Public Subnets"]
            ALB[Application Load Balancer<br/>HTTPS Listener 443<br/>HTTP to HTTPS Redirect<br/>Target Group ECS Tasks<br/>Health Check /health]
            IGW[Internet Gateway<br/>Internet Access]
        end

        %% Application Layer
        subgraph PrivateSubnets["Private Subnets - Multi-AZ"]
            subgraph AZ1["AZ us-east-1a"]
                ECS1[ECS Fargate Task 1<br/>Node.js Container<br/>CPU 0.25 vCPU<br/>Memory 0.5 GB<br/>Port 3000<br/>Health Check Endpoint]
                NAT1[NAT Gateway 1<br/>Outbound Internet<br/>for Private Subnet 1]
            end

            subgraph AZ2["AZ us-east-1b"]
                ECS2[ECS Fargate Task 2<br/>Node.js Container<br/>CPU 0.25 vCPU<br/>Memory 0.5 GB<br/>Port 3000<br/>Auto Scaling Enabled]
                NAT2[NAT Gateway 2<br/>Outbound Internet<br/>for Private Subnet 2]
            end

            ECSCluster[ECS Cluster<br/>fullstackapp-cluster<br/>Launch Type Fargate<br/>Service 2 Tasks<br/>Auto Scaling 1-10 tasks<br/>Container Insights Enabled]
        end

        %% Data Layer
        subgraph DataLayer["Data and Cache Layer"]
            Redis[ElastiCache Redis<br/>In-Memory Cache<br/>Node Type cache.t3.micro<br/>Port 6379<br/>Encryption At Rest and Transit<br/>Subnet Group Private Subnets]

            MongoDB[MongoDB Atlas<br/>Primary Database<br/>Cluster M0 Sandbox Free<br/>Region us-east-1<br/>Network Access 0.0.0.0/0<br/>Database fullstackapp]
        end
    end

    %% Security Groups
    subgraph SecurityGroups["Security Groups"]
        ALBSG[ALB Security Group<br/>Inbound 80 443 from 0.0.0.0/0<br/>Outbound All traffic]
        ECSSG[ECS Security Group<br/>Inbound 3000 from ALB-SG<br/>Outbound All traffic]
        RedisS[Redis Security Group<br/>Inbound 6379 from ECS-SG<br/>Outbound All traffic]
    end

    %% CI/CD Pipeline
    subgraph CICD["CI/CD Pipeline"]
        GitHub[GitHub Repository<br/>Source Code<br/>Frontend Angular App<br/>Backend Node.js API<br/>Infrastructure Docker Files]
        Actions[GitHub Actions<br/>Build and Deploy<br/>Frontend Build to S3 to CloudFront<br/>Backend Docker to ECR to ECS<br/>Auto Invalidation]
        ECR[Elastic Container Registry<br/>Docker Images<br/>Backend Node.js Images<br/>Automatic Builds<br/>Image Scanning]
    end

    %% Environment Variables
    subgraph Config["Configuration Management"]
        SSM[Systems Manager<br/>Parameter Store<br/>MongoDB Connection String<br/>JWT Secrets<br/>Redis Configuration<br/>Environment Variables]
    end

    %% Flow Connections
    User -->|1. HTTPS Request yourdomain.com| Domain
    Domain -->|2. DNS Resolution| R53
    R53 -->|3. Route to CloudFront A Record| CF
    CF -->|4. SSL Termination Certificate from ACM| ACM
    CF -->|5. Serve Static Content| S3

    %% API Flow
    User -->|API Requests api.yourdomain.com| R53
    R53 -->|Route to ALB A Record| ALB
    ALB -->|Load Balance Health Check /health| ECS1
    ALB -->|Load Balance Health Check /health| ECS2

    %% Backend Connections
    ECS1 <-->|Cache Operations Port 6379| Redis
    ECS2 <-->|Cache Operations Port 6379| Redis
    ECS1 <-->|Database Operations MongoDB Atlas| MongoDB
    ECS2 <-->|Database Operations MongoDB Atlas| MongoDB

    %% Security & Monitoring
    WAF -->|Protect| CF
    CW -->|Monitor| CF
    CW -->|Monitor| ALB
    CW -->|Monitor| ECS1
    CW -->|Monitor| ECS2
    CW -->|Monitor| Redis

    %% Infrastructure Connections
    ALB -.->|Uses| ALBSG
    ECS1 -.->|Uses| ECSSG
    ECS2 -.->|Uses| ECSSG
    Redis -.->|Uses| RedisS

    %% CI/CD Flow
    GitHub -->|Trigger Build on Push| Actions
    Actions -->|Deploy Frontend S3 + CloudFront| S3
    Actions -->|Build and Push Docker Images| ECR
    ECR -->|Deploy to ECS Rolling Update| ECS1
    ECR -->|Deploy to ECS Rolling Update| ECS2

    %% Configuration
    ECS1 <-->|Fetch Secrets Environment Variables| SSM
    ECS2 <-->|Fetch Secrets Environment Variables| SSM

    %% Internet Access for Private Subnets
    ECS1 -->|Outbound Internet Package Updates API Calls| NAT1
    ECS2 -->|Outbound Internet Package Updates API Calls| NAT2
    NAT1 -->|Internet Access| IGW
    NAT2 -->|Internet Access| IGW

    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    classDef backend fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#388e3c,stroke-width:2px
    classDef security fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef cicd fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef dns fill:#f1f8e9,stroke:#689f38,stroke-width:2px

    class S3,CF frontend
    class ALB,ECS1,ECS2,ECSCluster backend
    class Redis,MongoDB database
    class WAF,CW,ALBSG,ECSSG,RedisS,ACM security
    class GitHub,Actions,ECR,SSM cicd
    class Domain,R53 dns
```

### 🔍 **Architecture Components Details**

#### **🌐 DNS & Domain Management**

- **Custom Domain**: Your purchased domain (yourdomain.com)
- **Route 53**: DNS service with A records for main site and API subdomain
- **Certificate Manager**: SSL certificates for HTTPS across all services
- **Domain Routing**:
  - `yourdomain.com` → CloudFront Distribution
  - `www.yourdomain.com` → CloudFront Distribution
  - `api.yourdomain.com` → Application Load Balancer

#### **📱 Frontend Infrastructure**

- **CloudFront**: Global CDN for Angular app with custom domain SSL
- **S3 Bucket**: Static website hosting for Angular build files
- **WAF**: Web Application Firewall protecting CloudFront
- **SSL Termination**: Certificate Manager handles all HTTPS

#### **🏗️ Backend Infrastructure - VPC Architecture**

- **VPC**: 10.0.0.0/16 CIDR block across 2 Availability Zones
- **Public Subnets**: ALB and Internet Gateway (10.0.1.0/24, 10.0.2.0/24)
- **Private Subnets**: ECS Tasks and Redis (10.0.11.0/24, 10.0.12.0/24)
- **NAT Gateways**: Provide internet access for private subnet resources
- **Application Load Balancer**: HTTPS termination and health checking
- **ECS Fargate**: Serverless container hosting for Node.js API

#### **🗄️ Data Layer**

- **MongoDB Atlas**: Primary database with AWS integration
- **ElastiCache Redis**: In-memory caching for performance
- **Systems Manager**: Secure parameter storage for secrets

#### **🔒 Security Architecture**

- **Security Groups**: Network-level firewalling
  - ALB: Allows 80/443 from internet
  - ECS: Allows 3000 from ALB only
  - Redis: Allows 6379 from ECS only
- **IAM Roles**: Least privilege access for ECS tasks
- **Encryption**: At rest and in transit for all data

#### **🚀 CI/CD Pipeline**

- **GitHub**: Source code repository
- **GitHub Actions**: Automated build and deployment
- **ECR**: Docker image registry for backend
- **Automated Deployment**:
  - Frontend: Build → S3 → CloudFront invalidation
  - Backend: Docker build → ECR → ECS rolling update

### 💰 **Cost Breakdown by Component**

| Component               | Monthly Cost        | Usage                   |
| ----------------------- | ------------------- | ----------------------- |
| **Route 53**            | $0.50               | Hosted zone             |
| **Certificate Manager** | Free                | SSL certificates        |
| **S3 + CloudFront**     | $1-3                | Static hosting + CDN    |
| **ALB**                 | $22                 | Load balancer           |
| **ECS Fargate**         | $15-25              | 2 small tasks           |
| **NAT Gateways**        | $90                 | 2 gateways (major cost) |
| **ElastiCache**         | $10-15              | t3.micro Redis          |
| **MongoDB Atlas**       | Free                | M0 sandbox tier         |
| **Total**               | **~$140-155/month** | Full production setup   |

### 🎯 **Traffic Flow Examples**

#### **Frontend User Request:**

1. User visits `https://yourdomain.com`
2. DNS query to Route 53 → Returns CloudFront IP
3. Request to CloudFront → SSL termination with ACM certificate
4. CloudFront serves cached content or fetches from S3
5. Response delivered globally via edge locations

#### **Backend API Request:**

1. Frontend makes API call to `https://api.yourdomain.com/users`
2. DNS query to Route 53 → Returns ALB IP address
3. Request to ALB → SSL termination and health check
4. ALB forwards to healthy ECS task in private subnet
5. ECS task processes request:
   - Checks Redis cache first
   - Queries MongoDB Atlas if cache miss
   - Updates cache and returns response
6. Response flows back through ALB to frontend

#### **Deployment Flow:**

1. Developer pushes code to GitHub
2. GitHub Actions triggered automatically
3. **Frontend**: Angular build → Upload to S3 → CloudFront invalidation
4. **Backend**: Docker build → Push to ECR → ECS rolling update
5. Health checks ensure zero-downtime deployment

---

## 📚 **Table of Contents**

### **Part 1: AWS Account Setup & Prerequisites** _(✅ Complete)_

- ✅ AWS Account Creation & Free Tier Setup
- ✅ IAM Security Configuration (Root + Admin User)
- ✅ AWS CLI Installation & Configuration
- ✅ Cost Management & Budget Alerts
- ✅ Region Selection & Validation

### **Part 2: Domain & DNS Setup** _(✅ Complete)_

- ✅ Domain Registration via Route 53
- ✅ SSL Certificate Setup with Certificate Manager
- ✅ DNS Configuration & Security
- ✅ Health Checks & Monitoring

### **Part 3: Frontend Deployment (Angular)** _(✅ Complete)_

- ✅ S3 Bucket Creation & Configuration
- ✅ CloudFront Distribution Setup
- ✅ Angular Build & Deployment Process
- ✅ DNS Record Updates

### **Part 4: Backend Infrastructure** _(✅ Complete)_

- ✅ VPC & Security Groups Setup
- ✅ ECS Cluster Configuration
- ✅ Application Load Balancer
- ✅ Auto Scaling & Health Checks

### **Part 5: Database & Caching** _(✅ Complete)_

- ✅ MongoDB Atlas Integration
- ✅ ElastiCache Redis Setup
- ✅ Database Security & Backups
- ✅ Connection Configuration

### **Part 6: Security & Monitoring** _(✅ Complete)_

- ✅ WAF Configuration & Rules
- ✅ CloudWatch Logging & Metrics
- ✅ Security Best Practices
- ✅ Performance Monitoring

### **Part 7: CI/CD Pipeline** _(✅ Complete)_

- ✅ GitHub Actions Setup & Configuration
- ✅ Jenkins Alternative Setup
- ✅ Docker Containerization
- ✅ Automated Build & Deploy
- ✅ Environment Management
- ✅ Testing & Quality Gates

---

# 🏁 **Part 1: AWS Account Setup & Prerequisites**

## 🎯 **What We'll Cover in This Section**

Setting up your AWS environment properly from scratch, including cost controls and security fundamentals.

---

## 📋 **Step 1: AWS Account Creation**

### **1.1 Create Your AWS Account**

Navigate to AWS Homepage:

- Go to aws.amazon.com
- Click "Create an AWS Account"
- Choose "Personal" account type for learning
- Enter your email address and choose account name
- Verify email and complete phone verification
- Add payment method (required even for free tier)

### **1.2 Enable Free Tier Tracking**

Immediately after account creation:

- Go to AWS Console → Billing Dashboard
- Click "Free tier" in left navigation
- Review your free tier usage limits
- Set up free tier usage alerts

**Important Free Tier Limits for Your Project:**

- EC2: 750 hours per month (covers ECS Fargate partially)
- S3: 5GB storage, 20,000 GET requests, 2,000 PUT requests
- CloudFront: 50GB data transfer, 2,000,000 requests
- Route 53: First 25 hosted zones free
- ElastiCache: 750 hours of cache.t2.micro nodes

---

## 🔐 **Step 2: Security Setup (Critical First Step)**

### **2.1 Secure Your Root Account**

**Enable MFA on Root Account:**

- AWS Console → Account dropdown (top right)
- Click "My Security Credentials"
- Expand "Multi-factor authentication (MFA)"
- Click "Activate MFA"
- Choose "Virtual MFA device"
- Use Google Authenticator or similar app
- Scan QR code and enter two consecutive MFA codes

**Create Strong Root Password:**

- Go to "Password" section
- Click "Change password"
- Create strong, unique password
- Store securely (never share root access)

### **2.2 Create IAM Admin User (Never Use Root for Daily Work)**

Navigate to IAM Service:

- Search "IAM" in AWS Console search bar
- Click "Identity and Access Management (IAM)"

Create Admin Group:

- Click "User groups" in left sidebar
- Click "Create group"
- Group name: "AdminGroup"
- Attach policy: "AdministratorAccess"
- Click "Create group"

Create Your Personal Admin User:

- Click "Users" in left sidebar
- Click "Create user"
- Username: your-name-admin (e.g., "john-admin")
- Select "Provide user access to the AWS Management Console"
- Choose "I want to create an IAM user"
- Console password: Choose "Custom password"
- Uncheck "Users must create a new password at next sign-in"
- Click "Next"

Assign to Admin Group:

- Select "Add user to group"
- Check "AdminGroup"
- Click "Next"
- Review and click "Create user"

**Download Credentials:**

- Important: Download the CSV file with login credentials
- Save securely - you'll need these to log in

### **2.3 Enable MFA for Your Admin User**

Sign out of root account and sign in with your new admin user:

- Use the IAM sign-in URL provided
- Enter your admin username and password

Enable MFA:

- Click your username (top right) → "My Security Credentials"
- Click "Assign MFA device"
- Device name: your-phone-name
- Choose "Authenticator app"
- Scan QR with authenticator app
- Enter two consecutive codes
- Click "Add MFA"

---

## 💰 **Step 3: Cost Management & Budgets**

### **3.1 Set Up Billing Alerts**

Enable Billing Alerts:

- Go to Billing Console → "Billing preferences"
- Check "Receive Billing Alerts"
- Check "Receive Free Tier Usage Alerts"
- Enter your email address
- Click "Save preferences"

### **3.2 Create Budget Alerts**

Create Monthly Budget:

- Go to AWS Budgets → Click "Create budget"
- Choose "Cost budget"
- Budget name: "Monthly-Learning-Budget"
- Period: Monthly
- Budget amount: 20 USD (adjust based on your comfort)
- Click "Next"

Set Alert Thresholds:

- Alert 1: 50% of budget (10 USD)
- Alert 2: 80% of budget (16 USD)
- Alert 3: 100% of budget (20 USD)
- Email: your email address
- Click "Next" → "Create budget"

### **3.3 Resource Tagging Strategy**

Set up consistent tagging for cost tracking:

- All resources will use these tags:
  - Project: FullStackApp
  - Environment: Learning
  - Owner: YourName
  - CreatedDate: YYYY-MM-DD

---

## 🛠️ **Step 4: AWS CLI Setup**

### **4.1 Install AWS CLI**

For Windows:

- Download AWS CLI MSI installer from aws.amazon.com/cli
- Run installer as administrator
- Verify installation: Open PowerShell and run "aws --version"

For Mac:

- Install via Homebrew: "brew install awscli"
- Or download from AWS website

For Linux:

- Use package manager or download from AWS website

### **4.2 Configure AWS CLI**

Create Access Keys for CLI:

- Go to IAM → Users → your-admin-user
- Click "Security credentials" tab
- Scroll to "Access keys"
- Click "Create access key"
- Choose "Command Line Interface (CLI)"
- Check acknowledgment checkbox
- Optional: Add description tag
- Click "Create access key"
- Download CSV or copy keys (store securely!)

Configure CLI:

- Open terminal/PowerShell
- Run: aws configure
- Enter Access Key ID
- Enter Secret Access Key
- Default region: us-east-1 (recommended for learning)
- Default output format: json

Test Configuration:

- Run: aws sts get-caller-identity
- Should return your user information

---

## 🌍 **Step 5: Choose Your AWS Region**

### **5.1 Region Selection Strategy**

For learning purposes, choose based on:

**Recommended: us-east-1 (Northern Virginia)**

- Lowest costs for most services
- All services available
- Best for CloudFront (global edge locations)
- Default region for many AWS services

**Alternative: us-west-2 (Oregon)**

- Good performance for US West Coast
- Lower costs than us-west-1
- All modern services available

**European Options:**

- eu-west-1 (Ireland) - Most mature EU region
- eu-central-1 (Frankfurt) - Good for GDPR compliance

### **5.2 Verify Region Settings**

Check your current region:

- Top right of AWS Console shows current region
- Change if needed for consistency
- Remember: Some services are global (Route 53, CloudFront, IAM)

---

## ✅ **Step 6: Validation Checklist**

Before proceeding to Part 2, verify you have:

**Account Security:**

- Root account has MFA enabled
- Personal admin user created with MFA
- Never using root account for daily work
- Strong, unique passwords set

**Cost Management:**

- Billing alerts enabled
- Monthly budget created with alerts
- Free tier tracking enabled

**Development Environment:**

- AWS CLI installed and configured
- Can run aws sts get-caller-identity successfully
- Access keys stored securely

**Region Planning:**

- Chosen primary region (recommend us-east-1)
- Understand which services are global vs regional

---

## 🎯 **What's Next**

In **Part 2**, we'll cover:

- Purchasing and configuring your custom domain through Route 53
- Setting up SSL certificates with AWS Certificate Manager
- Configuring DNS for both your main domain and API subdomain
- Domain security best practices

---

## 💡 **Cost Optimization Tips for This Setup**

**Expected Monthly Costs (Learning Environment):**

- Domain registration: 12-15 USD per year
- Route 53 hosted zone: 0.50 USD per month
- S3 storage: Under 1 USD per month
- CloudFront: Usually free tier covers learning usage
- ECS Fargate: 10-20 USD per month (depending on usage)
- ElastiCache: 10-15 USD per month
- MongoDB Atlas: Free tier available

**Total estimated: 25-40 USD per month for learning**

**Money-Saving Tips:**

- Use free tier MongoDB Atlas instead of DocumentDB
- Stop ECS tasks when not actively learning
- Use smaller instance sizes for learning
- Set up automatic resource cleanup
- Monitor usage through Cost Explorer weekly

---

## 🚨 **Important Security Reminders**

**Never Do:**

- Share root account credentials
- Commit AWS keys to GitHub
- Use overly permissive IAM policies
- Leave resources running when not needed

**Always Do:**

- Use MFA on all accounts
- Rotate access keys regularly
- Use IAM roles for applications
- Tag all resources for cost tracking
- Review permissions regularly

---

---

# 🌐 **Part 2: Domain & DNS Setup**

## 🎯 **What We'll Cover in This Section**

Setting up your custom domain, SSL certificates, and DNS configuration for both frontend and backend services.

---

## 📋 **Step 1: Domain Registration via Route 53**

### **1.1 Navigate to Route 53**

Access Route 53 Service:

- Open AWS Console
- Search "Route 53" in the search bar
- Click "Route 53" from results
- You'll see the Route 53 dashboard

### **1.2 Search for Available Domains**

Register a New Domain:

- Click "Register domain" on Route 53 dashboard
- Or click "Registered domains" → "Register domain"

Choose Your Domain:

- Enter your desired domain name (e.g., "myawesomeapp")
- Click "Check availability"
- Browse through available extensions (.com, .net, .org, .io, etc.)
- Recommendation: .com for maximum compatibility, .io for tech projects

**Domain Naming Best Practices:**

- Keep it short and memorable
- Avoid hyphens and numbers if possible
- Consider future branding needs
- Check trademark issues

### **1.3 Configure Domain Registration**

Select Domain and Add to Cart:

- Choose your preferred domain
- Click "Add to cart"
- Review pricing (typically 12-15 USD per year)

Auto-Renewal Settings:

- Enable auto-renewal (recommended)
- This prevents accidental domain expiration

Domain Privacy Protection:

- Keep enabled (free with Route 53)
- Protects your personal information from WHOIS lookups

### **1.4 Complete Registration**

Contact Information:

- Enter accurate contact details
- Use real email address (verification required)
- Choose "Same as registrant contact" for admin/tech contacts

Terms and Conditions:

- Read and accept terms
- Click "Complete purchase"

**Registration Timeline:**

- Payment processing: Immediate
- Domain activation: 10-15 minutes
- Email verification: Check your email and click verification link
- Full propagation: Up to 24 hours (usually much faster)

---

## 🔒 **Step 2: SSL Certificate Setup**

### **2.1 Navigate to Certificate Manager**

Access AWS Certificate Manager:

- Search "Certificate Manager" in AWS Console
- Click "AWS Certificate Manager"
- Ensure you're in the correct region (us-east-1 for CloudFront)

**Important:** For CloudFront distributions, certificates must be in us-east-1 region, regardless of where your other resources are located.

### **2.2 Request Public Certificate**

Request New Certificate:

- Click "Request a certificate"
- Select "Request a public certificate"
- Click "Next"

### **2.3 Configure Certificate Domains**

Add Domain Names:

- Domain name 1: yourdomain.com (replace with your actual domain)
- Click "Add another name to this certificate"
- Domain name 2: www.yourdomain.com
- Click "Add another name to this certificate"
- Domain name 3: api.yourdomain.com
- Click "Add another name to this certificate"
- Domain name 4: \*.yourdomain.com (wildcard for future subdomains)

**Why These Domains:**

- yourdomain.com: Main domain
- www.yourdomain.com: Standard www subdomain
- api.yourdomain.com: Backend API subdomain
- \*.yourdomain.com: Wildcard for any future subdomains

### **2.4 Choose Validation Method**

DNS Validation (Recommended):

- Select "DNS validation"
- More secure than email validation
- Automatically renewable
- Works well with Route 53

Email Validation (Alternative):

- Only if you don't have DNS access
- Requires manual renewal
- Less secure

### **2.5 Add Tags**

Resource Tags:

- Project: FullStackApp
- Environment: Learning
- Owner: YourName
- CreatedDate: Today's date

### **2.6 Complete Certificate Request**

Review and Request:

- Review all domain names
- Click "Request"
- Certificate status will show "Pending validation"

### **2.7 Validate Certificate**

For DNS Validation with Route 53:

- Click on your certificate ARN
- Expand each domain name
- Click "Create record in Route 53" for each domain
- Confirm creation in popup
- Repeat for all domains

**Validation Timeline:**

- Route 53 record creation: Immediate
- Certificate validation: 5-30 minutes
- Status change to "Issued": Usually within 30 minutes

---

## 🗺️ **Step 3: DNS Hosted Zone Configuration**

### **3.1 Verify Hosted Zone Creation**

Check Hosted Zone:

- Go back to Route 53 dashboard
- Click "Hosted zones"
- Your domain should appear automatically after registration
- Click on your domain name to view records

Default Records Created:

- NS (Name Server) record: Points to AWS name servers
- SOA (Start of Authority) record: Contains zone information

### **3.2 Understand DNS Record Types**

**Record Types You'll Use:**

- A Record: Points domain to IPv4 address
- AAAA Record: Points domain to IPv6 address
- CNAME Record: Points subdomain to another domain
- ALIAS Record: AWS-specific, points to AWS resources

### **3.3 Plan Your DNS Structure**

**Your Final DNS Setup Will Be:**

- yourdomain.com → S3/CloudFront (Angular app)
- www.yourdomain.com → Redirect to yourdomain.com
- api.yourdomain.com → Application Load Balancer (Node.js API)

---

## 🔧 **Step 4: Initial DNS Records Setup**

### **4.1 Create Placeholder Records**

We'll create basic records now and update them later when we deploy services.

Create Root Domain Record (Placeholder):

- Click "Create record"
- Leave "Record name" blank (for root domain)
- Record type: A
- TTL: 300 (5 minutes for testing)
- Value: 192.0.2.1 (placeholder IP, we'll change this later)
- Click "Create record"

Create WWW Subdomain Record:

- Click "Create record"
- Record name: www
- Record type: CNAME
- TTL: 300
- Value: yourdomain.com (replace with your actual domain)
- Click "Create record"

Create API Subdomain Record (Placeholder):

- Click "Create record"
- Record name: api
- Record type: A
- TTL: 300
- Value: 192.0.2.2 (placeholder IP)
- Click "Create record"

### **4.2 Test DNS Resolution**

Using Command Line:

- Open terminal/PowerShell
- Test commands:
  - nslookup yourdomain.com
  - nslookup www.yourdomain.com
  - nslookup api.yourdomain.com

Using Online Tools:

- Visit whatsmydns.net
- Enter your domain name
- Check propagation globally
- May take up to 24 hours for full propagation

---

## 🛡️ **Step 5: Domain Security Configuration**

### **5.1 Configure DNSSEC (Optional but Recommended)**

Enable DNSSEC:

- In Route 53 hosted zone
- Click "DNSSEC signing"
- Click "Enable DNSSEC signing"
- AWS will create KSK (Key Signing Key)
- This adds cryptographic security to DNS

### **5.2 Domain Transfer Lock**

Verify Transfer Protection:

- Go to Route 53 → "Registered domains"
- Click on your domain
- Under "Domain details" → "Transfer lock"
- Ensure it's "Enabled"
- This prevents unauthorized domain transfers

### **5.3 Contact Information Privacy**

Verify Privacy Protection:

- In domain details
- Check "Privacy protection" status
- Should be "Enabled" (free with Route 53)

---

## 📊 **Step 6: DNS Health Checks (Optional)**

### **6.1 Set Up Basic Health Checks**

Create Health Check for Main Domain:

- Go to Route 53 → "Health checks"
- Click "Create health check"
- Health check name: "Main-Domain-Check"
- Monitor: "Endpoint"
- Protocol: HTTPS (once we set up CloudFront)
- Domain name: yourdomain.com
- Path: /
- Request interval: 30 seconds
- Failure threshold: 3

Create Health Check for API:

- Repeat above steps
- Health check name: "API-Domain-Check"
- Domain name: api.yourdomain.com
- Path: /health (you'll create this endpoint later)

### **6.2 Health Check Notifications**

Set Up SNS for Alerts:

- Search "SNS" in AWS Console
- Create topic: "DNS-Health-Alerts"
- Create subscription with your email
- Link health checks to this SNS topic

---

## 💰 **Step 7: Cost Management for DNS**

### **7.1 Route 53 Pricing Overview**

**Monthly Costs:**

- Hosted zone: 0.50 USD per month
- DNS queries: 0.40 USD per million queries (first 1 billion)
- Health checks: 0.50 USD per health check per month
- Domain registration: 12-15 USD per year (depends on TLD)

**For Learning Environment:**

- Expected monthly DNS costs: 1-2 USD
- Domain registration: Annual fee
- Health checks: Optional for learning

### **7.2 Monitor DNS Usage**

Track DNS Queries:

- CloudWatch → "Route 53" namespace
- Monitor query count and types
- Set up alerts if approaching limits

---

## ✅ **Step 8: Validation Checklist**

Before proceeding to Part 3, verify you have:

**Domain Registration:**

- Domain successfully registered
- Email verification completed
- Auto-renewal enabled
- Privacy protection active

**SSL Certificates:**

- Certificate requested for all needed domains
- DNS validation completed
- Certificate status shows "Issued"
- Certificate ARN copied for later use

**DNS Configuration:**

- Hosted zone exists and is active
- Placeholder DNS records created
- DNS resolution working globally
- DNSSEC enabled (optional)

**Security & Monitoring:**

- Domain transfer lock enabled
- Privacy protection active
- Health checks configured (optional)
- Cost monitoring active

---

## 🎯 **What's Next**

In **Part 3**, we'll cover:

- Setting up S3 bucket for Angular application
- Configuring CloudFront distribution with custom domain
- Deploying Angular build to S3
- Updating DNS records to point to CloudFront

---

## 🔧 **Troubleshooting Common Issues**

**Domain Not Resolving:**

- Check if domain registration is complete
- Verify email confirmation was clicked
- Wait up to 24 hours for propagation
- Use multiple DNS checking tools

**Certificate Validation Stuck:**

- Ensure DNS records were created correctly
- Check if you're in the right region (us-east-1 for CloudFront)
- Verify domain ownership
- Wait up to 30 minutes for validation

**High DNS Costs:**

- Review query patterns in CloudWatch
- Consider caching strategies
- Optimize TTL values
- Remove unnecessary health checks

**Domain Privacy Issues:**

- Verify privacy protection is enabled
- Check WHOIS data to confirm
- Contact AWS support if personal info is visible

---

## 💡 **Pro Tips for Domain Management**

**Domain Security:**

- Enable two-factor authentication on domain registrar
- Use strong, unique passwords
- Monitor domain expiration dates
- Keep contact information current

**DNS Performance:**

- Use appropriate TTL values (300s for testing, 3600s for production)
- Implement geographic routing for global apps
- Consider Route 53 resolver for VPCs
- Monitor DNS query patterns

**Cost Optimization:**

- Use longer TTL values in production
- Implement intelligent DNS routing
- Monitor unused health checks
- Consider reserved capacity for high-volume apps

**Ready for Part 3? Let me know when you want to continue with Frontend Deployment!** 🚀

---

# 📱 **Part 3: Frontend Deployment (Angular)**

## 🎯 **What We'll Cover in This Section**

Deploying your Angular application using S3 for hosting and CloudFront for global content delivery with custom domain integration.

---

## 📋 **Step 1: S3 Bucket Creation & Configuration**

### **1.1 Navigate to S3 Service**

Access S3 Service:

- Open AWS Console
- Search "S3" in the search bar
- Click "Amazon S3" from results
- You'll see the S3 dashboard with any existing buckets

### **1.2 Create S3 Bucket for Angular App**

Create New Bucket:

- Click "Create bucket"
- Bucket name: yourdomainname-frontend (e.g., "myawesomeapp-frontend")
- AWS Region: us-east-1 (same as your certificate region)

**Bucket Naming Rules:**

- Must be globally unique across all AWS accounts
- 3-63 characters long
- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- Cannot have consecutive periods

### **1.3 Configure Bucket Settings**

Object Ownership:

- Select "ACLs enabled"
- Choose "Bucket owner preferred"

Block Public Access Settings:

- Uncheck "Block all public access"
- Uncheck all four individual settings
- Check the acknowledgment box: "I acknowledge that the current settings might result in this bucket and the objects within becoming public"

Bucket Versioning:

- Choose "Enable" (recommended for deployment rollbacks)
- This allows you to keep multiple versions of files

Default Encryption:

- Choose "Enable"
- Encryption type: "Amazon S3 managed keys (SSE-S3)"
- This encrypts all objects automatically

Advanced Settings:

- Object Lock: Leave "Disable"
- Tags: Add your standard tags:
  - Project: FullStackApp
  - Environment: Learning
  - Owner: YourName
  - CreatedDate: Today's date

### **1.4 Complete Bucket Creation**

Review and Create:

- Review all settings
- Click "Create bucket"
- Bucket will be created in a few seconds

### **1.5 Configure Bucket for Website Hosting**

Enable Static Website Hosting:

- Click on your newly created bucket
- Go to "Properties" tab
- Scroll down to "Static website hosting"
- Click "Edit"

Static Website Hosting Settings:

- Select "Enable"
- Hosting type: "Host a static website"
- Index document: index.html
- Error document: index.html (for SPA routing)
- Click "Save changes"

**Note the Bucket Website Endpoint:**

- After enabling, you'll see a "Bucket website endpoint"
- Copy this URL for testing purposes
- Format: http://yourdomainname-frontend.s3-website-us-east-1.amazonaws.com

---

## 🔒 **Step 2: S3 Bucket Policy Configuration**

### **2.1 Create Bucket Policy for Public Read Access**

Navigate to Bucket Permissions:

- In your S3 bucket, click "Permissions" tab
- Scroll down to "Bucket policy"
- Click "Edit"

### **2.2 Add Public Read Policy**

Enter Bucket Policy:
Replace "yourdomainname-frontend" with your actual bucket name:

````

{
"Version": "2012-10-17",
"Statement": [
{
"Sid": "PublicReadGetObject",
"Effect": "Allow",
"Principal": "*",
"Action": "s3:GetObject",
"Resource": "arn:aws:s3:::yourdomainname-frontend/*"
}
]
}

```

Save Policy:

- Click "Save changes"
- The bucket policy will allow public read access to all objects

### **2.3 Verify Public Access**

Check Bucket Status:

- Go back to S3 buckets list
- Your bucket should show "Publicly accessible" in the "Access" column
- This indicates the policy is working correctly

---

## 🌐 **Step 3: CloudFront Distribution Setup**

### **3.1 Navigate to CloudFront**

Access CloudFront Service:

- Search "CloudFront" in AWS Console
- Click "Amazon CloudFront"
- You'll see the CloudFront dashboard

### **3.2 Create CloudFront Distribution**

Create Distribution:

- Click "Create distribution"
- Choose "Web" distribution type

### **3.3 Configure Origin Settings**

Origin Domain:

- Click in the "Origin domain" field
- Select your S3 bucket from the dropdown
- Choose the bucket website endpoint version (not the regular bucket)
- Example: yourdomainname-frontend.s3-website-us-east-1.amazonaws.com

Origin Path:

- Leave blank (serves content from root)

Origin ID:

- Auto-generated, leave as default
- Example: yourdomainname-frontend.s3-website-us-east-1.amazonaws.com

### **3.4 Configure Default Cache Behavior**

Viewer Protocol Policy:

- Select "Redirect HTTP to HTTPS"
- This forces secure connections

Allowed HTTP Methods:

- Choose "GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE"
- Allows all HTTP methods for SPA functionality

Cache Policy:

- Select "CachingOptimized"
- Good default for static websites

Origin Request Policy:

- Leave as "None"

### **3.5 Configure Distribution Settings**

Alternate Domain Names (CNAMES):

- Add your domain: yourdomain.com
- Add www subdomain: www.yourdomain.com

Custom SSL Certificate:

- Select "Custom SSL Certificate"
- Choose your certificate from the dropdown
- Should show your previously created certificate

Default Root Object:

- Enter: index.html
- This serves index.html when accessing root domain

### **3.6 Configure Error Pages for SPA**

Since Angular is a Single Page Application, configure custom error pages:

- Standard caching behavior is set above
- We'll configure custom error pages after creation

### **3.7 Additional Distribution Settings**

Supported HTTP Versions:

- Select "HTTP/2 and HTTP/3"

IPv6:

- Keep "On" (enabled)

Description:

- Enter: "Angular Frontend Distribution for FullStackApp"

Distribution State:

- Keep "Enabled"

Tags:

- Add your standard tags:
  - Project: FullStackApp
  - Environment: Learning
  - Owner: YourName

### **3.8 Create Distribution**

Review and Create:

- Review all settings
- Click "Create distribution"
- Distribution creation takes 5-15 minutes

**Copy Distribution Information:**

- Distribution ID: Copy for future reference
- Distribution domain name: Copy this CloudFront URL
- Status will show "Deploying" initially

---

## 📄 **Step 4: Configure Custom Error Pages**

### **4.1 Wait for Distribution Deployment**

Check Deployment Status:

- Refresh the CloudFront distributions page
- Wait until "Status" changes from "Deploying" to "Deployed"
- "Last modified" will show recent timestamp
- This usually takes 5-15 minutes

### **4.2 Add Custom Error Pages**

Navigate to Error Pages:

- Click on your distribution ID
- Go to "Error pages" tab
- Click "Create custom error response"

Configure 403 Error Page:

- HTTP Error Code: 403 Forbidden
- Customize error response: Yes
- Response Page Path: /index.html
- HTTP Response Code: 200 OK
- Click "Create custom error response"

Configure 404 Error Page:

- Click "Create custom error response" again
- HTTP Error Code: 404 Not Found
- Customize error response: Yes
- Response Page Path: /index.html
- HTTP Response Code: 200 OK
- Click "Create custom error response"

**Why These Settings:**

- Angular router handles all routing client-side
- Any unknown route should serve index.html
- Angular app then handles the routing internally

---

## 🔧 **Step 5: Update DNS Records**

### **5.1 Navigate Back to Route 53**

Access Route 53:

- Go to Route 53 service
- Click "Hosted zones"
- Click on your domain name

### **5.2 Update Root Domain Record**

Edit A Record:

- Find your root domain A record (currently pointing to 192.0.2.1)
- Click "Edit"
- Change Record type to "A - Routes traffic to IPv4 address and some AWS resources"
- Enable "Alias" toggle
- Route traffic to: "Alias to CloudFront distribution"
- Choose your CloudFront distribution from dropdown
- TTL: 300 (5 minutes for testing)
- Click "Save"

### **5.3 Update WWW Subdomain Record**

Edit CNAME Record:

- Find your www CNAME record
- Click "Edit"
- Change to Alias record instead
- Enable "Alias" toggle
- Route traffic to: "Alias to CloudFront distribution"
- Choose your CloudFront distribution
- Click "Save"

### **5.4 Verify DNS Changes**

Test DNS Resolution:

- Open terminal/PowerShell
- Run: nslookup yourdomain.com
- Should return CloudFront IP addresses
- Run: nslookup www.yourdomain.com
- Should also return CloudFront IP addresses

Online Verification:

- Visit whatsmydns.net
- Check your domain globally
- Should show CloudFront IPs worldwide

---

## 📱 **Step 6: Prepare Angular Application for Deployment**

### **6.1 Angular Build Configuration**

Open Your Angular Project:

- Navigate to your Angular project directory
- Ensure package.json contains build scripts

Check Angular CLI Version:

- Run: ng version
- Ensure you have Angular CLI installed
- If not: npm install -g @angular/cli

### **6.2 Configure Production Build**

Update angular.json for Production:

- Open angular.json file
- Under "build" → "configurations" → "production"
- Verify these settings exist:
  - "optimization": true
  - "outputHashing": "all"
  - "sourceMap": false
  - "namedChunks": false
  - "extractLicenses": true
  - "vendorChunk": false
  - "buildOptimizer": true

### **6.3 Add Base Href Configuration**

For Proper Routing:

- In your build command, ensure base href is set correctly
- Build command: ng build --configuration=production --base-href="/"

### **6.4 Build Your Angular Application**

Run Production Build:

- Open terminal in your Angular project
- Run: ng build --configuration=production --base-href="/"
- Build output will be in 'dist/your-project-name' folder
- Verify index.html and asset files are generated

**Build Output Verification:**

- Check dist folder contains:
  - index.html (main file)
  - CSS files with hash names
  - JavaScript files with hash names
  - Assets folder (if any)

---

## ⬆️ **Step 7: Deploy Angular Build to S3**

### **7.1 Using AWS CLI for Deployment**

Navigate to Build Directory:

- Open terminal/PowerShell
- Navigate to your Angular build output
- cd path/to/your-project/dist/your-project-name

Upload Files to S3:

- Run: aws s3 sync . s3://yourdomainname-frontend --delete
- The --delete flag removes old files
- All files will upload to your S3 bucket

**Alternative: Manual Upload via Console**

If you prefer using AWS Console:

- Go to S3 → Your bucket
- Click "Upload"
- Drag all files from dist folder
- Click "Upload"

### **7.2 Set Content Types (Important)**

For proper file serving, ensure correct content types:

Upload with Content Types:

- HTML files: text/html
- CSS files: text/css
- JS files: application/javascript
- Images: image/png, image/jpeg, etc.

AWS CLI with Content Types:

```

aws s3 sync . s3://yourdomainname-frontend --delete --content-type-by-extension

```

### **7.3 Verify Upload**

Check S3 Bucket:

- Go to S3 → Your bucket
- Verify all files are uploaded
- Check file sizes match your local build
- Verify index.html is present at root level

Test S3 Website Endpoint:

- Copy the S3 website endpoint URL
- Open in browser
- Should see your Angular application
- Test routing by navigating to different pages

---

## 🧪 **Step 8: Test Your Deployed Application**

### **8.1 Test via CloudFront**

Test CloudFront Distribution:

- Copy your CloudFront distribution domain name
- Open in browser: https://your-cloudfront-domain.cloudfront.net
- Should see your Angular app with HTTPS

### **8.2 Test via Custom Domain**

Test Custom Domain:

- Open browser and go to: https://yourdomain.com
- Should see your Angular application
- Test: https://www.yourdomain.com
- Should also work

### **8.3 Test SPA Routing**

Test Angular Routes:

- Navigate to different routes in your app
- Refresh the page on a specific route
- Should still work (not show 404)
- This confirms custom error pages are working

### **8.4 Test Performance**

Check Loading Speed:

- Open browser developer tools
- Go to Network tab
- Refresh your site
- Check load times for assets
- Should be fast due to CloudFront caching

Test from Different Locations:

- Use tools like GTmetrix or PageSpeed Insights
- Test from various geographic locations
- Should show good performance globally

---

## 🔧 **Step 9: Performance Optimization**

### **9.1 CloudFront Cache Optimization**

Update Cache Behaviors (Optional):

- Go to CloudFront → Your distribution
- Click "Behaviors" tab
- Edit default behavior if needed
- Adjust TTL values for better performance

### **9.2 Gzip Compression**

Verify Compression:

- CloudFront automatically compresses files
- Check response headers for "Content-Encoding: gzip"
- Should be enabled by default

### **9.3 Enable CloudFront Caching**

Cache Settings Verification:

- Static assets (CSS, JS, images) should cache for longer
- HTML files should have shorter cache times
- Default CloudFront settings are usually good for Angular

---

## ✅ **Step 10: Validation Checklist**

Before proceeding to Part 4, verify you have:

**S3 Configuration:**

- S3 bucket created and configured for static hosting
- Bucket policy allows public read access
- All Angular build files uploaded successfully
- Website endpoint accessible

**CloudFront Setup:**

- Distribution created and deployed
- Custom domain configured with SSL certificate
- Custom error pages configured for SPA routing
- Distribution accessible via CloudFront domain

**DNS Configuration:**

- Root domain points to CloudFront distribution
- WWW subdomain points to CloudFront distribution
- DNS changes propagated globally
- Both HTTP and HTTPS work correctly

**Application Testing:**

- Angular app loads correctly via custom domain
- SPA routing works when refreshing pages
- Performance is good globally
- SSL certificate is valid and working

---

## 🎯 **What's Next**

In **Part 4**, we'll cover:

- Setting up VPC and Security Groups for backend
- Creating ECS Cluster for Node.js application
- Configuring Application Load Balancer
- Setting up auto-scaling and health checks

---

## 💰 **Cost Breakdown for Frontend**

**S3 Costs (Monthly):**

- Storage: ~0.10 USD for typical Angular app (few MB)
- GET requests: ~0.01 USD for moderate traffic
- PUT requests: Minimal during deployment

**CloudFront Costs:**

- Data transfer: First 1TB free (covers most learning)
- Requests: First 10 million free
- SSL certificate: Free with ACM

**Route 53 Costs:**

- Already covered in Part 2
- No additional costs for frontend

**Total Monthly Cost for Frontend: ~0.15-0.50 USD**

---

## 🔧 **Troubleshooting Common Issues**

**Angular App Shows Blank Page:**

- Check browser console for JavaScript errors
- Verify base href is set correctly in index.html
- Ensure all files uploaded to S3 correctly
- Check S3 bucket policy allows public access

**404 Errors on Route Refresh:**

- Verify custom error pages are configured
- Check that error pages point to /index.html
- Ensure error response code is 200, not 404

**SSL Certificate Issues:**

- Verify certificate is in us-east-1 region
- Check that all domain names are included in certificate
- Ensure DNS validation is complete

**Slow Loading Times:**

- Check CloudFront cache hit ratio
- Verify gzip compression is enabled
- Review CloudFront distribution settings
- Consider optimizing Angular bundle size

**Domain Not Resolving:**

- Check DNS record configuration
- Verify CloudFront distribution is deployed
- Use multiple DNS checking tools
- Wait for DNS propagation (up to 24 hours)

---

## 💡 **Pro Tips for Frontend Deployment**

**Performance Optimization:**

- Enable Angular's production optimizations
- Use lazy loading for feature modules
- Implement service worker for caching
- Optimize images and assets

**Deployment Best Practices:**

- Use environment-specific configurations
- Implement proper error handling
- Set up monitoring and analytics
- Plan for rollback scenarios

**Security Considerations:**

- Never commit sensitive data to frontend
- Use environment variables for API endpoints
- Implement Content Security Policy headers
- Regular dependency updates

**Cost Optimization:**

- Monitor CloudFront usage in CloudWatch
- Use appropriate cache TTL values
- Consider S3 storage classes for backups
- Remove old deployment artifacts

**Ready for Part 4? Let me know when you want to continue with Backend Infrastructure!** 🚀

---

# 🏗️ **Part 4: Backend Infrastructure Setup**

## 🎯 **What We'll Cover in This Section**

Setting up secure, scalable backend infrastructure using VPC, ECS Fargate, and Application Load Balancer for your Node.js API with MongoDB Atlas integration.

---

## 🌐 **Step 1: VPC (Virtual Private Cloud) Setup**

### **1.1 Navigate to VPC Service**

Access VPC Service:

- Open AWS Console
- Search "VPC" in the search bar
- Click "VPC" from results
- You'll see the VPC dashboard

### **1.2 Create VPC Using VPC Wizard**

Use VPC and More Option:

- Click "Create VPC"
- Select "VPC and more" (recommended for beginners)
- This creates VPC with all necessary components

### **1.3 Configure VPC Settings**

VPC Configuration:

- Name tag auto-generation: fullstackapp-vpc
- IPv4 CIDR block: 10.0.0.0/16 (provides 65,536 IP addresses)
- IPv6 CIDR block: No IPv6 CIDR block
- Tenancy: Default

Availability Zones:

- Number of Availability Zones: 2
- Choose: us-east-1a and us-east-1b
- This provides high availability and fault tolerance

Subnets Configuration:

- Number of public subnets: 2
- Number of private subnets: 2
- Public subnet CIDR blocks:
  - Public subnet 1: 10.0.1.0/24 (AZ: us-east-1a)
  - Public subnet 2: 10.0.2.0/24 (AZ: us-east-1b)
- Private subnet CIDR blocks:
  - Private subnet 1: 10.0.11.0/24 (AZ: us-east-1a)
  - Private subnet 2: 10.0.12.0/24 (AZ: us-east-1b)

NAT Gateways:

- NAT gateways: 1 per AZ
- This allows private subnets to access internet for updates

VPC Endpoints:

- S3 Gateway: None (not needed for this setup)
- DynamoDB Gateway: None

DNS Options:

- Enable DNS hostnames: Checked
- Enable DNS resolution: Checked

### **1.4 Add Resource Tags**

Tags for All Resources:

- Project: FullStackApp
- Environment: Learning
- Owner: YourName
- CreatedDate: Today's date

### **1.5 Create VPC**

Review and Create:

- Review all settings
- Click "Create VPC"
- Creation takes 2-3 minutes
- Wait for "Successfully created" message

**What Gets Created:**

- VPC with specified CIDR block
- Internet Gateway
- 2 Public subnets with route tables
- 2 Private subnets with route tables
- 2 NAT Gateways (one in each AZ)
- Security groups

---

## 🔒 **Step 2: Security Groups Configuration**

### **2.1 Navigate to Security Groups**

Access Security Groups:

- In VPC dashboard, click "Security Groups" in left sidebar
- You'll see default security group created with VPC
- We'll create custom security groups for different components

### **2.2 Create ALB Security Group**

Create Load Balancer Security Group:

- Click "Create security group"
- Security group name: fullstackapp-alb-sg
- Description: Security group for Application Load Balancer
- VPC: Select your fullstackapp-vpc

Inbound Rules for ALB:

- Rule 1:
  - Type: HTTP
  - Protocol: TCP
  - Port range: 80
  - Source: 0.0.0.0/0 (Anywhere IPv4)
  - Description: HTTP from anywhere
- Rule 2:
  - Type: HTTPS
  - Protocol: TCP
  - Port range: 443
  - Source: 0.0.0.0/0 (Anywhere IPv4)
  - Description: HTTPS from anywhere

Outbound Rules for ALB:

- Keep default (All traffic to 0.0.0.0/0)

Tags:

- Add your standard tags
- Click "Create security group"

### **2.3 Create ECS Security Group**

Create ECS Tasks Security Group:

- Click "Create security group"
- Security group name: fullstackapp-ecs-sg
- Description: Security group for ECS tasks
- VPC: Select your fullstackapp-vpc

Inbound Rules for ECS:

- Rule 1:
  - Type: Custom TCP
  - Protocol: TCP
  - Port range: 3000 (or your Node.js app port)
  - Source: Select "Custom" and choose fullstackapp-alb-sg
  - Description: HTTP from ALB only

Outbound Rules for ECS:

- Keep default (All traffic to 0.0.0.0/0)
- This allows ECS tasks to connect to MongoDB Atlas and internet

Tags:

- Add your standard tags
- Click "Create security group"

### **2.4 Create Database Security Group (Future Use)**

Create Database Security Group:

- Click "Create security group"
- Security group name: fullstackapp-db-sg
- Description: Security group for database connections
- VPC: Select your fullstackapp-vpc

Inbound Rules for Database:

- Rule 1:
  - Type: Custom TCP
  - Protocol: TCP
  - Port range: 27017 (MongoDB port)
  - Source: Select "Custom" and choose fullstackapp-ecs-sg
  - Description: MongoDB from ECS tasks only

Outbound Rules:

- Keep default

Tags:

- Add your standard tags
- Click "Create security group"

**Note:** This security group will be used if you later migrate from MongoDB Atlas to a self-hosted MongoDB in AWS.

---

## 🔧 **Step 3: Application Load Balancer Setup**

### **3.1 Navigate to Load Balancers**

Access Load Balancer Service:

- Search "EC2" in AWS Console
- In EC2 dashboard, scroll down to "Load Balancing"
- Click "Load Balancers"

### **3.2 Create Application Load Balancer**

Create Load Balancer:

- Click "Create Load Balancer"
- Choose "Application Load Balancer"
- Click "Create"

### **3.3 Basic Configuration**

Load Balancer Settings:

- Load balancer name: fullstackapp-alb
- Scheme: Internet-facing
- IP address type: IPv4

### **3.4 Network Mapping**

VPC Configuration:

- VPC: Select your fullstackapp-vpc
- Mappings: Check both availability zones
  - us-east-1a: Select public subnet (10.0.1.0/24)
  - us-east-1b: Select public subnet (10.0.2.0/24)

### **3.5 Security Groups**

Security Group Assignment:

- Remove default security group
- Select: fullstackapp-alb-sg (created earlier)

### **3.6 Listeners and Routing**

Default Listener Configuration:

- Protocol: HTTP
- Port: 80
- Default action: We'll configure this after creating target group

Create Target Group (Temporary):

- Click "Create target group" link
- Opens in new tab

**Target Group Configuration:**

- Target type: IP addresses (for Fargate)
- Target group name: fullstackapp-tg
- Protocol: HTTP
- Port: 3000
- VPC: Select your fullstackapp-vpc
- Protocol version: HTTP1

Health Check Settings:

- Health check protocol: HTTP
- Health check path: /health
- Health check port: Traffic port
- Healthy threshold: 2
- Unhealthy threshold: 2
- Timeout: 5 seconds
- Interval: 30 seconds
- Success codes: 200

Advanced Health Check Settings:

- Healthy threshold: 2 consecutive health checks
- Unhealthy threshold: 2 consecutive health check failures

Tags:

- Add your standard tags
- Click "Create target group"

Return to Load Balancer Configuration:

- Refresh the target group dropdown
- Select: fullstackapp-tg
- Click "Create load balancer"

### **3.7 Configure HTTPS Listener**

After ALB Creation:

- Click on your load balancer name
- Go to "Listeners" tab
- Click "Add listener"

HTTPS Listener Configuration:

- Protocol: HTTPS
- Port: 443
- Default action: Forward to fullstackapp-tg
- Security policy: ELBSecurityPolicy-TLS13-1-2-2021-06 (latest)
- Certificate: Choose your certificate from ACM

HTTP to HTTPS Redirect:

- Edit the HTTP:80 listener
- Change default action to "Redirect"
- Protocol: HTTPS
- Port: 443
- Status code: 301 (Permanent redirect)

---

## 📦 **Step 4: ECS Cluster Setup**

### **4.1 Navigate to ECS Service**

Access ECS Service:

- Search "ECS" in AWS Console
- Click "Elastic Container Service"
- You'll see the ECS console

### **4.2 Create ECS Cluster**

Create Cluster:

- Click "Create cluster"
- Cluster name: fullstackapp-cluster
- Infrastructure: AWS Fargate (serverless)

Monitoring Settings:

- Use Container Insights: Enable
- This provides detailed monitoring and logging

Tags:

- Add your standard tags
- Click "Create"

### **4.3 Create Task Definition**

Navigate to Task Definitions:

- In ECS console, click "Task definitions" in left sidebar
- Click "Create new task definition"

### **4.4 Configure Task Definition**

Task Definition Family:

- Family name: fullstackapp-backend-task
- Launch type: AWS Fargate
- Operating system: Linux/X86_64
- CPU architecture: X86_64

Task Size:

- CPU: 0.25 vCPU (256 CPU units)
- Memory: 0.5 GB (512 MB)
- These are minimal settings for learning/testing

Task Role and Execution Role:

- Task execution role: Create new role (ecsTaskExecutionRole)
- Task role: None (for now)

### **4.5 Container Configuration**

Container Details:

- Container name: nodejs-backend
- Image URI: We'll use a placeholder for now: nginx:latest
- Port mappings:
  - Container port: 3000
  - Protocol: TCP
  - Port name: nodejs-port

**Note:** We'll update this image URI later when we build and push your Node.js application.

Environment Variables:

- Add environment variables for your app:
  - NODE_ENV: production
  - PORT: 3000
  - MONGODB_URI: (we'll add this in Part 5)

Logging Configuration:

- Log driver: awslogs
- Log group: Create new: /ecs/fullstackapp-backend-task
- Log region: us-east-1
- Log stream prefix: ecs

### **4.6 Create Task Definition**

Review and Create:

- Review all settings
- Click "Create"
- Task definition will be created

---

## 🚀 **Step 5: ECS Service Configuration**

### **5.1 Create ECS Service**

Navigate to Cluster:

- Go back to ECS clusters
- Click on fullstackapp-cluster
- Go to "Services" tab
- Click "Create"

### **5.2 Service Configuration**

Launch Type Configuration:

- Launch type: Fargate
- Platform version: LATEST

Service Settings:

- Service name: fullstackapp-backend-service
- Task definition: fullstackapp-backend-task:1
- Revision: LATEST
- Service type: REPLICA
- Number of tasks: 2 (for high availability)

### **5.3 Network Configuration**

VPC and Security:

- Cluster VPC: fullstackapp-vpc
- Subnets: Select both private subnets
  - Private subnet 1: 10.0.11.0/24
  - Private subnet 2: 10.0.12.0/24
- Security groups: fullstackapp-ecs-sg
- Auto-assign public IP: DISABLED (running in private subnets)

### **5.4 Load Balancer Integration**

Load Balancer Configuration:

- Load balancer type: Application Load Balancer
- Load balancer: fullstackapp-alb
- Container to load balance: nodejs-backend 3000:3000
- Target group: fullstackapp-tg

Health Check Grace Period:

- Set to 300 seconds
- Allows time for application startup

### **5.5 Auto Scaling Configuration**

Service Auto Scaling:

- Enable service auto scaling: Yes
- Minimum number of tasks: 1
- Maximum number of tasks: 10
- Desired number of tasks: 2

Scaling Policies:

- Target tracking scaling policy:
  - Policy name: TargetTrackingScalingPolicy
  - ECS service metric: Average CPU utilization
  - Target value: 70%
  - Scale out cooldown: 300 seconds
  - Scale in cooldown: 300 seconds

### **5.6 Create Service**

Review and Create:

- Review all configurations
- Click "Create service"
- Service creation takes 2-3 minutes

---

## 🔗 **Step 6: Update DNS for Backend API**

### **6.1 Get Load Balancer DNS Name**

Find ALB DNS Name:

- Go to EC2 → Load Balancers
- Click on fullstackapp-alb
- Copy the "DNS name"
- Example: fullstackapp-alb-123456789.us-east-1.elb.amazonaws.com

### **6.2 Update Route 53 Records**

Navigate to Route 53:

- Go to Route 53 → Hosted zones
- Click on your domain

Update API Subdomain:

- Find your api.yourdomain.com A record (currently 192.0.2.2)
- Click "Edit"
- Change to:
  - Record type: A
  - Enable "Alias" toggle
  - Route traffic to: "Alias to Application and Classic Load Balancer"
  - Choose region: us-east-1
  - Choose load balancer: fullstackapp-alb
- Click "Save"

### **6.3 Add HTTPS Listener for Custom Domain**

Update ALB HTTPS Listener:

- Go to EC2 → Load Balancers → fullstackapp-alb
- Click "Listeners" tab
- Edit HTTPS:443 listener
- Ensure certificate includes api.yourdomain.com
- If not, add api.yourdomain.com to your existing certificate

---

## 🧪 **Step 7: Test Backend Infrastructure**

### **7.1 Verify ECS Service Health**

Check Service Status:

- Go to ECS → Clusters → fullstackapp-cluster
- Click "Services" tab
- Service should show "Running" status
- Check "Health and metrics" tab for task health

Check Task Health:

- Go to "Tasks" tab
- Both tasks should show "Running" status
- If not, check "Logs" tab for errors

### **7.2 Test Load Balancer**

Test ALB Direct Access:

- Copy your ALB DNS name
- Open browser: http://your-alb-dns-name
- Should see nginx welcome page (placeholder)

Test HTTPS:

- Try: https://your-alb-dns-name
- Should work with SSL

### **7.3 Test API Subdomain**

Test Custom Domain:

- Open browser: https://api.yourdomain.com
- Should see nginx welcome page
- SSL should be valid
- This confirms DNS and ALB integration working

### **7.4 Verify Health Checks**

Check Target Group Health:

- Go to EC2 → Target Groups → fullstackapp-tg
- Click "Targets" tab
- Both targets should show "healthy" status
- If "unhealthy," check /health endpoint availability

---

## 📊 **Step 8: CloudWatch Monitoring Setup**

### **8.1 Verify Container Insights**

Access Container Insights:

- Go to CloudWatch
- Click "Container Insights" in left sidebar
- Select "ECS Clusters"
- You should see fullstackapp-cluster
- View CPU, Memory, and Network metrics

### **8.2 Create Custom Alarms**

CPU Utilization Alarm:

- Go to CloudWatch → Alarms
- Click "Create alarm"
- Select metric: ECS Service CPU Utilization
- Service: fullstackapp-backend-service
- Statistic: Average
- Period: 5 minutes
- Threshold: Greater than 80%
- Actions: Send SNS notification

Memory Utilization Alarm:

- Repeat above for Memory utilization
- Threshold: Greater than 80%

### **8.3 Set Up Log Groups**

Verify Log Groups:

- Go to CloudWatch → Log groups
- Should see: /ecs/fullstackapp-backend-task
- Click to view container logs
- Useful for debugging application issues

---

## 💰 **Step 9: Cost Optimization**

### **9.1 ECS Fargate Cost Optimization**

Resource Right-Sizing:

- Monitor actual CPU/Memory usage in CloudWatch
- Adjust task definition if over/under-provisioned
- Current settings (0.25 vCPU, 0.5GB) are minimal

Auto Scaling Configuration:

- Ensure minimum tasks set to 1 for cost savings
- Maximum can be higher for traffic spikes
- Consider scheduling for learning environments

### **9.2 Load Balancer Cost Optimization**

ALB Optimization:

- Application Load Balancer: ~22 USD/month base cost
- Additional charges per hour and LCU (Load Balancer Capacity Unit)
- Monitor LCU usage in CloudWatch

### **9.3 NAT Gateway Cost Optimization**

NAT Gateway Costs:

- Each NAT Gateway: ~45 USD/month
- Data processing charges apply
- Consider using single NAT Gateway for learning (reduces availability)

Alternative for Cost Savings:

- Use single NAT Gateway instead of one per AZ
- Only for learning/development, not production

### **9.4 Expected Monthly Costs**

Backend Infrastructure Costs:

- ECS Fargate (2 tasks, minimal size): 15-25 USD/month
- Application Load Balancer: 22 USD/month
- NAT Gateways (2): 90 USD/month
- Data transfer: 1-5 USD/month

Total Backend Cost: ~130-145 USD/month

**Cost Reduction Tips:**

- Use single NAT Gateway: Save 45 USD/month
- Run 1 task instead of 2: Save 8-12 USD/month
- Use smaller task sizes if adequate: Save 5-10 USD/month

---

## ✅ **Step 10: Validation Checklist**

Before proceeding to Part 5, verify you have:

**VPC Infrastructure:**

- VPC created with proper CIDR blocks
- Public and private subnets in multiple AZs
- Internet Gateway and NAT Gateways configured
- Route tables properly configured

**Security Groups:**

- ALB security group (allows HTTP/HTTPS from internet)
- ECS security group (allows traffic only from ALB)
- Database security group (allows MongoDB from ECS)

**Load Balancer:**

- Application Load Balancer created and running
- HTTPS listener configured with SSL certificate
- Target group created with health checks
- HTTP to HTTPS redirect configured

**ECS Infrastructure:**

- ECS cluster created with Fargate
- Task definition created (placeholder container)
- ECS service running with desired task count
- Service registered with load balancer
- Auto scaling policies configured

**DNS Configuration:**

- API subdomain points to load balancer
- SSL certificate works on custom domain
- Health checks passing

**Monitoring:**

- Container Insights enabled
- CloudWatch alarms configured
- Log groups created for troubleshooting

---

## 🎯 **What's Next**

In **Part 5**, we'll cover:

- MongoDB Atlas setup and integration
- ElastiCache Redis configuration
- Database security and connection management
- Environment variable configuration for your Node.js app

---

## 🔧 **Troubleshooting Common Issues**

**ECS Tasks Not Starting:**

- Check task definition resource allocation
- Verify security group allows outbound internet access
- Check CloudWatch logs for container errors
- Ensure proper IAM permissions for task execution role

**Load Balancer Health Checks Failing:**

- Verify health check path exists (/health)
- Check security group allows ALB to reach ECS tasks
- Verify container is listening on correct port
- Check health check timeout and interval settings

**DNS Not Resolving:**

- Verify Route 53 record points to correct ALB
- Check ALB listener configuration
- Ensure SSL certificate includes API subdomain
- Wait for DNS propagation (up to 30 minutes)

**High Costs:**

- Monitor NAT Gateway usage (major cost driver)
- Review ECS task sizing and count
- Check ALB usage patterns
- Consider using single AZ for learning

**SSL Certificate Issues:**

- Verify certificate includes api.yourdomain.com
- Check certificate is in us-east-1 region
- Ensure DNS validation completed
- Verify ALB listener uses correct certificate

---

## 💡 **Pro Tips for Backend Infrastructure**

**High Availability:**

- Deploy across multiple AZs
- Use health checks and auto scaling
- Implement circuit breaker patterns
- Plan for failure scenarios

**Security Best Practices:**

- Use least privilege IAM roles
- Keep ECS tasks in private subnets
- Enable VPC Flow Logs for monitoring
- Regularly update container images

**Performance Optimization:**

- Monitor ECS service metrics
- Use appropriate task sizing
- Implement caching strategies
- Configure proper health check intervals

**Cost Management:**

- Right-size ECS tasks based on actual usage
- Use Spot instances for development
- Monitor and set up billing alerts
- Consider reserved capacity for production

**Ready for Part 5? Let me know when you want to continue with Database & Caching!** 🚀

---

# 🗄️ **Part 5: Database & Caching Setup**

## 🎯 **What We'll Cover in This Section**

Setting up MongoDB Atlas for your primary database and ElastiCache Redis for caching, with secure connections from your ECS Fargate containers.

---

## 🍃 **Step 1: MongoDB Atlas Setup**

### **1.1 Create MongoDB Atlas Account**

Sign Up for MongoDB Atlas:

- Go to cloud.mongodb.com
- Click "Try Free"
- Create account with your email
- Verify email address
- Complete profile setup

### **1.2 Create New Project**

Create Project in Atlas:

- Click "New Project"
- Project name: "FullStackApp"
- Click "Next"
- Add members: Skip for now
- Click "Create Project"

### **1.3 Create Database Cluster**

Create Cluster:

- Click "Create a cluster"
- Choose "M0 Sandbox" (Free tier)
- Cloud Provider: AWS
- Region: US East (N. Virginia) us-east-1
- Cluster Name: "fullstackapp-cluster"
- Click "Create cluster"

**Free Tier Limits:**

- 512 MB storage
- No backup
- No sharding
- Perfect for learning and development

### **1.4 Configure Database Security**

Create Database User:

- In Atlas dashboard, click "Database Access" in left sidebar
- Click "Add New Database User"
- Authentication Method: Password
- Username: fullstackapp-user
- Password: Generate secure password and save it
- Database User Privileges: "Read and write to any database"
- Click "Add User"

Configure Network Access:

- Click "Network Access" in left sidebar
- Click "Add IP Address"
- For learning purposes, choose "Allow access from anywhere"
- IP Address: 0.0.0.0/0
- Comment: "Development access"
- Click "Confirm"

**Production Note:** In production, you would whitelist only your AWS VPC CIDR blocks (10.0.0.0/16).

### **1.5 Get Connection String**

Get Connection Details:

- Go back to "Clusters" in Atlas
- Click "Connect" button on your cluster
- Choose "Connect your application"
- Driver: Node.js
- Version: 4.1 or later
- Copy the connection string
- Example: mongodb+srv://fullstackapp-user:password@fullstackapp-cluster.abc123.mongodb.net/?retryWrites=true&w=majority

Save Connection String:

- Replace password placeholder with actual password
- Save this string securely - you'll need it for environment variables

### **1.6 Create Initial Database and Collections**

Using MongoDB Compass (Optional):

- Download MongoDB Compass (GUI tool)
- Connect using your connection string
- Create database: "fullstackapp"
- Create collections: "users", "products", "orders" (as examples)

### **1.7 Test Connection**

Test from Local Machine:

- Install MongoDB driver: npm install mongodb
- Create simple test script to verify connection
- Ensure you can read/write data successfully

---

## 🔴 **Step 2: ElastiCache Redis Setup**

### **2.1 Navigate to ElastiCache Service**

Access ElastiCache:

- Search "ElastiCache" in AWS Console
- Click "Amazon ElastiCache"
- You'll see the ElastiCache dashboard

### **2.2 Create Redis Subnet Group**

Create Subnet Group:

- Click "Subnet groups" in left sidebar
- Click "Create subnet group"
- Name: fullstackapp-redis-subnet-group
- Description: Subnet group for Redis cache
- VPC: Select fullstackapp-vpc
- Availability Zones: us-east-1a, us-east-1b
- Subnets: Select both private subnets
  - Private subnet 1: 10.0.11.0/24
  - Private subnet 2: 10.0.12.0/24
- Click "Create"

### **2.3 Create Redis Parameter Group (Optional)**

Create Custom Parameter Group:

- Click "Parameter groups" in left sidebar
- Click "Create parameter group"
- Family: redis7.x
- Name: fullstackapp-redis-params
- Description: Custom parameters for Redis
- Click "Create"

Configure Parameters (Optional):

- Click on your parameter group
- Click "Edit parameters"
- Useful parameters for learning:
  - maxmemory-policy: allkeys-lru
  - timeout: 300
- Click "Save changes"

### **2.4 Create Redis Cache Cluster**

Create Cache Cluster:

- Click "Redis clusters" in left sidebar
- Click "Create Redis cluster"
- Cluster mode: Disabled (simpler for learning)
- Cluster info:
  - Name: fullstackapp-redis
  - Description: Redis cache for FullStackApp
  - Port: 6379 (default)

### **2.5 Configure Cluster Settings**

Cluster Configuration:

- Engine version: 7.0 (latest compatible)
- Node type: cache.t3.micro (smallest for learning)
- Number of replicas: 0 (for cost savings)
- Multi-AZ: Disabled (for cost savings)

Subnet and Security:

- Subnet group: fullstackapp-redis-subnet-group
- Security groups: Create new security group

### **2.6 Create Redis Security Group**

Create Security Group for Redis:

- Open new tab: Go to EC2 → Security Groups
- Click "Create security group"
- Security group name: fullstackapp-redis-sg
- Description: Security group for Redis cache
- VPC: fullstackapp-vpc

Inbound Rules:

- Type: Custom TCP
- Protocol: TCP
- Port range: 6379
- Source: fullstackapp-ecs-sg (only ECS tasks can access)
- Description: Redis access from ECS tasks

Tags:

- Add your standard tags
- Click "Create security group"

### **2.7 Complete Redis Creation**

Back to Redis Creation:

- Security groups: Select fullstackapp-redis-sg
- Encryption at rest: Enabled
- Encryption in transit: Enabled
- Auth token: Disabled (simplicity for learning)

Advanced Settings:

- Parameter group: default.redis7.x (or your custom one)
- Log configuration: Slow log disabled
- Backup: Disabled (cost savings)
- Maintenance window: No preference

Tags:

- Add your standard tags
- Click "Create"

Redis Creation Time:

- Takes 10-15 minutes to create
- Status will show "Creating" then "Available"

### **2.8 Get Redis Connection Details**

After Creation:

- Click on your Redis cluster name
- Copy the "Primary endpoint"
- Example: fullstackapp-redis.abc123.cache.amazonaws.com:6379
- Save this endpoint for environment variables

---

## 🔗 **Step 3: Update ECS Task Definition with Database Connections**

### **3.1 Create New Task Definition Revision**

Navigate to ECS:

- Go to ECS → Task definitions
- Click on fullstackapp-backend-task
- Click "Create new revision"

### **3.2 Add Environment Variables**

Update Container Configuration:

- Keep all previous settings
- In Environment Variables section, add:

Essential Environment Variables:

- NODE_ENV: production
- PORT: 3000
- MONGODB_URI: your-complete-mongodb-atlas-connection-string
- REDIS_HOST: your-redis-primary-endpoint (without :6379)
- REDIS_PORT: 6379

Additional Configuration Variables:

- DB_NAME: fullstackapp
- REDIS_TTL: 3600
- LOG_LEVEL: info
- JWT_SECRET: your-secure-random-string

### **3.3 Store Sensitive Variables in AWS Systems Manager**

Create Parameter Store Values:

- Go to Systems Manager → Parameter Store
- Click "Create parameter"

MongoDB Connection String:

- Name: /fullstackapp/mongodb-uri
- Type: SecureString
- Value: your-mongodb-connection-string
- Click "Create parameter"

JWT Secret:

- Name: /fullstackapp/jwt-secret
- Type: SecureString
- Value: your-secure-random-jwt-secret
- Click "Create parameter"

### **3.4 Update Task Definition for Parameter Store**

Update Task Execution Role:

- Go to IAM → Roles
- Find ecsTaskExecutionRole
- Add policy: AmazonSSMReadOnlyAccess

Update Environment Variables in Task Definition:

- Change sensitive variables to use valueFrom:
- MONGODB_URI: valueFrom: /fullstackapp/mongodb-uri
- JWT_SECRET: valueFrom: /fullstackapp/jwt-secret

### **3.5 Create New Task Definition**

Save New Revision:

- Review all settings
- Click "Create"
- New revision will be created (revision 2)

---

## 🔄 **Step 4: Update ECS Service**

### **4.1 Update Service with New Task Definition**

Navigate to ECS Service:

- Go to ECS → Clusters → fullstackapp-cluster
- Click "Services" tab
- Select fullstackapp-backend-service
- Click "Update"

Update Configuration:

- Task definition: fullstackapp-backend-task:2 (latest revision)
- Keep all other settings same
- Click "Update"

### **4.2 Monitor Service Update**

Check Deployment:

- Go to "Deployments" tab
- Should see new deployment rolling out
- Old tasks will stop, new tasks will start
- Wait for "PRIMARY" deployment to show "STEADY_STATE"

Verify Task Health:

- Go to "Tasks" tab
- New tasks should show "RUNNING" status
- Check "Logs" tab if tasks fail to start

---

## 🧪 **Step 5: Test Database and Cache Connections**

### **5.1 Create Simple Node.js Test Application**

Sample Node.js App for Testing:
Create a simple app to test connections:

Basic package.json dependencies you'll need:

- express: Web framework
- mongoose: MongoDB ODM
- redis: Redis client
- cors: Cross-origin resource sharing

Basic Express.js routes you should implement:

- GET /health: Health check endpoint
- GET /api/test-db: Test MongoDB connection
- GET /api/test-cache: Test Redis connection
- POST /api/users: Create user (MongoDB + Redis caching)
- GET /api/users: List users (with Redis caching)

### **5.2 Container Image Requirements**

For your Node.js application container:

Dockerfile basics:

- Use Node.js 18 Alpine base image
- Copy package.json and install dependencies
- Copy application code
- Expose port 3000
- Set CMD to start your application

Environment variable usage:

- Read MONGODB_URI from environment
- Read REDIS_HOST and REDIS_PORT from environment
- Implement proper error handling for connection failures

### **5.3 Health Check Implementation**

Implement /health Endpoint:
Your health check should verify:

- Application is running
- MongoDB connection is active
- Redis connection is active
- Return HTTP 200 with status information

### **5.4 Test via Load Balancer**

After deploying your actual Node.js app:

Test Health Check:

- Visit: https://api.yourdomain.com/health
- Should return 200 OK with connection status

Test Database Operations:

- Visit: https://api.yourdomain.com/api/test-db
- Should confirm MongoDB connection

Test Cache Operations:

- Visit: https://api.yourdomain.com/api/test-cache
- Should confirm Redis connection

---

## 📊 **Step 6: Database Performance Monitoring**

### **6.1 MongoDB Atlas Monitoring**

Atlas Built-in Monitoring:

- Go to Atlas → Your cluster
- Click "Metrics" tab
- Monitor:
  - Operations per second
  - Connections
  - Network traffic
  - Storage usage

Set Up Atlas Alerts:

- Click "Alerts" in Atlas
- Create alerts for:
  - High CPU usage (>80%)
  - Connection limits approaching
  - Storage usage (>80% of free tier)

### **6.2 ElastiCache Monitoring in CloudWatch**

Redis Metrics to Monitor:

- Go to CloudWatch → Metrics
- Select ElastiCache namespace
- Monitor:
  - CPUUtilization
  - DatabaseMemoryUsagePercentage
  - CurrConnections
  - CacheHits vs CacheMisses

Create CloudWatch Alarms:

- CPU Utilization > 80%
- Memory Usage > 80%
- Cache Hit Ratio < 90%

### **6.3 Application-Level Monitoring**

Log Database Performance:

- Log slow MongoDB queries (>100ms)
- Log Redis cache hit/miss ratios
- Monitor connection pool usage
- Track database response times

ECS Container Monitoring:

- Use CloudWatch Container Insights
- Monitor application logs for database errors
- Set up custom metrics for database operations

---

## 🔒 **Step 7: Database Security Best Practices**

### **7.1 MongoDB Atlas Security**

Network Security:

- In production, whitelist only VPC CIDR blocks
- Use MongoDB Atlas VPC peering for enhanced security
- Enable Atlas audit logs

Authentication and Authorization:

- Use strong passwords (20+ characters)
- Implement least privilege database user roles
- Rotate database passwords regularly
- Use Atlas Access Manager for team access

Data Encryption:

- Encryption at rest: Enabled by default in Atlas
- Encryption in transit: Always use SSL/TLS connections
- Consider client-side field level encryption for sensitive data

### **7.2 ElastiCache Redis Security**

Network Security:

- Keep Redis in private subnets only
- Use security groups to restrict access
- Never expose Redis to public internet

Data Security:

- Enable encryption at rest
- Enable encryption in transit
- Consider Redis AUTH for additional security

Access Control:

- Use IAM policies for ElastiCache API access
- Implement application-level access controls
- Monitor access patterns in CloudWatch

### **7.3 Environment Variable Security**

Secure Storage:

- Use AWS Systems Manager Parameter Store for sensitive data
- Use SecureString type for passwords and secrets
- Implement proper IAM policies for parameter access

Rotation Strategy:

- Rotate database passwords quarterly
- Rotate JWT secrets regularly
- Use AWS Secrets Manager for automated rotation

---

## 💰 **Step 8: Cost Optimization for Database Layer**

### **8.1 MongoDB Atlas Cost Management**

Free Tier Optimization:

- M0 Sandbox: Free forever with limitations
- Monitor storage usage closely
- Clean up test data regularly
- Consider data compression strategies

Scaling Strategy:

- Start with M0 for learning
- Upgrade to M10 only when needed
- Monitor Atlas usage via dashboard
- Use Atlas Data API sparingly (usage-based pricing)

### **8.2 ElastiCache Cost Optimization**

Instance Right-Sizing:

- cache.t3.micro: ~15 USD/month
- cache.t3.small: ~30 USD/month
- Start small and scale based on actual usage

Cost-Saving Configurations:

- Single node (no replicas for learning)
- Disable automatic backups
- Use appropriate eviction policies
- Monitor memory usage and adjust size

Reserved Instances:

- Consider reserved instances for predictable workloads
- Significant savings for 1-year or 3-year commitments
- Only for production environments

### **8.3 Expected Monthly Costs**

Database and Caching Costs:

- MongoDB Atlas M0: Free
- ElastiCache cache.t3.micro: ~15 USD/month
- Data transfer (minimal): ~1-2 USD/month

Total Database Cost: ~15-17 USD/month

**Cost Reduction Strategies:**

- Use MongoDB Atlas M0 free tier
- Single Redis node without replicas
- Monitor and optimize query performance
- Clean up unused data regularly

---

## ✅ **Step 9: Validation Checklist**

Before proceeding to Part 6, verify you have:

**MongoDB Atlas:**

- Cluster created and running
- Database user created with proper permissions
- Network access configured
- Connection string tested and working
- Sample data can be inserted and retrieved

**ElastiCache Redis:**

- Redis cluster created and available
- Proper subnet group configuration
- Security groups allow access from ECS only
- Connection endpoint accessible from ECS tasks

**ECS Integration:**

- Task definition updated with database connection strings
- Environment variables properly configured
- Parameter Store used for sensitive data
- Service updated with new task definition
- Tasks running successfully with database connections

**Security:**

- Database access restricted to application only
- Encryption enabled for data at rest and in transit
- Sensitive credentials stored in Parameter Store
- IAM roles configured with least privilege

**Monitoring:**

- MongoDB Atlas monitoring enabled
- CloudWatch metrics for Redis configured
- Application health checks include database status
- Alerts set up for critical thresholds

---

## 🎯 **What's Next**

In **Part 6**, we'll cover:

- AWS WAF configuration for API protection
- CloudWatch comprehensive logging and metrics
- Security best practices implementation
- Performance monitoring and alerting

---

## 🔧 **Troubleshooting Common Issues**

**MongoDB Connection Failures:**

- Verify connection string format and credentials
- Check Atlas network access whitelist
- Ensure ECS tasks have outbound internet access
- Verify DNS resolution from ECS tasks

**Redis Connection Issues:**

- Check security group allows port 6379 from ECS
- Verify Redis is in same VPC as ECS tasks
- Ensure Redis endpoint is correct
- Check if encryption in transit is properly configured

**ECS Task Startup Failures:**

- Check CloudWatch logs for connection errors
- Verify environment variables are set correctly
- Ensure Parameter Store values are accessible
- Check IAM permissions for task execution role

**Performance Issues:**

- Monitor MongoDB Atlas metrics for slow queries
- Check Redis cache hit ratios
- Review connection pool configurations
- Monitor ECS task resource utilization

**High Costs:**

- Review MongoDB Atlas usage and storage
- Monitor ElastiCache instance sizing
- Check for unnecessary data retention
- Optimize query patterns for efficiency

---

## 💡 **Pro Tips for Database Management**

**Performance Optimization:**

- Implement proper database indexing strategies
- Use Redis for frequently accessed data
- Implement connection pooling in your application
- Monitor and optimize slow queries

**Scalability Planning:**

- Design schema for horizontal scaling
- Implement proper caching layers
- Plan for read replicas when needed
- Consider database sharding strategies

**Data Management:**

- Implement proper backup strategies
- Plan data retention policies
- Use appropriate data types for efficiency
- Implement data validation at application level

**Security Best Practices:**

- Regular security audits of database access
- Implement audit logging for sensitive operations
- Use encryption for sensitive data fields
- Regular password rotation policies

**Ready for Part 6? Let me know when you want to continue with Security & Monitoring!** 🚀

---

# 🛡️ **Part 6: Security & Monitoring**

## 🎯 **What We'll Cover in This Section**

Implementing comprehensive security measures with AWS WAF and setting up monitoring with CloudWatch for production-ready security and observability.

---

## 🔒 **Step 1: AWS WAF (Web Application Firewall) Setup**

### **1.1 Navigate to AWS WAF Service**

Access AWS WAF:

- Search "WAF" in AWS Console
- Click "AWS WAF & Shield"
- You'll see the WAF dashboard

### **1.2 Create Web ACL**

Create Web Access Control List:

- Click "Create web ACL"
- Web ACL name: fullstackapp-waf
- Description: Web Application Firewall for FullStackApp API
- CloudWatch metric name: fullstackappWAF
- Resource type: Application Load Balancer
- Region: US East (N. Virginia)

### **1.3 Associate with Load Balancer**

Associate Resources:

- Select your Application Load Balancer: fullstackapp-alb
- Click "Add"
- This protects your API from common attacks

### **1.4 Add Managed Rule Groups**

Core Rule Set:

- Click "Add rules" → "Add managed rule groups"
- AWS managed rule groups to add:
  - AWS Core Rule Set: Protects against common attacks
  - AWS Known Bad Inputs: Blocks malicious requests
  - AWS SQL Database: Protects against SQL injection
  - AWS Linux Operating System: Protects against Linux-specific attacks

Rate Limiting Rule:

- Click "Add rules" → "Add my own rules and rule groups"
- Rule type: Rate-based rule
- Rule name: RateLimitRule
- Rate limit: 2000 requests per 5 minutes (adjust for your needs)
- IP address to use as the aggregate key: Source IP
- Action: Block

### **1.5 Configure Rule Actions**

Default Action:

- Set to "Allow" (allows traffic that doesn't match any rules)

Rule Priority:

- Rate limiting rule: Priority 1
- Core Rule Set: Priority 2
- Known Bad Inputs: Priority 3
- SQL Database: Priority 4
- Linux OS: Priority 5

### **1.6 Configure CloudWatch Metrics**

Enable Logging:

- CloudWatch metrics: Enabled
- Sampled requests: Enabled (helps with debugging)

### **1.7 Review and Create**

Final Review:

- Review all rule configurations
- Check associated resources
- Click "Create web ACL"
- WAF will start protecting your application immediately

---

## 📊 **Step 2: Comprehensive CloudWatch Setup**

### **2.1 Create Custom Dashboard**

Create Application Dashboard:

- Go to CloudWatch → Dashboards
- Click "Create dashboard"
- Dashboard name: FullStackApp-Monitoring
- Select "Line" widget to start

### **2.2 Add ECS Metrics Widgets**

ECS Service Metrics:

- Add widget: Line graph
- Metrics: ECS → Service metrics
- Select your service: fullstackapp-backend-service
- Metrics to add:
  - CPUUtilization
  - MemoryUtilization
  - RunningTaskCount
  - DesiredCount

ECS Task Metrics:

- Add widget: Number display
- Metrics: ECS → Cluster metrics
- Select cluster: fullstackapp-cluster
- Add ActiveTaskCount and PendingTaskCount

### **2.3 Add Application Load Balancer Metrics**

ALB Performance Metrics:

- Add widget: Line graph
- Metrics: ApplicationELB → Per AppELB metrics
- Select your ALB: fullstackapp-alb
- Metrics to add:
  - RequestCount
  - TargetResponseTime
  - HTTPCode_Target_2XX_Count
  - HTTPCode_Target_4XX_Count
  - HTTPCode_Target_5XX_Count

ALB Health Metrics:

- Add widget: Number display
- Metrics: ApplicationELB → Per Target Group
- Select: fullstackapp-tg
- Add HealthyHostCount and UnHealthyHostCount

### **2.4 Add WAF Metrics**

WAF Security Metrics:

- Add widget: Line graph
- Metrics: AWS/WAFV2
- WebACL: fullstackapp-waf
- Metrics to add:
  - AllowedRequests
  - BlockedRequests
  - SampledRequests

### **2.5 Add Database and Cache Metrics**

ElastiCache Metrics:

- Add widget: Line graph
- Metrics: AWS/ElastiCache
- CacheClusterId: fullstackapp-redis
- Metrics to add:
  - CPUUtilization
  - DatabaseMemoryUsagePercentage
  - CacheHits
  - CacheMisses

### **2.6 Save Dashboard**

Finalize Dashboard:

- Arrange widgets logically
- Add text widgets for section headers
- Save dashboard
- Set auto-refresh to 5 minutes

---

## 🚨 **Step 3: CloudWatch Alarms Setup**

### **3.1 Critical Application Alarms**

High CPU Utilization Alarm:

- CloudWatch → Alarms → Create alarm
- Metric: ECS Service CPU Utilization
- Service: fullstackapp-backend-service
- Statistic: Average
- Period: 5 minutes
- Threshold: Greater than 80%
- Datapoints: 2 out of 3
- Name: FullStackApp-HighCPU
- Description: CPU utilization is too high

High Memory Utilization Alarm:

- Similar to CPU alarm
- Metric: ECS Service Memory Utilization
- Threshold: Greater than 85%
- Name: FullStackApp-HighMemory

### **3.2 Application Load Balancer Alarms**

High Response Time Alarm:

- Metric: ALB Target Response Time
- Load Balancer: fullstackapp-alb
- Statistic: Average
- Period: 5 minutes
- Threshold: Greater than 2 seconds
- Name: FullStackApp-HighResponseTime

High Error Rate Alarm:

- Metric: ALB HTTPCode_Target_5XX_Count
- Statistic: Sum
- Period: 5 minutes
- Threshold: Greater than 10
- Name: FullStackApp-HighErrorRate

### **3.3 Security Alarms**

WAF Blocked Requests Alarm:

- Metric: WAF BlockedRequests
- WebACL: fullstackapp-waf
- Statistic: Sum
- Period: 5 minutes
- Threshold: Greater than 100
- Name: FullStackApp-SecurityThreats

### **3.4 Database Performance Alarms**

Redis High Memory Alarm:

- Metric: ElastiCache DatabaseMemoryUsagePercentage
- CacheClusterId: fullstackapp-redis
- Threshold: Greater than 80%
- Name: FullStackApp-RedisHighMemory

Redis Low Cache Hit Rate:

- Calculated metric: CacheHits / (CacheHits + CacheMisses) \* 100
- Threshold: Less than 80%
- Name: FullStackApp-LowCacheHitRate

---

## 📧 **Step 4: SNS Notification Setup**

### **4.1 Create SNS Topic**

Create Alert Topic:

- Go to SNS → Topics
- Click "Create topic"
- Type: Standard
- Name: fullstackapp-alerts
- Display name: FullStackApp Alerts
- Click "Create topic"

### **4.2 Create Subscriptions**

Email Subscription:

- Click on your topic
- Click "Create subscription"
- Protocol: Email
- Endpoint: your-email@example.com
- Click "Create subscription"
- Check email and confirm subscription

SMS Subscription (Optional):

- Create another subscription
- Protocol: SMS
- Endpoint: your-phone-number
- For critical alerts only

### **4.3 Link Alarms to SNS**

Update All Alarms:

- Go back to each CloudWatch alarm
- Click "Edit"
- In "Actions" section
- Add action: Send notification to SNS topic
- Topic: fullstackapp-alerts
- Save changes

---

## 🔍 **Step 5: CloudWatch Logs Configuration**

### **5.1 Create Custom Log Groups**

Application Logs:

- CloudWatch → Log groups
- Create log group: /aws/fullstackapp/application
- Retention: 7 days (for learning, 30+ days for production)

Security Logs:

- Create log group: /aws/fullstackapp/security
- Retention: 30 days

### **5.2 Configure WAF Logging**

Enable WAF Logging:

- Go to AWS WAF → Web ACLs
- Click on fullstackapp-waf
- Go to "Logging and metrics" tab
- Click "Enable logging"
- Destination: CloudWatch Logs
- Log group: /aws/wafv2/logs
- Click "Enable"

### **5.3 Set Up Log Insights Queries**

Common Query Examples:

- Save these queries in CloudWatch Logs Insights

Error Analysis Query:

```

fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100

```

Performance Query:

```

fields @timestamp, @message
| filter @message like /response_time/
| stats avg(response_time) by bin(5m)

```

Security Events Query:

```

fields @timestamp, action, terminatingRuleId
| filter action = "BLOCK"
| stats count() by terminatingRuleId

```

---

## 🛠️ **Step 6: Application Performance Monitoring**

### **6.1 Enable AWS X-Ray (Optional)**

Setup X-Ray for Request Tracing:

- Go to X-Ray console
- Create service map for your application
- Add X-Ray SDK to your Node.js application
- Trace database calls and external API requests

### **6.2 Custom Metrics from Application**

Application-Level Metrics:
Your Node.js app should send custom metrics:

- Database query response times
- Cache hit/miss ratios
- Business-specific metrics (user registrations, orders, etc.)
- Error rates by endpoint

### **6.3 Health Check Enhancements**

Enhanced Health Endpoint:
Your /health endpoint should return:

- Application status
- Database connectivity
- Redis connectivity
- Dependency service status
- System resource usage

---

## 🔐 **Step 7: Security Best Practices Implementation**

### **7.1 IAM Role Hardening**

ECS Task Role Security:

- Go to IAM → Roles → your ECS task role
- Remove any unnecessary permissions
- Follow principle of least privilege
- Add only required permissions:
  - SSM Parameter access for secrets
  - CloudWatch logs write access
  - S3 access if needed for file uploads

Parameter Store Access:

- Create specific IAM policy for parameter access
- Restrict to only your application parameters
- Use resource-based permissions

### **7.2 VPC Security Enhancements**

VPC Flow Logs:

- Go to VPC → Your VPC
- Actions → Create flow log
- Destination: CloudWatch Logs
- Log group: /aws/vpc/flowlogs
- IAM role: Create new role
- This logs all network traffic

Security Group Audit:

- Review all security groups
- Remove any overly permissive rules
- Document the purpose of each rule
- Regular audit schedule

### **7.3 SSL/TLS Configuration**

ALB Security Policy:

- Go to EC2 → Load Balancers → fullstackapp-alb
- Edit HTTPS listener
- Security policy: Use latest TLS 1.3 policy
- Ensure no weak ciphers are enabled

Certificate Monitoring:

- Set up CloudWatch alarm for certificate expiration
- AWS Certificate Manager auto-renewal should be verified
- Monitor certificate usage in CloudWatch

---

## 📈 **Step 8: Performance Optimization Monitoring**

### **8.1 Database Performance Monitoring**

MongoDB Atlas Monitoring:

- Review Atlas performance advisor
- Set up Atlas alerts for slow queries
- Monitor connection pool usage
- Track index usage and optimize

Redis Performance:

- Monitor cache hit ratios
- Track memory usage patterns
- Optimize TTL settings based on usage
- Monitor connection counts

### **8.2 Application Performance Baselines**

Establish Performance Baselines:

- Document normal response times
- Set up percentile-based alerts (P95, P99)
- Monitor resource utilization trends
- Create capacity planning metrics

### **8.3 Cost Monitoring Integration**

Cost Alerts:

- Set up detailed cost tracking per service
- Monitor cost trends and unusual spikes
- Create cost-based alarms
- Regular cost optimization reviews

---

## 📋 **Step 9: Incident Response Procedures**

### **9.1 Runbook Creation**

Create Operational Runbooks:

- High CPU utilization response
- Database connection failures
- Security incident response
- Deployment rollback procedures
- Scale-out procedures

### **9.2 Escalation Procedures**

Alert Escalation:

- Level 1: Email notifications
- Level 2: SMS for critical issues
- Level 3: PagerDuty or similar for production
- Define severity levels and response times

### **9.3 Monitoring the Monitoring**

Monitor Your Monitoring:

- Set up alarms for alarm failures
- Monitor CloudWatch API usage
- Verify log ingestion rates
- Regular testing of notification channels

---

## ✅ **Step 10: Validation Checklist**

Before proceeding to Part 7, verify you have:

**WAF Security:**

- Web ACL created and associated with ALB
- Managed rule groups configured
- Rate limiting rules in place
- Logging enabled for security events

**CloudWatch Monitoring:**

- Custom dashboard with all key metrics
- Comprehensive alarm coverage
- SNS notifications configured
- Log groups organized and retention set

**Application Monitoring:**

- ECS service metrics tracking
- ALB performance monitoring
- Database and cache monitoring
- Custom application metrics (if implemented)

**Security Implementation:**

- IAM roles follow least privilege
- VPC Flow Logs enabled
- SSL/TLS properly configured
- Security group rules audited

**Incident Response:**

- Notification channels tested
- Escalation procedures defined
- Runbooks created
- Monitoring validated

---

## 🎯 **What's Next**

In **Part 7**, we'll cover:

- GitHub Actions CI/CD pipeline setup
- Automated testing and deployment
- Environment management strategies
- Quality gates and deployment safeguards

---

## 💰 **Security & Monitoring Costs**

**Monthly Cost Breakdown:**

- AWS WAF: 1 USD base + 1 USD per rule + request charges (~5-10 USD)
- CloudWatch Dashboards: 3 USD per dashboard
- CloudWatch Alarms: 0.10 USD per alarm (up to 10 alarms = 1 USD)
- CloudWatch Logs: Based on ingestion and storage (~2-5 USD)
- SNS: 0.50 USD per million notifications

**Total Security & Monitoring: ~10-20 USD per month**

---

## 🔧 **Troubleshooting Common Issues**

**WAF Blocking Legitimate Traffic:**

- Review WAF logs in CloudWatch
- Adjust rule sensitivity or add exceptions
- Use WAF count mode for testing rules
- Monitor blocked request patterns

**Missing CloudWatch Metrics:**

- Verify IAM permissions for CloudWatch
- Check metric retention periods
- Ensure services are publishing metrics
- Verify region consistency

**Alarm Fatigue:**

- Review alarm thresholds and adjust
- Implement alarm suppression during maintenance
- Use composite alarms for related metrics
- Regular alarm effectiveness review

**High Monitoring Costs:**

- Optimize log retention periods
- Use metric filters to reduce noise
- Archive old dashboard data
- Review unnecessary detailed monitoring

---

## 💡 **Pro Tips for Security & Monitoring**

**WAF Optimization:**

- Start with WAF in count mode to understand traffic
- Gradually enable blocking as you tune rules
- Regular review of blocked vs allowed traffic
- Geographic blocking if applicable

**Monitoring Best Practices:**

- Focus on business-critical metrics
- Use statistical analysis for anomaly detection
- Implement predictive alerting where possible
- Regular review and cleanup of unused metrics

**Cost Optimization:**

- Use log sampling for high-volume logs
- Implement log lifecycle policies
- Use reserved capacity for predictable usage
- Regular cost optimization reviews

**Security Automation:**

- Implement automated incident response
- Use AWS Config for compliance monitoring
- Regular security assessment automation
- Automated remediation where possible

**Ready for Part 7? Let me know when you want to continue with CI/CD Pipeline!** 🚀

---

# 🚀 **Part 7: CI/CD Pipeline Setup**

## 🎯 **What We'll Cover in This Section**

Setting up comprehensive CI/CD pipelines using both GitHub Actions and Jenkins for automated testing, building, and deployment of your full-stack application to AWS.

---

## 📋 **Pipeline Options Overview**

We'll cover two popular CI/CD solutions:

### **Option A: GitHub Actions** _(Cloud-based, Easy Setup)_

- Integrated with GitHub repositories
- No infrastructure management required
- Pay-per-use pricing model
- Excellent for modern DevOps workflows

### **Option B: Jenkins** _(Self-hosted, Full Control)_

- Run on AWS EC2 instance
- Complete customization and control
- Plugin ecosystem for extensive integrations
- Traditional CI/CD solution

---

# 🎯 **Option A: GitHub Actions CI/CD Pipeline**

## 🔧 **Step 1: Repository Setup**

### **1.1 Initialize Git Repository**

Create Local Repository:

- Open terminal in your project root
- Run: git init
- Run: git add .
- Run: git commit -m "Initial commit"

### **1.2 Create GitHub Repository**

Create Remote Repository:

- Go to github.com
- Click "New repository"
- Repository name: fullstackapp-deployment
- Visibility: Private (recommended for learning)
- Don't initialize with README (you have local files)
- Click "Create repository"

Connect Local to Remote:

- Copy the repository URL
- Run: git remote add origin your-repository-url
- Run: git branch -M main
- Run: git push -u origin main

### **1.3 Repository Structure**

Organize your repository:

```

fullstackapp-deployment/
├── frontend/ # Angular application
│ ├── src/
│ ├── package.json
│ └── angular.json
├── backend/ # Node.js application
│ ├── src/
│ ├── package.json
│ └── Dockerfile
├── .github/
│ └── workflows/
│ ├── frontend-deploy.yml
│ ├── backend-deploy.yml
│ └── full-deploy.yml
└── infrastructure/ # Terraform or CloudFormation (optional)

````

---

## 🐳 **Step 2: Docker Configuration**

### **2.1 Create Dockerfile for Node.js Backend**

Create backend/Dockerfile:

```dockerfile
# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# Change ownership of app directory
RUN chown -R nodejs:nodejs /app
USER nodejs

# Expose port
EXPOSE 3000

# Define health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start the application
CMD ["npm", "start"]
````

### **2.2 Create .dockerignore**

Create backend/.dockerignore:

```
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
.coverage
.coverage/
.vscode
```

### **2.3 Update package.json for Docker**

Add to backend/package.json scripts:

```json
{
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest",
    "test:coverage": "jest --coverage",
    "docker:build": "docker build -t fullstackapp-backend .",
    "docker:run": "docker run -p 3000:3000 fullstackapp-backend"
  }
}
```

---

## 🔐 **Step 3: AWS Credentials Setup**

### **3.1 Create IAM User for CI/CD**

Create Deployment User:

- Go to IAM → Users
- Click "Create user"
- Username: fullstackapp-cicd-user
- Access type: Programmatic access only

Attach Policies:

- AmazonECS_FullAccess
- AmazonS3FullAccess
- CloudFrontFullAccess
- AmazonEC2ContainerRegistryFullAccess
- AWSCloudFormationFullAccess (if using infrastructure as code)

Create Access Keys:

- Go to Security credentials tab
- Create access key → Command Line Interface
- Download CSV file with credentials

### **3.2 Setup GitHub Secrets**

Add Repository Secrets:

- Go to GitHub repository → Settings → Secrets and variables → Actions
- Click "New repository secret"

Required Secrets:

- AWS_ACCESS_KEY_ID: Your IAM user access key
- AWS_SECRET_ACCESS_KEY: Your IAM user secret key
- AWS_REGION: us-east-1
- ECR_REPOSITORY_URI: your-account-id.dkr.ecr.us-east-1.amazonaws.com/fullstackapp
- S3_BUCKET_NAME: your-frontend-bucket-name
- CLOUDFRONT_DISTRIBUTION_ID: your-cloudfront-distribution-id
- ECS_CLUSTER_NAME: fullstackapp-cluster
- ECS_SERVICE_NAME: fullstackapp-backend-service

### **3.3 Create ECR Repository**

Create Container Registry:

- Go to ECR (Elastic Container Registry)
- Click "Create repository"
- Repository name: fullstackapp-backend
- Visibility: Private
- Click "Create repository"
- Copy the repository URI for GitHub secrets

---

## 🔄 **Step 4: GitHub Actions Workflows**

### **4.1 Frontend Deployment Workflow**

Create .github/workflows/frontend-deploy.yml:

```yaml
name: Deploy Frontend to S3 and CloudFront

on:
  push:
    branches: [main]
    paths: ["frontend/**"]
  workflow_dispatch:

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci

      - name: Run tests
        working-directory: ./frontend
        run: npm run test -- --watch=false --browsers=ChromeHeadless

      - name: Run linting
        working-directory: ./frontend
        run: npm run lint

      - name: Build Angular app
        working-directory: ./frontend
        run: npm run build --prod

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Deploy to S3
        working-directory: ./frontend
        run: |
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"

      - name: Notify deployment success
        if: success()
        run: echo "Frontend deployed successfully to S3 and CloudFront cache invalidated"
```

### **4.2 Backend Deployment Workflow**

Create .github/workflows/backend-deploy.yml:

```yaml
name: Deploy Backend to ECS

on:
  push:
    branches: [main]
    paths: ["backend/**"]
  workflow_dispatch:

jobs:
  deploy-backend:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: "npm"
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        working-directory: ./backend
        run: npm ci

      - name: Run tests
        working-directory: ./backend
        run: npm run test

      - name: Run security audit
        working-directory: ./backend
        run: npm audit --audit-level=high

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push Docker image
        working-directory: ./backend
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: fullstackapp-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Build Docker image
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .

          # Push Docker image to ECR
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      - name: Update ECS service
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: fullstackapp-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Update task definition with new image
          aws ecs describe-task-definition \
            --task-definition fullstackapp-backend-task \
            --query taskDefinition > task-def.json

          # Update image URI in task definition
          jq --arg IMAGE_URI "$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" \
            '.containerDefinitions[0].image = $IMAGE_URI' \
            task-def.json > updated-task-def.json

          # Remove unnecessary fields
          jq 'del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.placementConstraints) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)' \
            updated-task-def.json > final-task-def.json

          # Register new task definition
          aws ecs register-task-definition \
            --cli-input-json file://final-task-def.json

          # Update ECS service
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER_NAME }} \
            --service ${{ secrets.ECS_SERVICE_NAME }} \
            --task-definition fullstackapp-backend-task

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster ${{ secrets.ECS_CLUSTER_NAME }} \
            --services ${{ secrets.ECS_SERVICE_NAME }}

      - name: Notify deployment success
        if: success()
        run: echo "Backend deployed successfully to ECS"
```

### **4.3 Full Application Deployment Workflow**

Create .github/workflows/full-deploy.yml:

```yaml
name: Deploy Full Stack Application

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Test Frontend
        working-directory: ./frontend
        run: |
          npm ci
          npm run test -- --watch=false --browsers=ChromeHeadless
          npm run lint
          npm run build --prod

      - name: Test Backend
        working-directory: ./backend
        run: |
          npm ci
          npm run test
          npm audit --audit-level=high

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/workflows/frontend-deploy.yml

  deploy-backend:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/workflows/backend-deploy.yml

  integration-test:
    needs: [deploy-frontend, deploy-backend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Run integration tests
        run: |
          # Add your integration test commands here
          echo "Running integration tests against deployed application"
          # Example: curl tests, Postman tests, etc.
```

---

# 🔧 **Option B: Jenkins CI/CD Pipeline**

## 🏗️ **Step 5: Jenkins Infrastructure Setup**

### **5.1 Create Jenkins EC2 Instance**

Launch EC2 Instance:

- Go to EC2 → Launch Instance
- AMI: Amazon Linux 2
- Instance type: t3.medium (minimum for Jenkins)
- Key pair: Create new or use existing
- Security group: Create jenkins-sg

Jenkins Security Group:

- SSH (22): Your IP address
- HTTP (8080): Your IP address (Jenkins web interface)
- HTTPS (443): Your IP address (if using SSL)

### **5.2 Install Jenkins on EC2**

Connect to Instance:

- SSH into your EC2 instance
- Run: ssh -i your-key.pem ec2-user@your-instance-ip

Install Jenkins:

```bash
# Update system
sudo yum update -y

# Install Java (required for Jenkins)
sudo yum install -y java-11-openjdk-devel

# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Jenkins
sudo yum install -y jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Install Docker (for building containers)
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker

# Add jenkins user to docker group
sudo usermod -a -G docker jenkins

# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Install Node.js and npm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18
```

### **5.3 Initial Jenkins Configuration**

Access Jenkins:

- Open browser: http://your-ec2-instance-ip:8080
- Get initial admin password: sudo cat /var/lib/jenkins/secrets/initialAdminPassword
- Install suggested plugins
- Create admin user
- Configure instance URL

Install Required Plugins:

- AWS Pipeline Plugin
- Docker Pipeline Plugin
- NodeJS Plugin
- Git Plugin
- Pipeline: Stage View Plugin
- Blue Ocean Plugin (for better UI)

### **5.4 Configure Jenkins Tools**

Configure NodeJS:

- Manage Jenkins → Global Tool Configuration
- NodeJS installations → Add NodeJS
- Name: NodeJS-18
- Version: 18.x
- Install automatically: Yes

Configure Docker:

- Add Docker installation
- Name: Docker
- Install automatically from docker.com

Configure AWS CLI:

- Global properties → Environment variables
- Add: AWS_DEFAULT_REGION = us-east-1

---

## 🔑 **Step 6: Jenkins Credentials Setup**

### **6.1 Add AWS Credentials**

Add AWS Credentials:

- Manage Jenkins → Credentials → Global
- Add Credentials → AWS Credentials
- ID: aws-credentials
- Access Key ID: Your IAM user access key
- Secret Access Key: Your IAM user secret key

### **6.2 Add GitHub Credentials**

Add GitHub Access:

- Add Credentials → Username with password
- ID: github-credentials
- Username: Your GitHub username
- Password: GitHub Personal Access Token

### **6.3 Add ECR Registry Info**

Add ECR Details:

- Add Credentials → Secret text
- ID: ecr-repository-uri
- Secret: your-account-id.dkr.ecr.us-east-1.amazonaws.com/fullstackapp

---

## 📋 **Step 7: Jenkins Pipeline Configuration**

### **7.1 Create Jenkinsfile for Full Pipeline**

Create Jenkinsfile in repository root:

```groovy
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECR_REPOSITORY_URI = credentials('ecr-repository-uri')
        S3_BUCKET_NAME = 'your-frontend-bucket-name'
        CLOUDFRONT_DISTRIBUTION_ID = 'your-cloudfront-distribution-id'
        ECS_CLUSTER_NAME = 'fullstackapp-cluster'
        ECS_SERVICE_NAME = 'fullstackapp-backend-service'
        ECS_TASK_DEFINITION = 'fullstackapp-backend-task'
    }

    tools {
        nodejs 'NodeJS-18'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('Frontend Dependencies') {
                    steps {
                        dir('frontend') {
                            sh 'npm ci'
                        }
                    }
                }
                stage('Backend Dependencies') {
                    steps {
                        dir('backend') {
                            sh 'npm ci'
                        }
                    }
                }
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Frontend Tests') {
                    steps {
                        dir('frontend') {
                            sh 'npm run test -- --watch=false --browsers=ChromeHeadless'
                            sh 'npm run lint'
                        }
                    }
                }
                stage('Backend Tests') {
                    steps {
                        dir('backend') {
                            sh 'npm run test'
                            sh 'npm audit --audit-level=high'
                        }
                    }
                }
            }
        }

        stage('Build Applications') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        dir('frontend') {
                            sh 'npm run build --prod'
                        }
                    }
                }
                stage('Build Backend Docker Image') {
                    steps {
                        script {
                            dir('backend') {
                                // Build Docker image
                                def image = docker.build("fullstackapp-backend:${BUILD_NUMBER}")

                                // Login to ECR
                                sh '''
                                    aws ecr get-login-password --region $AWS_DEFAULT_REGION | \
                                    docker login --username AWS --password-stdin $ECR_REPOSITORY_URI
                                '''

                                // Tag and push image
                                def repositoryName = ECR_REPOSITORY_URI.split('/')[1]
                                image.push("${BUILD_NUMBER}")
                                image.push("latest")
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to AWS') {
            when {
                branch 'main'
            }
            parallel {
                stage('Deploy Frontend') {
                    steps {
                        withAWS(credentials: 'aws-credentials', region: env.AWS_DEFAULT_REGION) {
                            dir('frontend') {
                                // Sync to S3
                                sh 'aws s3 sync dist/ s3://$S3_BUCKET_NAME --delete'

                                // Invalidate CloudFront
                                sh '''
                                    aws cloudfront create-invalidation \
                                        --distribution-id $CLOUDFRONT_DISTRIBUTION_ID \
                                        --paths "/*"
                                '''
                            }
                        }
                    }
                }
                stage('Deploy Backend') {
                    steps {
                        withAWS(credentials: 'aws-credentials', region: env.AWS_DEFAULT_REGION) {
                            script {
                                // Update ECS service with new image
                                def imageUri = "${ECR_REPOSITORY_URI}:${BUILD_NUMBER}"

                                sh '''
                                    # Get current task definition
                                    aws ecs describe-task-definition \
                                        --task-definition $ECS_TASK_DEFINITION \
                                        --query taskDefinition > task-def.json

                                    # Update image URI
                                    jq --arg IMAGE_URI "''' + imageUri + '''" \
                                        '.containerDefinitions[0].image = $IMAGE_URI' \
                                        task-def.json > updated-task-def.json

                                    # Clean up unnecessary fields
                                    jq 'del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.placementConstraints) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)' \
                                        updated-task-def.json > final-task-def.json

                                    # Register new task definition
                                    aws ecs register-task-definition \
                                        --cli-input-json file://final-task-def.json

                                    # Update service
                                    aws ecs update-service \
                                        --cluster $ECS_CLUSTER_NAME \
                                        --service $ECS_SERVICE_NAME \
                                        --task-definition $ECS_TASK_DEFINITION

                                    # Wait for deployment
                                    aws ecs wait services-stable \
                                        --cluster $ECS_CLUSTER_NAME \
                                        --services $ECS_SERVICE_NAME
                                '''
                            }
                        }
                    }
                }
            }
        }

        stage('Integration Tests') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Add integration tests here
                    sh '''
                        echo "Running integration tests..."
                        # Add your integration test commands
                        # Example: curl tests, API tests, etc.
                    '''
                }
            }
        }
    }

    post {
        always {
            // Clean workspace
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // Add notification logic here
        }
        failure {
            echo 'Pipeline failed!'
            // Add notification logic here
        }
    }
}
```

### **7.2 Create Jenkins Pipeline Job**

Create Pipeline Job:

- New Item → Pipeline → fullstackapp-pipeline
- Pipeline script from SCM
- SCM: Git
- Repository URL: Your GitHub repository URL
- Credentials: github-credentials
- Branch: main
- Script Path: Jenkinsfile

---

## 📊 **Step 8: Monitoring and Notifications**

### **8.1 GitHub Actions Monitoring**

Add Status Badges:
Add to your README.md:

```markdown
[![Frontend Deploy](https://github.com/your-username/your-repo/actions/workflows/frontend-deploy.yml/badge.svg)](https://github.com/your-username/your-repo/actions/workflows/frontend-deploy.yml)
[![Backend Deploy](https://github.com/your-username/your-repo/actions/workflows/backend-deploy.yml/badge.svg)](https://github.com/your-username/your-repo/actions/workflows/backend-deploy.yml)
```

Slack Notifications (Optional):
Add to workflow:

```yaml
- name: Slack Notification
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### **8.2 Jenkins Monitoring**

Install Notification Plugins:

- Email Extension Plugin
- Slack Notification Plugin
- Build Monitor View Plugin

Configure Email Notifications:

- Manage Jenkins → Configure System
- Extended E-mail Notification
- SMTP Server: Your email provider
- Add post-build actions to jobs

---

## 🛡️ **Step 9: Security and Quality Gates**

### **9.1 GitHub Actions Security**

Add Security Scanning:

```yaml
- name: Run security scan
  uses: github/super-linter@v4
  env:
    DEFAULT_BRANCH: main
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- name: Dependency vulnerability scan
  uses: actions/dependency-review-action@v3
```

SonarQube Integration:

```yaml
- name: SonarQube Scan
  uses: sonarqube-quality-gate-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### **9.2 Jenkins Security**

Add Security Stages:

```groovy
stage('Security Scan') {
    steps {
        script {
            // SAST scanning
            sh 'npm audit --audit-level=high'

            // Docker image security scan
            sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image fullstackapp-backend:latest'
        }
    }
}
```

---

## 📈 **Step 10: Performance and Optimization**

### **10.1 Build Optimization**

Cache Dependencies:

- GitHub Actions: Uses built-in caching
- Jenkins: Use pipeline caching plugins

Parallel Builds:

- Both platforms support parallel stage execution
- Frontend and backend can build simultaneously

### **10.2 Deployment Strategies**

Blue-Green Deployment:

- ECS supports blue-green via CodeDeploy
- Implement gradual traffic shifting

Rollback Strategies:

- Keep previous task definition versions
- Quick rollback using AWS CLI commands

---

## ✅ **Step 11: Validation Checklist**

Verify your CI/CD setup has:

**GitHub Actions (Option A):**

- Repository secrets configured
- ECR repository created
- Workflows trigger on code changes
- Successful frontend deployment to S3/CloudFront
- Successful backend deployment to ECS
- Integration tests passing

**Jenkins (Option B):**

- EC2 instance running Jenkins
- Required plugins installed
- AWS credentials configured
- Pipeline job created and working
- Docker builds and pushes to ECR
- ECS service updates automatically

**Common Validations:**

- Automated testing in pipeline
- Security scanning enabled
- Deployment notifications working
- Rollback procedures documented
- Performance monitoring active

---

## 💰 **CI/CD Cost Analysis**

### **GitHub Actions Costs:**

- 2,000 free minutes per month (private repos)
- Additional: 0.008 USD per minute
- Expected monthly: 5-15 USD for learning

### **Jenkins on EC2 Costs:**

- t3.medium instance: ~30 USD/month
- EBS storage: ~5 USD/month
- Data transfer: ~2 USD/month
- Expected monthly: ~37 USD

### **Cost Optimization Tips:**

- Use GitHub Actions free tier efficiently
- Stop Jenkins EC2 when not needed
- Use spot instances for Jenkins (if acceptable downtime)
- Optimize build times to reduce compute costs

---

## 🎯 **Next Steps and Best Practices**

### **Production Readiness Checklist:**

- [ ] Multi-environment setup (dev/staging/prod)
- [ ] Comprehensive test coverage (>80%)
- [ ] Security scanning integrated
- [ ] Monitoring and alerting configured
- [ ] Backup and disaster recovery plan
- [ ] Documentation updated
- [ ] Team training on CI/CD processes

### **Advanced Features to Implement:**

- Infrastructure as Code (Terraform/CloudFormation)
- Feature flagging and gradual rollouts
- Automated performance testing
- Cross-region deployments
- Compliance scanning (SOC2, PCI-DSS)

---

## 🎉 **Congratulations!**

You've successfully created a comprehensive AWS full-stack deployment with enterprise-grade CI/CD pipelines! Your architecture now includes:

✅ **Complete Infrastructure**: VPC, ECS, ALB, databases, caching  
✅ **Security**: WAF, SSL, encrypted connections, least privilege access  
✅ **Monitoring**: CloudWatch dashboards, alarms, and logging  
✅ **Automation**: Both GitHub Actions and Jenkins CI/CD options  
✅ **Scalability**: Auto-scaling, load balancing, multi-AZ deployment  
✅ **Cost Optimization**: Right-sized resources with monitoring

## 🚀 **Total Learning Investment:**

**Time Investment:** 15-20 hours to complete all parts  
**Monthly Cost:** 150-200 USD (can be optimized to 50-75 USD for learning)  
**Skills Gained:** Enterprise-level AWS deployment, DevOps, and automation

**You're now ready to deploy production-grade applications on AWS!** 🎯

---

## 🔧 **Troubleshooting CI/CD Issues**

### **GitHub Actions Common Issues:**

**Docker Build Failures:**

- Check Dockerfile syntax and dependencies
- Verify ECR permissions in IAM
- Ensure sufficient GitHub Actions minutes

**ECS Deployment Failures:**

- Verify task definition JSON formatting
- Check ECS service capacity and resources
- Validate security group configurations

**S3 Sync Issues:**

- Confirm S3 bucket permissions
- Check AWS credentials in secrets
- Verify CloudFront distribution settings

### **Jenkins Common Issues:**

**Plugin Conflicts:**

- Keep plugins updated
- Test in staging environment first
- Monitor Jenkins logs for errors

**AWS CLI Issues:**

- Verify IAM permissions
- Check AWS CLI version compatibility
- Ensure proper credentials configuration

**Pipeline Performance:**

- Use pipeline parallelization
- Implement proper caching strategies
- Monitor resource usage on EC2

### **General Troubleshooting:**

**Network Connectivity:**

- Verify VPC and security group settings
- Check NAT Gateway configuration
- Validate Route 53 DNS resolution

**Cost Overruns:**

- Monitor AWS Cost Explorer daily
- Set up billing alerts
- Review resource utilization regularly

---

## 💡 **Pro Tips for CI/CD Success**

### **Development Workflow:**

- Use feature branches for development
- Implement proper code review processes
- Maintain clean commit history
- Tag releases for easy rollbacks

### **Security Best Practices:**

- Rotate credentials regularly
- Use least privilege access patterns
- Scan containers for vulnerabilities
- Implement secret management properly

### **Performance Optimization:**

- Monitor build times and optimize
- Use caching strategies effectively
- Implement proper testing pyramids
- Scale infrastructure based on actual usage

### **Team Collaboration:**

- Document pipeline procedures clearly
- Provide team training on CI/CD tools
- Establish incident response procedures
- Regular pipeline maintenance and updates

---

**Your AWS full-stack deployment guide is now complete! Time to build amazing applications! 🚀**
