# Devops Take Home Exercise

- YAML kubernetes configuration file:
  - See "production" directory
  - Main file: production/03-deployment.yaml
    - Two HTTP web servers containers on port 80 to keep it easy to test: 
      - Nginx on Docker Host port 30500 and 
      - Apache httpd on Docker Host port 30800

- Multiple environments:
  - Production and staging environments are separated in two different namespaces:
    - namespace: production
    - namespace: staging

- Autoscale command (pay attention to the *-n* line option):
```
  kubectl autoscale deployment httpd --min=1 --max=4 --cpu-percent=70 -n production
  kubectl get hpa -n production      # to watch production namespace
```
  - Or just take a look in production/05-hpa.yaml
    - spec:
      - maxReplicas: 3
      - minReplicas: 1
      - targetCPUUtilizationPercentage: 70

- Rolling
  - Revision limit of 5 to do rollouts (in 03-deployment.yaml file): 
    - revisionHistoryLimit: 5   # in YAML file
  - History command:
```
  kubectl rollout history deployments -n production # or staging
```
  - To rollback:
```
  kubectl rollout undo deployment/httpd -n production # --revision=8 (optional rev number)
```
  - When applying a configuration I prefer to use the "--record" flag in order to have the change-cause described.

- IAM Controls
  - The cluster and its namespace should run with proper Service Account configured
  - The correct ARN should be annotated and should be listed in the "describe sa" command
```
  kubectl describe sa my-serviceAccount
```
    - If the SA is created after the pod, the pod must be re-created.


## Multiple environments:

As stated above there are two directories, one for each namespace. A namespace named *production* 
which deploys a whole environment related to production, its limits, services, and so on. And the
*staging* namespace which is analogous.

  - Files in each directory:
    - 01-namespace.yaml - creates the namespace itself
    - 02-resourceQuota.yaml - resources and their limits
    - 03-deployment.yaml - deployment of two containers and their replicas
    - 04-service.yaml - all services for each replicaset
    - 05-hpa.yaml - horizontalPodAutoscale - to scale the pod using a file

## Scale based on network latency

The solution, according to the official Kubernetes documentation is to make use
of _object metrics_. This is available in the *autoscaling/v2beta2* API version.

## References:

 - [https://kubernetes.io/docs](https://kubernetes.io/docs)
   - [https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
   - [https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
   - [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)

 - [https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html)
