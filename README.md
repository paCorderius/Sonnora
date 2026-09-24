# 🎧 Sonnora — Juego de Adivinar Canciones con Spotify

Juego sobre adivinar canciones de una playlist de Spotify escuchando fragmentos de 1, 3 o 10 segundos. Soporta partidas de 1 a 4 jugadores (o equipos) por turnos, con tablero de puntuación y color de interfaz personalizado por jugador.

## Cómo jugar

1. **Conectar**: pega tu Client ID y presiona "Conectar con Spotify" (te pedirá login/permiso). Al conectar arranca un temporizador de **1 hora** (duración real del token de Spotify) visible en la barra superior; cuando se acerque a expirar, vuelve a conectar.
2. **Configurar la partida**: pega el link de la playlist y elige:
   - **Validación de respuesta**: solo nombre de la canción, o nombre y artista.
   - **Modo de temporizador**: con límite de 45s por ronda, o sin límite.
   - **Número de jugadores**: de 1 a 4.
   - **Número de canciones**: hasta 60 (el juego sugiere que sea múltiplo del número de jugadores para repartir turnos parejo).
3. **Jugadores**: ingresa el nickname y elige un color para cada uno (Azul, Rojo, Verde, Amarillo, Rosa o Morado). El orden en que los configuras define el orden de turnos.
4. **Jugar cada ronda**:
   - Elige cuántos segundos escuchar. Los samples de **1s y 3s se pueden repetir** las veces que quieras; el de **10s solo se escucha una vez**.
   - Si el modo con temporizador está activo, al reproducir el primer sample de 1s arranca una cuenta regresiva de 45s (se pone grande, en negritas y roja/parpadeante en los últimos 10s). Si llega a 0, la ronda se pierde automáticamente.
   - Tienes **3 intentos** para adivinar.
   - Al acertar, fallar los 3 intentos o agotarse el tiempo, la canción se reproduce **completa y normal** hasta que presionas "Siguiente canción".
5. El color de la interfaz (bordes, botones, temporizador) cambia según el jugador en turno, y el tablero de puntuación muestra el nickname y las estrellas de cada uno.
6. Al terminar todas las canciones verás el resultado final: puntaje único (1 jugador) o el ranking completo con el ganador destacado (2-4 jugadores).

## Requisitos

- Cuenta Spotify **Premium** en cada dispositivo donde se vaya a reproducir audio (obligatorio para el Web Playback SDK).
- Una app creada en el [Spotify Developer Dashboard](https://developer.spotify.com/dashboard).
- Python o Node instalados (para servir el archivo localmente), o un hosting estático si lo vas a desplegar online.

## Configuración (una sola vez)

1. Entra a tu app en el [Dashboard de Spotify](https://developer.spotify.com/dashboard) → **Settings**.
2. En **Redirect URIs**, agrega exactamente la URL donde vas a abrir el juego:
   - Local: `http://127.0.0.1:8080/index.html` (Spotify ya no acepta `localhost`, usa `127.0.0.1`)
   - Desplegado: la URL pública exacta (por ejemplo `https://tuusuario.github.io/sonnora/index.html`)
3. Guarda cambios.
4. Copia tu **Client ID** (no necesitas el Client Secret — el juego usa PKCE, sin backend).
5. Si tu app sigue en modo **Development** en el dashboard, agrega el correo de cada persona que vaya a loguearse en **User Management** (límite de 25 usuarios sin aprobación de Spotify).

## Jugar desplegado en GitHub Pages (online)

Sonnora no tiene backend ni servidor propio: es un solo archivo HTML que corre 100% en el navegador de quien lo abre. Desplegarlo en GitHub Pages solo cambia *dónde* vive el archivo — no cambia cómo funciona el login ni comparte nada entre jugadores. Esto es importante para entender el modo de juego:

- **Cada persona necesita su propio Client ID.** El login es por navegador y por Client ID, no por dominio. Que el juego esté en una URL pública no significa que tus amigos usen tu autorización — cada uno debe:
  1. Crear su propia app en su [Spotify Developer Dashboard](https://developer.spotify.com/dashboard).
  2. Agregar tu misma URL de GitHub Pages como Redirect URI en **su** app.
  3. Pegar **su propio** Client ID en el campo de la pantalla de login cuando entre a jugar.
- **Cada uno necesita cuenta Premium.** El Web Playback SDK reproduce el audio dentro del navegador de quien está logueado en ese momento, así que sin Premium esa persona no podrá escuchar los fragmentos.
- **La sesión es individual, no compartida.** El token, el Client ID y el timestamp de expiración se guardan en el `localStorage` del navegador de cada quien. Si dos amigos abren la misma URL desde dispositivos distintos, cada uno tiene su propia sesión independiente — no hay nada centralizado ni un "anfitrión" del lado del servidor.
- **Modo de juego real: un dispositivo, turnos pasados a mano.** El modo multijugador (2-4 jugadores) está pensado para jugarse desde **una sola pantalla/dispositivo**, donde los jugadores se van pasando el turno físicamente. No es multijugador en tiempo real entre dispositivos distintos — cada quien vería su propia partida si abre el link por separado.
- **App en modo "Development" en Spotify**: mientras tu app no esté aprobada por Spotify en modo "Extended Quota", solo pueden loguearse los correos que agregues manualmente en **User Management** del dashboard (máximo 25). Esto solo aplica si tus amigos usan **tu** Client ID; si cada quien crea el suyo propio (recomendado, ver arriba), este límite no les afecta.
- **Playlists privadas**: si la playlist es privada y no pertenece a la cuenta que hace login (ni está guardada en su biblioteca), la API puede devolver error 403 al leer las canciones. Usa una playlist pública, o pide a cada jugador que la guarde en su propia cuenta antes de jugar.

## Cómo correrlo en local

Desde esta carpeta, levanta un servidor local (elige uno):

```bash
# Con Python 3
python3 -m http.server 8080

# o con Node
npx serve -l 8080
```

Abre en el navegador `http://127.0.0.1:8080/index.html` — debe coincidir EXACTO con el Redirect URI registrado en el paso anterior.

## Borrar caché / reiniciar sesión

El botón **"🗑️ Borrar caché / Cerrar sesión"** en la barra superior limpia el token, el Client ID guardado y cualquier dato de sesión en `localStorage`, y recarga la página desde cero. Útil si el token expiró o quieres conectar con otra cuenta.

## Notas técnicas

- El juego corre 100% en el navegador (sin backend), usando el flujo **Authorization Code + PKCE** de Spotify — no requiere el Client Secret.
- La reproducción real usa el **Web Playback SDK** de Spotify, que requiere Premium.
- La sesión (token, Client ID, timestamp del token) se guarda en `localStorage` de tu navegador; nada se envía a ningún servidor propio.
- Si despliegas el juego en un hosting estático (GitHub Pages, Vercel, Cloudflare Pages, etc.), la Redirect URI se calcula automáticamente según el dominio donde corre — solo debes registrar esa URL nueva en el Spotify Developer Dashboard.

