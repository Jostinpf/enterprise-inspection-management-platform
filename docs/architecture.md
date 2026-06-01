# Descripción General de la Arquitectura

Este documento detalla la arquitectura del sistema, el diseño de la base de datos relacional, la automatización del flujo de datos y los módulos estructurales clave de la **Enterprise Inspection Management Platform**.

---

## 1. Topología del Sistema y Stack Tecnológico

La plataforma implementa una arquitectura Serverless desacoplada utilizando el ecosistema de Google Workspace y el motor de AppSheet Core. Las responsabilidades principales se distribuyen en las siguientes capas:

| Capa | Componente | Implementación |
| :--- | :--- | :--- |
| **Frontend / Cliente** | Framework Nativo de AppSheet | Interfaz adaptativa (móvil/tablet), validación de datos en el lado del cliente, formateo dinámico de UI y almacenamiento en caché local. |
| **Backend / Base de Datos** | Google Sheets y Motores en la Nube | Esquema relacional estructurado, columnas virtuales de AppSheet para cálculos en tiempo real y filtros de seguridad a nivel de fila (RLS). |
| **Motor de Automatización** | Bots de Automatización de AppSheet | Pipeline asíncrono de síntesis de documentos, webhooks basados en eventos y sincronización de datos por disparadores. |
| **Almacenamiento y Recursos** | Google Drive Enterprise | Almacenamiento binario en la nube que actúa como el repositorio principal para los reportes PDF consolidados y las plantillas de estructura. |

---

## 2. Esquema Relacional y Modelado de Datos

La base de datos impone un diseño estructural jerárquico estricto de uno a muchos (One-to-Many). La entidad maestra de Inspección controla los metadatos y los estados de sincronización, enrutando los datos transaccionales específicos a cada submódulo de ingeniería.

    [Entidad Proyecto]
       │
       └───► [Registro Maestro de Inspección] (Control de Estado y Metadatos)
                │
                ├───► [Módulo de Inspección Eléctrica]
                ├───► [Módulo de Inspección de Aguas Negras]
                ├───► [Módulo de Inspección de Aguas Pluviales]
                ├───► [Módulo de Sistemas de Supresión de Incendios]
                ├───► [Módulo de Pruebas de Aislamiento de Acometida]
                └───► [Módulo de Inspección de Agua Potable]

---

## 3. Pipeline de Automatización y Generación de Reportes

El motor asíncrono de síntesis de documentos se activa inmediatamente después de que un inspector completa y confirma los datos estructurales desde el cliente móvil.

    [ Inspector en Campo ] ──(Envía Registro)──► [ Motor de Backend AppSheet ]
                                                           │
                                                   (Dispara el Bot)
                                                           ▼
    [ Almacenamiento Google Drive ] ◄──(Guarda PDF)── [ Síntesis de Documento PDF ]
                 │                                             ▲
         (Actualiza Ruta)                             (Puebla la Plantilla)
                 │                                             │
                 └─────────────────────────────────────────────┴── [ Plantilla Google Docs ]

1. **Ciclo de Vida de la Transacción:** El inspector finaliza el formulario móvil. El estado de la inspección cambia dinámicamente de `Pending` a `Completed` a través de columnas virtuales que evalúan la integridad del registro.
2. **Orquestación de Webhooks y Bots:** El Bot de Automatización intercepta la mutación del estado, bloqueando instantáneamente el conjunto de datos para mantener la auditabilidad de la información.
3. **Motor de Síntesis de Documentos:** El motor procesa una plantilla segura de Google Docs, inyectando de forma asíncrona las variables recopiladas y los datos de las mediciones de ingeniería.
4. **Archivado Físico:** El archivo `.pdf` finalizado se compila y se escribe dinámicamente en directorios jerárquicos dentro de la nube, estructurados de forma automatizada por Proyecto, Disciplina Técnica y Marca de Tiempo.

---

## 4. Módulos Funcionales Centrales

### Ingeniería Eléctrica y Pruebas de Aislamiento
* **Listas de Verificación de Tableros:** Aplica reglas estrictas de validación en campo para tableros de distribución, mediciones de corriente/voltaje, identificación de conductores, cumplimiento de torque de sujeción y normativas de seguridad ocupacional.
* **Aislamiento de Acometida Principal:** Submódulo especializado que rastrea pruebas de aislamiento eléctrico, datos de resistencia óhmica e inspección de la infraestructura de acometidas generales.

### Sistemas Hidráulicos y de Fluidos (Potable, Incendios, Pluvial y Sanitario)
* **Motor de Pruebas de Presión:** Monitoreo automatizado de los umbrales de presión estática inicial y final en las redes de tuberías.
* **Aprobaciones Algorítmicas:** Calcula la duración de las pruebas en tiempo real, ejecutando lógica de fondo para determinar de forma automática si cumple o rechaza según los márgenes de tolerancia técnica antes de habilitar el cierre del registro.

---

## 5. Seguridad, Gobernanza y Operaciones Offline

* **Control de Acceso Basado en Roles (RBAC):** Restringe las vistas de la interfaz, la visibilidad de los datos y los permisos de modificación según el perfil del usuario autenticado (Inspectores vs. Administradores/Auditores).
* **Reglas de Integridad de Datos:** Implementa restricciones personalizadas en la base de datos (reglas de unicidad en `Valid If`) que evitan colisiones de registros y garantizan la consistencia de las llaves primarias en el backend.
* **Independencia de Red (Modo Offline):** Soporta completamente el almacenamiento de datos en caché local en el dispositivo del inspector, permitiendo auditorías de infraestructura en entornos sin conectividad y ejecutando una sincronización atómica una vez que se restablece la red.

---

## 6. Roadmap y Mejoras Futuras

* **Integración de Dashboards de BI:** Vistas analíticas avanzadas integradas directamente en AppSheet para el seguimiento macroscópico de estadísticas de proyectos y estados de auditorías.
* **Reportes y Métricas de KPI:** Indicadores de rendimiento enfocados en los tiempos de respuesta de los inspectores y análisis de tendencias históricas en campo.
* **Análisis de Inspecciones Históricas:** Mapas consolidados de fallas recurrentes recopilados a partir de los puntos críticos de rechazo en las pruebas hidráulicas y eléctricas.
