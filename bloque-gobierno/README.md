# Gobierno de la Seguridad — Política ISO 27001, MAGERIT y Plan Director

![ISO 27001](https://img.shields.io/badge/ISO%2FIEC_27001-2022-informational?style=for-the-badge)
![MAGERIT](https://img.shields.io/badge/MAGERIT-v3-orange?style=for-the-badge)
![NIST CSF](https://img.shields.io/badge/NIST_CSF-5_funciones-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Bloque%20Gobierno-Completo-success?style=for-the-badge)

Último bloque del TFG, puramente documental: no añade sistemas nuevos, sino que formaliza y estructura — con los marcos de referencia habituales en gestión de la seguridad — todo lo que ya se construyó, probó y documentó en los cinco bloques anteriores. Sin imágenes propias: aquí el trabajo es la redacción, no la evidencia técnica.

---

## [Política de Seguridad de la Información](documentos/Politica_Seguridad_Informacion.md) — ISO/IEC 27001

Redactada siguiendo la estructura de dominios de ISO/IEC 27001 (versión resumida, sin perseguir certificación — no tendría sentido a la escala de un laboratorio de TFG), con un rasgo que la distingue de una plantilla genérica: cada apartado remite a un control que **ya se implementó y se verificó realmente** en un bloque anterior — política de contraseñas y MFA en el [Bloque Protección](../bloque-proteccion/), segmentación de red en el [Bloque Ofensivo](../bloque-ofensivo/), gestión de vulnerabilidades, gestión de incidentes en el [Bloque SOC/IR](../bloque-soc-ir/), continuidad de negocio — en vez de describir intenciones sin respaldo.

---

## [Evaluación de riesgos MAGERIT v3](documentos/Evaluacion_Riesgos_MAGERIT.md)

Tabla de **8 activos reales** del laboratorio (no genéricos: el servidor web, la base de datos, el SIEM, la máquina de ataque, la red interna, las credenciales/claves SSH, las copias de seguridad y los logs de auditoría), valorados en las cinco dimensiones MAGERIT y contrastados contra amenazas reales de su catálogo. Lo relevante no es la tabla en sí, sino que el riesgo residual de cada activo se calcula **después** de aplicar las salvaguardas que de verdad se implementaron a lo largo del proyecto: dos activos se quedan deliberadamente en riesgo residual Medio, no Bajo, porque tienen mitigaciones reales todavía pendientes — algo que una evaluación de cara a la galería no se permitiría admitir.

---

## [Plan Director de Seguridad (PDS) — 12 meses](documentos/Plan_Director_Seguridad_12_meses.md) — NIST CSF

Estructurado según las cinco funciones del NIST Cybersecurity Framework (Identificar, Proteger, Detectar, Responder, Recuperar), con una particularidad que lo distingue de un PDS de plantilla: cada acción tiene un **origen explícito**, el apartado concreto donde se detectó el hueco que la motiva — nada en este plan está inventado para rellenar el documento. Las tres acciones de prioridad Alta del primer trimestre coinciden exactamente con las mitigaciones todavía pendientes identificadas en la evaluación MAGERIT anterior.

---

## Lo que me llevo

Este bloque me enseñó que un documento de gobierno no vale nada si no es trazable hasta algo que de verdad pasó — por eso cada control de la política, cada riesgo residual de MAGERIT y cada acción del PDS señala a un apartado concreto de los bloques anteriores. Es mucho más fácil (y más tentador) escribir una política o un plan director genérico que suene bien; lo que cuesta de verdad es que cada línea responda, si alguien pregunta "¿y esto de dónde sale?", con un hecho concreto del propio laboratorio.

---

*Documentación de gobierno redactada sobre el propio laboratorio, de un único operador, descrito en los cinco bloques anteriores de este mismo proyecto.*
