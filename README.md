Think of the **Load Balancer** as the main entrance to an apartment building.

* The **Home Page** is the lobby (Path `/`).
* **AWS, Azure, and GCP** are three different apartments (Paths `/aws`, `/azure`, `/gcp`).
* The **Target Groups** are the specific doorways to those apartments.
* The **EC2 Instances** are the actual residents inside those apartments.

---

### 🏗️ The Architecture Breakdown

* **1 Application Load Balancer (ALB):** The single entry point.
* **4 Target Groups (TG):**

  * `TG-Home` (Path `/`)
  * `TG-AWS` (Path `/aws`)
  * `TG-Azure` (Path `/azure`)
  * `TG-GCP` (Path `/gcp`)
* **4 EC2 Instances:** One dedicated to each application.
* **Listener Rules:** The routing logic inside the ALB.

---

### 🚀 Step-by-Step Implementation Guide

#### Phase 1: Provision the Compute (The 4 Servers)

Launch 4 EC2 instances (e.g., `t2.micro` or `t3.micro` with Amazon Linux 2023).

1. **Instance 1 (Home):** Install Apache/Nginx. Create an `index.html` that says "Welcome to the Home Page".
2. **Instance 2 (AWS):** Install Apache/Nginx. Create an `aws.html` (or a folder `/aws/index.html`) that says "Welcome to the AWS App".
3. **Instance 3 (Azure):** Install Apache/Nginx. Create an `azure.html` (or `/azure/index.html`) that says "Welcome to the Azure App".
4. **Instance 4 (GCP):** Install Apache/Nginx. Create a `gcp.html` (or `/gcp/index.html`) that says "Welcome to the GCP App".

> **Trainer Tip:** Make sure your EC2 Security Groups allow HTTP (Port 80) traffic from the Load Balancer's Security Group.

#### Phase 2: Create the Target Groups (The Doorways)

Go to the EC2 Console -> Target Groups. Create 4 Target Groups (Type: Instances).

1. **`TG-Home`**: Register Instance 1. Health check path: `/`.
2. **`TG-AWS`**: Register Instance 2. Health check path: `/aws` (or `/aws.html`).
3. **`TG-Azure`**: Register Instance 3. Health check path: `/azure` (or `/azure.html`).
4. **`TG-GCP`**: Register Instance 4. Health check path: `/gcp` (or `/gcp.html`).

#### Phase 3: Create the Application Load Balancer (The Front Door)

1. Create an **Internet-facing** ALB.
2. Assign it to at least 2 Availability Zones (best practice).
3. **Listeners:** Create an HTTP:80 listener.
4. **Default Action:** Forward traffic to **`TG-Home`**. (This acts as your `/` catch-all).

#### Phase 4: Configure the Listener Rules (The Routing Magic)

This is where you connect the LB to the other 3 applications. In the ALB console, click on your Listener (HTTP:80) and go to **Rules**.

You will create 3 specific rules (and keep the default rule as your Home page):

* **Rule 1 (AWS):**

  * **IF:** Path is `/aws` OR `/aws/*`
  * **THEN:** Forward to `TG-AWS`
* **Rule 2 (Azure):**

  * **IF:** Path is `/azure` OR `/azure/*`
  * **THEN:** Forward to `TG-Azure`
* **Rule 3 (GCP):**

  * **IF:** Path is `/gcp` OR `/gcp/*`
  * **THEN:** Forward to `TG-GCP`
* **Default Rule (Home):**

  * **IF:** No other rules match
  * **THEN:** Forward to `TG-Home`

> **Important Note on Priorities:** AWS evaluates rules from top to bottom based on priority numbers (1, 2, 3, etc.). Your `/aws`, `/azure`, and `/gcp` rules must have higher priority (lower numbers) than the default rule. The default rule is always evaluated last.
