<img width="987" height="717" alt="image" src="https://github.com/user-attachments/assets/8d29fbaf-0334-4247-891c-3a80a180d72d" /># Building clusters the declarative way

A Kubernetes cluster is far more than an abstract box with a control plane and worker nodes; for a platform engineer, it's a complex, layered system requiring a full-stack view. The average Kubernetes cluster has more than 20 distinct software elements installed to make up its complete stack.

The full stack of a typical Kubernetes cluster can be understood as a series of layers, working from the bottom up:

Operating system (OS): This is the foundational layer every cluster node runs on. The choice of OS, such as Ubuntu, RHEL, or Talos Linux, is critical as it impacts kernel-level security, performance, patching behaviour, and CVE exposure.

Kubernetes distribution: This layer determines the core features and capabilities of the cluster. Different distributions like K3s, RKE2, or cloud-specific versions like EKS-D are optimised for different uses, such as FIPS compliance for security or a small footprint for the edge.

Container Networking Interface (CNI): The CNI is responsible for how pods communicate with each other. Options like Calico, Cilium, and Flannel directly impact the cluster's performance, security, and ability to enforce network policies.

Container Storage Interface (CSI): This layer manages how storage volumes are provisioned and attached to pods. The choice of CSI, such as EBS, Portworx, or Longhorn, is crucial because different applications have different needs for performance, durability, and data protection.

Ingress controller: This component manages external access to services within the cluster. Tools like NGINX, Contour, or Traefik handle critical functions such as TLS termination, routing, and load balancing. A poor choice can lead to performance bottlenecks or security gaps.

Core platform services: This broad layer includes the essential tools needed to secure, observe, and operate the cluster. It comprises several key functions:

Observability: Metrics (Prometheus, OpenTelemetry, Datadog), logs (Fluent Bit, Loki), and tracing (Jaeger).
Security & policy: Secret management (Vault, Sealed Secrets), identity management (Dex, OIDC), and policy agents (OPA Gatekeeper, Kyverno).
DNS: Such as CoreDNS.
Developer experience tools: This layer includes the tools that developers interact with daily, such as CI/CD agents, GitOps operators (Flux, Argo CD), and API gateways.

Applications and workloads: At the very top of the stack are the microservices, APIs, data pipelines, and other applications that the cluster was built to run. While this is the "reason" the cluster exists, everything underneath must be solid for these workloads to run effectively.

<img width="987" height="717" alt="image" src="https://github.com/user-attachments/assets/8d6677d2-23b7-4d44-a9d3-92cfa531f85d" />

## Different clusters? Different setups!
While standardizing on a single cluster stack is an appealing goal, the reality is not so straightforward. Different kinds of clusters have different needs, dictated by their environment, purpose, and workload.

For example, an AWS cluster uses native services like EBS for storage, while a bare-metal cluster needs alternatives like MetalLB. A production cluster demands hardened security and full observability, whereas a dev cluster might use lightweight proxies to cut costs. Similarly, an ML cluster requires GPU-aware schedulers, unlike a standard web app cluster. This forces platform teams to balance consistency with flexibility.

This diversity, however, is good. In the context of platform engineering, diversity within the open-source ecosystem drives innovation and offers flexibility. This extensibility is what has made Kubernetes so popular and widely adopted, with the Cloud Native Computing Foundation (CNCF) ecosystem as a "buffet of innovation", allowing you to pick and choose the tools that are right for your teams' specific needs.

However, this must be a balancing act between flexibility and consistency. Forcing everyone into a single, rigid model is counterproductive and may cause development teams to "revolt," while too much uncontrolled diversity results in unique "snowflake" clusters that are impossible to support.

Regarding the size of the open-source ecosystem, the sources mention that the CNCF landscape has 1,500 logos on its chart, which illustrates the vast number of projects and products available.


## Keeping dev/test close to production
While dev/test and production environments will never be identical, achieving "behavioral parity" between them is a critical goal for platform engineers. The objective is to make non-production environments look and feel as much like production as possible, ensuring that workloads behave predictably when deployed and that tests are accurate. This helps prevent surprises caused by unintended interactions between different software layers.

To achieve this, teams should aim for consistency in key areas, even if the scale is smaller. This includes using the same base images, CNI, CSI, and Ingress controllers where feasible. It also means applying the same GitOps mechanics, security policies, and scanning tools across all environments. Although a dev environment might have less traffic or fewer security components, maintaining the same fundamental "shape" as production is essential for reliable testing and smoother deployments.

## Beware of unintended interactions
The hardest problem in Kubernetes isn't building the full software stack, but managing the complex interactions between its many layers. A single breaking change in one component (such as the CNI, CSI, or operating system) can cause unintended consequences across the entire system.

For example, a new version of your CNI might break compatibility with an older kernel, or a change in your service mesh configuration could break routing in only one cluster. These issues are notoriously difficult to test and debug, which is why maintaining behavioral parity between dev/test and production environments is crucial for making tests more accurate.

## Achieving full-stack visibility and transparency

Who sees what in the Kubernetes stack? The truth is that different teams across an organization interact with the Kubernetes stack through their own lenses, often seeing only the layers relevant to their roles.

For example, developers typically focus on the CI/CD and application layers, concerned primarily with getting their code into production. Site Reliability Engineers (SREs) live in the observability and workload layers, concentrating on uptime, incident response, and root cause analysis. The security team cares about secrets, network policies, and scanning for CVEs, while leadership, such as a CIO, is concerned with high-level business outcomes like cost, uptime, and overall security exposure.

In contrast, the platform engineering team is often the only group in the business with a true, full-stack view of this complexity.

They are responsible for seeing the entire system - from the OS and CNI up to the developer tools - and ensuring it works as a cohesive unit across all clusters and environments. This unique perspective means platform engineers must define what is in the stack, assign ownership of different components to relevant teams, and manage how the entire system is deployed, updated, and kept healthy.

## The politics of infra and platform teams

With so many stakeholders involved in the stacks, it's inevitable for tensions between between traditional infrastructure teams and modern platform teams to not shape up - especially around who owns provisioning.

Legacy infrastructure teams, focused on stability and control, typically own the provisioning of bare metal, VMs, and networks, often working through manual ticket-based systems over weeks-long cycles. Conversely, platform teams prioritize speed, developer self-service, and automation.

This fundamental conflict creates friction, as you cannot build a fast, declarative platform on a foundation of manual processes and tribal knowledge.

Platform engineers must navigate this relationship carefully. The solution involves building bridges, aligning on shared outcomes like security, documenting dependencies, and treating the underlying infrastructure as a programmable API. This approach helps reconcile the "generational drift" between these two worlds, ensuring both teams can contribute to business goals effectively.

<img width="1107" height="455" alt="image" src="https://github.com/user-attachments/assets/ab96f60e-c3e1-4e03-bfb9-d0e13b540b69" />

## How do you build a Kubernetes cluster?

There is no single right way to build a Kubernetes cluster; the method you choose depends on your specific needs, trade-offs, and production considerations. The landscape of tooling is broad, ranging from manual learning exercises to fully managed platforms.

Here are the common approaches:

Manual & learning approaches: For understanding the fundamentals, "Kubernetes the Hard Way" by Kelsey Hightower provides a deep, manual dive into every component, from TLS certificates to systemd configs. The official kubeadm tool is a step up, bootstrapping a basic cluster but leaving infrastructure provisioning and full lifecycle management to you.

Infrastructure-as-Code (IaC) tools: Tools like Terraform, Pulumi, and Kubespray offer more automation for provisioning. However, they can be fragile at scale and typically lack support for Day 2 operations like upgrades and lifecycle management, essentially acting as "fire and forget" installers.

Managed services: Cloud providers offer popular services like EKS, AKS, and GKE, which reduce operational overhead by managing the control plane for you. You are still responsible for worker nodes, security, observability, and other add-ons.

Declarative fleet management: For managing many clusters at scale, Cluster API (CAPI) is a powerful, Kubernetes-native solution. It treats clusters as code, automating the entire lifecycle from creation to deletion declaratively.

Specialized use cases: vCluster - this approach doesn't build a true cluster but provides a "Kubernetes cluster, like experience" by running fast, lightweight, and isolated virtual clusters inside a real one. It reuses the host cluster's control plane, saving significant time and overhead. This makes vClusters ideal for ephemeral environments like CI/CD, testing, and development.

Bootable images: For edge or air-gapped environments, tools like Kairos can create self-contained, bootable OS images with embedded cluster definitions.

## How do you get the full payload deployed?

Many of the approaches to building K8s clusters don’t actually deploy all of it for you. They give you the control plane and worker nodes and stop there. To deploy the full payload - including the essential observability, security, and policy stacks - several approaches exist.

You can use push-based pipelines with Infrastructure-as-Code tools like Terraform or Ansible to install packages after the cluster is created. Alternatively, a pull-based GitOps approach with Flux or Argo CD allows the cluster to reconcile its state from a versioned source of truth, making it self-healing and auditable. Other options include baking everything into a custom installer image or using a full-stack management platform.

## Intentionality is key
Don’t just layer tools on top of tools: mature platform teams use a strategic mix.

<img width="1162" height="427" alt="image" src="https://github.com/user-attachments/assets/37ecb864-66aa-4b28-9b0b-6796a24191a7" />

