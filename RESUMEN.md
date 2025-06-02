# TOPIC de examen

- [ ] Configurar htpasswd como proveedor de identidades
- [ ] 

## HTPasswd

> web console Cluster Settings > Global Configuration > OAuth

```bash
# `-B` 
hspasswd -c -B -b </path_to/htpasswd_file> <user_name> <passord>
hspasswd    -B -b </path_to/htpasswd_file> <user_name> <passord>

oc create secret generic htpass-secret --from-file=htpasswd=</path_to/htpasswd_file> -n openshift-config

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders
    - name: my_htpasswd_provider
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpass-secret

oc aplly -f <path_to/CR>
oc login -u <username>
oc whoami
```