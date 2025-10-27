# Proyecto Final - Ingeniería de Datos e IA con GCP

## **Caso de estudio**
Tienda virtual multi-categoría que vende productos tecnológicos, juguetes, motocicletas y maquillaje. Cuenta con una plataforma eCommerce funcional con control de stock y canales de atención: WhatsApp Business, Facebook/Messenger, Llamadas, Correo y Web.

## **Problema**
Actualmente la plataforma eCommerce funciona, pero el control de inventario es manual. Cada día debo:
* Entrar a la web del eCommerce e iniciar sesión.
* Revisar producto por producto para ver el stock actual.
* Anotar manualmente cuáles productos están bajos.
* Cuando un cliente pregunta por WhatsApp, Facebook o correo, buscar en la web y responder manualmente.
* Para decidir compras, no hay predicciones; las decisiones son a ojo.

Tiempo invertido: entre 2–3 horas diarias en consultas y verificación de productos.

## **Solución Propuesta**
Automatizar el control de inventario usando servicios de Google Cloud Platform (GCP)
* para que el sistema trabaje 24/7 sin mi intervención. El sistema hará:
* Verificación automática de stock cada hora y por evento.
* Alertas a los canales internos (p. ej., WhatsApp) cuando un producto esté bajo: “iPhone 15 – Stock 3/10”.
* Predicción con IA de ventas por categoría y SKU: “Se proyectan 18 labiales este mes; reponer 20 unidades”.
* Atención automática multicanal (WhatsApp, Facebook/Messenger, correo, llamadas IVR) con IA para consultas de disponibilidad, precio y estado de pedido.
* Dashboards en tiempo real con ventas, stock y proyecciones.

## **Servicios GCP Utilizados**
1. Cloud SQL
Base de datos centralizada. Complementa o conecta con tu eCommerce actual para centralizar datos.
2. Cloud Scheduler
Programador automático. Activa tareas cada hora sin intervención manual.
3. Cloud Functions
Código que ejecuta las tareas: verificar stock, enviar alertas, sincronizar datos.
4. BigQuery
Almacén de datos. Guarda histórico de ventas e inventario para análisis.
5. Vertex AI
Inteligencia Artificial. Predice demanda futura basándose en histórico de ventas.
6. Dialogflow CX
Chatbot para WhatsApp. Responde automáticamente consultas de disponibilidad.
7. Looker Studio
Dashboards visuales. Muestra métricas actualizadas en tiempo real.


## 🗺️ Diagrama de Arquitectura Actual - AS IS

```mermaid
%%{init: {"theme":"base"}}%%
flowchart LR

  %% Estilos
  classDef canal fill:#E3F2FD,stroke:#1A73E8,stroke-width:1.5px,color:#0F1A2B;
  classDef ia fill:#FFF8E1,stroke:#FB8C00,stroke-width:1.5px,color:#402A00;
  classDef api fill:#E8F5E9,stroke:#2E7D32,stroke-width:1.5px,color:#0F2A10;
  classDef datos fill:#FCE8E6,stroke:#C62828,stroke-width:1.5px,color:#3A0E0E;

  %% Canales del Cliente
  subgraph Canal["Canales del Cliente"]
    WAB[WhatsApp Business]
    FB[Facebook / Messenger]
    CALL[Llamadas / Call Center]
    MAIL[Correo Electrónico]
  end
  class WAB,FB,CALL,MAIL canal;

  %% IA Conversacional
  subgraph IA["IA Conversacional"]
    BOT[Dialogflow CX]
  end
  class BOT ia;

  %% Backend / API
  subgraph API["API Backend"]
    SRV[Cloud Run / API REST]
  end
  class SRV api;

  %% Base de Datos
  subgraph DB["Datos"]
    MYSQL[(MySQL<br/>Minegocio)]
  end
  class MYSQL datos;

  %% Flujo
  WAB --> BOT
  FB --> BOT
  CALL --> BOT
  MAIL --> BOT

  BOT -->|Webhook| SRV
  SRV -->|Consultas / Pedidos / Stock| MYSQL
  SRV -->|Respuestas| BOT

  BOT --> WAB
```
## 🗺️ Diagrama de Arquitectura GCP - TO BE
```mermaid
%%{init: {"theme":"base"}}%%
flowchart LR

  %% Estilos
  classDef canal fill:#D7EBFF,stroke:#1A73E8,stroke-width:1.3px,color:#003366;
  classDef ia fill:#FFF4CC,stroke:#FF9800,stroke-width:1.3px,color:#663C00;
  classDef api fill:#E7F8EC,stroke:#2E7D32,stroke-width:1.3px,color:#0F3D1E;
  classDef datos fill:#FFE6E6,stroke:#C62828,stroke-width:1.3px,color:#4A0606;
  classDef bi fill:#EEF3FF,stroke:#3F51B5,stroke-width:1.3px,color:#0D1331;

  %% Canales del cliente
  subgraph CANALES["Canales del cliente"]
    WAB[WhatsApp Business]
    FB[Facebook o Messenger]
    CALL[Llamadas o Call Center]
    MAIL[Correo Electronico]
    WEB[Plataforma Web de Pedidos]
  end
  class WAB,FB,CALL,MAIL,WEB canal;

  %% IA Conversacional
  subgraph IA["IA Conversacional"]
    BOT[Dialogflow CX]
  end
  class BOT ia;

  %% Backend Automatizado
  subgraph APP["Automatizacion Backend"]
    SCH[Cloud Scheduler]
    FUNC[Cloud Functions]
  end
  class SCH,FUNC api;

  %% Base de datos
  subgraph DATA["Base de Datos"]
    SQL[(Cloud SQL)]
  end
  class SQL datos;

  %% Analitica e IA predictiva
  subgraph ANALYTICS["Analitica y Prediccion"]
    BQ[(BigQuery - Historico)]
    VA[Vertex AI - Prediccion]
    LOOKER[Looker Studio - Dashboard]
  end
  class BQ,VA,LOOKER bi;

  %% Flujo Operativo
  WAB --> BOT
  FB --> BOT
  CALL --> BOT
  MAIL --> BOT
  WEB --> FUNC

  BOT -->|webhook| FUNC
  FUNC -->|consultas y pedidos| SQL
  FUNC -->|respuesta| BOT
  FUNC -->|respuesta| WEB

  %% Automatizacion Stock
  SCH --> FUNC

  %% Analitica
  FUNC --> BQ
  BQ --> VA
  BQ --> LOOKER
  VA --> LOOKER

  %% Respuesta hacia canales
  BOT --> WAB
  BOT --> FB
  BOT --> CALL
  BOT --> MAIL
```
## **Descripción del Funcionamiento**
## **Flujo 1: Descripción del Funcionamiento (Actualizado Multicanal)**
  1. Cloud Scheduler activa la verificación automáticamente cada hora (ej: 8:00, 9:00, 10:00…).
  2. Cloud Functions consulta el inventario en Cloud SQL (Minegocio).
  3. Si encuentra productos con:
    stock_actual < stock_minimo
    → registra el evento en BigQuery (histórico).
  4. Cloud Functions envía una alerta automática al canal interno (WhatsApp Business).
  5. El Dashboard en Looker Studio se actualiza automáticamente.
## **Flujo 2: Predicción de Demanda (IA para reposición inteligente)**
  1. Cada noche a las 11:00 PM, Cloud Scheduler ejecuta el análisis de demanda.
  2. Cloud Functions extrae ventas históricas desde BigQuery (ej: últimos 12 meses).
  3. Vertex AI analiza el comportamiento de ventas y predice la demanda futura.
  4. Se calcula automáticamente la cantidad recomendada de reposición:
    demanda_predicha – stock_actual
  5. Se guarda la recomendación en BigQuery y se muestra en Looker Studi
## **Flujo 3: Atención Automática Multicanal (WhatsApp, Facebook, Llamadas, Correo y Web)**
  1. El cliente realiza una consulta desde cualquier canal:
      * WhatsApp Business
      * Facebook/Messenger
      * Llamada (IVR/Agente asistido)
      * Correo electrónico
      * Plataforma Web
  2. Dialogflow CX interpreta la intención del cliente (ej: consultar stock, precio, estado de pedido).
  3. Cloud Functions consulta la información en Cloud SQL (Minegocio).
  4. El bot responde automáticamente en el mismo canal donde el cliente inició la conversación.al
