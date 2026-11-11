# Playbook de Detección y Respuesta
## Fuerza bruta SSH contra host monitorizado por syslog

**Código:** PB-SOC-001
**Autor:** Daniel Gallardo Fernández
**Versión:** 1.0 — 16 de octubre de 2026
**Clasificación:** Versión pública de portfolio (evidencias completas disponibles bajo petición)
**Táctica/Técnica MITRE ATT&CK:** TA0006 (Credential Access) — T1110 (Brute Force)

---

## 1. Objetivo y alcance

Este playbook describe el procedimiento a seguir cuando el SIEM (Wazuh) detecta un patrón de intentos de autenticación fallidos compatible con un ataque de fuerza bruta contra un servicio SSH. Está escrito a partir de un caso real, documentado en el apartado 5.2 del informe principal (alerta `Rule 2501`, generada por Metasploitable2) y validado con datos reales en el apartado 5.3 (Threat Hunting).

Se centra en hosts que envían logs por **syslog sin agente Wazuh registrado** (como Metasploitable2 en este laboratorio) porque es el caso más delicado: sin agente, Wazuh no puede aislar la máquina de forma automática ni correlacionar eventos por `agent.id`, así que buena parte de la respuesta es necesariamente manual. El mismo procedimiento aplica, simplificado en las fases de contención, a hosts con agente (como DVWA-VICTIM), donde Wazuh sí permite acciones de respuesta activa.

## 1.1 Diagrama de flujo de decisión

```
                    ┌─────────────────────────────┐
                    │ Evento rule.id:2501 recibido │
                    └──────────────┬──────────────┘
                                   │
                                   ▼
                  ┌────────────────────────────────┐
                  │ ¿≥3 eventos mismo usuario/origen │
                  │        en <5 minutos?            │
                  └───────┬───────────────┬─────────┘
                       NO │               │ SÍ
                          ▼               ▼
                 ┌─────────────┐   ┌──────────────────────┐
                 │ Informativo │   │ Severidad MEDIA       │
                 │ No se activa│   │ Activar playbook      │
                 │ el playbook │   │ → Fase 1: Triage      │
                 └─────────────┘   └───────────┬───────────┘
                                                │
                                                ▼
                               ┌────────────────────────────────┐
                               │ ¿Hay al menos 1 login EXITOSO    │
                               │ del mismo origen tras los fallos?│
                               └───────┬────────────────┬────────┘
                                    NO │                │ SÍ
                                       ▼                ▼
                          ┌─────────────────────┐  ┌───────────────────────┐
                          │ Mantener en MEDIA    │  │ Escalar a ALTA         │
                          │ Vigilar, sin         │  │ → saltar directo a    │
                          │ contención activa    │  │   Fase 2: Contención  │
                          └─────────────────────┘  └───────────┬───────────┘
                                                                │
                                                                ▼
                                                   Fase 2 → 3 → 4 → 5 → 6
                                                   (Contención → Erradicación
                                                    → Recuperación →
                                                    Comunicación → Lecciones)
```

## 2. Activación (trigger)

Este playbook se activa cuando se cumple cualquiera de estas condiciones:

- **Activación manual (caso actual):** una query de Threat Hunting como `rule.id: 2501` devuelve **3 o más eventos del mismo usuario y origen en una ventana de 5 minutos** (umbral definido a partir del dato real observado: 8 eventos en 23 segundos es claramente anómalo; 3 en 5 minutos es un umbral conservador pensado para no generar ruido con errores de tecleo aislados).
- **Activación automática (pendiente, ver apartado 7):** en cuanto exista una regla de correlación propia en `local_rules.xml` que agrupe estos eventos, la activación pasaría a ser la propia alerta de Wazuh de nivel ≥10.

## 3. Clasificación de severidad inicial

| Condición observada | Severidad |
|---|---|
| 1-2 fallos aislados, sin patrón temporal | Informativo — no se activa el playbook |
| ≥3 fallos del mismo usuario/origen en <5 min | **Media** — se activa este playbook |
| ≥3 fallos + al menos 1 éxito posterior del mismo origen | **Alta** — posible compromiso, saltar directamente a Contención |

## 4. Procedimiento

### Fase 1 — Detección y triage

1. Confirmar el patrón en el dashboard de Wazuh: Threat Hunting → Events → `rule.id: 2501` (o la regla de correlación futura), ajustando el rango de fechas a la ventana donde se observó el pico.
2. Revisar el campo `full_log` de cada evento para extraer: usuario objetivo (`user=`), host remoto reportado (`rhost=`) y hora exacta.
3. **Importante (hallazgo real documentado en 5.1):** si el origen llega por una red con NAT (como la red Shared de UTM en este laboratorio), el campo `location`/IP de origen que ve Wazuh es la IP pública del NAT, no la IP real del atacante — el `rhost` del propio mensaje syslog es la única pista fiable del origen real dentro de la red interna.
4. Determinar si el usuario objetivo (`msfadmin` en el caso real) es una cuenta válida del sistema. Si no lo es, el riesgo baja (no hay credencial que adivinar), pero el intento sigue siendo indicio de reconocimiento activo.
5. Clasificar según la tabla de severidad del apartado 3.

### Fase 2 — Contención

- **Host con agente Wazuh (ej. DVWA-VICTIM):** Wazuh permite activar una respuesta activa (`active-response`) que puede bloquear la IP de origen automáticamente vía `firewall-drop`. No configurado todavía en este laboratorio — queda como mejora pendiente (apartado 7).
- **Host sin agente, solo syslog (Metasploitable2, caso real):** no hay mecanismo de contención automática. Contención manual:
  1. Si el origen es identificable y interno (red del laboratorio), bloquear a nivel de red: regla de firewall en el host origen o en el propio Metasploitable2 (`iptables -A INPUT -s <IP> -j DROP`) — sabiendo que esta VM, por su antigüedad, carece de `fail2ban` (a diferencia de DVWA-VICTIM, ver apartado 4.2 del informe principal).
  2. Si el patrón indica un ataque ya en curso contra una cuenta válida, considerar deshabilitar temporalmente esa cuenta (`passwd -l <usuario>`) hasta confirmar que no ha sido comprometida.

### Fase 3 — Erradicación

1. Confirmar que no hay sesiones activas abiertas con el usuario objetivo en el momento del ataque (`who`, `last` en el host afectado).
2. Si se confirma un acceso exitoso tras los fallos (severidad Alta), tratar el host como potencialmente comprometido: revisar procesos en ejecución, crontab, y archivos modificados recientemente — mismo criterio de post-explotación ya aplicado en el Bloque Ofensivo (apartado 2.5).
3. Forzar cambio de contraseña del usuario objetivo si la cuenta es válida y el ataque fue sostenido.

### Fase 4 — Recuperación

1. Retirar el bloqueo de contención si era temporal y se ha confirmado que el origen ya no es una amenaza activa.
2. Verificar que el servicio SSH del host afectado sigue respondiendo con normalidad para los usuarios legítimos.
3. Documentar el incidente (ver Fase 6).

### Fase 5 — Comunicación y escalado

**Criterios de escalado:**

| Condición | A quién escalar | Cuándo |
|---|---|---|
| Severidad Media, contenida en <30 min | Ninguno — se cierra a nivel de analista | Tras Fase 4 |
| Severidad Alta (login exitoso tras fuerza bruta) | Responsable del sistema afectado | Inmediatamente al confirmar la severidad (Fase 1, paso 5) |
| Confirmación de compromiso real (datos exfiltrados, persistencia instalada) | Responsable del sistema + CISO | Inmediatamente, en paralelo a la contención, no después |
| Indicios de que el origen es externo a la red del laboratorio | CISO (posible brecha con impacto fuera del entorno controlado) | Inmediatamente |

En un entorno real, este paso implicaría notificar al responsable del sistema afectado y, si procede, al CISO, por el canal de comunicación formal de la organización (ticketing, chat de guardia, etc.). En el contexto de este laboratorio (operador único), se documenta como si existiera ese canal: registro del incidente con hora de detección, hora de contención y resumen, usando la plantilla de ticket del apartado 6.

### Fase 6 — Lecciones aprendidas

Preguntas a responder tras cada ejecución real de este playbook:

- ¿Se detectó a tiempo, o se encontró retrospectivamente mediante Threat Hunting manual? *(En el caso real documentado en 5.3: retrospectivamente — no había alerta automática de nivel alto, solo 8 eventos individuales de nivel 5.)*
- ¿La contención fue automática o manual? ¿Por qué?
- ¿Qué mejora de detección/respuesta se puede implementar antes del próximo incidente similar?

## 4.1 Condiciones de cierre del incidente

El incidente activado por este playbook se considera **cerrado** cuando se cumplen TODAS las siguientes condiciones:

1. El origen del ataque ya no tiene acceso activo al sistema (contención verificada: bloqueo de red confirmado o IP ya no aparece en nuevos intentos).
2. No queda ningún mecanismo de persistencia, cuenta nueva ni proceso atribuible al atacante en el host afectado (erradicación verificada).
3. El servicio afectado responde con normalidad para usuarios legítimos (recuperación verificada).
4. Se ha completado el registro de evidencias con timeline (ver plantilla de ticket, apartado 6) y, si aplica, se ha preservado copia de los logs relevantes antes de que roten o se sobrescriban.
5. Se ha documentado al menos una lección aprendida o mejora derivada del incidente (Fase 6).

Si alguna de estas condiciones no se cumple, el incidente permanece **abierto** aunque haya pasado tiempo desde la detección inicial.

## 5. Indicadores de compromiso (IOC) relacionados

| Tipo | Valor / patrón | Fuente |
|---|---|---|
| Regla Wazuh | `rule.id: 2501` ("syslog: User authentication failure") | Manager Wazuh |
| Patrón de comportamiento | ≥3 eventos `rule.id: 2501` del mismo `user` en <5 min | Threat Hunting / Events |
| Cuenta objetivo observada (caso real) | `msfadmin` | `full_log` del evento |

## 6. Plantilla de ticket de incidente

Plantilla a rellenar cada vez que se activa este playbook. Como ejemplo de cómo quedaría rellena, se usa a continuación el incidente ya documentado en 5.3 (la ráfaga de 8 intentos de fuerza bruta generada para el Threat Hunting), tratado como si se hubiera gestionado como un incidente formal:

| Campo | Valor |
|---|---|
| **ID de ticket** | INC-2026-10-16-001 |
| **Playbook aplicado** | PB-SOC-001 |
| **Fecha/hora de detección** | 2026-10-16, 12:26:45 (hora de la última de las 8 alertas) |
| **Detectado por** | Query manual de Threat Hunting (`rule.id: 2501`, Last 15 minutes) — no hubo alerta automática de severidad alta |
| **Host afectado** | Metasploitable2 (192.168.64.5 en red Shared / 192.168.128.4 en Host-Only) |
| **Usuario objetivo** | `msfadmin` |
| **Origen** | IP pública del NAT de UTM (`<IP pública del Mac>`); origen real interno no identificable más allá del `rhost` reportado por el propio host |
| **Severidad asignada** | Media (8 fallos en 23s, **sin** login exitoso posterior — no se escaló a Alta) |
| **Resumen** | 8 intentos de login SSH fallidos contra `msfadmin`, con 8 contraseñas distintas, en una ventana de 23 segundos. Patrón consistente con T1110 (Brute Force). |
| **Acciones de contención tomadas** | Ninguna activa (severidad Media, sin éxito posterior) — se mantuvo en observación |
| **Acciones de erradicación tomadas** | No aplica — no hubo compromiso |
| **Hora de cierre** | 2026-10-16, 12:36 (tras confirmar en el dashboard que no hubo ningún login exitoso posterior del mismo origen) |
| **Lección aprendida** | Sin regla de correlación propia, este patrón solo se detecta con una query manual — mejora pendiente documentada en el apartado 7 |

## 7. Mejoras pendientes (honestas, no implementadas aún)

1. **Regla de correlación propia** en `local_rules.xml` del manager que agrupe automáticamente varios `rule.id: 2501` del mismo origen/usuario en una ventana corta y los escale a severidad alta (nivel ≥10) — eliminaría la dependencia de una query manual de Threat Hunting para detectar el patrón, que es la principal debilidad identificada en el apartado 5.3.
2. **Respuesta activa automática** (`active-response` de Wazuh) para hosts con agente, de forma que la contención de la Fase 2 no dependa de intervención manual.
3. **`fail2ban` o equivalente en Metasploitable2** — inviable en la práctica por la antigüedad del sistema (ver 4.2 y 5.1), documentado como limitación aceptada, no como tarea pendiente real.
