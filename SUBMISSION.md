# Submission Notes: The Junior Platform Engineer Technical Task

## How I ran the application

Firstly, I cloned the repository, installed dependencies with `npm install`, and started
the server with `npm start`. The app ran on `http://localhost:3000` and displayed
the release status table as expected.

Ran the test suite with `npm test` to confirm the health endpoint was working
before making any changes.

Initially the app failed to start with an `EADDRINUSE: address already in use :::3000` error. 
I had been using `Ctrl+Z` instead of `Ctrl+C` to stop the server, which suspends the process rather than killing it, leaving it holding the port in the background. 
I was not aware of it to be honest, so I used an AI assistant to identify the cause and learned to use `fuser -k 3000/tcp` to terminate the zombie process.
Small thing but worth noting.

---

## Issues I identified:

### CI/CD Pipeline (`deploy.yml` and `deploy.sh`)

The pipeline had no test step. It was checking out the code, installing
dependencies, and deploying without even testing if anything broke or had any conflicts when it comes to maybe dependenices versions mismatches or conflicts.
 => I added `npm test` as a step between dependency installation
and deployment, so a failing test will stop the pipeline before anything reaches
production.

`deploy.sh` felt kinda not so productive or informative. It echoes a start message, sleeps for two seconds, and
prints a completion message. I have noted this
but left it as a dummy script file since the real deployment logic would depend on the
specific infrastructure setup.

### Dockerfile

There is no `.dockerignore` file (We always have gitignore for code to keep the code secrets from public access ) Without it, a local `node_modules`
folder or any `.env` files could be copied into the image during the build.

### Terraform (`infra/main.tf`)

I reviewed the Terraform but did not apply any changes, as the task did not
require deployment.

A few things stood out:

**SSH open to Internet.** Port 22 is open to `0.0.0.0/0`, which means
anyone can attempt an SSH connection to the instance. This should be restricted
to known IP ranges. In my previous role I used to whitelist specific IP
addresses in Nginx for exactly this reason. The same principle applies here.

**Port 3000 also open to the internet.** We are open to anyone from the internet to get into our container.
For a production, service this is worth addressing because the app should not be publicly reachable
on its application port directly.

**Instance Size:** The instance type was `t3.xlarge`and you mentioned in your README file"This repository contains a small Node.js application and a simple CI/CD pipeline used as part of a Junior Platform Engineer technical task."
 => I changed it to `t3.micro` in the file, though the
right size in practice would depend on expected load.

**Instance Zone:** I remember that N.Virginia is cheaper in general but since EU might have regulation that data should reside on EU soil, maybe we can use the Irish zone as I remeber it was cheaper than the London one when I was investigating it for a previous project

### Application (`server.js`)

The `getReleases()` function reads the file synchronously on every request with
no error handling. If the file is missing => the server will crash
rather than return a meaningful error response. A try/catch with a proper error
response would be more practical here.

---

## Changes I made

- Added `npm test` to the pipeline before the deploy step
- Added a `.dockerignore` file
- Changed the instance type in `main.tf` from `t3.xlarge` to `t3.micro`

---

## What I would do with more time

- Replace `deploy.sh` with real deployment logic, maybe updating an ECS service or Lambda function
- Add a manual approval step before production deployments maybe
- Restrict the SSH ingress rule to specific IP ranges
- Add EBS encryption and access policies to the Terraform as I believe we need to configure IAM here
- Add error handling to `getReleases()` in the server