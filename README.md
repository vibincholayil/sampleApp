# Jenkins CI/CD Pipeline

## Introduction

This Jenkins pipeline automates the end-to-end process of deploying a containerized application. It fetches code from a Git repository, builds a Docker image, and deploys it to a Kubernetes cluster.

Key features include:

- Parameterization: Allows dynamic selection of the target Kubernetes namespace at runtime.
- Reusable Stages: Pipeline steps are modular and can be reused across multiple projects.
- Dynamic Build IDs: Docker images are tagged with the current date for traceability and version control.

This pipeline streamlines the CI/CD workflow, reduces manual steps, and ensures consistent, reproducible deployments across environments.

---

## Step 1: Setup Environment

1. Created an Ubuntu VM using VMware.
2. Installed Kubernetes on the VM.
3. Installed Jenkins in Kubernetes using Helm:

   helm repo add jenkins https://charts.jenkins.io
   helm repo update
   helm install my-jenkins jenkins/jenkins

4. Checked Jenkins pod status:

   kubectl get pods
   - Initially, the pod was Pending due to PVC issues.

---

## Step 2: Configure Persistent Storage

1. Created a PersistentVolume (PV) and PersistentVolumeClaim (PVC) using pv-pvc.yml.
2. Applied the PV and PVC:

   kubectl apply -f pv-pvc.yml

3. Removed the taint on the control-plane node to allow pod scheduling:

   kubectl taint nodes ubuntu node-role.kubernetes.io/control-plane:NoSchedule-

4. Verified that the pod and service were running.

---

## Step 3: Expose Jenkins

1. Exposed Jenkins service as a NodePort:

   kubectl expose pod my-jenkins-0 --type=NodePort --name=my-jenkins --port=8080 --target-port=8080

2. Accessed Jenkins in a browser using:

   http://<VM_IP>:<NodePort>

3. Retrieved Jenkins admin credentials:

   kubectl get secret my-jenkins -o jsonpath="{.data.jenkins-admin-user}" | base64 --decode
   kubectl get secret my-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode

---

## Step 4: Jenkins Configuration

- General Settings:
  - Discard old builds: Days to keep builds = 3, Max # of builds = 3

- Pipeline Settings:
  - Pipeline script from SCM → Git → Repository URL

- System Configuration:
  - Number of executors = 4
  - Jenkins URL = http://192.168.152.136:30460/

- Installed Plugins:
  - Kubernetes
  - Docker

- Nodes:
  - Created a new node 'build-agent'
  - Remote root directory = /home/jenkins
  - Number of executors = 4

- Credentials:
  - Added Docker registry credentials for image push

---

## Step 5: Fetch Git Repository

Repository: https://github.com/vibincholayil/sampleApp.git

Add the following files:
- Jenkinsfile – CI/CD pipeline (declarative)
- Dockerfile – Docker image build
- deployment.yaml – Kubernetes deployment

Pipeline Features:
- Generates a build tag with the current date
- Builds Docker image
- Pushes image to Docker registry
- Deploys to Kubernetes using parameters for namespace selection

---

## Step 6: Run the CI/CD Pipeline

- The pipeline fetches code from your Git repository, builds the Docker image, pushes it to the Docker registry, and deploys it to Kubernetes.
- Monitor the pipeline stages and logs in Jenkins.

---

## Conclusion

In this project, I successfully implemented a complete CI/CD pipeline using Jenkins, Docker, and Kubernetes. The pipeline:

- Automatically fetches code from a Git repository
- Builds and pushes Docker images
- Deploys applications to a Kubernetes cluster
- Supports dynamic namespace selection
- Reuses modular pipeline stages
- Generates dynamic build tags based on the current date

This setup demonstrates end-to-end automation, enabling faster and reliable software delivery while reducing manual intervention and deployment errors. It also highlights effective integration of Jenkins agents, Kubernetes resources, and Docker credentials for a fully functional CI/CD workflow.

