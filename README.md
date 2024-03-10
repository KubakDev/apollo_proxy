# Apolloproxy Server CI/CD Pipeline

## Overview

This guide outlines the process of setting up a Continuous Integration and Continuous Deployment (CI/CD) pipeline for the Apolloproxy server. The pipeline will automate the process of building the Apolloproxy server from source, running any tests, and deploying the binary to a remote server.

## Prerequisites

- A GitHub account and repository for the Apolloproxy project.
- A server with SSH access for deploying the binary. In this case, `tunl.kubakgroup.com` with the user `developer`.
- The SSH private key for the `developer` user on the server, securely stored.
- Basic familiarity with GitHub Actions.

## 1. Setup GitHub Repository

Ensure your boringproxy project is hosted on GitHub. Fork the original [boringproxy repository](https://github.com/boringproxy/boringproxy) if you haven't already.

## 2. Prepare the Server

- Ensure the server `tunl.kubakgroup.com` is accessible via SSH with the `developer` user.
- Install any necessary dependencies required for running the Apolloproxy server on the server.
- Setup the working directory and ensure the directory exists as specified in the service file instructions.

## 3. GitHub Secrets

Store necessary secrets in your GitHub repository:

- `SERVER_SSH_KEY`: The SSH private key for accessing your server.
- `SERVER_IP`: The IP address or hostname of your server, `tunl.kubakgroup.com`.
- `DEPLOY_USER`: The user for SSH on your server, `developer`.

Navigate to your GitHub repository settings, find the "Secrets" section, and add these secrets.

## 4. Create GitHub Actions Workflow

Create a `.github/workflows/cicd-pipeline.yml` file in your repository. This file will define the CI/CD pipeline.

```yaml
name: Apolloproxy CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Apolloproxy
        run: make

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            mv /usr/local/bin/Apolloproxy /usr/local/bin/Apolloproxy_old
            scp ./Apolloproxy $DEPLOY_USER@$SERVER_IP:/usr/local/bin/Apolloproxy
            systemctl daemon-reload
            systemctl restart Apolloproxy-server.service
```

## 5. Understanding the Workflow

- The `build` job will compile the Apolloproxy source code.
- The `deploy` job will replace the old binary on the server with the new one and restart the service.

## 6. Final Steps

- Commit and push the `.github/workflows/cicd-pipeline.yml` file to your GitHub repository.
- On push to the `main` branch, the CI/CD pipeline will automatically build and deploy the Apolloproxy server.

This simple CI/CD pipeline sets the foundation for automating the build and deployment process for the Apolloproxy server. You can expand it by adding steps for running tests, notifying team members of deployment status, or integrating with other tools and services.
