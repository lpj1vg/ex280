# Enable Developer Self-Service

**Ref: Building applications (2.3)**
**Ref: Postinstallation configuration > Modifying the template for new projects (8.2)**

Crear una template de proyecto con cuotas y límites.

# Creamos un NAMESPACE (no un project)

```bash
# Como admin del clúster
oc create namespace template-test
```

## Cuotas en el namespace `template-test`

Administration > CustomResourceDefinitions

- limits.cpu
- limits.memory
- requests.cpu
- requests.memory

## Límites (por proyecto) en el namespace `template-test`

Administration > LimitRanges, and then click Create LimitRange

## Creación plantilla de proyecto

```bash
oc adm create-bootstrap-project-template \
  -o yaml >template.yaml

oc get quota -n template-test \
  -o yaml >>template.yaml

oc get limitrange -n template-test \
  -o yaml >>template.yaml
```

Editamos el fichero `template.yaml`:

1. Mueve las definiciones de limit range y quota inmediatamente después de la definición de role binding en el archivo `template.yaml`.
2. Elimina cualquier contenido sobrante después del bloque de parámetros (`parameters`).
3. Borra las siguientes claves de las definiciones de limit range y quota:
  - `creationTimestamp`
  - `resourceVersion`
  - `uid`
  - `status`
4. Reemplaza el texto `namespace: template-test` por `namespace: ${PROJECT_NAME}`.

Esto asegura que la plantilla esté limpia, reutilizable y lista para ser aplicada a nuevos proyectos.

## Aplicamos la plantilla de proyecto

```bash
oc create -f template.yaml -n openshift-config
oc edit projects.config.openshift.io cluster

spec:
  projectRequestTemplate:
    name: project-request # Nombre de la template por defecto

# Verificamos
oc get pods -n openshift-apiserver
```
