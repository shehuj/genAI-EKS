# genAI-EKS
Deploy Generative AI Models on Amazon EKS

GenAI models represent a significant breakthrough in the field of Artificial Intelligence/Machine Learning, due to its wide ranging applicability along with easy accessibility for non-AI experts. Traditionally, utilizing AI meant creation of a specialized model for each specific use-case, which required a huge amount of compute and human resources each time. GenAI models overcome this bottleneck by creating Foundation Models (FM). FMs allow reuse by providing ability to fine-tune them to be utilized for multiple use-cases without having to build models from the ground up repeatedly. The most popularly used foundational models today utilize transformers (text generation)/diffusers (i.e., image generation) to achieve this adaptability. These models have potential applicability across a wide range of use-cases and industry verticals ranging from chatbots and virtual assistants to generating videos completely via text prompts for marketing.

Solution overview
Architecture for deploying Stable Diffusion Model on Amazon EKS

![Screenshot 2025-01-20 at 15 39 25](https://github.com/user-attachments/assets/e2eefe22-0e41-4925-b22c-b58cd878cf6f)

There is a vast eco-system of tools available to build and run models, even within the kubernetes landscape. One emerging stack on kubernetes is Jupyterhub, Argo Workflows, Ray and Kubernetes. We call this the JARK stack and you can run this entire stack on Amazon EKS

![Screenshot 2025-01-20 at 15 43 30](https://github.com/user-attachments/assets/534002cc-43ae-45c9-9220-5079a6bb16b5)

