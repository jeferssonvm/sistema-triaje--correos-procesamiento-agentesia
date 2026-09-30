# Proyecto Personal / PoC: Sistema de Triaje de Correos y Procesamiento Documental Multimodal con Agentes IA

## Información del Proyecto
- **Tipo**: Proyecto Personal / Prueba de Concepto (PoC) & Entorno de Simulación.
- **Caso de Uso Simulado**: Operación corporativa y atención multicanal para la empresa ficticia *"Soluciones Tec"*.
- **Enfoque**: Validación técnica de arquitecturas multi-agente, manejo de adjuntos binarios (PDFs) y automatización basada en eventos (*event-driven automation*).

---

## Resumen Ejecutivo
Prueba de concepto funcional desarrollada como proyecto personal para simular y validar una arquitectura automatizada de atención corporativa y triaje multicanal. El sistema emula la recepción, clasificación semántica, validación de reglas de negocio en tiempo real y despacho de respuestas automatizadas de correos con o sin archivos adjuntos (PDFs).

A través de modelos de lenguaje (Google Gemini) y herramientas dinámicas (Google Sheets), el flujo clasifica y atiende requerimientos en cuatro departamentos empresariales simulados: **Ventas**, **Soporte Técnico**, **Facturación** y **Recursos Humanos**.

---

## Aspectos Destacados de Ingeniería
- **Pipeline Condicional Multimodal**: Detección reactiva de archivos binarios adjuntos (`PDF`) y enrutamiento hacia un subflujo de extracción de texto antes de la inferencia del modelo.
- **Arquitectura Multi-Agente Asíncrona**:
  - *Agente Documental (Extractor)*: Analiza el texto del PDF y genera metadatos estructurados según el tipo de archivo (facturas, CVs, reportes técnicos, soportes fiscales).
  - *Agente de Triaje y Negocio (Orquestador)*: Utiliza herramientas dinámicas (`Google Sheets Tool`) para verificar los campos obligatorios y opcionales parametrizados para cada área.
  - *Agente Redactor de Fallbacks*: Genera respuestas automáticas en HTML asistidas por IA cuando el remitente omite datos obligatorios indispensables.
- **Salidas Deterministas con JSON Schema**: Implementación de `Structured Output Parser` para garantizar respuestas tipadas y evitar alucinaciones en la lógica del workflow.
- **Enrutamiento y Lógica Condicional (Switch / If)**: Bifurcaciones basadas en reglas para despachar respuestas HTML personalizadas (agendamiento interactivo en Meet, confirmación de radicación contable con SLA, o registro de talento).

---

## Arquitectura del Flujo de Trabajo (Workflow Breakdown)

```
                       [ Gmail Trigger ]
                               |
                        [ Edit Fields ] (Normalización de metadatos)
                               |
                        [ If Binary? ]
                       /              \
             (Con PDF)                 (Solo Texto)
                 |                          |
        [ Get Message ]                     |
                 |                          |
       [ Extract from File ]                |
                 |                          |
        [ AI Agent 2 (Gemini) ]             |
        (Clasificación Doc)                 |
                 |                          |
        [ Edit Fields 1 ]                   |
                 \                         /
                  --> [ Merge Node ] <-----
                             |
                     [ AI Agent (Gemini) ]
                     ├── Tool: Google Sheets (Reglas de negocio)
                     └── Output Parser: JSON Schema
                             |
                     [ Switch (4 Áreas) ]
                 /         |          |         \
            [Ventas]   [Soporte]  [Factura]   [RRHH]
               |           |          |         |
             (If Obligatorios Completos?) x4
             ├── [True]  --> Respuesta de éxito personalizada (HTML / Meet / Radicación)
             └── [False] --> [ AI Agent 1 ] --> Solicitud de datos faltantes interactiva
```

---

## Especificaciones Técnicas

| Componente | Tecnología / Módulo | Propósito en la Simulación |
|---|---|---|
| **Orquestador** | n8n (Workflow Automation) | Control del ciclo de vida del flujo, bifurcaciones lógicas y manejo de datos. |
| **Inferencia / LLM** | Google Gemini (`models/gemini-3.5-flash-lite`) | Extracción de entidades, razonamiento semántico y síntesis de respuestas. |
| **Integraciones Core** | Gmail OAuth2, Google Sheets API | Ingesta de correos simulados, consulta de reglas de negocio y respuestas. |
| **Parsers & Validadores** | `Extract from File (PDF)`, `StructuredOutputParser` | Conversión binaria a texto plano y validación de tipos por esquema JSON. |
| **Frontend de Salida** | HTML5 / CSS Inline Responsive | Comunicación institucional estructurada con bloques informativos y trazabilidad. |

---

## Casos de Uso Simulados por Departamento

### 1. Ventas y Nuevos Negocios
- **Entrada**: Solicitud de servicios, presupuestos o cotizaciones comerciales.
- **Validación**: Comprobación de producto de interés, requerimiento y datos de contacto.
- **Acción Exitosa**: Confirmación formal por correo con botón de agendamiento interactivo en tiempo real vía Google Meet y Calendar.

### 2. Soporte Técnico y Operaciones
- **Entrada**: Notificación de incidencias técnicas o logs de fallos de sistemas.
- **Validación**: Verificación de descripción de la incidencia y número de contacto.
- **Acción Exitosa**: Creación de ticket simulado con SLA de atención prioritaria (<15 min) y bloque de trazabilidad.

### 3. Facturación y Cuentas por Pagar
- **Entrada**: Facturas electrónicas o cuentas de cobro adjuntas en PDF.
- **Validación**: Extracción automática de CUFE/referencia, NIT del tercero emisor, total y concepto.
- **Acción Exitosa**: Notificación de radicación contable con compromiso de respuesta y causación en 3 días hábiles.

### 4. Selección y Recursos Humanos (RRHH)
- **Entrada**: Candidaturas espontáneas o postulaciones laborales con currículum (CV) adjunto.
- **Validación**: Identificación del perfil profesional, datos de contacto y validación de archivo curricular.
- **Acción Exitosa**: Notificación de inclusión del candidato en la base de talento para futuras vacantes.

### 5. Bucle de Solicitud de Datos Faltantes (Fallback Loop)
- Si una solicitud carece de campos esenciales (`obligatorios_completos: false`), el flujo activa `AI Agent1` para redactar una respuesta cordial que detalla exactamente la información ausente y propone campos opcionales para agilizar el trámite.

---

## Formato para Hoja de Vida (Bullets Resumidos)

> **Proyecto Personal / PoC: Sistema de Triaje Documental y Automatización con Agentes IA (n8n + Gemini)**
> - Diseñó y construyó una prueba de concepto (PoC) para simular la mesa de ayuda multicanal de una organización (*Soluciones Tec*), automatizando el triaje y respuesta de correos entrantes.
> - Desarrolló un pipeline multimodal con bifurcación condicional para procesar adjuntos PDF (facturas, CVs, reportes), extrayendo entidades con Google Gemini y validando datos contra reglas de negocio en Google Sheets.
> - Implementó salidas tipadas mediante esquemas JSON rígidos (`Structured Output Parser`) y enrutamiento determinista hacia plantillas corporativas HTML y un flujo de recuperación de información faltante.

## Evidencias de Ejecución (Outputs Generados)

Las respuestas procesadas y enviadas por el sistema se encuentran respaldadas en formato PDF en el directorio `respuestas/`:

- **Recursos Humanos**: [`respuestas/Gmail - solicitud de trabajo.pdf`](respuestas/Gmail%20-%20solicitud%20de%20trabajo.pdf) — Confirmación automática de postulación y registro en la base de talento.
- **Facturación**: [`respuestas/solicitud de pago.pdf`](respuestas/solicitud%20de%20pago.pdf) — Constancia de radicación contable con extracción de datos del soporte y SLA de 3 días hábiles.
- **Soporte Técnico**: [`respuestas/soporte tecnico.pdf`](respuestas/soporte%20tecnico.pdf) — Notificación de apertura de ticket con tiempo estimado de respuesta (<15 min).
- **Ventas**: [`respuestas/ventas.pdf`](respuestas/ventas.pdf) — Confirmación comercial con botón de agendamiento interactivo en Google Meet.