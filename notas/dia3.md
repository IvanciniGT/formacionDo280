# Qué definimos en un archivo de manifiesto de Kubernetes para un despliegue?

PREVIO!
- Namespace
- ServiceAccount (o varios)
- Resource Quota
- LimitRange

Wordpress + MariaDB + Redis

- Deployment Wordpress (configuraciones de la app, usuarios/contraseñas, límites, ...)
- Statefulset MariaDB
- Statefulset Redis
- Service Wordpress
- Service MariaDB
- Service Redis
- Router/Ingress Wordpress
- PVC MariaDB
- PVC Redis
- PVC Wordpress
- Secretos y configmaps (credenciales, configuraciones, ...)
- NetworkPolicy para Redis y MariaDB (si queremos restringir el acceso a estas bases de datos solo a Wordpress)
- PodDisruptionBudget para Wordpress (para asegurar que siempre haya al menos una réplica disponible durante actualizaciones o mantenimiento)
- HorizontalPodAutoscaler para Wordpress (para escalar automáticamente según la carga)
- ServiceMonitor (si queremos monitorear el rendimiento de Wordpress , MariaDB y Redis con Prometheus)
- Backups para Wordpress, MariaDB y Redis (usando CronJobs o herramientas específicas de backup como Velero o Stash)
- ...

> Qué tipo de cosas estamos metiendo en el fichero? A grandes rasgos.. si las juntamos en categorías!

- Mariadb versión 10.5 con 8Gbs de Ram y 4 CPUs, con almacenamiento de 100GB    <- INFRAESTRUCTURA
- Redis versión 6.2 con 4Gbs de Ram y 2 CPUs, con almacenamiento de 50GB        <- INFRAESTRUCTURA
- Apache versión 2.4 con PHP 8.0, con 2Gbs de Ram y 1 CPU                       <- INFRAESTRUCTURA 
  + Wordpress versión 5.8.1                                                     <- APLICACIÓN
    + quiero X plugins, quiero X tema, quiero X usuario                         <- CONFIGURACIONES (Aplicación)
- Balanceador de carga para Wordpress                                           <- INFRAESTRUCTURA
- Política de escalado para el wordpress                                        <- REGLAS DE OPERACIÓN
- Politica de mínimos de réplicas durante actualizaciones                       <- REGLAS DE OPERACIÓN
- Backups diarios para Wordpress, MariaDB y Redis                               <- REGLAS DE OPERACIÓN
- ServiceMonitor para monitorear el rendimiento de Wordpress, MariaDB y Redis con Prometheus <- CONFIGURACIONES (Monitorización)

Estamos metiendo un huevo de cosas ahí dentro... de naturalezas muy diferentes.
Eso tiene que ir con mucho control.

---

# Qué metemos dentro de la definición de una plantilla de POD?

- Service account
- Afinidades y anti-afininidades (Que son pistas para el scheduler para decidir dónde colocar los pods)
  - nodeName
  - nodeSelector
  - affinity
    - nodeAffinity
    - podAffinity
    - podAntiAffinity
- Los init-contenedores que lleva dentro
- Los contenedores que lleva dentro
  - Información de la imagen: registry/repo:tag, política de pull, ...                          INFRAESTRUCTURA
  - Variables de entorno                                                                        CONFIGURACIONES (Aplicación)       
  - Comando del contenedor (con el que arranca)
  - Recursos que necesita el contenedor (requests y limits de CPU y memoria)
  - Los puntos de montaje de los volúmenes que van a ser usados por cada contenedor
  - Los puertos de escucha
  - Pruebas para determinar el estado del contenedor.                                           REGLAS DE OPERACIÓN
  - SecurityContext (privilegios, usuario, capacidades, ...)
- Los volúmenes a los que pueden tener acceso esos contenedores

## Pruebas

En docker, existe el concepto de HEALTHCHECK, que es un comando que se puede incluir en la imagen para que se ejecute periódicamente dentro del contenedor para verificar su estado de salud. Si el comando devuelve un código de salida distinto de cero, Docker considera que el contenedor no está saludable... y puede reiniciarlo.

Eso se queda muy corto en un entorno de producción.
En Kubernetes, tenemos tres tipos de pruebas para verificar el estado de los contenedores:
- Startup probe
- Liveness probe
- Readiness probe

Para cada uno de ellos se configuran varias cosas:
- El tipo de prueba:
  - HTTP GET: Kubernetes hace una petición HTTP a una ruta concreta del contenedor y espera un código de respuesta XXX para considerarlo OK.
  - TCP Socket: Kubernetes intenta abrir una conexión TCP a un puerto concreto del contenedor y si lo consigue, considera que el contenedor está OK.
  - Exec: Kubernetes ejecuta un comando dentro del contenedor y si el comando devuelve un código de salida 0, considera que el contenedor está OK.
- Cada cuanto se ejecuta la prueba (periodSeconds)
- Timeout en la resuesta de la prueba (timeoutSeconds)
- Número de intentos fallidos para considerar que el contenedor no está OK (failureThreshold)
- Número de intentos exitosos para considerar que el contenedor está OK (successThreshold)
- Delay antes de empezar a ejecutar la prueba (initialDelaySeconds)

Lo que cambia es el objetivo de cada una de ellas... y por ende el comportamiento que kubernetes tendrá ante cada una de ellas:

### Startup

Es la primera prueba que se ejecuta.
Sirve para determinar que el contenedor ha arrancado correctamente. 
La idea es simple, hay programas que tardan un huevo en arrancar...
Y mientras arrancan no están para dar servicio.
Esperamos a que estén arrancados. 

    Si ya tarda mucho en arrancar (si las starup probes dan NOK) kubernetes REINICIA EL POD.

    Con algunos programas y en determinadas circustancias hay que tener cuidado con esto... mucho cuidado!
    Nextcloud cuando actualizamos de version... hace un huevo de cosas... cambia estructura de BBDD, actualiza los programas, cambia datos a otros formatos... y puede llevarse 20 minutos tranquilamente arrancando y haciendo ese trabajo.
    De normal, arranca en 20s... Con una actualización de por medio... lo que necesite!

    Como tenga puesto un tiempo pequeño para la prueba, corro el riesgo de que Kubernetes cruja el proceso en medio de la faena = RUINA GIGANTE!

### Liveness

Una vez que el contenedor ha arrancado, comenzamos a ejecutar las pruebas de vida.

Conceptualmente indican que el contenedor (el programa principal que corre dentro) esta en un estado SALUDABLE.

Si las pruebas de vida se marcan como fallidas en un momento dado kubernetes REINICIA EL POD. DE FORMA INCANSABLE!

### Readiness

Una vez sabemos que un contenedor está vivo, tenemos que determinar si está listo para recibir tráfico / prestar servicio.

Si las pruebas de readiness se marcan como fallidas en un momento dado, kubernetes LO SACA DE BALANCEO (lo quita del servicio) PERO NO LO REINICIA.

Que un contenedor no esté listo para prestar servicio no implica que esté en un estado NO SALUDABLE.
Puedo tener una BBDD en modo mantenimiento, haciendole un backup... que tardará 30 minutos... y mientras eso ocurre el contenedor no está listo para prestar servicio... pero eso no significa que no esté en un estado saludable ni que haya que reinicialo. ESTA COJONUDO, haciendo cosas que no son prestar servicio... pero importantes.

SOLO LOS PODS cuyos contenedores están en un estado saludable y listo para prestar servicio, son los que se añaden al balanceo de carga (servicio) y reciben tráfico.

DESARROLLO/NEGOCIO suele rellenar estos valores... y está bien que lo hagan... al menos la mayor parte.
Los comandos que deben ejecutarse los conocen ellos... pero con los tiempos, es otro cantar.

Y aquí depende mucho de la forma de usar / implantación de Kubernetes/Openshift en cada empresa.
Si el control y operación de las apps está descentralizado.. es su mierda.. y paso.

En muchas empresas... se hace un despliegue... pero luego tengo un dpto central de operaciones cuidándolo... Y ahñi hay que andar con cuidado monitorizando lque los tiempos que se hayan configurado sean adecuados.

## SecurityContext

El SecurityContext nos permite configurar los privilegios de los contenedores que se ejecutan dentro del pod.

Los ServiceAccounts pueden tener asociados ROLES/CLUSTERROLES que nos permiten definir los privilegios que un programa tiene a la hora de comunicarse con el cluster.
Esos service accounts pueden ser usados por clientes, que necesiten conectarse al cluster de kubernetes (hablar con el apiServer). Algunos de esos "clientes" pueden ser programas que se ejecuten dentro de un POD/CONTENEDOR y requieran hablar con el cluster para hacer cosas.

Los ServiceAccounts también pueden tener asociados perfiles de seguridad (SecurityContextConstraints en Openshift) que nos permiten definir los privilegios que un programa qu esté corriendo en un POD/CONTENEDOR dentro del cluster tiene a la hora de pedir cosas al sistema operativo del nodo donde se ejecuta.

En KUBERNETES NORMAL, puedo hacer cosas como:

```yaml
 #... dentro de un pod
  containers:
   - name: mi-contenedor
     image: mi-imagen:tag
     securityContext:
       privileged: true
       runAsUser: 0          # Ejecutar como root
  volumes:
   - name: mi-volumen
     hostPath:
        path: /var/run/docker.sock  # Docker in docker (que un contenedor pueda abrir contenedores dentro del nodo)
```

Eso OPENSHIFT POR DEFECTO NO LO PERMITE. NI CERCA !
De hecho una cosa muy común es que cojamos una imagen de DOCKER HUB, que funciona perfecta en kubernetes y que al llevarla a openshift, no funcione porque está usando usuario root.

Que podemos meter en el securityContext de un contenedor?
- Privilegios (privileged: true/false)
- Usuario con el que se ejecuta el proceso dentro del contenedor: root
- Usar una hostNetwork (que el contenedor tenga acceso a la red del nodo donde se ejecuta)
- Montar volumenes del nodo (hostPath) 
- Use contextos de SELinux (seLinuxOptions)
- Capabilities (añadir o quitar capacidades al contenedor, como NET_ADMIN para gestionar la red, SYS_ADMIN para gestionar el sistema, ...)

Y esto va en 2 partes:
- Dentro del contenedor, en el bloque securityContext de cada contenedor, configuro las necesidades de ese contenedor en concreto.
  -> NEGOCIO/DESARROLLO 
- Ese contenedor tendrá un serviceAccount asociado, y ese service account tendrá un perfil de seguridad asociado (SecurityContextConstraints en Openshift) que tendrá reglas que permitán o no ejecutar ese contenedor con esas necesidades.
  -> ADMINISTRACIÓN/OPERACIONES del cluster

La idea es que no porque un usuario ponga:
```yaml
    # dentro del pod
    serviceAccountName: mi-service-account
    # dentro de un contenedor
     securityContext:
       privileged: true
       runAsUser: 0          # Ejecutar como root
```
pueda hacerlo... sino que para poder hacerlo, el service account que tenga asociado ese contenedor tenga un perfil de seguridad asociado que permita ejecutar contenedores con esas necesidades.

$ oc adm policy add-scc-to-user \
                privileged \                        # Que pueda ejecutar contenedores privilegiados
                -z mi-service-account \             # El service account al que le quiero asociar el perfil de seguridad
                -n mi-namespace                     # Namespace donde se encuentra el service account

$ oc adm policy add-scc-to-user \
                anyuid \                             # Que pueda ejecutar contenedores con cualquier usuario (no solo root)                -z mi-service-account \                 # El service account al que le quiero asociar el perfil de seguridad
                -n mi-namespace                      # Namespace donde se encuentra el service account

El concepto es similar a los RBAC, en varias cosas:
- Gestionan privilegios
- Van asociados a service accounts

Pero mientras que los RBAC gestionan permisos a nivel de recursos de Kubernetes,
los SCC gestionan permisos a nivel de contenedores y acceso al sistema operativo de los nodos.

Hay una serie de scc predefinidos en Openshift, como:
- restricted: El más restrictivo, no permite ejecutar contenedores privilegiados ni con usuarios no root. Es el que se asigna por defecto a los service accounts.
- anyuid: Permite ejecutar contenedores con cualquier usuario, incluido root.
- privileged: Permite ejecutar contenedores privilegiados, con cualquier usuario y con acceso a la red y al sistema del nodo.
- hostaccess: Permite ejecutar contenedores con acceso a la red y al sistema del nodo, pero no permite ejecutar contenedores privilegiados ni con usuarios no root.

Lo normal es usar uno de esos:

    $ oc get scc

Con ese comando podemos ver todos los perfiles de seguridad que tenemos en el cluster, y con:

    $ oc describe scc privileged

Dentro de un pod, Openshidt introduce una Anotación que se llama "openshift.io/scc" que nos indica el perfil de seguridad que se ha aplicado a ese pod (en base a su service account).

    $ oc get pod mi-pod -o jsonpath='{.metadata.annotations.openshift\.io/scc}'

Raro que programas de negocio necesiten ejecutar contenedores con privilegios elevados... puede pasar.. pero es raro.

Esto nos pasa mucho en pods de infraestructura, como los de monitorización, logging, backup, ... que necesitan acceso a la red del nodo, o a volúmenes del nodo, o ejecutar comandos dentro del nodo... y para eso necesitan perfiles de seguridad más permisivos.

Y en openshift hay que activar esos perfiles de seguridad a los service accounts que usen esos pods... y eso es algo que suele hacer el equipo de operaciones del cluster, no el equipo de desarrollo/negocio.

## JSONPATH

Es un estandar de facto... no es realmente un estandar formal, que permite extraer información de un JSON.
Esto se usa mucho... el probleamilla es que como no es un estandar formal, cada herramienta lo implementa a su manera... y las sintaxis a veces pueden variar un poco entre herramientas.... los conceptos generales son siempre los mismos.


---

# URL de una imágen de contenedor?

registry/repo:tag

Si no pongo registry, que es opcional, se ataca al/a los que tenga configurado por defecto mi gestor de contenedores (crio, containerd, docker, ...) por defecto.

Docker tiene configurado por defecto el registry de docker hub, así que si no pongo registry, se ataca a docker hub.
Podman tiene configurado por defecto el registry de docker hub, pero también tiene configurado por defecto el registry de quay.io, así que si no pongo registry, se ataca a docker hub y a quay.io.

Yo puedo cambiar esas configuraciones, para que busque en un registry provado que yo tenga.

El tag también es opcional. Si no lo pongo se usa como valor por defecto el tag "latest"... que puede existir o no.
No es una palabra mágica que signifique "La última versión de la imagen", sino que es un tag más, que quien gestione el repo puede haber creado O NO.

De hecho muchos fabricantes no usan tag LATEST... por estar considerado una mala práctica su uso.

Comentario en este sentido.

No solo el tag latest es una mala práctica...

    latest          NO SE USA EN PRODUCCION 
                        En un momento dado puede apuntar a la version 1.1.1, pero mañana puede apuntar a la 2.0.0
                        Puede haber Breacking Changes, y no quiero que mi aplicación deje de funcionar por eso.
    1               NO SE USA EN PRODUCCION
                        En un momento dado puede apuntar a la version 1.1.1, pero mañana puede apuntar a la 1.2.0
                        Trae nueva funcionalidad que no uso.. porque no la estaba usando... pero que puede venir con nuevos bugs
    1.1             ES BUENA OPCION PARA PRODUCCIÓN
                        En un momento dado puede apuntar a la version 1.1.1, pero mañana puede apuntar a la 1.1.2
                        Trae correcciones de bugs, pero no nueva funcionalidad, y eso es deseable!
    1.1.1           ES LA MAS CONSERVADORA

---

Servicios:
- ClusterIP
- NodePort
- LoadBalancer


- ExternalName
    Entrada en DNS que apunta a un servicio fuera del cluster. Quizás ese servicio en el futuro sea parte del cluster... o no.
    Si lo es quiero una transición sencilla.

    El objetivo es que aunque a día de hoy esté fuera del cluster, quiero tenerlo desde ya ese servicio registrado en el DNS del cluster, para que cuando en el futuro lo meta dentro del cluster, no tenga que cambiar nada en la configuración de mi aplicación.

```yaml
kind: Service
apiVersion: v1
metadata:
    name: mi-servicio-clusterip
spec:
    type: ClusterIP
    ports:
    - port: 80
      targetPort: 8080
    selector:
    app: mi-aplicacion
---
kind: Service
apiVersion: v1
metadata:
  name: mi-servicio-externo
spec:
  type: ExternalName
  externalName: servicio-externo.ejemplo.com
```

Que fqdn puedo usar para acceder a un servicio de tipo ClusterIP o a un servicio de tipo ExternalName?

    mi-servicio-clusterip                   Ese nombre (el name, del metadata) del servicio puedo usarlo si estoy en el mismo namespace.
    mi-servicio-clusterip.mi-namespace      Si estoy en otro namespace, tengo que usar el nombre del servicio + el namespace.

        Puedo estar en el namespace "desa", y atacar a "bbdd.prod" sin problemas, usando "bbdd.prod" como fqdn, porque el cluster de kubernetes resuelve ese nombre a la ip del servicio.
        No querré hacerlo... pero ahí entran los NetworkPolicy.
    
    mi-servicio-clusterip.svc.cluster.local   Es el fqdn completo, que puedo usar desde cualquier parte del cluster, 
                                              y que siempre resolverá a la ip del servicio. Lo usamos menos... es escribir mucho sin sentido


- ExternalIP        NO ES UN TIPO DE SERVICIO.
                    Es una propiedad que puedo declarar en servicios de tipo ClusterIP (incluyendo NodePort y LoadBalancer)

                    Si hago esto, es decir, si a un servicio le pongo un ExternalIP, cualquier petición que llegue a cualquiera de los nodos, mediante esa IP, será redirigida al servicio, y por tanto a los pods que hay detrás de ese servicio... en el puerto del servicio.


```yaml
kind:               Service
apiVersion:         v1
metadata:
  name:             mi-servicio

spec:
    type:             NodePort
    ports:
    - port:           80                # Este es el puerto del servicio (en el que contesta la VIPA del servicio)
      targetPort:     8080              # Este es el puerto de los contenedores al que se redirige el tráfico (balanceo)
      nodePort:       30080             # Este es el puerto del nodo por el que se puede acceder al servicio desde fuera del cluster 
    selector:
        app:          mi-aplicacion # Etiquetas que tienen los pods a los que se redirige el tráfico
    externalIPs:      
                    - 192.168.1.101
```

Si el servicio pilla la IP: 10.0.2.127, en esa IP ataco al puerto? 80
Si uno de los nodos tiene la IP: 192.168.1.87, en esa IP ataco al puerto? 30080
Si he definido un externalIP: 192.168.1.101, en esa IP ataco al puerto? 80          AL DEL SERVICIO

Definir una externalIP NO SIGNIFICA QUE KUBERNETES vaya a crear esa IP, asignar esa IP a alguno de los nodos.... NADA DE ESO.
Eso es u ntrabajo que hay que gestionar de forma externa.
Por ejemplo eso es lo que hacen los servicios de tipo LoadBalancer.
- En un cloud, se crea una VIPA (LoadBalancer) y esa VIPA se enruta en la red del cloud a los nodos del cluster
- On prem, cuando usamos MetalLB, MetalLB publica en la red externa (ARP) la IP externa... asociada a uno de los nodos del cluster... y cuando un programa externo quiere ir a la IP externa, como ya sido anunciada en la red, el tráfico llega a uno de los nodos del cluster... y ese nodo se encarga de redirigirlo al servicio... y por tanto a los pods que hay detrás del servicio.

Pero ese trabajo hay que hacerlo de forma externa a kubernetes.

Lo podría hacer a nivel de una máquina external:

    +-------------------------------------------+-- red de mi empresa
    |                                           |
    192.168.1.87                            192.168.1.99
    |                                           |              
    NodoA                                   Máquina externa        
     ^
     Kubernetes
     Y tengo un servicio llamado: mi-servicio, con una externalIP: 192.168.1.101

Si en la máquina externa hago http://192.168.1.101... No contesta. Esa dirección en red no existe... no es una dirección que esté asignada a ningún dispositivo de red... ni a ningún nodo del cluster... ni a ningún servicio de kubernetes... ni a nada... esa IP no existe en la red.

Podría coger esa máquina, si fuera por ejemplo una máquina linux y hacer:
    $ route add -host 192.168.1.101 via 192.168.1.87


Habitualmente cuando trabajamos con un LoadBalancer externo, definimos un pool de IPs.
Los servicios de tipo LoadBalancer toman una IP de ese pool.. la que sea.
A veces no me interesa que la IP sea dinámica... y la puedo forzar.. también mediante el campo externalIP del servicio.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: mi-servicio-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mi-aplicacion
  externalIPs:
  - 192.168.1.101
```

# Exposición de apps que trabajan por HTTP

Si trabajan por http, lo que debo hacer es usar un ingress / ingressController
En el caso de Openshift, metes una ruta (route)

# Exposición de apps que no trabajan por HTTP (solo tcp-bbdd- o ni siquiera tcp, por ejemplo alguien que use UDP)

Lo primero que debería mirar es si en ingressController que tengo (su proxy reverso interno) soporta trafico TCP/UDP... porque algunos ingressControllers lo soportan, y otros no.

- Kubernetes/Openshift y montar nginxIngressController: SI ADMITE protocolo UDP, TCP y HTTP
  - Normalmente esto requiere configuraciones adicionales (en el ingressController) para indicarle que puertos TCP/UDP quiero que gestione
- Openshift por defecto trae un HA/Proxy... puedo montar otro adicional si quiero.
  El HAProxy, no me admite UDP, pero sí me admite TCP... 
  Si uso una BBDD que trabaja por TCP, puedo usar una Route de tipo passthrough, y el HAProxy se limita a pasar el tráfico sin meter mano en él.

- Si quiero UDP, o si quiero saltarme el HAProxy (cosa que NO DEBERIA HACER A LA LIGERA):
  - Servicio de tipo LoadBalancer (si tengo un cloud que me lo permita)
  - Servicio de tipo NodePort, con la castaña de estar usando un puerto muy alto (30000-32767) y tener que abrir ese puerto en el firewall de mi empresa, y en la red del cloud, y en la red de mis clientes... un horror.
  - Servicio de tipo ClusterIP, con una externalIP, y gestionando yo el en rutamiento de esa IP a los nodos del cluster... que puede ser un horror o no.. depende de la infraestructura de red que tenga mi empresa.

A tener en cuenta:
- Cuando usamos UDP, en general buscamos una latencia muy baja:
  - Juegos online
  - Videollamadas
  - Voz IP
  - ...

En esos casos, aunque el ingressController que tenga mi cluster soporte UDP, es posible que no sea la mejor opción, estoy metiendo un componente más en la comunicación.. y eso si o si va a aumentar la latencia... y en algunos casos, esa latencia adicional puede ser un gran problema.

Aun teniendo soporte udp en mi ingressController, posiblemente prefiera: 
- LoadBalancer (si tengo un cloud que me lo permita)
- Servicio de tipo NodePort, con la castaña de estar usando un puerto muy alto
- Servicio de tipo ClusterIP, con una externalIP, y gestionando yo el en rutamiento de esa IP a los nodos del cluster... que puede ser un horror o no.. depende de la infraestructura de red que tenga mi empresa.

Esto me puede pasar también con las BBDD... aunque vayan por tcp puedo quererlo. No siempre... y aqui decido:
- IngressController (proxy reverso) me estandariza el circuito... la comunicación... sigo teniendo todo montado igual! Y eso es conveniente.
- Si me preocupa mucho la latencia, igual que con el UDP... posiblemente prefiera:
  - LoadBalancer (si tengo un cloud que me lo permita)
  - Servicio de tipo NodePort, con la castaña de estar usando un puerto muy alto
  - Servicio de tipo ClusterIP, con una externalIP, y gestionando yo el en rutamiento de esa IP a los nodos del cluster... que puede ser un horror o no.. depende de la infraestructura de red que tenga mi empresa.

El node port es un poco cabrón, necesito tener un balanceador externo, para tener HA.
    Hay aplicaciones que me permiten meter un pool de ips a las que atacar.. y si no llega a una, pues va a la otra
    Hay apps que no, que solo permiten una única ip... y ahí tengo que meter un balanceador externo.
        Aquí volvemos a lo mismo.. quizás mi infra de red sea cojonuda y tenga un F5 (me interesa algo más FISICO que software tipo nginx o haproxy)

Reglas mágicas no hay. Hay opciones.. y en cada caso, y dependiendo de mi infra me interesará más una u otra.

> OJO! dependiendo de como tenga configurado esto...

Puede ser que tenga escalado de mi app (5 pods), pero que todo el tráfico esté llegando por un único nodo - MetalLB trabaja así... metallb no hace "BALANCEO" aunque lo ponga en su nombre... hace FAILOVER... asigna la IP Externa (la anuncia via ARP) a un nodo concreto... y todo el tráfico llega a ese nodo... y ese nodo se encarga de redirigirlo al servicio... y por tanto a los pods que hay detrás del servicio... pero todo el tráfico llega a un único nodo.... Si el nodo se cae, MetalLB hace failover y anuncia la IP externa en otro nodo... y el tráfico llega a ese nodo.

HA tengo... pero todo el tráfico llega por un nodo... y puedo saturarle la red a la mínima... y por más réplicas que tenga... el tubo es el tubo. Y si no tengo más ancho de banda... voy jodido.


Nodos Maestros
Nodos de infra <- IngressControllers y a los que asocio la IP Externa de balanceo de los ingressControllers
Nodos de trabajo <- Donde corren mis aplicaciones
Y en 3/4 de ellos tengo un servicio udp...
    Y monto nodeport
    Pero el balanceador externo lo mando solo a estos 3/4 nodos... no a los de infra, ni a los otros workers.

            BBDD (RAM/Disco)
            Videollamadas (red)
            IA (CPU/GPU)

Hay una anotacio que tiene kubernetes que puede aplicarse a los nodos, para precisamente decidir si ese nodo deberia ser candidato a balanceo para los servicios de tipo Load Balancer o no:

    node.kubernetes.io/exclude-from-external-load-balancers=true

---

# Afinidades y anti-afininidades (Que son pistas para el scheduler para decidir dónde colocar los pods)
  - nodeName
  - nodeSelector
  - affinity
    - nodeAffinity
    - podAffinity
    - podAntiAffinity

- NodeName   -> Asigno el pod a 1 nodo.... RUINA!
- NodeSelector -> Asigno el pod a un grupo de nodos que tengan una etiqueta concreta... Suele ser suficiente para muchos casos y es muy fácil de configurar.
- Afinidades... Lo más potente, también lo más complejo de configurar.
  - Afinidad a nivel de nodo
    Mucho más potente que nodeName y nodeSelector...
    Me permite:
    - Usar no solo tiene o no una etiqueta con un valor exacto el nodo
      Sino operadores más complejos: que tenga una etiqueta con un valor dentro de un rango, o que no tenga una etiqueta concreta, o que tenga una etiqueta con un valor distinto a X, ...
    - Me permite dar reglas que no sea de estricto cumplimiento, sino que sean preferencias.
      > Quiero que INTENTES poner el pod en un nodo que tenga esta etiqueta... pero si no puedes, pues ponlo donde puedas... y no pasa nada.
  - Afinidad a nivel de pod
  - Anti-afinidad a nivel de pod
    Esta la usamos SIEMPRE. DE SERIE.
    Cuando monto un cluster de pods... ofreciendo un determinado servicio y no quiero que todo vayan al mismo host...
    Antiafinidad preferida entre ellos.

---

# Queremos desplegar un ElasticSearch + Kibana en un cluster de Openshift.

En el cluster de Elastic queremos tener: 3 maestros + 2 data + 2 coordinadores.

Vamos a montar un operador que nos ayudará con eso: Elastic Cloud on Kubernetes (ECK).
El operador le montamos también via un chart de helm.

El operador nos ofrecerá unos CRDs (Custom Resource Definition) que nos permitirán crear recursos personalizados de tipo ElasticSearch y Kibana, que el operador se encargará de gestionar.

Una vez montado el operador, para desplegar un ElasticSearch + Kibana, tenemos que crear un yaml usando esos CRDs... y eso lo haremos mediante un CHART de helm: ElasticStack. 


---
Yo, un operador, lo instalo solo 1 vez en el cluster.

Una vez que tengo el operador puedo hacer N instalaciones de lo que sea que gestiona el operador... en nuestro caso clusters de ES.

---

# Instalación del operador:

    $ helm repo add repositorio-charts-de-elastic https://helm.elastic.co
    $ helm repo update
    $ helm install operador-elastic repositorio-charts-de-elastic/eck-operator -n elastic-system --create-namespace
                    ^^^^^^^^          ^^^^^                     ^^^^^^^^^^^^
            Nombre de despliegue      Repo                    Chart que usamos (plantilla)

Esto dará lugar a un despliegue llamado operador-elastic, que iremos controlando en el futuro:
- Subiendo versiones
- Dar marcha atrás en una versión
- Desinstalar el despliegue

## Cosas especiales del Openshift:

- [Optional] If the Software Defined Network is configured with the ovs-multitenant plug-in, you must allow the elastic-system namespace to access other Pods and Services in the cluster:

    $ oc adm pod-network make-projects-global elastic-system

- Crear un namespace para los recursos de Elastic:

    $oc new-project elastic

- Añadir usuarios al namespace elastic, para que puedan gestionar los recursos de Elastic:

    $ oc adm policy add-role-to-user elastic-operator developer1 -n elastic
    $ oc adm policy add-role-to-group elastic-operator developers -n elastic

---
Una vez instalado el operador, que lo instalaré YO! (SE INSTALA SOLO UNA VEZ)

Cada uno iremos creando nuestro cluster en nuestro namespace / proyecto

3 nodos mastros de datos + 2 nodos de datos + 2 nodos coordinadores + kibana

Eso lo haremos con otro chart de helm, que se llama ECKStack, y que se basa en los CRDs que nos ha creado el operador.

    Lo primero será descargar el values.yaml del repo de eck-stack, para ver qué cosasme interesan configurar... lo copio, le cambio el nombre : mis-valores-de-ivan.yaml
    Y a tocar lo que necesite... que será mucho!

    $ helm install cluster-de-elastic-de-ivan repositorio-charts-de-elastic/eck-stack -n elastic -v mis-valores-de-ivan.yaml