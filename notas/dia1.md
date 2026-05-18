# Qué es un contenedor?

Es un entorno aislado (dentro de un SO con kernel Linux) donde ejecutar procesos.
Ese entorno aislado, tiene:
- Sus propias variables de ENTORNO, como entorno que es.
- Sus propio sistema de archivos
- Su propia configuración de red
- Puede tener limitación de acceso a los recursos físicos de la máquina.

Los contenedores los creamos desde imágenes de contenedor. Que descargamos de Registro de repositorios de imágenes de contenedor, como:
- Docker hub
- Quay.io
- Microsoft artifact registry
- Oracle container registry
- ...
---
# Qué es kubernetes?

Es una herramienta que nos permite definir mediante un lenguaje declarativo UN ENTORNO DE PRODUCCION, para que sea creado y operado en AUTOMATICO por kubernetes. En ese entorno de producción, los trabajos (los procesos de SO) se ejecutan dentro de contenedores (pero esto es casí anecdótico).

## Lenguaje declarativo

Lo que estamos haciendo con kubernetes es automatizar la creación y operación de un entorno de producción.

### Automatizar?

Crear una máquina (o cambiar el comportamiento de una mediante programas) que haga lo que antes hacia un humano con sus manos.

Puedo automatizar el lavado de la ropa (LAVADORA).. que puedo incluso cambiar su comportamiento con programas (PROGRAMAS DE LAVADO: frio, prendas delicadas. algodon a 90...). Esto implica que el humano desaparezca? NO... quién le da al botón: Quién solicita la ejecución del proceso? El humano

En nuestro caso, la máquina la tenemos: LA COMPUTADORA... y lo que necesitamos es crear un programa o configurarlo... Luego lo lanzo (yo humano.. o no). Y ese programa hace cosas sin nuestra intervención.

En concreto, bajando esto a kubernetes y al despliegue de entornos de producción... llevamos décadas creando scripts (programas) que hacen lo que antes hacía un humano con sus manos... instalar apps, configurarlas...

Hoy en día, en lugar de crear scripts, usamos un programa (son muchos en realidad) que ya existe: KUBERNETES... y le damos configuraciones a kubernetes... y órdenes.

La gracia es que las configuraciones se las damos en lenguaje declarativo.

Cuando creamos un script (.sh, .bat, .ps1) que tipo de lenguaje usamos? imperativo

mkdir ventana  # make directory ventana:        ORDEN DIRECTA AL COMPUTADOR para que cree un directorio llamado ventana
cd ventana     # change directory ventana:      ORDEN DIRECTA AL COMPUTADOR para que cambie el directorio actual a ventana

> Felipe, pon una silla debajo de la ventana       IMPERATIVO

Y el lenguaje imperativo, al que estamos muy acostumbrados, después de décadas de usarlo es una mierdaa!

> Felipe, IF hay algo que no sea una silla debajo de la ventana:
>   QUITALO                                                         Imperativo
> Felipe, IF no hay silla debajo de la ventana:
    > Felipe, if not silla (silla ==False) THEN:
        > GOTO IKEA                                                 Imperativo
        > compra silla                                              Imperativo
    > Felipe, pon una silla debajo de la ventana                        IMPERATIVO

En lugar de esto, podría haber usado un lenguaje declarativo:

> Felipe, debajo de la mesa tiene que haber una silla. Es tu responsabilidad.    DECLARATIVO

No digo lo que tiene que hacer... digo lo que es.

Es interesante que al hacer esto, delego la responsabilidad de que eso se cumpla a otra persona... en este caso será a otro programa: KUBERNETES.

Quien crea un entorno de producción, quien crera un balanceador, una regla de firewall no soy yo.. es kubernetes. yo la defino/declaro.
Si... al final si hay una orden... Ceea eso... Borra eso... pero eso va después... En la definición no hay verbos... no hay ordenes... solo hay declaraciones de lo que es el entorno de producción que quiero gestionar (crear o borrar o actualizar...)

Una cosa del lenguaje declarativo es que es idempotente!

### Idempotente aplicado al mundo este...

Significa que pueda ejecutar un programa todas las veces que quiera sin que eso cambie el resultado.
Que dé igual el estado inicial del sistema, que el resultado final sea el mismo siempre, cuando termine de ejecutarse el programa.

Y el lenguaje declarativo nos regala eso... per se.

### Control de acceso

Cuentas de servicio / Service Accounts
Roles
ClusterRoles
Namespace 

### ~~Controla el ciclo de vida de las apps~~

El ciclo de vida de una app no es solo la operación... es también el desarrollo.
Esto lo hacen otras herramienatas: ArgoCD, Jenkins, Gitlab CI/CD, etc... que se encargan del ciclo de vida de un sistema.

Kubernetes controla el despliegue, la operación y algo de monitoreo... pero no controla el ciclo de vida completo de una app.

### Kubernetes es la herramienta que hoy se usa en los entornos de producción.

El SO de los centros de datos modernos

### Autoescalado (pods... máquinas?... ahi es donde ayudan el openshift, tanzu, karbon...)

Kubernetes nos ayuda a montar más réplicas de los programas .. y a balancear entre ellas... pero! Limitado a los recursos que tengo efectivos en un cluster.
Si necesito más recursos físicos (Memoria, CPU, etc) para montar más réplicas... tengo que añadir más máquinas al cluster... y eso no lo hace kubernetes.
Hay distribuciones de kubernetes que si lo hacen o pueden ayudar:
- Redhat Openshift con su autoescalado de nodos / maquinas (si trabajo en cloud)
- Tanzu con su autoescalado de nodos / maquinas (si trabajo contra un VSphere)
- Karbon con su autoescalado de nodos / maquinas (si trabajo contra un Nutanix)
- EKS con su autoescalado de nodos / maquinas (si trabajo contra AWS)
- AKS con su autoescalado de nodos / maquinas (si trabajo contra Azure)
- GKE con su autoescalado de nodos / maquinas (si trabajo contra Google Cloud)


# Arquitectura de Kubernetes

Trabaja en cluster.

Tendremos distintos tipos de nodos:
- Maestros (Control-plane). Son los nodos que corren los propios programas de kubernetes.
  Kubernetes no es un programa. son muchos programas.
  Algunos de ellos corren como contenedores. Otros corren de forma local, sobre el propio SO.

    Programas que corren como contenedores:
    - etcd: es la base de datos de kubernetes, donde se guarda toda la información del cluster.
    - API server: es el programa que recibe las peticiones de otros programas y las enruta.
    - Controller manager: es el programa que se encarga de controlar el estado del cluster, y de ejecutar las acciones necesarias para mantenerlo en el estado deseado.
    - Scheduler: es el programa que se encarga de asignar los pods a los nodos de trabajo, teniendo en cuenta los recursos disponibles y las necesidades de los pods.
    - CoreDNS: es el programa que se encarga de resolver los nombres de los servicios dentro del cluster.
    - Kube-proxy: es el programa que facilita las comunicaciones...
                  Va dando de alta y modificando las reglas de netfilter en cada nodo del cluster.
    - Driver de red... para montar una red virtual entre los nodos a la que poder pinchar los contenedores/pods.

    Programas que corren de forma local:
    - kubelet: es el programa que se encarga de transladar las ordenes de kubernetes a los gestores de contenedores (docker, containerd, cri-o) para que creen los contenedores y los ejecuten.
    - kubeadm: es el programa que se encarga de inicializar el cluster, y de unir los nodos al cluster... y otras operaciones de mentanimiento / Administración del cluster.
    - kubectl: es un cliente de kubernetes. De los muchos clientes que hay.
      Distros concretas de kubernetes tienen su propio cliente:
      - Openshift: oc
      - Tanzu: tanzu
      Muchos de ellos están a su vez montados sobre kubectl, pero con funcionalidades adicionales. 

- Nodos de trabajo (Worker nodes). Son los nodos donde se ejecutan los procesos de las apps que queremos montar en nuestro entorno de producción.

En openshift se suele hablar también de nodos de infraestructura, que son nodos donde se ejecutan otros programas no finales, pero que me son necesarios para el funcionamiento del cluster (como el router, el registry, monitores, etc...)

---

# Qué es un Operador?

Lo primero.. mirad el nombre OPERADOR.
Lo que hacemos es montar un PERSONITA ahi dentreo del cluster.. que sabe un huevo de un tema concreto... y le damos la responsabilidad de gestionar ese tema concreto dentro del cluster.
- Operador de MariaDB
- Operador de Elasticsearch
- Operador de Postgres
- Operador de Kafka

Son programas que vienen con conocicmiento extendido para administrar/desplegar/configurar/operar ese tipo de aplicaciones dentro del cluster.
Es como si contrato a un experto en MariaDB dentro de mi empresa.

Estos operadores, lo que me permiten es comunicarme con ellos mediante un lenguaje declarativo, pero de más alto nivel que el que ofrece de serie kubernetes.

Si yo tengo en mi empresa, en mi departamento de sistemas a un Sysadmin experto en linux.. y procesos... y algo de infra, le puedo decir:
- Oye, quiero que me reserves un servidor. Con tantos gigas de RAM, tanta CPU.
- Quiero que me pongas dentro este programa MariaDB : (aqui te paso el script de instalación)
- Quiero que me montes en el FileSystem del servidor un almacenamiento de tantos gigas, con este tipo de sistema de archivos, con esta configuración...
- Quiero que me pongas delante una VIPA.. y que tengas una réplica de esa BBDD en espera por si la principal se cae... y que esté smonitorizando la BBDD con este comando, y si ves que deja de responder, que me hagas un failover automático a la réplica... cambiando la VIPA, para que apunte a la réplica... 
- Quiero que ejecutes este script de BBDD que te doy yo, para crear una BBDD dentro de la instancia... y que configure estos usuarios... Pero vamos.. que realmente lo unico que tienes que hacer es esjecutar el script sql que te paso.
- Quiero un. backup diario... de la BBDD, para ello, debes copiar lo que hay en tal directorio todas las noches.. metiendo primero la BBDD en modo mantenimiento... que lo haces con este comando... y luego la sacas del modo mantenimiento... con este otro comando...

Ahora.. si tengo un experto en MariaDB, hablaría con el de otra forma... a más alto nivel.
- Oye, quiero tener un MariaDB, con tantos gigas de RAM, tanta CPU.
  > Ya sabe que eso lo tiene que poner en un servidor.
 ~~- Quiero que me pongas dentro este programa MariaDB : (aqui te paso el script de instalación)~~
~~- Quiero que me montes en el FileSystem del servidor un almacenamiento de tantos gigas, con este tipo de sistema de archivos, con esta configuración...~~ -> Quiero tener 20Gbs dde almacenamiento para el MariaDB.
~~- Quiero que me pongas delante una VIPA.. y que tengas una réplica de esa BBDD en espera por si la principal se cae... y que esté smonitorizando la BBDD con este comando, y si ves que deja de responder, que me hagas un failover automático a la réplica... cambiando la VIPA, para que apunte a la réplica... ~~ -> Si acaso lo único decirle: La quiero en espejo (modo replicación)
~~- Quiero que ejecutes este script de BBDD que te doy yo, para crear una BBDD dentro de la instancia... y que configure estos usuarios... Pero vamos.. que realmente lo unico que tienes que hacer es esjecutar el script sql que te paso.~~ -> Quiero tener tal BBDD con tales usuarios.
- Quiero un backup diario... de la BBDD todas las noches ~~, para ello, debes copiar lo que hay en tal directorio .. metiendo primero la BBDD en modo mantenimiento... que lo haces con este comando... y luego la sacas del modo mantenimiento... con este otro comando...~~
 
Es decir, se queda asi:
- Oye, quiero tener un MariaDB, con tantos gigas de RAM, tanta CPU.
- Quiero tener 20Gbs dde almacenamiento para el MariaDB.
- Si acaso lo único decirle: La quiero en espejo (modo replicación)
- Quiero tener tal BBDD con tales usuarios.
- Quiero un backup diario... de la BBDD todas las noches

Ese lenguaje se consigue extendiendo los conceptos de los que hablo normalmente con kubernetes. 
- El kubernetes es como un sysadmin de linux... Si le explico las cosas despacio, no tiene problemas en ejecutarlas... pero se las tengo que explicar despacio.
- Los operadores son como expertos en un tema concreto... que saben de ese tema concreto ... y por eso, con ellos puedo hablar a más alto nivel.
Ese lenguaje extendido es lo que llamamos los CRD (Custom Resource Definition) de kubernetes.

Kubernetes no sabe lo que es un MariaDB.
Pero un operador si... si es experto en MariaDB... y eso me permite hablarle usando otras palabras (CRDs).
Y una vez que les pido algo, se ponen en ello... De hecho.. lo que hacen es un poco simple...
- Traducen mi lenguaje de alto nivel (CRD) a lenguaje de bajo nivel (kubernetes)
- Es decir, yo ya no hablo con el sysadmin... hablo con el experto... y el experto se encarga de hablar con el sysadmin para que haga lo que yo necesito... pero él ya se pega con el sysadmin.
  Traducido a kubernetes, yo le pido algo al operador... y el operador se encarga de traducir eso a lenguaje kubernetes... y de pedirle a kubernetes que haga lo que necesito.

Hay miles de operadores. No necesito un experto de todo. Si voy a tener 1 MariaDB en la empresa.. me interesa contratar un experto? En general me interesará mucho menos que si tuiviera que montar 40 MariaDBs... en ese caso, el expoerto me va a ayudar a montar esos 40 MariaDBs... y a gestionarlos... y a mantenerlos... y a monitorizarlos... y a hacerles backup... y a hacerles failover... etc...

No significa que sin experto (operador) no pueda hacer mi instalación... pero me toca pegarme más.
Ahora.. contratar un experto tampoco es gratis:
- Necesito hacer proceso de selección           Buscar operador
- Contratarlo                                   Instalarlo en el cluster
- Pagarle el sueldo                             Consumir recursos del cluster para que se ejecute el operador
- Aprender a hablar con el en su lenguaje       Aprender los CRDs del operador, y su funcionamiento... y su comportamiento... y sus limitaciones... y sus bugs... y sus actualizaciones... y sus parches de seguridad...
- etc...

Y muchas veces, no compensa... el sobre esfuerzo. Sobre todo si voy a tener pocas instancias de ese tipo de aplicación.
Seguro que si voy a instalar / operar poco esa historia.. algún script de instalación (CHARTS HELM) más tradicional encuentro por ahí... y me resuelve la paleta de forma más sencilla.
---

# Qué es Openshift?

Es la solución de Kubernetes empaquetada por REDHAT.


---


# Entorno de producción tradicional
        Mi infra.                                           La infra del cliente
    ----------------------------------------------------  ------------------------
    BBDD <- Serv. App 1
            Serv. App 2 <- Balanceador <- Proxy reverso <- Proxy <- Cliente
            Serv. App 3

    Almacenamiento
        Cabina de almacenamiento
        NAS

    Comunicaciones: Reglas de firewall

    DNS
    
    Certificados (comunicaciones seguras) SSL/TLS

Y esto es lo que hoy en día podemos hacer que Kubernetes cree por nosotros.

- Balanceadores                                                     Service
- Configure proxies reversos                                        Ingress/Routas
- Configure reglas de firewall                                      Network policies
- Configure el acceso a volumenes de almacenamiento externos        PVC/PV
- Gestiones de certificados                                         Cert-manager
- Altas de DNS                                                      Route
                                                                    Service
- Escalado                                                          Horizontal Pod Autoscaler

Hay contenedores? Si... el Weblogic.. y la BBDD corren dentro de un contenedor. Kubernetes va mucho más allá de eso!
De hecho, la gestión de los contenedores, no la hace kubernetes... la hace un gestor de contenedores (tipo docker: containerd, cri-o) que es el que se encarga de crear los contenedores, arrancarlos, pararlos, etc. Kubernetes se encarga de orquestar esos contenedores, pero no de gestionarlos.

Kubernetes es un orquestador de contenedores? NO
Si acaso sería un gestor de gestores de contenedores... pero su trabajo va mucho más allá.
Crear y operar en automático un entorno de producción completo, que no es solo la ejecución de unos procesos.

---

# Sobre virtualización....

## Kubernetes gestiona VMs?

NO... ni cerca.
Otra cosa.. es que Redhat tiene un producto llamado OpenShift Virtualization, que es el Redhat Virtualization de toda la vida... que lo ha enchufado dentro del Openshift.. a ver si mete en las empresas.. con ese no triunfón (la gente tiró por VMWare).. pero como con el Openshift lo petó... pues a ver si metiendolo dentro cuela.

## Los contenedores tienen algo que ver con virtualización?

NADA QUE VER!

## Cosas a tener en cuenta:

1. Dentro de un contenedor, lo que ejecutamos son procesos.
   Un proceso puede ser :
   - Una aplicación
   - Un comando
   - Un script
   - ...
2. Ligero... dependerá de lo que haya dentr de la imagen del contenedor que de lugar al contenedor.
   Lo de ligero tiene sentido cuando comparamos con algo.
   Más ligero o menos ligero... En este caso comparamos con las VMs.


---

Los contenedores nos resuelven muchos de los problemas que también nos resolvían las VMs... pero de otra forma muy diferente!
Esto nos hace pensar en ocasiones que son otra forma de virtualización. Y NO ES NI PARECIDO.. aunque en la práctica me resuletan problemas equivalentes (algunos de ellos... no todos).
Lo que pasa es que las cosas que podemos resolver con ambos (contenedores y vms) son mucho mejor resolverlas con contenedores... y por eso se han impuesto. La cosa es que las VMs, aunque resuelven ciertos problemas, plantean otros! Cosa que no ocurre, al menos en la misma media al trabajar con contenedores.

## Principal diferencia entre un contenedor y una VM?

En una VM lo primero que necesito es un SO completo.. con su kernel independiente, que es el que se encarga de ejecutar los procesos que corren dentro del contenedor.
Puedo montar un SO completo (con su kernel) dentro de un contenedor? NO... es imposible. esto no se puede hacer.
    3 Gbs                       a       28 Mbs
    Imagen ISO de ubuntu                Imagen de contenedor de Ubuntu

    LO MISMO NO ES!

    30Mb / 3Gb = 0,01: 1% del tamaño... lo mismo no es.

Dentro de un contenedor ES IMPOSIBLE montar un SO completo con su kernel independiente. No funciona así.. y esto es una duda muy común, debido a esas imágenes llamadas: ubuntu, fedora, debian, alpine... Que nos invitan a pensar que es posible montar esos SO dentro de un contenedor... pero no es así.

### Forma de instalar basada en Máquinas Virtuales

        App1    |     App2 + App3
    -------------------------------------
        SO1     |     SO2     
    -------------------------------------
        VM1     |     VM2     
    -------------------------------------
        Hipervisor
    -------------------------------------
        Sistema Operativo
    -------------------------------------
        Computadora física

### Forma de instalar basada en Contenedores

        App1    |     App2 + App3
    -------------------------------------
        C1      |     C2     
    -------------------------------------
        Gestor de contenedores
        Docker, podman, containerd, cri-o
    -------------------------------------
        Sistema Operativo (Linux)
    -------------------------------------
        Computadora física

---

## Sobre el lenguaje que usamos para hablar con kubernetes...

En la mayor parte de escarios (no en todos) tendemos a mandarle a kubernetes: Archivos de manifiesto.
Es curioso el nombre... MANIFIESTO: ESTO TIENE QUE VER CON EL LENGUAJE DECLARATIVO! 
                        UN PROCEDIMIENTO: ESTO TIENE QUE VER CON EL LENGUAJE IMPERATIVO! Y así suelen ser nuestros scripts de instalación tradicionales.

En un manifiesto yo declaro lo que quiero. Lo que es.
El rey manifiesta que ... El rey no da órdenes.
El rey no dice: tu haz esto, tu haz lo otro... El rey no da órdenes... 
El rey dice: Esto es lo que quiero... hágase mi voluntad.

Sea mediante esos archivos o mediante comandos (que también se permite... aunque en general tendemos a usarlos poco) lo que kubernetes nos ofrece es un vocablo de palabras. Las cosas que el entiende. Los recursos que me puede gestionar.
Luego habrá recursos adicionales (Custom Resource Definition) que me ofrecen los operadores... pero kubernetyes en si mismo ya me ofrece un conjunto inicial de recursos... y un vocabulario de palabras para hablar con él... y ese vocabulario es el que me permite usar el lenguaje declarativo para decirle lo que quiero.

Recursos que podemos gestionar vía kubernetes:
- Node                          Representación de una máquina integrada dentro de nuestro cluster de kubernetes
                                (NOTA: En openshift existe el concepto de Machine, que es una representación de una máquina física o virtual, pero que no es un nodo. No forma parte -al menos aún- del cluster)
- Namespace                     Grupo de recursos que gestiono (en parte) de forma conjunta:
                                - Les asocio administradores
                                - Limito los recursos que pueden consumir
                                De donde viene el nombre? Por qué Namespace?
                                - Dentro de un espacio de nombres, no puede haber 2 recursos del mismo tipo con el mismo nombre.
                                - Es un espacio (grupo de recursos), donde el nombre es UNICO por tipo de objeto.
                                Para qué lo usamos: juntar todo lo relativo a un despliegue concreto, un cliente concreto... un entorno concreto. 
- Pod                           Un conjunto de al menos 1 contenedor, que:
                                  - Se despliegan juntos en el mismo host
                                    - Pueden compartir almacenamiento LOCAL!
                                    - Comparten RED... comparten IP... y entre ellos pueden comunicarse por localhost
                                  - Se escalan juntos.
- Deployment                    Plantilla de pod + número inicial de réplicas.
- Statefulset                   Plantilla de pod + al menos 1 plantilla de peticion de volumen + número inicial de réplicas.
                                Cada instancia / réplica tiene su propio volumen de almacenamiento, independiente de las otras réplicas.
                                    Se usa para BBDD, sistemas de mensajería (Kafka, RabbitMQ, etc...), indexadores (Elasticsearch, Solr, etc...).
                                    En general para herramienats en cluster donde cada réplica guarda sus propios datos, independientes de los datos de las otras réplicas.

                MariaDB-Galera-1        dato1   dato2
                MariaDB-Galera-2        dato1   dato3
                MariaDB-Galera-3        dato2   dato3

            Si el dato lo guardo en 2.. tengo una mejora teórica de dendimiento de un 50%.
            Con una máquina, en 2 unidades de tiempo, puedo guardar 2 datos. Con 3 máquinas, en 2 unidades de tiempo, puedo guardar 3 datos. Es decir, con 3 máquinas, tengo un rendimiento de 3/2 = 1,5 veces el rendimiento de una máquina. Es decir, una mejora del 50% respecto a una máquina.

            Es frustrante.. ya que hago un x3 en la infra y solo consigo un x1,5 en el rendimiento... La HA es cara...
            Y eso es la mejora teórica... en la práctica, la mejora de rendimiento suele ser mucho menor...y se me queda en un 20-30%

            MariaDB funciona así. Kafka funciona así. Elasticsearch funciona así.

        Para que monto una BBDD en cluster activo/activo? Alta disponibilidad? NO SOLO
            Busco escalabilidad... Mejora de rendimiento.
        Si solo busco HA, posiblemente me interese mucho más una estregia de tipo: ACTIVO/PASIVO

        Como cada instancia tiene sus propios datos, eso convierte a cada instancia (réplica) en distinta a las demás.
        Los statefulset también me dan un mecanismo para acceder a cada réplica de forma individual, mediante un nombre único FQDN.


- DaemonSet                     Plantilla de pod de la que Kubernetes monta una réplica en cada nodo del cluster.
- HorizontalPodAutoscaler       Monitoriza si unas métricas de nuestra elección soin superadas o no... y en función de eso, escala (arriba o abajo) el número de réplicas de un deployment o statefulset.
- PodDisruptionBudget           Tiene que ver con la cantidad mínima de réplicas que tienen que quedar en funcionamiento, incluso durante tareas de mantenimiento del cluster, para garantizar la disponibilidad de la aplicación.
  - Si hay que evacuar un nodo por mantenimiento
  - Si hay que actualizar el cluster
  - Si hay que hacer un failover de una máquina

- Jobs                          Es un pod, donde los contenedores acaban... finalizan su trabajo. 
                                Es decir, donde el proceso principal que se está ejecutando en el contenedor es un comando, script... algo que acaba.
                                Entonces, eso significa que los contenedores de un pod no pueden acabar?
                                - Kubernetes entiende que los contenedores de un POD NUNCA deben dejar de estar en ejecución.
                                - Si un contenedor de un POD deja de funcionar, Kubernetes entra en pánico.. y hace cuanto esté en su mano para que ese contenedor vuelva a estar en ejecución... Y SI LO TIENE QUE REINICIAR 18987 veces... es incansable!
                                - Kubernetes entiende que los contenedores de un JOB DEBEN dejar de funcionar... y de hecho, es lo que se espera que hagan... y por eso, cuando un contenedor de un JOB acaba, Kubernetes no se pone en pánico...
                                - Por eso, si un proceso de un JOB tarda más de lo que kubernetes espera que tarde, Kubernetes lo mata... y lo vuelve a arrancar... y así hasta que el proceso acabe... o hasta que se alcance el número máximo de intentos de reinicio, que es configurable.
                                    - POD: Servicios/Demonios/Aplicaciones
                                    - JOB: Comandos/Scripts/ETLs
- CronJobs                      Plantilla de JOB, junto con un CRON para indicar cada cuanto es necesario generar una "réplica" / instancia
                                de ese JOB, basándose en la plantilla de JOB que le hemos dado.

- Service                       
    - ClusterIP                 Es una VIPA de balanceo de carga con una entrada en DNS, que apunta a los pods que cumplen unas condiciones.
    - NodePort                  Cluster IP + NAT (port forwarding) para que se pueda acceder desde fuera del cluster a esa VIPA de balanceo de carga.
    - LoadBalancer              NodePort + VIPA de balanceo de carga en la red externa al cluster.
    - ExternalName              ---
- Ingress
- NetworkPolicy

- PV
- PVC
- StorageClass
- ReplicaSet

- ConfigMap
- Secret

- ClusterRole
- Role
- ClusterRoleBinding
- RoleBinding
- ServiceAccount

- ResourceQuota
- LimitRange



---

# Comunicaciones dentro de un cluster.. y fuera!

    192.168.2.200:80:   http://192.168.0.101:30080
                        http://192.168.0.102:30080
                        http://192.168.0.103:30080
                        http://192.168.0.111:30080
                        http://192.168.0.112:30080          mi-app -> 192.168.2.200
                        http://192.168.0.113:30080            |                             Navegador: http://mi-app
          Balanceador externo                             DNS Externo                   FedericoPC
                  |                                           |                          |
             192.168.2.200                              192.168.2.239               192.168.2.240
                  |                                           |                          |
    +-------------+-------------------------------------------+--------------------------+--- red de mi empresa (192.168.0.0/16)
    |
    |
    ++--- 192.168.0.101 - Nodo Maestro 1
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.101:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    |+-------- 10.0.0.201 ------ Pod CoreDNS
    ||                                   mi-bbdd  -> 10.0.1.101
    ||                                   mi-nginx -> 10.0.1.102
    ||                                   mi-ingress-controller -> 10.0.1.103
    ||
    ++--- 192.168.0.102 - Nodo Maestro 2
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.102:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    |+-------- 10.0.0.202 ------ Pod CoreDNS
    ||                                   mi-bbdd  -> 10.0.1.101
    ||                                   mi-nginx -> 10.0.1.102
    ||                                   mi-ingress-controller -> 10.0.1.103
    ||
    ++--- 192.168.0.103 - Nodo Maestro 3
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.103:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    ||
    ||
    ++--- 192.168.0.111 - Nodo Trabajo 1
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.111:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    |+------- 10.0.0.104 ---------Pod Ingress Controller (proxy reverso)
    ||                               contenedor nginx:80
    ||                                  en el fichero de configuración de este nginx: nginx.conf:
    ||                                    location / {                          \
    ||                                      server_name mi-app;                  \ Esta regla la de de alta un programa que viene junto con el nginx
    ||                                      proxy_pass http:/mi-nginx:8080;      /  En base lo definido en un INGRESS (INGRESS = REGLA DE CONFIGURACIÓN PARA EL PROXY REVERSO)
    ||                                   }                                      /  
    |+------- 10.0.0.103 ---------Pod NGINX
    ||
    ||
    ++-- 192.168.0.112 - Nodo Trabajo 2
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.112:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    |+------- 10.0.0.102 ---------Pod NGINX
    ||                               Contenedor NGINX:80.           El puerto se abre en la: 10.0.0.102:80    
    ||                                      En un fichero de configuración, quiero apuntar a la BBDD. Funcionaría: 10.0.0.101:3306: SI
    ||                                              Pero... el problema es otro: Si me falla el pod/nodo... y se recrea en otro sitio, no tengo garantía de IP
    ||                                      Teniendo un DNS interno, puedo registrar mi-bbdd -> 10.0.0.101.
    ||                                              Me resuelve esto el problema? NO.. por la cache DNS... si cambia el valor (IP) en el futuro. el nginx no se entera.
    ||                                      Teniendo una VIPA estable y un fqdn:       mi-bbdd:3307
    ||
    ++--- 192.168.0.113 - Nodo Trabajo 3
    ||                       Linux
    ||                          NetFilter
    ||                              10.0.1.101:3307 -> 10.0.0.101:3306
    ||                              10.0.1.102:8080 -> 10.0.0.102 | 10.0.0.103 :80
    ||                              10.0.1.103:8080 -> 10.0.0.104:80
    ||                              192.168.0.113:30080 -> 10.0.1.103:8080 (Kubernetes obliga a que este puerto sea superior al 30000) Por Forwarding
    ||                       Crio
    ++------- 10.0.0.101 ---------Pod MariaDB1
     |                               Contenedor MariaDB1:3306.      El puerto se abre en la: 10.0.0.101:3306
     |
     |
     | Red virtual.. que se apoya sobre la red física de mi empresa, y que me permite conectar los contenedores entre ellos. (10.0.0.0/16)


    Un servicio de tipo CLUSTER IP = es una VIPA que lleva asociada un nombre FQDN dado de alta en el DNS interno del cluster, y que apunta a los pods que cumplen unas condiciones (normalmente, que tengan una etiqueta concreta).
        Además, puede hacer balanceo de carga... caso que haya más de 1 pod que cumpla esas condiciones.
        Eso lo hace en automático NETFILTER.
        Una IP Estable a nivel de cluster... que ya me hace balanceo y que tiene asociado un nombre de red.

    Un servicio de tipo NODEPORT = es un servicio de tipo CLUSTER IP + NAT (port forwarding) a nivel de cada host, usando un puerto por encima del 30000.
    Me permite acceder desde la red de fuera del cluster a una IP DE BALANCEO (servicio) dentro del cluster.

    Un servicio de tipo LOADBALANCER = es un servicio de tipo NODEPORT + VIPA de balanceo de carga en la red externa al cluster.
        Eso lo puedo montar de varias formas. 
        - Los clouds me "regalan" (€€€€€) balanceadores de carga en sus redes. Y kubernetes sabe hablar con ellos, para pedirles ips.. e ir configurando la lista de redirecciones asociada.
        - Si me lo monto on-premise, necesitaré un balanceo de carga externo al cluster pero compatible con kubernetes: METAL-LB

En el kernel de linux, hay un programa llamado NETFILTER. Por ese programa del kernel de linux, pasan todos los paquetes de red que entran o salen del nodo.
IPTABLES? IPTables es solo un comando para dar de alta reglas en NETFLITER.

    En un cluster estandar de kubernetes, cuántos servicios tengo de cada tipo?

                            % ?
        ClusterIP           Todos menos 1           Comunicaciones internas
            -------------------------------------> 
        NodePort            0                       Comunicaciones externas
        LoadBalancer        1
            |
            v
            Proxy reverso / proxy inverso

Cómo se llama en Kubernetes al proxy reverso? Ingress Controller
Un ingress es otrra cosa...
Un ingress es una regla de configuración para nuestro ingress controller.
