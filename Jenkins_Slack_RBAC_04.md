#  How to Integrate Jenkins with Slack

## 🔹 Step 1: Login to Slack
- Visit: https://slack.com  --> Sign in or create your workspace.

## 🔹 Step 2: Create a Slack Channel
- Go to Channels → Create Channel  
  Name it (e.g., #jio)  
  Set as Public  
  Click Create  

## 🔹 Step 3: Invite Team Members
- Go to your Slack workspace (e.g., KK FUNDA)  
  Click: "Invite people to KK FUNDA"  
  Copy invite link  

## 🔹 Step 4: Add Jenkins CI App to Slack
- In Slack, go to Tools and Settings → Manage Apps  
- Search for: Jenkins CI  
- Click "Add to Slack"  
- Choose a channel (e.g., #jio)  
- Click "Add Jenkins CI Integration"  

## 🔹 Step 5: Install Slack Plugin in Jenkins
- In Jenkins:  
  Go to Manage Jenkins → Plugin Manager  
  Install: Slack Notification Plugin  

## 🔹 Step 6: Configure Slack in Jenkins
- Go to: Manage Jenkins → Configure System  

- Workspace: kkfunda  

- Credentials:  
  Click Add  
  Choose Secret Text  
  Secret: ********* 
  Add a description (e.g., Slack Token)  

- Default channel: #jio  
- Click Test Connection  
- Click Save  

## 🔹 Step 7: Configure Job to Send Slack Notifications
- Go to a Jenkins Job → Configure  
- Scroll to Post-build Actions  
- Select: Slack Notifications  

- Click Advanced and:  
  Customize notification settings (Start, Success, Failure, etc.)  
  Optionally set a custom channel (e.g., #builds)  

- Click Save  

---

# RBAC (Role-Based Access Control) in Jenkins

- RBAC (Role-Based Access Control) is a critical security feature for managing permissions and access across Jenkins.  
- Implementing RBAC ensures that each team member has the right level of access based on their role, which helps in both security and efficiency.  
- Without RBAC, Jenkins by default grants admin access to all users, which is risky. This means anyone can modify system settings, delete jobs, or access sensitive configurations.  

## User Information Storage
Jenkins stores user account details in the **/users/** directory inside the Jenkins home folder  
(e.g., `/var/lib/jenkins/users/`).  

Each user has their own folder containing configuration data like **config.xml**.

## Key Features of RBAC in Jenkins (Real-World Use Cases)

### 1. Granular Access Control
RBAC in Jenkins, you can assign specific roles to different users, limiting their access to only what they need.Reduces mistakes and improves security.

### 2. Group-Based Management
RBAC allows you to create groups based on roles (e.g., Developers, Admins, QA, Testers) and assign permissions to each group. This makes it much easier to manage access control, especially in large teams.

### 3. Restrict Job Access
RBAC lets you control who can trigger builds, configure pipelines, or manage credentials. This is crucial in preventing unauthorized actions, such as someone accidentally running a job or altering a critical production pipeline.

---

# Setting Up RBAC in Jenkins Using Project-based Matrix Authorization Strategy

## Step1: Install Required Plugin
- Go to **Manage Jenkins → Plugin Manager → Available**  
- Search and install: **Role-based Authorization Strategy plugin**

## Step2: Create Users in Jenkins
- Go to **Manage Jenkins → Manage Users**  
- Click **Create User**

- Fill out the form:  
  - Username  
  - Password  
  - Full name  
  - Email  

- Click **Create User**

## Step 3: Configure Global RBAC (Matrix Permissions)

1. Go to **Manage Jenkins → Configure Global Security**

2. By default, Jenkins sets:  
   **Authorization → Logged-in users can do anything**

   This means every authenticated user has full admin rights, which is not safe for real-world teams.

   Change this to:  
   **Project-based Matrix Authorization Strategy** under Authorization.

3. A permission matrix will appear — by default, **no user has any permissions**, so:

4. Add users or groups manually

Then check the boxes for permissions like:
- **Overall → Administer** (for full access)  
- **Job → Read, Build, Configure** (for pipeline control)  
- **Credentials → View, Create, Update** (if needed)

## RBAC in Jenkins: Example with User sai

### Global-Level Access
We created a user **sai** and gave only **Job → Read** permission globally.

When sai logs in:

## Job-Level RBAC in Jenkins (Example: jio-dev job)

After setting up global security, you can assign specific permissions at the job level:

1. Go to the job (**jio-dev**) → Click **Configure**  
2. Scroll down and enable **Enable project-based security**  
3. From the user list, add your user (e.g., **sai G**)

4. Grant permissions like:
- **Job → Read, Build, Configure**  
- **Run → Workspace**

## Now when sai logs in:
- He can view and manage only **jio-dev**  
- He won’t see any other jobs or settings  

This setup keeps your Jenkins access tightly controlled and job-specific.
