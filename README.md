# insight_project
> chaos @ scale

## Table of Contents

  - [Introduction](README.md#Introduction)
  - [Setup](README.md#Setup)
  - [Notes](README.md#Notes)

## Introduction

### containerize, iac, orchestration

In this project, we will be performing the experiments on a music recommendation data pipeline. The original source code for this application can be found at this [link](https://github.com/ajgrande924/insight-music-project).

Scale is a music recommendation engine that finds similar songs based on shared instruments. The data pipeline is shown below:

S3 -> Spark -> Postgres -> Flask (replace w/ diagram)

  - S3: storage of midi files
  - Spark: extract data about instruments from midi files
  - Postgres: store results
  - Flask: view results

In this part of the project, I will be containerizing the data pipeline and automating the deployment utilizing IaC & container orchestration.

Services within the kubernetes cluster:

  - spark (stateless, kube / ec2?)
  - postgres (stateful, kube / rds?)
  - flask (stateless)

### chaos testing

As an industry, we are quick to adopt practices that increase the flexibility of developement and velocity of deployment. But how much confidence do we have on the complex systems we put out in production. Chaos engineering is the discipline of experimenting on a distributed system in order to gain confidence in a system's capability to withstand turbulent conditions in production. To address this uncertainty, we will be running a set of experiments to uncover systemic weaknesses.

In this experiment, we will be deploying two instances of the data pipeline, a control group and an experimental group. We also will define a 'steady state' which will indicate some sort of normal behavior for the pipeline. We will be running a set of tests on the experimental group which will try to disrupt the steady state, giving us more confidence in our system.

Tests:

  - Will the system will be able to handle CPU / shutdown attacks at the container level?
  - Will the system be able to handle attacks at the pod level?
  - Will the system be able to handle an attack on availability zone?

Results:

  - need to figure out how we test steady state, what metrics from the distributed system are we going to track?
  - for mvp, simplest monitoring/observability mechanism would be CloudWatch? for stretch, incorporate what?

Experiments:

  - application receives increased 

## Setup

#### setup eks cluster

The application requires the following dependencies:

  - jq
  - awscli
  - kubetcl
  - aws-iam-authenticator
  - terraform @ >= 0.12

To start the application, you can run `./oneclick deploy`, which essentially runs these commands, but with auto approve:
```sh
# provision with terraform
cd terraform
terraform init
terraform plan
terraform apply

# add new context to ~/.kube/config
aws eks update-kubeconfig --name <cluster_name> # test-eks-ajdev

# checks
kubectl version
kubetctl get nodes
kubectl config current-context
kubectl config view
```

To bring down the application you can run `./oneclick destroy`, which essentially runs these commands, but with auto approve:
```sh
# destroy infrastructure
cd terraform
terraform destroy -auto-approve
rm -rf .terraform terraform.tfstate*
```

## Notes

Questions I need to answer:

  - why are you containerizing this application?
  - why did you decide to containerize spark?
  - why did you decide to containerize postgress instead of using aws rds?
  - why are you using kubernetes for container orchestration?
  - how do you plan to set up application infrastructure within kubernetes cluster?
  - what type of chaos experiments will I run and how will I monitor this?

What baseline metrics to collect for CE, or use for comparison between experimental / control group:

  - Infrastructure Monitoring Metrics

    - Resource: CPU, IO, Disk & Memory
    - State: Shutdown, Processes, Clock Time
    - Network: DNS, Latency, Packet Loss

  - Alerting and On-Call Metrics

    - Total alert counts by service per week
    - Time to resolution for alerts per service
    - Noisy alerts by service per week (self-resolving)
    - Top 20 most frequent alerts per week for each service.

  - High Severity Incident (SEV) Metrics

    - Total count of incidents per week by SEV level
    - Total count of SEVs per week by service
    - MTTD, MTTR and MTBF for SEVs by service

  - Application Metrics

    - Events
    - Stack traces
    - Context
    - Breadcrumbs

### todo

  - [ ] test application locally
  - [ ] containerize data pipeline 
  - [ ] deploy a simple example on aws eks and enable Cloudwatch 
  - [ ] deploy real project on aws eks w/ a control group and experimental group; enable cloud watch
  - [ ] perform attacks on experimental group at container, pod, zone level
  - [ ] iterate over infrastructure / architechture based on results

### references

  - [Principles of Chaos Engineering](http://principlesofchaos.org/?lang=ENcontent)
  - [awesome-chaos-engineering](https://github.com/dastergon/awesome-chaos-engineering)
  - [chaos engineering monitoring metrics guide](https://www.gremlin.com/community/tutorials/chaos-engineering-monitoring-metrics-guide/)
  - [Why run Spark on Kubernetes?](https://medium.com/@rachit1arora/why-run-spark-on-kubernetes-51c0ccb39c9b)