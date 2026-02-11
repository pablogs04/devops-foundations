# Linux troubleshooting notes


## Incidente 1 — systemd user service falla con 203/EXEC (ExecStart inválido)

**Síntoma:**
- `systemctl --user status fail-demo.service` muestra `failed` con `status=203/EXEC`.

**Detección (comandos):**
- `systemctl --user status fail-demo.service --no-pager`
- `sudo journalctl --no-pager -n 80 _SYSTEMD_USER_UNIT=fail-demo.service`

**Evidencia (logs):**
- `Failed to locate executable /usr/bin/does-not-exist: No such file or directory`
- `Failed at step EXEC spawning /usr/bin/does-not-exist: No such file or directory`

**Causa raíz:**
- En la unidad, `ExecStart` apuntaba a un binario inexistente (`/usr/bin/does-not-exist`), por lo que systemd no pudo ejecutar el proceso (203/EXEC).

**Solución (comandos):**
- Editar `ExecStart` a un comando válido que se mantenga vivo:
  - `sed -i 's|ExecStart=.*|ExecStart=/usr/bin/sleep infinity|' ~/.config/systemd/user/fail-demo.service`
- Recargar unidades:
  - `systemctl --user daemon-reload`
- Reiniciar y validar:
  - `systemctl --user restart fail-demo.service`
  - `systemctl --user status fail-demo.service --no-pager`

**Prevención / qué aprendí:**
- Para servicios `systemd --user`, el journal puede requerir `sudo` y el filtro `_SYSTEMD_USER_UNIT`.
- Ante `203/EXEC`, comprobar `ExecStart` y validar rutas con `command -v` o `ls -la`.
- Tras cambios en unidades: `daemon-reload` + `restart` + `status`.

## Incidente 2 — Puerto ocupado: OSError [Errno 98] Address already in use

**Síntoma:**
- Al ejecutar `python3 -m http.server 8080` falla con:
  - `OSError: [Errno 98] Address already in use`

**Detección (comandos):**
- Confirmar que hay un listener en 8080:
  - `ss -ltn | grep ':8080'`
- Identificar proceso (PID/programa):
  - `ss -ltnp | grep ':8080'`
- Verificar qué ejecuta el PID:
  - `ps -fp <PID>`
  - `tr '\0' ' ' < /proc/<PID>/cmdline; echo`

**Causa raíz:**
- Ya existía un proceso escuchando en `0.0.0.0:8080`, por lo que el nuevo servidor no pudo hacer `bind()` al puerto.

**Solución (comandos):**
- Parar el proceso que ocupaba el puerto:
  - `kill <PID>`
- Verificar que el puerto queda libre:
  - `ss -ltnp | grep ':8080' || echo "8080 libre"`
- Reintentar el arranque:
  - `python3 -m http.server 8080`
- Parar el servidor (cuando termine la prueba):
  - `Ctrl+C`

**Prevención / qué aprendí:**
- Ante “Address already in use”, primero identificar el proceso dueño del puerto antes de matar nada.
- `ss -ltnp` da PID y programa; `ps -fp` confirma qué es.
- Si es producción y no puedes matar el proceso, alternativa: cambiar el puerto o reconfigurar el servicio.

## Incidente 3 — Docker: permission denied en /var/run/docker.sock

**Síntoma:**
- `docker ps` falla con:
  - `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`

**Detección (comandos):**
- Ver permisos del socket:
  - `ls -la /var/run/docker.sock`
  - Ejemplo esperado: `root docker` y permisos `srw-rw----`
- Confirmar que con sudo funciona:
  - `sudo docker ps`

**Causa raíz:**
- El socket Docker pertenece a `root:docker` y el usuario no estaba en el grupo `docker`, así que no tenía permisos de lectura/escritura sobre el socket.

**Solución (comandos):**
- Añadir usuario al grupo docker:
  - `sudo usermod -aG docker <user>`
- Aplicar cambios (re-login o sesión nueva):
  - `newgrp docker` (para aplicar en la sesión actual) o cerrar y abrir sesión
- Validar:
  - `docker ps` (sin sudo)

**Prevención / qué aprendí:**
- Docker se controla vía `docker.sock`; los permisos del socket mandan.
- En servidores, usar grupo `docker` evita depender de sudo (pero es un permiso potente).

## Incidente 4 — Archivos del repo con owner root (por usar sudo) → no puedo editarlos sin sudo

**Síntoma:**
- <<<Describe el síntoma exacto>>>  
  Ejemplos típicos:
  - No puedo guardar/editar `docker/README.md` sin sudo.
  - VS Code/terminal me da “Permission denied” al editar.
  - `ls -la` muestra archivos con `root root`.

**Detección (comandos):**
- Ver en qué carpeta estoy:
  - `pwd`
- Ver propietario/permisos de los archivos afectados:
  - `ls -la docker`
  - (si aplica) `ls -la docker/<archivo>`
- Confirmar usuario actual:
  - `whoami`
- (Opcional) Ver grupos del usuario:
  - `id`

**Evidencia (salida relevante):**
- <<<Pega 2–4 líneas clave>>>  
  Ejemplo:
  - `-rw-r--r--. 1 root root 155 ... docker-compose.yml`
  - `-rw-r--r--. 1 root root 734 ... README.md`

**Causa raíz:**
- Se ejecutaron comandos con `sudo` creando/modificando archivos dentro del repositorio.
- `sudo` escribe como `root`, por eso el owner quedó `root:root` y luego el usuario normal no puede editar cómodamente.

**Solución (comandos):**
- Cambiar owner del árbol afectado al usuario normal:
  - `sudo chown -R <tu_usuario>:<tu_usuario> docker`
- Verificar:
  - `ls -la docker`
  - Confirmar que ya no aparece `root root` en esos ficheros.

**Prevención / qué aprendí:**
- Evitar `sudo` dentro del repositorio salvo necesidad real.
- Si necesito `sudo` para Docker, mejor arreglar permisos/grupo (`docker`) que andar usando sudo siempre.
- Si me vuelve a pasar: `ls -la` para detectar owner + `chown -R` para corregir.

## Incidente 5 — Proceso al 100% CPU (runaway process)

**Síntoma:**
- El servidor se nota “pesado” / CPU alta.
- En este lab, se observó un proceso consumiendo ~100% CPU.

**Detección (comandos):**
- Crear el problema (controlado) y obtener PID:
  - `yes > /dev/null & echo $!`
- Ver consumo del proceso:
  - `ps -p <PID> -o pid,pcpu,pmem,cmd`
- Ver en tiempo real:
  - `top -p <PID>`
- Alternativa rápida para encontrar “top CPU” sin saber PID:
  - `ps aux --sort=-%cpu | head`

**Evidencia (salida observada):**
- `ps` mostró algo similar a:
  - `PID <PID>  %CPU ~100  CMD yes`
- `top` confirmó el proceso `yes` con ~99–100% CPU.

**Causa raíz:**
- Un proceso en bucle (`yes`) estaba ejecutándose sin parar.
- Aunque se redirigió a `/dev/null` (no genera carga de disco), sigue consumiendo CPU porque calcula/escribe continuamente.

**Solución (comandos):**
- Parar el proceso de forma “normal” (SIGTERM por defecto):
  - `kill <PID>`
- Verificar que ya no existe:
  - `ps -p <PID> || echo "muerto"`

**Prevención / qué aprendí:**
- Ante CPU alta: primero **identificar** el proceso (no matar “a ciegas”).
- `ps/top` te da evidencia clara (PID + %CPU + comando).
- `kill` (SIGTERM) es el primer intento; si no muere, entonces evaluar escalado (`kill -9`) *solo después* de confirmar que es el proceso correcto.
- En producción: si pasa a menudo, plantear límites/contención (systemd limits, cgroups) y monitorización/alertas.

