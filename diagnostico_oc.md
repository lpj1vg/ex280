# Collect OpenShift config files for support

Permisos de `cluster-admin`

```bash
# 1. Crear directorio para la salida (opcional, pero ordenado)
mkdir ~/ocp_support_data

# 2. Ejecutar must-gather
oc adm must-gather --dest-dir=~/ocp_support_data
```

Producto Final: un archivo `must-gather.local.<timestamp>.tar.gz` en tu `--dest-dir`.