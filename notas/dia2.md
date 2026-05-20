# Formas de desplegar apps/sistemas en kubernetes

Kubernetes lo único que traga son archivos de Manifiestos YAML.

## Cómo hacemos esos archivos de manifiesto YAML:

- A manopla!
- Mediante un chart de helm
  Chart = Plantilla personalizable.
          En la plantilla puedo activar o desactivar muchas cosas.
  Helm lo que hace es:
    PLANTILLA + PARAMETROS -> Manifiesto yaml
                values.yaml
        - Una vez tiene ese manifiesto YAML, puedo pedirle que se encargue él de desplegarlo en el cluster.
          Y en ese caso, la gestión del ciclo de vida del despliegue, también se la dejo a helm.
          Actualizaciones, el control de versiones del despliegue, el dar marcha atrás, el desmantelamiento cuando toque.
- Kustomize
  Es una herramienta oficial.. viene integrada con kubectl.
  Son otra forma de plantillas. Se usa mucho menos:
    - El lenguaje para definir plantillas no es tan potente como el de helm.
    - No tiene gestión del ciclo de vida del despliegue.
  Aunque helm no era la herramienta oficial, se ha convertido en el estándar de facto para la gestión de plantillas en kubernetes.

- Operadores... Esto si .. pero no...
  Es decir, no es en si un método de despliegue de cosas en kubernetes.
  Kubernetes ya hemos dicho que traga archivos de manifiesto YAML.

  Hay 2 partes al trabajar:
  1. Instalar el operador = Contratar a un tio experto.

    Yo puedo instalar un operador. Y lo que tengo instalado es el OPERADOR. No tengo programa. Tengo un TIO/A que sabe un huevo de ese programa.. pero no hay programa.

  2. Desplegar el programa = Darle al tio experto un manual de instrucciones EN UN LENGUAJE DE ALTO NIVEL.
  Al operador lo que le paso es un archivo de manifiesto YAML... que no está escrito conlos recursos típicos de kubernetes.. sino con recursos personalizados que el operador define (CRDs).
  El operador coge esos recursos personalizados, los interpreta, y a partir de ellos, genera los recursos típicos de kubernetes que hacen falta para desplegar el programa. Es decir, genera manifiestos en lenguaje Kubernetes.. para que se los trague.

  La pregunta es:

  > Cómo genero esos YAML que le paso al operador?

  - A manopla!
  - Mediante un chart de helm
  - Mediante kustomize


---

# IaC = Infraestructura como código.

No es tengo la infraestructura declarada en un lenguaje que puedo meter en un fichero.
El concepto real es:
VOY A TRATAR LA INFRAESTRUCTURA COMO SI FUERA CÓDIGO:
- control de versiones de la infraestructura
   v1.0.0 de la infra!
   v1.0.1 de la infra!
   v1.1.0 de la infra!
   v2.0.0 de la infra!
- Tener entornos donde desplegar la infraestructura, para probarla antes de ponerla en producción.
- Tengo control del ciclo de vida de la infraestructura.
  - Despliegue
  - Actualización
  - Dar marcha atrás
  - Desmantelamiento

    Yo tendré una v1.1.0 del producto, desplegado en una versión 1.0.0 de la infraestructura.
    Quiero actualizar el producto a la v2.0.0, pero para eso necesito actualizar la infraestructura a la v1.1.0.
    Y hay cierta correlación entre las versiones de la infra y las versiones del producto.

Pero luego hay otro concepto... Versión del despliegue!

    INFRA           DESPLIEGUE                 PRODUCTO
    v1.0.0          v1.0.0                      v1.0.0
                    v1.1.0
    v1.1.0          v2.0.0                      v2.0.0
                    v2.1.0
                    v2.1.1

Mi producto en v1.0.0 tiene una determinada funcionalidad. Y para ello necesita 1 servidor de BBDD y 1 cluster de servidores con 3 nodos al menos + balanceadores, reglas de firewall...(v1.0.0 de la infraestructura).
 Y creo la infra y hago un despliegue... con una determina configuración v1.0.0 del despliegue.
 Veo que funciona.. pero que me hace falta una funcionalidad que no he activado en el producto.
 Genero una nueva versión del despliegue: v1.1.0 del despliegue.. que activa esa funcionalidad.
    No cambio la versión del producto que instalo
    No cambio la versión de la infraestructura que tengo desplegada.
    Lo que cambio es la configuración del despliegue.
Sale nueva versión del producto... usa un Redis como caché... para que vaya más rápido todo.
Necesito ahora actualizar la infra metiendo 3 servers nuevos.. para acomodar ese Redis.
Qué versión del producto tengo? v2.0.0
La infra sube a 1.1.0... añade funcionalidad (3 servers nuevos)
El despliegue sube a v2.0.0... activa la funcionalidad de usar Redis como caché.

Después digo... voy a activar ahora una funcionalidad del producto que no había activado: v2.1.0 del despliegue. Envío de emails a los clientes.

Mierda... me equivoque con la formación del servidor de email.. y no manda emails: BUG en la configuración de despliegue...
Lo arreglo y hago una nueva versión del despliegue: v2.1.1 del despliegue... que arregla ese bug.

EN SERIO TIO!!! No me jodas.. que es to es un tostón.. en serio necesito llevarlo así?
Bienvenido al maravilloso mundo del software con el que los programadores llevamos DECADAS LIDIANDO. Y es que esto no es nada nuevo.. lo que pasa es que ahora con la infraestructura como código, el despliegue como código... pues se ha hecho más evidente.

Y esto encaja muy bien con HELM.
Helm me permite crear ficheros de parametros: values.yaml... Que tendrán su versión! Y que estarán en un git!
Y esa versión es la versión del despliegue...
que funciona sobre una determinada versión del CHART!
y que funciona sobre una determina versión del producto.
y que funcionará sobre una determinada versión de la infra!

---

# Esto es lo que llamamos el esquema semántico de versiones: SEMVER.

vA.B.C

                    Cuándo suben esos números?
    A   MAJOR       Breaking change. Cambios incompatibles con versiones anteriores.
    B   MINOR       Nueva funcionalidad 
                    o funcionalidad marcada como obsoleta
                    + Adicionalmente pueden venir bugfixes... Pero si solo vienen bugfixes, no se sube el número de versión MINOR.
    C   PATCH       Arreglo de bugs. BUG FIX

---

Tengo un cluster y quiero meter almacenamiento de tipo NFS.
Qué necesito para esto?
- Lo primero un backend que me proporcione ese almacenamiento NFS.(NAS...)
- Comunición por red con el backend
- Permisos
- cliente nfs en los nodos del cluster
  apt install nfs-common
  yum install nfs-utils 
- PV

Esto implica que en todos los nodos del cluster tengo que tener instalado el cliente nfs.
Cómo puedo asegurarme de eso? Desplegando un DaemonSet que se encargue de eso.
El daemonset ya me asegura que en cada nodo del cluster se ejecute un contenedor. Ese contenedor hago que instale el cliente nfs. Y así me aseguro de que en cada nodo del cluster tengo el cliente nfs instalado.

Antiguamente en ElasticSearch, había que activar uan configuración a nivel de kernel en cada nodo del cluster:
vm.max_map_count=262144.. por defecto en la mayor parte de las ditros de GNU/Linux está en 65536.. y eso no es suficiente para ElasticSearch.
Entonces, para asegurarnos de que esa configuración se aplica en cada nodo del cluster, también se desplegaba un DaemonSet que se encargaba de eso.

Hoy en día lo hacen de otra forma: Mediante init containers.
El init container tiene una ventaja:
- No deja nada corriendo de perpetuo en el nodo del cluster. Es decir, una vez que ha hecho su trabajo, se termina y no consume recursos.
Inconveniente:
- Como alguien cambie la configuración de ese dato posteriormente, no hay nadie que se encargue de volver a ponerlo en su sitio.

Los daemon set tiene sus ventajas y sus inconvenientes:
Ventajas:
- Siempre hay un contenedor corriendo en cada nodo del cluster, que se encarga de mantener la configuración que le hemos dicho.
Inconvenientes:
- Consume recursos en cada nodo del cluster, aunque no se necesiten.

---

# PV

Un PV no es un volumen de almacenamiento.
El volumen existe en un backend... en un NAS, en una cabina (LUN).
Un PV es el registro que hacemos en kubernetes de ese volumen de almacenamiento que existe en el backend.

Esto es importante. El PV no crea el volumen de almacenamiento. Solo lo registra en kubernetes...
Si yo registro algo que no está creado en el backend, pues no va a funcionar. El PV no va a funcionar.

Es más, puedo tener una LUN en una cabina con 10Tbs, y registrarlo en kubernetes como si fuera de 1 Tb.
Y kubernetes ni se entera.. para kubernetes tiene 1 Tb de almacenamiento... aunque en realidad tenga 10 Tbs.

El PV ES SOLO EL REGISTRO que hacemos en kubernetes de un volumen de almacenamiento que DEBE EXISTIR PREVIAMENTE EN EL BACKEND.

Un PV tiene 2 partes:
- Metadatos del volumen que existe en el backend
- Información de cómo acceder a ese volumen de almacenamiento en el backend.

```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
    name: mi-pv
spec:
    # Metadatos
    capacity:
        storage: 1Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: rapidito-redundante
    # Información de acceso
    nfs:
        server: mi.nas
        path: /ruta/del/volumen
```

# PVC 

Una solicitud de almacenamiento, que hace el que necesita almacenamiento para desplegar su aplicación, a kubernetes.
Esto no es nuevo... de toda la vida, cunado alquien queria desplegar una aplicación, y esa aplicación necesitaba almacenamiento, me hacian una petición:
- Vía email
- Vía Ticket de soporte
- Vía llamada telefónica

Oye Menchu, que necesito 10 Gbs rapiditos y redundantes... de tal tipo de almacenamiento:
- Bloques       iSCSI, Fibre Channel, RBD...
                En un almacenamiento de bloque, montarlo en 2 servidores a la vez, donde ambos puedan leer y escribir en el mismo bloque, es complicado. Normalmente este tipo de almacenamiento es para montarlo en un solo servidor.
                O si lo quiero montar en solo lectura... entonces me la puedo jugar más a montarlo en varios servidores a la vez... pero si lo quiero montar en lectura-escritura... entonces lo normal es montarlo en un solo servidor.
                    BBDD, Sistemas de mensajería, indexadores... suelen usar almacenamiento de bloques.
                    No es volumen compartido... y hay menos sobrecarga de protocolo.
- Archivos      NFS, CIFS...
                Aquí no tengo problema en montar en varios servidores a la vez, y que todos puedan leer y escribir en el mismo directorio.
                Aunque si todos intentan escrbir en el mismo archivo a la vez, pues eso ya es otra historia... pero a nivel de directorio no hay problema.   
                    Por ejemplo, para una app web.. que necesita guardar archivos que suben los clientes... lo normal es usar almacenamiento de archivos. Porque lo que necesito es un directorio compartido donde todos los servidores de mi app web puedan leer y escribir los archivos que suben los clientes.
- Objetos       S3, Swift...    
                    También podría montar un almacenamieento de objetos. 
                    Para backups usamos mucho almacenamiento de objetos... porque lo que necesito es guardar secuencias de bytes, y luego acceder a ellas por una clave. No necesito montar un directorio compartido donde pueda leer y escribir archivos... lo que necesito es guardar secuencias de bytes, y luego acceder a ellas por una clave.

No funcionan ni parecido los almacenamientos.
Si tengo almacenamiento de bloques, lo que entrego es un "disco sin preparar" .. ya se ocupará el que lo reciba de darle formato, crearle particiones, etc...
Si tengo almacenamiento de archivos, lo que entrego es un "directorio vacío", que usará un almacenamiento físico por debajo... pero el cliente se arbstrae de eso... y lo que ve es un directorio vacío, que puede usar como quiera, con los comandos típicos de un sistema de archivos:
  - mkdir
  - touch
  - rm
Si tengo almacenamiento de objetos, lo que entrego es una especie de BBDD donde meto secuencias de bytes, y cada secuencia de bytes la identifico con una clave. Y accedo a ello posteriormente por la clave.
    Es una forma más sofisticada de almacenamiento que los archivos... con los archivos para ciertas cosas nos quedamos jodidos.
    Qué tal en windows o linux tener una carpeta que tenga dentro 1M de archivos? Le escuece... ya acabamos creando estructuras de directorios forzadas para evitar eso... con el almacenamiento de objetos no hay ese problema... puedo meter 1M de objetos sin que me escueza.


```yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mi-pvc
spec:
    # Metadatos
    resources:
        requests:
            storage: 1Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: rapidito-redundante
```


Una PVC al final hay que asoiarla a un PV... que a su vez asocia la PVC al volumen de almacenamiento que hay en el backend.

    Volumen real de almacenamiento <-> PV (registro en kubernetes) <-> PVC (solicitud de almacenamiento)
                                    ^                               ^ 
                                    1                               2

Quién hace la vinculación 1? El que crea el PV. Que podrá ser:
- Un humano 
- Un programa: Provisionador dinamico de volumenes de almacenamiento.

Quién hace la vinculación 2?
- Kubernetes!
  PV y PVCs se crean por separado... y kubernetes se encarga de vincularlos... buscando PVs compatibles con la PVC.
  Es el TINDER de los volumenes. Busca algún PV que haga match con la PVC... y los vincula.
  Para ello se basa en los metadatos de ambos:


```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
    name: mi-pv
spec:
    # Metadatos
    capacity:
        storage: 2Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: rapidito-redundante
    # Información de acceso
    nfs:
        server: mi.nas
        path: /ruta/del/volumen
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mi-pvc
spec:
    # Metadatos
    resources:
        requests:
            storage: 1Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: rapidito-redundante
```
En un caso como éste, el PV cumple con los requisitos de la PVC... y kubernetes los vincula.

> Pregunta... kubernetes le entrega 1Tb o los 2?

Le entrega los 2... Un volumen no lo puedo partir a la mitad con un cuchillo!
Es como si voy a MediaMarkt.. y pido un disco para poder guardar mis fotos del verano que ocupan 789Gb
El del mediamark me dice.. pues tengo 1 disco de 1Tb, que te sirve...
Pero no lo corta ocn cuchillo para darte solo los 789Gb que necesitas... te da el disco entero de 1Tb... y ya tú te las apañas para meter solo lo que necesitas dentro del disco.

De hecho, kubernetes no sabe siquiera si en realidad ese columen tiene o no tiene los 2 Tbs en el backend... lo que sabe es que el PV tiene registrado que tiene 2 Tbs... y como la PVC pide 1 Tb, pues kubernetes dice.. pues este PV cumple con los requisitos de esta PVC... y los vincula.

# StorageClass

Un storage class es un TIPO de ALMACENAMIENTO pre-registrado en kubernetes.
Un triste METADATO!
Un String!

Nada más!

De hecho... si tengo un cluster de kubernetes donde existen los storage class:
- rapidito
- rapidito-redundante
- lentito
- lentito-redundante

Puede un pvc o un pv usar un storage class name: `encriptado`?

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: mi-pvc
spec:
    # Metadatos
    resources:
        requests:
            storage: 1Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: encriptado
```
SIN PROBLEMA!

Y un PV como este?
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
    name: mi-pvc
spec:
    # Metadatos
    capacity:
        storage: 1Tb
    accessModes:
        - ReadWriteOnce
    storageClassName: encriptado
    # Información de acceso
    nfs:
        server: mi.nas
        path: /ruta/del/volumen
```
SIN PROBLEMA!

Y kubernetes hace match? SIN PROBLEMA !

No necesito tener un storageclass pre-registrado para poder usarlo. Esto no hace falta en kubernetes.

Lo que me da kubernetes es otra cosa.. 
Es la posibilidad de vincular un Storageclass predefinido a un Provisionador dinámico de volumenes de almacenamiento.

## Un provisionador dinámico de volumenes de almacenamiento 

Es un programa que instalo independientemente de kubernetes, y que:
- Monitoriza las PVCs que se crean en kubernetes.
- Si una es de su storage class,
  - Crea el volumen de almacenamiento en el backend
  - Registra ese volumen de almacenamiento en kubernetes mediante un PV, con el mismo storage class que la PVC.
- Kubernetes hace match entre el PV y la PVC, y los vincula.

No es obligatorio usar Provisionadores dinámicos de volumenes de almacenamiento.
- Puedo yo a manopla ir a un backend, crear un volumen de almacenamiento
- Y luego registrar ese volumen de almacenamiento en kubernetes mediante un PV.

Pero.. esto es muy poco práctico. No tiene sentido. Y al final siempre acabamos usando provisionadores dinámicos de volumenes de almacenamiento.

Y tendremos varios provisionadores dinámicos de volumenes de almacenamiento.
Y tendremos varios storage class predefinidos, cada uno vinculado a un provisionador dinámico de volumenes de almacenamiento diferente.

Tendré storageclasses al menos para:
- Almacenamiento de bloques
- Almacenamiento de archivos

Eso es un mínimo!
Posiblemente tenga más de 1 tipo de almacenamiento de bloques... y más de 1 tipo de almacenamiento de archivos... y por tanto, tendré más storage class predefinidos.

- block-gold
- block-silver
- block-bronce
- file-gold
- file-silver
- file-bronce
- object-gold
- object-silver
- object-bronce

Puedo tener almacenamiento de objetos gold.. sobre nvme... para usarlo de CDN para servir las imágenes de mi app web... y así que vayan rapidito a los clientes.

Y me interesa tener un almacenamiento de objetos de tipo bronze sobre rotaciones de 5400 rpm... para guardar los backups de mi BBDD... que no necesito que vayan tan rapidito... y así me ahorro un dinerillo.

La diferencia en pasta será brutal!

> Para qué existen en kubernetes los PV y los PVC?

Para separar responsabilidades.
- Negocio/desarrollo -> PVC
- Administración (normalmente delegado a un provisionador dinámico de volumenes de almacenamiento) -> PV

Kubernetes está en medio.


---

# Comunicaciones

## Servicios

### ClusterIP

VIPA = Virtual IP Address que hace balanceo + FQDN en el DNS interno del cluster
Sirven para comunicaciones internas entre los pods.

Qué resuelven?
- El problema de la IP dinámica de los pods.
- Escalabilidad (por el balanceo)

### NodePort

Cluster IP + NAT (a nivel de puerto... port forwarding) para exponer el servicio fuera del cluster.
El servicio se expone un puerto por encima del 30000 en cada nodo del cluster.

### LoadBalancer

NodePort + VIPA en la red fuera del cluster, con balanceo entre los nodos del cluster (apuntando a los puertos NodePort de cada nodo del cluster).

## IngressController

Son 2 cosas:
- Proxy reverso (nginx, apache httpd, haproxy, envoy...)
- Programa que monitoriza los INGRESS que se crean en el cluster y modifica la configuración del proxy reverso concreto que usemos en base a esos INGRESS.

## Ingress

Una regla de configuración de proxy reverso.


## Ejemplo de configuración de regla de proxy reverso en Apache httpd:

```
<virtualHost *:80>                                  Si llegan peticiones a este puerto, haz lo siguiente:
    ServerName mi-app                               Si la petición es para este dominio, haz lo siguiente:
    ProxyPreserveHost On                            Mantén el host original que viene en la petición
    ProxyPass / http://mi-nginx:8080/               Haz de proxy reverso a esta dirección. 
    ProxyPassReverse / http://mi-nginx:8080/        
</virtualHost>
```
## Si trabajo con nginx:

```
server {
    listen 80;                                      Si llegan peticiones a este puerto, haz lo siguiente:
    server_name mi-app;                             Si la petición es para este dominio, haz lo siguiente:
    location / {                                    Si la petición es para esta ruta, haz lo siguiente:
        proxy_pass http://mi-nginx:8080;            Haz de proxy reverso a esta dirección.
    }
}
```

## Si trabajo con haproxy

```
frontend http-in
    bind *:80                                      Si llegan peticiones a este puerto, haz lo siguiente:
    acl host_mi_app hdr(host) -i mi-app            Si la petición es para este dominio, haz lo siguiente:
    use_backend mi_nginx if host_mi_app             Haz de proxy reverso a esta dirección.
backend mi_nginx
    server mi-nginx mi-nginx:8080                 El servidor al que hago proxy reverso.
```

El objetivo de un ingress es tener una sintaxis UNICA para configurar el proxy reverso... con independencia del proxy reverso concreto que usemos.

Junto con el proxy reverso, viene un programa que monitoriza los INGRESS que se crean en el cluster, y en base a ellos, va modificando la configuración del proxy reverso concreto que usemos.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: mi-ingress
spec:
    rules:
    - host: mi-app                          # Si la petición es para este dominio, haz lo siguiente:
      http:
        paths:  
            - path: /                       # Si la petición es para esta ruta, haz lo siguiente:
              pathType: Prefix
              backend:
                service:
                  name: mi-nginx            # El servicio al que quiero hacer proxy reverso.
                  port:
                    number: 8080            # El puerto del servicio al que quiero hacer proxy reverso.
```

Aqui es donde configuro también si quiero o no usar SSL.

En Openshift NO SE USAN INGRESS... 
SE USAN ROUTES...

### Routes de Openshift

Es como un ingress... pero con una sintaxis diferente... y con algunas funcionalidades adicionales que no tiene el ingress.

Entre ellas:
- Gestión de comunicaciones seguras:
  - Podemos configurar el proxy reverso en modo Passthrough
  - Podemos configurar el proxy reverso en modo Re-encrypt
  - Podemos configurar el proxy reverso en modo Termination

    POD (http/https) <- SERVICIO INTERNO (VIPA) <-  PROXY REVERSO <- Cliente
                                                          ^
                                                        Route

    Hay 2 comunicaciones:
        Cliente         -> https        -> Proxy Reverso
        Proxy Reverso   -> http/https   -> Servicio Interno

    Mi servicio puede estar configurado por http o https.
    El cliente siempre quiero que se comunique con el proxy reverso por https.

    Termination (edge) -> La comunicación entre proxy reverso y servicio interno es por http. 
    Re-encrypt -> La comunicación entre proxy reverso y servicio interno es por https.
        El proxy reverso tiene su propio certificado y clave privada.
        El servicio (pod) tiene su propio certificado y clave privada.
        La comunicacion entre cliente y proxy reverso es por https, usando el certificado del proxy reverso.
        El proxi reverso mira la petición y la reenvía al servicio interno por https, usando el certificado del servicio interno.
    Passthrough -> La comunicación entre cliente y servicio interno es por https, y el proxy reverso no interviene mas que en enrutar la petición al servicio interno.
        El proxy reverso tendrá o no certificado y clave privada.... pero no se usa en esta comunicación.
            Realmente el proxy reverso siempre tiene certificado y clave privada (WILDCARD)

    Adicionalmente en Openshift registramos en un DNS externo el dominio apuntando a una VIPA fuera del cluster (load balancer)
    Pero también se registra en dns que todas las peticiones que vayan a ese dominio, subdominios incluidos (*.mi-app) vayan a esa VIPA fuera del cluster (load balancer).
    Y nos quitamos el andar creando entradas en DNS cada vez que queremos crear un nuevo servicio interno y exponerlo por el proxy reverso.

    Eso lo podemos hacer sin openshift.

        DNS -> IP FIJA -> ROUTER -> Nat -> ROUTER -> Nat -> VIPA (MetalLB) -> Nodo del cluster
         ^                           80               80
         *.ivanosuna.com            443              443

    En mi caso, tengo instalado CERT MANAGER (es un operador, con sus propios CRDs) que se encarga de gestionar certificados y claves privadas para los servicios del cluster (contra let's encrypt) y de renovar esos certificados cuando toque.

    Openshift también usa cert-manager.. pero viene instalado de serie.. no es algo que yo necesite instalar.

---

# Autenticación

Cuando me conecto contra un cluster de kubernetes, desde un cliente (kubectl, oc, dashboard gráfico de kubernetes o de openshift)...
Con quién habla realmente el cliente?
                            Kubernetes
                            ----------------------------------------
 Yo humano  ->  cliente ->  ApiServer

Y ... que hay que pasarle al APISERVER para que se digne a contestarme?
    - Service Account           <- Me identifica
    - Token de autenticación    <- Para que ApiServer me pueda autenticar
      (similar a una contraseña) 

Ese service account tendrá asignados unos roles (RBAC) que me darán permisos para hacer unas cosas sí, y otras no.

Además de YO, quién puede necesitar un Service Account? PROGRAMAS.
Cómo cuales? Para hacer qué?
- CI/CD...
  Al final en un Argo, Jenkins,.. lo que hacemos es llamar a kubectl. 
- IngressController
  Qué dijimos que hace un Ingress Controller?
  Monitoriza los INGRESS que se crean en el cluster y modifica la configuración del proxy reverso concreto que usemos en base a esos INGRESS.
  Es decir.. que comandos ejecuta?
    kubectl get ingress --all-namespaces --watch                 -> nombre/id del ingress que se ha creado
  Para leer luego los datos de un ingress que se haya cargado
    kubectl describe ingress mi-ingress                         -> datos del ingress en formato yaml
- Provisionadores dinámicos de volumenes de almacenamiento
  Qué dijimos que hace un provisionador dinámico de volumenes de almacenamiento?
  Monitoriza las PVCs que se crean en kubernetes.
    kubectl get pvc --all-namespaces --watch                 -> nombre/id de la pvc que se ha creado
  Para leer luego los datos de una pvc que se haya cargado
    kubectl describe pvc mi-pvc                         -> datos de la pvc en formato yaml
  Crear pv:
    kubectl create -f mi-pv.yaml

En realidad, este tipo de programas no usan kubectl... directamente atacan al api-server, como un cliente más... por http.
Pero para hacer ciertas operaciones concretas.A
Ahí es donde entran los roles.

## Roles en kubernetes

Es una serie de permisos. 
Cada permiso es:
- Tipo de objeto sobre el que se puede hacer una determinada operación ( API Version + Kind)
- Los verbos permitidos (operaciones): LIST, GET, WATCH, CREATE, UPDATE, DELETE...

Los roles se definen sobre objetos se agrupan en namespaces.

> Todo recurso de kubernetes se crea dentro de un namespace? NO

Ejemplos que si:
Pod, Deployment, Service, Ingress, PVC, ConfigMap, Secret, ServiceAccount, Role, RoleBinding, NetworkPolicy...

Ejemplos que no van dentro de un namespace:
PV, StorageClass, Node, Namespace, ClusterRole, ClusterRoleBinding...

Los permisos que aplican sobre este tipo de objetos que no van definidos a nivel de namespace se declaran en ClusterRoles, y no en Roles.


    ServiceAccount   <-     RoleBinding         ->  Role (tipos de recursos + operaciones permitidas)
                     <-     ClusterRoleBinding  ->  ClusterRole (tipos de recursos + operaciones permitidas)

A nivel de kubernetes esto es todo.
Pero Openshift pone otra cosa encima de la mesa.

Usuarios y Grupos de usuarios.
En kubernetes no hay usuarios ... un usuario cuando se conecta usa un ServiceAccount.

Eso no está mal... pero en muchos casos no es lo mejor:
- El token ese es muy engorroso... yo usuario no me acuerdo de eso.
- No tengo MFA (autenticación multifactor) con un token de servicio.
- Ese token lo gestiona Kubernetes.
  Lo normal es que en mi empresa tenga un servicio centralizado de Autenticación:
    - Directorio Activo / LDAP
    - IAM (Gestión de identidades y accesos) en la nube o en local (Keycloak, Okta, Auth0...)

Esto es lo que hace Openshift... me permite llevar una gestión de la autenticación de los usuarios a través de un servicio externo de autenticación... y luego, una vez que el usuario se ha autenticado contra ese servicio externo de autenticación, Openshift le asigna un ServiceAccount concreto dentro del cluster, con unos roles concretos.

En un cluster de kubernetes.. podemos montarlo.

Openshift tiene la identificación del usuario, su vinculación a un ServiceAccount.. que tendrá unos determinados roles y cluster roles asociados.
Pero la autenticación la delega en sistemas externos.
Lleva un sistema mini, interno, de autenticación:  htpasswd. Ese siempre se usa.. para tener 1 usuario: admin general, que en caso de caída del servicio externo de autenticación, pueda seguir entrando al cluster.
Pero siempre lo configuramos contra un srvicio externo de autenticación... no tiene sentido usar ese htpasswd... es muy cutre.. no tiene MFA... y no es nada práctico para los usuarios.

La autenticación y la identificación van por separado.

Openshift gestiona la identificación / autorización
Openshift no gestiona la autenticación

Pero cuidado... si elijo por elemplo GITHUB como herramienta de AUTENTICACIÓN, eso no significa que cualuier usuario de github pueda entrar en mi cluster.
Cuando vaya a entrar en Openshift, Openshift me redirigirá a github para que me autentique... y una vez que me autentique, y github, si tengo cuente en github me autenticará, Openshift mira si ese usuario que ya sabe que es quien dice ser, lo tiene registrado internamente. Si no lo tiene registrado... no entra!

Para este chiringuito es para lo que en Openshift tenemos los conceptos de USUARIOS y GRUPOS DE USUARIOS.
Los grupos de usuario me permiten hacer asignación de roles más cómoda.
---

Identificarse:  Decir quien soy
Autenticarse:   Demostrar que soy quien digo ser
Autorización:   Determinar, sabiendo que eres quien dices ser, qué puede hacer o no


---
# Operación diaria de un administrador de Openshift/Kubernetes

## Crear usuarios y asignarles permisos -> namespace

No se trata solo de limitar qué cosas puede o no tocar dentro de ese namespace...
Sino también de limitar la cantidad de recursos físicos del cluster de la que puede hacer uso:
- LimitRange
- ResourceQuota

Esos dados de abajo: requests y limits los marca desarrollo/negocio.. quien despliega una app.
Pero claro.. auncha es Castilla:
```yaml
 # ...
 resources:
    requests:
        cpu: 20000m
        memory: 40Gi
    limits:
        cpu: 40
        memory: 40Gi
```
No tio...

Lo que hacemos es cerrar la cantidad de recursos que pueden consimirse. Y lo hacemos a 2 niveles:
- SIEMPRE: ResourceQuota a nivel de namespace.
- A veces (pocas) también: LimitRange a nivel de namespace.

# ResourceQuota

Limita el total de recursos físicos del cluster que pueden consumir dentro de ese namespace.
Puedo limitar:
- CPU
- Memoria
- Cantidad de almacenamiento efímero (volumenes de almacenamiento que se montan en los pods)
- Cantidad de almacenamiento persistente (PVCs), incluso por tipo de almacenamiento (storage class)
- Cantidad de objetos de kubernetes (pods, deployments, services, ingresses, configmaps, secrets, etc...)

  No puedes hacer más de 3 pvc
  No puedes hacer más de 10 pods
  No puedes usar más de 10 cores
  No puede usar más de 20 Gbs de RAM

Limitamos totales. Y en la mayoría de casos es suficiente.
Quien quiera desplegar.. me pide una cantidad de RAM y de CPU.. y de almacenamiento y paga acorde a eso.

```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
    name: mi-resource-quota
spec:
    hard:
        requests.cpu: "10"
        requests.memory: 20Gi
        limits.cpu: "10"
        limits.memory: 20Gi 
        persistentvolumeclaims: "3"
        rapidito-redundante.storageclass.storage.k8s.io/requests.storage: 1Tb
        pods: "10"
        persistentvolumeclaims: "3"
```

# LimitRange

Aquí vamos más fino.

Auí limito a nivel de cada pod o de cada contenedor, la cantidad de recursos físicos que pueden consumir (en request y en limit).

Para qué sirve esto? Para que no tenga que jugar al tetris con los pods... cuando se ubiquen.A 
mi , como administrador del cluster (que es quién configura estos objetos: ResourceQuota LimitRange) me intersan pods cuanto más pequeños mejor. Eso le da flexibilidad al Scheduler a la hora de ubicar un Pod.

Si la gente me pide pods con 10 cores y 40Gbs de RAM... a ver donde cojones lo meto. Y cuando lo meta.. me deja la máquina exhausta.

Llevarlo al límite. Tengo 6 nodos con 64 Gbs de RAM... 
Y todo el mundo pide pods de 40Gbs de RAM... No entra más que un pod por nodo. y me quedan 24Gb sin poder usarlos.

Entonces, en total, podrán hacer uso de 40. pero a cachitos. Prefiero 10 pods de 4 que uno de 40
10 pods de 4... les busco hueco. 1 de 40 a ver que hostias hago con él.

Esto es un control mucho más fino de recursos. No nos volvemos locos. Pocas veces configuramos con esmero los Limit Range... solemos uno de plantilla y ya... lo aplicamos a todo del mundo.

Hay programas que gastan mucha más RAM si los parto en trozos... si los escalo en horizontal, que si los escalo en vertical.

ELASTICSEARCH!

    Guarda datos de indexación. (GOOGLE)
    Tengo un huevo de páginas de datos... indexadas: 40Gbs.

    Si los pongo en 1 pod... necesito 40Gbs de RAM
    Si los pongo en 2 pods, necesito 2 pods de 25Gbs. Coño.. ahora son 50Gb
    Si los pongo en 3 pods, necesito 3 pods de 20Gbs. Coño.. ahora son 60Gb

    Elastic por ejemplo guarda por un lado los términos de búsqueda y por otro dónde aparecen esos términos.
    El listado de términos lo necesita en cada nodo. Otra cosa es que un documento se guarde en un nodo y otro documento que haya indexado en otro nodo... Pero en ambnos puede habrá muchas palabras comunes... que se guardan en ambos.

MI OBJETIVO AL ADMINISTRAR UN CLUSTER FORMZAR ESCALADO HORIZONTAL.
Eso hace que cuando el programa no esta en uso, quito replicas para ganar hueco!
Eso lo consigo con LimtRange bajitos.. y el que NECESITE MAS que pida... y estudiamos.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: mi-limit-range
spec:
    limits:
    - type: Container
      max:
        cpu: "4"
        memory: 20Gi
      default:
        cpu: "2"
        memory: 8Gi
      defaultRequest:
        cpu: "1"
        memory: 4Gi
    - type: Pod
      max:
        cpu: "10"
        memory: 40Gi
      default:
        cpu: "4"
        memory: 16Gi
      defaultRequest:
        cpu: "2"
        memory: 8Gi
```

### Recursos a nivel de contenedor.

Cuando definimos un contenedor dentro de una plantilla de POD, definimos un bloue con los recursos que ese contenedor necesita para funcionar:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: mi-pod
spec:
    containers:
    - name: mi-contenedor
      # ...

      resources:
        requests:
            cpu: 2000m
            memory: 8Gi
        limits:
            cpu: 4
            memory: 20Gi
```

Lo primero es entender bien esto:
- Requests          Lo que kubernetes le va a garantizar al contenedor caso que consiga ubicarlo en un nodo.
- Limits            Lo que le dejará condumir como máximo si hay hueco en el nodo en el que ya esté ubicado.

Quién hace uso de los request es el Scheduler, para ubicar el contenedor en un nodo del cluster que tenga al menos esa cantidad de recursos físicos disponibles.
El scheduler tiene en cuenta MUCHAS VARIABLES... una de ellas los request de recursos físicos que el contenedor necesita para funcionar.

    Tengo un contenedor que pide 500m de cpu y 2Gi de memoria.
    El contenedor tiene asignados unos limites de 2 cpu y 4Gi de memoria.

                                            Uso                 Libre actual        No comprometido
    Nodo1:       8 cores      32 Gbs                           0 cores   0 Gbs    0 cores    0 Gbs
     mi-pod-1    2 cores       8 Gbs        2 core    8 Gb
     mi-pod-2    2 cores       8 Gbs        2 core    8 Gb
     mi-pod-3    2 cores       8 Gbs        2 core    15 Gb
     mi-pod-4    2 cores       8 Gbs        2 core    1 Gb

Al scheduler el libre actual se la trae al peiro. 
Solo se fija en otro dato: LO NO COMPROMETIDO!

Si un pod se ha pasado de cores... con respecto a lo garantizado.. si hacen falta, se le quita el pie del acelerador... Linux empieza a encolar las peticiones de los hilos a la CPU... y el contenedor va a ir más lento... pero no va a afectar al resto de contenedores que estén en el mismo nodo del cluster.

A un proceso de SO le puedo ralentizar las peticiones que hace a la CPU... pero no puedo quitarle un trozo de RAM que ya le he asignado... Qué trozo le quita el SO? Donde tiene el código del programa? donde tiene los datos? No sabe... No puede.

Si el pod4 quiere 3 Gbs, y hay que dárselos, porque le han sido garantizados,
Kubernetes tira un SigQuit al pod3.. que se ha pasado de RAM: OYE MACHO APAGATE, que te voy a reiniciar para liberar RAM.
Y le configura un timeout... Si no contesta... le manda un SigKill (kill -9)... y lo cruje por REAL DECRETO!
Ale, ya hay hueco para que el pod4 pueda coger los 2 Gbs adicionales que le han sido garantizados.

OJO A LOS LIMITES DE MEMORIA!
Ponemos siempre lo mismo que tenemos puesto en el request. ESO ES LA BUENA PRACTICA.
El limit de RAM = request de RAM
El limit de CPU = Lo que me de la real gana! (no importa!)

Es igual que en JAVA... que podemos configurar el valor de inicio y maximo de la RAM de la JVM:
    java -Xms2G -Xmx4G MiPrograma
    En JAVA la recomendación es poner el mismo valor para ambos: -Xms2G -Xmx2G

El punto es simple.. Si un programa va a necesitar 4 Gbs... que los pida de antemano para que cuando hagan falta, existan...

Para qué existe entonces esa opción? Para casos especiales... donde me vale MIERDA si un pod es crujido.
- Me vale mierda que una BBDD de producción sea crujida?        NO -> limit de RAM = request de RAM = 4Gi
- Me vale mierda que un servidor de apps sea crujido?           NO -> limit de RAM = request de RAM
- Me vale mierda que un servidor de mensajería sea crujido?     NO -> limit de RAM = request de RAM

- Me vale mierda que el provisionador de volumenes sea crujido? SI
  Ese programa no esta prestando servicio continuo. 
  Entra a funcionar cuando alguien pide un PVC... y tarda 3 segundos en dárselo. 2 veces al mes.
  Tener una cantidad de RAM no disponible para ese programa, es una putada... para 3 veces al mes que entra 5 segundos.
  Y no tengo un provisionador.. que tengo 10.
  Y un certmanager.. para generar certificados. Cada cuanto genero/actualizo un certificado SSL? 3 veces al mes.
  Y sigue sumando.
  
  A este tipo de programas les suelo poner un request muy bajo: 50m y 16Mi y un limit mucho más generoso: 500m y 1Gi... para que tengan hueco para crecer si les hace falta... pero que no me jodan el cluster por un pico de consumo puntual.

  Si no puede hacer su trabajo... cuando sea necesario.. pues habrá que moverlo a otro nodo... esperar... o lo que sea... pero no me jode el cluster bloqueando recursos que no están usando.

  Que el programa se cruje a la mitad... que reinicie.. que el volumen en lugar de crearse en 5 segundos se crea en 10 minutos. Me la pela! Estoy en un despliegue... El programa añun no está dando servicio. Cuando dé 
  servicio, tendré que cumplir un SLA... pero entre tanto.

Una BBDD siempre usara el 100% de la RAM... y está bien
Y si le das más usará el 100% de la RAM
Y si le das toda la RAM del universo usará el 100% de la RAM
Guarda en caché TODO lo que entra. Mientras más RAM. más cosas mete en caché.
Si la CPU está al 100% ahí si estamos jodidos. Aquí:
- Si es puntual, no pasa nada... para eso hay un límite.
- Si no es puntual: Me toca escalar:
  - Vertical: Sube CPU
  - Horizontal: Más réplicas

Un programa eficiente debería hacer uso del 100% de la RAM siempre!
El propio SO.. según lo uso come toda la ram del universo:
 Caches y buffers.

Tengo que asegurarme que haya suficiente hueco en RAM para caché, para tener un buen rendimiento.

# Milicores:

Estamos usando Linux, un kernel de SO de tiempo compartido.
Las cifras de cpu no son CORES, son tiempo de uso de CPU.

2 cores = 2000m

Significa que se dejará usar al proceso(s) que se ejecuta dentro del contenedor, el equivalente a 2 cores completos de CPU por unidad de tiempo.
Puede ser que el trabajo se reparta entre 8 cores... a los que en una unidad de tiempo no se deja usa más del 25% de cada uno... pero que en total sumen el equivalente a 2 cores completos de CPU por unidad de tiempo.
