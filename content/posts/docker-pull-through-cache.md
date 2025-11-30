---
title: "Docker Pull Through Cache"
date: 2023-05-11T12:03:59+03:00
draft: false
---
# Local Deployment

DockerHub has a fair-use policy which limits the number of pull requests. The deployment of a docker pull-through registry in local/cloud enviroment woule help in reducing the total number of image requests, thus mitigate the stress of running out of the docker fair-use allowance.

An official practice of deploying a docker registry can be found through this link: https://docs.docker.com/registry/deploying/

## Official Registry Image

Docker provides an official image which acts as a registry out of the box.

https://hub.docker.com/_/registry

## HTTPS & Certs

Insecure docker registry can leads to great problems. Hence it's highly recommended to enable HTTPS support. For local deployment, one can use self-signed certificates for dev/testing.

### Generate Self-signed Certificates

https://docs.docker.com/registry/insecure/#use-self-signed-certificates


# AWS EKS Settings

## PV & PVC


```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cache-pv
  namespace: cluster-registry
  labels:
    app: cache
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
allowVolumeExpansion: true

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cache-pvc
  namespace: cluster-registry
  labels:
    app: cache
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeMode: Filesystem
  storageClassName: cache-pv
```


## Official Local Docker Registry Image


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cache-pod
  namespace: cluster-registry
  labels:
    app: cache
spec:
  containers:
  - name: cache-containers
    image: registry:2
    resources:
      limits:
        memory: "512Mi"
        cpu: "256m"
    ports:
      - containerPort: 5000
        name: cache-port
    volumeMounts:
      - name: cache-volumn
        mountPath: /var/lib/registry
  volumes:
    - name: cache-volumn
      persistentVolumeClaim:
        claimName: cache-pvc

```

## Ingress
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

![Ingress Workflow](https://d33wubrfki0l68.cloudfront.net/91ace4ec5dd0260386e71960638243cf902f8206/c3c52/docs/images/ingress.svg)
Prerequisites:
> An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cache-ingress
  labels:
    app: cache
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: cache-service
                port:
                  number: 8888
```

## Service


```yaml
apiVersion: v1
kind: Service
metadata:
  name: cache-service
  namespace: cluster-registry
  labels:
    creator: kai
    vpc: d2
    cluster: metaplay-d2
    app: cache
  annotations:
  	alb.ingress.kubernetes.io/target-type: instance # https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
    # service.beta.kubernetes.io/aws-load-balancer-internal: "true" # route internal traffic
spec:
  selector:
    app: cache
  ports:
  - name: cache-service
    port: 5000
    targetPort: cache-port
  type: NodePort
```