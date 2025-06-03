# Configuración de IdP HTPasswd en OpenShift

## 1. Gestión de usuarios htpasswd

- Agrega/actualiza usuarios:

```sh
htpasswd -c -b -B tmp_users user01 'password'
htpasswd -b -B tmp_users user02 'password'
htpasswd -b -B tmp_users user03 'password'
```

## 2. Configuración del IdP en OpenShift

- Crea el secreto con los usuarios:

> Buscar `htpasswd CR` en documento de networking.

```sh
oc create secret generic htpasswd-secret \
  --from-file htpasswd=tmp_users \ # KEY:htpasswd
  -n openshift-config              # NAMESPACE: openshift-config
```

- Edita el recurso OAuth:

```yaml
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: htpasswd-secret
    mappingMethod: claim
    name: htpasswd
    type: HTPasswd
```

- Aplica los cambios:

```sh
oc replace -f oauth.yaml
watch oc get pods -n openshift-authentication
```

## 3. Roles y permisos

- Asigna cluster-admin a un usuario:

```sh
oc adm policy add-cluster-role-to-user cluster-admin new_admin
```

- Elimina la capacidad de crear proyectos a todos:

```sh
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
```

- Permite crear proyectos solo a un grupo:

```sh
oc adm groups new managers
oc adm policy add-cluster-role-to-group self-provisioner managers
```

- Asigna roles en un proyecto:

```sh
oc adm groups add-users developers new_developer
oc policy add-role-to-group edit developers -n <project-name>
oc adm groups new qa
oc adm groups add-users qa tester
oc policy add-role-to-group view qa -n <project-name>
```

## 4. Limpieza

- Elimina el usuario kubeadmin (tras verificar acceso con IdP):

```sh
oc delete secret kubeadmin -n kube-system
```
