# Proyecto Final - Ingeniería de Datos e IA con GCP

## **Caso de estudio**
Tienda virtual multi-categoría que vende productos tecnológicos, juguetes,
motocicletas y maquillaje. Tiene plataforma eCommerce funcional con control de stock

## **Problema**
Actualmente tengo una plataforma eCommerce que funciona bien, pero el control de
inventario es completamente MANUAL. Cada día debo:
• Entrar a la web del eCommerce, iniciar sesión
• Revisar producto por producto para ver el stock actual
• Anotar manualmente cuáles productos están bajos
• Cuando un cliente pregunta por WhatsApp, debo buscar en la web y responder manualmente
• Para decidir qué comprar, no tengo predicciones, decido 'a ojo'
Tiempo invertido: En consultas y verificación de productos se invierte diariamente entre 2-3
horas diarias aproximadamente.

## **Solución Propuesta**
Automatizar el control de inventario usando servicios de Google Cloud Platform (GCP)
para que el sistema trabaje 24/7 sin mi intervención. El sistema hará:
• Verificar automáticamente el stock cada hora (sin que yo entre a la web)
• Enviar alertas a WhatsApp cuando un producto esté bajo: 'iPhone 15: Stock 3/10'
• Predecir con IA cuánto se venderá: 'Venderás 18 labiales este mes, pide 20 unidades'
• Responder automáticamente a clientes en WhatsApp: '¿Tienes iPhone? ® Sí, 5 unidades'
• Mostrar dashboards con ventas, stock y predicciones actualizados en tiempo real

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
%%{init: {"theme":"base", "themeVariables":{
"primaryColor":"#e8f0fe","primaryTextColor":"#1f1f1f","primaryBorderColor":"#1a73e8",
"lineColor":"#5f6368","fontFamily":"Inter,Segoe UI,Helvetica,Arial"}}}%%
flowchart LR
  classDef canal fill:#E3F2FD,stroke:#1A73E8,stroke-width:1.5px,color:#0F1A2B;
  classDef ia fill:#FFF8E1,stroke:#FB8C00,stroke-width:1.5px,color:#402A00;
  classDef api fill:#E8F5E9,stroke:#2E7D32,stroke-width:1.5px,color:#0F2A10;
  classDef datos fill:#FCE8E6,stroke:#C62828,stroke-width:1.5px,color:#3A0E0E;

  subgraph Canal["Canales del Cliente"]
    WAB[WhatsApp Business]
    WEB[Plataforma Web<br/>Consultas y Pedidos]
  end
  class WAB,WEB canal;

  subgraph IA["IA Conversacional"]
    BOT[Motor de IA / NLU<br/>Dialogflow u otro]
  end
  class BOT ia;

  subgraph API["API Backend"]
    SRV[Servicio API<br/>REST / Webhook]
  end
  class SRV api;

  subgraph DB["Datos"]
    MYSQL[(MySQL<br/>Base de Datos)]
  end
  class MYSQL datos;

  WAB -->|Mensajes| BOT
  BOT -->|Webhook| SRV
  WEB -->|HTTP/HTTPS| SRV
  SRV -->|CRUD| MYSQL
  SRV -->|Respuesta| WEB
  SRV -->|Respuesta/Texto| BOT
  BOT -->|Mensaje| WAB
```
## 🗺️ Diagrama de Arquitectura GCP - TO BE
```mermaid
%%{init: {"theme":"base"}}%%
flowchart LR
  subgraph Canal["Canal del Cliente"]
    WAB[WhatsApp Business]
  end
  subgraph IA["IA Conversacional"]
    CX[Dialogflow CX]
  end
  subgraph App["Aplicación Serverless"]
    RUN[Cloud Run / Functions]
    SCH[Cloud Scheduler]
  end
  subgraph Datos["Transaccional"]
    SQL[(Cloud SQL — Réplica de Minegocio)]
  end
  subgraph Analitica["Analítica"]
    BQ_RAW[(BigQuery - RAW)]
    BQ_VIEW[(BigQuery - VIEW)]
    LOOKER[Looker Studio]
  end
  WAB --> CX
  CX --> RUN
  SCH -. cada hora .-> RUN
  RUN --> SQL
  RUN -- histórico -> BQ_RAW
  BQ_RAW --> BQ_VIEW
  BQ_VIEW --> LOOKER
```
## **Descripción del Funcionamiento**
## **Flujo 1: Monitoreo Automático**
  1. Cloud Scheduler activa cada hora (ej: 8am, 9am, 10am...)
  2. Cloud Function lee stock de todos los productos
  3. Si detecta stock_actual < stock_minimo ® Guarda en BigQuery
  4. Envía mensaje WhatsApp: 'nn Labial Maybelline: 3 unidades (mínimo: 8)'
  5. Dashboard Looker se actualiza automáticamente
## **Flujo 2: Predicción de Demanda**
  1. Cada noche a las 11pm, Cloud Scheduler activa análisis
  2. Cloud Function extrae histórico de ventas (últimos 12 meses)
  3. Vertex AI analiza y predice: 'Venderás 23 labiales próximo mes'
  4. Calcula: 23 - 3 (stock actual) = 20 ® 'Pide 20 unidades'
  5. Guarda recomendación en BigQuery
## **Flujo 3: Atención Cliente WhatsApp**
  1. Cliente: '¿Tienes iPhone 15?'
  2. Dialogflow CX procesa pregunta
  3. Cloud Function consulta stock en base de datos
  4. Bot responde: 'Sí, tenemos 5 unidades a S/. 4,500'
  5. Todo en 2 segundos, sin intervención manual
