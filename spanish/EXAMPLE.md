# TITULO DEL DESIGN DOC
Link: [Link a este design doc](#)

Author(s): Charlie L

Status: [Draft, Ready for review, In Review, Reviewed]

Ultima actualización: YYYY-MM-DD

## Contenido
- Goals
- Non-Goals
- Background
- Overview
- Detailed Design
  - Solucion 1
    - Frontend
    - Backend
  - Solucion 2
    - Frontend
    - Backend
- Consideraciones
- Métricas

## Links
- [Un link](#)
- [Otro link](#)

## Objetivo
Una aplicación web que transfiera playlists de Spotify a YouTube Music

Actualmente, ninguna app ofrece una forma de transferir playlists desde una plataforma a la otra

## Goals
- Transferir playlists de Spotify a YouTube Music
- Ofrecer opciones para elegir el reemplazo de una canción si no se encuentra el match
## Non-Goals
- Transferir playlists de YouTube Music a Spotify
- Mantener en sincronía ambas playlist

## Background
Hace poco me cambié de Spotify a YouTube Music, y no pude transferir mis playlists porque ambas apps no ofrecen herramientas para ello

Hay sitios web que ofrecen este feature, pero mi segunda intención es aprender a usar las APIs de YT Music y Spotify

## Overview
Necesitamos una API que convierta una playlist de un usuario de Spotify a una playlist de YouTube Music

Cada playlist tiene un id, y podemos consultar todas sus canciones a través de la API oficial usando ese id

Spotify ofrece una [API](https://developer.spotify.com/documentation/web-api/reference/#/operations/get-list-users-playlists) que podemos usar para obtener:
- Obtener las playlists de un user
- Obtener las canciones de esa playlist

Por el otro, YouTube Music no ofrece una **API pública**, pero existe una [API no oficial](https://ytmusicapi.readthedocs.io/en/latest/) que provee los métodos necesarios para crear una playlist, los cuales son:
- Crear playlist
- Buscar canción
- Añadir canción a la playlist

## Detailed Design

## Solución 1

## Spotify
### Obtener playlist del usuario
A través del endpoint */users/{user_id}/playlists* podemos obtener todas las playlist del usuario
Cuando el user seleccione una playlist, guardamos el ID

### Obtener canciones de la playlist
Con la plalist ID, a través del endpoint */playlists/{playlist_id}/tracks* podemos obtener todas las canciones
El endpoint regresa un resultado con la siguiente forma:
```json
{
  "href": "https://api.spotify.com/v1/me/shows?offset=0&limit=20\n",
  "items": [
    {}
  ],
  "limit": 20,
  "next": "https://api.spotify.com/v1/me/shows?offset=1&limit=1",
  "offset": 0,
  "previous": "https://api.spotify.com/v1/me/shows?offset=1&limit=1",
  "total": 4
}
```
Las canciones están en el campo **items**, la cual es una lista de objetos que representan a las canciones

## YouTube Music
YouTube Music no cuenta con una API oficial. Pero existe una librería hecha en Python que provee acceso a esta API.

[Librería](https://ytmusicapi.readthedocs.io/en/latest/)

### Crear playlist
Podemos crear una playlist usando la siguiente función de la librería

```python
YTMusic.create_playlist(title: str, description: str, privacy_status: str = 'PRIVATE', video_ids: List[T] = None, source_playlist: str = None) → Union[str, Dict[KT, VT]]
```

### Buscar canciones de Spotify en YouTube Music
Podemos usar la función de search para buscar cada canción
Esto significa que por *n canciones*, se tienen que hacer *n llamadas*

```python
# Snippet
YTMusic.search(query: str, filter: str = None, scope: str = None, limit: int = 20, ignore_spelling: bool = False) → List[Dict[KT, VT]]
```

```python
# Ejemplo
youtubeSongs = []
for item in canciones:
    # crear el query de la canción
    cancion = item['title'] + " " + item['artist'] + " " + item['album']
    # buscar la canción y agregar el primer resultado
    youtubeSongs.append(
        Songs.YTMusic.search(
        query: cancion,
        filter: "songs",
        limit: 20,
        ignore_spelling: True)[0]
    )
```

### Guardar canción en playlist

```python
def getId(song):
    return song['id']

youtubeSongsIds = map(addition, youtubeSongs)

YTMusic.add_playlist_items(playlistId: playlistId, videoIds: youtubeSongsIds, source_playlist: None, duplicates: False)
```

## Solucion 2
Podemos reusar los mismos métodos de la solución 1, con los siguientes cambios

### Buscar canciones de Spotify en YouTube Music
En lugar de agregar la primer opción de la búsqueda inmediatamente, podemos ofrecer todas las opciones al user para que elija cual es el mejor match de la canción

Esto implica ofrecer una UI donde:
- Se muestren todas las opciones
- Se puedan reproducir cada opción para que el user elija cual es la adecuada para la playlist

Esto necesitaría otro design doc enfocado a la UI si se elige esta solución

## Consideraciones
- La librería solo esta en Python. Tal vez podemos entender como funciona para implementarla por nuestra cuenta en otro lenguaje si es necesario

## Métricas
- Checar Web Vitals para entender si la app carga de forma adecuada y tiene buen performance
- Validar cuantos usuarios terminan el proceso de migración de una playlist