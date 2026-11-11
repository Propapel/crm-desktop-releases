# crm-desktop-releases

Instaladores (MSI) del CRM Propapel para Windows. Usado por el actualizador automático de la app.

> Este repositorio debe seguir siendo **público**: la app consulta la API de GitHub sin token
> (un token dentro del MSI podría extraerse).

## Cómo funciona la actualización

Al abrir la app instalada, `GitHubUpdateSource` consulta:

```
GET https://api.github.com/repos/Propapel/crm-desktop-releases/releases/latest
```

y compara el tag con `APP_VERSION` (definido en `composeApp/build.gradle.kts` del repo
`CRM-PROPAPEL`, `val version = "X.Y.Z"`). Si el release es más nuevo y trae un `.msi`, ofrece
descargarlo; la descarga se reanuda si se corta y el MSI se instala en modo pasivo.

## Publicar una versión nueva

1. En `CRM-PROPAPEL/composeApp/build.gradle.kts` sube `val version = "X.Y.Z"`
   (MSI solo acepta números `MAJOR.MINOR.BUILD`, sin sufijos como `-beta`).
2. Genera el instalador en Windows:
   ```
   gradlew :composeApp:packageMsi
   ```
   El archivo queda en `composeApp/build/compose/binaries/main/msi/CRM-X.Y.Z.msi`.
3. En este repo crea un release:
   - **Tag:** `vX.Y.Z` (exactamente la misma versión que `APP_VERSION`).
   - **Título:** `CRM X.Y.Z`.
   - **Descripción:** novedades en lenguaje para el usuario; se muestran en el diálogo de actualización.
   - **Asset:** el `.msi` generado.
   - No lo marques como *draft* ni *pre-release*: el actualizador solo lee el release *latest*.

Con `gh`:

```bash
gh release create vX.Y.Z CRM-X.Y.Z.msi \
  --repo Propapel/crm-desktop-releases \
  --title "CRM X.Y.Z" \
  --notes "Novedades de esta versión…"
```

## Notificaciones push (opcional)

La app recibe push de Firebase en escritorio con [fcm-desktop](https://github.com/eliasmp07/fcm-desktop).
Para activarlas, antes de generar el MSI agrega en `CRM-PROPAPEL/local.properties`
(o como variables de entorno) los datos de una app **Web** de Firebase del mismo proyecto que usa el backend:

```properties
FCM_WEB_PROJECT_ID=tu-proyecto
FCM_WEB_APP_ID=1:1234567890:web:abc123
FCM_WEB_API_KEY=AIza...
```

- En Google Cloud la API key debe permitir **Firebase Installations API** y **FCM Registration API**, sin restricción de referrer, Android ni iOS.
- Si se compila sin estos datos el push queda apagado y la app sigue avisando de tickets nuevos consultando al servidor cada 30 s.
- Los equipos necesitan salida a `mtalk.google.com` por el puerto 5228 (o 443).

## Reglas que no hay que romper

- El `upgradeUuid` del MSI (`0CFFB049-CB02-40E6-B9F1-606ECC204D7E`) nunca cambia; si cambia,
  Windows instala la versión nueva junto a la anterior en lugar de reemplazarla.
- La versión del tag siempre debe ser mayor que la anterior; nunca reutilices un tag.
- Si un MSI sale mal, publica una versión más alta con la corrección (no borres el release
  anterior mientras haya usuarios descargándolo).

## Estado

| Versión de la app (`build.gradle.kts`) | Último release publicado |
| --- | --- |
| 2.0.5 | v2.0.5 (09/10/2026) |
