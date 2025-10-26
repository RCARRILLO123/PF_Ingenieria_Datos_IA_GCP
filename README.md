# Proyecto Final - Ingeniería de Datos e IA con GCP

## **Arquitectura del Caso: Automatización de Ecommerce + WhatsApp**

```mermaid
flowchart LR
  WAB[WhatsApp Business] --> CX[Dialogflow CX]
  CX --> RUN[Cloud Run - API Webhook]
  RUN --> SQL[(Cloud SQL - Productos, Stock, Pedidos)]
  RUN --> BQRAW[(BigQuery - Datos RAW)]
  BQRAW --> BQVIEW[(BigQuery - Vista Procesada)]
  BQVIEW --> LOOKER[Looker Studio - Dashboard]
  SCH[Cloud Scheduler] --> RUN
```
