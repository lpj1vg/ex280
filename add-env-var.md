# Variables de Entorno en OpenShift

Este documento explica cómo listar, agregar las variables de entorno en recursos de OpenShift como Deployment y DeploymentConfig.

---

## Mostrar variables de entorno configuradas

Para listar las variables de entorno asociadas a un Deployment:

```sh
oc set env deploy/mi-app --list
```

## Agregar o modificar variables de entorno

Puedes agregar o modificar variables de entorno de la siguiente manera:

```sh
oc set env deploy/mi-app \
  <KEY>=<VALOR> \
  [<KEY2>=<VALOR2> ...]
```

También puedes cargar variables desde un archivo `.env`:

```sh
oc set env deploy/mi-app \
  --from-file=my-vars.env
```

Desde un `ConfigMap` con prefijo (opcional):

```sh
oc set env deploy/mi-app \
  --from=configmap/nombre-de-mi-configmap \
  --prefix=APP_
```

Desde un `Secret` con prefijo (opcional):

```sh
oc set env deploy/mi-app \
  --from=secret/nombre-de-mi-secret \
  --prefix=DB_
```

## Eliminar variables de entorno

Para eliminar una variable de entorno:

```sh
oc set env deploy/mi-app <KEY>-
```
