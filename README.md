#
This example is based on traefiklabs' [Secure Web Applications with Traefik Proxy, cert-manager, and Let’s Encrypt](https://traefik.io/blog/secure-web-applications-with-traefik-proxy-cert-manager-and-lets-encrypt/). The main change is configuring `NodePort` on the Traefik service to expose it on the server.


# Using Cloudflare

This example uses Cloudflare.  

For the DNS entry, Proxy Status had to be DNS only.

Create an API token, per the [cert-manager docs](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/):
* My Profile > API Tokens > Create Token
* Create Custom Token
* Add Permissions
 * Zone > Zone > Read
 * Zone > DNS > Edit
* Zone Resources: Include > All Zones
* Save and note your API key


# To deploy 

* `kindnodes.yml` from the `wt-k8s` repo
* Deploy Traefik with `NodePort` via `traefik.values.yml`
* Deploy cert-manager
* Deploy whoami service via `whoami.yml`
* Deploy ingress via `ingress.yml`


```
cluster --name=tns-multi-test --config=kindnodes.yml
helm install traefik traefik/traefik -f traefik.values.yml 
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.10.1/cert-manager.yaml
kubectl apply -f whoami.yml 
kubectl apply -f ingress.yml 
```

```
kubectl -n whoami get issuer -o wide
NAME             READY   STATUS                                                 AGE
le-example-dns   True    The ACME account was registered with the ACME server   56s

# DNS propagation can take a few minutes. Check for errors in cert-manager logs:
kubectl logs -n cert-manager cert-manager-6bbf5c9c95-btfdn

kubectl -n whoami get certificateRequest -o wide
kubectl -n whoami get certificateRequest -o wide
NAME                           APPROVED   DENIED   READY   ISSUER           REQUESTOR                                         STATUS                                         AGE
tls-whoami-ingress-dns-l45cs   True                True    le-example-dns   system:serviceaccount:cert-manager:cert-manager   Certificate fetched from issuer successfully   2m19s

kubectl -n whoami get certificates
NAME                     READY   SECRET                   AGE
tls-whoami-ingress-dns   True    tls-whoami-ingress-dns   2m33s

curl -k -H "Host: whoami.k8s.coldframe.org" https://localhost:443
Hostname: whoami-76c79d59c8-2ml6j
IP: 127.0.0.1
IP: ::1
IP: 10.244.2.4
IP: fe80::30f9:4fff:fe94:cad0
RemoteAddr: 10.244.1.2:51918
GET / HTTP/1.1
Host: whoami.k8s.coldframe.org
User-Agent: curl/7.81.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.19.0.4
X-Forwarded-Host: whoami.k8s.coldframe.org
X-Forwarded-Port: 443
X-Forwarded-Proto: https
X-Forwarded-Server: traefik-77f8d8ff7-wmtlf
X-Real-Ip: 172.19.0.4
```
