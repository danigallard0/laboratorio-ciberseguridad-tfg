# Laboratorio de Ciberseguridad — TFG

![Status](https://img.shields.io/badge/5%2F5%20bloques-Completo-success?style=for-the-badge)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Wazuh](https://img.shields.io/badge/Wazuh-005571?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Gophish](https://img.shields.io/badge/Gophish-4285F4?style=for-the-badge)

Laboratorio de ciberseguridad diseñado, construido y operado de forma íntegra por mí, como Trabajo de Fin de Grado en Ciberseguridad. Tres máquinas virtuales propias (una atacante, dos víctimas) en una red aislada, más un SIEM real desplegado en la nube, usadas para recorrer el ciclo completo de la seguridad ofensiva y defensiva: atacar un sistema de verdad, analizar su código antes de desplegarlo, proteger lo que ya está roto, detectar y responder cuando falla, y gobernar todo el proceso con los marcos de referencia reales del sector.

No es una colección de writeups de vulnerabilidades conocidas. El valor de este proyecto está en lo que no sale en un tutorial: los fallos reales de cada herramienta, las decisiones tomadas con criterio propio en vez de seguir una checklist, y la honestidad de documentar lo que queda pendiente en vez de maquillarlo.

---

## Los 5 bloques

| Bloque | Contenido | Enlace |
|---|---|---|
| **1. Ofensivo** | Reconocimiento, OpenVAS, 4 vectores de explotación reales, post-explotación, informe de pentest profesional | [`bloque-ofensivo/`](bloque-ofensivo/) |
| **2. Desarrollo Seguro** | SAST (Semgrep), DAST (OWASP ZAP), CVSS manual, modelado de amenazas STRIDE, pipeline DevSecOps real en GitHub Actions, 3 vulnerabilidades corregidas y verificadas | [`bloque-devseguro/`](bloque-devseguro/) |
| **3. Protección** | IAM y MFA real por TOTP, hardening y firewall deny-by-default, gestión formal de vulnerabilidades, campaña de phishing real con Gophish | [`bloque-proteccion/`](bloque-proteccion/) |
| **4. SOC / Respuesta a Incidentes** | SIEM Wazuh en la nube, caso de uso SOC, Threat Hunting, playbook operativo, simulacro de incidente con exfiltración, forense con Autopsy, PCN probado de verdad | [`bloque-soc-ir/`](bloque-soc-ir/) |
| **5. Gobierno** | Política de Seguridad (ISO 27001), evaluación de riesgos (MAGERIT v3), Plan Director de Seguridad a 12 meses (NIST CSF) | [`bloque-gobierno/`](bloque-gobierno/) |

---

## Arquitectura

Tres máquinas virtuales sobre UTM (Apple Silicon), en una red interna Host-Only sin conectividad externa salvo cuando una fase lo requiere explícitamente:

| Máquina | Rol |
|---|---|
| Kali Linux | Atacante |
| Metasploitable2 | Víctima — sistema operativo deliberadamente desactualizado |
| DVWA (Ubuntu + Apache + MariaDB) | Víctima — aplicación web vulnerable |

A partir del Bloque SOC/IR se añade una cuarta pieza fuera del laboratorio local: una instancia de **Wazuh en Oracle Cloud**, que recibe logs de las dos máquinas víctima para poder tener un SIEM real sin necesitar más RAM de la que da el propio Mac.

---

## Cómo leer este repositorio

Cada bloque tiene su propio README con metodología, hallazgos reales (incluidos los que no salieron a la primera), evidencias seleccionadas y, donde aplica, el documento formal completo (informe de pentest, playbook de respuesta, política de seguridad...). Los 5 bloques están pensados para leerse en orden — cada uno se apoya en hallazgos del anterior — pero cada README funciona también de forma independiente.

---

*Todos los sistemas, datos y evidencias de este proyecto pertenecen a un laboratorio propio, aislado y sin conexión a redes de producción ni de terceros. Ninguna prueba se realizó contra sistemas ajenos.*
