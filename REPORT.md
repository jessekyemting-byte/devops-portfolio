# Continuous Integration Pipeline for Portfolio Architecture

**Author:** Jesse Kyemting
**Date:** July 7, 2026
**Cohort:** SkillsNG – Helxi Cohort (Cloud Computing & DevOps Engineering)

---

## 1. Executive Summary & Objective

In this engineering project, I successfully architected, configured, and secured a fully automated Continuous Integration and Continuous Deployment (CI/CD) workflow for my professional portfolio website.

My primary goal was to satisfy a rigorous hands-free automation framework: any code modifications I commit and push to my version control system must automatically trigger a remote build server, execute multi-stage lifecycle validations, and deploy a containerized production environment onto an isolated cloud instance.

By tying together GitHub, an automated Jenkins controller, and Docker containers running on an AWS EC2 cloud architecture, I established a resilient GitOps pipeline built on modern infrastructure-as-code principles.

---

## 2. System Architecture & Component Breakdown

The deployment ecosystem I constructed utilizes a hybrid topology spanning localized automation engines, secure ingress tunnels, and decoupled public cloud compute instances.

### 🛠️ The Technical Stack

| Component | Details |
|---|---|
| **Source Control Management (SCM)** | GitHub — single source of truth containing my portfolio's core application code, asset packages, static structure, configuration manifests (`Dockerfile`), and declarative pipeline script (`Jenkinsfile`). |
| **CI/CD Automation Engine** | Jenkins (v2.555.3) — running locally on port `8080`, manages execution workloads, hooks into downstream SSH build nodes, monitors polling queues, and orchestrates pipeline execution. |
| **Target Cloud Compute Environment** | AWS EC2 — an isolated `t2.micro` compute instance running Ubuntu Server 24.04 LTS, serving as the live public production host. |
| **Runtime Containerization Engine** | Docker Engine (Community Edition) combined with an optimized `nginx:alpine` image for minimal footprint, sub-second spin-up, and isolated app execution layers. |
| **Edge Ingress Gateway Tunneling** | Serveo.net — a zero-installation reverse SSH port-forwarding utility used to bridge local private loopback interfaces securely to the public internet, satisfying external webhook ingress demands. |

**Core Plugins Utilized:**

- **Git Plugin & Pipeline** — allows Jenkins to clone and parse code states dynamically.
- **Blue Ocean** — provides an updated, highly visual interface for observing stage logs and tracking deployment timelines.
- **Docker Pipeline** — grants the pipeline native capabilities to interface with local or remote Docker daemons to build and tag container objects.

---

## 3. Implemented Pipeline Stages (Jenkinsfile Structure)

The automated build lifecycle was controlled entirely through a declarative structure within the repository's `Jenkinsfile`. The core operational blocks function as follows:

### Stage 1: Source Checkout

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

**Mechanism:** Jenkins intercepts the GitHub execution signal, instantiates a workspace mount, invokes the Git plugin, and clones down a pristine copy of the repository's main branch. This stage registers the specific commit hash and metadata responsible for firing the workflow.

### Stage 2: Environment & Dependency Validation

```groovy
stage('Environment Verification') {
    steps {
        sh 'docker --version'
        sh 'pwd && ls -la'
    }
}
```

**Mechanism:** This stage actively checks the host environment to ensure all prerequisite tools are ready. It prints out active working directory boundaries and checks for the existence of a valid, live Docker daemon before starting compilation phases.

### Stage 3: Container Compilation & Deployment

```groovy
stage('Build & Deploy Application') {
    steps {
        sh 'docker build -t website-img .'
        sh 'docker stop kyemting-site || true'
        sh 'docker rm kyemting-site || true'
        sh 'docker run -d -p 80:80 --name kyemting-site website-img'
    }
}
```

**Mechanism:**
1. Compiles a brand new production image tagged `website-img` using the workspace's static source and instructions within the project's `Dockerfile`.
2. Safely stops (`docker stop`) and removes (`docker rm`) any pre-existing, stale versions of the `kyemting-site` container to free up system sockets, appending `|| true` to prevent workflow crashes if a previous container did not exist.
3. Launches a fresh container detached in the background (`-d`), mapping external web traffic from the host machine's public Port 80 straight into the inner Nginx server container space at Port 80.

---

## 4. Challenges Encountered & Detailed Resolutions

### Challenge 4.1: Cloud-Layer and Host-Layer Network Isolation (Port 80/TCP)

**Detailed Manifestation:** After validating that the Docker pipeline completed successfully with zero error logs, attempts to view the live portfolio site by typing the public IP address (`16.170.212.161`) into a web browser resulted in an infinite loading spinner followed by an `ERR_CONNECTION_TIMED_OUT` error page.

**Root Cause Analysis:** The application layer inside Docker was listening perfectly, but external traffic was running into two separate, hardened firewalls: the AWS EC2 security perimeter and the native host OS operating framework.

**Resolution Path:**
- **AWS Infrastructure:** Navigated to the EC2 Dashboard, opened the instance's associated Security Group (`sg-0494d21c619843d46`), clicked **Edit Inbound Rules**, and appended a rule allowing standard HTTP (Port 80) from any global IP address resource (`0.0.0.0/0`).
- **Ubuntu OS Host Layer:** Accessed the server terminal via SSH and verified that the Uncomplicated Firewall (`ufw`) was dropping active incoming packets on Port 80. Executed `sudo ufw allow 80/tcp` to manually update the operating system kernel routing tables. The browser instantly established a connection, successfully pulling up the dark-themed portfolio site.

### Challenge 4.2: Local Loopback Isolation (localhost) vs. GitHub Webhook Ingress

**Detailed Manifestation:** To satisfy the Task 4 requirements, a GitHub Webhook was initially established using the target cloud server's IP. However, attempts to test the link resulted in a critical red warning icon displaying an explicit network fault: `failed to connect to host`.

**Root Cause Analysis:** Diagnostic execution of port socket tracking utilities via `sudo ss -tulnp | grep 8080` returned a completely null value on the remote cloud node. This confirmed that while the live application container was running on AWS, the underlying Jenkins controller dashboard was actually running locally inside a private network infrastructure, bound strictly to `localhost:8080`. Because GitHub resides on the public internet, it had no native path to look inside a private consumer router or interface directly with a closed localhost circuit.

**Resolution Path:** Rather than spending hours downloading heavy third-party agent apps, configuring intricate configuration profiles, or redeploying the core controller framework, an engineering bypass was executed via secure shell reverse port forwarding tunnels using **serveo.net**.

From the local Windows administrative terminal, the following network mapping command was issued:

```bash
ssh -R 80:localhost:8080 serveo.net
```

This command securely established a public, encrypted endpoint proxy (`https://ad3fe68caa289c96-197-211-63-59.serveousercontent.com`). GitHub was then updated with this live address, appended with the explicit path `/github-webhook/`. Upon triggering a redelivery packet, the server successfully bounced the payload through the tunnel straight into the local Jenkins controller, returning a pristine `200 OK` HTTP status response code and completing the automated validation phase.

---

## 5. Verification & Submission Artifact Inventory

The final implementation completely satisfies all guidelines outlined in the cohort requirements. The corresponding verified proof data consists of:

- **GitHub Repository State:** Accessible via the active codebase containing the production configuration.
- **Pipeline Execution Integrity:** Verified on the Jenkins UI under **Build #19**, confirming fully successful multivariable execution stages with an output log state of `Finished: SUCCESS`.
- **Webhook Ingress Validation:** Verified within GitHub Settings, documenting successful outbound payload distribution with a green checkmark indicating flawless handshake metrics.
- **Application Browser Accessibility:** Verified by visiting the live public portfolio application hosted at `http://16.170.212.161`, showing clear execution of skills, experience histories, and terminal themes.
- docs: add comprehensive CI/CD project report
