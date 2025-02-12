# 🚀 Next.js Deployment to cPanel via GitHub Actions

This guide explains how to deploy a Node.js application from a **private GitHub repository** to **cPanel** using **GitHub Actions** and FTP.

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