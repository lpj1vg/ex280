# Configurar IdP HTPasswd

```bash
#----------------------------------------------------
# 1. Gestión del Archivo HTPasswd
#----------------------------------------------------

# Elimina al usuario 'analyst' del archivo htpasswd especificado.
htpasswd -D tmp_users analyst

# Itera sobre la lista de nombres de usuario (tester, leader, new_admin, new_developer).
# Para cada nombre, actualiza (si existe) o agrega (si no existe) el usuario
# en el archivo htpasswd con la contraseña 'L@bR3v!ew'.
# La opción -b permite pasar la contraseña directamente en el comando.
for NAME in tester leader new_admin new_developer ; \
    do \
    htpasswd -b tmp_users ${NAME} 'L@bR3v!ew' ; \
    done

# Muestra el contenido del archivo htpasswd.
cat tmp_users

#---------------------------------------------------------------------
# 2. Configuración del Proveedor de Identidades HTPasswd en OpenShift
#---------------------------------------------------------------------

# Inicia sesión en el clúster de OpenShift.
# -u especifica el nombre de usuario.
# -p especifica la contraseña.
# Se incluye la URL del servidor API del clúster.
oc login -u admin -p redhatocp https://api.ocp4.example.com:6443

# Crea un secreto genérico llamado 'auth-review'.
# --from-file htpasswd=<ruta_al_archivo> crea una clave 'htpasswd' en el secreto
# con el contenido del archivo especificado.
# -n openshift-config especifica el namespace donde se creará el secreto.
oc create secret generic auth-review \
    --from-file htpasswd=/home/student/DO280/labs/auth-review/tmp_users \
    -n openshift-config

# Obtiene el recurso 'oauth' a nivel de clúster.
# -o yaml especifica que la salida sea en formato YAML.
# > redirecciona la salida al archivo especificado.
oc get oauth cluster -o yaml > oauth.yaml

spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers  # Nombre del recurso secreto que contiene el archivo htpasswd con los usuarios y contraseñas.
    mappingMethod: claim
    name: htpasswd        # Nombre identificador del proveedor de identidad.
    type: HTPasswd

# Aplicamos los cambios:
oc replace -f oauth.yaml

# Reemplaza el recurso existente con la configuración del archivo proporcionado.
# (Se asume que oauth.yaml ha sido modificado para incluir el IdP HTPasswd)
oc replace -f oauth.yaml

# Ejecuta 'oc get pods -n openshift-authentication' repetidamente (por defecto cada 2 segundos).
# Útil para monitorizar cambios en el estado de los pods en tiempo real.
# Presionar Ctrl+C para salir.
watch oc get pods -n openshift-authentication

#---------------------------------------------------------------------
# 3. Asignación de Roles y Verificación de Privilegios
#---------------------------------------------------------------------

# Agrega el ClusterRole 'cluster-admin' al usuario 'new_admin'.
# Esto otorga al usuario permisos completos sobre todo el clúster.
# La advertencia "User not found" puede aparecer si el usuario aún no ha iniciado sesión
# por primera vez, pero el rol se asigna correctamente para cuando inicie sesión.
oc adm policy add-cluster-role-to-user cluster-admin new_admin

#---------------------------------------------------------------------
# 4. Gestión de la Creación de Proyectos
#---------------------------------------------------------------------

# Inicia sesión en el clúster con el usuario 'new_admin' (si no se ha hecho ya).
oc login -u new_admin -p 'L@bR3v!ew'

# Deshabilitar la capacidad de creación de proyectos
# para usuarios que no son administradores de clúster en OpenShift
# Elimina el ClusterRole 'self-provisioner' del grupo 'system:authenticated:oauth'.
# Esto restringe la capacidad de los usuarios autenticados de crear proyectos por sí mismos.
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

#---------------------------------------------------------------------
# 5. Creación de Grupos y Asignación de Permisos Específicos
#---------------------------------------------------------------------

# Crea un nuevo grupo de usuarios llamado 'managers'.
oc adm groups new managers

# Agrega el usuario 'leader' al grupo 'managers'.
oc adm groups add-users managers leader

# Otorga al grupo 'managers' la capacidad de crear proyectos (el rol 'self-provisioner').
oc adm policy add-cluster-role-to-group self-provisioner managers

# Inicia sesión como el usuario 'leader'.
oc login -u leader -p 'L@bR3v!ew'

# Agrega el usuario 'new_developer' al grupo 'developers'.
oc adm groups add-users developers new_developer

# Otorga al grupo 'developers' el rol 'edit' dentro del proyecto 'auth-review'.
# El rol 'edit' permite modificar la mayoría de los recursos dentro de un proyecto.
# Es importante estar en el proyecto 'auth-review' o especificarlo con '-n auth-review'.
oc policy add-role-to-group edit developers -n auth-review

# Crea un nuevo grupo de usuarios llamado 'qa'.
oc adm groups new qa

# Agrega el usuario 'tester' al grupo 'qa'.
oc adm groups add-users qa tester

# Otorga al grupo 'qa' el rol 'view' dentro del proyecto 'auth-review'.
# El rol 'view' permite ver la mayoría de los recursos dentro de un proyecto, pero no modificarlos.
# Es importante estar en el proyecto 'auth-review' o especificarlo con '-n auth-review'.
oc policy add-role-to-group view qa -n auth-review

# Eliminar el usuario kubeadmin (sólo tras verificar acceso con con IdP)
oc delete secret kubeadmin -n kube-system
```
