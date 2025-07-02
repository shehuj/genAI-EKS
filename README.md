# genAI-EKS
Deploy Generative AI Models on Amazon EKS

GenAI models represent a significant breakthrough in the field of Artificial Intelligence/Machine Learning, due to its wide ranging applicability along with easy accessibility for non-AI experts. Traditionally, utilizing AI meant creation of a specialized model for each specific use-case, which required a huge amount of compute and human resources each time. GenAI models overcome this bottleneck by creating Foundation Models (FM). FMs allow reuse by providing ability to fine-tune them to be utilized for multiple use-cases without having to build models from the ground up repeatedly. The most popularly used foundational models today utilize transformers (text generation)/diffusers (i.e., image generation) to achieve this adaptability. These models have potential applicability across a wide range of use-cases and industry verticals ranging from chatbots and virtual assistants to generating videos completely via text prompts for marketing.

Solution overview
Architecture for deploying Stable Diffusion Model on Amazon EKS

![Screenshot 2025-01-20 at 15 39 25](https://github.com/user-attachments/assets/e2eefe22-0e41-4925-b22c-b58cd878cf6f)

There is a vast eco-system of tools available to build and run models, even within the kubernetes landscape. One emerging stack on kubernetes is Jupyterhub, Argo Workflows, Ray and Kubernetes. We call this the JARK stack and you can run this entire stack on Amazon EKS

![Screenshot 2025-01-20 at 15 43 30](https://github.com/user-attachments/assets/534002cc-43ae-45c9-9220-5079a6bb16b5)

JARK Architecture

JupyterHub provides a shared platform for running notebooks that are popular in business, education, and research. It promotes interactive computing where users can execute code, visualize results, and work together. In the realm of GenAI, JupyterHub accelerates the experimentation process, especially in the feedback loop. It’s also where Data Engineers collaborate on models for Prompt Engineering.

Argo Workflows  is an open source container-native workflow engine for orchestrating parallel jobs on Kubernetes. It provides a structured and automated pipeline tailored for the fine-tuning of models.

The Argo workflow pipeline composes of the following stages:

Data Preparation: Organize and preprocess training datasets.
Model Configuration: Define the architecture and hyper-parameters for LLM fine-tuning.
Fine-tuning: Execute the training regimen.
Validation: Gauge the performance of the fine-tuned model.
Hyperparameter Tuning: Optimize settings for peak performance.
Model Evaluation: Assess the model’s efficacy using separate test data.
Deployment: Host the model to cater to inference requests.

Ray is an open-source distributed computing framework that makes it easy to scale applications and to use state-of-the-art machine learning libraries. Ray is used to distribute the training of generative models across multiple nodes, which accelerates the training process and allows for the handling of larger datasets.

Ray Serve is a powerful model serving library that facilitates online inference application programming interface (API) creation. Notably, it’s compatible with major frameworks like PyTorch, Keras, and Tensorflow. It’s optimized for serving LLMs with features like response streaming, dynamic request batching, and multi-node/multi-GPU (Graphical processing Unit) support. Beyond just model serving, Ray Serve allows the integration of multiple models and business rules into a single service. Built on Ray, it’s designed for scalability across machines and offers resource-efficient scheduling. In relation to JupyterHub, while the text doesn’t explicitly state a connection, both tools can be part of a larger ML ecosystem, with JupyterHub facilitating interactive computing and Ray Serve handling model deployment and serving.

Kubernetes is a powerful container orchestration platform that automates the deployment, scaling, and management of containerized applications. Kubernetes provides the infrastructure to run and scale GenAI models in containers, which ensures high availability, fault tolerance, and efficient resource utilization.

Solution Architecture:

<img width="910" alt="Screenshot 2025-07-02 at 15 59 46" src="https://github.com/user-attachments/assets/f35a268c-6387-4091-b71d-1f58de94de15" />

Walkthrough:

Step-by-step guide on how to tune and deploy a generative model on Amazon EKS
Use-case

As we mentioned earlier, most downstream use-cases require fine-tuning an LLM for specific tasks as per your business requirements. This typically just requires a small dataset with relatively few examples and in most cases can be performed with a single GPU. In this post, we use the example of Dreambooth to demonstrate how we can adapt a large text-to-image model such as Stable Diffusion to generate contextualized images of a subject (e.g., a dog) in different scenes. The Dreambooth paper describes an approach to bind a unique identifier with the subject (e.g., a photo of [v]dog), in order to synthesize photos of the said subject in photorealistic images based on the input prompt (e.g., a photo of [v]dog on the moon).

Commercial applications of Dreambooth may include:

    Generating images from text descriptions for social media platforms, e-commerce sites, and other other online platforms.
    Creating personalized avatars of profile pictures for users.
    Generating product images for online stores
    Creating marketing materials and educational content that uses visual aids, etc.

We refer to our sample model and inference service as dogbooth for the remainder of the post. To run the fine-tuning for dogbooth on Amazon EKS, we make use of several open-source technologies for a self-managed approach.

In addition to the JARK stack, we also take advantage of the following two libraries from Hugging Face that give us the tools to personalize the Stable Diffusion model: Accelerate and Diffusers.

Accelerate is an open-source library specifically designed to simplify and optimize the process of training and fine-tuning deep learning models. For our purpose, it provides a high-level API that makes it easy to experiment with different hyper-parameters and training configurations without the need to rewrite the training loop each time and efficiently use available hardware resources.

Diffusers is the go-to library for state-of-the-art pre-trained diffusion models for generating images, audio, and even 3D structures of molecules. They provide easy to use training examples as a collection of scripts to demonstrate how to effectively use the diffusers library for a variety of personalization tasks, such as Unconditional Training, Text-to-Image Training, Dreambooth, ControlNet, Custom Diffusion, etc.
Steps to deploy Stable Diffusion Model on Amazon EKS
Pre-requisites

    AWS Command Line Interface (AWS CLI) v2 – the CLI for AWS services
    kubectl – the Kubernetes CLI
    Terraform 1.5 – an infrastructure as code tool
    Hugging Face Token with writescope
    jq – a lightweight and flexible command line JSON processor

Step 1: Clone the GitHub repository

git clone https://github.com/awslabs/data-on-eks.git

Step 2: Deploy the sample blueprint

Navigate to the ai-ml/jark-stack blueprint directory and run the ./install.sh script. This script runs the terraform init and terraform -apply commands. Note that by default the solution is configure in us-west-2 Region. Please update the variables.tf file to deploy it to another AWS region. Note that this might take approximately 30 minutes for the deployment to complete successfully.

cd data-on-eks/ai-ml/jark-stack/terraform
export TF_VAR_huggingface_token=hf_XXXXXXXXXX

./install.sh  

Initializing ...
Initializing the backend...
Initializing modules...

Initializing provider plugins...
Terraform has been successfully initialized!
...
SUCCESS: Terraform apply of all modules completed successfully

The Terraform based blueprint provisions the following:

    Amazon Virtual Private Cloud (Amazon VPC) with subnets, route tables, NAT gateway
    Amazon EKS cluster (version 1.27)
    Amazon EKS core-managed node groupused to host some of the add-ons that we’ll provision on the cluster.
    Another EKS gpu-managed node group used to provision GPU based instances. For the purpose of the post, while we chose to use the Amazon Elastic Compute Cloud (Amazon EC2 ) G5 Instancesthat are based on the NVIDIA A10G Tensor Core GPUs that features 28GB memory per GPU.
    A Kubernetes secret for the Hugging Face token and the configmapcontaining our sample ipython notebook that will be mounted on the notebook pod.
    Install several add-ons we discuss in the next section.

Add-ons

Let’s take a look at the add-ons, which are the operational software pods that are deployed as a part of the stack.

aws eks update-kubeconfig --name jark-stack --region us-west-2

kubectl get deployments -A
                                                                
NAMESPACE              NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx          ingress-nginx-controller                             1/1     1            1           36h
jupyterhub             hub                                                  1/1     1            1           36h
jupyterhub             proxy                                                1/1     1            1           36h
kube-system            aws-load-balancer-controller                         2/2     2            2           36h
kube-system            coredns                                              2/2     2            2           2d5h
kube-system            ebs-csi-controller                                   2/2     2            2           2d5h
kuberay-operator       kuberay-operator                                     1/1     1            1           36h
nvidia-device-plugin   nvidia-device-plugin-node-feature-discovery-master   1/1     1            1           36h

Amazon EBS CSI Driver

The Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver allows Amazon Elastic Kubernetes Service (Amazon EKS) clusters to manage the lifecycle of Amazon EBS volumes for persistent volumes.

Set the default StorageClass to gp3 .

AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. You need a Network Load Balancer to access our Jupyter notebooks and eventually another Network Load Balancer that provides an ingress for our self-hosted inference endpoint, which is discussed later on in the post.

NVIDIA Device Plugin

The NVIDIA device plugin for Kubernetes is a DaemonSet that allows you to automatically expose the number of GPUs to Kubernetes thus allowing us to run GPU enabled containers on our cluster.

If you look under the data-on-eks/ai-ml/jark-stack/terraform/helm-values folder, you will see the values three HELM values file.  In this example, pass a minimal values.yaml to the helm chart that enables the gpu-feature-discovery and node-feature-discovery features of the chart as well as a toleration that allows the node-feature-discovery pods to run on the GPU nodes we created via the blueprint. We’ll dive deeper in to advanced configuration of the NVIDIA Device Plugin/NVIDIA GPU Operator in another post.

gfd:
  enabled: true
nfd:
  enabled: true
  worker:
    tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      - operator: "Exists"

JupyterHub 

Similarly, you’ll see a jupyterhub-values.yaml.  The Terraform script installed JupyterHub.  In this example, we passed a values.yaml to the helm chart that configures JupyterHub to use a Load Balancer for access, specify GPU requirement on the resource, an Amazon EBS based storage volume for persistence. Please note that we show the use of basic user authentication based on username and password for the notebooks for demonstration purpose only. For real-world setup consider using an identity provider.

...
proxy:
  service:
    annotations:
      service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
      service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
      service.beta.kubernetes.io/aws-load-balancer-type: external
      service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: 'true'
      service.beta.kubernetes.io/aws-load-balancer-ip-address-type: ipv4
singleuser:
  image:
    name: public.ecr.aws/h3o5n2r0/gpu-jupyter
    tag: v1.5_cuda-11.6_ubuntu-20.04_python-only
    pullPolicy: Always
..
  extraResource:
    limits:
      nvidia.com/gpu: "1"
  extraEnv:
    HUGGING_FACE_HUB_TOKEN:
      valueFrom:
        secretKeyRef:
          name: hf-token
          key: token
  storage:
    capacity: 100Gi
...
      - name: notebook
        configMap:
          name: notebook
...

The dockerfile for the container image that we use for the notebook is provided in the repository under src/notebook/Dockerfile directory.

Ingress-Nginx

Ingress-nginx allows us to use some path rewrite rules to expose both the Ray dashboard and the inference endpoint using the same load balancer. This model also allows us to run multiple Ray Serve endpoints and use path based routing to serve say different model versions for example using the same load balancer.

Kuberay-Operator

The KubeRay Operator makes deploying and managing Ray clusters on top of Kubernetes painless. Clusters are defined as a custom RayCluster resource and managed by a fault-tolerant Ray controller. The KubeRay Operator automates Ray cluster lifecycle management, autoscaling, and other critical functions.

Later in the post, we describe how to create an inference service for dogbooth using the RayService custom resource definition on the cluster.
Step 3: Fine-Tune Stable Diffusion Model

You are now ready start experimenting with our model and prepare a notebook that helps us personalize it for your needs. Get the Load Balancer DNS.

kubectl get svc proxy-public -n jupyterhub --output jsonpath='{.status.loadBalancer.ingress[0].hostname}'

Open the returned DNS hostname (e.g., k8s-jupyterh-proxypub-xxx.elb.us-west-2.amazonaws.com) in the web browser.

Login using the username user1 and the password as specified in the jupyterhub-values.yaml.

Jupyterhub server is starting

This triggers a pod jupyter-user1 to be provisioned on the g5 instance.   You can see the pod if you issue  kubectl get pods -n jupyterhub

This triggers a pod jupyter-user1 to be provisioned on the g5 instance.   You can see the pod if you issue  kubectl get pods -n jupyterhub

Upon successful launch, you should be redirected to the notebook console on the browser.

Start the provided python notebook under the dogbooth directory in the notebook user interface (UI’s) file browser as shown in the following figure.

Start the provided python notebook under the dogbooth directory in the notebook user interface (UI’s) file browser as shown in the following figure.

You can then step through the notebook’s cells as shown in the following figure. The first cell runs the NVIDIA System Management Interface (nvidia-smi) to verify our notebook instance is correctly provisioned on the GPU node and it sees the underlying NVIDIA A10G GPU.

You can then step through the notebook’s cells as shown in the following figure. The first cell runs the NVIDIA System Management Interface (nvidia-smi) to verify our notebook instance is correctly provisioned on the GPU node and it sees the underlying NVIDIA A10G GPU.

The next four cells setup our development environment by cloning the Hugging Face diffusers GitHub repository and installing some python dependencies that the diffusers need. Additionally, install xFormers in order to enable memory efficient attention, as described in the dreambooth example.

The next four cells setup our development environment by cloning the Hugging Face diffusers GitHub repository and installing some python dependencies that the diffusers need. Additionally, install xFormers in order to enable memory efficient attention, as described in the dreambooth example.

Once you have stepped through those tasks, install bitsandbytes so that you use the 8-bit optimizer to reduce memory requirements further.

Once you have stepped through those tasks, install bitsandbytes so that you use the 8-bit optimizer to reduce memory requirements further.

Upon successful installation of bitsandbytes, next setup the requirements for running the dreambooth training script. This includes installing some additional dependencies, setting up a default configuration for accelerate , logging into Hugging Face, and downloading a sample dataset from Hugging Face.

Upon successful installation of bitsandbytes, next setup the requirements for running the dreambooth training script. This includes installing some additional dependencies, setting up a default configuration for accelerate , logging into Hugging Face, and downloading a sample dataset from Hugging Face.

Now, you can launch training after setting up environment variables for the location of the input model, dataset directory, and output directory of the tuned model. Hugging Face accelerate does all the heavy lifting to help us experiment with the model. The hyper-parameters used for the following sample are optimized for the training to run successfully on 1 NVIDIA A10G GPU with 24 GB memory.

Now, you can launch training after setting up environment variables for the location of the input model, dataset directory, and output directory of the tuned model. Hugging Face accelerate does all the heavy lifting to help us experiment with the model. The hyper-parameters used for the following sample are optimized for the training to run successfully on 1 NVIDIA A10G GPU with 24 GB memory.

This takes about an 1.5 hours to complete – perfect time to grab some food. You can reduce the amount of training time by changing some of the hyper-parameters (e.g.,  –max_train_steps=400 ) but this comes at the expense of model’s performance and accuracy.

After the training script completes, you can verify the model has been created and run a sample inference to check how it performs.

After the training script completes, you can verify the model has been created and run a sample inference to check how it performs.

Open the  dog-bucket.png file.  This picture is stored under /home/jovyan/diffusers/examples/deambooth folder

Open the  dog-bucket.png file.  This picture is stored under /home/jovyan/diffusers/examples/deambooth folder

Since accelerate uploads the model to Hugging Face as well, you can even test a sample inference on their Hosted Inference API. You’ll find it if you navigate to https://huggingface.co/spaces/<huggingface_username>/dogbooth or the value that you provided for $OUTPUT_DIR.

Since accelerate uploads the model to Hugging Face as well, you can even test a sample inference on their Hosted Inference API. You’ll find it if you navigate to https://huggingface.co/spaces/<huggingface_username>/dogbooth or the value that you provided for $OUTPUT_DIR.

If the model overfits or underfits, then please refer to an in-depth analysis of dreambooth performed by Hugging Face to help you adjust the hyper-parameters to improve model performance. Those recommendations are beyond the scope of this post.
Step 4: Serving the Large Language Model

Now that you have fine-tuned the model, host an inference endpoint for dogbooth on our Amazon EKS cluster.

You can use the RayService custom resource definition (CRD) to deploy a RayCluster with a RayServe application that pulls the dogbooth model from Hugging Face that you pushed earlier via accelerate training script as an output of the fine-tune experiment.

Define Entrypoint for RayService

The RayServe python application is packaged in a container image that can be pulled down for the RayCluster during deployment. Ray documentation  provides a sample code to create an application for inference using Ray Serve and FastAPI .  We tweak the provided python code to pass our custom dogbooth model that was pushed to Hugging Face as model_id by passing an environment variable MODEL_ID to the RayService configuration as shown in the following steps. Review the python application under src/service/dogbooth.py. To introspect the Dockerfile used to build a container image for the RayCluster, the head and worker nodes see src/service/Dockerfile.

Advanced configuration of the RayService is left as an exercise to the reader.
Define RayService

You are now ready to deploy the RayService.  We have provided a ray-service.yaml in the data-on-eks/ai-ml/jark-stack/terraform/src/service directory note in the following manifest ray-service.yaml that:

    Creates a namespace called dogbooth where we deploy the RayCluster.
    Creates an Ingressso that you can expose the RayService endpoint via ingress-nginx out to the AWS Network Load Balancer with path based routing for dashboard and the inference services.
    Edit the MODEL_IDunder runtime_env.env_vars to change the model repository to the one you create during fine-tune.

---
apiVersion: v1
kind: Namespace
metadata:
  name: dogbooth
---
apiVersion: ray.io/v1alpha1
kind: RayService
metadata:
  name: dogbooth
  namespace: dogbooth
spec:
...
  serveConfig:
    importPath: dogbooth:entrypoint
    runtimeEnv: |
      env_vars: {"MODEL_ID": "askulkarni2/dogbooth"}
  rayClusterConfig:
    rayVersion: '2.6.0'
    headGroupSpec:
...
      template:
        spec:
          containers:
            - name: ray-head
              image: $SERVICE_REPO:0.0.1-gpu
              resources:
                limits:
                  cpu: 2
                  memory: 16Gi
                  nvidia.com/gpu: 1
...
    workerGroupSpecs:
      - replicas: 1
...
        template:
          spec:
            containers:
              - name: ray-worker
                image: $SERVICE_REPO:0.0.1-gpu
...
                resources:
                  limits:
                    cpu: "2"
                    memory: "16Gi"
                    nvidia.com/gpu: 1
...
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dogbooth
  namespace: dogbooth
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: "/\$1"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
        - path: /dogbooth/(.*)
          pathType: ImplementationSpecific
          backend:
            service:
              name: dogbooth-head-svc
              port:
                number: 8265
        - path: /dogbooth/serve/(.*)
          pathType: ImplementationSpecific
          backend:
            service:
              name: dogbooth-head-svc
              port:
                number: 8000

Run kubectl apply -f src/service/ray-service.yaml to create the RayService in the dogbooth namespace.

Once applied, the RayCluster’s head node and worker node are scheduled on the GPU nodes and inference endpoint are  available to us via the load balancer’s DNS hostname. Because of the large image size of the GPU based rayproject/ray-ml:2.6.0-gpu base image, this can take up to eight minutes to complete.

Wait for the pods to be up, run  kubectl get pods -n dogbooth –watch and then get the load balancer DNS hostname and explore the Ray Dashboard in the browser.

kubectl get ingress dogbooth -n dogbooth --output jsonpath='{.status.loadBalancer.ingress[0].hostname}'

Open this URL in a browser  http://k8s-ingressn-ingressn-xxx.elb.us-east-1.amazonaws.com/dogbooth/

You can now  view the RayService under the Serve tab on the dashboard.

You can now  view the RayService under the Serve tab on the dashboard.

Finally, verify our dogbooth model deployment with a prompt such as:

 http://k8s-ingressn-ingressn-xxx.elb.us-east-1.amazonaws.com/dogbooth/serve/imagine?prompt=a photo of [v]dog on the beach

Finally, verify our dogbooth model deployment with a prompt such as: http://k8s-ingressn-ingressn-xxx.elb.us-east-1.amazonaws.com/dogbooth/serve/imagine?prompt=a photo of [v]dog on the beach
But what about Argo Workflows?

Glad you asked and no we haven’t forgotten. The previous steps we discussed above are great for experimentation in earlier phases of problem forming and use-case analysis. Once you have found the models that meet the specific goals and you want to deploy them in production, then the Machine Learning Operations (MLOps) approach is used to collaborate between data scientists, developers, operations teams, and domain experts. This collaboration ensures that the model is developed, deployed and managed effectively, meets business goals, and meets operational requirements. This is often where a workflow engine comes into play. Popular options include Kubeflow Pipelines (which are based on Argo Workflows), Apache Airflow, AWS Step Functions, etc. We discuss a similar approach in this post where we present the use of Argo Events and Argo Workflows as a Kubernetes-native workflow engine to orchestrate data processing using Spark. Extending this approach to MLOps, we can start to see how we can build an MLOps platform for GenAI projects. We’ll dive much deeper into this topic and others in upcoming posts and re:Invent workshops.
Cleaning up

To destroy and clean up all the infrastructure created in this blog, simply run the provided ./cleanup.sh script. It will delete the RayService  and run terraform destroy to destroy all the infrastructure resources created.

./cleanup.sh

Conclusion

In this post we showed you the advent of GenAI models, their advantages, key use-cases, and the resource-intensive nature required to create robust outputs. We also spoke about the key advantages that Amazon EKS provides for these use-cases through its built-in scalability, resiliency, and repeatable deployments across environments while enabling customers to have more control, flexibility as well as drive cost effectiveness. We then stepped you through the steps to deploy a GenAI model on EKS utilizing the JARK stack.

AWS also provides you the ability to accelerate ML adoption via the  DataonEKS initiative. The DataonEKS is an open-source project from AWS that provides best practices, benchmark reports, infrastructure as Code (Terraform) templates, deployment examples,  and reference architectures for deploying data workloads on EKS.

In addition, AWS provides managed ML solutions such as Amazon Bedrock and Amazon Sagemaker that can be used to easily deploy and serve existing off-the-shelf FM models or create and run your own models.

GenAI is in a very early stage and evolving at rapid pace with continuous innovations almost everyday. AWS is excited to be your partner to support whatever your GenAI needs maybe with the broadest and most complete set of ML capabilities in the market. We look forward to working with you on your GenAI and ML journey and the road that lays ahead!


