# 08 · Contenedores

> NGINX · Portainer · WordPress + Compose

[⬅️ Volver al índice](../README.md)

## 🎬 Playlists

- Práctica 01: `PENDIENTE`
- Práctica 02: `PENDIENTE`
- Práctica 03: `PENDIENTE`

> [!IMPORTANT]
> El enunciado original pide Docker. En RHEL 10, Red Hat indica que Docker Engine / `docker` no está soportado y recomienda Podman; el paquete `podman-docker` puede proporcionar compatibilidad de CLI para comandos Docker. citeturn447372search1

---

# 🧪 Práctica 01 — NGINX en contenedor

## 1. Instalar Podman

```bash
sudo dnf install -y podman
podman --version
```

RHEL 10 documenta `container-tools` como el meta-paquete para herramientas de contenedores y también permite instalar Podman individualmente. citeturn447372search1

## 2. Crear website persistente

```bash
mkdir -p ~/website
cat > ~/website/index.html <<'HTML'
<!doctype html>
<html lang="es">
<head><meta charset="utf-8"><title>RHEL 10 Containers</title></head>
<body>
  <h1><TU_NOMBRE></h1>
  <p>Matrícula: <TU_MATRICULA></p>
</body>
</html>
HTML
```

## 3. Descargar NGINX

```bash
podman pull docker.io/library/nginx:latest
```

## 4. Ejecutar

Equivalente nativo al objetivo Docker del laboratorio:

```bash
podman run -d \
  --name nginx-lab \
  -p 8888:80 \
  -v "$HOME/website:/usr/share/nginx/html:Z" \
  docker.io/library/nginx:latest
```

El volumen `:Z` es importante en hosts con SELinux para relabelizar el contenido según el aislamiento del contenedor.

## 5. Verificar

```bash
podman ps
curl http://127.0.0.1:8888
```

Navegador:

```text
http://<IP_HOST>:8888
```

---

# 🧪 Práctica 02 — Portainer

Portainer está orientado principalmente a administrar entornos de contenedores; la compatibilidad exacta con el motor y la edición utilizada debe comprobarse con la documentación de la versión elegida.

## Flujo

```text
Imagen Portainer
      ↓
Contenedor Portainer
      ↓
Interfaz HTTPS :9443
      ↓
Endpoint de contenedores
      ↓
Administración de nginx-lab
```

En esta práctica documenta:

1. Imagen utilizada y versión/tag.
2. Puertos publicados.
3. Volúmenes persistentes.
4. Endpoint configurado.
5. Evidencia de detener `nginx-lab` desde la interfaz.
6. Resultado del navegador tras detenerlo.

> [!NOTE]
> No publiques credenciales iniciales ni tokens de sesión. Usa secretos de laboratorio y reemplázalos en capturas públicas.

---

# 🧪 Práctica 03 — WordPress con Compose

## Objetivo original

Desplegar un contenedor de WordPress y otro de base de datos mediante un archivo `docker-compose.yml`.

## Ruta RHEL 10

Usa Podman y una herramienta Compose compatible con tu entorno. RHEL 10 soporta la ejecución de cargas con Podman y ofrece herramientas de administración de contenedores. citeturn447372search1turn551718search10

Ejemplo conceptual de `compose.yaml`:

```yaml
services:
  db:
    image: docker.io/library/mariadb:11
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: <CAMBIAR>
      MYSQL_ROOT_PASSWORD: <CAMBIAR>
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: docker.io/library/wordpress:latest
    ports:
      - "8088:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: <CAMBIAR>
    depends_on:
      - db
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:
```

> [!WARNING]
> Este YAML es una plantilla didáctica. Fija versiones y usa un archivo de secretos o variables de entorno fuera de Git antes de usarlo de forma real.

### ✅ Verificación

```bash
podman ps
curl -I http://127.0.0.1:8088
```

Después completa la instalación desde el navegador.

---

## 🧠 RHEL 10 vs Docker

| Enunciado original | Ruta documentada aquí |
|---|---|
| Docker Engine | Podman |
| `docker run` | `podman run` |
| Docker-compatible CLI | `podman-docker` (opcional) |
| Docker Compose | Herramienta Compose compatible con Podman |
| GUI | Portainer, validando compatibilidad |

Red Hat afirma explícitamente que Docker Engine no está soportado en RHEL 10 y que `podman-docker` puede proporcionar una interfaz compatible para comandos Docker. citeturn447372search1

---

## ✅ Checklist

- [ ] Podman instalado.
- [ ] NGINX en `:8888`.
- [ ] Volumen persistente.
- [ ] Portainer probado.
- [ ] WordPress + DB desplegados.
- [ ] Credenciales fuera del repo.
