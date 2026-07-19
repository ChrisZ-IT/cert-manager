# cert-manager
My homelab deployment of cert manager

I just followed the [AWS + LoadBalancer + Let's encrypt](https://cert-manager.io/docs/tutorials/getting-started-aws-letsencrypt/) tutorial and just adjusted for self hosted k8s

I installed the base service via the helm install. So installing cert-manager to your cluster is a pre-req.
Im also using cloudflare for my domain registration/public DNS. Adjust deployment if you use something else

1. Install cert-manager (updating the version to your required version)
```
helm install cert-manager \
    oci://quay.io/jetstack/charts/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --version v1.21.0 \
    --set crds.enabled=true
```
2. Wait for pods in `cert-manager` namespace to enter `running` state
3. Deploy issuers.
```
export DOMAIN_NAME=domain.local
export CLOUDFLARE_TOKEN=<Enter Your API Token>

# Create secret with cloudflare api token
envsubst < secret.yml | k apply -f -

# Create issuer for self signed certs. Useful for testing
k apply -f selfsigned.yml


# Deploy lets encrypt staging issuer. Great for testing lets encrypt before you gen prod certs
envsubst < staging_issuer.yml | k apply -f -

# Deploy prod lets encrypt issuer. Full ssl validation for end to end encryption
envsubst < prod_issuer.yml | k apply -f -
```

4. Optional: Create certs for services outside of K8s using cert-manager
See my examples of opnsense and synology certificate resources for generating lets encrypt certs in k8s

Example of exporting a cert from k8s so it can be used. in opnsense for this example
```
k get secret -n istio-ingress opnsense-cert-request -o jsonpath='{.data.tls\.crt}' | base64 --decode > server.crt
k get secret -n istio-ingress opnsense-cert-request -o jsonpath='{.data.tls\.key}' | base64 --decode > server.key

# Then import into opnsense via its gui or api
```


I will be creating some automation for cert rotation on those platforms. Since Im using awx it will probably just be a scheduled ansible job.