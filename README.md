# OpenClaw

OpenClaw es una plataforma de agentes de IA de código abierto que permite ejecutar y gestionar asistentes de IA de forma local o en la nube.

## Tabla de Contenidos
- [Introducción](#introducción)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Ejecutar OpenClaw](#ejecutar-openclaw)
- [Acceder a los Servicios](#acceder-a-los-servicios)
- [Detener y Limpiar](#detener-y-limpiar)
- [Actualizar OpenClaw](#actualizar-openclaw)
- [Backup y Restauración](#backup-y-restauración)
- [Solución de Problemas](#solución-de-problemas)
- [Recursos Adicionales](#recursos-adicionales)

## Introducción

Esta documentación proporciona instrucciones para instalar y configurar OpenClaw usando Docker Compose. OpenClaw consiste en varios servicios:
- **Traefik**: Proxy inverso, balanceador de carga y gestor de certificados TLS
- **OpenClaw Gateway**: Puerta de enlace principal de la API para la plataforma OpenClaw
- **OpenClaw CLI**: Interfaz de línea de comandos para interactuar con OpenClaw

## Requisitos Previos

Antes de comenzar, asegúrate de tener instalado lo siguiente:
- [Docker Engine](https://docs.docker.com/engine/install/) (versión 20.10 o superior)
- [Docker Compose](https://docs.docker.com/compose/install/) (versión 2.0 o superior)
- Git (para clonar el repositorio, si aplica)

## Instalación

1. **Clonar el repositorio** (si aún no lo has hecho):
   ```bash
   git clone https://github.com/openclaw/openclaw.git
   cd openclaw
   ```

2. **Configurar la red externa de Docker**:
   Puesto que la configuración utiliza una red externa para Traefik, debes crearla antes de iniciar:
   ```bash
   docker network create proxy
   ```

3. **Copiar el archivo de entorno de ejemplo** y configurarlo:
   ```bash
   cp .env.example .env
   ```

4. **Configurar el archivo `.env`**:
   - Establece `DOMAIN_NAME` con tu dominio deseado (ej. `openclaw.example.com`).
   - Establece `ACME_EMAIL` para los certificados de Let's Encrypt.
   - Establece `TRAEFIK_DASHBOARD_AUTH` para proteger el dashboard de Traefik (formato: `usuario:contraseña_hash`, genera con `htpasswd -nb usuario contraseña`).
   - Configura al menos una API key de un proveedor de modelos (ej. `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc.).
   - Ajusta otras variables según sea necesario (consulta el archivo `.env` para ver los comentarios).

## Configuración

### Variables de Entorno

El archivo `.env` contiene todas las variables de configuración. Aquí están las más importantes:

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `DOMAIN_NAME` | El nombre de dominio para acceder a OpenClaw | `openclaw.example.com` |
| `SUB_DOMAIN_NAME_GT` | Subdominio específico para el Gateway | `gw` |
| `TIMEZONE` | Zona horaria del servidor | `America/Bogota` |
| `OPENCLAW_IMAGE` | Imagen de Docker a utilizar | `ghcr.io/openclaw/openclaw:latest` |
| `OPENCLAW_TZ` | Zona horaria de OpenClaw (generalmente igual a TIMEZONE) | `America/Bogota` |
| `OPENCLAW_GATEWAY_TOKEN` | Token secreto para autenticación del gateway | `tu_token_secreto_aqui` |
| `ACME_EMAIL` | Email para certificados Let's Encrypt | `admin@example.com` |
| `TRAEFIK_DASHBOARD_AUTH` | Auth básica del dashboard de Traefik (usuario:hash) | `admin:$apr1$...` |
| `OPENAI_API_KEY` | API key de OpenAI (si usas modelos de OpenAI) | `sk-...` |
| `ANTHROPIC_API_KEY` | API key de Anthropic (si usas modelos de Claude) | `sk-ant-...` |
| `GEMINI_API_KEY` | API key de Google Gemini | `AI...` |
| `MINIMAX_API_KEY` | API key de MiniMax | `sk-cp-...` |

### Volúmenes y Datos Persistentes

OpenClaw usa volúmenes Docker para datos persistentes:
- `openclaw/` - Datos de la aplicación y configuración
- `openclaw/workspace/` - Espacio de trabajo del agente
- `openclaw-auth-profile-secrets/` - Secretos de autenticación
- `traefik/acme.json` - Certificados TLS

Estos directorios están montados desde el host, por lo que tus datos persisten entre reinicios de contenedores.

## Ejecutar OpenClaw

1. **Iniciar los servicios**:
   ```bash
   docker compose up -d
   ```
   Esto iniciará todos los servicios definidos en modo detached.

2. **Verificar el estado**:
   ```bash
   docker compose ps
   ```
   Deberías ver todos los servicios como "healthy" o "starting".

3. **Ver los logs**:
   ```bash
   docker compose logs -f
   ```
   O para un servicio específico:
   ```bash
   docker compose logs -f openclaw-gateway
   ```

## Acceder a los Servicios

Una vez que los servicios estén funcionando:
- **OpenClaw Gateway**: Accede vía `https://${DOMAIN_NAME}` (o `http://localhost:18789` para acceso local sin Traefik)
- **Dashboard de Traefik**: Accede vía `https://traefik.${DOMAIN_NAME}` (requiere que `TRAEFIK_DASHBOARD_AUTH` esté configurado)
- **OpenClaw CLI**: Puedes ejecutar comandos usando el servicio `openclaw-cli`:
  ```bash
  docker compose run --rm openclaw-cli tu-comando-aqui
  ```

## Detener y Limpiar

- **Detener todos los servicios**:
  ```bash
  docker compose down
  ```

- **Detener y eliminar volúmenes** (ADVERTENCIA: ¡Esto eliminará todos los datos persistentes!):
  ```bash
  docker compose down -v
  ```

## Actualizar OpenClaw

Para actualizar a la última versión:
```bash
docker compose pull   # Descargar últimas imágenes
docker compose up -d   # Recrear contenedores con nuevas imágenes
```

## Backup y Restauración

### Backup
Para hacer backup de tus datos de OpenClaw, simplemente copia los siguientes directorios:
- `openclaw/`
- `openclaw-auth-profile-secrets/`
- `traefik/acme.json`
- `.env` (contiene configuración pero también secretos - maneja con cuidado)

### Restauración
1. Detener OpenClaw: `docker compose down`
2. Restaurar los directorios mencionados a sus ubicaciones originales.
3. Iniciar OpenClaw: `docker compose up -d`

## Solución de Problemas

### Problemas Comunes

1. **Servicios no iniciando correctamente**:
   - Revisar logs: `docker compose logs -f <nombre_servicio>`
   - Verificar que las variables de entorno requeridas estén configuradas en `.env`
   - Confirmar disponibilidad de puertos (80, 443, 18789, 18790)

2. **Problemas de HTTPS/TLS**:
   - Verificar que `ACME_EMAIL` esté correctamente configurado en `.env`
   - Asegurar que los puertos 80 y 443 sean accesibles desde internet (para validación de Let's Encrypt)
   - Revisar logs de Traefik para errores de emisión de certificados

3. **Problemas de autenticación**:
   - Verificar que `OPENCLAW_GATEWAY_TOKEN` en `.env` y `openclaw/openclaw.json` coincidan
   - Comprobar que `OPENCLAW_GATEWAY_BIND` esté configurado apropiadamente (`loopback` para solo local, `lan` para acceso de red)

### Obtener Ayuda

Si encuentras problemas no cubiertos aquí:
- Revisar la documentación de OpenClaw: https://docs.openclaw.dev
- Visitar el repositorio de GitHub de OpenClaw: https://github.com/openclaw/openclaw
- Abrir un issue en GitHub con detalles de tu problema

## Recursos Adicionales

- [Repositorio de GitHub de OpenClaw](https://github.com/openclaw/openclaw)
- [Documentación de OpenClaw](https://docs.openclaw.dev)
- [Documentación de Docker](https://docs.docker.com/)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)

---
*Última actualización: 21 de Mayo de 2026*
