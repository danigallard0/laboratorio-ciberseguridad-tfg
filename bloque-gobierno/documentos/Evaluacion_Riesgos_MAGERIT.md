# Evaluación de Riesgos — Metodología MAGERIT v3
## Laboratorio de Ciberseguridad — TFG

**Código:** RIESGO-001
**Autor:** Daniel Gallardo Fernández
**Versión:** 1.0 — 19 de octubre de 2026
**Metodología:** MAGERIT v3 (Metodología de Análisis y Gestión de Riesgos de los Sistemas de Información)

---

## 1. Alcance y método

Se identifican los activos reales del laboratorio, se valoran en las dimensiones MAGERIT (Confidencialidad, Integridad, Disponibilidad, Autenticidad, Trazabilidad), se asocian a amenazas del catálogo MAGERIT, y se calcula el riesgo como `Probabilidad × Impacto` sobre una escala 1-5 (riesgo bajo: 1-6; medio: 7-14; alto: 15-25). Para cada activo se documentan las salvaguardas **ya aplicadas y verificadas** en bloques anteriores del TFG, y el riesgo residual que queda tras ellas — no un riesgo teórico sin contrastar con lo que realmente se hizo.

**Escala de valoración de activos (dimensiones MAGERIT):** 1 (Muy Bajo) a 5 (Muy Alto).

## 2. Identificación y valoración de activos

| # | Activo | Tipo MAGERIT | C | I | D | A | T |
|---|---|---|---|---|---|---|---|
| 1 | DVWA-VICTIM (Apache/PHP) | [S] Servicio | 3 | 4 | 4 | 3 | 3 |
| 2 | Base de datos MariaDB (datos DVWA) | [D] Datos | 4 | 5 | 3 | 3 | 3 |
| 3 | SIEM Wazuh (Oracle Cloud) | [S] Servicio | 4 | 5 | 4 | 4 | 5 |
| 4 | Kali-Attacker (herramientas de auditoría) | [SW] Software | 2 | 3 | 2 | 2 | 2 |
| 5 | Red interna del laboratorio (segmento Host-Only + túnel hacia Oracle) | [COM] Comunicaciones | 4 | 4 | 3 | 3 | 2 |
| 6 | Credenciales y claves SSH (incl. clave privada de la VM Oracle) | [D] Datos (clave) | 5 | 5 | 2 | 4 | 2 |
| 7 | Copias de seguridad (backups de DVWA) | [D] Datos | 3 | 5 | 4 | 2 | 2 |
| 8 | Logs de auditoría (`auth.log`, alertas Wazuh, evidencias del TFG) | [D] Datos | 3 | 5 | 2 | 4 | 5 |

## 3. Amenazas, riesgo y salvaguardas

| # | Activo | Amenaza principal (catálogo MAGERIT) | Prob. | Impacto | Riesgo | Salvaguardas ya aplicadas (verificadas en el TFG) | Riesgo residual |
|---|---|---|---|---|---|---|---|
| 1 | DVWA-VICTIM | [A.11] Acceso no autorizado | 3 | 4 | 12 (Medio) | Firewall deny-by-default + fail2ban (4.2), MFA en SSH (4.1), correcciones reales de SQLi/Command Injection/CSRF (3.6) | 6 (Bajo) |
| 2 | BD MariaDB | [A.18] Destrucción de información | 2 | 5 | 10 (Medio) | Copias de seguridad reales + prueba de restauración verificada con RTO cumplido (5.7) | 4 (Bajo) |
| 3 | SIEM Wazuh | [A.24] Denegación de servicio / [A.11] Acceso no autorizado | 3 | 5 | 15 (Alto) | Firewall en la instancia cloud, `allowed-ips` restringido en el conector syslog — pero **IP pública dinámica del Mac rompe la conectividad sin aviso** (incidente real recurrente, documentado en 5.1, no un hecho aislado — se repitió tres veces a lo largo del bloque), y las credenciales siguen siendo las generadas por el instalador sin rotar (también en 5.1) | 10 (Medio) — riesgo no completamente mitigado, recogido en el PDS |
| 4 | Kali-Attacker | [E.1] Errores de los usuarios | 2 | 2 | 4 (Bajo) | Aislamiento de red, sin datos de negocio reales que perder | 2 (Bajo) |
| 5 | Red del laboratorio | [A.5] Suplantación de identidad / [A.9] Reencaminamiento de mensajes | 2 | 4 | 8 (Medio) | Segmentación Host-Only sin salida a Internet salvo conmutación puntual y documentada (1.2); tráfico hacia Wazuh limitado a los puertos necesarios | 4 (Bajo) |
| 6 | Credenciales/claves SSH | [A.19] Divulgación de información | 3 | 5 | 15 (Alto) | Permisos `chmod 600` en la clave; único usuario con acceso local a la máquina que la almacena, sin otros usuarios del sistema que puedan leerla | 4 (Bajo) — mitigación aplicada; queda como buena práctica pendiente rotar la clave periódicamente, no como riesgo abierto |
| 7 | Backups | [A.18] Destrucción de información / [E.15] Alteración accidental | 2 | 5 | 10 (Medio) | Prueba de recuperación real ejecutada (5.7); **pero el backup se guardó en el mismo servidor que protege (`/tmp`)**, no en almacenamiento externo — limitación ya documentada honestamente en 5.7 | 8 (Medio) — mitigación pendiente en el PDS |
| 8 | Logs de auditoría | [A.19] Divulgación de información / [E.15] Alteración accidental | 2 | 4 | 8 (Medio) | Reenvío a un SIEM externo al propio host (dificulta la manipulación por un atacante que comprometa solo la máquina origen); regla de correlación aún no implementada para parte de las fuentes (hallazgo de 5.3) | 6 (Bajo) |

## 4. Conclusiones

Los dos activos que partían de riesgo **Alto antes de salvaguardas** (SIEM Wazuh y credenciales/claves SSH) han tenido trayectorias distintas tras aplicar los controles: las credenciales/claves SSH sí bajan a **Bajo** (permisos de fichero restrictivos y un único usuario con acceso local dejan la exposición real baja), pero el SIEM Wazuh se queda en **Medio**, no Bajo, porque todavía tiene dos salvaguardas reales sin aplicar: la IP dinámica del Mac rompiendo la conectividad (recurrente, no resuelta de forma permanente) y la rotación de las credenciales generadas por el instalador. El activo Backups (nº7) también se queda en riesgo residual **Medio**, por el mismo motivo honesto de siempre: la prueba de recuperación es real y funcionó, pero la copia se guarda en el mismo servidor que protege, no en almacenamiento externo. Ninguno de estos dos huecos es un fallo de la evaluación: es exactamente el tipo de brecha que debe alimentar el PDS (apartado 6.3), para que quede como acción concreta con plazo, no como una nota perdida en un documento de riesgos que nadie vuelve a mirar.
