# Guía de Migración: AWS a Azure

## 📋 Tabla de Contenidos
- [Resumen Ejecutivo](#resumen-ejecutivo)
- [Mapeo de Servicios](#mapeo-de-servicios)
- [Análisis Detallado por Servicio](#análisis-detallado-por-servicio)
- [Estrategia de Migración](#estrategia-de-migración)
- [Costos Estimados](#costos-estimados)

---

## Resumen Ejecutivo

Este documento proporciona un mapeo completo de servicios AWS a Azure basado en tu infraestructura actual, incluyendo análisis de riesgos, pros/contras y recomendaciones específicas.

**Costo Total Actual AWS (Sept 2025):** ~$8,199.19

---

## Mapeo de Servicios

| # | Servicio AWS | Equivalente Azure | Alternativa Azure | Complejidad Migración |
|---|-------------|-------------------|-------------------|----------------------|
| 1 | RDS (Relational Database Service) | Azure SQL Database | Azure Database for PostgreSQL/MySQL | Media |
| 2 | OpenSearch Service | Azure Cognitive Search | Self-hosted on AKS | Alta |
| 3 | Elastic Container Service (ECS) | Azure Container Apps | Azure Kubernetes Service (AKS) | Media |
| 4 | CloudWatch | Azure Monitor | Application Insights | Baja |
| 5 | X-Ray | Application Insights | Azure Monitor (Distributed Tracing) | Media |
| 6 | Config | Azure Policy | Azure Resource Graph | Media |
| 7 | VPC | Azure Virtual Network (VNet) | - | Baja |
| 8 | AppSync | Azure Functions + GraphQL | Azure API Management | Alta |
| 9 | CloudTrail | Azure Activity Log | Azure Monitor Logs | Baja |
| 10 | EC2 Instances | Azure Virtual Machines | Azure Container Instances | Media |
| 11 | EC2-Other (EBS, Snapshots) | Azure Managed Disks | Azure Backup | Baja |
| 12 | Security Hub | Microsoft Defender for Cloud | Azure Security Center | Media |
| 13 | Kinesis | Azure Event Hubs | Azure Stream Analytics | Media |
| 14 | ElastiCache | Azure Cache for Redis | - | Baja |
| 15 | Elastic Load Balancing | Azure Load Balancer | Azure Application Gateway | Baja |
| 16 | WAF | Azure Web Application Firewall | Azure Front Door | Baja |
| 17 | Managed Streaming for Kafka | Azure Event Hubs (Kafka API) | HDInsight Kafka | Media |

---

## Análisis Detallado por Servicio

### 1. RDS → Azure SQL Database / Azure Database for PostgreSQL

**Costo AWS:** $2,113.24/mes

#### Opciones Azure

| Opción | Descripción | Caso de Uso |
|--------|-------------|-------------|
| **Azure SQL Database** | Base de datos SQL Server administrada | Aplicaciones Microsoft Stack |
| **Azure Database for PostgreSQL** | PostgreSQL administrado | Apps open source, compatibilidad PostgreSQL |
| **Azure Database for MySQL** | MySQL administrado | Apps LAMP, WordPress |
| **Azure Cosmos DB** | NoSQL multimodelo | Apps globales, alta disponibilidad |

#### Comparativa Detallada

| Aspecto | AWS RDS | Azure SQL Database | Ganador |
|---------|---------|-------------------|---------|
| **Precio** | $$$ | $$ - $$$ | Azure (con reservas) |
| **Escalabilidad** | Vertical + Read Replicas | Serverless + Elastic Pool | Azure |
| **Backup automático** | 7-35 días | 7-35 días | Empate |
| **Multi-región** | Manual setup | Geo-replicación nativa | Azure |
| **Compatibilidad** | MySQL, PostgreSQL, Oracle, SQL Server | SQL Server, PostgreSQL, MySQL | AWS (más opciones) |
| **Tiempo de recuperación** | Minutos | Segundos (con Azure SQL) | Azure |

#### ✅ Pros Azure
- **Serverless compute**: Paga solo cuando usas (ideal para dev/test)
- **Hyperscale**: Escala hasta 100TB sin migraciones
- **Seguridad avanzada**: Always Encrypted, Dynamic Data Masking
- **Integración**: Mejor integración con servicios Microsoft

#### ❌ Contras Azure
- **Vendor lock-in**: Más difícil migrar de Azure SQL
- **Limitaciones PostgreSQL**: Menos extensiones que RDS PostgreSQL
- **Costos ocultos**: Storage, backup, IOPS por separado

#### ⚠️ Riesgos de Migración
- **Alto**: Diferencias en T-SQL vs PL/pgSQL
- **Medio**: Downtime durante migración (usar Azure DMS)
- **Bajo**: Incompatibilidad de features específicos

#### 💡 Recomendación
```
Si usas PostgreSQL → Azure Database for PostgreSQL (Flexible Server)
Si usas MySQL → Azure Database for MySQL
Si migras a SQL Server → Azure SQL Database (con Hyperscale)
```

---

### 2. OpenSearch → Azure Cognitive Search

**Costo AWS:** $1,115.49/mes

#### Opciones Azure

| Opción | Descripción | Mejor Para |
|--------|-------------|------------|
| **Azure Cognitive Search** | Búsqueda con IA | Búsqueda empresarial, e-commerce |
| **Azure OpenSearch (AKS)** | Self-hosted OpenSearch | Compatibilidad 100% OpenSearch |
| **Elasticsearch on AKS** | Elastic Stack en Kubernetes | Control total, features Elastic |

#### Comparativa Detallada

| Aspecto | AWS OpenSearch | Azure Cognitive Search | Azure OpenSearch (AKS) |
|---------|----------------|------------------------|------------------------|
| **Compatibilidad OpenSearch** | 100% | 0% (API diferente) | 100% |
| **Gestión** | Administrado | Totalmente administrado | Manual (tú gestionas) |
| **IA/ML integrado** | Básico | Avanzado (Azure AI) | Manual |
| **Precio** | $$$ | $$ | $ (pero + trabajo) |
| **Escalabilidad** | Buena | Excelente | Excelente (manual) |
| **Analizadores de idiomas** | Bueno | Excelente (50+ idiomas) | Configurable |

#### ✅ Pros Azure Cognitive Search
- **IA integrada**: OCR, análisis de sentimientos, extracción de entidades
- **Skillsets**: Pipelines de enriquecimiento de datos
- **Semantic ranking**: Búsqueda semántica con Azure OpenAI
- **Menos mantenimiento**: Sin gestión de índices

#### ❌ Contras Azure Cognitive Search
- **Reescritura**: Queries OpenSearch no compatibles
- **Features limitados**: No log analytics como OpenSearch
- **Vendor lock-in**: API propietaria

#### ⚠️ Riesgos de Migración
- **Crítico**: Reescritura completa de queries y aplicaciones
- **Alto**: Pérdida de dashboards Kibana
- **Medio**: Curva de aprendizaje del nuevo API

#### 💡 Recomendación
```
Si necesitas compatibilidad OpenSearch → Self-hosted en AKS
Si quieres búsqueda empresarial moderna → Azure Cognitive Search
Si usas características específicas OpenSearch → Mantener AWS o self-hosted
```

---

### 3. ECS → Azure Container Apps / AKS

**Costo AWS:** $1,182.61/mes

#### Opciones Azure

| Opción | Descripción | Mejor Para |
|--------|-------------|------------|
| **Azure Container Apps** | Serverless containers | Microservicios, APIs, workers |
| **Azure Kubernetes Service (AKS)** | Kubernetes administrado | Apps complejas, control total |
| **Azure Container Instances (ACI)** | Contenedores on-demand | Jobs, burst workloads |
| **Azure App Service** | PaaS tradicional | Apps web sencillas |

#### Comparativa Detallada

| Aspecto | AWS ECS | Azure Container Apps | Azure AKS |
|---------|---------|---------------------|-----------|
| **Complejidad** | Media | Baja | Alta |
| **Precio** | $$ | $ | $$ |
| **Escalado** | Service Auto Scaling | KEDA-based (event-driven) | Kubernetes HPA/VPA |
| **Networking** | AWS VPC | VNet integration | VNet + Azure CNI |
| **Observabilidad** | CloudWatch | Application Insights | Prometheus + Grafana |
| **Vendor lock-in** | Alto | Medio | Bajo (Kubernetes estándar) |

#### ✅ Pros Azure Container Apps
- **Serverless**: Paga por segundo, escala a cero
- **KEDA nativo**: Escala basado en eventos (queues, HTTP, etc.)
- **Dapr integrado**: Service mesh, pub/sub, state management
- **Revisiones**: Gestión de versiones built-in

#### ❌ Contras Azure Container Apps
- **Limitaciones**: No control de nodos, menos flexible que AKS
- **Nuevo**: Servicio relativamente nuevo, menos maduro
- **Debugging**: Más difícil que en AKS

#### ⚠️ Riesgos de Migración
- **Medio**: Cambio en task definitions → Container Apps YAML
- **Bajo**: Networking similar VPC → VNet
- **Bajo**: ECR → Azure Container Registry (ACR)

#### 💡 Recomendación
```
Si usas ECS Fargate → Azure Container Apps (equivalente más cercano)
Si usas ECS EC2 o necesitas control → AKS
Si tienes arquitectura simple → Azure App Service (containers)
```

---

### 4. CloudWatch → Azure Monitor + Application Insights

**Costo AWS:** $882.77/mes

#### Opciones Azure

| Opción | Descripción | Mejor Para |
|--------|-------------|------------|
| **Azure Monitor** | Monitoreo de infraestructura | Métricas, logs, alertas |
| **Application Insights** | APM para aplicaciones | Tracing, performance, errores |
| **Log Analytics** | Análisis de logs | Query avanzado, dashboards |

#### Comparativa Detallada

| Aspecto | AWS CloudWatch | Azure Monitor | Ventaja |
|---------|----------------|---------------|---------|
| **Precio ingesta** | $0.50/GB | $2.76/GB (Pay-as-you-go) | AWS |
| **Retención** | 15 meses | 30-730 días configurable | Empate |
| **Query language** | CloudWatch Insights | KQL (Kusto) | Azure (más potente) |
| **Dashboards** | Básico | Avanzado (Workbooks) | Azure |
| **Alertas** | Bueno | Excelente (Smart Detection) | Azure |
| **Distributed tracing** | Con X-Ray | Nativo en App Insights | Azure |

#### ✅ Pros Azure Monitor
- **Unified platform**: Logs, métricas, traces en un solo lugar
- **KQL poderoso**: Lenguaje de query muy potente
- **Workbooks**: Dashboards interactivos avanzados
- **Azure Monitor Agent**: Single agent para todo
- **Integración AI**: Anomaly detection automático

#### ❌ Contras Azure Monitor
- **Costo**: Puede ser más caro que CloudWatch
- **Curva de aprendizaje**: KQL requiere aprendizaje
- **Retención**: Costo adicional por retención larga

#### ⚠️ Riesgos de Migración
- **Medio**: Migración de dashboards CloudWatch → Workbooks
- **Medio**: Re-instrumentar aplicaciones con SDK Azure
- **Bajo**: Alertas similares, fácil migración

#### 💡 Recomendación
```
Migración recomendada:
CloudWatch Logs → Log Analytics Workspace
CloudWatch Metrics → Azure Monitor Metrics
CloudWatch Dashboards → Azure Workbooks
CloudWatch Alarms → Azure Monitor Alerts
```

---

### 5. X-Ray → Application Insights

**Costo AWS:** $438.64/mes

#### Comparativa Detallada

| Aspecto | AWS X-Ray | Azure Application Insights | Ventaja |
|---------|-----------|----------------------------|---------|
| **Distributed tracing** | ✅ Excelente | ✅ Excelente | Empate |
| **Precio** | $5/millón traces | Incluido en Monitor | Azure |
| **Sampling** | Manual config | Adaptive automático | Azure |
| **Performance insights** | Básico | Avanzado (profiling) | Azure |
| **User analytics** | No | ✅ Sí | Azure |
| **Mapa de aplicación** | Service Graph | Application Map | Azure |

#### ✅ Pros Application Insights
- **Todo incluido**: Traces, logs, métricas en uno
- **Live Metrics**: Telemetría en tiempo real
- **Smart Detection**: Anomalías automáticas con ML
- **User tracking**: Analytics de usuarios finales
- **Snapshot debugger**: Debug en producción

#### ❌ Contras Application Insights
- **Vendor lock-in**: SDK propietario
- **Overhead**: Más datos recopilados = más costo

#### ⚠️ Riesgos de Migración
- **Medio**: Re-instrumentar código con SDK Azure
- **Bajo**: Conceptos similares (spans, traces)

---

### 6. Config → Azure Policy + Resource Graph

**Costo AWS:** $460.12/mes

#### Opciones Azure

| Opción | Descripción | Mejor Para |
|--------|-------------|------------|
| **Azure Policy** | Governance y compliance | Forzar estándares |
| **Azure Resource Graph** | Query de recursos | Inventario, auditoría |
| **Azure Blueprints** | Plantillas de infraestructura | Entornos repetibles |

#### Comparativa Detallada

| Aspecto | AWS Config | Azure Policy | Azure Resource Graph |
|---------|-----------|--------------|---------------------|
| **Compliance tracking** | ✅ | ✅ | Limitado |
| **Remediation automática** | Con SSM | ✅ DeployIfNotExists | No |
| **Query de recursos** | Config queries | No | ✅ KQL queries |
| **Precio** | $0.003/item | Incluido | Incluido |
| **Configuración histórica** | ✅ | Limitado | No |

#### ✅ Pros Azure Policy
- **Gratis**: Incluido sin costo adicional
- **Preventivo**: Bloquea recursos no conformes
- **Remediation**: Auto-corrige configuraciones
- **Initiatives**: Grupos de políticas (ej: ISO 27001)

#### ❌ Contras Azure Policy
- **No histórico detallado**: Como Config
- **Menos granular**: Config tiene más detalle

#### 💡 Recomendación
```
AWS Config Rules → Azure Policy definitions
Config Aggregator → Resource Graph queries
Remediation → Azure Policy DeployIfNotExists
```

---

### 7. VPC → Azure Virtual Network (VNet)

**Costo AWS:** $273.72/mes

#### Comparativa Detallada

| Aspecto | AWS VPC | Azure VNet | Diferencia Clave |
|---------|---------|-----------|-----------------|
| **Subnets** | Múltiples por AZ | Múltiples por región | Similar |
| **Routing** | Route tables | Route tables + UDR | Similar |
| **NAT** | NAT Gateway | NAT Gateway | Similar |
| **VPN** | VPN Gateway | VPN Gateway | Similar |
| **Peering** | VPC Peering | VNet Peering | Azure más flexible |
| **Precio NAT** | $0.045/hr + data | $0.045/hr + data | Igual |

#### ✅ Pros Azure VNet
- **Global VNet peering**: Sin limitaciones de región
- **Service Endpoints**: Más servicios soportados
- **Private Link**: Mejor integración PaaS

#### ❌ Contras Azure VNet
- **CIDR changes**: Más restrictivo en Azure
- **Subnet delegation**: Requerido para algunos servicios

#### ⚠️ Riesgos de Migración
- **Bajo**: Conceptos casi idénticos
- **Medio**: Rediseñar CIDR si hay conflictos

---

### 8. AppSync → Azure Functions + GraphQL

**Costo AWS:** $165.25/mes

Ya cubierto en detalle anteriormente. Ver sección de AppSync.

#### 💡 Recomendación Rápida
```
AppSync → Azure API Management (GraphQL support)
Alternativa: Azure Functions + Apollo Server + SignalR
```

---

### 9. CloudTrail → Azure Activity Log + Monitor Logs

**Costo AWS:** $183.51/mes

#### Comparativa Detallada

| Aspecto | AWS CloudTrail | Azure Activity Log | Azure Monitor Logs |
|---------|----------------|-------------------|-------------------|
| **Auditoría de API calls** | ✅ | ✅ | ✅ |
| **Retención por defecto** | 90 días | 90 días | 30 días |
| **Precio almacenamiento** | S3 storage | Gratis (90 días) | $2.76/GB adicional |
| **Query** | Athena | Portal + KQL | KQL avanzado |
| **Compliance** | Bueno | Excelente | Excelente |

#### ✅ Pros Azure
- **Gratis**: 90 días incluidos sin costo
- **Unified logs**: Todo en Log Analytics
- **Microsoft Sentinel**: SIEM nativo

#### 💡 Recomendación
```
CloudTrail → Activity Log (management events)
CloudTrail Data Events → Diagnostic Settings (cada recurso)
CloudTrail Insights → Azure Monitor insights
```

---

### 10. EC2 Instances → Azure Virtual Machines

**Costo AWS:** $168.16/mes

#### Comparativa Detallada

| Aspecto | AWS EC2 | Azure VMs | Ventaja |
|---------|---------|-----------|---------|
| **Tipos de instancia** | 400+ | 300+ | AWS |
| **Reserved Instances** | 1-3 años, 75% descuento | 1-3 años, 72% descuento | Similar |
| **Spot Instances** | Hasta 90% descuento | Hasta 90% descuento | Similar |
| **Burstable** | T-series | B-series | Similar |
| **Hibernación** | ✅ | ✅ (limitado) | AWS |
| **Live migration** | No | ✅ | Azure |

#### ✅ Pros Azure VMs
- **Azure Hybrid Benefit**: Reutiliza licencias Windows
- **Ultra Disk**: Discos de ultra performance
- **VM Scale Sets**: Auto-scaling integrado
- **Availability Zones**: Gratis (AWS cobra data transfer)

#### ❌ Contras Azure VMs
- **Menos tipos**: Menos opciones de instancia
- **Regiones**: Menos regiones que AWS

#### 💡 Recomendación
```
EC2 T-series → Azure B-series (burstable)
EC2 M-series → Azure D-series (general purpose)
EC2 C-series → Azure F-series (compute optimized)
EC2 R-series → Azure E-series (memory optimized)
EC2 Reserved → Azure Reserved VM Instances
```

---

### 11. EC2-Other (EBS, Snapshots) → Azure Managed Disks

**Costo AWS:** $137.10/mes

#### Comparativa Detallada

| Aspecto | AWS EBS | Azure Managed Disks | Ventaja |
|---------|---------|---------------------|---------|
| **Tipos** | gp3, io2, st1, sc1 | Premium SSD, Standard SSD, Standard HDD, Ultra Disk | Azure (Ultra) |
| **Max IOPS** | 64,000 (io2) | 160,000 (Ultra) | Azure |
| **Max throughput** | 1,000 MB/s | 2,000 MB/s | Azure |
| **Snapshots** | Incremental | Incremental | Empate |
| **Encriptación** | Default | Default | Empate |
| **Precio gp3** | $0.08/GB | $0.12/GB (Premium SSD) | AWS |

#### ✅ Pros Azure Managed Disks
- **Ultra Disk**: Performance extremo
- **No límite de IOPS**: En Ultra Disk
- **Disk Reservation**: Ahorro hasta 40%
- **Shared disks**: Para clusters

#### ❌ Contras Azure Managed Disks
- **Precio Standard SSD**: Más caro que gp3
- **Menos tipos**: Menos opciones que EBS

---

### 12. Security Hub → Microsoft Defender for Cloud

**Costo AWS:** $138.85/mes

#### Comparativa Detallada

| Aspecto | AWS Security Hub | Microsoft Defender for Cloud | Ventaja |
|---------|------------------|----------------------------|---------|
| **Precio** | $0.0010/check | $15/server/mes | Variable |
| **Compliance frameworks** | 10+ | 15+ | Azure |
| **Threat detection** | Con GuardDuty | Integrado | Azure |
| **CSPM** | ✅ | ✅ | Empate |
| **Multi-cloud** | No | ✅ (AWS, GCP, on-prem) | Azure |
| **Vulnerability scanning** | Con Inspector | Integrado | Azure |

#### ✅ Pros Microsoft Defender
- **Todo incluido**: Security Center + Sentinel + Defender
- **Multi-cloud**: Gestiona AWS y GCP
- **Threat Intelligence**: Microsoft's global threat data
- **JIT access**: Just-in-time VM access
- **Regulatory compliance**: Dashboards automáticos

#### ❌ Contras
- **Costo**: Puede ser más caro que Security Hub
- **Complejidad**: Muchos componentes

#### 💡 Recomendación
```
Security Hub → Microsoft Defender for Cloud (CSPM)
GuardDuty → Microsoft Defender for Cloud (Threat Protection)
Inspector → Microsoft Defender for Servers (Vulnerability Assessment)
Macie → Microsoft Purview (Data governance)
```

---

### 13. Kinesis → Azure Event Hubs

**Costo AWS:** $108.00/mes

#### Comparativa Detallada

| Aspecto | AWS Kinesis | Azure Event Hubs | Ventaja |
|---------|-------------|------------------|---------|
| **Throughput** | 1 MB/s por shard | 1 MB/s por Throughput Unit | Similar |
| **Retención** | 1-365 días | 1-90 días | AWS |
| **Precio** | $0.015/hr/shard | $0.015/hr/TU | Similar |
| **Kafka compatible** | No | ✅ Sí | Azure |
| **Capture** | Kinesis Firehose | Event Hubs Capture | Similar |
| **Consumer lag** | CloudWatch | Built-in metrics | Azure |

#### ✅ Pros Azure Event Hubs
- **Kafka API**: Compatible con clientes Kafka
- **Auto-inflate**: Escala automáticamente TUs
- **Schema Registry**: Gestión de esquemas
- **Geo-disaster recovery**: Failover automático
- **Menor latencia**: Para casos de uso real-time

#### ❌ Contras Azure Event Hubs
- **Retención**: Máximo 90 días vs 365 de Kinesis
- **Ordenamiento**: Solo dentro de partición

#### 💡 Recomendación
```
Kinesis Data Streams → Event Hubs Standard/Premium
Kinesis Data Firehose → Event Hubs Capture + Azure Functions
Kinesis Data Analytics → Azure Stream Analytics
```

---

### 14. ElastiCache → Azure Cache for Redis

**Costo AWS:** $97.92/mes

#### Comparativa Detallada

| Aspecto | AWS ElastiCache | Azure Cache for Redis | Ventaja |
|---------|-----------------|---------------------|---------|
| **Engines** | Redis, Memcached | Redis | AWS |
| **Redis version** | 7.x | 7.x | Empate |
| **Persistence** | RDB, AOF | RDB, AOF | Empate |
| **Clustering** | ✅ | ✅ | Empate |
| **Geo-replication** | Global Datastore | Geo-replication | Empate |
| **Precio** | $0.034/hr (cache.t3.medium) | $0.04/hr (C1) | AWS |
| **Tiers** | 3 tiers | 5 tiers | Azure |

#### ✅ Pros Azure Cache for Redis
- **Enterprise tier**: Redis Enterprise integrado
- **Active geo-replication**: Writes en múltiples regiones
- **Zone redundancy**: Alta disponibilidad
- **RediSearch**: Búsqueda full-text
- **RedisBloom**: Probabilistic data structures

#### ❌ Contras Azure Cache for Redis
- **No Memcached**: Solo Redis
- **Precio Enterprise**: Más caro que ElastiCache

#### ⚠️ Riesgos de Migración
- **Bajo**: Dump & restore de Redis
- **Medio**: Si usas Memcached (no existe en Azure)

#### 💡 Recomendación
```
ElastiCache Redis → Azure Cache for Redis (Standard/Premium)
ElastiCache Memcached → Azure Cache for Redis + ajustar aplicación
Para alta disponibilidad → Premium tier con zone redundancy
```

---

### 15. Elastic Load Balancing → Azure Load Balancer

**Costo AWS:** $107.32/mes

#### Opciones Azure

| Opción | Tipo | Mejor Para |
|--------|------|------------|
| **Azure Load Balancer** | Layer 4 (TCP/UDP) | VMs, tráfico interno |
| **Azure Application Gateway** | Layer 7 (HTTP/HTTPS) | Apps web, WAF integrado |
| **Azure Front Door** | Global Layer 7 | CDN + load balancing global |
| **Azure Traffic Manager** | DNS-based | Multi-región, geo-routing |

#### Comparativa Detallada

| Aspecto | AWS ALB/NLB | Azure App Gateway | Azure Load Balancer |
|---------|-------------|------------------|-------------------|
| **Layer** | 7 (ALB) / 4 (NLB) | 7 | 4 |
| **SSL termination** | ✅ | ✅ | No |
| **WAF** | Con AWS WAF | Integrado | No |
| **Path-based routing** | ✅ | ✅ | No |
| **WebSocket** | ✅ | ✅ | ✅ |
| **Precio** | $0.0225/hr + LCU | $0.125/hr + CU | $0.005/hr |

#### ✅ Pros Azure
- **Application Gateway**: WAF integrado sin costo extra
- **Load Balancer Basic**: Gratis para uso básico
- **Front Door**: CDN + LB + WAF global
- **Health probes**: Más flexibles

#### ❌ Contras Azure
- **Application Gateway**: Más caro que ALB
- **Menos mature**: ALB tiene más features

#### 💡 Recomendación
```
AWS ALB (Application Load Balancer) → Azure Application Gateway v2
AWS NLB (Network Load Balancer) → Azure Load Balancer (Standard)
AWS ELB Classic → Azure Load Balancer (Basic)
Global apps → Azure Front Door
```

---

### 16. WAF → Azure Web Application Firewall

**Costo AWS:** $88.52/mes

#### Comparativa Detallada

| Aspecto | AWS WAF | Azure WAF | Ventaja |
|---------|---------|-----------|---------|
| **Deployment** | CloudFront, ALB, API Gateway | App Gateway, Front Door, CDN | Similar |
| **Managed rules** | AWS Managed Rules | Azure Managed Rules (OWASP) | Similar |
| **Precio base** | $5/mes | Incluido con App Gateway | Azure |
| **Rules** | $1/rule | $1/rule | Empate |
| **Bot protection** | $10/mes | $200/mes (Premium) | AWS |
| **Custom rules** | ✅ | ✅ | Empate |
| **Geo-blocking** | ✅ | ✅ | Empate |

#### ✅ Pros Azure WAF
- **Incluido**: Con Application Gateway (sin costo extra base)
- **OWASP Top 10**: Protección automática
- **Custom rules**: Rate limiting, geo-filtering
- **Bot protection**: En Front Door Premium

#### ❌ Contras Azure WAF
- **Bot protection**: Solo en Front Door Premium (caro)
- **Menos granular**: Que AWS WAF v2

#### ⚠️ Riesgos de Migración
- **Medio**: Convertir reglas AWS WAF → Azure WAF
- **Bajo**: Conceptos similares

#### 💡 Recomendación
```
AWS WAF (on ALB) → Azure WAF (on Application Gateway)
AWS WAF (on CloudFront) → Azure WAF (on Front Door)
AWS Shield → Azure DDoS Protection
```

---

### 17. Managed Streaming for Kafka → Azure Event Hubs for Kafka

**Costo AWS:** $89.83/mes

#### Comparativa Detallada

| Aspecto | AWS MSK | Azure Event Hubs (Kafka) | Azure HDInsight Kafka |
|---------|---------|--------------------------|----------------------|
| **Kafka version** | 3.x | Compatible 1.0+ | 2.x / 3.x |
| **Gestión** | Administrado | Totalmente administrado | Semi-administrado |
| **Precio** | $0.21/hr (kafka.t3.small) | $0.015/hr/TU | $0.27/hr/node |
| **Zookeeper** | AWS gestiona | No necesario | Incluido |
| **Kafka Connect** | Manual setup | Azure Functions | Incluido |
| **Schema Registry** | Glue Schema Registry | Event Hubs Schema Registry | Manual |
| **Compatibilidad** | 100% Kafka | 95% Kafka | 100% Kafka |

#### ✅ Pros Azure Event Hubs (Kafka)
- **Serverless**: Sin gestión de brokers
- **Compatible**: Drop-in replacement para clientes Kafka
- **Menor costo**: Para workloads pequeños/medios
- **Integración**: Nativa con Azure services
- **Auto-scaling**: Escala automáticamente

#### ❌ Contras Azure Event Hubs (Kafka)
- **Limitaciones**: No 100% compatible (algunas APIs faltan)
- **No Kafka Streams**: Necesitas Stream Analytics
- **Consumer groups**: Límite de 20 por namespace

#### ✅ Pros HDInsight Kafka
- **100% compatible**: Kafka nativo
- **Control total**: Sobre brokers y configuración
- **Kafka Streams**: Soportado completamente

#### ❌ Contras HDInsight Kafka
- **Gestión**: Tú gestionas el cluster
- **Más caro**: Para workloads pequeños
- **Complejidad**: Requiere más conocimiento

#### 💡 Recomendación
```
Si tu aplicación usa APIs básicas Kafka → Event Hubs for Kafka
Si necesitas 100% compatibilidad → HDInsight Kafka
Si buscas menos gestión → Event Hubs for Kafka
Si usas Kafka Streams o Connect → HDInsight Kafka
```

---

## Estrategia de Migración

### Fase 1: Servicios de Bajo Riesgo (Semanas 1-4)
```
✅ Prioridad Alta - Bajo Riesgo:
1. VPC → VNet (networking base)
2. EC2-Other → Managed Disks + Backup
3. CloudTrail → Activity Log
4. CloudWatch Logs → Log Analytics
5. ElastiCache → Azure Cache for Redis
```

### Fase 2: Servicios de Medio Riesgo (Semanas 5-12)
```
⚠️ Prioridad Alta - Riesgo Medio:
1. ECS → Container Apps / AKS (requiere testing)
2. RDS → Azure Database (usar Azure DMS)
3. EC2 Instances → Azure VMs (lift & shift)
4. Kinesis → Event Hubs
5. X-Ray → Application Insights (re-instrumentar)
6. ELB → Application Gateway / Load Balancer
7. WAF → Azure WAF
8. MSK → Event Hubs for Kafka
```

### Fase 3: Servicios de Alto Riesgo (Semanas 13-20)
```
🔴 Prioridad Media - Riesgo Alto:
1. OpenSearch → Cognitive Search (requiere reescritura)
2. AppSync → API Management + Functions (arquitectura nueva)
3. Config → Azure Policy (mapeo de reglas)
4. Security Hub → Defender for Cloud (configuración compleja)
```

### Enfoque Recomendado

#### 1. **Strangler Fig Pattern**
```
AWS ←→ Azure (ambos corriendo en paralelo)
    ↓
Migrar servicio por servicio
    ↓
Validar cada uno antes de siguiente
    ↓
Apagar AWS gradualmente
```

#### 2. **Tooling de Migración**

| Herramienta | Propósito | Servicio |
|-------------|-----------|----------|
| **Azure Migrate** | Assessment inicial | Todos |
| **Azure Database Migration Service** | Migración de BD | RDS → Azure SQL/PostgreSQL |
| **Azure Site Recovery** | Migración de VMs | EC2 → Azure VMs |
| **Azure Data Factory** | Migración de datos | S3 → Blob Storage |
| **Azure DevOps** | CI/CD migration | CodePipeline → Azure Pipelines |

#### 3. **Networking Híbrido**
```
AWS VPC ←→ VPN/ExpressRoute ←→ Azure VNet
```
Mantener conectividad durante migración.

---

## Costos Estimados

### Comparación de Costos Mensuales

| Servicio | AWS Actual | Azure Estimado | Diferencia | % |
|----------|-----------|----------------|------------|---|
| RDS | $2,113.24 | $1,800-2,000 | -$113-313 | -5-15% |
| OpenSearch | $1,115.49 | $800-1,500* | -$315 a +385 | -28 a +35% |
| ECS | $1,182.61 | $900-1,100 | -$82-282 | -7-24% |
| CloudWatch | $882.77 | $950-1,200 | +$67-317 | +8-36% |
| X-Ray | $438.64 | Incluido | -$438 | -100% |
| Config | $460.12 | Incluido | -$460 | -100% |
| VPC | $273.72 | $250-300 | -$23 a +26 | -8 a +10% |
| AppSync | $165.25 | $100-200 | -$65 a +35 | -39 a +21% |
| CloudTrail | $183.51 | Incluido (90d) | -$183 | -100% |
| EC2 Instances | $168.16 | $150-180 | -$18 a +12 | -11 a +7% |
| EC2-Other | $137.10 | $120-150 | -$17 a +13 | -12 a +9% |
| Security Hub | $138.85 | $200-400 | +$61-261 | +44-188% |
| Kinesis | $108.00 | $100-120 | -$8 a +12 | -7 a +11% |
| ElastiCache | $97.92 | $100-130 | +$2-32 | +2-33% |
| ELB | $107.32 | $90-120 | -$17 a +13 | -16 a +12% |
| WAF | $88.52 | $50-100 | -$38 a +12 | -43 a +14% |
| MSK | $89.83 | $80-110 | -$9 a +20 | -10 a +22% |
| **TOTAL** | **$8,199.19** | **$6,000-8,500** | **-$2,199 a +301** | **-27 a +4%** |

*Depende de opción: Cognitive Search vs self-hosted

### 💰 Oportunidades de Ahorro en Azure

#### 1. **Azure Reserved Instances (RI)**
- **Ahorro**: 40-72% en VMs, SQL, Cache
- **Compromiso**: 1 o 3 años
- **Estimado**: $1,500-2,000/mes de ahorro

#### 2. **Azure Hybrid Benefit**
- **Ahorro**: Hasta 85% en Windows VMs
- **Requisito**: Licencias Windows Server existentes
- **Estimado**: $300-500/mes de ahorro

#### 3. **Servicios Incluidos (Sin Costo Extra)**
- Activity Log (CloudTrail)
- Azure Policy (Config)
- Application Insights tracing (X-Ray)
- Basic Load Balancer
- **Estimado**: $1,000-1,500/mes de ahorro

#### 4. **Dev/Test Pricing**
- **Ahorro**: 40-55% en entornos no-prod
- **Estimado**: $500-800/mes de ahorro (si aplica)

#### 5. **Serverless / Consumption-Based**
- Container Apps escala a cero
- Azure Functions pay-per-execution
- **Estimado**: $300-600/mes de ahorro

### 📊 Escenarios de Costo

#### Escenario 1: Migración Optimizada (Mejor Caso)
```
Costo base:                      $6,000/mes
+ Reserved Instances savings:    -$1,800/mes
+ Hybrid Benefit:                -$400/mes
+ Servicios incluidos:           -$1,000/mes
+ Serverless optimization:       -$500/mes
────────────────────────────────────────────
COSTO TOTAL:                     $2,300/mes
AHORRO vs AWS:                   $5,899/mes (72%)
```

#### Escenario 2: Migración Estándar (Caso Realista)
```
Costo base:                      $7,000/mes
+ Reserved Instances (parcial):  -$1,000/mes
+ Servicios incluidos:           -$800/mes
────────────────────────────────────────────
COSTO TOTAL:                     $5,200/mes
AHORRO vs AWS:                   $2,999/mes (37%)
```

#### Escenario 3: Lift & Shift (Peor Caso)
```
Costo base sin optimizar:        $8,500/mes
+ Costos de migración:           +$500/mes (temporal)
────────────────────────────────────────────
COSTO TOTAL:                     $9,000/mes
SOBRECOSTO vs AWS:               +$801/mes (10%)
```

---

## Matriz de Decisión

### Criterios de Evaluación

| Servicio | Complejidad Migración | Riesgo | Ahorro Potencial | Prioridad | Acción Recomendada |
|----------|----------------------|--------|------------------|-----------|-------------------|
| RDS | Media | Medio | Media | 🔴 Alta | Migrar con Azure DMS |
| OpenSearch | Alta | Alto | Alta* | 🟡 Media | Evaluar Cognitive Search vs self-hosted |
| ECS | Media | Medio | Alta | 🔴 Alta | Migrar a Container Apps |
| CloudWatch | Baja | Bajo | Baja | 🔴 Alta | Migrar a Monitor |
| X-Ray | Media | Medio | Alta | 🟢 Baja | Incluido en App Insights |
| Config | Media | Medio | Alta | 🟢 Baja | Migrar a Policy |
| VPC | Baja | Bajo | Baja | 🔴 Alta | Crear VNet primero |
| AppSync | Alta | Alto | Baja | 🟡 Media | Rediseñar con API Management |
| CloudTrail | Baja | Bajo | Alta | 🟢 Baja | Activity Log automático |
| EC2 | Media | Bajo | Media | 🔴 Alta | Lift & shift a VMs |
| Security Hub | Media | Medio | Negativa | 🟡 Media | Evaluar Defender for Cloud |
| Kinesis | Media | Medio | Baja | 🟡 Media | Migrar a Event Hubs |
| ElastiCache | Baja | Bajo | Baja | 🟢 Baja | Dump & restore a Azure Cache |
| ELB | Baja | Bajo | Baja | 🔴 Alta | Migrar a App Gateway |
| WAF | Baja | Bajo | Media | 🟢 Baja | Incluido en App Gateway |
| MSK | Media | Medio | Baja | 🟡 Media | Event Hubs for Kafka |

---

## Checklist de Migración

### Pre-Migración
- [ ] Audit completo de recursos AWS
- [ ] Documentar arquitectura actual
- [ ] Identificar dependencias entre servicios
- [ ] Calcular costos Azure con Azure Pricing Calculator
- [ ] Setup Azure subscription y governance
- [ ] Crear VNet y networking base
- [ ] Configurar ExpressRoute o VPN site-to-site
- [ ] Setup Azure DevOps / GitHub Actions
- [ ] Capacitar equipo en servicios Azure

### Durante Migración
- [ ] Migrar por fases (ver estrategia arriba)
- [ ] Mantener ambientes en paralelo
- [ ] Testing exhaustivo en Azure
- [ ] Monitoreo dual (AWS + Azure)
- [ ] Documentar cambios y decisiones
- [ ] Backup completo antes de cada migración

### Post-Migración
- [ ] Validar todos los servicios en Azure
- [ ] Optimizar costos (RIs, Hybrid Benefit)
- [ ] Implementar Azure Monitor completo
- [ ] Setup alertas y dashboards
- [ ] Disaster recovery testing
- [ ] Documentación final
- [ ] Decommission AWS (gradual)
- [ ] Post-mortem y lessons learned

---

## Recursos y Referencias

### Documentación Oficial
- [Azure Migration Center](https://azure.microsoft.com/en-us/migration/)
- [AWS to Azure Services Comparison](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/)
- [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)
- [Azure Database Migration Service](https://azure.microsoft.com/en-us/products/database-migration/)

### Herramientas
- [Azure Migrate](https://azure.microsoft.com/en-us/products/azure-migrate/)
- [Azure TCO Calculator](https://azure.microsoft.com/en-us/pricing/tco/calculator/)
- [Azure Cost Management](https://azure.microsoft.com/en-us/products/cost-management/)

### Training
- [Microsoft Learn - Azure Fundamentals](https://learn.microsoft.com/en-us/training/paths/azure-fundamentals/)
- [AWS to Azure Migration Learning Path](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/)

---

## Conclusiones y Recomendaciones Finales

### ✅ Ventajas de Migrar a Azure
1. **Ahorro de costos**: 25-40% con optimización adecuada
2. **Servicios incluidos**: Policy, Activity Log, App Insights tracing sin costo extra
3. **Integración Microsoft**: Si usas Office 365, AD, GitHub
4. **Hybrid capabilities**: Mejor integración on-premises con Azure Arc
5. **Enterprise support**: Mejores programas de soporte enterprise

### ⚠️ Consideraciones Importantes
1. **Curva de aprendizaje**: El equipo necesita training en Azure
2. **Tiempo de migración**: 4-6 meses para migración completa
3. **Riesgo OpenSearch**: Mayor complejidad de migración
4. **Vendor lock-in**: Cambiar de cloud provider siempre tiene riesgos

### 🎯 Recomendación Final

**Proceder con migración a Azure SI:**
- Buscas reducción de costos (25-40%)
- Tu equipo puede invertir tiempo en learning
- Tienes 4-6 meses para migración gradual
- Usas servicios Microsoft (Office 365, GitHub, AD)

**Considerar mantener AWS SI:**
- Equipo muy especializado en AWS
- Usas servicios AWS muy específicos
- No tienes tiempo para migración
- Costo actual es aceptable

### 📞 Siguiente Paso Recomendado

1. **Azure Assessment Workshop**: Workshop de 2-4 semanas con Azure para:
   - Architecture review
   - Cost estimation detallado
   - Migration roadmap
   - POC de servicio crítico

2. **POC Recomendado**: Migrar primero un servicio no crítico
   - Sugerencia: Empezar con CloudWatch → Azure Monitor
   - Validar el proceso de migración
   - Medir tiempos y dificultades reales

---

**Última actualización**: Octubre 2025  
**Versión**: 1.0  
**Autor**: Guía de Migración AWS → Azure

---

## Apéndice: Scripts y Comandos Útiles

### Azure CLI - Setup Inicial
```bash
# Login a Azure
az login

# Crear resource group
az group create --name migration-rg --location eastus

# Crear VNet
az network vnet create \
  --name migration-vnet \
  --resource-group migration-rg \
  --address-prefix 10.0.0.0/16 \
  --subnet-name default \
  --subnet-prefix 10.0.1.0/24
```

### Azure PowerShell - Cost Analysis
```powershell
# Get cost and usage
Get-AzConsumptionUsageDetail -StartDate 2025-09-01 -EndDate 2025-09-30 | 
  Group-Object InstanceName | 
  Select-Object Name, @{Name="Cost";Expression={($_.Group | Measure-Object -Property PretaxCost -Sum).Sum}} |
  Sort-Object Cost -Descending
```

### Terraform - Multi-Cloud State
```hcl
# Mantener estado de AWS y Azure durante migración
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# AWS Resources
provider "aws" {
  region = "us-east-1"
}

# Azure Resources
provider "azurerm" {
  features {}
}
```

---

*Este documento es una guía de referencia. Los precios y características pueden variar. Consulta la documentación oficial de Azure para información actualizada.*
