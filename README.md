# FlowPyme — Ecosistema de Automatización IA Autónomo

Proyecto final del curso de **AI Automation**. Un ecosistema de automatización de punta a punta que genera **propuestas comerciales personalizadas** para una consultora de automatización (caso ficticio: *FlowPyme*), con un punto de validación humana (Human-in-the-Loop) antes de contactar al cliente.

---

## 🎯 Qué hace

Un prospecto completa un formulario describiendo el proceso que quiere automatizar. El sistema:

1. Guarda el lead en una base de datos (Airtable) como memoria del sistema.
2. Valida que los datos estén completos (si faltan, registra el error y no avanza).
3. Un **agente de IA con RAG** consulta el catálogo de servicios y redacta una propuesta personalizada, recomendando el paquete adecuado **con precios reales del catálogo** (sin inventar).
4. Deja la propuesta en estado *En Revisión* y avisa al equipo por email.
5. Un humano revisa y **aprueba** (HITL). Recién ahí el sistema envía la propuesta al cliente y cierra el registro.

---

## 🧱 Stack (tecnologías integradas)

| Componente | Tecnología |
|---|---|
| Orquestador | **n8n** |
| Base de datos / memoria | **Airtable** (3 tablas con relaciones y estados) |
| Procesamiento IA | **Agente n8n (AI Agent)** + modelo vía **OpenRouter**, con **RAG** sobre el catálogo |
| Canal de salida | **Gmail (SMTP)** |

---

## 🏗️ Arquitectura

El sistema se compone de **dos flujos independientes** coordinados por el campo `Estado` de la base de datos (máquina de estados):

- **Flujo A — Generación** (trigger: formulario / webhook): recibe el lead → valida → agente IA + RAG → guarda borrador → aviso HITL.
- **Flujo B — Distribución** (trigger: Schedule): detecta las propuestas aprobadas → envía la propuesta al cliente → marca como *Enviado*.

Diagrama completo y explicación en **[`docs/EntregaFinal_Arquitectura.pdf`](docs/EntregaFinal_Arquitectura.pdf)**.

---

## ✅ Requisitos cubiertos

- **Disparo, procesamiento con IA desde la DB y salida sin intervención manual** (salvo la aprobación humana intencional).
- **Rutas de error:** validación de datos faltantes con registro en `Error_Log`, más Error Handler con reintentos sobre la IA.
- **Human-in-the-Loop:** aprobación manual obligatoria antes de contactar al cliente.
- **Anti-bucle infinito:** el cambio de estado saca la fila de la vista que observa el trigger.
- **Tipos de datos correctos:** el presupuesto se compara como número.
- **Prompt dinámico** con variables del sistema, sin datos hardcodeados.

---

## 📁 Contenido del repositorio

- [`docs/EntregaFinal_Arquitectura.pdf`](docs/EntregaFinal_Arquitectura.pdf) — Diagrama de arquitectura y documentación.
- [`workflow/Entrega-Final.json`](workflow/Entrega-Final.json) — Lógica del flujo (exportado de n8n, sin credenciales).
- [`evidencias/`](evidencias/) — Capturas del sistema funcionando (test de estrés, estados, mails).

## 🔗 Enlaces

- **Base de datos (Airtable, modo lectura):** https://airtable.com/appV8S3y4kk105pBD/shrf5dtdAH3eIc97P
- **Video demo (3 min): https://drive.google.com/file/d/1UsT_K-jSvmFySVwegfY_YTIxojfu-1pk/view?usp=sharing
---

## 🔐 Nota de seguridad

El archivo JSON exportado **no contiene credenciales**: las API keys y contraseñas viven en el almacén de credenciales de n8n y solo se referencian por nombre. Las claves fueron ocultadas en el video de la demo.
