# **Servidor .onion en tu Router con OpenWRT**

**Powered by M-Society Research Lab**

---

## **Visión General**

Este proyecto transforma un router doméstico común en un **servidor .onion completamente funcional**, capaz de alojar páginas web HTML/PHP, manejar formularios, servir contenido estático y dinámico, e incluso almacenar bases de datos internas, todo bajo un túnel cifrado provisto por la red Tor.

A diferencia de un VPS que cuesta dinero y puede ser dado de baja en cualquier momento, un router:

* Está **encendido 24/7**.
* Consume **menos que un foco LED**.
* No depende de terceros.
* No se puede censurar.
* Permite alojar un servicio Onion de forma gratuita, silenciosa y privada.

Este repositorio documenta el proceso completo para convertir routers compatibles en **infraestructura descentralizada de grado militar**, lista para producir una dirección `.onion` de 56 caracteres inaccesible para rastreadores, gobiernos y proveedores.

---

## **Características Principales**

* Configuración completa de **OpenWRT** sobre routers soportados.
* Instalación y configuración de:

  * Servidor web **uHTTPd** (ligero y eficiente).
  * **Nginx** (opcional).
  * Servicio Tor como **Hidden Service**.
* Manejo seguro de archivos sensibles (hardening).
* Montaje de almacenamiento **USB externo** para ampliar capacidad.
* Generación y personalización de direcciones Onion (incluye **vanity addresses**).
* Estructura de directorios para hosting eficiente.
* Seguridad extrema: permisos restrictivos, aislamiento del localhost y fortificación de configuraciones.
* Compatible con routers desde **$15–$30 USD** (Netgear R6700, XR500, TP-Link Archer C7, ASUS RT-AC68U y muchos más).

---

## **Créditos**

Este proyecto fue documentado, redactado y optimizado por **M-Society**, en colaboración con desarrolladores independientes y comunidades de privacidad alrededor del mundo.
M-Society agradece profundamente a la comunidad de OpenWRT, Tor Project y a los investigadores que impulsan tecnologías libres, no censurables y soberanas.

---

# **1. Requisitos Previos**

### **Hardware Compatible**

Se debe verificar primero que tu router soporte OpenWRT:

1. Entrar a:
   `https://openwrt.org/toh/start`
2. Buscar tu modelo.
3. Confirmar:

   * Target
   * Última versión disponible
   * Imágenes Factory/Sysupgrade

**Recomendación:**
Para la primera instalación, siempre usar **imagen Factory**.

---

### **Software Necesario**

* OpenWRT (última versión compatible).
* Cliente SSH.
* Linux nativo, WSL o cualquier entorno con terminal.
* Memoria USB (opcional pero recomendado).
* Paquetes instalables vía `opkg`.

---

# **2. Instalación de OpenWRT**

1. Descargar la imagen Factory correcta para tu router.
2. Entrar al panel web original del router.
3. Subir la imagen desde la sección de actualización.
4. Esperar reinicio automático.

Una vez en OpenWRT:

```
ssh root@192.168.1.1
```

---

# **3. Verificación de Espacio y Preparación de USB**

### Comprobar espacio interno:

```
df -h
```

Si tu router tiene solo ~128–256MB flash, una memoria USB es ideal.

**Formatear USB (XFS recomendado):**

```
mkfs.ext4 /dev/sda
mkdir -p /mnt/usb
mount /dev/sda /mnt/usb
```

Crear estructura base:

```
mkdir /mnt/usb/www
mkdir /mnt/usb/logs
mkdir /mnt/usb/tor_hidden
mkdir /mnt/usb/tor_data
```

---

# **4. Instalación de Dependencias**

### Actualizar paquetes:

```
opkg update
```

(No usar `opkg upgrade`; puede romper OpenWRT.)

### Instalar dependencias críticas:

```
opkg install block-mount kmod-usb-storage kmod-fs-ext4
opkg install uhttpd uhttpd-mod-ubus
opkg install nginx     # opcional
opkg install tor
```

---

# **5. Configuración del Servidor Web**

El archivo principal:

```
/etc/config/uhttpd
```

Configurar para servidor onion:

* Escuchar en: `127.0.0.1`
* Puerto: `8089`
* Root: `/mnt/usb/www`

Ejemplo:

```
config uhttpd 'onion'
    option listen_http '127.0.0.1:8089'
    option home '/mnt/usb/www'
```

---

# **6. Configuración de Tor Hidden Service**

Editar:

```
/etc/tor/torrc
```

Agregar:

```
HiddenServiceDir /mnt/usb/tor_hidden/
HiddenServicePort 80 127.0.0.1:8089
Log notice file /mnt/usb/logs/tor.log
User tor
```

Reiniciar:

```
/etc/init.d/tor restart
```

Ver tu dirección onion:

```
cat /mnt/usb/tor_hidden/hostname
```

---

# **7. Hardening (Seguridad Extrema)**

### Archivos críticos deben ser accesibles solo por Tor:

```
chmod 700 /mnt/usb/tor_hidden
chown -R tor:tor /mnt/usb/tor_hidden
chmod 600 /mnt/usb/logs/tor.log
```

### Verificar que solo escuche en localhost:

```
netstat -tulnp | grep 8089
```

Debe aparecer:

```
127.0.0.1:8089
```

---

# **8. Personalizar Direcciones Onion (Vanity)**

Opcional pero espectacular.

Instalar generador de direcciones:

```
git clone https://github.com/cathugger/mk220lor
cd mk220lor
make
```

Ejemplo:

```
./mk220lor mso
```

Esto intenta generar direcciones como:

```
msoxxxxxxx.onion
```

(Advertencia: generar patrones largos puede tardar horas o días.)

---

# **9. Estructura Recomendada para Hosting**

```
/mnt/usb/www/
│── index.html
│── style.css
│── app.php
│── assets/
└── forms/
```

Para PHP:

```
opkg install php8 php8-cgi
```

---

# **10. Solución de Fallas Comunes**

### **1. No aparece dirección onion**

* Ver logs:
  `cat /mnt/usb/logs/tor.log`
* Revisar permisos de carpeta HiddenService.

### **2. El sitio no carga**

* Revisar que uHTTPd esté en local:
  `service uhttpd restart`
* Probar localmente:
  `curl 127.0.0.1:8089`

### **3. Tor no inicia**

Ejecutar:

```
tor --verify-config
```

---

# **11. Advertencias Importantes**

* Este proyecto es **100% legal** mientras alojes contenido legal.
* Evita exponer servicios en 0.0.0.0.
* No uses “upgrade” en OpenWRT.
* Mantén siempre permisos restrictivos.

---

# **12. Contacto**

Para colaboración, auditorías o soporte profesional:

**M-Society Cyber Research Division**
contact: https://discord.gg/9QRngbrMKS

