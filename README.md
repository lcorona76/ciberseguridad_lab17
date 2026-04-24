# Ciberseguridad Práctica b17
## Configurando un IDS

Un IDS (Intrusion Detection System o Sistema de Detección de Intrusiones) es una herramienta de seguridad diseñada para monitorear el tráfico de una red o los eventos de un sistema en busca de actividades sospechosas o violaciones de políticas. 
Para entenderlo mejor, piensa en el IDS como una alarma de seguridad: su función principal es "mirar y avisar", pero tradicionalmente no interviene para detener el ataque (esa es la diferencia clave con el IPS, que es lo que tú estás configurando). 

### ¿Cómo funciona?
Un IDS analiza los datos de dos maneras principales: 
Por firmas: Compara el tráfico con una base de datos de "huellas" de ataques conocidos (como un antivirus). Si el tráfico coincide con un patrón de ataque (ej. un ataque SQLi conocido), genera una alerta.
Por anomalías: Establece una "línea de base" de lo que es un comportamiento normal en tu red. Si de pronto hay un pico masivo de tráfico o conexiones extrañas a las 3 AM, el IDS marca esto como sospechoso aunque no conozca el ataque específico.

### Tipos de IDS
  - <b>NIDS (Network IDS):</b> Analiza todo el tráfico que circula por la red. Se coloca en puntos estratégicos (como cerca del router o firewall) para vigilar todos los dispositivos. Snort es el ejemplo más famoso de este tipo.
  - <b>HIDS (Host IDS):</b> Se instala en un solo dispositivo (servidor o PC) y vigila lo que ocurre dentro de ese equipo (cambios en archivos del sistema, registros de log, intentos de inicio de sesión). 

### ¿Por qué usar Snort como IDS/IPS?
  - <b>Visibilidad:</b> Te dice exactamente quién está intentando entrar y qué técnica está usando.
  - <b>Cumplimiento:</b> Muchas normativas de seguridad exigen tener sistemas de detección activos.
  - <b>Automatización:</b> Con PulledPork (que instalamos antes), el IDS se mantiene actualizado contra las amenazas que aparecen cada día.

### Requerimientos
En nuestro caso Lo instalaremos como HIDS para lo cual ocuparemos:
  - Sistema Operativo compatible Linux o Windows (En mi caso Ubuntu 24.04)
  - Interface de red
  - Kali linux (para validar funcionalidad)

### Instalando snort

1. Instala Snort desde los repositorios oficiales (más sencillo para Ubuntu 24.04):
````bash
sudo apt update
sudo apt install snort -y
````
  - Interfaz de red: Durante la instalación, se te puede preguntar qué interfaz debe escuchar. Usa ip a en otra terminal para identificar la tuya (ej. eth0 o enp0s3).
  - Rango de red: Para un HIDS, ingresa la dirección IP específica de tu servidor con máscara /32 (ej. 192.168.1.10/32) para que Snort se enfoque solo en el tráfico que entra o sale de ese host.

<img width="978" height="367" alt="image" src="https://github.com/user-attachments/assets/b6adbe2c-8b49-4431-954e-8dff9f35b1e1" />

2. Configuración como HIDS

Debes editar el archivo principal de configuración para definir el entorno local:

  - Edita el archivo: sudo nano /etc/snort/snort.conf.
  - Busca la variable HOME_NET y asegúrate de que apunte a la IP de tu servidor, ejemplo: ipvar HOME_NET 192.168.1.10/32 #(si dice any hay que cambiarlo)
  - Define EXTERNAL_NET como cualquier cosa que no sea tu host, ejemplo: ipvar EXTERNAL_NET !$HOME_NET

<img width="744" height="317" alt="Captura de pantalla 2026-04-24 155721" src="https://github.com/user-attachments/assets/bf6739ff-eb36-4265-9dc1-218029f84bec" />

3. Creación de una Regla de Prueba
Para validar el funcionamiento, crea una regla simple que detecte pings (ICMP):

  - Edita el archivo de reglas locales: sudo nano /etc/snort/rules/local.rules.
  - Añade la siguiente línea: alert icmp any any -> $HOME_NET any (msg:"ICMP Test Detectado"; sid:1000001; rev:1;)
  - Guarda y asegúrate de que esta línea esté activa en snort.conf buscando include $RULE_PATH/local.rules.

Reglas ejemplo:

#:wq!:Detectar Nmap ICMP Ping (-sn o -sP):
alert icmp any any -> $HOME_NET any (msg:"Nmap ICMP Ping detectado"; itype:8; sid:1000004; rev:1;)

Si no toma el valor de la variable tambien lo puedes definir directamente
#:wq!:Detectar Nmap ICMP Ping (-sn o -sP):
alert icmp any any -> 192.168.8.25/32 any (msg:"Nmap ICMP Ping detectado"; itype:8; sid:1000004; rev:1;)

Asi se deberia ver

<img width="902" height="298" alt="image" src="https://githenp0s8ub.com/user-attachments/assets/e0d4c72e-166b-43be-be05-d37353960f27" />

4. Validación del Funcionamiento
Para confirmar que Snort está detectando tráfico correctamente, sigue estos pasos:

  - Prueba de configuración: Ejecuta el siguiente comando para verificar errores de sintaxis:
````bash
sudo snort -T -c /etc/snort/snort.conf -i <tu_interfaz>
````

<img width="694" height="187" alt="image" src="https://github.com/user-attachments/assets/6b0ee524-83ad-4575-98aa-d07144d7439a" />


<img width="763" height="690" alt="image" src="https://github.com/user-attachments/assets/d0974243-bc58-4fa5-bc22-7dde5248ac0e" />


  Si ves "Snort successfully validated the configuration!", puedes proceder.
  - Prueba en tiempo real: Inicia Snort en modo consola para ver las alertas inmediatamente:

````bash
sudo snort -A console -q -c /etc/snort/snort.conf -i <tu_interfaz>
````
  - Generar la alerta: Desde el kali, realiza un ping a la IP del servidor Ubuntu.
````bash
ping 192.168.8.25
````
<img width="615" height="235" alt="image" src="https://github.com/user-attachments/assets/6ffd9e40-816d-4e3c-a579-0b0ea8d82f3f" />

  - Deberías ver el mensaje "ICMP Test Detectado" aparecer en la terminal de Snort.

<img width="1505" height="187" alt="image" src="https://github.com/user-attachments/assets/f7f7a77a-f605-440c-9cab-34e67883458b" />

Ahora puedes crear las reglas de deteción de acuerdo a tus necesidades.
