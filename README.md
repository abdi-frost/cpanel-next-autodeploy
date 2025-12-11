# 🚀 Next.js Auto Deployment to cPanel via GitHub Actions

This guide explains how to deploy a Node.js application from a **private GitHub repository** to **cPanel** using **GitHub Actions** and FTP.

Note that this is for dynamic nextjs application with server side function/pages and IS NOT FOR STATIC EXPORT

Original content from [SyntaNext YouTube Channel](https://www.youtube.com/@syntanext)

---

## 📌 Prerequisites

Before you begin, ensure you have:

- ✅ A **private GitHub repository** with your Node.js app
- ✅ Access to **cPanel** with compatible (for your nextjs version) Node.js support
- ✅ An **FTP account** created in cPanel

---

## 🛠️ Deployment Steps

### 1️⃣ Prepare a Private GitHub Repository
Create a new private repository on GitHub and push your Nextjs project.

### 2️⃣ Create a Node.js App on cPanel
- Log in to **cPanel** and go to **Setup Node.js App**.
- Click **Create Application** and configure:
    - **Application Root**: e.g., `your subdomain.files`
    - **Application URL**: Your app's URL
    - **Application Startup File**: e.g., `server.js`
- Click **Create**.

### 3️⃣ Test if the Node.js App is Working
- Open your application URL in a browser.
- Check for errors and debug if necessary.

### 4️⃣ Install `cnad` for Deployment Automation
Add `cnad` to your project:

```sh
npm install @bitc/cnad
```
### 5️⃣ Create a `server.js` File
Create a `server.js` file in the root of your project with the following sample code:

```javascript
//server.js
const { createServer } = require('http')
const { parse } = require('url')
const next = require('next')
const cnad = require("@bitc/cnad");

// replace with your files
cnad.config("/home/useranme/nodevenv/application-root/nodeversion");

// Make CNAD listen for other files to restart the server

const dev = process.env.NODE_ENV !== 'production'
const hostname = 'localhost'
const port = 3000
// when using middleware `hostname` and `port` must be provided below
const app = next({ dev, hostname, port })
const handle = app.getRequestHandler()

app.prepare().then(() => {

    cnad.start();

    createServer(async (req, res) => {
        try {
            // Be sure to pass `true` as the second argument to `url.parse`.
            // This tells it to parse the query portion of the URL.
            const parsedUrl = parse(req.url, true)
            const { pathname, query } = parsedUrl

            if (pathname === '/a') {
                await app.render(req, res, '/a', query)
            } else if (pathname === '/b') {
                await app.render(req, res, '/b', query)
            } else {
                await handle(req, res, parsedUrl)
            }
        } catch (err) {
            console.error('Error occurred handling', req.url, err)
            res.statusCode = 500
            res.end('internal server error')
        }
    })
        .once('error', (err) => {
            console.error(err)
            process.exit(1)
        })
        .listen(port, () => {
            console.log(`> Ready on http://${hostname}:${port}`)
        })
})
```
### 6️⃣ Add Start Script to `package.json`
Update your `package.json` file to include a start script that sets the `NODE_ENV` to production and starts your server:

```json
{
  "scripts": {
    "start": "NODE_ENV=production node server.js"
  }
}
```
### 7️⃣ Create GitHub Actions Workflow
Create a `.github/workflows/deploy.yml` file in the root of your project with the following content:

**Dont forget to remove the comments from yml file**

```yaml
name: 🚀 CNAD

on:
  push:
    # name of your listener branch
    branches: [ "main" ] 

jobs:
  build:
    name: Build Nextjs app
    runs-on: ubuntu-latest

    steps:
      - name: Clone repository
        uses: actions/checkout@v4

        # replace with your nodejs version
      - name: Use Node.js 20.x 
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

        # this is where you inject env files for building
        # you can add here or in github secrets

      - name: Generate build
        run: npm run build

      - name: Upload .next folder
        uses: actions/upload-artifact@v4
        with:
          name: dot_next_folder
          path: .next/
          # added on upload-artifact v4 for hidden files
          include-hidden-files: true

  web-deploy:
    name: 🚀 Deploy
    runs-on: ubuntu-latest
    needs: [build]

    steps:
      - name: Clone repository
        uses: actions/checkout@v4

      - name: Creating restart file
        run: |
          mkdir tmp && touch tmp/restart.txt
          echo $RANDOM > tmp/restart.txt

      - name: Download .next folder
        uses: actions/download-artifact@v4
        with:
          name: dot_next_folder
          path: .next

      - name: Sync files
        uses: SamKirkland/FTP-Deploy-Action@4.3.2
        with:
          # define these in github repo setting secrets actions
          server: ${{ secrets.FTP_HOST }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          exclude: |
            **/.next/cache/**
            **/.github/**
            **/.git/**
            pages/**
            postcss.config.mjs
            tailwind.config.mjs
            README.md
            .gitignore
            .eslintrc.
```                
### 8️⃣ Push Changes to GitHub
Commit and push your changes to the `main` branch of your GitHub repository.

### 9️⃣ Monitor GitHub Actions
- Go to the **Actions** tab in your GitHub repository.
- Monitor the deployment workflow for any errors.

### 🔟 Verify Deployment in cPanel
- Log in to **cPanel** and navigate to the **File Manager**.
- Check if the files have been deployed correctly.

### 1️⃣1️⃣ Install Node Modules
Since this is the first time, you need to install the node modules on your server:
```sh
npm install
```

### 1️⃣2️⃣ Add Node Environment Variables in cPanel
- If you have any environment variables, add them in cPanel. If not, you can skip this step.

### 1️⃣3️⃣ Restart the Node.js Application
- Go to Setup Node.js App in cPanel.
- Restart your application.
### 1️⃣4️⃣ Make Changes and Test Auto Deployment
- Make changes to your code and push them to the main branch.
- Verify that the changes are automatically deployed by checking your application URL.
