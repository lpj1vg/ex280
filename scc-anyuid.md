# Cambiar la SCC de un Pod en OpenShift

Este documento describe cómo asociar una Service Account (SA) personalizada a un pod en OpenShift para utilizar una Security Context Constraint (SCC) específica, como `anyuid`.

---

## Pasos

### 1. Identificar la SCC necesaria

> _Requiere privilegios de **cluster-admin**_

```sh
oc get pod <nombre_pod> -o yaml | oc adm policy scc-subject-review -f -
```

### 2. Crear una Service Account personalizada

> _Usuario regular_

```sh
oc create sa <nombre_sa>
```

### 3. Asignar la SCC a la nueva Service Account

> _Requiere privilegios de **cluster-admin**_

```sh
oc adm policy add-scc-to-user nombre-scc -z nombre_sa
```

### 4. Modificar el recurso para usar la nueva Service Account

> _Usuario regular_

- **Si es un Deployment:**

  ```sh
  oc set serviceaccount deployment <nombre_deployment> <nombre_sa>
  ```

- **Si es un DeploymentConfig:**

  ```sh
  oc set serviceaccount dc <nombre_dc> <nombre_sa>
  ```

- **Si es un Pod directo:**

  ```sh
  oc set serviceaccount pod <nombre_pod> <nombre_sa>
  ```

---

## Notas

- Solo los usuarios con privilegios de `cluster-admin` pueden asignar SCCs.
- Utilizar una SA personalizada mejora la seguridad y el control sobre los permisos del pod.
