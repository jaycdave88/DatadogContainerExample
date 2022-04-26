# DatadogContainerExample
## Information on application 
- Runtime & Version: .NET 6 
- OS: Debian 

## Datadog Agent Setup steps: 
 Create secrets for the Datadog Agent. The values can be found within your Datadog instance. 
    
    kubectl create secret generic datadog-agent --from-literal api-key=<DD_API_KEY> --from-literal app-key=<DD_APP_KEY>
 
 Install the Datadog Linux Agent via Helm 
 _Install Helm_
 
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \ chmod 700 get_helm.sh \ ./get_helm.sh

Create a new file called `values.yaml`& within the new file set the following: 

    datadog:
     clusterName: <CLUSTER-NAME>
     apiKeyExistingSecret: datadog-agent
     apm:
      portEnabled: true
    logs:
      enabled: true
      containerCollectAll: true
    processAgent:
      processCollection: true
    dogstatsd:
      useHostPort: true
    kubeStateMetricsCore:
      enabled: true
    kubeStateMetricsEnabled: false
    kubelet:
      tlsVerify: false
    networkMonitoring:
      enabled: true
    securityAgent:
      compliance:
        enabled: true
      runtime:
        enabled: true
        syscallMonitor:
          enabled: true
    agents:
    # This toleration allows the agent to be deployed to every node.
    tolerations:
      - operator: Exists

Add the Datadog Helm repository: 

    helm repo add datadog https://helm.datadoghq.com
    
Fetch the latest version of the newly added charts: 
    
    helm repo update 
 
 Create the namespace 
    
    kubectl create namespace <NAMESPACE>
    Example: kubectl create namespace datadog-agent
    
Deploy the Datadog Agent
    
    helm install <RELEASE_NAME> -n <NAMESPACE> -f datadog-values.yaml --set datadog.site='datadoghq.com' datadog/datadog
    
Verify that the Datadog Agent pods are running by the following command: 

    kubectl get pods --namespace <NAMESPACE>
    Example: kubectl get pods --namespace datadog-agent
    
## Deploy the .NET application 

The Datadog .NET APM Library must be installed in the application container (i.e. Dockerfile) in order to bind the the application process. Please reference the Dockerfile in this GH repository to see how that is done. 

Deploy the application pod

    kubeclt create -f <APPLICATION_YAML>
    Example: kubectl create -f dotnet-app.yaml

Verify the application pod is running: 

    kubectl get po | grep dotnet

## Troubleshooting
If you are updating the applicaiton manifest with a newer version: 

    kubectl delete -f dotnet-app.yaml 
    kubectl create -f dotnet-app.yaml 

Check the Load Balancer external IP: 

    kubectl get svc 

To check the `ContainerPort` and `HostPort`: 

    kubectl describe pod <POD_NAME>
    Example: kubectl describe pod dotnetapp-55f55dbbft-74cnf
    

    
    
