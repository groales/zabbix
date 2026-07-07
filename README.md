# Zabbix

Zabbix es una plataforma de monitorización empresarial open-source que permite supervisar servidores, redes, aplicaciones y servicios desde una interfaz web centralizada.

Referencia oficial de instalación: https://www.zabbix.com/documentation/current/en/manual/installation/containers

## Características

- Monitorización de disponibilidad, rendimiento y métricas de red.
- Alertas y notificaciones configurables por múltiples canales.
- Gráficas, dashboards y reportes detallados.
- Agentes nativos y soporte SNMP, IPMI, JMX, HTTP.
- Auto-discovery y monitorización sin agente.

## Requisitos Previos

- Docker Engine instalado.
- Docker Compose instalado.
- Red Docker externa `proxy` creada para el proxy inverso.

## Archivos de este Repositorio

- `compose.yaml` - Definición de los servicios Zabbix (servidor, frontend web y base de datos).
- `.env.example` - Variables de entorno necesarias. Copiar a `.env` antes del despliegue.
- `README.md` - Esta documentación.

---

## Despliegue con Docker Compose

### 1. Clonar el repositorio

```bash
git clone https://github.com/groales/zabbix.git
cd zabbix
```

### 2. Crear el archivo `.env`

```bash
cp .env.example .env
```

Editar `.env` y establecer contraseñas seguras:

```env
POSTGRES_DB=zabbix
POSTGRES_USER=zabbix
POSTGRES_PASSWORD=tu_password_seguro
PHP_TZ=Europe/Madrid
```


### 3. Crear la red y levantar el servicio

```bash
docker network create proxy
docker compose up -d
```

---

## Método Alternativo: Crear Manualmente

Puedes copiar el `compose.yaml` y `.env.example` en una carpeta nueva, crear tu `.env` y ejecutar el mismo despliegue.

---

## Acceso Inicial

- Con proxy inverso: accede al dominio configurado apuntando al contenedor `zabbix-web` puerto `8080`.

Credenciales por defecto:
- Usuario: `Admin`
- Contraseña: `zabbix`

> **Importante:** Cambia la contraseña del administrador en el primer acceso.

## Puertos

| Puerto | Servicio         | Descripción                               |
|--------|------------------|-------------------------------------------|
| 10051  | Zabbix Server    | Comunicación con agentes Zabbix (público) |
| 8080   | Zabbix Web       | Interfaz web (solo expuesto a la red proxy) |

## Comandos Útiles

```bash
docker compose ps
docker compose logs -f zabbix-server
docker compose logs -f zabbix-web
docker compose logs -f zabbix-agent2
docker compose logs -f postgres
docker compose restart zabbix-server
docker compose pull
docker compose up -d
docker compose down
```

## Estructura de Volúmenes

```text
Volúmenes nombrados:
├── zabbix_pgdata    -> /var/lib/postgresql       (datos de PostgreSQL)
├── zabbix_snmptraps -> /var/lib/zabbix/snmptraps (traps SNMP)
└── zabbix_export    -> /var/lib/zabbix/export    (exportaciones de configuración)
```

## Configuración Avanzada

Para Nginx Proxy Manager (o equivalente):

- Forward Hostname/IP: `zabbix-web`
- Forward Port: `8080`
- SSL: certificado válido y Force SSL habilitado

El puerto `8080` está declarado con `expose` (no con `ports`), por lo que solo es accesible desde la red `proxy` y no se publica directamente en el host.

## Solución de Problemas

- **Zabbix Server no conecta a la BD**: verificar que `postgres` esté healthy antes de que arranque el servidor. El `depends_on` con `condition: service_healthy` lo gestiona automáticamente.
- **Frontend muestra "Zabbix server is not running"**: el servidor puede tardar hasta 60 segundos en inicializarse completamente.
- **Agentes no conectan**: asegúrate de que el puerto `10051` está accesible desde los hosts monitorizados.
