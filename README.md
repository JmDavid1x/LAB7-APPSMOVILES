# Laboratorio 7 · Fotos en el muro

**Jose David Mayor López** · Aplicaciones Móviles · Universidad Icesi

El muro del Laboratorio 6, ahora con fotos: cada persona sube su foto de perfil desde la galería, y los posts pueden llevar una imagen.

Las fotos de perfil van en un **bucket público**, `avatars`, y se muestran con una URL fija. Las imágenes de los posts van en un **bucket privado**, `post-images`, y solo quien inició sesión las ve, con URLs firmadas que duran una hora.

## Lo que se hizo

### Supabase

- Conexión con el proyecto propio de Supabase: URL y publishable key en `lib/main.dart`, y carpeta `supabase/` con el `config.toml` del curso.
- Bucket público `avatars` y bucket privado `post-images`, los dos de máximo 2 MiB y solo `jpeg`, `png` y `webp`.
- Migraciones `create_avatars` y `create_post_images`: las columnas `profiles.avatar_path` y `posts.image_path`, y una política por bucket para los usuarios autenticados.

### Parte 1 · Foto de perfil

- **`DevicePhotoDataSource`**: abre la galería con `image_picker` (512 × 512, calidad 80) y devuelve un `PickedPhoto`.
- **`SupabaseAvatarDataSource`**: sube la foto a `avatars/<id del usuario>/<uuid>.<extensión>` y borra la anterior.
- **`AvatarRepositoryImpl`**: delega en los dos data sources.
- **`ChangeAvatarUseCase`**: elige la foto, rechaza las de más de 2 MB, sube la nueva, actualiza el perfil y al final borra la vieja. Si la persona cierra la galería sin elegir, no pasa nada.
- **`SupabaseProfileDataSource`**: `updateAvatarPath` y la URL pública de la foto en `getProfile`.
- **`SupabasePostDataSource`**: trae el `avatar_path` del autor y arma su URL pública, así cada post muestra la foto de quien lo escribió, o su inicial si no tiene foto.
- **`FeedBloc`**: muestra un indicador mientras sube, un `SnackBar` si falla, y recarga el muro con la foto nueva.
- **`HomeScreen`**: el botón de la cámara del perfil lanza `AvatarChangeRequested`.

### Parte 2 · Posts con imagen

- **`SupabasePostImageDataSource`**: sube a `post-images` y pide URLs firmadas de una hora.
- **`PostRepositoryImpl`**: `pickImage`, `uploadImage` e `imageUrl`.
- **`SupabasePostDataSource`**: guarda `image_path` al crear el post y lo lee en el muro.
- **`PickPostImageUseCase`**: elige la imagen y rechaza las de más de 2 MB.
- **`PublishPostUseCase`**: un post puede ser texto, foto o las dos cosas. La foto se sube antes de crear el post.
- **`GetFeedUseCase` y `GetMyPostsUseCase`**: completan cada post con la URL firmada de su imagen, todas al mismo tiempo con `Future.wait`.
- **`ComposeBloc`**: adjunta, quita y publica la foto.
- **`ComposeScreen`**: el botón *Agregar foto* lanza `ComposePhotoRequested`.

## Cómo correrlo

```bash
flutter pub get
flutter run
```
