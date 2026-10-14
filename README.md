# Control remoto BLE para silla de ruedas (controlador Melaunch)

**Abrir:** https://tecnibrain.github.io/silla-ble/

Página web única (`index.html`) que usa **Web Bluetooth** para:

1. **Conectarse** al controlador de la silla.
2. **Explorar** sus servicios/características BLE, leer, suscribirse a notificaciones y enviar bytes en hex.
3. **Registrar** todo el tráfico (y descargarlo en .txt).
4. **Controlar** la silla con un joystick táctil, usando una plantilla de comando configurable.

## Cómo abrirla

Web Bluetooth exige `https://` o `localhost`. Opciones:

- PC: `npx serve .` y abrir `http://localhost:3000` en Chrome/Edge.
- Android: subir la carpeta a cualquier hosting https (o GitHub Pages) y abrirla en Chrome.
- iPhone: Safari no soporta Web Bluetooth; usar la app gratuita **Bluefy**.

## Paso clave: descubrir el protocolo

El fabricante no publica el protocolo Bluetooth, así que primero hay que averiguar **qué bytes** espera el controlador.
La plantilla que trae la página (`AA {x} {y} {cs}`) es solo un ejemplo.

1. **Con la app oficial** (el QR del manual lleva a la cuenta de WeChat; suele haber un mini-programa o una app Android):
   - En Android: Opciones de desarrollador → *Habilitar registro de Bluetooth HCI (snoop log)*.
   - Desactiva y activa el Bluetooth, usa la app oficial: avanza, retrocede, gira, cambia velocidad, para.
   - Genera un *informe de errores* (bug report) y extrae `btsnoop_hci.log`; ábrelo con **Wireshark**
     y filtra por `btatt.opcode == 0x52 || btatt.opcode == 0x12` (escrituras). Ahí verás la característica y los bytes exactos.
2. **Sin app oficial**: conéctate con esta página (o con **nRF Connect**), suscríbete a todas las características
   *notify* y mueve el joystick físico. Si el controlador reporta su estado, verás qué bytes cambian con X/Y.
3. Anota: UUID de servicio y característica, cabecera, orden de X/Y, rango (con signo, offset 128, 0–200…), checksum,
   y si necesita un comando de “desbloqueo/encendido” o envío periódico (heartbeat).
4. Pon esos datos en la sección **4. Control remoto** (se guardan en el navegador).

Si el UUID del servicio no aparece, escríbelo en “UUIDs de servicio extra” (Web Bluetooth solo deja ver los servicios declarados).

## Medidas de seguridad incluidas

- Joystick bloqueado hasta marcar la confirmación “silla vacía, ruedas elevadas”.
- **Hombre muerto**: solo envía movimiento mientras el dedo está sobre el joystick; al soltar envía la parada 3 veces.
- Parada automática al cambiar de pestaña, perder el foco o desconectarse.
- Límite de velocidad (por defecto 30 %).

Aun así, el controlador debe tener su propio *timeout* (detenerse si deja de recibir comandos). Compruébalo con las
ruedas elevadas: desconecta el Bluetooth en pleno movimiento y verifica que los motores paran solos.
**No uses el control remoto con una persona sentada hasta haber verificado eso.**
