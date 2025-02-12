# 🚀 Node.js Deployment to cPanel via GitHub Actions

This guide explains how to deploy a Node.js application from a **private GitHub repository** to **cPanel** using **GitHub Actions** and FTP.

---

## 📌 Prerequisites

Before you begin, ensure you have:

- ✅ A **private GitHub repository** with your Node.js app
- ✅ Access to **cPanel** with Node.js support
- ✅ An **FTP account** created in cPanel

---

## 🛠️ Deployment Steps

### 1️⃣ Prepare a Private GitHub Repository
Create a new private repository on GitHub and push your Node.js project.

### 2️⃣ Create a Node.js App on cPanel
- Log in to **cPanel** and go to **Setup Node.js App**.
- Click **Create Application** and configure:
    - **Application Root**: e.g., `nodejsapp`
    - **Application URL**: Your app's URL
    - **Application Startup File**: e.g., `server.js`
- Click **Create**.

### 3️⃣ Test if the Node.js App is Working
- Open your application URL in a browser.
- Check for errors and debug if necessary.

### 4️⃣ Install `cnad` for Deployment Automation
Add `cnad` to your project:

```sh
npm install cnad