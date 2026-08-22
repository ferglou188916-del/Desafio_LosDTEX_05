# Solución para el Almacenamiento

**Cliente:** InnovaCloud Solutions
**Firma consultora:** Los DTEX
**Autor:** Adal Isaac Cárcamo Sánchez — CS230279

## 1. Problema encontrado

Uno de los principales problemas que presenta InnovaCloud Solutions es que el servidor principal ha tenido fallas en sus discos y actualmente no cuenta con una solución adecuada de redundancia.

Esto es un riesgo importante para la empresa porque, si un disco deja de funcionar y contiene información necesaria para los servicios, se podrían perder datos o provocar que algunos sistemas dejen de estar disponibles.

Al tratarse del servidor principal, consideramos que no es conveniente depender de un solo disco. Una falla de almacenamiento podría afectar el trabajo de los empleados y causar tiempo de inactividad mientras se recupera la información.

Por esta razón, proponemos utilizar un sistema RAID que permita trabajar con varios discos y mantener redundancia de los datos.

## 2. Propuesta: RAID 10

Para el servidor principal de InnovaCloud Solutions proponemos utilizar **RAID 10**.

Elegimos este tipo de RAID porque combina características de RAID 1 y RAID 0. Esto permite mantener copias de la información en diferentes discos y, al mismo tiempo, distribuir los datos para obtener un buen rendimiento.

RAID 10 necesita como mínimo cuatro discos. Debido a que parte del almacenamiento se utiliza para mantener copias de los datos, no se aprovecha toda la capacidad física disponible.

Aun así, consideramos que esta opción es adecuada para InnovaCloud Solutions porque en un servidor principal es más importante tener disponibilidad y redundancia que utilizar el 100 % del espacio de almacenamiento.

### Ventajas de utilizar RAID 10

* Permite mantener redundancia de la información.
* Reduce el riesgo de pérdida inmediata de datos cuando falla un disco.
* Ofrece un buen rendimiento de lectura y escritura.
* Facilita la recuperación del arreglo cuando se reemplaza un disco dañado.
* Es una opción adecuada para servidores que necesitan mantener sus servicios disponibles.

También es importante mencionar que **RAID no debe considerarse un reemplazo de las copias de seguridad**.

RAID ayuda principalmente cuando existen fallas en los discos, pero la empresa debe mantener respaldos adicionales para proteger la información ante otros problemas.

## 3. Administración con mdadm

En Ubuntu Server podemos utilizar la herramienta `mdadm` para crear, revisar y administrar arreglos RAID.

Antes de realizar cualquier configuración debemos identificar los discos disponibles.

```bash
lsblk
```

Este comando permite visualizar los discos y particiones reconocidos por el servidor.

> Los nombres de discos utilizados a continuación son solamente un ejemplo. Antes de realizar una implementación real se deben identificar correctamente los discos del servidor.

### Instalar mdadm

Primero actualizamos la información de los paquetes:

```bash
sudo apt update
```

Luego instalamos la herramienta:

```bash
sudo apt install mdadm
```

### Crear RAID 10

Como ejemplo, vamos a suponer que el servidor cuenta con cuatro discos disponibles:

* `/dev/sdb`
* `/dev/sdc`
* `/dev/sdd`
* `/dev/sde`

El arreglo se podría crear de la siguiente manera:

```bash
sudo mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

En este comando:

* `/dev/md0` es el nombre que tendrá el arreglo RAID.
* `--level=10` indica que se utilizará RAID 10.
* `--raid-devices=4` indica que participarán cuatro discos.

## 4. Verificar el estado del RAID

Después de crear el arreglo podemos revisar su estado utilizando:

```bash
cat /proc/mdstat
```

También podemos obtener información más detallada con:

```bash
sudo mdadm --detail /dev/md0
```

Estos comandos ayudan al administrador a comprobar que los discos se encuentren funcionando correctamente y permiten identificar si el arreglo presenta algún problema.

## 5. Preparar el almacenamiento para utilizarlo

Después de crear correctamente el RAID podemos crear un sistema de archivos EXT4:

```bash
sudo mkfs.ext4 /dev/md0
```

Luego creamos una carpeta que funcionará como punto de montaje:

```bash
sudo mkdir -p /srv/datos
```

Montamos el arreglo:

```bash
sudo mount /dev/md0 /srv/datos
```

Finalmente podemos verificar el almacenamiento disponible con:

```bash
df -h
```

## 6. ¿Cómo resuelve esta propuesta el problema?

Con RAID 10, InnovaCloud Solutions dejaría de depender únicamente de un solo disco para almacenar la información del servidor principal.

Al tener redundancia, una falla física en uno de los discos no necesariamente significaría la pérdida inmediata de todos los datos.

Además, el uso de `mdadm` permitirá al administrador revisar periódicamente el estado del arreglo y detectar problemas en los discos.

De esta manera, la empresa tendría una solución de almacenamiento más confiable y estaría mejor preparada ante fallas de hardware.

## 7. Conclusión

Consideramos que RAID 10 es una buena alternativa para InnovaCloud Solutions porque ofrece un equilibrio entre redundancia y rendimiento.

Aunque necesita varios discos y parte de la capacidad se utiliza para mantener copias de los datos, este costo se justifica por la importancia de proteger la información almacenada en el servidor principal.

También recomendamos mantener copias de seguridad independientes, ya que RAID ayuda ante fallas de discos, pero no sustituye un sistema de respaldo.
