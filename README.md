# Home-K8s

Guide to move your self-hosted apps to Kubernetes.  I'm coming directly from Docker Swarm mode after several years of self-hosting successfully.

This guide uses MicroK8s installed on a Ubuntu 20.04 node.

# Kubernetes Cheat Sheet

Create traefik-values.yml file to hold the arguements for Traefik:

```
additionalArguments:
  - "--certificatesresolvers.http-le.acme.email=${EMAIL}"
  - "--certificatesresolvers.http-le.acme.storage=/data/acme.json"
  - "--certificatesresolvers.http-le.acme.caserver=https://acme-v02.api.letsencrypt.org/directory"
  - "--certificatesResolvers.http-le.acme.httpchallenge=true"
  - "--certificatesResolvers.http-le.acme.httpchallenge.entrypoint=web"
  - "--api.insecure=true"
  - "--accesslog=true"
  - "--log.level=WARN"
  - "--serversTransport.insecureSkipVerify=true"
```

helm upgrade --install traefik traefik/traefik --values traefik-values.yml -n traefik


Create a new file to hold all ingress rules for the applications you want Traefik to route to:

New-Item traefik-ingress.yml

kubectl apply -f traefik-ingress.yml

enter:

```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`traefik.${DOMAIN}`) 
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
  tls:
    certResolver: letsencrypt

---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: adguard-https
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`adguard.${DOMAIN}`)
      kind: Rule
      services:
        - name: adguard-home-1637009749
          port: 3000
  tls:
    certResolver: letsencrypt

---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: homeassistant-https
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`homeassistant.${DOMAIN}`)
      kind: Rule
      services:
        - name: home-assistant-1639354586
          port: 8123
  tls:
    certResolver: letsencrypt
```

Deleting resources, method 1: 

To get all the resources.

kubectl get pods,services,deployments,jobs,daemonset,replicasets
 -or-
kubectl get all -n default

Delete the resources, 1 at a time, like below:

kubectl delete deployments <deployment>
kubectl delete services <services>
kubectl delete pods <pods>
kubectl delete daemonset <daemonset>
  
method 2:
  
kubectl get deployments
  
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           9d
  
kubectl delete all -l app=nginx
