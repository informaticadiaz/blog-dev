# Guía de Referencia: Scripts para Cambiar entre Ventanas en Ubuntu

## 📋 Introducción

Esta guía te ayudará a crear scripts personalizados para cambiar entre ventanas de aplicaciones específicas usando atajos de teclado en Ubuntu.

## 🛠️ Herramientas Necesarias

### Instalar dependencias:
```bash
sudo apt update
sudo apt install wmctrl xdotool
```

### Verificar instalación:
```bash
wmctrl -l  # Lista todas las ventanas
xdotool getactivewindow  # Obtiene ID de ventana activa
```

## 📁 Estructura de Directorios

```
~/.local/bin/
├── switch-vscode.sh
├── switch-brave.sh
├── switch-firefox.sh
└── switch-terminal.sh
```

## 🔧 Comandos Útiles

### wmctrl (Window Manager Control)
```bash
# Listar todas las ventanas
wmctrl -l

# Cambiar a ventana por nombre
wmctrl -a "nombre_ventana"

# Cambiar a ventana por clase
wmctrl -x -a "clase.Clase"

# Cambiar a ventana por ID
wmctrl -i -a "window_id"

# Obtener información detallada
wmctrl -lG  # Con geometría
wmctrl -lp  # Con PID
```

### xdotool (Herramienta de automatización X11)
```bash
# Obtener ventana activa
xdotool getactivewindow

# Buscar ventanas por nombre
xdotool search --name "nombre"

# Activar ventana
xdotool windowactivate "window_id"

# Obtener información de ventana
xdotool getwindowname "window_id"
```

## 📝 Plantilla Base de Script

```bash
#!/bin/bash
# Plantilla para cambiar entre ventanas de [APLICACIÓN]

# Configuración
APP_NAME="nombre_aplicacion"
APP_CLASS="clase.Clase"
APP_COMMAND="comando_para_abrir"

# Obtener ventanas de la aplicación
windows=$(wmctrl -l | grep -i "$APP_NAME" | awk '{print $1}')
current=$(xdotool getactivewindow 2>/dev/null)

# Verificar si hay ventanas abiertas
if [ -n "$windows" ]; then
    window_array=($windows)
    
    # Si hay múltiples ventanas, ciclar entre ellas
    if [ ${#window_array[@]} -gt 1 ]; then
        for i in "${!window_array[@]}"; do
            if [[ "${window_array[$i]}" == "0x$(printf '%x' $current)" ]]; then
                next_index=$(( (i + 1) % ${#window_array[@]} ))
                wmctrl -i -a "${window_array[$next_index]}"
                exit 0
            fi
        done
    fi
    
    # Ir a la primera ventana
    wmctrl -i -a "${window_array[0]}"
else
    # Si no hay ventanas, abrir la aplicación
    $APP_COMMAND &
fi
```

## 🎯 Ejemplos Específicos

### Visual Studio Code
```bash
#!/bin/bash
# Script para VS Code
wmctrl -x -a "code.Code" 2>/dev/null || \
wmctrl -a "Visual Studio Code" 2>/dev/null || \
code &
```

### Brave Browser
```bash
#!/bin/bash
# Script para Brave
wmctrl -x -a "brave-browser.Brave-browser" 2>/dev/null || \
wmctrl -a "Brave" 2>/dev/null || \
brave-browser &
```

### Firefox
```bash
#!/bin/bash
# Script para Firefox
wmctrl -x -a "firefox.Firefox" 2>/dev/null || \
wmctrl -a "Firefox" 2>/dev/null || \
firefox &
```

### Terminal
```bash
#!/bin/bash
# Script para Terminal
wmctrl -x -a "gnome-terminal.Gnome-terminal" 2>/dev/null || \
wmctrl -a "Terminal" 2>/dev/null || \
gnome-terminal &
```

### LibreOffice Writer
```bash
#!/bin/bash
# Script para LibreOffice Writer
wmctrl -x -a "libreoffice-writer.libreoffice-writer" 2>/dev/null || \
wmctrl -a "LibreOffice Writer" 2>/dev/null || \
libreoffice --writer &
```

## ⚙️ Configuración de Atajos

### Método 1: Interfaz Gráfica
1. **Abrir Configuración:** `gnome-control-center`
2. **Ir a:** Keyboard → View and Customize Shortcuts
3. **Agregar Custom Shortcut:**
   - Name: "Cambiar a [Aplicación]"
   - Command: `/home/usuario/.local/bin/switch-app.sh`
   - Shortcut: `Super + Tecla`

### Método 2: Línea de Comandos
```bash
# Configurar atajo personalizado
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybindings \
"['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"

# Configurar nombre, comando y atajo
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
name 'Cambiar a VS Code'

gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
command '/home/usuario/.local/bin/switch-vscode.sh'

gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
binding '<Super>v'
```

## 🔍 Identificar Nombres y Clases de Aplicaciones

### Método 1: wmctrl
```bash
# Con la aplicación abierta
wmctrl -l | grep -i "parte_del_nombre"
```

### Método 2: xprop
```bash
# Ejecutar y hacer clic en la ventana
xprop | grep -E "(WM_CLASS|WM_NAME)"
```

### Método 3: xdotool
```bash
# Buscar por nombre
xdotool search --name "nombre_aplicacion"
```

## 🎨 Combinaciones de Teclas Recomendadas

### Seguras (raramente en conflicto):
- `Super + V` → VS Code
- `Super + B` → Brave/Browser
- `Super + T` → Terminal
- `Super + F` → Firefox
- `Super + E` → Editor de texto
- `Super + M` → Música/Media
- `Super + C` → Calculadora
- `Super + N` → Notas

### Alternativas:
- `Super + Shift + [Letra]`
- `Super + Alt + [Letra]`
- `Ctrl + Alt + [Letra]`

## 🔧 Solución de Problemas

### Verificar si el script funciona:
```bash
# Hacer ejecutable
chmod +x ~/.local/bin/switch-app.sh

# Probar manualmente
~/.local/bin/switch-app.sh
```

### Debugging:
```bash
# Agregar debug al script
echo "Debug: Buscando ventanas de $APP_NAME"
wmctrl -l | grep -i "$APP_NAME"
```

### Problemas comunes:
1. **Script no ejecuta:** Verificar permisos (`chmod +x`)
2. **No encuentra ventana:** Verificar nombre exacto con `wmctrl -l`
3. **Atajo no funciona:** Verificar conflictos con otros atajos
4. **Aplicación no abre:** Verificar comando en terminal

## 📋 Lista de Verificación

- [ ] Instalar `wmctrl` y `xdotool`
- [ ] Crear directorio `~/.local/bin/`
- [ ] Escribir script personalizado
- [ ] Hacer script ejecutable (`chmod +x`)
- [ ] Probar script manualmente
- [ ] Configurar atajo de teclado
- [ ] Probar atajo completo
- [ ] Documentar atajo asignado

## 🎯 Atajos Sugeridos por Aplicación

| Aplicación | Atajo Recomendado | Comando |
|------------|-------------------|---------|
| VS Code | `Super + V` | `code` |
| Brave Browser | `Super + B` | `brave-browser` |
| Firefox | `Super + F` | `firefox` |
| Terminal | `Super + T` | `gnome-terminal` |
| LibreOffice | `Super + L` | `libreoffice` |
| File Manager | `Super + E` | `nautilus` |
| Calculator | `Super + C` | `gnome-calculator` |
| Text Editor | `Super + N` | `gedit` |

## 🔄 Mantenimiento

### Actualizar scripts:
```bash
# Respaldar configuración actual
cp ~/.local/bin/switch-*.sh ~/scripts-backup/

# Editar scripts
nano ~/.local/bin/switch-app.sh
```

### Ver todos los atajos configurados:
```bash
gsettings list-recursively org.gnome.desktop.wm.keybindings | grep -v disabled
```

---

**Nota:** Recuerda reiniciar la sesión o ejecutar `source ~/.bashrc` después de crear nuevos scripts para que estén disponibles en el PATH.