## INSTALACIÓN K3D

```bash
# Current last release
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Specific release
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | TAG=v5.0.0 bash

# Autocompletado
k3d completion bash | sudo tee /etc/bash_completion.d/k3d > /dev/null
```

## KUBECTL

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Validar el binario
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

# Instalar kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

# Autocompletado
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
```

## INSTALACIÓN DE HELM2

```bash
# helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# autocompletado
helm completion bash | sudo tee /etc/bash_completion.d/helm > /dev/null
```

## OLM

```bash
curl -L https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh -o install.sh
chmod +x install.sh
./install.sh v0.31.0
```

## K3D

Despliegue de Strimzi con olm:

```bash

k3d cluster create kafka-dev --servers 1 --agents 4 \
  --k3s-node-label "topology.kubernetes.io/zone=zona-a@agent:0" \
  --k3s-node-label "topology.kubernetes.io/zone=zona-a@agent:1" \
  --k3s-node-label "topology.kubernetes.io/zone=zona-b@agent:2" \
  --k3s-node-label "topology.kubernetes.io/zone=zona-b@agent:3" \
  --k3s-arg "--disable=traefik@server:0" \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer"


# NGINX ingress con ssl passthrough
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.ingressClassResource.default=true \
  --set controller.extraArgs.enable-ssl-passthrough=""

# IPs para nip.io
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)
/k3d-cluster-k3d-serverlb - 172.18.0.3 <--
/k3d-cluster-k3d-server-0 - 172.18.0.2

# Instalación de olm
curl -L https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.31.0/install.sh -o install.sh
chmod +x install.sh
./install.sh v0.31.0

# Instalación de operador Strimzi en el namespace kafka
export NAMESPACE="kafka"
kubectl create namespace $NAMESPACE
kubectl apply -f operatorgroup.yml -n $NAMESPACE
kubectl apply -f subscription-strimzi-0.42.0.yaml
kubectl get installplans -n operators

kubectl patch installplan install-wj65z -n operators --type merge --patch '{"spec":{"approved":true}}'
kubectl patch installplan <installplans_name> -n operators --type merge --patch '{"spec":{"approved":true}}'
kubectl get csv -n operators
kubectl apply -f kafka-kraft-jbod.yml -n $NAMESPACE

# Conexión externa TLS
kubectl get secret kafka-cluster-lab-cluster-ca-cert -n $NAMESPACE -o jsonpath='{.data.ca\.crt}' | base64 -d > ca.crt
keytool -import -trustcacerts -alias root -file ca.crt -keystore truststore.jks -storepass password -noprompt
kubectl get secret kafka -n $NAMESPACE -o jsonpath='{.data.password}' | base64 -d

# Verificar SSL Passtrought
kubectl get ingress
NAME                                CLASS   HOSTS                         ADDRESS      PORTS     AGE
kafka-cluster-lab-broker-0          nginx   broker-0.172.18.0.3.nip.io    172.18.0.2   80, 443   3m57s
kafka-cluster-lab-broker-1          nginx   broker-1.172.18.0.3.nip.io    172.18.0.2   80, 443   3m57s
kafka-cluster-lab-broker-2          nginx   broker-2.172.18.0.3.nip.io    172.18.0.2   80, 443   3m57s
kafka-cluster-lab-broker-3          nginx   broker-3.172.18.0.3.nip.io    172.18.0.2   80, 443   3m57s
kafka-cluster-lab-kafka-bootstrap   nginx   bootstrap.172.18.0.3.nip.io   172.18.0.2   80, 443   3m57s

# Obetenemo cert y lo decodificamos https://online-ssl-certificate-decoder.syshunt.com/
openssl s_client -connect bootstrap.172.18.0.3.nip.io:443 -servername bootstrap.172.18.0.3.nip.io

-----BEGIN CERTIFICATE-----
MIIGfDCCBGSgAwIBAgIUaiFhY81ccV5NXQc/4oKJNN3hpj0wDQYJKoZIhvcNAQEN
BQAwLTETMBEGA1UECgwKaW8uc3RyaW16aTEWMBQGA1UEAwwNY2x1c3Rlci1jYSB2
MDAeFw0yNTA1MTMwNzQzMjhaFw0yNjA1MTMwNzQzMjhaMDcxEzARBgNVBAoMCmlv
LnN0cmltemkxIDAeBgNVBAMMF2thZmthLWNsdXN0ZXItbGFiLWthZmthMIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsKqExj4IKowrFfQrgid7Fv2r5Soi
a4ondt9K+xBCDHZ9NQupRsxZXkhk01lKTOOhWS36Amj8DpjrLAGf++qQUNZatq83
zH+qKaqPFgF3Apcsl+vPY+EUkqk2NaIqPhbpsCj2K1auzhR8CQtw7RPfUe6OTQSi
b87asPe/bH57cgrRtEhbSjCmdt5FnV3WSsgYvn2CdwUU2WpicB0jqlPqyzuJ8fW0
a+zSqcP/uABmQ9ETrFWDq7qP/Ry4o7igRITH1aot/XQqexHsP7QWTy3fdxfu9Ca4
NliU1F44LLSU9bRzT3GIN/JFbES2SeUGImAjesN94WEWj+EByt0het4j0QIDAQAB
o4ICiDCCAoQwggJABgNVHREEggI3MIICM4I3a2Fma2EtY2x1c3Rlci1sYWIta2Fm
a2EtYnJva2Vycy5rYWZrYS5zdmMuY2x1c3Rlci5sb2NhbIJSa2Fma2EtY2x1c3Rl
ci1sYWItYnJva2VyLTIua2Fma2EtY2x1c3Rlci1sYWIta2Fma2EtYnJva2Vycy5r
YWZrYS5zdmMuY2x1c3Rlci5sb2NhbIIfa2Fma2EtY2x1c3Rlci1sYWIta2Fma2Et
YnJva2Vyc4Ira2Fma2EtY2x1c3Rlci1sYWIta2Fma2EtYm9vdHN0cmFwLmthZmth
LnN2Y4IaYnJva2VyLTIuMTcyLjE4LjAuMy5uaXAuaW+CRGthZmthLWNsdXN0ZXIt
bGFiLWJyb2tlci0yLmthZmthLWNsdXN0ZXItbGFiLWthZmthLWJyb2tlcnMua2Fm
a2Euc3ZjgiFrYWZrYS1jbHVzdGVyLWxhYi1rYWZrYS1ib290c3RyYXCCOWthZmth
LWNsdXN0ZXItbGFiLWthZmthLWJvb3RzdHJhcC5rYWZrYS5zdmMuY2x1c3Rlci5s
b2NhbIIbYm9vdHN0cmFwLjE3Mi4xOC4wLjMubmlwLmlvgilrYWZrYS1jbHVzdGVy
LWxhYi1rYWZrYS1icm9rZXJzLmthZmthLnN2Y4Ina2Fma2EtY2x1c3Rlci1sYWIt
a2Fma2EtYm9vdHN0cmFwLmthZmthgiVrYWZrYS1jbHVzdGVyLWxhYi1rYWZrYS1i
cm9rZXJzLmthZmthMB0GA1UdDgQWBBTCl0EXCiyIjmbL3xs/MCCD6wQBZjAfBgNV
HSMEGDAWgBRtJecQca6oYMf8pI+nPdXewMEuiTANBgkqhkiG9w0BAQ0FAAOCAgEA
HzeAtglmt5VTq0yowE+VUJS0tJBTP6kBYESX9hD1Q0fG1xCHG3U95WlUezFKG9Yt
TDjN6o6mR/mTuaA7g6uWZXQddZMIPWnkhQxIsjUMkvXpP8BM2MFHa8n5jC4HqYm5
iZWl22GwANc9IzSccc2Ixw/FHzYZuCSWR+bx3nnuOF09VpBVzjrMPrkcKgDNtexk
6inoPsdhkYhvul9hWfh/B6upqM8Ll2sGgW2JWqTgyxvfAw0Hi/sUHj6I/FQA+Boj
Vbn22TjsaW8+cs/FrBYesD/ASCAFX0oHZ6dh6pGsi+wGtdfnCc3KydwKsuv2zZZL
5auCjt+6OUevc2TcpFnFpFCBN69SYYTsarDmPBjVBNrOQepo6Ch2hXe2+aKBNAww
L4qE6uY1hqh72FXy3twijmohodSzhr1Vpxpqjc+wv9F31sZ4KAqcryuJ3kUn0xHw
Mu9uJRug+Geq6t4xoBduxFmB1xk5RNvune+elVendd/NXIpnq/+sMlCqW8L6OjOF
CISdYHu5BpXtel1K3b31zXAURuP3IUdriK2TC1Dh5w9/nPme1cMM9z0xRQQIWDKp
01JBGWi8yWvXGdqIL4QsoyhcapvHeH5szZu7IIQpjvfV+aujy9MGQkOTkKZYPEq7
IpF0Ugg7FnDhJY5K0g7XHB7+Gxu7YhUdpOJ0wtw8vho=
-----END CERTIFICATE-----
```
