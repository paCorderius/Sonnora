# Sonnora — adivina la canción con tus amigos

Juego para adivinar canciones de una playlist de Spotify escuchando fragmentos de 1, 3 o 10 segundos. El anfitrión crea una sala y pone la música desde su cuenta Spotify Premium. Los demás (hasta 4 jugadores en total) se unen desde su propio teléfono con un código de 5 letras, sin necesidad de cuenta.

## Cómo se juega

1. **El anfitrión crea la sala.** Toca *Crear sala*, conecta su cuenta de Spotify, pega el link de una playlist suya y elige:
   - **Qué cuenta como respuesta:** solo el nombre de la canción, el nombre aproximado de la canción, o el nombre de la canción o del artista.
   - **Temporizador:** 45 s por ronda más 10 s para robar, o sin límite de tiempo (sin robo).
   - **Número de canciones:** hasta 60. Lo ideal es un múltiplo del número de jugadores.
2. **Los demás se unen.** Tocan *Unirme con código*, escriben el código, eligen color y nombre. El orden en que entran es el orden de turnos.
3. **En cada turno:**
   - El jugador en turno elige cuánto escuchar. Los fragmentos de 1 s y 3 s se pueden repetir; el de 10 s solo una vez.
   - **Las estrellas dependen del fragmento más largo que hayas escuchado en la ronda:** 1 s = 3 estrellas, 3 s = 2, 10 s = 1.
   - Hay que escuchar al menos un fragmento antes de poder responder.
   - Con temporizador, el reloj de 45 s arranca con el primer fragmento.
   - Hay 3 intentos. Si se acaban los intentos o el tiempo, los demás tienen 10 s para **robar** la ronda por 1 estrella (solo con temporizador y 2 jugadores o más).
   - Al terminar la ronda suena la canción completa y el anfitrión pasa a la siguiente.
4. **Al final** se muestra el podio con los tres primeros lugares.

La interfaz toma el color del jugador en turno (barra de turno y botones de fragmento).

### Qué cuenta como respuesta correcta

- No importan mayúsculas, acentos ni signos de puntuación.
- Se ignoran agregados del título como "(feat. …)", "[Live]" o "- Remastered 2011".
- Se aceptan errores de dedo pequeños (más o menos uno cada 5 letras).
- Se puede escribir de más (hasta 3 palabras extra), siempre que vaya el título completo ("bohemian rhapsody de queen").
- Funciona con títulos en cualquier alfabeto (japonés, coreano, cirílico, etc.).

Según el modo que elija el anfitrión:

- **Solo el nombre de la canción:** hay que escribir el título completo. Una palabra suelta no cuenta ("rhapsody" no acierta "Bohemian Rhapsody").
- **Nombre aproximado de la canción:** basta con acertar al menos el 40 % de las palabras importantes del título (no cuentan "el", "the", "de", "vol", "remix"…), y al menos la mitad de lo que escribas tiene que estar en el título. Si escribes un número, tiene que coincidir. Ejemplos con "Shakira: Bzrp Music Sessions, Vol. 53":
  - "bzrp 53", "music sessions 53" o "bizarrap music session 53" sí aciertan.
  - "bzrp", "shakira" o "music sessions 52" no aciertan.
- **Nombre de la canción o del artista:** como el primero, pero también vale el nombre de cualquiera de los artistas.

## Requisitos

- **Anfitrión:** cuenta Spotify **Premium** y un navegador compatible con el Web Playback SDK de Spotify.
- **Invitados:** solo el código de la sala. No necesitan cuenta de Spotify.
- Una app creada en el [Spotify Developer Dashboard](https://developer.spotify.com/dashboard).
- Un proyecto de Firebase con Realtime Database (ya configurado en `index.html`).

> **Límites de Spotify (desde febrero de 2026):** las apps en modo *Development* requieren que el dueño de la app tenga Premium y tienen un límite bajo de usuarios autorizados. Además, Spotify solo entrega las canciones de playlists que son tuyas o en las que colaboras. Revisa la [guía de migración de Spotify](https://developer.spotify.com/documentation/web-api/tutorials/february-2026-migration-guide) para los detalles vigentes.

## Configuración de Spotify (una sola vez)

1. Entra a tu app en el [Dashboard de Spotify](https://developer.spotify.com/dashboard) → **Settings**.
2. En **Redirect URIs** agrega exactamente la URL donde abres el juego:
   - Local: `http://127.0.0.1:8080/index.html` (Spotify no acepta `localhost`).
   - GitHub Pages: la URL pública exacta, por ejemplo `https://tuusuario.github.io/Sonnora/index.html`.
3. Guarda los cambios y copia el **Client ID**. No necesitas el Client Secret: el juego usa PKCE, sin backend.
4. Si la app está en modo *Development*, agrega en **User Management** el correo de cada persona que vaya a ser anfitrión.

La sesión de Spotify se renueva sola mientras juegas, así que no se corta a la hora.

## Configuración de Firebase

1. En la [consola de Firebase](https://console.firebase.google.com/), abre el proyecto → **Realtime Database** → **Reglas**.
2. Copia el contenido de `database.rules.json`, pégalo y toca **Publicar**.

Estas reglas:
- Solo aceptan salas con la forma que usa el juego (código de 5 letras, máximo 4 jugadores, nombres de hasta 20 caracteres, etc.).
- Impiden listar todas las salas.
- Permiten que el juego borre automáticamente salas abandonadas de hace más de 24 horas. Cada vez que alguien crea una sala, se borran hasta 20 salas viejas.

Las salas también se borran cuando el anfitrión toca *Salir* o *Jugar de nuevo*.

> El juego no usa inicio de sesión en Firebase, así que cualquiera que conozca el código de una sala puede escribir en ella. Para una app pública, el siguiente paso sería activar Firebase Authentication anónima y limitar las escrituras por usuario.

## Correrlo en local

Desde esta carpeta:

```bash
# Con Python 3
python3 -m http.server 8080

# o con Node
npx serve -l 8080
```

Abre `http://127.0.0.1:8080/index.html`. Debe coincidir exactamente con la Redirect URI registrada.

## Publicarlo en GitHub Pages

Sube `index.html` (y este README) al repositorio y activa GitHub Pages en **Settings → Pages**. Registra la URL pública como Redirect URI en Spotify.

## Si algo falla

- **Se recargó la página o se cerró la app:** al volver a abrirla, el juego te reconecta solo a la sala en la que estabas.
- **El jugador en turno se desconectó:** con temporizador, la ronda avanza sola cuando se acaba el tiempo. Sin temporizador, el anfitrión puede tocar *Saltar esta ronda*.
- **Sesión de Spotify vencida:** aparece un aviso con el botón *Reconectar*. Al volver de Spotify, regresas a la misma sala.
- **Cerrar sesión de Spotify:** botón *Cerrar sesión de Spotify* en la pantalla de crear sala.

## Notas técnicas

- Todo corre en el navegador: un solo `index.html`, sin servidor propio.
- **Spotify:** flujo Authorization Code + PKCE con renovación automática del token. El audio lo reproduce el Web Playback SDK en el dispositivo del anfitrión.
- **Firebase Realtime Database** sincroniza la sala:
  - Todos los cambios de fase (adivinar, robar, fin del tiempo) se hacen con transacciones, así que solo un dispositivo gana cada cambio y no hay dobles escrituras.
  - Los temporizadores usan la hora del servidor de Firebase, así que todos los teléfonos ven el mismo tiempo.
  - Cualquier dispositivo puede cerrar una ronda cuando se acaba el tiempo, no solo el del jugador en turno.
- Los fragmentos se miden desde que el audio realmente empieza a sonar. Si se pide otro fragmento antes de que termine el anterior, se cancela la pausa pendiente.
- Al cargar la playlist se descartan canciones locales y canciones no disponibles en el país del anfitrión.
- Los nombres de los jugadores se muestran siempre como texto, nunca como HTML.
- La pantalla se mantiene encendida durante la partida (Screen Wake Lock API, donde el navegador lo permite).
