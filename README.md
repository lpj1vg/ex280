
## Acceso a consola web

En primer lugar es necesario autenticarse en `cli`:

```bash
oc login -u developer -p developer https://api.ocp4.example.com:6443
```
Localiza la url con `oc whoami --show-console`

## Tipos de usuarios

- Usuarios regulares: objetos `user`
- Usuarios de sistema
- Cuentas de servicio: Objetos `ServiceAccount`

## Operadores

- CVO (Cluster Version Operator): `clusteroperators`
- OLM (Operator Lifecycle Manager): `operators`

## Habilitar las sesiones permanentes para las aplicaciones web `session affinity`

```bash
oc annotate ingress <ingress_name> \
  ingress.kubernetes.io/affinity=cookie
```

El Ingress puede usar cookies para asegurar que las peticiones de un cliente se dirijan siempre al mismo pod, lo que permite mantener sesiones persistentes cuando el estado se almacena localmente.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <ingress_name>
  annotations:
    ingress.kubernetes.io/affinity: "cookie" # <-- Aquí está la anotación

spec:
  rules:
  # ... tus reglas de Ingress aquí
```

## Asignar variables de entorno a un deployment

```bash
oc set env deployment/mysql-app \
  MYSQL_USER=redhat \
  MYSQL_PASSWORD=redhat123 \
  MYSQL_DATABASE=world_x
```

Listar las variables de entorno de un pod:

```bash
oc set env pod/<pod-name> --list
```

## Creación de secretos

Cree un secreto genérico que contenga pares de clave-valor de valores literales escritos en la línea de comandos:

```bash
oc create secret generic secret_name \
  --from-literal key1=secret1 \
  --from-literal key2=secret2
```

Cree un secreto genérico con los nombres de clave especificados en la línea de comandos y los valores de los archivos:

```bash
oc create secret generic ssh-keys \
  --from-file id_rsa=/path-to/id_rsa \
  --from-file id_rsa.pub=/path-to/id_rsa.pub
```

Cree un secreto TLS que especifique un certificado y la clave asociada:

```bash
oc create secret tls secret-tls \
  --cert /path-to-certificate \
  --key /path-to-key
```

### Añadir un secreto como volumen a un deployment

```bash
oc set volume deployment/demo \
--add --type secret \
--secret-name secret_name \
--mount-path /app-secrets
```

Se crean en el `/app-secrets` tantos ficheros con nombre la KEY y contenido el VALUE.

## Creación de ConfigMaps

```bash
oc create cm my-config \
--from-literal key1=config1 \
--from-literal key2=config2 \
--from-literal key3=config3
```

### Añadir un ConfigMap (cm) como volumen a un deployment (dc)

```bash
oc set volume deployment/demo \
--add --type configmap \
--configmap-name demo-map \
--mount-path /app-secrets
```

## Actualización de los secretos y mapas de configuración

```bash
# Extraemos los secretos
oc extract secret/demo-secrets \
--to /tmp/demo --confirm

# Modificamos el fichero `/tmp/demo`

# Actualizamos el secreto o cm
oc set data secret/demo-secrets \
-n <nombre-projecto> \
--from-file /tmp/demo
```

Debe reiniciar los pods que usan variables de entorno para que los pods lean el mapa secreto o de configuración actualizado. Los pods que usan un montaje de volumen para hacer referencia a secretos o mapas de configuración reciben las actualizaciones sin un reinicio.

## Uso de istag (ImageStreamTags)

- Configure su proyecto para que sus cargas de trabajo se refieran a la imagen de la base de datos con el nombre corto **mysql8:1**.
- El nombre corto debe apuntar a la imagen del contenedor **registry.ocp4.example.com:8443/rhel9/mysql-80:1-228**. Se espera que el nombre de la imagen de la base de datos y su registro de origen cambien en un futuro cercano y usted desea aislar sus cargas de trabajo de ese cambio.
- La configuración del aula copió la imagen del Red Hat Ecosystem Catalog. La imagen original es **registry.redhat.io/rhel9/mysql-80:1-228**.

```bash
oc create istag mysql8:1 \
  --from-image registry.ocp4.example.com:8443/rhel9/mysql-80:1-228

oc set image-lookup mysql8

oc set image-lookup
NAME    LOCAL
mysql8  true
```