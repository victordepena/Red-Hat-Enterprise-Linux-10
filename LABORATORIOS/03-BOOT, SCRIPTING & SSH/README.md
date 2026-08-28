# 03 · Boot, scripting y SSH

> GRUB · Recuperación · Bash/TAR · SSH + llaves

[⬅️ Volver al índice](../README.md)

## 🎬 Playlists

- Práctica 01: `PENDIENTE`
- Práctica 02: `PENDIENTE`
- Práctica 03: `PENDIENTE`

---

# 🧪 Práctica 01 — GRUB y recuperación de acceso

## A. Mostrar GRUB durante 20 segundos

En RHEL moderno, la configuración persistente de GRUB se gestiona mediante las herramientas de generación de configuración propias de la distribución. Antes de modificarla, crea un snapshot de la VM.

Consulta:

```bash
sudo grubby --info=ALL | head
```

Para revisar la configuración:

```bash
sudo grep -R "GRUB_TIMEOUT" /etc/default/grub /etc/default/grub.d 2>/dev/null
```

Establece:

```text
GRUB_TIMEOUT=20
```

Después regenera la configuración de GRUB usando el procedimiento apropiado para tu plataforma y verifica el archivo resultante. Reinicia solo cuando tengas un snapshot.

> [!IMPORTANT]
> El archivo exacto de salida de GRUB puede variar según firmware BIOS/UEFI y la disposición de la instalación. No copies una ruta de otra VM sin comprobar primero tu entorno.

## B. Recuperación del acceso root olvidado

El objetivo académico es demostrar un método de recuperación. En una VM de laboratorio, una ruta habitual es entrar en un entorno de recuperación de GRUB (`rd.break`), remontar el sistema raíz con escritura, cambiar la contraseña y efectuar el relabel de SELinux cuando corresponda.

Flujo conceptual:

```text
GRUB → editar entrada → rd.break → /sysroot → chroot → passwd → autorelabel → reboot
```

### ✅ Evidencia

- Menú GRUB mostrando el timeout.
- Consola de recuperación.
- Acceso administrativo restaurado.

> [!WARNING]
> Este procedimiento debe realizarse únicamente sobre tus propios sistemas o VMs de laboratorio. El objetivo es recuperación administrativa, no acceso a sistemas de terceros.

---

# 🧪 Práctica 02 — Scripts

## A. Backup TAR con fecha y hora

Crea `backup_home.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SOURCE="$HOME"
OUTDIR="$HOME/backups"
STAMP="$(date +'%d-%m-%Y:%H:%M')"
mkdir -p "$OUTDIR"

tar -czf "$OUTDIR/home_$STAMP.tar.gz" "$SOURCE"
echo "Backup creado: $OUTDIR/home_$STAMP.tar.gz"
```

Permisos y ejecución:

```bash
chmod +x backup_home.sh
./backup_home.sh
ls -lh ~/backups
```

> El formato pedido en el enunciado contiene `:`. En Linux es válido dentro de nombres de archivo.

## B. Script interactivo para guardar `ifconfig`

En RHEL 10, `ip` es la herramienta moderna de administración de interfaces. El laboratorio, sin embargo, solicita específicamente el resultado de `ifconfig`. Por compatibilidad académica, si el binario existe:

```bash
ifconfig
```

Crea `capturar_ifconfig.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

DESKTOP="$HOME/Desktop"
mkdir -p "$DESKTOP"

read -r -p "Nombre del archivo (sin extensión): " NAME

if command -v ifconfig >/dev/null 2>&1; then
  ifconfig > "$DESKTOP/${NAME}.txt"
else
  echo "ifconfig no está instalado en este sistema. Usa ip address o documenta la instalación del paquete compatible de tu entorno."
  exit 1
fi

echo "Archivo creado: $DESKTOP/${NAME}.txt"
```

---

# 🧪 Práctica 03 — SSH, llaves y acceso sin contraseña

## 1. Validar conectividad

```bash
ping -c 4 <IP_SERVIDOR>
```

## 2. Habilitar SSH

```bash
sudo dnf install -y openssh-server
sudo systemctl enable --now sshd
sudo systemctl status sshd
```

## 3. Abrir firewall para SSH

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

## 4. Primera conexión desde Windows

```powershell
ssh <USUARIO>@<IP_SERVIDOR>
```

## 5. Generar llave en Windows

En PowerShell:

```powershell
ssh-keygen -t ed25519
```

Deja la passphrase vacía **solo porque así lo pide el laboratorio**, entendiendo que una llave sin passphrase tiene menor protección ante robo de la clave privada.

## 6. Copiar la pública

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

En RHEL, añade el contenido a:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vi ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## 7. Probar

```powershell
ssh <USUARIO>@<IP_SERVIDOR>
```

### ✅ Verificación

```bash
sudo sshd -T | grep -E 'pubkeyauthentication|passwordauthentication'
```

### 📸 Evidencia

- Ping desde Windows.
- Estado de `sshd`.
- Primera conexión por contraseña.
- `id_ed25519.pub` generado.
- Conexión posterior usando la llave.

---

## ✅ Checklist

- [ ] GRUB en 20 s.
- [ ] Método de recuperación documentado.
- [ ] Backup TAR funcionando.
- [ ] Script interactivo funcionando.
- [ ] SSH habilitado.
- [ ] Llave Ed25519 generada.
- [ ] `authorized_keys` configurado.
- [ ] SSH sin pedir contraseña del usuario.
