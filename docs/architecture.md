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

```text
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
