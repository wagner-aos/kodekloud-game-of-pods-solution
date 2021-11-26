
### Pento

Troubleshooting Nodes and Deploying File Server

![](images/pento-architecture.png)

1. The Game of Pods cluster is Broken! Troubleshoot, fix the cluster issues and then deploy the below architecture to unlock our Image Gallery!

Master node: coredns deployment has image: 'k8s.gcr.io/coredns:1.6.7'
Fix kube-apiserver. Make sure its running and healthy.
kubeconfig = /root/.kube/config, User = 'kubernetes-admin' Cluster: Server Port = '6443'

***Master Node***

a. Change the parameter --client-ca-file=/etc/kubernetes/pki/ca-authority.crt of kube-apiserver static pod to the right certificate authority path --client-ca-file=/etc/kubernetes/pki/ca.crt.

```sh
    vi /etc/kubernetes/manifests/kube-apiserver.yaml    
```
b. Change the image of CoreDNS Deployment to k8s.gcr.io/coredns:1.6.7 
```sh
    k edit deployment coredns -n kube-system    
```

***Worker Node***

c. 
```sh
    
```


2.

```sh

```

3.

```sh

```

4.

```sh

```

5.

```sh

```

6.

```sh

```


