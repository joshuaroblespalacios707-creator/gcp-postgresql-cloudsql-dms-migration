**🇲🇽 Español** | [🇬🇧 English](#-postgresql-to-cloud-sql-database-migration-with-gcp-dms)

# 🗄️ Migración de Base de Datos PostgreSQL a Cloud SQL con GCP Database Migration Service (DMS)

Documentación técnica de una migración en vivo de PostgreSQL desde una VM en Compute Engine hacia Cloud SQL, usando **Database Migration Service (DMS)** con replicación lógica CDC y **cero tiempo de inactividad**.

---

## 📌 Resumen Ejecutivo

Este informe documenta el procedimiento técnico completo ejecutado para migrar una base de datos PostgreSQL en vivo desde una máquina virtual en Compute Engine (`postgresql-vm`) hacia una instancia administrada en Cloud SQL (`postgresql-cloudsql`) en la región `us-central1`. La migración se realizó mediante **Database Migration Service (DMS)**, utilizando replicación lógica (CDC — *Continuous Data Capture*) a través de un emparejamiento de red **VPC Peering**. Se destaca la resolución de fallos críticos de conectividad IP e incompatibilidad de versiones, logrando reducir el tiempo de ejecución de 4 horas a 30 minutos.

---

## 📊 Resumen de Métricas

| Métrica | Valor | Nota |
|---|---|---|
| Tiempo de laboratorio | 30 minutos | Reducción del 87.5% comparado con el intento previo (4 hrs) |
| Motor origen | Compute Engine VM (`10.128.0.10:5432`) | PostgreSQL 15 + pglogical *(originalmente 17, degradado a 15 por compatibilidad — ver sección de troubleshooting)* |
| Motor destino | Cloud SQL (`postgresql-cloudsql`) | PostgreSQL 15 |
| Modo de replicación | CDC continua | Sin tiempo de inactividad (*zero downtime*) |
| Verificación CDC | ID 11: Aguascalientes Hub | Replicado en tiempo real |

---

## 📋 Desglose de Tareas del Laboratorio

### Tarea 1 — Preparación del Servidor Origen (VM)

Configuración del entorno en Compute Engine. Instalación del motor PostgreSQL 15 y el paquete `postgresql-15-pglogical`. Ajuste de parámetros clave en `postgresql.conf`:

```conf
wal_level = logical
max_replication_slots = 10
shared_preload_libraries = 'pglogical'
```

Creación del usuario de migración `migration_admin` con privilegios de replicación y la base de datos `orders` con la tabla `public.distribution_centers`.

---

### Tarea 2 — Aprovisionamiento de la Instancia Destino en Cloud SQL

Creación de la instancia administrada de Cloud SQL for PostgreSQL en la versión 15, dentro de la misma región (`us-central1`), configurando credenciales de acceso administrativo y habilitando conectividad privada VPC.

---

### Tarea 3 — Configuración del Trabajo de Migración en DMS

Creación del perfil de conexión `postgres-vm-profile2` apuntando a la IP interna de la VM, establecimiento del emparejamiento de red VPC Peering, y ejecución exitosa del análisis de prerrequisitos (*pre-flight test*).

---

### Tarea 4 — Replicación CDC y Verificación en Tiempo Real

Inicio de la transmisión continua de datos (CDC). Inserción manual de un nuevo registro en la VM origen (`ID 11: Aguascalientes Hub`) y confirmación de su sincronización inmediata en la base de datos destino en Cloud SQL.

---

## 🔧 Diagnóstico Técnico y Resolución de Problemas

### A. Corrección de IP Interna Dinámica

- **Problema:** el perfil de DMS fallaba al intentar conectar a `10.128.0.8`.
- **Diagnóstico:** al reinstalar la VM en `us-central1-a`, Google Cloud asignó dinámicamente la nueva IP interna `10.128.0.10`.
- **Solución:** corrección del perfil de conexión en DMS para apuntar a `10.128.0.10:5432`, restableciendo la comunicación.

### B. Incompatibilidad de Versión Mayor de PostgreSQL

- **Problema:** error en la prueba pre-flight: `"Incompatible database source and destination versions (Source: 17.11 vs Destination: 15.18)"`.
- **Diagnóstico:** la replicación lógica de PostgreSQL no permite transmitir datos desde una versión superior hacia una versión menor.
- **Solución:** purgado total de PostgreSQL 17 en la VM origen, instalación de PostgreSQL 15 + pglogical 15, e igualación exacta del motor origen con la instancia de Cloud SQL v15.

---

## ✅ Evidencia de Validación de Datos

```sql
-- Consulta ejecutada en el origen
SELECT * FROM public.distribution_centers ORDER BY id;
```

```
 id |                    name                     | latitude | longitude
----+---------------------------------------------+----------+-----------
  1 | Memphis TN                                  |  35.1174 |  -89.9711
  2 | Chicago IL                                  |  41.8369 |  -87.6847
  3 | Houston TX                                  |  29.7604 |  -95.3698
  4 | Los Angeles CA                              |    34.05 |   -118.25
  5 | New Orleans LA                              |    29.95 |  -90.0667
  6 | Port Authority of New York/New Jersey NY/NJ |   40.634 |  -73.7834
  7 | Philadelphia PA                             |    39.95 |  -75.1667
  8 | Mobile AL                                   |  30.6944 |  -88.0431
  9 | Charleston SC                               |  32.7833 |  -79.9333
 10 | Savannah GA                                 |  32.0167 |  -81.1167
 11 | Aguascalientes Hub                          |  21.8853 | -102.2916
(11 rows)
```

---

## 🎓 Conclusión

La práctica ha finalizado exitosamente. El aprendizaje técnico adquirido en el manejo de redes VPC, control de versiones en bases de datos relacionales y monitoreo de servicios de migración administrados permitió ejecutar el laboratorio en tiempo récord y con cero errores residuales.

---

## 📌 Notas

Este laboratorio forma parte de mi preparación continua para la certificación **Google Associate Cloud Engineer (ACE)**, con enfoque práctico en migración de bases de datos y replicación en tiempo real en GCP.

---
---

[⬆ Español version above](#️-migración-de-base-de-datos-postgresql-a-cloud-sql-con-gcp-database-migration-service-dms)

# 🗄️ PostgreSQL to Cloud SQL Database Migration with GCP DMS

Technical documentation of a live PostgreSQL migration from a Compute Engine VM to Cloud SQL, using **Database Migration Service (DMS)** with logical CDC replication and **zero downtime**.

---

## 📌 Executive Summary

This report documents the full technical workflow executed to perform a live PostgreSQL database migration from a Compute Engine Virtual Machine (`postgresql-vm`) to a managed Cloud SQL instance (`postgresql-cloudsql`) in the `us-central1` region. The migration was orchestrated via **Database Migration Service (DMS)**, leveraging Change Data Capture (CDC) continuous logical replication over **VPC Peering**. Critical troubleshooting was conducted regarding IP shifts and PostgreSQL major version compatibility, optimizing total execution time from 4 hours down to 30 minutes.

---

## 📊 Key Metrics Summary

| Metric | Value | Note |
|---|---|---|
| Lab duration | 30 minutes | 87.5% reduction compared to previous attempt (4 hrs) |
| Source engine | Compute Engine VM (`10.128.0.10:5432`) | PostgreSQL 15 + pglogical *(originally 17, downgraded to 15 for compatibility — see troubleshooting section)* |
| Target engine | Cloud SQL (`postgresql-cloudsql`) | PostgreSQL 15 |
| Replication mode | Continuous CDC | Zero downtime |
| Live CDC validation | ID 11: Aguascalientes Hub | Replicated in real time |

---

## 📋 Lab Tasks Breakdown

### Task 1 — Source Server Preparation (VM)

Source environment preparation on Compute Engine. Installation of PostgreSQL 15 and `postgresql-15-pglogical`. Configuration of key WAL parameters in `postgresql.conf`:

```conf
wal_level = logical
max_replication_slots = 10
shared_preload_libraries = 'pglogical'
```

Creation of `migration_admin` user with replication rights, and initializing the `orders` database with the `public.distribution_centers` table.

---

### Task 2 — Target Cloud SQL Instance Provisioning

Provisioning of the managed Cloud SQL for PostgreSQL v15 instance within the `us-central1` region, establishing root administrative credentials and configuring private network connectivity.

---

### Task 3 — DMS Migration Job Configuration

Creation of connection profile `postgres-vm-profile2` mapping to the VM's internal IP, configuring the VPC Peering network topology, and successfully executing the pre-flight connectivity test.

---

### Task 4 — CDC Replication & Live Verification

Launching the CDC continuous data stream. Manual insertion of a test record into the source VM (`ID 11: Aguascalientes Hub`) and confirming its instant streaming replication into the Cloud SQL destination.

---

## 🔧 Technical Troubleshooting

### A. Dynamic Internal IP Fix

- **Issue:** DMS connection profile failed while attempting to reach `10.128.0.8`.
- **Root cause:** upon recreating the VM instance in `us-central1-a`, GCP assigned a new dynamic internal IP address `10.128.0.10`.
- **Fix:** re-created the DMS connection profile mapping directly to `10.128.0.10:5432`, immediately restoring connectivity.

### B. PostgreSQL Major Version Alignment

- **Issue:** pre-flight test failure: `"Incompatible database source and destination versions (Source: 17.11 vs Destination: 15.18)"`.
- **Root cause:** native PostgreSQL logical replication does not support replicating from a higher major version down to a lower version.
- **Fix:** purged PostgreSQL 17 from the source VM, installed PostgreSQL 15 along with pglogical 15, achieving perfect version parity with Cloud SQL v15.

---

## ✅ Data Validation Evidence

```sql
-- Query executed on source DB
SELECT * FROM public.distribution_centers ORDER BY id;
```

```
 id |                    name                     | latitude | longitude
----+---------------------------------------------+----------+-----------
  1 | Memphis TN                                  |  35.1174 |  -89.9711
  2 | Chicago IL                                  |  41.8369 |  -87.6847
  3 | Houston TX                                  |  29.7604 |  -95.3698
  4 | Los Angeles CA                              |    34.05 |   -118.25
  5 | New Orleans LA                              |    29.95 |  -90.0667
  6 | Port Authority of New York/New Jersey NY/NJ |   40.634 |  -73.7834
  7 | Philadelphia PA                             |    39.95 |  -75.1667
  8 | Mobile AL                                   |  30.6944 |  -88.0431
  9 | Charleston SC                               |  32.7833 |  -79.9333
 10 | Savannah GA                                 |  32.0167 |  -81.1167
 11 | Aguascalientes Hub                          |  21.8853 | -102.2916
(11 rows)
```

---

## 🎓 Conclusion

The laboratory execution concluded successfully. The technical expertise acquired in VPC network troubleshooting, PostgreSQL engine version control, and managed GCP migration workflows enabled completing the lab in record time with zero residual issues.

---

## 📌 Notes

This lab is part of my ongoing preparation for the **Google Associate Cloud Engineer (ACE)** certification, with a hands-on focus on database migration and real-time replication in GCP.
