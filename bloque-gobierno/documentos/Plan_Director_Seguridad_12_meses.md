# Plan Director de Seguridad (PDS) — 12 meses
## Laboratorio de Ciberseguridad — TFG

**Código:** PDS-001
**Autor:** Daniel Gallardo Fernández
**Versión:** 1.0 — 19 de octubre de 2026
**Marco de referencia:** NIST Cybersecurity Framework (funciones Identificar / Proteger / Detectar / Responder / Recuperar)

---

## 1. Origen de este plan

A diferencia de un PDS genérico, cada acción de este plan nace de un hallazgo real documentado en algún bloque anterior del TFG — no de una lista de buenas prácticas sin contrastar. La columna "Origen" de cada tabla remite al apartado del informe donde se detectó el hueco.

## 2. Hoja de ruta por función NIST CSF

### IDENTIFICAR

| Acción | Prioridad | Plazo | Origen |
|---|---|---|---|
| Revisar y actualizar la evaluación de riesgos MAGERIT tras cada cambio relevante de infraestructura | Media | Trimestral (continuo) | Apartado 6.2 — práctica estándar de gestión de riesgos |
| Documentar un inventario formal de activos más allá del laboratorio (si el entorno crece) | Baja | Mes 6 | Apartado 6.2 |

### PROTEGER

| Acción | Prioridad | Plazo | Origen |
|---|---|---|---|
| Rotar las credenciales del dashboard/indexer/API de Wazuh desde las generadas automáticamente por el instalador | Alta | Mes 1 | Apartado 5.1 (pendiente ya anotado) |
| Mover las copias de seguridad de DVWA a almacenamiento externo al propio servidor (no en `/tmp` del mismo host) | Alta | Mes 1 | Apartado 5.7 (limitación documentada en la prueba de recuperación) |
| Evaluar una solución de IP dinámica (DNS dinámico o VPN) para que el `allowed-ips` del SIEM no dependa de la IP pública del equipo del analista | Media | Mes 3 | Apartado 5.1 (incidencia real, IP del Mac rompiendo la conectividad) |
| Eliminar o proteger `setup.php` de DVWA tras la instalación inicial (endpoint de creación/reseteo de BD sin ninguna restricción de acceso) | Media | Mes 2 | Apartado 6.3 (SQLi crítico identificado por ZAP, pendiente — ver también 3.4) |

### DETECTAR

| Acción | Prioridad | Plazo | Origen |
|---|---|---|---|
| Implementar una regla de correlación propia en `local_rules.xml` para fuerza bruta SSH (agrupar fallos repetidos y escalar severidad automáticamente) | Alta | Mes 2 | Apartados 5.3/5.4 (mejora pendiente ya documentada en el playbook) |
| Investigar y resolver la causa raíz de por qué los logins exitosos vía syslog remoto (sin agente) no generan alerta pese a decodificarse correctamente | Media | Mes 4 | Apartado 5.5 (hallazgo de detección documentado) |
| Extender la cobertura de logs de aplicación/firewall con decoders propios a cualquier host nuevo que se incorpore al laboratorio (ya resuelto para DVWA-VICTIM/UFW) | Baja | Según se incorporen hosts | Apartado 5.1 |

### RESPONDER

| Acción | Prioridad | Plazo | Origen |
|---|---|---|---|
| Configurar `active-response` de Wazuh para automatizar la contención en hosts con agente registrado | Media | Mes 5 | Apartado 5.4 (playbook, mejora pendiente) |
| Formalizar más plantillas de ticket de incidente (una por cada caso de uso de detección nuevo que se añada) | Baja | Continuo | Apartado 5.4 |

### RECUPERAR

| Acción | Prioridad | Plazo | Origen |
|---|---|---|---|
| Automatizar las copias de seguridad de DVWA (actualmente manuales) con una tarea programada | Media | Mes 3 | Apartado 5.7 |
| Repetir la prueba de recuperación con el escenario de pérdida del propio SIEM (Wazuh), no probado en 5.7 por el riesgo de romper la única instancia disponible | Baja | Mes 8 | Apartado 5.7 (RTO/RPO definidos pero no probados para este servicio) |

## 3. Priorización resumida (primeros 90 días)

Las **tres** acciones de prioridad Alta concentran el esfuerzo del primer trimestre, y cada una cierra una brecha concreta ya identificada, no una genérica: rotar las credenciales de Wazuh cierra la única mitigación pendiente del activo que se queda en riesgo residual **Medio** (SIEM Wazuh) en la evaluación MAGERIT del apartado 6.2; mover los backups fuera del propio servidor cierra, de la misma forma, el riesgo residual **Medio** del activo Backups en ese mismo apartado; y la regla de correlación de fuerza bruta cierra la mejora pendiente ya documentada en el playbook (apartado 5.4), no ligada a un activo concreto de MAGERIT sino a la capacidad de detección del SOC en general.
