# Política de Seguridad de la Información
## Laboratorio de Ciberseguridad — TFG

**Código:** POL-SEG-001
**Autor:** Daniel Gallardo Fernández
**Versión:** 1.0 — 19 de octubre de 2026
**Marco de referencia:** ISO/IEC 27001:2022 (aplicación resumida, adaptada a la escala de un laboratorio de TFG — no se persigue certificación)

---

## 1. Objeto y alcance

Esta política establece los principios y controles de seguridad de la información que rigen el laboratorio de ciberseguridad construido para este TFG. Se aplica a todos los activos descritos en la evaluación de riesgos (apartado 6.2 del informe principal): los sistemas (DVWA-VICTIM, Metasploitable2, Kali-Attacker, Wazuh en Oracle Cloud), los datos que manejan, y los procesos de trabajo seguidos a lo largo de las fases del proyecto.

No es una política corporativa genérica traducida al contexto del laboratorio: cada apartado remite, cuando aplica, a un control que **ya se implementó y verificó realmente** en algún bloque anterior del TFG, no a una intención.

## 2. Roles y responsabilidades

| Rol | Responsable | Funciones |
|---|---|---|
| Responsable de Seguridad (CISO del laboratorio) | Daniel Gallardo Fernández | Define esta política, autoriza excepciones, revisa el cumplimiento |
| Administrador de sistemas | Daniel Gallardo Fernández | Aplica los controles técnicos (hardening, parcheo, backups) |
| Analista SOC | Daniel Gallardo Fernández | Monitorización (Wazuh), respuesta a incidentes, Threat Hunting |

En un entorno real estos roles estarían separados entre varias personas; en este laboratorio los asume la misma persona, lo cual se documenta explícitamente como limitación de alcance.

## 3. Clasificación de la información

| Nivel | Ejemplos en este laboratorio | Tratamiento |
|---|---|---|
| Confidencial | Claves privadas SSH, credenciales de servicios (Wazuh, BD), hashes de contraseñas | Nunca en texto plano fuera del sistema que las necesita; excluidas del ZIP de entrega del TFG |
| Interno | Logs de auditoría (`auth.log`, alertas de Wazuh), configuraciones de los sistemas | Accesible solo al responsable del laboratorio |
| Público | Informe del TFG, diagramas de red, metodología | Entregable final |

## 4. Control de acceso e identidad

Implementado y verificado en el apartado 4.1 del informe principal:

- Política de contraseñas con longitud mínima (12 caracteres), complejidad (mayúscula/minúscula/dígito/símbolo) y rotación (máximo 90 días) — aplicada vía `libpam-pwquality` y `login.defs`.
- Autenticación multifactor (TOTP) obligatoria para el acceso SSH administrativo.
- Principio de mínimo privilegio: cuentas con roles diferenciados y permisos `sudo` acotados a comandos concretos, no acceso administrativo completo por defecto.

## 5. Seguridad de las comunicaciones y segmentación de red

El laboratorio se segmenta en una red interna aislada sin salida a Internet (apartado 1.2), con conmutación puntual y documentada a una red con salida a Internet solo cuando es estrictamente necesario (instalación de paquetes, comunicación con el SIEM en la nube). El tráfico hacia el SIEM en Oracle Cloud se limita a los puertos estrictamente necesarios (514/1514/1515/443), con reglas de firewall deny-by-default tanto en el laboratorio local (apartado 4.2) como en la instancia cloud.

## 6. Seguridad en el ciclo de vida del desarrollo

Cubierto en el Bloque Desarrollo Seguro del informe (apartado 3): análisis estático y dinámico de código antes de desplegar cualquier aplicación, modelado de amenazas, y un pipeline de integración continua con etapas de seguridad obligatorias (SAST, SCA, detección de secretos, escaneo de imagen Docker, DAST). Ninguna corrección de vulnerabilidad se da por buena sin repetir el ataque original y comprobar que ahora falla.

## 7. Gestión de vulnerabilidades

Proceso formal de inventario → escaneo → priorización por CVSS → decisión de remediación (corregir / aceptar el riesgo / no remediar con justificación), descrito en el apartado 4.3. Ninguna vulnerabilidad queda sin una decisión explícita y documentada.

## 8. Gestión de incidentes de seguridad

Cubierta en el Bloque SOC/IR (apartado 5): SIEM desplegado con dos fuentes de log, un caso de uso de detección documentado, un playbook de respuesta formal (con criterios de severidad, escalado y condiciones de cierre), y un simulacro de incidente real ejecutado de principio a fin con las 6 fases del PRI. Esta política exige que cualquier incidente futuro en el laboratorio siga ese mismo playbook, no un proceso ad-hoc.

## 9. Continuidad de negocio

Descrita en el apartado 5.7: activos críticos identificados, RTO/RPO definidos, y una prueba de recuperación real ya ejecutada (no solo teórica) para el servicio de mayor criticidad.

## 10. Cumplimiento normativo

Este laboratorio no maneja datos de carácter personal de terceros ni presta un servicio real, por lo que no aplican directamente RGPD ni otras normativas sectoriales. Se documenta igualmente el principio de **minimización**: ninguna actividad del laboratorio se ha dirigido nunca contra sistemas o personas reales ajenas al propio entorno controlado (ver nota de ética en el apartado 4.4, campaña de phishing dirigida únicamente a la cuenta propia del autor).

## 11. Revisión y actualización

Esta política se revisa cada vez que se cierra un bloque del TFG con hallazgos relevantes (p. ej., los descubrimientos reales documentados en el Bloque SOC/IR sobre brechas de visibilidad del SIEM). La versión 1.0 incorpora el estado del laboratorio a fecha de cierre de los Bloques I a V. Las mejoras identificadas pero no implementadas quedan recogidas en el PDS (apartado 6.3).
