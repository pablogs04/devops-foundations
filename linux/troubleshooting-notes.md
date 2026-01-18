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

