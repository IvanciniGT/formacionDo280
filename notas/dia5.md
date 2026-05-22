Tenemos 1 proyecto por usuario
ResourceQuota: 4 Gi de memoria, 2 CPU
LimitRange:    1 Gi de memoria, 1 CPU

---
create -> Crea recursos... si ya existen PETA!
apply  -> Crea recursos... si ya existen LOS ACTUALIZA (si es que puede, no siempre se puede)
    No toda propiedad de todo recurso se puede modificar en calient, con apply

    Por ejemplo, al desplegar un deployment:
    - La imagen del contenedor se puede modificar con apply
    - El número de réplicas se puede modificar con apply
    - Si quiero modificar los recursos (CPU, memoria) del contenedor, no puedo hacerlo con apply, tengo que eliminar el deployment y volverlo a crear con create (o usar patch)



---


Un statefulset crea sus pods.
Si hay un problema, lo veo a nivel del statefulset.

Los deployments no crean sus pods.
Delegan ese trabajo a un objeto intermedio de kubernetes llamado replicaset.

El deployment le dice al replicaset asociado -> Queremos tener X pods.

Y el replicaset es el que genera los pods.

---

# Comentarios sobre el init container en el proyecto de elasticsearch

Lo que hemos hecho con el init container es un DESASTRE DE PROPORCIONES EPICAS!
NO DE BROMA NADA PARECIDO EN UN ENTORNO DE PRODUCCION!
Los de Elastic me dan eso y se quedan más agusto que la PETRI!!!!
Y funciona (con apañitos...)!

Quién rellena el archivo con el que hemos hecho el despliegue? En el que viene el initcontainer?
El tio que quiera desplegar un cluster.
Y para poder rellenarlo y que funcione necesita taner un ServiceAccount en su PROYECTO con permisos de ANYUID y PRIVILEGED.
Ese lo hemos creado nosotros (SUPERADMINISTRADORES SUPERSUPREMOS!)

Esa persona, lo mismo que ha creado un initContainer con el comando: sysctl vm.max_map_count=262144, también podría haber creado un initContainer con el comando: rm -rf /

Solución?
1. Ni de broma initContainers y serviceAccounts con permisos ANYUID y PRIVILEGED en un entorno de producción.
2. Me creo un namespace para mi JEFESUPREMO--con un sa con permisos ANYUID y PRIVILEGED
3. Ahí despliego un DaemonSet que ejecute el comando sysctl vm.max_map_count=262144 en cada nodo del cluste con mi sa con permisos ANYUID y PRIVILEGED
4. Y esto se lo doy ya hecho.. EL NO PUEDDE NI DEBE PODER TOCARLO, SOLO USARLO