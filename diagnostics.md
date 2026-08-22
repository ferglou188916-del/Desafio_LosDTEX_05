# Verificación y Diagnóstico de Red

**Autor:** Fernando José López Domínguez LD230289

## 1. El Problema: Ausencia de un Procedimiento Estandarizado

En la infraestructura actual de **InnovaCloud Solutions**, la carencia de un procedimiento estandarizado para auditar los servicios activos y las interfaces de red representa un riesgo crítico para la continuidad del negocio y la seguridad de la información. Los principales inconvenientes de esta falta de estandarización incluyen:

* **Tiempos de inactividad prolongados (Downtime):** Al no contar con un protocolo de diagnóstico claro, la detección y resolución de problemas de conectividad depende de la intuición del administrador en turno, lo que retrasa la reactivación de los servicios.
* **Vulnerabilidades de Seguridad:** La falta de auditorías sobre los puertos abiertos significa que servicios innecesarios o mal configurados podrían estar expuestos, creando vectores de ataque explotables desde el exterior o desde la red interna.
* **Inconsistencia en el Entorno de Desarrollo:** Sin una validación estandarizada, las máquinas virtuales pueden quedar aisladas o con configuraciones erróneas (como IP duplicadas o interfaces caídas), frenando el flujo de trabajo colaborativo.

---

## 2. Propuesta de Solución: Estandarización del Diagnóstico en Linux

Para establecer un control riguroso y proactivo, nuestra firma propone el siguiente flujo de verificación y diagnóstico basado en utilidades de consola nativas y modernas en entornos Linux.

### A. Validación de Interfaces Activas
El primer paso es confirmar que las tarjetas de red están correctamente levantadas y tienen asignadas las direcciones IP esperadas (especialmente tras aplicar configuraciones con Netplan).

```bash
# Muestra el estado de todas las interfaces de red y sus direcciones IP asignadas
ip a

# Muestra únicamente el estado del enlace de capa 2 (UP / DOWN) para detectar cables desconectados o interfaces apagadas
ip link show
```

### B. Verificación de Conectividad de Extremo a Extremo
Una vez validadas las interfaces, se debe confirmar que los paquetes alcanzan su destino, ya sea dentro de la red interna o hacia el exterior.

```bash
# Verifica conectividad básica de capa 3 con el gateway o un servidor específico enviando 4 paquetes de prueba
ping -c 4 192.168.1.254

# Traza la ruta completa de los paquetes para identificar en qué salto (router) se pierde la conexión
traceroute 8.8.8.8
```
*(Nota: En entornos modernos se recomienda instalar y usar `mtr` para combinar la funcionalidad de ping y traceroute en tiempo real).*

### C. Auditoría de Puertos Abiertos
Es imperativo conocer qué puertos están a la escucha para cerrar aquellos que no correspondan a servicios oficiales de InnovaCloud Solutions. Utilizaremos `ss` (Socket Statistics), el cual es más eficiente que el antiguo `netstat`.

```bash
# Lista todos los puertos TCP (-t) y UDP (-u) que están a la escucha (-l), mostrando los números de puerto en lugar de nombres de servicio (-n)
ss -tuln
```

### D. Auditoría de Servicios en Ejecución
Finalmente, se debe verificar que los demonios y servicios críticos del sistema (como SSH, bases de datos o servidores web) estén operando correctamente.

```bash
# Verifica el estado actual, el tiempo en línea (uptime) y los registros recientes de un servicio específico (ej. servidor SSH)
systemctl status sshd

# Lista de forma rápida todos los servicios que han fallado durante el arranque o en su ejecución actual
systemctl --failed
```
