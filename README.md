# 🚀 CI/CD Pipeline – GitHub Actions

This repository uses a GitHub Actions CI/CD pipeline to automate the full development lifecycle:

- Java compilation with Maven
- Security scanning (Trivy & Gitleaks)
- Unit testing
- SonarQube analysis
- Docker image build & push to Docker Hub
- Deployment to Amazon EKS

---

## 🧠 Workflow Trigger

This pipeline runs **on every push to the `main` branch**.

---

## 🧩 Workflow Jobs

### 🔧 1. `compile`
**Purpose**: Compile the Java application.  
**Steps**:
- Checkout code
- Set up JDK 17 (Temurin)
- Run `mvn compile`

---

### 🔐 2. `security-check`
**Depends on**: `compile`  
**Purpose**: Perform security and secrets scanning.  
**Steps**:
- Install [Trivy](https://github.com/aquasecurity/trivy) and scan filesystem
- Install [Gitleaks](https://github.com/gitleaks/gitleaks) and scan for secrets
- Output saved to `fs-report.json` and `gitleaks-report.json`

---

### 🧪 3. `test`
**Depends on**: `security-check`  
**Purpose**: Run unit tests.  
**Steps**:
- Checkout code
- Set up JDK 17
- Run `mvn test`

---

### 📦 4. `build_project_and_sonar_scan`
**Depends on**: `test`  
**Purpose**: Build the project and run SonarQube analysis.  
**Steps**:
- Package the JAR using `mvn package`
- Upload the JAR as a GitHub artifact
- Re-checkout the repo with full history for Sonar analysis
- Run SonarQube scan and Quality Gate check

**Secrets/Vars Required**:
- `SONAR_TOKEN`
- `SONAR_HOST_URL`

---

### 🐳 5. `build_docker_image_and_push`
**Depends on**: `build_project_and_sonar_scan`  
**Purpose**: Build and push Docker image to Docker Hub.  
**Steps**:
- Download the JAR artifact
- Login to Docker Hub
- Build and push Docker image tagged: `bhuvanreddy/bankapp:latest`

**Secrets/Vars Required**:
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

---

### ☸️ 6. `deploy_to_kubernetes`
**Depends on**: `build_docker_image_and_push`  
**Purpose**: Deploy the image to Amazon EKS using `kubectl`.  
**Steps**:
- Install and configure AWS CLI
- Set up `kubectl` and kubeconfig from secret
- Apply Kubernetes deployment from `ds.yml`

**Secrets Required**:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `EKS_KUBECONFIG` (base64 or inline kubeconfig)

---

## 🔐 Secrets & Variables Setup

| Name                  | Type       | Purpose                                      |
|-----------------------|------------|----------------------------------------------|
| `SONAR_TOKEN`         | Secret     | For SonarQube authentication                 |
| `SONAR_HOST_URL`      | Variable   | SonarQube base URL                           |
| `DOCKERHUB_USERNAME`  | Variable   | Docker Hub username                          |
| `DOCKERHUB_TOKEN`     | Secret     | Docker Hub access token                      |
| `AWS_ACCESS_KEY_ID`   | Secret     | AWS access for EKS                           |
| `AWS_SECRET_ACCESS_KEY`| Secret    | AWS secret key for EKS                       |
| `EKS_KUBECONFIG`      | Secret     | Kubernetes config to access EKS              |

---



