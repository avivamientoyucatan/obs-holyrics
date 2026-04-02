# Conexion de OBS a Holyrics via Tailscale Serve

**Iglesia Avivamiento Yucatan** | Fecha de creacion: 2 de abril de 2026

---

## Resumen del problema

OBS (en PC-1) necesita conectarse al servidor de Holyrics (en PC-2) para mostrar texto en vivo. La conexion directa por IP de Tailscale no funciona porque una Politica de Grupo (GPO) del firewall de Windows bloquea las reglas locales de entrada en el PC-2.

La solucion es usar **Tailscale Serve**, que crea un proxy interno que evita el firewall.

---

## Datos de la red

| Dispositivo | Nombre | IP Tailscale |
|---|---|---|
| PC-1 (OBS) | avivamiento-yucatan-pc-1 | 100.91.45.50 |
| PC-2 (Holyrics) | avivamiento-yucatan-pc-2 | 100.122.150.38 |

**URL que se usa en OBS:**

```
https://avivamiento-yucatan-pc-2.taile184f2.ts.net/view/text2
```

---

## Configuracion inicial (solo se hace una vez)

En el **PC-2** (donde corre Holyrics), abrir una terminal como **Administrador** y ejecutar:

```bash
tailscale serve --bg 81
```

Donde `81` es el puerto en el que Holyrics esta escuchando. Esta configuracion es **persistente**, sobrevive reinicios del PC.

---

## Si cambia el puerto de Holyrics

Si por alguna razon el puerto de Holyrics cambia (por ejemplo a 82), ejecutar en el **PC-2** como **Administrador**:

1. Apagar el serve actual:

```bash
tailscale serve --https=443 off
```

2. Crear el serve con el nuevo puerto:

```bash
tailscale serve --bg 82
```

> **Nota:** La URL en OBS **NO cambia**. Sigue siendo la misma:
> `https://avivamiento-yucatan-pc-2.taile184f2.ts.net/view/text2`

---

## Como verificar que todo funciona

1. Verificar que Tailscale esta activo en ambos PCs:

```bash
tailscale status
```

2. Verificar que el serve esta corriendo (en PC-2):

```bash
tailscale serve status
```

3. Verificar el puerto de Holyrics (en PC-2):

```bash
netstat -an | findstr LISTENING | findstr :81
```

4. Probar la conexion desde PC-1:

```bash
curl https://avivamiento-yucatan-pc-2.taile184f2.ts.net/view/text2
```

---

## Si deja de funcionar - Pasos de diagnostico

1. **Verificar que Tailscale esta conectado en ambos PCs**
   - Revisar el icono de Tailscale en la barra de tareas

2. **Verificar que Holyrics esta corriendo en PC-2**
   - Abrir Holyrics y confirmar que el servidor esta activo

3. **Verificar que el serve esta activo** (en PC-2):

   ```bash
   tailscale serve status
   ```

   Si no aparece nada, volver a crearlo:

   ```bash
   tailscale serve --bg 81
   ```

4. **Verificar que el puerto no haya cambiado** (en PC-2):

   ```bash
   netstat -an | findstr LISTENING | findstr :81
   ```

   Si el puerto cambio, seguir los pasos de la seccion "Si cambia el puerto de Holyrics".

5. **Hacer ping entre PCs** para confirmar conectividad:

   ```bash
   ping 100.122.150.38
   ```

---

## Para desactivar Tailscale Serve

Si ya no se necesita, ejecutar en el **PC-2** como **Administrador**:

```bash
tailscale serve --https=443 off
```

---

## Notas importantes

- Tailscale Serve es **gratuito** dentro de la tailnet
- Las IPs de Tailscale (100.x.x.x) **no cambian**
- La URL DNS **tampoco cambia**
- La conexion entre PCs en la misma red local es **directa** (peer-to-peer), no depende de los servidores de Tailscale una vez establecida
- **No** se necesita configurar reglas de firewall con esta solucion
