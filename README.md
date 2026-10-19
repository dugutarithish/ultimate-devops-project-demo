# OpenTelemetry Astronomy Shop on AWS EKS

A DevOps learning project that packages and deploys the OpenTelemetry Astronomy Shop Demo, a microservices-based e-commerce application, on Amazon EKS. I used this project to practice containerization, Kubernetes deployment, AWS networking, Terraform, Helm, GitHub Actions, Argo CD, and custom-domain routing.

> **Project note:** This repository is a fork and deployment adaptation of the upstream OpenTelemetry Demo. The application itself is the Astronomy Shop demo; my work focuses on the DevOps infrastructure and delivery workflow around it. See the credits below.

## Project overview

The Astronomy Shop simulates an online store for astronomy-related products such as telescopes. It consists of multiple services written in different languages and communicating through protocols such as gRPC, HTTP, and TCP. The service map below is the architecture of the demo application, not the AWS infrastructure diagram.

![OpenTelemetry Astronomy Shop microservices architecture](https://raw.githubusercontent.com/dugutarithish/ultimate-devops-project-demo/main/docs/images/architecture.png)

> Replace the architecture image path above with the actual committed image path if you add the screenshot to this repository. The original architecture screenshot is not currently assumed to exist at this path.

## What I built and practised

- **Docker:** Reviewed and built Docker images for the application's different services. Worked with service-specific Dockerfiles and multi-stage builds to separate build-time dependencies from runtime images.
- **Kubernetes:** Prepared Kubernetes Deployments and Services for the microservices and learned how workloads discover and communicate with each other inside the cluster.
- **Helm:** Used Helm charts to package and manage Kubernetes resources, including installing, upgrading, and uninstalling releases.
- **AWS infrastructure with Terraform:** Provisioned the VPC and EKS infrastructure with public and private subnets across multiple Availability Zones.
- **Terraform remote state:** Configured an S3 backend for Terraform state and DynamoDB-based state locking in the setup I used. Confirm the backend configuration in the current Terraform files before recreating it; locking configuration can vary by Terraform version.
- **Cluster access:** Used an EC2 bastion host as an administrative access point to reach private resources and work with the EKS cluster.
- **Ingress and domain routing:** Configured ingress-based routing toward the frontend proxy and connected DNS for my domain, `rithish.co.in`, using Amazon Route 53. The domain and DNS are environment-specific and require valid DNS records and a working ingress endpoint.
- **CI with GitHub Actions:** Practised automated build and test steps, code-quality checks, and building/pushing container images.
- **CD with Argo CD and Helm:** Used a GitOps workflow in which Argo CD reconciles Kubernetes resources from the Git repository and Helm configuration. The exact trigger and image-update behaviour depends on the workflow and Argo CD configuration committed in this repository.

## Architecture

The high-level deployment flow is:

```text
Developer pushes code
        |
        v
GitHub repository
        |
        v
GitHub Actions (CI)
  - checkout
  - tests and quality checks
  - build container image
  - push image to registry
        |
        v
Updated deployment configuration in Git
        |
        v
Argo CD + Helm (CD / GitOps)
        |
        v
Amazon EKS cluster in private subnets
        |
        v
Kubernetes Services -> Frontend Proxy -> Astronomy Shop
        ^
        |
Ingress / load balancer
        ^
        |
Route 53 DNS -> rithish.co.in
```

### AWS network layout

- **VPC:** Isolated network for the deployment.
- **Public subnets:** Used for public-facing infrastructure and the bastion host, according to the deployment setup.
- **Private subnets:** Used for the EKS worker capacity and application workloads.
- **Multiple Availability Zones:** Subnets were spread across more than one AZ for the intended network layout.
- **Amazon EKS:** Runs the microservices as Kubernetes workloads.
- **Amazon Route 53:** Hosts DNS records for the custom domain.

This is a learning deployment, not a claim of production readiness. Before using it for production, review IAM permissions, network exposure, secrets handling, TLS, autoscaling, backups, observability, costs, and disaster recovery.

## Technology stack

| Area | Tools |
| --- | --- |
| Cloud | AWS |
| Infrastructure as Code | Terraform |
| Containerization | Docker, multi-stage Dockerfiles |
| Orchestration | Kubernetes, Amazon EKS |
| Kubernetes packaging | Helm |
| Continuous Integration | GitHub Actions |
| Continuous Delivery / GitOps | Argo CD |
| Networking and DNS | VPC, public/private subnets, Ingress, Route 53 |
| Terraform state | Amazon S3 and DynamoDB locking (as configured in my setup) |
| Application | OpenTelemetry Astronomy Shop Demo |

## Repository structure

Important project paths include:

- `src/` — application microservices and source code.
- `kubernetes/` — Kubernetes workload manifests, Services, and ingress configuration.
- `.github/workflows/` — GitHub Actions workflows.
- `docker-compose.yml` — local multi-container demo setup.
- `docker-compose.minimal.yml` — reduced local demo configuration.

Terraform and Helm paths may differ by branch or by the files used to provision the environment. Check the repository tree before running deployment commands.

## CI/CD workflow

1. A code change is pushed to GitHub or a pull request is opened.
2. GitHub Actions runs the configured checks, including service tests/build and code-quality steps where defined.
3. The pipeline builds a container image and pushes it to the configured registry when the workflow conditions and credentials allow.
4. The deployment configuration is updated to reference the intended image version, if that step is enabled in the workflow.
5. Argo CD detects Git changes and reconciles the application's desired state into EKS using the configured manifests or Helm chart.

Use immutable image tags (for example, a commit SHA) rather than relying only on `latest`. Keep registry credentials and AWS credentials in GitHub Secrets or an appropriate identity-based configuration. Never commit real credentials, private keys, or sensitive `.env` values.

## Key things I learned

- How multi-stage Dockerfiles reduce the runtime image to the artifacts and libraries actually needed.
- How a multi-service application is modelled with Kubernetes Deployments and Services.
- How Helm simplifies installation, upgrades, and removal of Kubernetes applications.
- How VPCs, public/private subnets, Availability Zones, and bastion access fit into an EKS deployment.
- How Terraform remote state and state locking help coordinate infrastructure changes.
- How ingress rules and DNS records work together to expose an application through a custom domain.
- How CI image building and GitOps-based delivery separate application packaging from cluster reconciliation.
- How to debug real deployment problems involving networking, load balancer provisioning, IngressClass ownership, DNS, Terraform state, and Kubernetes resources.

## Running the demo locally

For local development, start with the upstream OpenTelemetry Demo instructions rather than assuming the AWS deployment configuration is portable:

- [OpenTelemetry Demo repository](https://github.com/open-telemetry/opentelemetry-demo)
- [Docker deployment documentation](https://opentelemetry.io/docs/demo/docker_deployment/)
- [Kubernetes deployment documentation](https://opentelemetry.io/docs/demo/kubernetes_deployment/)

The AWS deployment requires your own AWS account, IAM permissions, Terraform variables, DNS configuration, registry credentials, and Kubernetes tooling. Review the configuration before applying it. Cloud resources can incur charges.

## Screenshots

Add screenshots you have committed to this repository, then link them using their exact case-sensitive paths. Example:

```markdown
![Astronomy Shop homepage](docs/images/home-page.png)
![EKS workloads](docs/images/eks-workloads.png)
![Argo CD application](docs/images/argocd-application.png)
```

GitHub image links return 404 when the referenced file is not committed on the branch at that path. Confirm the file exists in the repository before adding it here.

## Credits and references

- **Application:** [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo)
- **DevOps learning reference:** [Abhishek Veeramalla's ultimate-devops-project-demo](https://github.com/iam-veeramalla/ultimate-devops-project-demo)
- **My repository:** [dugutarithish/ultimate-devops-project-demo](https://github.com/dugutarithish/ultimate-devops-project-demo)

This repository builds on upstream open-source work. Credit original authors and retain the applicable license and attribution notices. My contribution described here is the infrastructure, deployment, and CI/CD learning and customisation performed for this environment.
