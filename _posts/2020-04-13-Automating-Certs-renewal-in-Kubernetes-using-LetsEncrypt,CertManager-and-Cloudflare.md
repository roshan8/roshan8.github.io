### What’s Lets-Encrypt? 
<p> Let's Encrypt is a non-profit certificate authority run by Internet Security Research Group (ISRG) that provides X.509 certificates for Transport Layer Security (TLS) encryption at `no charge`. The certificate is `valid for 90 days`, during which renewal can take place at any time. </p> 

### What’s Cert-Manager? 
<p>[cert-manager](https://github.com/jetstack/cert-manager) is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources(Lets-Encrypt is one of these sources). It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry. Cert-manager is based on [kube-lego](https://github.com/jetstack/kube-lego) 

### Why did we choose `DNS as challenge type` for Let’s Encrypt
Let’s Encrypt supports various types of [challenges types](https://letsencrypt.org/docs/challenge-types/), We chose to use DNS-01 challenge instead of more popular HTTP-01 challenge for the following reasons
DNS challenge type supports wildcard certificates. HTTP-01 challenge issue certificate for every subdomain and that can cause problems sometimes (Too many lets-encrypt calls when your infra grows, Your API calls can get rate-limited by Let’s Encrypt )
It works well even if you have multiple web servers. 

### So, Let’s begin!

Create cert-manager namespace
```sh
kubectl create namespace cert-manager
```
Install cert-manager
```sh
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.11.1/cert-manager.yaml
```
Get the Global token from [Cloudflare](https://support.cloudflare.com/hc/en-us/articles/200167836-Managing-API-Tokens-and-Keys) 
Get base64 string from Cloudflare global token key using the following command
```sh
echo -n "YourGlobalTokenHere" | base64
```
Create file `create-secret.yaml`
```sh
---
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-key-secret
  namespace: cert-manager
type: Opaque
data:
  cloudflare-api-key: EnterBase64StringFromPreviousStep

```
Create file `prod-cluster-issuer.yaml`
	```sh
---
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
 labels:
   name: letsencrypt-prod
 name: letsencrypt-prod
spec:
 acme:
   email: youremailid@example.com   # Please enter a valid email id
   server: https://acme-v02.api.letsencrypt.org/directory
   privateKeySecretRef:
     name: letsencrypt-prod
   solvers:
   - selector: {}    
     dns01:
        cloudflare:
          email: it@example.com    
          apiKeySecretRef:
            name: cloudflare-api-key-secret
            key: cloudflare-api-key
``` 
Create `prod-cluster-certificate.yaml`
	```sh
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: example-com
  namespace: default
spec:
  secretName: example-prod-gateway-tls
  duration: 2160h # 90d
  renewBefore: 360h # 15d
  commonName: '*.example.com'
  dnsNames:
  - '*.example.com'
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```
Let’s create a sample nginx-ingress file. (expose.yaml)
```sh
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-prod-gateway
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/tls-acme: "true"
    nginx.org/server-snippets: "gzip on;"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - host: test.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: dex
              servicePort: 80
    - host: trial.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: hello-world
              servicePort: 80
  tls:
    - secretName: example-prod-gateway-tls
      hosts:
        - test.example.com
        - trial.example.com

```
kubectl create -f expose.yaml
kubectl create -f create-secret.yaml
kubectl create -f prod-cluster-issuer.yaml
kubectl create -f prod-cluster-certificate.yaml

Cert-manager should be able to get new certificates from Lets-Encrypt within 2 mins.
