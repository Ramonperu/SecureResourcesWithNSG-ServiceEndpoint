

# Secure Resources NSG and Service Endpoint

Bienvenidos a la guía sobre la seguridad de recursos con NSG y Service Endpoint en Azure, este resumen practico esta hecho gracias a la documentación oficial de Microsoft https://learn.microsoft.com/en-us/training/modules/secure-and-isolate-with-nsg-and-service-endpoints/?wt.mc_id=certnurture_StayCurrent30DayReminder_email_wwl&ns-enrollment-type=Collection

Realizada para practicar para la renovación de la certificación **AZ-104** a **Marzo de 2024**

## Control de acceso a la red con NSG

### 

<img align="center" src="/img/1ºimagenn.PNG" style="zoom:50%;" />



Los grupos de seguridad de red en Azure gestionan el tráfico entrante y saliente hacia y desde los recursos de la plataforma. Estos grupos incorporan reglas de seguridad que los usuarios configuran para permitir o bloquear el flujo de datos. Permiten filtrar el tráfico entre máquinas virtuales o subredes, ya sea dentro de una red virtual o desde Internet.

Aplicar un grupo de seguridad de red a una subred en lugar de interfaces de red individuales puede reducir los esfuerzos de administración y gestión. Este enfoque también garantiza que todas las máquinas virtuales dentro de la subred especificada estén protegidas con el mismo conjunto de reglas.

A cada subred e interfaz de red se le puede aplicar un grupo de seguridad de red. Los grupos de seguridad de red admiten TCP, UDP e ICMP y operan en la capa 4 del modelo OSI.

### Reglas de seguridad predeterminadas

Trafico entrante

| Prioridad | Nombre de la regla            | Descripción                                                  |
| :-------- | :---------------------------- | :----------------------------------------------------------- |
| 65000     | PermitirVnetInbound           | Permitir entradas provenientes de cualquier VM a cualquier VM dentro de la red virtual |
| 65001     | AllowAzureLoadBalancerInbound | Permitir el tráfico desde el equilibrador de carga predeterminado a cualquier VM dentro de la subred |
| 65500     | Denegar todo incluido         | Denegar el tráfico de cualquier fuente externa a cualquiera de las VM |

Trafico saliente

| Prioridad | Nombre de la regla          | Descripción                                                  |
| :-------- | :-------------------------- | :----------------------------------------------------------- |
| 65000     | PermitirVnetSaliente        | Permitir la salida desde cualquier VM a cualquier VM dentro de la red virtual |
| 65001     | PermitirInternetSaliente    | Permitir que el tráfico saliente vaya a Internet desde cualquier VM |
| 65500     | Negar todo fuera de límites | Denegar el tráfico desde cualquier VM interna a un sistema fuera de la red virtual |

### Reglas de seguridad

Estas son las propiedades cuando nos disponemos a crear una regla de seguirdad

| Propiedad        | Explicación                                                  |
| :--------------- | :----------------------------------------------------------- |
| Nombre           | Un nombre único dentro del grupo de seguridad de red.        |
| Prioridad        | Un número entre 100 y 4096                                   |
| Origen y destino | Cualquiera o una dirección IP individual, un bloque de enrutamiento entre dominios sin clases (CIDR) (10.0.0.0/24, por ejemplo), una etiqueta de servicio o un grupo de seguridad de aplicaciones |
| Protocolo        | TCP, UDP o cualquiera                                        |
| Dirección        | Si la regla se aplica al tráfico entrante o saliente         |
| Rango de puertos | Un puerto individual o rango de puertos                      |
| Acción           | Permitir o denegar el tráfico                                |

### Reglas de seguridad aumentadas

Estas reglas son especialmente útiles cuando se necesita implementar conjuntos de reglas de red más complejos. Con las reglas aumentadas, es posible agregar varias opciones en una sola regla de seguridad, como múltiples direcciones IP, múltiples puertos, etiquetas de servicio y grupos de seguridad de aplicaciones.

### Etiquetas de Servicio

Las etiquetas de servicio en Azure simplifican la seguridad de las máquinas virtuales y las redes virtuales al permitir restringir el acceso por recursos o servicios específicos. Estas etiquetas representan grupos de direcciones IP y facilitan la configuración de reglas de seguridad, ya que para los recursos especificados mediante una etiqueta no es necesario conocer la dirección IP ni los detalles del puerto.

Con las etiquetas de servicio, se puede restringir el acceso a una variedad de servicios. Es importante destacar que Microsoft administra estas etiquetas, por lo que no es posible crear etiquetas personalizadas. Algunos ejemplos de estas etiquetas incluyen:

- **VirtualNetwork**: representa todas las direcciones de red virtual en Azure, así como en redes locales si se utiliza conectividad híbrida.
- **AzureLoadBalancer**: indica el equilibrador de carga de infraestructura de Azure, traduciéndose en la dirección IP virtual del host (168.63.129.16) donde se originan las sondas de estado de Azure.
- **Internet**: representa cualquier recurso fuera de la dirección de red virtual que pueda ser accedido públicamente, incluyendo recursos con direcciones IP públicas, como la característica de aplicaciones web de Azure App Service.
- **AzureTrafficManager**: representa la dirección IP de Azure Traffic Manager.
- **Almacenamiento**: representa el espacio de direcciones IP para Azure Storage, permitiendo especificar si el tráfico está permitido o denegado, y si se permite el acceso solo a una región específica, aunque no es posible seleccionar cuentas de almacenamiento individuales.
- **SQL**: representa las direcciones de los servicios Azure SQL Database, Azure Database para MySQL, Azure Database para PostgreSQL y Azure Synapse Analytics, permitiendo especificar si el tráfico está permitido o denegado y limitarlo a una región específica.
- **AppService**: representa los prefijos de direcciones para Azure App Service.

### ASG o Grupos de seguridad de Aplicaciones

Los grupos de seguridad de aplicaciones facilitan la configuración de la seguridad de red para los recursos utilizados por aplicaciones específicas. Esto permite agrupar máquinas virtuales de manera lógica, independientemente de su dirección IP o asignación de subred.

Dentro de un grupo de seguridad de red, puede emplear grupos de seguridad de aplicaciones para aplicar reglas de seguridad a conjuntos de recursos. Esta práctica simplifica la implementación y escalabilidad de cargas de trabajo de aplicaciones específicas. Simplemente agregue una nueva implementación de máquina virtual a uno o más grupos de seguridad de aplicaciones, y esa máquina virtual seleccionará automáticamente las reglas de seguridad adecuadas para esa carga de trabajo.

Los grupos de seguridad de aplicaciones permiten agrupar interfaces de red, lo que posibilita utilizar este grupo como regla de origen o destino dentro de un grupo de seguridad de red.

<img align="center" src="/img/2ºimagenn.PNG" style="zoom:50%;" />

Sin grupos de seguridad de aplicaciones, necesitaría crear una regla separada para cada VM o agregar un grupo de seguridad de red a una subred y luego agregar todas las VM a esa subred.

## Ejercicio: crear y administrar grupos de seguridad de red

### Crear red virtual y un NSG

*Conviene abrir el Azure CLI desde https://portal.azure.com/#home*

Vamos a asociar el grupo de recursos de nuestro sandbox a la variable rg:

Hemos de introducir este comando en el Azure CLI:

```cli
rg="learn-568c04b5-8b6b-4a2e-ab38-f685e0f6bf96"
```

Después de esto vamos a crear la red virtual de servidores `ERP` y la subred de `Applications`:

```
az network vnet create \
--resource-group $rg \
--name ERP-servers \
--address-prefixes 10.0.0.0/16 \
--subnet-name Applications \
--subnet-prefixes 10.0.0.0/24
```

Lo que nos devuelve algo tal que asi:

<img align="center" src="/img/3ºimagenn.PNG" style="zoom:50%;" />

Ahora vamos a crear la subred de `Databases` introduciremos el siguiente comando:

```
az network vnet subnet create \
--resource-group $rg \
--vnet-name ERP-servers \
--address-prefixes 10.0.1.0/24 \
--name Databases
```

Tras esto crearemos el NSG llamado `ERP-SERVERS-NSG` con el siguiente comando:

```
az network nsg create \
--resource-group $rg \
--name ERP-SERVERS-NSG
```

Como vemos se aplican las reglas de seguridad predeterminadas:

<img align="center" src="/img/5ºimagenn.PNG" style="zoom:50%;" />





### Crear maquinas virtuales con Ubuntu

Vamos a crear la maquina virtual ` AppServer` de ubuntu desde Azure CLI, como se puede ver en el comando hemos de asociarle la red y la subred, el NSG la imagen que sera Ubuntu2204, el tamaño, si queremos SSH keys o no, y un usuario y contraseña (Astrom@ngo222) de administrador, ademas vamos a introducirle un archivo yml custom que obtendremos de git con un comando wget:

```
wget -N https://raw.githubusercontent.com/MicrosoftDocs/mslearn-secure-and-isolate-with-nsg-and-service-endpoints/master/cloud-init.yml && 
az vm create \
--resource-group $rg \
--name AppServer \
--vnet-name ERP-servers \
--subnet Applications \
--nsg ERP-SERVERS-NSG \
--image Ubuntu2204 \
--size Standard_DS1_v2 \
--generate-ssh-keys \
--admin-username azureuser \
--custom-data cloud-init.yml \
--no-wait \
--admin-password Astrom@ngo222
```

Al introducir `--no-wait` no se mostrara el proceso de creación en el cli pero podemos ir a comprobarlo desde el portal

<img align="center" src="/img/6ºimagenn.PNG" style="zoom:50%;" />



Vamos a crear la otra maquina virtual con las mismas caracteristicas exceptuando la subred

```
az vm create \
--resource-group $rg \
--name DataServer \
--vnet-name ERP-servers \
--subnet Databases \
--nsg ERP-SERVERS-NSG \
--size Standard_DS1_v2 \
--image Ubuntu2204 \
--generate-ssh-keys \
--admin-username azureuser \
--custom-data cloud-init.yml \
--no-wait \
--admin-password Astrom@ngo222
```

Podemos comprobar la creación de las vms desde el CLI con el siguiente comando

```
az vm list \
--resource-group $rg \
--show-details --query "[*].{Name:name, Provisioned:provisioningState, Power:powerState}" \
--output table
```

<img align="center" src="/img/7ºimagenn.PNG" style="zoom:50%;" />



### Verificación conectividad

Para conectarnos a las VMs, usaremos SSH directamente desde Cloud Shell. Para hacer esto, necesitaremos las direcciones IP públicas que se han asignado a sus máquinas virtuales. Ejecutaremos el siguiente comando:

```
az vm list \
--resource-group $rg \
--show-details \
--query "[*].{Name:name, PrivateIP:privateIps, PublicIP:publicIps}" --output table
```

Que nos devuelve la información de las IPs publicas y privadas

<img align="center" src="/img/8ºimagenn.PNG" style="zoom:50%;" />

Para realizar la conexión entre las VMs de manera mas sencilla, asignaremos 2 variables, cada una con su ip correspondiente con el siguiente comando:

```
APPSERVERIP="$(az vm list-ip-addresses \
                 --resource-group $rg \
                 --name AppServer \
                 --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
                 --output tsv)"

DATASERVERIP="$(az vm list-ip-addresses \
                 --resource-group $rg \
                 --name DataServer \
                 --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
                 --output tsv)"
```

Si intentamos conectarnos por SSH obtendremos un timeout con ambas VM's:

```
ssh azureuser@$APPSERVERIP -o ConnectTimeout=5
```

<img align="center" src="/img/9ºimagenn.PNG" style="zoom:50%;" />

**NOTA IMPORTANTE:** *Recordemos que las reglas predeterminadas niegan todo el tráfico entrante a una red virtual, a menos que este tráfico provenga de la misma red virtual.*

### Crear regla seguridad para acceso SSH

Como hemos comprobado hemos de crear una regla para poder tener acceso mediante SSH:

Introduciremos el siguiente comando para habilitar el acceso SSH mediante el puerto 22 con el Protoclo TCP, la regla sera de trafico entrante y de prioridad 100

```
az network nsg rule create \
--resource-group $rg \
--nsg-name ERP-SERVERS-NSG 
--name AllowSSHRule \
--direction Inbound \
--priority 100 \
--source-address-prefixes '*' \
--source-port-ranges '*' \
--destination-address-prefixes '*' \
--destination-port-ranges 22 \
--access Allow \
--protocol Tcp \
--description "Allow inbound SSH"
```

Que responde con algo así:

<img align="center" src="/img/10ºimagenn.PNG" />

***Nota importante: Hemos de activar el ssh en el firewall de cada VM desde el serial console con un `sudo ufw allow ssh`, si no la conexion ssh seguira fallando, si las credenciales introducidas no son correctas hemos de resetear el pass***

<img align="center" src="/img/12ºimagenn.PNG" />





<img align="center" src="/img/13ºimagenn.PNG" />

Volvemos a intentar la conexión con el mismo comando que antes `ssh azureuser@$APPSERVERIP -o ConnectTimeout=5` y ahora si se nos pedirá la contraseña:

<img align="center" src="/img/11ºimagenn.PNG" />

Comprobamos la conexion a la otra maquina virtual:

<img align="center" src="/img/14ºimagenn.PNG" />

### Crear regla de seguridad para impedir acceso a la web

Vamos a añadir otra regla para impedir el acceso a la web,  introducimos el siguiente comando:

```
az network nsg rule create \
--resource-group $rg \
--nsg-name ERP-SERVERS-NSG \
--name httpRule \
--direction Inbound \
--priority 150 \
--source-address-prefixes 10.0.1.4 \
--source-port-ranges '*' \
--destination-address-prefixes 10.0.0.4 \
--destination-port-ranges 80 \
--access Deny \
--protocol Tcp \
--description "Deny from DataServer to AppServer on port 80"
```

Que nos devuelve lo siguiente

<img align="center" src="/img/15ºimagenn.PNG" />





### Probar la conectividad HTTP entre máquinas virtuales

Comprobamos ahora que haya conectividad http entre maquinas virtuales, `AppServer` debería poder comunicarse con `DataServer` a través de HTTP. `DataServer` no debería poder comunicarse con `AppServer` a través de HTTP.

Ejecutamos el comando, e introducimos las credenciales:

```
ssh -t azureuser@$APPSERVERIP 'wget http://10.0.1.4; exit; bash'
```

<img align="center" src="/img/16ºimagenn.PNG" />

Y efectivamente si comprobamos la conexion de `DataServer`a `AppServer` :

<img align="center" src="/img/17ºimagenn.PNG" />

###  Implementar un grupo de seguridad de aplicaciones

- A continuación, cree un grupo de seguridad de aplicaciones para servidores de bases de datos para que a todos los servidores de este grupo se les pueda asignar la misma configuración. Está planeando implementar más servidores de bases de datos y desea evitar que estos servidores accedan a servidores de aplicaciones a través de HTTP. Al asignar fuentes en el grupo de seguridad de la aplicación, no es necesario mantener manualmente una lista de direcciones IP en el grupo de seguridad de la red. En su lugar, asigna las interfaces de red de las máquinas virtuales que desea administrar al grupo de seguridad de la aplicación. 

Para crear un nuevo grupo de seguridad de aplicaciones llamado **ERP-DB-SERVERS-ASG** , ejecutaremos el siguiente comando:

```
az network asg create \
--resource-group $rg \
--name ERP-DB-SERVERS-ASG
```

Lo que nos devuelve

<img align="center" src="/img/18ºimagenn.PNG" />

Asociamos `DataServer` con el grupo de seguridad de la aplicación de la siguiente manera:

```
az network nic ip-config update \
--resource-group $rg \
--application-security-groups ERP-DB-SERVERS-ASG \
--name ipconfigDataServer \
--nic-name DataServerVMNic \
--vnet-name ERP-servers \
--subnet Databases
```

Lo que nos devuelve algo asi

<img align="center" src="/img/19ºimagenn.PNG" />

Para actualizar la regla HTTP en el grupo de seguridad de red **ERP-SERVERS-NSG** , ejecutaremos el siguiente comando en Cloud Shell:



```
az network nsg rule update \
--resource-group $rg \
--nsg-name ERP-SERVERS-NSG \
--name httpRule \
--direction Inbound \
--priority 150 \
--source-address-prefixes "" \
--source-port-ranges '*' \
--source-asgs ERP-DB-SERVERS-ASG \
--destination-address-prefixes 10.0.0.4 \
--destination-port-ranges 80 \
--access Deny \
--protocol Tcp \
--description "Deny from DataServer to AppServer on port 80 using application security group"
```

<img align="center" src="/img/20ºimagenn.PNG" />

El comportamiento sigue siendo el mismo desde `AppServer`  hasta `DataServer`  si funciona pero a la inversa no

<img align="center" src="/img/21ºimagenn.PNG" />
