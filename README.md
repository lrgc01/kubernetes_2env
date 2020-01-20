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
  - Files for production under "production" directory
  - Files for staging under "staging" directory

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

- The rolling history
  - Revision limit of 5 to do rollouts (in 03-deployment.yaml file): 
    - revisionHistoryLimit: 5   # in YAML file
  - History command:
```
  kubectl rollout history deployments -n production # or staging
```
  - When applying a configuration I prefer to use the "--record" flag

## Scale based on network latency

The solution, according to the official Kubernetes documentation is to make use
of _object metrics_. This is available in the *autoscaling/v2beta2* API version.

## References:

 - [https://kubernetes.io/docs](https://kubernetes.io/docs)
   - [https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
   - [https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
   - [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#autoscaling-on-multiple-metrics-and-custom-metrics)
