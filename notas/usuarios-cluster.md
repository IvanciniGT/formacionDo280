# Usuarios del clúster ROSA `prueba`

**URL de la consola web**: https://console-openshift-console.apps.rosa.prueba.osqa.p3.openshiftapps.com
**API server**: https://api.prueba.osqa.p3.openshiftapps.com:443
**Identity Provider a elegir en el login**: `htpasswd`

## Alumnos

| Alumno                          | Usuario          | Proyecto                  |
|---------------------------------|------------------|---------------------------|
| Enrique de la Ossa Melero       | `enrique`        | `proyecto-enrique`        |
| Manuel Nazario Alba García      | `manuel`         | `proyecto-manuel`         |
| Jorge Luis Montalvo Falcón      | `jorge`          | `proyecto-jorge`          |
| Javier Tercero Jiménez          | `javier`         | `proyecto-javier`         |
| Juan Manuel Álvarez Conde       | `juan-manuel`    | `proyecto-juan-manuel`    |
| Ana García Almaraz              | `ana`            | `proyecto-ana`            |
| Ángel Domínguez Sevilla         | `angel`          | `proyecto-angel`          |
| Víctor Manuel Zamora            | `victor-manuel`  | `proyecto-victor-manuel`  |
| José Ángel Olivares Fernández   | `jose-angel`     | `proyecto-jose-angel`     |
| Pedro Ces Losada                | `pedro`          | `proyecto-pedro`          |

## Administrador

| Usuario | Grupos                            |
|---------|-----------------------------------|
| `admin` | `cluster-admins`, `dedicated-admins` |

## Permisos de los alumnos

- Rol `admin` sobre su propio `proyecto-<usuario>` (control total dentro del namespace).
- `self-provisioner` mantenido → pueden crear sus propios projects adicionales.
- **No** son cluster-admin.

## Login desde CLI

```bash
oc login -u <usuario> -p 'PASSWORD' https://api.prueba.osqa.p3.openshiftapps.com:443
oc project proyecto-<usuario>
```

## Login desde la consola web

1. Abrir https://console-openshift-console.apps.rosa.prueba.osqa.p3.openshiftapps.com
2. Elegir el IdP **`htpasswd`**.
