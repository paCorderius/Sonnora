# Sonnora para Android

La app es una **Trusted Web Activity (TWA)**: abre https://pacorderius.github.io/Sonnora/index.html
en pantalla completa usando Chrome, con su propio ícono y pantalla de carga. Como el juego vive en
GitHub Pages, cualquier cambio a `index.html` llega a la app sin generar otro APK.

Requisitos en el teléfono: Android 5.0 o superior y Chrome instalado (si no hay Chrome, se abre en
otro navegador compatible).

## Cómo se genera el APK

El workflow `.github/workflows/android.yml` compila el APK en GitHub Actions:

- Corre solo cuando cambia algo dentro de `android/` o el propio workflow.
- También se puede lanzar a mano: pestaña **Actions → APK de Android → Run workflow**.
- Cada compilación publica `sonnora.apk` en **Releases** (la más reciente queda marcada como *Latest*).

## Firma

Para que Android acepte cada versión nueva como actualización, todas tienen que estar firmadas con la
misma llave. La llave **no** está en este repositorio; se guarda en dos secretos del repositorio
(**Settings → Secrets and variables → Actions**):

| Secreto | Contenido |
|---|---|
| `SONNORA_KEYSTORE_B64` | La llave `.jks` codificada en base64 |
| `SONNORA_KEYSTORE_PASSWORD` | Su contraseña (la misma para la llave y el almacén; alias `sonnora`) |

Si los secretos no existen, el APK se firma con una llave temporal: instala bien, pero para instalar
la siguiente versión habrá que desinstalar la anterior.

## Quitar la barra de dirección (opcional)

Android muestra una barra pequeña con la dirección mientras el dominio no confirme que la app es suya.
Para quitarla hay que publicar `assetlinks.json` (con la huella SHA-256 de la llave) en
`https://pacorderius.github.io/.well-known/assetlinks.json`, es decir, en el repositorio
`paCorderius.github.io`, con un archivo `.nojekyll` en su raíz.
