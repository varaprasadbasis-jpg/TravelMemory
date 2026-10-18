# TravelMemory — Detailed AWS Deployment Architecture
  
```text
[User Browser]
       │
       ▼ (HTTP:80)
┌──────────────┐
│ Frontend ALB │
└──────┬───────┘
       ├───────────────────────┐
       ▼                       ▼
┌──────────────┐        ┌──────────────┐
│ Frontend EC2 │        │ Frontend EC2 │
│  (Master)    │        │   (Node-2)   │
└──────┬───────┘        └──────┬───────┘
       │ (API Requests)        │
       └───────────┬───────────┘
                   ▼
            ┌─────────────┐
            │ Backend ALB │
            └──────┬──────┘
       ┌───────────┴───────────┐
       ▼                       ▼
┌──────────────┐        ┌──────────────┐
│ Backend EC2  │        │ Backend EC2  │
│  (Master)    │        │   (Node-2)   │
└──────┬───────┘        └──────┬───────┘
       │ (:3000)               │ (:3000)
       └───────────┬───────────┘
                   ▼
        ┌────────────────────┐
        │   MongoDB Atlas    │
        │  (Cloud Database)  │
        └────────────────────┘
```

This repository documents the production-grade, highly-available deployment of **TravelMemory**, a full-stack MERN application. The environment is hosted on **AWS EC2 instances running Ubuntu** configured across two load-balanced tiers (Frontend and Backend) with automated failover and native connection pooling to a cloud-hosted database.

  
 Configuration & Deployment 

Step 1: Database Setup (MongoDB Atlas)
1. Sign in to your console at [://mongodb.com](https://://mongodb.com).
2. Create a new organization project named `travelmemory-bsvprasad`.
3. Provision a **Free Sandbox M0 Cluster** hosted on AWS in your local region (e.g., `ap-south-1` Mumbai).
4. Add a new dedicated Database User with a secure password under Database Access.
5. Navigate to Network Access -> click Add IP Address-> type `0.0.0.0/0` to ensure elastic EC2 nodes can scale and connect without interruption.
6. Verify access by connecting locally via MongoDB Compass using your cluster connection URI:
   `mongodb+srv://<user>:<password>@travelmemory-cluster.uztt61j.mongodb.net/`

Step 2: Establish Master Infrastructure Templates
Launch two base Ubuntu EC2 instances (Type: t3.micro) within your active AWS console to serve as base master images:
  backend-master
  frontend-master

Ensure your attached AWS Security Groups permit inbound traffic rules for **SSH (Port 22)** and **HTTP (Port 80)**.

 Step 3: Configure Backend Master
1. Access the `backend-master` instance via SSH or EC2 Instance Connect.
2. Initialize the package registry and install Node.js dependencies:
  bash
   sudo apt update
   sudo apt install nodejs npm git -y
  
3. Clone the source repository to the user directory:
  bash
   git clone https://github.com
   cd TravelMemory/backend
   npm install
  
  4. Construct your production environment configuration:
  bash
   nano .env
  
   Populate with your custom cluster configuration variables:
  env
   MONGO_URI=mongodb+srv://varaprasadbasis_db_user:5WYueKvIDisMM1eB@travelmemory-cluster.uztt61j.mongodb.net/travelmemory
   PORT=3000
  
5. Deploy a automated daemon service manager file to maintain high availability:
  bash
   sudo nano /etc/systemd/system/travelmemory-backend.service
  
   Add the following unit configuration data:
  ini
   [Unit]
   Description=TravelMemory Backend API
   After=network.target

   [Service]
   Type=simple
   User=ubuntu
   WorkingDirectory=/home/ubuntu/TravelMemory/backend
   EnvironmentFile=/home/ubuntu/TravelMemory/backend/.env
   Environment=NODE_ENV=production
   ExecStart=/usr/bin/npm start
   Restart=always
   RestartSec=5

   [Install]
   WantedBy=multi-user.target
  
6. Start and verify the application background service process:
   bash
   sudo systemctl daemon-reload
   sudo systemctl enable travelmemory-backend
   sudo systemctl start travelmemory-backend
   sudo systemctl status travelmemory-backend
   ```

Step 4: Configure Frontend Master
1. Access the `frontend-master` server tier via SSH client terminals.
2. Update global packages and fetch Nginx utilities:
   bash
   sudo apt update
   sudo apt install nodejs npm nginx git -y
   
3. Move into the application assets structure:
   bash
   git clone https://github.com
   cd TravelMemory/frontend
   
4. Map the API location configurations using an initial connection check:
   bash
   nano src/url.js
   
   Change the baseline structure pointer configuration to point to the temporary master engine IP:
   javascript
   export const baseUrl = "http://<YOUR_BACKEND_MASTER_IP>:3000";
   
5. Compile production code structures and align with Nginx virtual directory requirements:
   bash
   npm install
   npm run build
   sudo mkdir -p /var/www/html
   sudo cp -r build/* /var/www/html/
   
6. Start the web proxy infrastructure daemon:
   bash
   sudo systemctl enable nginx
   sudo systemctl restart nginx
   

---

Scaling & Redundant Load Balancing Tiers

Step 5: Generate Production AMIs
Once connectivity validation succeeds on your browser profile tests, clean down your master node states and create an immutable template from the AWS EC2 Actions dropdown:
1. Select `backend-master` -> Image and templates -> Create image (Name: `travelmemory-backend-ami`).
2. Select `frontend-master` -> Image and templates -> Create image (Name: `travelmemory-frontend-ami`).

Step 6: Provision Scale Pair Members
Using your newly built custom AMI templates, launch your secondary architecture pair matching identical regional parameters:
*   Launch `backend2` out of the custom backend AMI configuration.
*   Launch `frontend2` out of the custom frontend AMI configuration.

Step 7: Form Target Groups & Deploy Application Load Balancers (ALBs)
1. Create Backend Target Group: Target type: `Instances` | Protocol: `HTTP` | Port: `3000` | Health Check Path: `/hello`. Register `backend-master` and `backend2`.
2. Create Frontend Target Group: Target type: `Instances` | Protocol: `HTTP` | Port: `80` | Health Check Path: `/`. Register `frontend-master` and `frontend2`.
3. Navigate to **EC2 -> Load Balancers -> Create Load Balancer**. Setup both an internet-facing `travelmemory-frontend-alb` and an internal routing structure named `travelmemory-backend-alb`.

Step 8: Final Network Target Remapping
Because the target backend load balancing endpoint DNS is computed fluidly after creating your initial disk images, you must perform a final configuration update on the active front-end fleet:
1. Access both active frontend server instances (`frontend-master` & `frontend2`) over terminal connection arrays.
2. Open the URL route definition manifest file:
   bash
   nano src/url.js
  
3. Re-point the connection target configuration to the fully-qualified DNS record provided by your active AWS Backend ALB:
   javascript
   export const baseUrl = "http://amazonaws.com";
   
4. Rebuild the optimized application distribution blocks, clear the runtime folder contents, and re-trigger the Nginx service wrapper:
   bash
   npm run build
   sudo rm -rf /var/www/html/*
   sudo cp -r build/* /var/www/html/
   sudo systemctl restart nginx

   Please try with frontendapplication loadbaler url
   http://http://ravelmemory-frontend-alb-340298857.ap-southeast-2.elb.amazonaws.com/addexperience

   
