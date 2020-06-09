# AKS Security Checklist

This list contains hints for security aspects to be considered running Azure Kubernetes Service and corresponding links.
This is not an exhaustive list and it's work in progress. Use this to find areas which you didn't consider yet.

|  | Level | Area | Questions to be asked |Guidence  |Solutions |
| ---- |-----|-----|------|---|--|
|| **Application** *Development* ||
| &#9744; ||Who's allowed to push code?| Restrict permission to only the appropriate group of people and set up proper branching. | [Set up permissions in Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/organizations/security/default-git-permissions?view=azure-devops),[Branching in GitHub](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-branches)<br> 
| &#9744; |||What's the process to review code?|Enforce Pull Requests and Code Reviews to double check code.| [PR Flow in Git](https://docs.microsoft.com/en-us/azure/devops/repos/git/pull-requests-overview?view=azure-devops) using Azure DevOps<br>[PR Flow in Git](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests) using GitHub
|||External Dependencies|
| &#9744; |||Which external dependencies are allowed? |Consider using a Dependency Scanner to scan for vulnerabilities and license violations for your source code and application |  [Whitesource](https://www.whitesourcesoftware.com/whitesource-core/), [GitHub advanced Security](https://github.com/features/security)
| &#9744; |||Where are external dependencies pulled from?|        Consider using a private repository | Use [Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-registry/) with RBAC permissions
|| **Application** *Deployment* ||
| &#9744; || Deployment Identity|    Which identity is being used to run a deployment?| Use a non-human service user, automation account or Service principal to run deployments.| [AppRegistration and SPN on Azure](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals), [Managed Identity on Azure](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)
| &#9744; || Deployment Workflow |     What is the process of a deployment? |Automate your deployment using a build and release pipeline | Set up Azure [Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops) <br> Set up [GitHub Actions](https://help.github.com/en/actions)
| &#9744; || Deployment Workflow |    Can you retrace a deployment? | Use a pipeline which supports traceability | [GitHub Actions](https://help.github.com/en/actions), <br>Azure [Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops)
| &#9744; || Deployment Stages |    Do you have multiple copies of the same infrastructure to be able to test your changes on without affecting a user? | Set up multiple environments with similar configuration using Infrastructure as Code | [Terrafrom](https://www.terraform.io/)<br>[Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
| &#9744; ||  Deployment Infrastructure  |    Who is allowed to make changes in the deployment infrastructure? | Secure your deployment infrastructure | [Azure Resource Locks](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources) <br> [Azure RBAC](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview)
|| **Application** *Runtime* ||
| &#9744; ||Runtime Container Scan | Do the running *containers* contain vulnerabilities or malware?| Install a runtime image scanner and watch behaviour of containers for malicious activities. | [Azure Security Center](https://docs.microsoft.com/en-us/azure/security-center/azure-kubernetes-service-integration)
| &#9744; ||Pulling from Registry|Which registry do you pull images from?| Restrict images to be pulled from specific (private) registries only.<br>Disallow pull from public registries.
| &#9744; || Secrets| Where do you store secrets?    | Use a secure store which stores secret values (like passwords, certificates) outside of your sorce repos and allows access management. | [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/), [AKS Pod Identity](https://github.com/Azure/aad-pod-identity)
| &#9744; ||Limit SysCalls| What is your running application allowed to do?| Limit potential OS calls of your containerized application by restricting syscalls using. | [AppArmour](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-cluster-security#app-armor)
| &#9744; ||Limit Priviliges |  Does your container need advanced privilidges to run? | Forbid advanced priviliges in your container application | [Pod Security Policies](https://docs.microsoft.com/en-us/azure/aks/use-pod-security-policies)
|| **Networking** *cluster-intern* ||
| &#9744; || Networking Policies|   Which components are allowed to talk to each other within your cluster?| Restrict internal traffic to only the pods who should be communicating |[Network Policies](https://docs.microsoft.com/en-us/azure/aks/use-network-policies), [Service Mesh](https://docs.microsoft.com/en-us/azure/aks/servicemesh-about)
| &#9744; || Encryption / MTLS |   Should traffic within your cluster be encrypted? | Encrypt Pod-to-Pod communication | [Service Mesh](https://docs.microsoft.com/en-us/azure/aks/servicemesh-about)
|| **Networking** *cluster-extern* ||
| &#9744; || Public Endpoints|    Does your cluster need public endpoints? | Disallow public endpoints by integrating your cluster into a private VNET| [Internal LoadBalancer](https://docs.microsoft.com/en-us/azure/aks/internal-lb)
| &#9744; || Public Endpoints| Is your public endpoint secured using SSL?|    Setup HTTPS endpoints using an ingress controller or gateway for SSL offload | [NGINX](https://docs.microsoft.com/en-us/azure/aks/ingress-tls)<br>[Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/ingress-controller-expose-service-over-http-https)
| &#9744; || Public API/Master Endpoint |    Do you need a public endpoint for your Kubernetes API?| Restrict access to the master to traffic from a private vnet | [Private Link](https://docs.microsoft.com/en-us/azure/aks/private-clusters)
| &#9744; || Ingress Scan|     Which traffic is allowed to access the cluster? | Limit incoming traffic using a Firewall | [Application Gateway WAF](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-network#secure-traffic-with-a-web-application-firewall-waf), [Application Gateway Ingress Controller WAF](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/annotations.md#azure-waf-policy-for-path)
| &#9744; ||Egress Scan |    Which traffic is allowed to leave the cluster? | Limit egress traffic by locking down your external traffic to specific targets or route via a firewall | [AKS egress](https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic)
| &#9744; ||VPN connectivity |    Do you want to access your cluster using a private network?| Integrate your AKS cluster into your on-prem or cloud hosted exisiting network infrastructure | [Kubenet Networking](https://docs.microsoft.com/en-us/azure/aks/concepts-network#kubenet-basic-networking), [Azure CNI](https://docs.microsoft.com/en-us/azure/aks/concepts-network#azure-cni-advanced-networking)
||**Kubernetes Level**
| &#9744; ||RBAC, Roles & Rolebindings |What should identites be allowed to do within your cluster|Which Define roles within your cluster with a dedicated set of permissions. | [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
| &#9744; ||Identity Management|     How do you mangage users of your cluster?| Bind identites from your identity management system to Kubernets Roles | [Azure AD and AKS](https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac)
||**Infrastructure Level**
| &#9744; ||OS Security Updates| What's your process for installing new OS versions for your nodes?
| &#9744; ||Kubernetes Version Updates|    What's your process for installing new Kubernetes versions| Automate the process of rebooting nodes in case of kernel updates |[Kured](https://docs.microsoft.com/en-us/azure/aks/node-updates-kured)
| &#9744; ||Node Access|     How do you limit remote access to nodes?| Restrict access using Azure RBAC. Access nodes via jumpserver.| [Azure Bastion](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-network#securely-connect-to-nodes-through-a-bastion-host)
||**Organization Level**
| &#9744; || Azure Subscription Management |     Who is allowed to make changes in your Azure subscription / resource group? | Set up RBAC in Azure| [Azure RBAC](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview)
| &#9744; || Multi cluster management |     How can you enforce your organization to follow security guidelines? |Set up Azure Policy to enforce specific rules| [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/policy-for-kubernetes)
| &#9744; || Multi cluster management |How can you get notified in case of violations? |-  Set up [Azure Security Center ](https://docs.microsoft.com/en-us/azure/security-center/azure-kubernetes-service-integration)for Containers<br> - Set up Azure Arc to ensure consistent configuration of multiple clusters| [Azure Security Center ](https://docs.microsoft.com/en-us/azure/security-center/azure-kubernetes-service-integration)<br>[Azure Arc](https://docs.microsoft.com/de-de/azure/azure-arc/kubernetes/)











