# Customer Ticket Support System (CTSS)
## Complete Web Deployment Guide

This guide provides a comprehensive, step-by-step walkthrough for deploying the **Customer Ticket Support System (CTSS)** on production cloud infrastructure:
- **Database**: MongoDB Cloud (MongoDB Atlas Free M0 Cluster)
- **Backend API**: Render (Web Service running Node.js & Express)
- **Frontend Client**: Netlify (Global CDN hosting Vite & React SPA)

---

## Table of Contents
1. [Summary of Codebase Changes Made for Deployment](#1-summary-of-codebase-changes-made-for-deployment)
2. [Deployment Architecture](#2-deployment-architecture)
3. [Step 1: Set Up MongoDB Cloud (MongoDB Atlas)](#step-1-set-up-mongodb-cloud-mongodb-atlas)
4. [Step 2: Push Project Codebase to GitHub](#step-2-push-project-codebase-to-github)
5. [Step 3: Deploy Backend on Render](#step-3-deploy-backend-on-render)
6. [Step 4: Deploy Frontend on Netlify](#step-4-deploy-frontend-on-netlify)
7. [Step 5: Post-Deployment Verification & Seeding](#step-5-post-deployment-verification--seeding)
8. [Troubleshooting & Pro Tips](#troubleshooting--pro-tips)

---

## 1. Summary of Codebase Changes Made for Deployment

The following optimizations and adjustments have already been implemented directly in your project files to guarantee zero configuration errors in the cloud:

| File Modified / Added | Purpose & Change Details |
| :--- | :--- |
| `client/src/api/axiosInstance.js` | Updated `baseURL` from hardcoded `'/api'` to `import.meta.env.VITE_API_URL || '/api'`. This allows the client to communicate directly with your live Render backend URL in production while seamlessly falling back to local proxy in development. |
| `client/public/_redirects` | Created Netlify Single-Page Application (SPA) redirect rule: `/*    /index.html   200`. Prevents 404 errors when refreshing dynamic routes (e.g., `/customer/dashboard`, `/admin/tickets/:id`). |
| `client/.env.example` | Added environment variable reference file documenting `VITE_API_URL=https://your-backend.onrender.com/api`. |
| `server/server.js` | Verified dynamic port binding (`process.env.PORT || 5000`) required by Render's runtime container environment. |

---

## 2. Deployment Architecture

```
                       +-------------------------------+
                       |        Client Browser         |
                       +---------------+---------------+
                                       |
                       +---------------+---------------+
                       |                               |
       (HTML / Assets) |                               | (REST API Calls)
                       v                               v
             +-------------------+           +-------------------+
             |      Netlify      |           |      Render       |
             |  (React Frontend) |           | (Express Backend) |
             +-------------------+           +---------+---------+
                                                       |
                                                       | (Mongoose ODM)
                                                       v
                                             +-------------------+
                                             |   MongoDB Atlas   |
                                             |  (Cloud Database) |
                                             +-------------------+
```

---

## Step 1: Set Up MongoDB Cloud (MongoDB Atlas)

### 1.1 Create Account & Cluster
1. Navigate to [https://www.mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas) and sign in or create a free account.
2. In your Atlas dashboard, click **"Create"** or **"Build a Database"**.
3. Choose the **M0 Shared (Free Forever)** tier:
   - Cloud Provider: **AWS** or **Google Cloud**.
   - Region: Select the region geographically closest to your target users (e.g., Mumbai, Singapore, Frankfurt, N. Virginia).
   - Cluster Name: `Cluster0` (or leave default).
4. Click **"Create Deployment"**.

### 1.2 Create Database User Credentials
1. Under **"Quickstart"** / **"Security Quickstart"**:
   - Authentication method: **Username and Password**.
   - Username: e.g., `ctss_admin`
   - Password: Click **"Autogenerate Secure Password"** or enter a strong password (e.g., `CtssCloudPass2026!`).
   - **Important**: Copy this password and keep it safe—you will need it in your connection string.
2. Click **"Create Database User"**.

### 1.3 Configure Network Access (IP Whitelist)
1. On the same screen under **"Where would you like to connect from?"**:
   - Select **"Cloud Environment"** or **"Allow Access from Anywhere"**.
   - IP Address: Enter `0.0.0.0/0`.
   - Description: `Allow all (Required for Render dynamic IPs)`.
2. Click **"Add Entry"** / **"Finish and Close"**.

### 1.4 Copy Connection String
1. In your Atlas cluster dashboard, click the **"Connect"** button on your cluster.
2. Under "Connect to your application", select **"Drivers"** (Node.js).
3. Copy the SRV connection string:
   ```text
   mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
4. Replace `<username>` and `<password>` with your actual database user credentials.
5. Append `/ctss_db` right before the `?` query parameter to name your database:
   ```text
   mongodb+srv://ctss_admin:CtssCloudPass2026!@cluster0.xxxxx.mongodb.net/ctss_db?retryWrites=true&w=majority
   ```

---

## Step 2: Push Project Codebase to GitHub

Both Render and Netlify connect directly to your GitHub repository for automated continuous deployment (CI/CD).

1. Open your terminal in `c:\Users\mishr\Desktop\CTSS-MERN`.
2. Verify all files are committed to Git:
   ```bash
   git status
   ```
3. Stage and commit all changes:
   ```bash
   git add .
   git commit -m "Configure production deployment for Render and Netlify"
   ```
4. Create a new repository on [GitHub](https://github.com/new) (e.g., `CTSS-MERN`).
5. Link your local repository and push:
   ```bash
   git branch -M main
   git remote add origin https://github.com/<your-username>/CTSS-MERN.git
   git push -u origin main
   ```

---

## Step 3: Deploy Backend on Render

Render will host the Node.js / Express backend as a managed Web Service.

### 3.1 Create Web Service
1. Log in to [https://render.com](https://render.com).
2. Click **"New +"** in the top navigation bar and select **"Web Service"**.
3. Select **"Build and deploy from a Git repository"** and click **Next**.
4. Connect your GitHub account and choose your `CTSS-MERN` repository.

### 3.2 Configure Service Settings
Fill in the deployment settings exactly as specified below:

| Configuration Field | Value | Explanation |
| :--- | :--- | :--- |
| **Name** | `ctss-backend` (or your preferred name) | Determines your Render sub-domain. |
| **Region** | Select closest region (e.g., Singapore, Frankfurt, Oregon) | Match your MongoDB Atlas region for lowest latency. |
| **Branch** | `main` | Production branch. |
| **Root Directory** | `server` | **Critical!** Informs Render that the backend code resides in the `server` subdirectory. |
| **Runtime** | `Node` | Node.js execution environment. |
| **Build Command** | `npm install` | Installs server production dependencies. |
| **Start Command** | `npm start` | Runs `node server.js`. |
| **Instance Type** | `Free` | Free Tier ($0/month). |

### 3.3 Add Environment Variables
Scroll down to the **"Environment Variables"** section on the same configuration page and click **"Add Environment Variable"** for each of the following:

| Key | Value | Description |
| :--- | :--- | :--- |
| `NODE_ENV` | `production` | Enables production optimizations. |
| `MONGODB_URI` | `mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/ctss_db?retryWrites=true&w=majority` | Your MongoDB Atlas connection string from Step 1.4. |
| `JWT_SECRET` | `ctss_production_secure_token_secret_key_2026_xyz` | Strong random secret string used to sign JWTs. |
| `CLIENT_URL` | `*` (Update after Step 4 with your Netlify URL) | Allowed CORS origin. |

### 3.4 Deploy & Verify
1. Click **"Deploy Web Service"**.
2. Wait 2–3 minutes for Render to install dependencies and boot the server.
3. Once the status shows **"Live"**, your backend URL will be displayed at the top (e.g., `https://ctss-backend.onrender.com`).
4. **Test Health Endpoint**:
   Open a browser tab and navigate to:
   ```text
   https://ctss-backend.onrender.com/api/health
   ```
   You should receive a JSON response:
   ```json
   {"status":"ok","timestamp":"2026-11-20T..."}
   ```
   *Copy your Render URL—you will provide it to Netlify in the next step.*

---

## Step 4: Deploy Frontend on Netlify

Netlify will host the compiled React + Vite frontend as a globally distributed static website.

### 4.1 Import Project
1. Log in to [https://app.netlify.com](https://app.netlify.com).
2. Click **"Add new site"** -> **"Import an existing project"**.
3. Select **GitHub** as your Git provider and authorize Netlify.
4. Select your `CTSS-MERN` repository from the list.

### 4.2 Configure Build Settings
Fill in the configuration fields:

| Configuration Field | Value | Explanation |
| :--- | :--- | :--- |
| **Base directory** | `client` | **Critical!** Tells Netlify the frontend code is in `client`. |
| **Build command** | `npm run build` | Runs Vite production bundler (`vite build`). |
| **Publish directory** | `dist` | The folder Vite outputs production files into. |

### 4.3 Configure Environment Variables
1. On the same setup screen, click **"Add environment variables"** (or go to **Site configuration > Environment variables**).
2. Add the following variable:
   - **Key**: `VITE_API_URL`
   - **Value**: `https://ctss-backend.onrender.com/api`
   *(Replace with your actual Render URL from Step 3.4, ensuring you include the `/api` suffix)*.

### 4.4 Deploy
1. Click **"Deploy CTSS-MERN"** (or **"Deploy Site"**).
2. Netlify will build the client in approximately 1 minute.
3. Once finished, Netlify provides a live public URL (e.g., `https://peaceful-haven-12345.netlify.app`).
4. (Optional) In **Site configuration > Change site name**, you can customize the domain to something cleaner like `ctss-support-portal.netlify.app`.

---

## Step 5: Post-Deployment Verification & Seeding

### 5.1 Seed the Cloud MongoDB Atlas Database
To populate your live cloud database with the default demo accounts (`Admin`, `Agent`, `Customer`) and realistic tickets:

1. Open your local terminal in `c:\Users\mishr\Desktop\CTSS-MERN\server`.
2. Temporarily update your local `.env` file or run the seed command pointing directly to your Atlas connection URI:
   ```powershell
   $env:MONGODB_URI="mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/ctss_db?retryWrites=true&w=majority"
   npm run seed
   ```
3. The terminal will log:
   ```text
   Connected to MongoDB for seeding
   Seeding completed successfully!
   Accounts created:
    - Admin: admin@ctss.com / AdminPassword123
    - Agent: sarah.agent@ctss.com / AgentPassword123
    - Customer: customer@ctss.com / CustomerPassword123
   ```
4. Reset your local `server/.env` back to local MongoDB (`mongodb://127.0.0.1:27017/ctss_db`) if you plan to continue local development.

### 5.2 Update Render CORS (Best Security Practice)
1. Go back to your [Render Dashboard](https://dashboard.render.com).
2. Select your `ctss-backend` service -> **Environment**.
3. Update the `CLIENT_URL` variable to your live Netlify domain:
   ```text
   CLIENT_URL=https://ctss-support-portal.netlify.app
   ```
4. Render will automatically redeploy the backend with the updated CORS rule in 30 seconds.

### 5.3 Verify Live Workflows
1. Open your live Netlify frontend URL in your browser.
2. **Test One-Click Login**:
   - Click the **"Admin"** button on the login screen -> Sign In -> You should immediately land on the **Support Operations Console** (`/admin/dashboard`).
   - Sign out, then click **"Customer"** -> Sign In -> You should land on the **Customer Workspace** (`/customer/dashboard`).
3. **Test Ticket Submission**:
   - As a Customer, submit a new ticket (e.g., "Cloud Production Verification Test").
   - Confirm it appears in your ticket list.
4. **Test Browser Refresh on Subroutes**:
   - While on `/customer/tickets` or `/admin/tickets`, refresh the browser (<kbd>F5</kbd> or <kbd>Ctrl+R</kbd>).
   - The page should reload smoothly without a 404 error (verifying `_redirects` functionality).

---

## 8. Troubleshooting & Pro Tips

### 1. Render Free Tier "Cold Start" Delay
- **Symptom**: The first API request after 15 minutes of inactivity takes 40–50 seconds to respond.
- **Cause**: Render's free tier spins down idle web services to preserve resources.
- **Solution**: This is normal behavior on free hosting. Once the service receives an initial request, it wakes up and subsequent requests respond in under 150ms. For instant responsiveness 24/7 without cold starts, you can upgrade to Render's Starter plan ($7/mo) or use a free uptime monitor (e.g., [UptimeRobot](https://uptimerobot.com)) to ping your `/api/health` endpoint every 10 minutes.

### 2. MongoDB "MongoServerSelectionError" or IP Whitelist Issues
- **Symptom**: Render build succeeds, but logs report `MongoServerSelectionError: connection timed out` or server crashes on startup.
- **Cause**: MongoDB Atlas Network Access is blocking Render's IP address.
- **Solution**: In MongoDB Atlas, go to **Network Access > Add IP Address > Allow Access from Anywhere (`0.0.0.0/0`)**. Render uses dynamic IP ranges that change on each container restart, making `0.0.0.0/0` mandatory.

### 3. Netlify 404 Not Found on Page Reload
- **Symptom**: Navigating to `https://yoursite.netlify.app` works, but refreshing on `/admin/dashboard` returns Netlify 404.
- **Solution**: Ensure `client/public/_redirects` exists with `/*    /index.html   200`. This file is already configured in your repository and copied into `dist/` upon building.

### 4. CORS Blocked Errors
- **Symptom**: Browser console logs `Access to XMLHttpRequest has been blocked by CORS policy`.
- **Solution**: Verify that your Render backend environment variable `CLIENT_URL` matches your exact Netlify URL (including `https://` without a trailing slash), or set `CLIENT_URL=*`.

---

## Quick Reference Summary

| Service | Setting | Recommended Value |
| :--- | :--- | :--- |
| **MongoDB Atlas** | Network Access | `0.0.0.0/0` (Allow from anywhere) |
| **MongoDB Atlas** | Database Name | `ctss_db` |
| **Render** | Root Directory | `server` |
| **Render** | Build Command | `npm install` |
| **Render** | Start Command | `npm start` |
| **Render** | Environment Variables | `NODE_ENV=production`, `MONGODB_URI=...`, `JWT_SECRET=...` |
| **Netlify** | Base Directory | `client` |
| **Netlify** | Build Command | `npm run build` |
| **Netlify** | Publish Directory | `dist` |
| **Netlify** | Environment Variable | `VITE_API_URL=https://<your-render-name>.onrender.com/api` |
