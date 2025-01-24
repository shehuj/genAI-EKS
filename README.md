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

