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