# CloudMart – MultiCloud, DevOps & AI Challenge

Welcome to **CloudMart** — a comprehensive hands-on guide and project that teaches you how to provision real-world AWS infrastructure using **Terraform**, containerize applications using **Docker**, and deploy them using **Kubernetes** (EKS), all assisted by **AI tools** like Claude or ChatGPT.

This project is ideal for DevOps professionals, cloud learners, and automation enthusiasts looking to:

* Deploy a secure and versioned **S3 bucket**
* Create production-style **DynamoDB tables**
* Build a reusable Terraform workflow from an **EC2 workstation**
* Deploy containerized applications to **EKS** with **CI/CD pipelines**
* Integrate **AI Assistants** using **Amazon Bedrock**, **OpenAI**, and **BigQuery**

---

## What You'll Learn

* How to use AI to write and refine Terraform code
* Setting up secure AWS resources using Infrastructure as Code (IaC)
* Best practices for provisioning and verifying AWS services (S3, DynamoDB)
* Using Docker to containerize applications
* Building and pushing Docker images to ECR
* Deploying containerized workloads to AWS EKS
* Integrating with Amazon Bedrock and OpenAI Assistants
* Automating deployment with GitHub and AWS CodePipeline
* Exporting data to Google BigQuery and Azure Text Analytics

---

## Prerequisites

* AWS account with administrative access
* AWS CLI installed and configured
* Terraform installed
* EC2 instance with Amazon Linux 2
* GitHub account for CI/CD integration

---

## 1. Configure AWS CLI

```bash
aws configure
aws configure get region
```

Region: `us-east-1`

## 2. Create IAM Role & Launch EC2

IAM Role:

* IAM → Roles → Create role:

  * Trusted entity: AWS Service → EC2
  * Attach: AdministratorAccess policy
  * Name: `EC2Admin`

Launch EC2:

* AMI: Amazon Linux 2
* Instance type: `t2.micro`
* IAM Role: `EC2Admin`
* Enable public IP
* Add SSH access from your IP
* Tag: `Name = workstation`

## 3. Connect & Install Terraform

```bash
sudo yum update -y
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install -y terraform
terraform version
```

## 4. Create Terraform Project

```bash
mkdir terraform-project && cd terraform-project
nano main.tf
```

Paste Terraform code into `main.tf`

## 5. Initialize & Apply

```bash
terraform init
terraform plan
terraform apply
```

Type `yes` when prompted.

## 6. Verify CloudMart Resources

```bash
aws s3 ls
aws dynamodb list-tables --region us-east-1
```

Expected DynamoDB tables:

* `cloudmart-products`
* `cloudmart-orders`
* `cloudmart-tickets`

## 7. Backend & Frontend Docker Setup

**Backend Dockerfile**:

```Dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

**Frontend Dockerfile**:

```Dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install && npm run build
EXPOSE 3000
CMD ["npm", "run", "preview"]
```

**.env template**:

```
VITE_API_BASE_URL=http://<your_url_kubernetes_api>:5000/api
```

## 8. Build & Push Docker Images

Replace with your ECR URIs.

```bash
cd backend
docker build -t cloudmart-backend .
docker tag cloudmart-backend:latest <your-ecr-uri>/cloudmart-backend:latest
docker push <your-ecr-uri>/cloudmart-backend:latest
```

Repeat for frontend.

## 9. Create ECR Repositories (Optional via Console)

* cloudmart-backend
* cloudmart-frontend

## 10. Kubernetes Deployment

`cloudmart-backend.yaml` and `cloudmart-frontend.yaml` contain the Kubernetes manifests.
Use `kubectl apply -f <filename>` to deploy.

## 11. Setup GitHub & CI/CD

* Create GitHub repo
* Push your code
* Create CodeBuild projects for `cloudmartBuild` and `cloudmartDeploy`
* Create CodePipeline using GitHub source

**Buildspec for Build Project** (`buildspec.yml`): See detailed YAML in earlier steps.

**Buildspec for Deploy Project**: See full YAML in earlier steps.

## 12. AI Assistant Integration

* Amazon Bedrock: Create agent, action group, IAM role & policy
* OpenAI Assistant: Create assistant, enable API key, and save IDs

Redeploy backend with updated `.yaml` file and env variables for Bedrock + OpenAI.

## 13. Add Lambda for Product Listing

Follow `main.tf` updates for:

* Lambda role & policy
* `list_products` Lambda function
* `aws_lambda_permission` for Bedrock

## 14. Update Frontend & Backend Code

```bash
cp -R challenge-day2/ challenge-day2_bkp
cp -R terraform-project/ terraform-project_bkp
```

Remove non-Docker/YAML files from backend and frontend:

```bash
cd challenge-day2/backend
rm -rf $(find . -mindepth 1 -maxdepth 1 -not \( -name ".*" -o -name Dockerfile -o -name "*.yaml" \))
```

Repeat for frontend.

Download updated source code:

```bash
wget https://tcb-public-events.s3.amazonaws.com/mdac/resources/final/cloudmart-backend-final.zip
unzip cloudmart-backend-final.zip
```

Repeat for frontend. Push changes to GitHub.

## 15. Google Cloud BigQuery Setup

* Create GCP Project, Enable BigQuery API
* Create Dataset `cloudmart` and Table `cloudmart-orders`
* Service Account: Name `cloudmart-bigquery-sa` with BigQuery Data Editor role
* Create JSON key file `google_credentials.json`
* Copy to Lambda directory, zip Lambda:

```bash
cd challenge-day2/backend/src/lambda/addToBigQuery
zip -r dynamodb_to_bigquery.zip .
```

Update `.gitignore` to exclude `google_credentials.json`

## 16. Update Terraform for BigQuery Lambda

Update `main.tf` with:

* Lambda function for BigQuery
* Event source mapping
* Re-apply Terraform

```bash
terraform apply
```

## 17. Azure Text Analytics Setup

* Create Azure account and resource
* Retrieve `AZURE_ENDPOINT` and `AZURE_API_KEY`

## 18. Update Kubernetes Deployment

Update `cloudmart-backend.yaml` with Azure, Bedrock, and OpenAI env vars.
Redeploy:

```bash
kubectl apply -f cloudmart-backend.yaml
```

## 19. Validate

* Ensure frontend + backend are working
* Test AI assistants
* Confirm BigQuery ingestion and Azure sentiment analysis

---

Credits
This project was built by following the MultiCloud, DevOps & AI Challenge Bootcamp hosted by AWS and its partners. The bootcamp provided hands-on experience with real-world infrastructure provisioning, CI/CD pipelines, Kubernetes deployments, AI integrations, and more.

Special thanks to the instructors and organizers of the challenge for their detailed walkthroughs and resources.
