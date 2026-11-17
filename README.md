# LG Conference (`com.sergio.conference` · v2.0.0)

Aplicación de videoconferencias grupales **WebRTC / PeerJS** en malla P2P (hasta **4 participantes simultáneos**) diseñada para televisores **LG webOS** y compatible con navegadores de **celulares y PC**.

> **Nota de arquitectura y separación de versiones:** Este repositorio corresponde exclusivamente a **LG Conference v2 (`com.sergio.conference`)**, separado del proyecto original 1-a-1 `LG Video Call` (`com.sergio.lgvideocall`, v1) para permitir instalar ambas aplicaciones en paralelo en el televisor sin interferir con la versión estable 1-a-1.

---

## Capacidades implementadas en `index.html`

- **Malla P2P multi-participante (`MAX_PARTICIPANTS = 4`):**
  - Interconexión automática entre todos los participantes mediante canales de datos (`DataConnection` con mensajes `PEER_LIST`) y llamadas WebRTC de malla (`metadata: { mesh: true }`) con `offerToReceiveAudio: true` y `offerToReceiveVideo: true`.
  - Si un nuevo participante llama a una sala con una reunión en curso, el anfitrión recibe el modal **"Nuevo participante"** mostrando el código y el cupo (`3/4`, `4/4`).
- **Cuadrícula dinámica y proporción exacta (`#video-grid` + `fitCellVideo`):**
  - 1 remoto: pantalla completa (`100% × 100%`).
  - 2 remotos: `50% × 100%` en horizontal (TV/PC) o apilado `100% × 50%` en celulares verticales.
  - 3 remotos: 2 superiores (`50% × 50%`) + 1 inferior (`100% × 50%`).
  - 4 remotos: cuadrícula `2 × 2` (`50% × 50%`).
- **Estabilidad de audio multicanal en webOS 6 (`Chrome/79` WAM) y móviles:**
  - Los elementos `<video>` se mantienen silenciados (`muted = true`, `volume = 0`) en todos los dispositivos para evitar colisiones de `WebMediaPlayerMS`.
  - **En televisor (`isTV`):** webOS `Chrome/79` congela el sink ALSA cuando se reproducen múltiples etiquetas `<audio>` WebRTC simultáneas; por ello todas las pistas de audio entrantes y el timbre comparten un **único `AudioContext` persistente** (`sharedAudioCtx`) que mezcla en tiempo real los flujos hacia `sharedAudioCtx.destination`.
  - **En celulares y PC (`!isTV`):** se asigna a cada interlocutor un canal `<audio>` dedicado (`remote-audio-0..2`) alimentado únicamente con sus pistas de audio (`new MediaStream(audioTracks)`), preservando cancelación de eco (AEC) del navegador.
  - Monitor automático cada 2 segundos (`audioWatchId`) y eventos `onclick` / `touchstart` / `keydown` para reanudar pistas si algún navegador móvil las pausa.
  - En clientes móviles/PC se aplica una captura de video moderada (`640×480` ideal, `18–24 fps`) para aliviar la decodificación por software en el SoC ARMv7 del televisor durante conferencias de 3–4 personas (el TV mantiene `{video: true}` nativo para abrir `camera2` sin demoras).

---

## Dependencias y servicios externos

- **PeerJS 1.5.2 (local):** Cargado desde `./Assets/js/peerjs.min.js` (sin dependencia externa de CDN en tiempo de ejecución y adaptado sin encadenamiento opcional `?.` para webOS 6 / `terser`).
- **Servidor de señalización público:** `0.peerjs.com` (`port: 443`, `secure: true`, `path: '/'`).
- **Servidores ICE (STUN/TURN públicos):**
  - `stun:stun.l.google.com:19302`
  - `stun:stun1.l.google.com:19302`
  - `stun:stun.cloudflare.com:3478`
  - `turn:openrelay.metered.ca:80` y `turn:openrelay.metered.ca:443` (`openrelayproject`)

*(Al tratarse de señalización y TURN públicos gratuitos, quedan sujetos a disponibilidad y cuota externa; en una futura Fase 2 se prevé migrar a `peerjs-server` + `coturn` propios o a un servidor SFU/MCU).*

---

## Empaquetado e instalación en webOS

```bash
ares-package -e ".git*" -e "*.ps1" -e "README.md" .
ares-install -d mitv com.sergio.conference_2.0.0_all.ipk
ares-launch -d mitv com.sergio.conference
```

---

## Estructura del repositorio

- `index.html`: Aplicación principal webOS y cliente web multi-participante.
- `appinfo.json`: Manifiesto webOS (`id: com.sergio.conference`, `version: 2.0.0`, `title: LG Conference`).
- `Assets/js/peerjs.min.js`: PeerJS 1.5.2 local compatible con Chromium 79/87 de webOS.
- `icon.png`, `largeIcon.png`, `og-image.png`: Iconografía de la app.
- `com.sergio.conference_2.0.0_all.ipk`: Paquete instalable generado con `ares-package`.
- `diagnostico.html`: Utilidad independiente de diagnóstico de cámara y micrófono (`getUserMedia`).
