


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

<u>process to crete the ALB and TG with the Rule</u>

## Phase 1: Create the 4 Target Groups and Register EC2 Instances

### Step 1: Go to the Target Groups page

1. Log in to the AWS Management Console. In the top search bar, type **EC2** and open the EC2 console.
2. In the left navigation pane, scroll down to the **Load Balancing** section and click **Target Groups**.
3. Click **Create target group** (top right).

### Step 2: Create the Home target group (`TG-Home`)

1. **Choose a target type**: Select **Instances**.
2. **Target group name**: Enter `TG-Home`.
3. **Protocol**: Select **HTTP**.
4. **Port**: Enter `80`.
5. **VPC**: Select the same VPC as your 4 EC2 instances.
6. **Protocol version**: Keep default **HTTP1**.
7. **Health checks**:

   * Health check path: Enter `/` (the home app serves from the root path).
   * Leave advanced health check settings as default.
8. Click **Next**.
9. On the **Register targets** page:

   * In the instance list, check **Instance 1 (Home page server)**.
   * Keep port as `80`.
   * Click **Include as pending below**.
10. Click **Create target group**.

### Step 3: Create the AWS target group (`TG-AWS`)

Repeat the same steps as Step 2, but use these values:

| **Setting**       | **Value**                                             |
| ----------------- | ----------------------------------------------------- |
| Target group name | `TG-AWS`                                              |
| Protocol          | HTTP                                                  |
| Port              | 80                                                    |
| VPC               | Same VPC                                              |
| Health check path | `/aws` (or `/aws.html`, depending on your file setup) |
| Register target   | **Instance 2 (AWS app server)**                       |

Click **Create target group**.

### Step 4: Create the Azure target group (`TG-Azure`)

Repeat with:

| **Setting**       | **Value**                         |
| ----------------- | --------------------------------- |
| Target group name | `TG-Azure`                        |
| Protocol          | HTTP                              |
| Port              | 80                                |
| VPC               | Same VPC                          |
| Health check path | `/azure` (or `/azure.html`)       |
| Register target   | **Instance 3 (Azure app server)** |

Click **Create target group**.

### Step 5: Create the GCP target group (`TG-GCP`)

Repeat with:

| **Setting**       | **Value**                       |
| ----------------- | ------------------------------- |
| Target group name | `TG-GCP`                        |
| Protocol          | HTTP                            |
| Port              | 80                              |
| VPC               | Same VPC                        |
| Health check path | `/gcp` (or `/gcp.html`)         |
| Register target   | **Instance 4 (GCP app server)** |

Click **Create target group**.

### Step 6: Verify all target groups are healthy

1. In the Target Groups list, click each target group.
2. Click the **Targets** tab.
3. Confirm each instance shows **Status: healthy**. If it shows `initial` or `unhealthy`, check that the health check path is correct and that the EC2 security group allows HTTP (port 80) traffic from the ALB security group. Wait for health checks to pass.

---

## Phase 2: Add Path Routing Rules to the ALB Listener

### Step 7: Open the ALB listener page

1. In the EC2 console left navigation, click **Load Balancers**.
2. Find your ALB in the list and click its name.
3. Click the **Listeners** tab.
4. Locate the **HTTP:80** listener (the one you created with the ALB).
5. Click **View/edit rules** on the right side of that listener.

You will see:

* **Default rule**: Should currently forward to `TG-Home` (the default action you set when creating the ALB).
* Any existing rules below (if any).

### Step 8: Add the AWS path rule (priority 1)

1. Click **Add rule** (top right or above the rule list).
2. Under **Name and tags**:

   * Name: Enter `rule-aws` (optional but recommended for management).
3. Under **IF (conditions)**:

   * Click **Add condition** → select **Path**.
   * In the path pattern input box, enter: `/aws` and `/aws/*`

     * Type `/aws`, press Enter. Then type `/aws/*`, press Enter.
     * Add both patterns to match `/aws` and `/aws/anything`.
4. Under **THEN (actions)**:

   * Confirm the action is **Forward to**.
   * From the dropdown, select **`TG-AWS`**.
5. Click **Save** (bottom or top right).

> **Priority note**: AWS evaluates rules from lowest priority number to highest. New rules get an automatic priority number (usually 1, 2, 3...). Ensure your path rules have lower priority numbers than the default rule (the default rule is always evaluated last).

### Step 9: Add the Azure path rule (priority 2)

1. Click **Add rule** again.
2. **Name**: Enter `rule-azure`.
3. **IF (conditions)**:

   * Select **Path**.
   * Enter `/azure` and `/azure/*` (press Enter after each).
4. **THEN (actions)**:

   * Select **Forward to** → **`TG-Azure`**.
5. Click **Save**.

### Step 10: Add the GCP path rule (priority 3)

1. Click **Add rule** again.
2. **Name**: Enter `rule-gcp`.
3. **IF (conditions)**:

   * Select **Path**.
   * Enter `/gcp` and `/gcp/*`.
4. **THEN (actions)**:

   * Select **Forward to** → **`TG-GCP`**.
5. Click **Save**.

### Step 11: Confirm the default rule

1. In the rule list, find the **Default rule** (labeled "Default" or "默认").
2. Confirm its action is **Forward to `TG-Home`**.
3. The default rule needs no conditions; it triggers when no path rules match.

> **If the default rule is not what you want**: Click **Edit** next to the default rule, change the forward target to `TG-Home`, and save.

---

## Rule Priority Summary Table

After saving, your listener rules should look like this (ordered from highest to lowest priority):

| **Priority**   | **Condition**                  | **Action**            | **Description** |
| -------------- | ------------------------------ | --------------------- | --------------- |
| 1              | Path is `/aws` or `/aws/*`     | Forward to `TG-AWS`   | AWS app         |
| 2              | Path is `/azure` or `/azure/*` | Forward to `TG-Azure` | Azure app       |
| 3              | Path is `/gcp` or `/gcp/*`     | Forward to `TG-GCP`   | GCP app         |
| Default (last) | None (catch-all)               | Forward to `TG-Home`  | Home page       |


