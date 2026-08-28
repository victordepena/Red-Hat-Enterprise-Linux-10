# 04 · Servicios de red

> HTTP · Virtual Hosts · SMTP · CUPS

[⬅️ Volver al índice](../README.md)

## 🎬 Playlists https://www.youtube.com/playlist?list=PLQ9VDfI34EclPKPJziFwlAyN4DjNBG0Uj
---

# 🧪 Práctica 01 — Apache / NGINX y Virtual Hosts

RHEL 10 documenta Apache como `httpd`; NGINX también está disponible como servidor web. citeturn447372search5turn447372search6

## Ruta recomendada: Apache `httpd`

### 1. Instalar

```bash
sudo dnf install -y httpd
sudo systemctl enable --now httpd
```

### 2. Abrir HTTP

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

### 3. Website “Hola Mundo”

```bash
sudo mkdir -p /var/www/html/hola
sudo tee /var/www/html/hola/index.html <<'HTML'
<!doctype html>
<html lang="es">
<head><meta charset="utf-8"><title>Hola Mundo</title></head>
<body><h1>Hola Mundo</h1></body>
</html>
HTML
```

### 4. Virtual Host :80

Crea `/etc/httpd/conf.d/hola.conf`:

```apache
<VirtualHost *:80>
    ServerName hola.local
    DocumentRoot /var/www/html/hola

    <Directory /var/www/html/hola>
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Valida y recarga:

```bash
sudo apachectl configtest
sudo systemctl reload httpd
```

### 5. Website con datos del estudiante

```bash
sudo mkdir -p /var/www/html/estudiante
sudo tee /var/www/html/estudiante/index.html <<'HTML'
<!doctype html>
<html lang="es">
<head><meta charset="utf-8"><title>Laboratorio RHEL 10</title></head>
<body>
  <h1><TU_NOMBRE></h1>
  <p>Matrícula: <TU_MATRICULA></p>
  <p>Materia: <NOMBRE_MATERIA></p>
</body>
</html>
HTML
```

### 6. Virtual Host :8080

Añade `/etc/httpd/conf.d/estudiante.conf`:

```apache
Listen 8080

<VirtualHost *:8080>
    ServerName estudiante.local
    DocumentRoot /var/www/html/estudiante

    <Directory /var/www/html/estudiante>
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Después:

```bash
sudo apachectl configtest
sudo systemctl restart httpd
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### ✅ Verificar

```bash
curl http://127.0.0.1/
curl http://127.0.0.1:8080/
sudo ss -lntp | grep -E ':80|:8080'
```

> [!NOTE]
> Si pruebas desde otro host, revisa también SELinux, firewall, DNS/`/etc/hosts` y la conectividad IP.

---

# 🧪 Práctica 02 — Postfix SMTP

## Objetivo

Instalar Postfix y demostrar el funcionamiento de un MTA en el laboratorio.

### 1. Instalar

```bash
sudo dnf install -y postfix
sudo systemctl enable --now postfix
systemctl status postfix
```

### 2. Identidad del host

Configura un hostname coherente y documenta el dominio de laboratorio.

```bash
hostnamectl
postconf myhostname
postconf mydomain
```

### 3. Prueba local

Para no exponer credenciales ni depender del correo de terceros, prueba primero una entrega local:

```bash
echo "Nombre: <TU_NOMBRE>\nMatrícula: <TU_MATRICULA>" | mail -s "MambruSeFueALaGuerra" <DESTINO_LOCAL>
```

Luego inspecciona logs:

```bash
sudo journalctl -u postfix --since "10 minutes ago"
```

> [!IMPORTANT]
> El enunciado original pide enviar a un correo del profesor. El repositorio no debe publicar esa dirección. Sustitúyela localmente por el destinatario indicado por tu curso y documenta únicamente el resultado.

### ✅ Evidencia

- `systemctl status postfix`.
- Logs de entrega.
- Prueba de recepción autorizada.

---

# 🧪 Práctica 03 — CUPS e impresora PDF

## 1. Instalar CUPS

```bash
sudo dnf install -y cups
sudo systemctl enable --now cups
```

## 2. Verificar

```bash
systemctl status cups
lpstat -r
lpstat -p
```

## 3. Impresora virtual PDF

El método exacto para un backend PDF depende de los paquetes disponibles en tu instalación. Documenta qué backend/cola instalaste y valida que CUPS la reconoce:

```bash
lpstat -v
lpstat -p
```

## 4. Cliente

Desde el cliente añade la impresora por la URL/IP expuesta por CUPS según la configuración del laboratorio.

## 5. Prueba

Imprime un documento y verifica que aparece en el destino PDF configurado.

### ✅ Checklist

- [ ] CUPS instalado.
- [ ] Cola PDF creada.
- [ ] Cliente configurado.
- [ ] Documento de prueba entregado.
