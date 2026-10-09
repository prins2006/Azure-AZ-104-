# Azure Web Apps and Container Services — Topics 51–60

Detailed learning notes with examples, Azure Portal steps, commands, troubleshooting, production use cases, and interview questions.

## Learning roadmap

- **Topics 51–55: Azure Web Apps / App Service**
  - Making simple changes to a Web App
  - Adding a simple web application
  - Deployment slots
  - Lab: Deployment slots
  - Autoscaling for Web Apps
- **Topics 56–60: Containers and Azure Container Registry**
  - Container-based applications
  - Installing Docker on an Azure VM
  - Containerizing an application
  - Azure Container Registry (ACR)
  - Publishing images to ACR

---

# 51. Making Simple Changes to the Web App

## What is an Azure Web App?

Azure Web App is a managed service within Azure App Service that hosts websites and web applications without requiring you to manage the underlying operating system or web-server infrastructure directly.

Examples of supported application types include:

- PHP websites
- Node.js applications
- Python web applications
- ASP.NET Core applications
- Custom container-based applications

## Basic architecture

1. A user opens a website URL in a browser.
2. Azure App Service receives the HTTP/HTTPS request.
3. The web application processes the request.
4. The response is returned to the user's browser.

## Create a simple Web App in Azure Portal

1. Sign in to the [Azure Portal](https://portal.azure.com/).
2. Search for **App Services**.
3. Select **Create → Web App**.
4. Choose the subscription and resource group.
5. Enter a globally unique name, for example `prins-webapp-demo`.
6. Select a supported runtime stack and region.
7. Choose an appropriate pricing plan.
8. Select **Review + create → Create**.
9. Open the new App Service and select its default domain or **Browse**.

The URL typically looks like:

```text
https://YOUR-APP-NAME.azurewebsites.net
```

The exact URL depends on the name selected.

## Example HTML page

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Azure Web App</title>
</head>
<body>
    <h1>Welcome to My Azure Web App</h1>
    <p>This website is hosted on Microsoft Azure.</p>
</body>
</html>
```

To make a change, update the heading:

```html
<h1>Welcome to My DevOps Journey</h1>
```

Redeploy the updated files and refresh the website.

**Important:** Editing a local file does not automatically change the deployed website. Use a supported deployment method, such as GitHub Actions, ZIP deployment, or the deployment tools supported by your runtime. The exact method depends on whether the app is code-based or container-based.

## Real-world use case

A developer fixes a typo on the homepage, tests the change, and deploys the updated application.

---

# 52. Lab — Azure Web Apps: Adding a Simple Web Application

## Objective

Deploy a basic application to Azure App Service and verify that it is accessible through an Azure-generated URL.

## Lab procedure

1. Create an App Service with a supported runtime and suitable plan.
2. Prepare the application source files.
3. Deploy the files using a supported deployment method.
4. Open the app URL and confirm that the application loads.
5. Update the application and redeploy.
6. Confirm that the updated content appears.

## Example: Simple PHP application

Create `index.php`:

```php
<?php
$title = "Azure Web App";
$message = "My first PHP application on Azure";
?>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title><?php echo $title; ?></title>
</head>
<body>
    <h1><?php echo $message; ?></h1>
    <p>Deployed using Azure App Service.</p>
</body>
</html>
```

For a PHP application, select a PHP runtime supported by App Service on Linux and use a compatible deployment method.

## Expected result

The website displays:

- Azure Web App
- My first PHP application on Azure
- Deployed using Azure App Service

## Verification checklist

- [ ] App Service deployment completed.
- [ ] Website URL opens successfully.
- [ ] Application content appears correctly.
- [ ] Updated code appears after redeployment.
- [ ] No unexpected resource costs are accumulating.

## Troubleshooting

| Problem | What to check |
|---|---|
| Default welcome page appears | Verify the deployment package, entry point, and deployment logs |
| HTTP 500 error | Inspect application logs and runtime configuration |
| Site does not load | Check app status, deployment status, DNS, and networking |
| Changes are not visible | Confirm the correct app/slot was updated and redeployment completed |

---

# 53. Azure Web Apps — Deployment Slots

## What is a deployment slot?

A deployment slot is a separate live instance of an Azure Web App associated with the same App Service app. Slots have their own URLs and allow teams to test a new version before promoting it to production.

Common slots:

- **Production:** The live application.
- **Staging:** The next version under test.
- **Testing:** An environment for additional validation.

Example URLs:

| Slot | Example URL | Purpose |
|---|---|---|
| Production | `https://myapp.azurewebsites.net` | Current live version |
| Staging | `https://myapp-staging.azurewebsites.net` | Test the next release |

These URLs are illustrative.

## How slot swapping works

1. Deploy the new version to staging.
2. Test the application and verify configuration.
3. Start a swap from staging to production.
4. Azure applies the slot-swap rules, including warm-up and configuration behavior.
5. Verify the production application after the swap.

A swap does not simply merge two code folders. Settings marked as deployment-slot settings remain associated with their respective slots.

## Why use deployment slots?

- Test a release before production.
- Reduce deployment downtime.
- Validate startup behavior and configuration.
- Make it easier to return to a previous application version when appropriate.
- Separate release preparation from production traffic.

## Portal procedure

1. Open the App Service in Azure Portal.
2. Select **Deployment slots**.
3. Select **Add Slot**.
4. Enter a slot name such as `staging`.
5. If available, select the option to clone settings from production.
6. Create the slot and open its URL.
7. Deploy a different version to staging.
8. Test the application.
9. Select **Swap** and review the source and target.
10. Confirm the swap only after validation.

**Plan requirement:** Deployment slots generally require an App Service plan tier that supports them. Verify current plan capabilities before starting the lab.

## Real-world example

A company needs to release a new checkout page. Developers deploy it to staging, test login and payment flows, and then swap staging with production. If the release has a problem, they assess whether swapping back is appropriate.

---

# 54. Lab — Azure Web Apps: Deployment Slots

## Objective

Create a staging slot, deploy a test version, verify it, and swap it with production.

## Step 1: Open your Web App

Go to **Azure Portal → App Services → Your Web App**.

## Step 2: Create the staging slot

- Open **Deployment slots**.
- Select **Add**.
- Enter `staging`.
- Clone production settings if the option is available and appropriate.
- Create the slot.

## Step 3: Verify both environments

| Environment | Role |
|---|---|
| Production | Current live website |
| Staging | New version for testing |

## Step 4: Deploy different content

Deploy updated application content to the staging slot. For example:

```html
<h1>Version 2 - Staging Environment</h1>
```

Open the staging URL and verify the new heading. Confirm that production still displays the old version.

## Step 5: Swap staging with production

1. Return to **Deployment slots**.
2. Select **Swap**.
3. Choose staging as the source and production as the target.
4. Review the swap settings.
5. Start the swap and wait for completion.
6. Open the production URL and verify the new version.

## What if the new version fails?

If the previous version remains available in the other slot after the swap, swapping back may restore the earlier application version. First investigate configuration changes, database migrations, and external dependencies. A slot swap does not reverse database changes or external service changes.

## Production best practices

- Keep secrets and environment-specific configuration separate.
- Test startup, health checks, and application behavior in staging.
- Use deployment-slot settings for values that must remain with a specific slot.
- Do not assume that swapping application code reverses database schema changes.
- Monitor logs and availability after a release.

---

# 55. Autoscaling for Web Apps

## What is autoscaling?

Autoscaling adjusts the number of application instances or resources according to demand or a configured schedule.

For example, a website normally receives 100 requests per minute but receives 2,000 requests per minute during a sale. Autoscaling can add instances to help handle the increase.

## Types of scaling

| Type | Meaning | Example |
|---|---|---|
| Scale up / down (vertical) | Change the capacity or tier of a resource | Move to a larger App Service plan |
| Scale out / in (horizontal) | Increase or decrease the number of app instances | Increase from 2 instances to 5 |

## Example autoscale settings

| Setting | Example |
|---|---|
| Minimum instances | 2 |
| Default instances | 2 |
| Maximum instances | 5 |
| Scale-out threshold | CPU above 70% |
| Scale-in threshold | CPU below 30% for a sustained period |

These are example values, not universal recommendations. Production thresholds depend on workload, response times, memory use, and cost limits.

## How autoscaling works

1. Azure Monitor collects supported metrics.
2. Autoscale rules evaluate thresholds, schedules, and cooldown periods.
3. Azure changes the number of instances within configured limits.
4. The operator monitors performance and cost.

## Portal lab

1. Open the App Service.
2. Select **Scale out (App Service plan)** or the corresponding scale-out option.
3. Check the plan tier and supported scaling features.
4. Select **Custom autoscale** if available for the plan and configuration.
5. Define minimum, maximum, and default instance counts.
6. Create a scale-out rule for a suitable metric.
7. Create a scale-in rule for sustained low demand.
8. Save the settings and monitor the result.

The exact menu and available rules vary by App Service plan and configuration. Some plans offer only manual scaling.

## Interview scenario

**Question:** What would you do if a production website became slow during peak traffic?

**Answer:** I would inspect response times, CPU, memory, request volume, and application logs. If the bottleneck is instance capacity, I would configure autoscaling within appropriate limits, validate scale-out behavior, and monitor performance and cost. If the bottleneck is the database or an external API, adding web instances alone might not solve it.

**Important:** Autoscaling does not automatically fix inefficient code, database bottlenecks, or external dependency failures.

---

# 56. Container-Based Applications

## What is a container?

A container packages an application with its required libraries, runtime, and configuration so it can run consistently across environments.

For example, a Node.js app can be packaged with its Node runtime and dependencies instead of relying on a separately configured runtime on every server.

## Virtual machine vs container

| Feature | Virtual machine | Container |
|---|---|---|
| Isolation | Virtualized machine with its own OS | Isolated processes sharing the host kernel |
| Operating system | Guest OS per VM | Shares the host kernel |
| Startup | Usually slower | Usually faster |
| Resource usage | Generally higher | Generally lower |
| Packaging | VM image | Container image |
| Typical use | Full OS workloads | Portable application workloads |

Containers are lightweight, but they do not replace every VM workload. Isolation and security depend on configuration and the runtime.

## Important Docker terms

- **Image:** A read-only template used to create containers.
- **Container:** A running or stopped instance of an image.
- **Dockerfile:** Instructions for building an image.
- **Registry:** A service that stores and distributes images.
- **Volume:** Storage that can persist independently of a container's writable layer.
- **Port mapping:** Connects a host port to a container port.

## Container workflow

1. Write the application source code.
2. Create a Dockerfile.
3. Build a container image.
4. Run a container from that image.
5. Test the application.
6. Push the image to a registry.
7. Deploy the image to a target environment.

## Where can containers run in Azure?

- **Azure Virtual Machines:** Install Docker and manage the runtime yourself.
- **Azure Container Instances (ACI):** Run containers without managing a full VM as the container host.
- **Azure App Service:** Host web applications using supported custom container configurations.
- **Azure Kubernetes Service (AKS):** Orchestrate containers using managed Kubernetes.

Choose based on workload complexity, operational responsibility, scaling needs, and cost.

## Useful Docker commands

```bash
# Check Docker version
docker --version

# List local images
docker images

# List running containers
docker ps

# List all containers
docker ps -a

# Download an image
docker pull nginx:alpine

# Run a web server
docker run -d --name web -p 8080:80 nginx:alpine

# View logs
docker logs web

# Stop and remove the container
docker stop web
docker rm web
```

On the same machine, open `http://localhost:8080`. If Docker runs on a remote Azure VM, use the VM's accessible address and configure networking appropriately.

## Real-world use case

A DevOps engineer packages a web application into a Docker image, tests it locally, publishes it to a registry, and deploys the same image to test or production.

---

# 57. Lab — Setting up Docker on an Azure Virtual Machine

## Objective

Create or use an Azure Linux VM, install Docker Engine, run a container, and access the application remotely.

## Step 1: Connect to the VM

From your local Ubuntu terminal:

```bash
ssh -i ~/.ssh/your-private-key azureuser@YOUR_VM_PUBLIC_IP
```

Replace the key path, username, and IP with your actual values.

## Step 2: Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

## Step 3: Install Docker

For a quick lab on Ubuntu, install the distribution's Docker package:

```bash
sudo apt install -y docker.io
```

Start Docker and enable it at boot:

```bash
sudo systemctl enable --now docker
```

Verify:

```bash
docker --version
sudo systemctl status docker
```

For production, use Docker's official Ubuntu installation instructions to choose an appropriate supported Engine version and repository.

## Step 4: Run a test container

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  nginx:alpine
```

Check the container and logs:

```bash
sudo docker ps
sudo docker logs my-nginx
```

Test from the VM:

```bash
curl http://localhost:8080
```

You should receive HTML from Nginx.

## Step 5: Access the application from a browser

1. Open the VM in Azure Portal.
2. Navigate to **Networking → Network settings → Inbound security rules**.
3. Add an inbound TCP rule for port `8080`, restricted to your own public IP address for this lab.
4. Open:

```text
http://YOUR_VM_PUBLIC_IP:8080
```

**Security:** Avoid exposing test ports to the entire internet. For production, prefer HTTPS through a reverse proxy or load balancer and restrict network access.

## Step 6: Troubleshooting

| Problem | What to check |
|---|---|
| Docker command fails | `sudo systemctl status docker` |
| Port is unreachable | Docker port mapping, Azure NSG, host firewall |
| Container exits | `sudo docker ps -a` and `sudo docker logs my-nginx` |
| SSH fails | SSH key, username, VM networking and NSG |
| Port already in use | `sudo ss -lntp` |

## Cleanup

```bash
sudo docker stop my-nginx
sudo docker rm my-nginx
```

If you created a VM solely for the lab, stop it when finished. Deallocate or delete resources as appropriate to avoid unnecessary costs. Stopping a VM from inside the OS may not stop all compute charges.

---

# 58. Lab — Containerize an Application

## Objective

Create a simple application, write a Dockerfile, build an image, and run the application in a container.

This lab uses Node.js.

## Step 1: Create the project

```bash
mkdir -p ~/container-lab
cd ~/container-lab
```

Create `package.json`:

```json
{
  "name": "azure-container-lab",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^5.0.0"
  }
}
```

Create `app.js`:

```javascript
const express = require("express");

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
    res.send(`
        <h1>Hello from Docker!</h1>
        <p>My application is ready for Azure.</p>
    `);
});

app.get("/health", (req, res) => {
    res.status(200).json({ status: "healthy" });
});

app.listen(PORT, "0.0.0.0", () => {
    console.log(`Server running on port ${PORT}`);
});
```

**Why `0.0.0.0`?** It lets the app listen on the container's network interfaces instead of only its loopback interface.

## Step 2: Create the Dockerfile

Create a file named `Dockerfile`:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package.json ./

RUN npm install --omit=dev

COPY app.js ./

ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

CMD ["npm", "start"]
```

## Dockerfile explanation

| Instruction | Meaning |
|---|---|
| `FROM` | Selects the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies project files into the image |
| `RUN` | Executes a command during image building |
| `ENV` | Defines environment variables |
| `EXPOSE` | Documents the application's listening port |
| `CMD` | Specifies the default command when the container starts |

`EXPOSE 3000` does not publish the port by itself. Publish it with `-p` when starting the container.

## Step 3: Build the image

```bash
docker build -t prins-webapp:v1 .
```

Verify:

```bash
docker images
```

## Step 4: Run the container

```bash
docker run -d \
  --name prins-webapp \
  -p 3000:3000 \
  prins-webapp:v1
```

## Step 5: Test the application

```bash
curl http://localhost:3000
curl http://localhost:3000/health
```

Expected health response:

```json
{"status":"healthy"}
```

View logs:

```bash
docker logs prins-webapp
```

## Step 6: Rebuild after a code change

Update the HTML response in `app.js`, then run:

```bash
docker build -t prins-webapp:v2 .
docker stop prins-webapp
docker rm prins-webapp

docker run -d \
  --name prins-webapp \
  -p 3000:3000 \
  prins-webapp:v2
```

In a production workflow, build, test, tag, and publish an image in CI/CD, then deploy it through a controlled release process.

## Production improvements

- Run the application as a non-root user.
- Pin base images and dependencies to suitable versions.
- Add a `.dockerignore` file.
- Use a lockfile and `npm ci` for reproducible Node.js dependency installation.
- Do not embed credentials in the Dockerfile.
- Configure health checks and resource limits in the deployment environment.

---

# 59. Lab — Azure Container Registry Service

## What is Azure Container Registry (ACR)?

Azure Container Registry is a managed private registry service used to store, manage, and distribute container images.

You build an image, upload it to ACR, and authorized machines or Azure services can pull it for deployment.

## ACR workflow

1. Developer or CI pipeline builds and tests an image.
2. Image is pushed to Azure Container Registry.
3. An authorized deployment target pulls the image.
4. The image runs as a container.

## ACR terminology

| Term | Explanation | Example |
|---|---|---|
| Registry | Registry service | `containerprins` |
| Login server | Registry endpoint | `containerprins.azurecr.io` |
| Repository | A collection of image tags | `mysql` |
| Tag | Identifies an image version/reference | `8.4` |
| Image reference | Full registry/repository/tag path | `containerprins.azurecr.io/mysql:8.4` |

The registry name must be globally unique and follow Azure's naming rules.

## Step 1: Create ACR in Azure Portal

1. Sign in to the [Azure Portal](https://portal.azure.com/).
2. Search for **Container registries**.
3. Select **Create**.
4. Choose the subscription and resource group.
5. Enter a globally unique name such as `containerprins`.
6. Choose a region.
7. Select an appropriate SKU.
8. Select **Review + create → Create**.

Available features and limits depend on the selected SKU.

## Step 2: Find the login server

Open your registry and find **Login server** in the overview or properties.

Example:

```text
containerprins.azurecr.io
```

Do not confuse the registry name with the full login server.

## Step 3: Sign in from a machine with Azure CLI and Docker

Install Azure CLI if necessary, then sign in:

```bash
az login
```

Check the current subscription:

```bash
az account show
```

Select the intended subscription if you have more than one:

```bash
az account set --subscription "YOUR_SUBSCRIPTION_ID"
```

Sign in to ACR:

```bash
az acr login --name containerprins
```

You need appropriate permissions for the registry, and the relevant authentication and Docker tools must be available.

## Step 4: Verify the registry

```bash
az acr show \
  --name containerprins \
  --query loginServer \
  --output tsv
```

Expected output:

```text
containerprins.azurecr.io
```

**Production security:** Prefer Microsoft Entra ID authentication, managed identities, and appropriately scoped roles. Avoid enabling the registry admin user simply for convenience when identity-based authentication is available.

---

# 60. Lab — Publishing to Azure Container Registry

## Objective

Build an image, tag it with the ACR login server, push it to the registry, and verify that it uploaded successfully.

## Step 1: Check local images

```bash
docker images
```

Suppose the image built in Topic 58 is:

```text
prins-webapp   v1
```

You can use the same workflow for a MySQL image.

## Step 2: Authenticate to ACR

```bash
az login
az acr login --name containerprins
```

You need appropriate registry permissions.

## Step 3: Tag the image

For the Node.js example:

```bash
docker tag prins-webapp:v1 \
  containerprins.azurecr.io/prins-webapp:v1
```

For a MySQL example:

```bash
docker pull mysql:8.4

docker tag mysql:8.4 \
  containerprins.azurecr.io/mysql:8.4
```

The example tags an existing MySQL image; it does not build a custom MySQL image.

## Step 4: Verify the tag

```bash
docker images
```

You should see an image reference similar to:

```text
containerprins.azurecr.io/prins-webapp   v1
```

## Step 5: Push the image

For Node.js:

```bash
docker push containerprins.azurecr.io/prins-webapp:v1
```

For MySQL:

```bash
docker push containerprins.azurecr.io/mysql:8.4
```

Docker uploads the image layers to ACR.

## Step 6: Verify the image in Azure

List repositories:

```bash
az acr repository list \
  --name containerprins \
  --output table
```

List Node.js image tags:

```bash
az acr repository show-tags \
  --name containerprins \
  --repository prins-webapp \
  --output table
```

For MySQL, replace `prins-webapp` with `mysql`.

You can also open the registry in Azure Portal and inspect **Repositories**.

## Step 7: Pull the image from another machine

On a VM or Docker host that has access to the registry:

```bash
az login
az acr login --name containerprins

docker pull containerprins.azurecr.io/prins-webapp:v1
```

Run it:

```bash
docker run -d \
  --name web-from-acr \
  -p 3000:3000 \
  containerprins.azurecr.io/prins-webapp:v1
```

Test it:

```bash
curl http://localhost:3000
```

When running on an Azure VM, `localhost` refers to that VM.

## Common error: `tag does not exist`

Example:

```text
The push refers to repository [...]
tag does not exist
```

This means the exact local image tag used by the push command does not exist.

Check available images:

```bash
docker images
```

If the local image is `mysql:8.4`, tag it before pushing:

```bash
docker tag mysql:8.4 \
  containerprins.azurecr.io/mysql:latest
```

Then push:

```bash
docker push containerprins.azurecr.io/mysql:latest
```

Alternatively, preserve the version tag:

```bash
docker tag mysql:8.4 \
  containerprins.azurecr.io/mysql:8.4

docker push containerprins.azurecr.io/mysql:8.4
```

The source image name and tag must match an image that exists locally.

## Common ACR errors

| Error | Likely cause | Solution |
|---|---|---|
| `unauthorized` | Missing authentication or permissions | Sign in and check registry roles |
| `denied` | Insufficient push/pull permissions | Verify assigned roles and registry permissions mode |
| `tag does not exist` | Incorrect local image tag | Check `docker images` and run `docker tag` |
| DNS/network error | Connectivity or DNS issue | Check hostname, DNS, and network restrictions |
| Repository not found | Incorrect repository name or image not pushed | Verify the repository and push command |
| ACR login fails | Azure CLI or Docker authentication issue | Check `az login`, subscription, roles, and Docker |

## Recommended production workflow

1. Commit source code to Git.
2. CI pipeline builds and tests the image.
3. Push a versioned image to ACR.
4. Deploy that image to Azure.
5. Verify health, logs, and availability.

Use versioned tags such as `v1.0.0` or a commit SHA for release tracking. Avoid relying exclusively on `latest`, because it does not identify a unique release version.

---

# Revision and Interview Questions

## 1. What is Azure App Service?

A managed platform for hosting web applications and APIs without directly managing the underlying web-server infrastructure.

## 2. What is the purpose of a deployment slot?

It provides a separate environment for deploying and testing a new version before swapping it into production.

## 3. Does swapping slots roll back database changes?

No. A slot swap changes application versions and slot configuration according to Azure's swap rules. Database changes need their own migration and rollback strategy.

## 4. What is the difference between scale up and scale out?

- **Scale up:** Change the capacity or tier of a resource.
- **Scale out:** Change the number of application instances.

## 5. What is a Docker image?

A packaged, versioned template used to create containers.

## 6. What does `EXPOSE 3000` do?

It documents the container port. It does not publish that port to the host.

## 7. Why use Azure Container Registry?

To store and distribute private container images for authorized deployment targets.

## 8. What is the difference between a repository and a tag in ACR?

A repository groups image tags, while a tag identifies a particular version or reference within the repository.

## 9. Why must an image be tagged before pushing to ACR?

The image needs a destination reference containing the ACR login server, repository, and tag.

## 10. How do you troubleshoot a failed container?

Check `docker ps -a`, `docker logs`, port mappings, configuration, resource usage, and the container's exit status.

---

# Practical Tasks Checklist

- [ ] Create an Azure Web App and deploy a simple application.
- [ ] Update the application and verify the change.
- [ ] Create a staging deployment slot and test a separate version.
- [ ] Configure or investigate autoscaling for the selected App Service plan.
- [ ] Install Docker on an Ubuntu Azure VM and run Nginx.
- [ ] Build a Node.js Docker image and run it locally.
- [ ] Create an Azure Container Registry.
- [ ] Tag and push an image to ACR.
- [ ] Pull the image from another machine and run it.
- [ ] Clean up lab resources and review costs.

---

# Final Summary

- **Topics 51–55:** Azure's managed web hosting, deployment slots, release management, and scaling.
- **Topics 56–58:** Container fundamentals, Docker on an Azure VM, and building/running a containerized application.
- **Topics 59–60:** Creating Azure Container Registry, authenticating, tagging, pushing, listing, and pulling container images.

Together, these topics provide a practical foundation for deploying containerized applications using Azure and DevOps.
