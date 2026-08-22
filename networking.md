# Solución para la Configuración de Red



**Cliente:** InnovaCloud Solutions

**Firma consultora:** Los DTEX

**Autor:** Rodrigo Alejandro Marinero Rivas — MR240333



## 1. Problema encontrado



Actualmente las máquinas virtuales utilizadas por InnovaCloud Solutions trabajan con la configuración de red NAT de VirtualBox.



NAT es útil cuando solamente necesitamos que una máquina virtual tenga acceso a Internet, ya que utiliza la conexión del equipo anfitrión para comunicarse hacia el exterior.



Sin embargo, para un entorno de desarrollo donde varias máquinas virtuales necesitan comunicarse entre sí y también con otros equipos de la red de la empresa, este modo puede presentar algunas limitaciones.



Una de las principales dificultades es que la máquina virtual queda detrás del equipo anfitrión y no se comporta directamente como otro dispositivo de la red física. Esto puede complicar la comunicación directa con otros servidores, equipos o recursos utilizados por el grupo de desarrollo.



Por esta razón consideramos necesario cambiar la configuración de red utilizada por las máquinas virtuales.



## 2. Comparación de los modos de red de VirtualBox



### NAT



NAT es el modo que VirtualBox utiliza normalmente de forma predeterminada.



La máquina virtual puede conectarse a Internet utilizando la conexión del equipo anfitrión. Es una opción sencilla de configurar y también brinda cierto aislamiento porque la máquina virtual no queda expuesta directamente en la red física.



Su principal desventaja para este caso es que puede dificultar la comunicación directa entre la máquina virtual y otros dispositivos de la red corporativa.



### Adaptador Puente



El modo Puente conecta la máquina virtual directamente a la red física.



En este caso, la máquina virtual se comporta como si fuera otro equipo conectado al mismo switch o router que la computadora anfitriona.



Esto permite que la máquina virtual tenga su propia dirección IP y pueda comunicarse de una manera más directa con otros dispositivos de la red.



Por esta razón es una opción adecuada para servidores y estaciones de trabajo que necesitan tener visibilidad dentro de la red.



### Red Interna



La Red Interna crea una red privada y aislada entre las máquinas virtuales que utilizan el mismo nombre de red interna.



Este modo es útil para laboratorios, pruebas o ambientes donde se necesita que las máquinas virtuales se comuniquen entre ellas sin tener acceso directo a la red física.



Sin embargo, por sí sola no permite comunicación con Internet ni con otros equipos externos, a menos que se configure una máquina adicional como puerta de enlace.



## 3. Modo de red propuesto



Para el entorno de desarrollo de InnovaCloud Solutions proponemos utilizar **Adaptador Puente**.



Consideramos que este modo es el más conveniente porque las máquinas virtuales podrán comportarse como equipos independientes dentro de la red de la empresa.



Esto facilitará que los desarrolladores puedan acceder a servicios instalados en otras máquinas virtuales o servidores y también permitirá realizar pruebas de conectividad de una forma más sencilla.



Con el modo Puente, cada máquina virtual puede tener una dirección IP propia dentro de la misma red que utilizan los demás dispositivos.



## 4. Configuración del Adaptador Puente en VirtualBox



Para realizar la configuración se deben seguir estos pasos:



1\. Apagar la máquina virtual si se encuentra encendida.

2\. Abrir VirtualBox.

3\. Seleccionar la máquina virtual de Ubuntu Server.

4\. Entrar a **Configuración**.

5\. Seleccionar el apartado **Red**.

6\. Habilitar el adaptador de red.

7\. En **Conectado a**, seleccionar **Adaptador Puente**.

8\. Seleccionar la tarjeta de red física que utiliza el equipo anfitrión.

9\. Guardar los cambios.

10\. Iniciar nuevamente Ubuntu Server.



Después de iniciar el servidor debemos identificar el nombre de la interfaz de red.



## 5. Identificar la interfaz de red en Ubuntu Server



Podemos utilizar:



```bash

ip addr

```



También se puede utilizar:



```bash

ip link

```



Estos comandos permiten observar las interfaces disponibles.



Por ejemplo, una interfaz podría llamarse:



```text

enp0s3

```



El nombre puede ser diferente en cada servidor, por lo que primero debe verificarse antes de modificar la configuración.



## 6. Configuración de una dirección IP estática con Netplan



Ubuntu Server utiliza Netplan para administrar la configuración de red mediante archivos YAML.



Primero debemos revisar qué archivos existen dentro del directorio de Netplan:



```bash

ls /etc/netplan

```



Podría aparecer un archivo parecido a:



```text

01-netcfg.yaml

```



El nombre puede cambiar dependiendo de la instalación de Ubuntu Server.



### Abrir el archivo de configuración



Por ejemplo:



```bash

sudo nano /etc/netplan/01-netcfg.yaml

```



Siguiendo el formato trabajado en clase, podemos utilizar una configuración como la siguiente:



```yaml

network:

&#x20; version: 2

&#x20; ethernets:

&#x20;   enp0s3:

&#x20;     addresses:

&#x20;       - 192.168.1.100/24

&#x20;     gateway4: 192.168.1.1

&#x20;     nameservers:

&#x20;       addresses:

&#x20;         - 8.8.8.8

&#x20;         - 8.8.4.4

```



En este ejemplo:



\* `enp0s3` es la interfaz de red.

\* `192.168.1.100/24` es la dirección IP estática asignada al servidor.

\* `192.168.1.1` representa la puerta de enlace.

\* `8.8.8.8` y `8.8.4.4` son los servidores DNS utilizados como ejemplo.



Estos valores deben adaptarse a la red real donde se encuentre instalado el servidor.



## 7. Aplicar la configuración



Después de guardar el archivo YAML se aplican los cambios con:



```bash

sudo netplan apply

```



Luego podemos revisar nuevamente la dirección IP:



```bash

ip addr

```



También podemos verificar la tabla de enrutamiento:



```bash

ip route

```



## 8. Verificación de conectividad



Después de configurar la dirección IP estática es importante comprobar que exista comunicación.



Primero podemos probar la puerta de enlace:



```bash

ping 192.168.1.1

```



Luego podemos verificar comunicación hacia Internet, por ejemplo:



```bash

ping 8.8.8.8

```



Si las pruebas funcionan correctamente, significa que la interfaz tiene conectividad y que existe una ruta válida hacia otras redes.



## 9. ¿Cómo ayuda esta solución a InnovaCloud Solutions?



Al utilizar Adaptador Puente, las máquinas virtuales del entorno de desarrollo podrán tener una dirección IP propia y comunicarse de forma más directa con otros equipos y recursos de la red.



Además, al utilizar una dirección IP estática mediante Netplan, el servidor mantendrá una dirección conocida y predecible.



Esto facilita el acceso a los servicios instalados en la máquina virtual y también permite que los desarrolladores tengan una configuración más ordenada al momento de realizar pruebas.



## 10. Conclusión



Para InnovaCloud Solutions consideramos que el modo Adaptador Puente es más apropiado que NAT para el entorno de desarrollo colaborativo.



NAT continúa siendo útil para máquinas que solamente necesitan salir a Internet y Red Interna es útil para ambientes aislados. Sin embargo, Puente permite una comunicación más directa con la red física y se adapta mejor a la necesidad planteada en este caso.



La configuración de una IP estática mediante Netplan también ayuda a mantener una dirección estable para el servidor y facilita su administración dentro de la red.



