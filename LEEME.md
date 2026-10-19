# laberinto-paquetes

Repositorio de **paquetes de conocimiento gratuitos** del Desafío del
Laberinto. El juego lee aquí el catálogo (`index.json`) y se baja cada
paquete de una vez al activarlo: después funciona igual offline, sin
necesitar el servidor ni la base de datos.

Los paquetes **de pago no viven aquí** (siguen en el servidor y solo se
obtienen con redención de código).

## Cómo está hecho

```
index.json                          ← manifiesto: catálogo + sha256 de cada fichero
paquetes/libres/<id>-v<version>.labpaq  ← la MISMA envoltura firmada que la app
```

- Cada fichero `.labpaq` es exactamente el formato `laberinto-package/1`
  con firma HMAC-SHA256 que la app valida offline al importar. Nada nuevo.
- **La versión va en el NOMBRE** del fichero: un contenido nuevo es un
  fichero nuevo, así que ninguna caché (raw.githubusercontent o
  cdn.jsdelivr.net/gh) puede servir una versión vencida.
- `index.json` es lo único que la app lee para el catálogo; su texto es
  canónico: si un `sha256` no coincide con el fichero bajado, la app
  aborta la instalación con mensaje.

## Cómo se actualiza

1. En el panel de administración se cura el paquete como siempre.
2. Se vuelca al repo (hoy a mano; luego el panel lo hará en un botón):
   ```
   dart run tool/volcar_paquetes_libres.dart --repo ../laberinto-paquetes
   ```
   El script baja los gratuitos del servidor, reconstruye las envolturas,
   recalcula hashes y reescribe `index.json`. No hace push solo.
3. Revisar el diff, y **el humano sube**:
   ```
   git add -A && git commit -m "paquete: <id> v<version>" && git push
   ```
4. La app entra al catálogo y trae la versión nueva: `index.json` cambia
   (raw, sin caché) y el fichero nuevo llega por su nombre con `vN+1`.

## Reglas del repo

- Nada de credenciales ni secretos en este repo.
- Los `.labpaq` nunca se editan a mano: se regeneran.
- Si un paquete se retira, se borra de `index.json` primero y su fichero
  en un segundo commit: la app vieja sigue funcionando un tiempo con su
  copia instalada local.
