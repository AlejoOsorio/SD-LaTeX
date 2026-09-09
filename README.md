# 🔐 Implementación de un Servicio de Directorio para el Grupo de Investigación GRID

## 📖 Descripción

Este proyecto corresponde a una **tesis de pregrado** cuyo propósito es **implementar un servicio de directorio para la infraestructura del Grupo de Investigación en Redes, Información y Distribución (GRID)** de la **Universidad del Quindío**.

La propuesta busca centralizar la gestión de identidades y accesos (IAM) del GRID mediante una solución de **software libre y código abierto**, permitiendo que los usuarios utilicen **una sola cuenta** para autenticarse en los servicios del grupo. La solución integra **FreeIPA** como componente central —que combina el directorio **389 Directory Server**, la autenticación **Kerberos** y la autoridad certificadora **Dogtag CA**— junto con un servidor réplica y un balanceador **HAProxy** para garantizar redundancia y continuidad del servicio.

El servicio de directorio se integró con los servicios existentes del GRID —**OpenVPN**, **OpenProject** y **Xen Orchestra**— validándose mediante una batería de pruebas que confirman la centralización de identidades, la autorización diferenciada por grupos y la disponibilidad ante fallas del servidor principal.

---

## 🎯 Objetivos

### Objetivo general

Implementar un servicio de directorio para la infraestructura del grupo de investigación GRID de la Universidad del Quindío.

### Objetivos específicos

- Identificar la necesidad funcional y técnica del GRID con relación a un servicio de directorio.
- Caracterizar alternativas de software libre/código abierto para la implementación de un servicio de directorio en la infraestructura del grupo de investigación GRID.
- Seleccionar una alternativa de software libre/código abierto para la implementación de un servicio de directorio en la infraestructura del grupo de investigación GRID.
- Diseñar el servicio de directorio para el GRID según la arquitectura de software seleccionada.
- Implementar prototipo funcional del servicio de directorio para el GRID.
- Validar prototipo funcional del servicio de directorio implementado para el GRID.

---

## 📚 Marco Conceptual

### Gestión de Identidades y Accesos (IAM)

Conjunto de políticas, procesos y tecnologías orientadas a gestionar el ciclo de vida de las identidades digitales y a controlar el acceso a los recursos de una organización.

### Autenticación

Proceso mediante el cual un sistema verifica la identidad de un usuario o entidad antes de permitir el acceso a sus recursos.

### Autorización

Proceso de determinación de los permisos y recursos a los que un usuario ya autenticado tiene derecho de acceso, considerando el principio de *mínimo privilegio*.

### Servicios de Directorio

Sistemas centralizados que almacenan, organizan y presentan información sobre usuarios, grupos y recursos de una organización, facilitando su administración y consulta.

### LDAP

Protocolo abierto e independiente del proveedor para acceder a servicios de directorio distribuidos en redes TCP/IP, estructurados mediante un Árbol de Información de Directorio (DIT).

### Autenticación Federada

Mecanismo que permite a los usuarios acceder a múltiples sistemas con una única identidad, delegando la verificación de credenciales a un proveedor de identidad.

### Software Libre y Código Abierto

Software cuyo código fuente puede ser utilizado, estudiado, modificado y redistribuido libremente, sin depender de licenciamiento comercial.

---

## 🧪 Metodología

La metodología se desarrolló en fases sucesivas, siguiendo un enfoque sistemático:

1. **Caracterización del GRID:** análisis de stakeholders, misión, líneas de trabajo y necesidades del grupo mediante una entrevista estructurada, consolidados en términos NPO (Necesidades, Problemas y Oportunidades).
2. **Estudio de Mapeo Sistemático (SMS):** revisión de literatura con modelo GQM y PICOC sobre 5 bases de datos (Web of Science, IEEE Xplore, Springer, ScienceDirect, Taylor & Francis), depurada hasta un conjunto final de **22 estudios**.
3. **Análisis DAR (Decision Analysis and Resolution):** evaluación de 5 alternativas de código abierto (OpenLDAP, 389 Directory Server, ApacheDS, OpenDJ, FreeIPA) mediante 13 criterios ponderados, resultando **FreeIPA** como la alternativa seleccionada.
4. **Diseño de la solución:** modelado arquitectónico con **ArchiMate** y especificación de configuraciones (política de contraseñas, modelo de identidades, máquinas virtuales).
5. **Implementación del prototipo:** despliegue de FreeIPA con servidor réplica y balanceador HAProxy, e integración de los servicios del GRID por LDAP.
6. **Validación:** ejecución de 9 casos de prueba frente a los requisitos funcionales R-01 a R-08.

---

## ⚙️ Tecnologías Implementadas

Tecnologías que el proyecto desplegó directamente:

- **FreeIPA:** servicio de directorio central, implementado sobre los servidores Fedora Server 44 (`ipa1` principal y `ipa2` réplica), que integra nativamente:
  - **389 Directory Server** — repositorio LDAP de identidades, almacenado en la base LMDB.
  - **Kerberos (KDC)** — autenticación mediante tickets para el reino `GRID.UNIQUINDIO.EDU.CO`.
  - **Dogtag (CA)** — emisión y administración de certificados digitales.
  - **Policy Engine y Web UI/API** — motor de autorización e interfaz de administración.
- **HAProxy:** balanceador y proxy inverso (Debian 12) que concentra el acceso LDAP/LDAPS (puertos 389/636) con *roundrobin* y *health checks*.
- **Fedora Server 44 / Debian 12:** sistemas operativos instalados y configurados en los nodos del entorno de pruebas.

---

## 🏗️ Arquitectura de la Solución

### Componentes Principales

La solución se compone de un **dominio de autenticación** centralizado y un **dominio de aplicaciones consumidoras**:

- **Servidor principal `ipa1`** (`ipa1.grid.uniquindio.edu.co`, 172.30.30.61): nodo maestro de la identidad del GRID.
- **Servidor réplica `ipa2`** (`ipa2.grid.uniquindio.edu.co`, 172.30.30.62): copia sincronizada mediante **replicación Multi-Master** que asegura la continuidad del servicio.
- **HAProxy** (172.30.30.66): punto de acceso común `ipa.grid.uniquindio.edu.co` para las aplicaciones, balanceando las consultas LDAP/LDAPS entre `ipa1` e `ipa2`.
- **Servicios integrados:** OpenVPN, OpenProject y Xen Orchestra autenticados y autorizados vía LDAP contra FreeIPA.

### Modelo de Autorización

- **Grupos de autorización** `openvpn`, `openproject` y `xenorchestra` determinan a qué servicio accede cada usuario.
- **Cuentas de servicio** (`svc-openvpn`, `svc-openproject`, `svc-xenorchestra`) con política de contraseñas dedicada para las consultas LDAP de las aplicaciones.
- **Dominio:** `grid.uniquindio.edu.co` — DIT `dc=grid,dc=uniquindio,dc=edu,dc=co`.

### Flujo de Gestión

- **Gestión de usuarios:** solicitud → validación → procesamiento → asignación de grupos y permisos → aprovisionamiento.
- **Gestión de accesos:** inicio de sesión → validación de credenciales → verificación de permisos → acceso o rechazo.

---

## 🚀 Impacto Esperado

### Para el Grupo GRID

- Centralización de la gestión de identidades y accesos de la infraestructura del grupo.
- Control del ciclo de vida de las cuentas y de la autorización diferenciada por servicio.
- Redundancia y continuidad del servicio de autenticación ante fallas del servidor principal.
- Base para incorporar nuevos servicios al directorio como punto central de identidades.

### Para la Comunidad Académica

- Escenario práctico de despliegue de un servicio de directorio con software libre en producción.
- Referente de solución IAM aplicable a laboratorios y entornos de investigación.
- Soporte para la formación en administración de identidades, LDAP y seguridad.

---

## 📊 Beneficios de la Solución

- **Centralización:** una única fuente de verdad para identidades, grupos y políticas.
- **Redundancia:** replicación Multi-Master y balanceo HAProxy que garantizan disponibilidad.
- **Seguridad:** política de contraseñas (longitud mínima de 12 caracteres, complejidad y bloqueo por intentos) y cifrado TLS sobre LDAPS.
- **Cumplimiento:** alineación con controles de ISO/IEC 27002:2022 (controles 5.17 y 8.5).
- **Trazabilidad:** registro y auditoría de actividades de acceso.
- **Interoperabilidad:** integración LDAP/Kerberos compatible con los servicios existentes del GRID.
- **Orientación al código abierto:** eliminación de dependencias de licenciamiento propietario.
