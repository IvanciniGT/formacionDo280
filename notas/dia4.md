## Cliente de Openshift

oc <verbo> <recurso> <args>

oc get     nodes

verbos:
- get
- describe  NOMBRE_DEL_RECURSO_CONCRETO
- create
- delete    NOMBRE_DEL_RECURSO_CONCRETO

recursos:
- nodes                     node
- namespaces                namespace                   ns
- pods
- services                  service                     svc
- deployments               deployment                  deploy
- statefulsets
- daemonsets
- configmaps
- secrets
- namespaces
- events
- ingress
- routes
- persistentvolumes                                     pv
- persistentvolumeclaims    persistentvolumeclaim       pvc

args:
  --all-namespaces
  -n, --namespace NOMBRE-DEL-ESPACIO-DE-NOMBRES


c:\usuarios\MI-USUARIO\.kube\config
oc get nodes
# HELM COMANDOS

    $ helm install <nombre-del-despliegue> <repo>/<chart> -n <namespace> -v <values.yaml> --create-namespace
                   mi-cluster-elastic      repositorio-charts-de-elastic/elastic-eck -f mis-valores-de-ivan.yaml -n ivan 

    Osea: 
        helm install mi-cluster-elastic repositorio-charts-de-elastic/elastic-eck -f mis-valores-de-ivan.yaml -n ivan --create-namespace

    $ helm upgrade <nombre-del-despliegue> <repo>/<chart> -n <namespace> -v <values.yaml>

    $ helm list -n <namespace>

    $ helm history <nombre-del-despliegue> -n <namespace>

    $ helm rollback <nombre-del-despliegue> <revision> -n <namespace>

    $ helm uninstall <nombre-del-despliegue> -n <namespace>

    $ helm template <repo>/<chart> -v <values.yaml>


---

# Tintes y toleraciones (Taints & Tolerations)

Ayer hablamos de afinidades / antiafinidades.
Ese es un concepto que gestiona NEGOCIO/DESARROLLO.
Cuando hago un despliegue puede ser que tenga preferencias con respecto a dónde quiero que se desplieguen los pods.
La última palabra la tiene el scheduler.

- No quiero que los 5 pods de mi cluster se desplieguen en el mismo nodo / Regla de Anti-Afinidad de pod
- A ser posible, prefiero que la BBDD y el Servidor de Apps estén en el mismo nodo / Regla de Afinidad de pod
- Mi pod necesita una GPU, por lo que solo se puede desplegar en nodos con GPU / Recurso de GPU. Regla de Afinidad de nodo

Yo (NEGOCIO/DESARROLLO) Al desplegar tengo mis preferencias o necesidades.

Esto es lo que cubren las afinidades y antiafinidades.

Los Taints y las toleraciones son la CONTRAPARTE de las afinidades y antiafinidades.
Trabajan en conjunto con ellas: Esto significa que el scheduler tiene en cuenta tanto unas como otras a la hora de decidir dónde desplegar los pods.

Tengo 5 nodos en el cluster. Y 2 de ellos tienen GPU (nodo4, nodo5).
> CASO 1: Un desarrollador/negocio cuando vaya a desplegar una app1 puede decirle a Kubernetes: Necesito que mi app1 vaya a un nodo con GPU.
El scheduler pondrá ese pod en nodo4 o nodo5, pero no en nodo1, nodo2 o nodo3.
Y ESO ES PERFECTO!

> CASO 2: Un desarrollador/negocio cuando vaya a desplegar una app2 (que no necesita GPU) y no pide nada especial a Kubernetes.
El scheduler puede poner ese pod en nodo1, nodo2, nodo3, nodo4 o nodo5.
Y ESO ES UNA RUINA!

El nodo 4 y el nodo 5 perfectamente pueden ser alternativas para desplegar la app2... pero yo (administrador del cluster) quiero reservar esos nodos para las aplicaciones que realmente necesitan GPU. Y no quiero que una app que no necesita gpu se despliegue en esos nodos.

Este es uno de los casos de uso de los Taints y las toleraciones, en concreto las que se definen con efecto NoSchedule o PreferNoSchedule.

Yo administrador, puedo coger el nodo 4 y el nodo 5 y aplicarles un tinte (taint) con efecto NoSchedule.

                                ETIQUETA: VALOR : EFECTO
    $ kubectl  taint node  nodo4   gpu=true:NoSchedule
    $ oc       taint node  nodo4   gpu=true:NoSchedule

Con eso lo que acabo de hacer es decirle al scheduler: "Oye scheduler, el nodo4 tiene un tinte que dice gpu=true:NoSchedule. Eso significa que no pongas ningún pod en ese nodo a menos que EXPLICITAMENTE ese pod tenga tolere ese tinte.

En lugar de NoSchedule también podría haber usado PreferNoSchedule, que es un tinte más suave. Con PreferNoSchedule le estoy diciendo al scheduler: "Oye scheduler, haz todo lo posible por no poner ningún pod en ese nodo a menos que EXPLICITAMENTE ese pod tolere ese tinte. Pero si no puedes evitarlo, entonces pon el pod ahí".

    $ kubectl  taint node  nodo5   gpu=true:PreferNoSchedule
    $ oc       taint node  nodo5   gpu=true:PreferNoSchedule

De esta forma, el nodo 4 lo reservo exclusivamente para las aplicaciones que necesitan GPU, mientras que el nodo 5 lo dejo como una opción secundaria para aplicaciones que no necesitan GPU pero que pueden tolerar compartir nodo con aplicaciones que sí necesitan GPU.

Cuando un desarrollador ahora queiere desplegar la app1, que necesitaba GPU, debe indicar 2 cosas por separado:
1. Que su app1 necesita GPU (afinidad de nodo)
2. Que su app1 tolera el tinte gpu=true:NoSchedule (toleración)
    ```yaml
        #.. plantilla de pod
             tolerations:
                - key: "gpu"
                  operator: "Equal"
                  value: "true"
                  effect: "NoSchedule"
                - key: "gpu"
                  operator: "Equal"
                  value: "true"
                  effect: "PreferNoSchedule"
    ```
4. También, que su app1 tolera el tinte gpu=true:PreferNoSchedule (toleración)

Por contra, el usuario que despliega la app2, que no necesita GPU, no indicará nada.
Y al tener los nodos 4 y 5 con el tinte gpu=true:NoSchedule y gpu=true:PreferNoSchedule, el scheduler no pondrá la app2 en esos nodos, reservándolos para las aplicaciones que realmente necesitan GPU.

    gpu=true       ... en lugar de eso podría haber escrito menchu=federico

Esto ayuda a hacer un uso más adecuado de los recursos del cluster, reservando nodos específicos para aplicaciones que tienen necesidades especiales, y evitando que aplicaciones que no necesitan esos recursos ocupen esos nodos.

El propio kubernetes hace uso interno de tintes y toleraciones.
Por ejemplo, si una máquina no tiene mucha RAM libre, kubernetes la marca con un tinte que dice que no se pueden programar pods en esa máquina a menos que esos pods toleren ese tinte. De esta forma, el scheduler evita programar pods en máquinas que están muy cargadas, a menos que esos pods estén diseñados para tolerar esa situación.
Lo mismo con CPU.
    $ kubectl taint node nodo4 memory-pressure:NoSchedule
    $ kubectl taint node nodo5 cpu-pressure:NoSchedule

este es uno de los usos... Y ojo, pone muy claro: NoSchedule... Eso significa que no tiene efecto retroactivo. 
Es decir, si ya hay pods desplegados en esos nodos, esos pods no se ven afectados por el tinte, ya que ya fueron scheduled,
El tinte solo afecta a los nuevos pods que se intenten desplegar después de aplicar el tinte.

---

El otro uso de los tintes es más interno de kubernetes, y es para marcar nodos que tienen algún tipo de problema, o nodos que queremos actualizar o eliminar del cluster.
En ese caso, el administrador del cluster o kubernetes aplica tintes con otro efecto: NoExecute.

NoExecute no es ya solo No Planifiques pods en un nodo... es más agresivo: No planifiques pods en ese nodo, y si ya hay pods desplegados en ese nodo, expúlsalos de ese nodo.

    $ kubectl taint node nodo4 gpu=true:NoExecute
    $ kubectl taint node nodo5 gpu=true:NoExecute

Kubernetes también aplica tintes con efecto NoExecute de forma automática cuando detecta que un nodo tiene un problema grave, como por ejemplo que el nodo no responde o que tiene un fallo de hardware. En ese caso, kubernetes aplica un tinte con efecto NoExecute para evitar que se programen nuevos pods en ese nodo, y para expulsar los pods que ya están desplegados en ese nodo, permitiendo así que esos pods se reprogramen en otros nodos del cluster.

Aunque esto existe... lo reservamos más para uso interno de kubernetes.
Si yo quiero quitar todos los pods de un nodo, normalmente hago otra cosa:

    $ kubectl drain nodo4
    $ kubectl drain nodo5

    Ahí vamos sin piedad... todo lo que hay en esos nodos se va a ir a otros nodos del cluster, y no se van a programar nuevos pods en esos nodos hasta que los vuelva a habilitar con:

    $ kubectl uncordon nodo4
    $ kubectl uncordon nodo5


---

1. Cargar algo en kubernetes es subirle un archivo de manifiesto YAML.
2. Ese archivo puedo:
   - crearlo yo
        oc create -f mi-manifiesto.yaml  (en este escenario, mi archivo lleva rcursos propios de kubernetes)
   - pedirle a un operador que lo cree
        oc create -f mi-manifiesto.yaml  (en este otro, mi archivo lleva recursos de tipo CRD... de los del operador)
   - pedirle a helm que lo cree.
        helm install mi-despliegue repo/chart -n mi-namespace -f mis-valores.yaml 

Hay una chart (PlANTILLA) que me da la gente de elastic: elastic-eck.
Ese nos genera un archivo de manifiesto yaml con CRDs aptos para el operador.
Para configurar el despliegue usamos también un archivo YAML... pero no un archivo YAML de manifiesto de kubernetes.


HELM define plantillas... que permiten generar archivos de kubernetes. Esas plantillas se llaman CHARTS!
Al gener un archivo de kubernetes con una plantilla, puedo darle parámetros.
Esos parámetros también se los damos en un archivo YAML... pero ese archivo YAML no es un manifiesto de kubernetes, sino un archivo de configuración de HELM.

Cada plantilla de helm define la estructura de su archivo de parametros.
2 plantillas distintas usan archivos de parametros con estructuras distintas.



---
```yaml

cestaComprA:
    patatas: 3
    leche: 6
    huevos: 12

```
---


```yaml

alumnos:
    - nombre: Pedro
      edad: 44
      domicilio:
        calle: Calle Falsa
        numero: 123
    - nombre: Ana
      edad: 22
      domicilio:
            calle: Calle Verdadera
            numero: 321
```

Un archivo de Kubernetes es un archivo YAML con un a estructura muy concreta:

```yaml
apiVersion: <grupo>/<version>
kind: <tipo-de-recurso>

metadata:
  name: <nombre-del-recurso>

spec:
    # Aquí van las especificaciones concretas de cada recurso
```

Hay plantillas de helm CHARTS, que usan SUBCHARTS... y esto se complica.
Cada subchart tiene su propio archivo values.yaml


Kubernetes reemplaza al sysadmin de toda la vida...
Ese ya no tiene trabajo! Esta en la calle!

---
2021-2022 Linkedin -> Despidos masivos en las tecnologicas: 200.000
Meta: 20.000
AWS: 10.000
Microsoft: 4.000

Y este es el tema. AUTOMATIZACIONES

En España vamos 5 años de retraso con el resto del mundo.

