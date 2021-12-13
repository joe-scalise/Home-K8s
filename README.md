# Work in Progress

# Home-K8s

Guide to move your self-hosted apps to Kubernetes.  I'm coming directly from Docker Swarm mode after several years of self-hosting successfully.

This guide uses MicroK8s installed on a Ubuntu 20.04 node.

# Kubernetes Cheat Sheet

To enable Let's Encrypt, create traefik-values.yml file to hold the arguements for Traefik:

```
additionalArguments:
  - --certificatesresolvers.letsencrypt.acme.tlschallenge=true
  - --certificatesresolvers.letsencrypt.acme.email=youremail@domain.com
  - --certificatesresolvers.letsencrypt.acme.storage=/data/letsencrypt.json
```

helm upgrade --install traefik traefik/traefik --values traefik-values.yml -n traefik

Verify the manifest updates the values above:

helm get manifest traefik -n traefik

Also, verify in the logs that you don't see any issues with Let's Encrypt:

kubectl logs $(kubectl get pod -n traefik -o name) -n traefik

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
  
## Execute inside the container, looks similar to Docker:
  
`kubectl exec -it $(kubectl get pod -n traefik -o name) -n traefik -- ls /data`
