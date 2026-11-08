# Jxxvault: notas para trabajar en el proyecto

## Plan

- **Ahora:** web instalable (PWA) publicada con GitHub Pages desde este repo. Se sigue ampliando con nuevas utilidades.
- **Al final:** una app Android (.apk) hecha con **Capacitor**, con todos los archivos de la web **dentro** de la APK, para que funcione sin depender de GitHub.

Todo cambio se hace pensando en ese paso final. Si algo nuevo choca con estas reglas, se avisa antes de hacerlo.

## Reglas para cada cambio

1. **Todo autocontenido.** La app es `index.html` (más `manifest.json`, `sw.js` e iconos). Nada se carga de un CDN: ni librerías, ni fuentes, ni hojas de estilo. Si hace falta una librería, se copia dentro del proyecto.
2. **Rutas relativas.** Nada de rutas que empiecen por `/` ni que supongan que la app vive en `*.github.io`. Si se usa `location.hostname`, tiene que haber alternativa para cuando no sea GitHub Pages.
3. **Los datos solo en el dispositivo.** Todo lo personal se guarda cifrado en el almacenamiento local. Nunca un servidor propio ni datos personales en el repo.
4. **Red solo para precios y datos públicos.** Cada servicio externo (CoinGecko, Finnhub, Yahoo…) se llama desde una única función, para poder cambiar la forma de llamarlo en la APK. Lo que hoy viene del robot de GitHub Actions (`data/*.json`) necesita un plan B para la APK: pedirlo directamente a la fuente, cosa que la APK sí puede hacer.
5. **Guardar archivos con una sola función.** En la APK, `a.download` no funciona. Toda exportación (copia de seguridad, CSV…) pasa por la misma función, que en la APK se cambiará por el sistema de archivos o por "Compartir".
6. **La contraseña siempre sirve para entrar.** La huella por WebAuthn no funciona dentro de la APK. Se sustituirá por un complemento biométrico nativo, así que ninguna función puede depender solo de la huella.
7. **Enlaces externos.** Los `target="_blank"` se abrirán con el navegador del sistema en la APK.
8. **`sw.js` es solo para la web.** En la APK no se registra porque los archivos ya están dentro.
9. **Pensado para el móvil.** Respetar los márgenes seguros (cámara y barra de gestos). Las hojas y diálogos deben poder cerrarse con el botón "atrás" de Android.
10. **La copia de seguridad es el puente.** Para pasar de la web a la APK se exporta la copia en la web y se importa en la APK. El formato de la copia debe seguir siendo compatible con versiones anteriores: se puede añadir campos, nunca romper los que hay.

## Puntos conocidos para el paso a APK

- **ETFs europeos:** `repoFetch()` lee `data/precios.json` e `historico.json`, que genera `.github/workflows/precios.yml`. En la APK habrá que pedir a Yahoo directamente.
- **Históricos de acciones y ETFs (gráficos):** `stockHist()` lee la rama `datos` (`h/<SÍMBOLO>.json`), que publica `.github/workflows/historico.yml` con un catálogo amplio para no revelar las posiciones. En la APK, `stockHist()` pedirá el histórico a Yahoo directamente y el catálogo dejará de hacer falta. Oro, plata y bitcoin (`coinHist()`) ya van directos a CoinGecko.
- **Huella:** el bloque `huella (WebAuthn + PRF)` y el modo «llave» con IndexedDB necesitan un complemento biométrico nativo.
- **Copia de seguridad:** el bloque `copia de seguridad` usa `URL.createObjectURL` + `a.download`.
- **Service worker:** se registra al final del script (`navigator.serviceWorker.register`). Hay que saltarlo en la APK.
- **Un enlace externo** con `target="_blank"` (registro en Finnhub).
- `confirm()`, los selectores de archivo (`<input type="file">`), `localStorage`, IndexedDB y `DecompressionStream` funcionan en el WebView de Capacitor.
