# 02 · Administración del sistema

> DNF · Repositorios · Cron · at · Discos y montaje

[⬅️ Volver al índice](../README.md)

## 🎬 Playlists

- Práctica 01: `PENDIENTE`
- Práctica 02: `PENDIENTE`
- Práctica 03: `PENDIENTE`

---

# 🧪 Práctica 01 — DNF y repositorios

## Objetivo

Actualizar el sistema, inspeccionar repositorios, localizar una herramienta, instalarla, desinstalarla y limpiar dependencias no utilizadas.

## 1. Actualizar

```bash
sudo dnf check-update || true
sudo dnf upgrade -y
```

## 2. Enumerar repositorios

```bash
dnf repolist
sudo dnf repolist --all
```

## 3. Buscar paquetes

```bash
dnf search bashtop
dnf info bashtop
```

> [!NOTE]
> La disponibilidad de `bashtop` depende de los repositorios habilitados. No asumas que una URL de terceros del enunciado es un repositorio confiable o compatible con RHEL 10. Documenta el repositorio exacto que realmente utilizaste.

## 4. Instalar / desinstalar

Si el paquete está disponible:

```bash
sudo dnf install <PAQUETE>
<PAQUETE> --help
```

Luego:

```bash
sudo dnf remove <PAQUETE>
sudo dnf autoremove
```

Inspecciona dependencias antes de borrar:

```bash
dnf repoquery --installed --whatrequires <PAQUETE>
```

### ✅ Evidencia

```bash
dnf repolist
rpm -qa | grep -i <PAQUETE>
dnf history info last
```

---

# 🧪 Práctica 02 — Cron y `at`

## A. Actualizar el sistema cada día a las 23:00

Editar el crontab de root:

```bash
sudo crontab -e
```

Ejemplo de entrada:

```cron
0 23 * * * /usr/bin/dnf upgrade -y
```

Comprueba:

```bash
sudo crontab -l
systemctl status crond
```

> [!TIP]
> En entornos de producción suele ser preferible gestionar actualizaciones con políticas de mantenimiento y herramientas de gestión centralizada. Aquí se conserva `cron` porque es el objetivo didáctico del laboratorio.

## B. Reinicio semanal — domingo 03:00

```cron
0 3 * * 0 /usr/bin/systemctl reboot
```

> [!WARNING]
> Un reinicio programado es disruptivo. Solo ejecútalo dentro de una VM de laboratorio o durante una ventana de mantenimiento.

## C. `at` para limpiar `/tmp` en un minuto

Crear el trabajo:

```bash
echo 'find /tmp -mindepth 1 -maxdepth 1 -exec rm -rf -- {} +' | at now + 1 minute
```

Ver trabajos:

```bash
atq
```

Antes y después:

```bash
ls -la /tmp
# esperar la ejecución del trabajo
ls -la /tmp
```

> [!CAUTION]
> Para el laboratorio, considera crear primero archivos de prueba propios dentro de `/tmp`. No utilices este ejercicio sobre un sistema real sin comprender qué archivos va a eliminar.

---

# 🧪 Práctica 03 — Disco de 20 GB y montaje

RHEL 10 soporta ext4 y documenta `mount`, `findmnt` y `/etc/fstab` para administrar sistemas de archivos. citeturn551718search7turn551718search8

## 1. Adjuntar disco

Añade un disco virtual de 20 GB a la VM y reinicia si tu hipervisor lo requiere.

Identifica el nuevo dispositivo:

```bash
lsblk
sudo fdisk -l
```

## 2. Crear partición

Ejemplo con `fdisk`:

```bash
sudo fdisk /dev/<DISCO>
```

Crea una partición Linux que ocupe el disco. Luego:

```bash
sudo partprobe /dev/<DISCO>
lsblk
```

## 3. Formatear ext4

```bash
sudo mkfs.ext4 /dev/<PARTICION>
```

## 4. Montar en una carpeta de laboratorio

```bash
mkdir -p ~/disco20
sudo mount /dev/<PARTICION> ~/disco20
findmnt ~/disco20
df -h ~/disco20
```

## 5. Crear archivo

```bash
cd ~/disco20
echo "Laboratorio RHEL 10" | tee AdrianAlcantara.txt
cat AdrianAlcantara.txt
```

## 6. Desmontar y montar en `/mnt`

```bash
cd ~
sudo umount ~/disco20
sudo mount /dev/<PARTICION> /mnt
findmnt /mnt
ls -l /mnt
```

El archivo debe seguir presente porque reside en el sistema de archivos del disco, no en el directorio que servía como punto de montaje. Red Hat documenta que un sistema de archivos se adjunta al árbol mediante un punto de montaje y puede montarse nuevamente en otra ubicación. citeturn551718search4

### ✅ Verificación final

```bash
cat /mnt/AdrianAlcantara.txt
findmnt /mnt
```

---

## ✅ Checklist

- [ ] Repositorios inspeccionados.
- [ ] Paquetes buscados e instalados/desinstalados.
- [ ] `cron` comprobado.
- [ ] `at` ejecutado y validado.
- [ ] Disco de 20 GB detectado.
- [ ] ext4 creado.
- [ ] Archivo persistente verificado después de remonte.
