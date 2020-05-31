# DevOps, Microservices

The project's goal is to deploy a microservice in a highly available Kubernetes cluster.

The microservice I choose is a flask based calculator that is containerized and deployed in a Kubernetes cluster through a Jenkins pipeline. I ensured Node affinity, Pod Disruption budget, offload tls at the load balancer/ingress are working before it is served with traffic.


## To Recreate this project

you will need

- [Jenkins](https://www.jenkins.io/doc/book/installing/)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

Installed and configured using your credentials and docker being a sudoer, add this repository to your jenkins pipeline and the deployment is automated in jenkins


## Deployment strategy

I used blue/green deployment strategy, as it reduces downtime by running two identical environments with only one serving at a given time


these are two pipelines I have created

- [cluster formation](https://github.com/ph4n666/cluster-formation-) - creates a kubernetes cluster using eksctl and updates the Jenkins server the respective config file.

![img-1](images/createcluster.png)

- [Jenkins Pipeline](https://github.com/ph4n666/jenkins-pipeline)(This is the project)

![img-2](images/pipeline.png)


this pipeline does the following steps

1. builds and pushes the docker container
2. sets labels to nodes so as to use them in node affinity
3. deploys blue and green deployments
4. installs metrics server and uses it in horizontal pod autoscaling
5. adds pod disruption budget to both blue and green deployments
6. exposes the deployments using a load balancer port 8080

## note
TO use ingress expose the application through NodePort and Run

- Kubectl apply -f ingress.yml

I haven't used it as I do not have a registered domain name.

Also, Please refer to images of deployment for a better understanding 


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
