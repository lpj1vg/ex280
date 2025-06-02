# Trabajo de laboratorio: Implementación de aplicaciones empaquetadas

```bash
lab start packaged-review
oc login -u developer -p developer \
  https://api.ocp4.example.com:6443
oc new-project packaged-review
oc new-project packaged-review-prod
helm repo list
helm repo add \
  do280-repo http://helm.ocp4.example.com/charts
helm search repo --versions
oc project packaged-review
helm show values do280-repo/etherpad --version 0.0.6

echo 

helm install test do280-repo/etherpad \
  -f values-test.yaml --version 0.0.6
```
