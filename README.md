# NFL Survivor Online

Versión multiusuario con:
- Login por email y contraseña.
- Base de datos común para todos.
- Hora de elección guardada por el servidor.
- Un equipo no puede repetirse por jugador en una temporada.
- Resultados de temporada regular marcados después por un administrador.
- Cuadro completo de Playoffs obligatorio antes de la hora límite.
- 13 pronósticos: 6 Wild Card + 4 Divisionales + 2 finales de conferencia + campeón de Super Bowl.
- Resultados de Playoffs y penalización +5 gestionados después por el administrador.
- Clasificación compartida.
- Tema claro/oscuro.
- Compartir última elección por WhatsApp.

## Arquitectura recomendada

Frontend: GitHub Pages.
Backend: Supabase (Postgres + Auth).
WordPress: incrustar la misma app publicada usando el archivo `wordpress-embed.html`.

No uses GitHub como base de datos: GitHub Pages es alojamiento estático. La base común la proporciona Supabase.

## 1. Crear Supabase

1. Crea un proyecto en Supabase.
2. Abre SQL Editor.
3. Ejecuta TODO `supabase.sql`.
4. En Authentication configura Email/Password.
5. Copia Project URL y la publishable key (o anon key del proyecto).
6. Copia `config.example.js` como `config.js`.
7. Rellena:
   - `SUPABASE_URL`
   - `SUPABASE_KEY`

NUNCA pongas la `service_role` key en `config.js`.

## 2. Crear tu administrador

1. Abre la app.
2. Crea tu propia cuenta.
3. En Supabase > SQL Editor ejecuta:

```sql
update public.profiles
set role='admin'
where id=(select id from auth.users where email='TU_EMAIL');
```

Cierra sesión y vuelve a entrar.

El administrador puede:
- Marcar victorias/derrotas.
- Corregir resultados de Playoffs.
- Aplicar la penalización +5.
- Crear una temporada nueva y fijar el cierre de Playoffs.

## 3. Jugadores

Cada jugador crea una cuenta con:
- nombre visible
- email
- contraseña

Al registrarse, el trigger lo añade automáticamente a la temporada que esté marcada como activa.

## 4. Desplegar en GitHub Pages

1. Crea un repositorio.
2. Sube:
   - `index.html`
   - `config.js`
3. Settings > Pages.
4. Elige Deploy from a branch.
5. Selecciona `main` y `/root`.
6. Abre la URL de GitHub Pages.

Importante: la web publicada puede ser pública, pero los datos siguen protegidos por login y RLS en Supabase. No subas secretos.

## 5. Integrarlo en WordPress

Recomendado: publicar primero en GitHub Pages y luego incrustar la URL en WordPress mediante iframe.

Abre `wordpress-embed.html`, cambia:
`https://TU-USUARIO.github.io/TU-REPOSITORIO/`
por tu URL real y pega el iframe en un bloque HTML personalizado de WordPress.

Así WordPress solo muestra la aplicación; Supabase sigue siendo la única base de datos.

## 6. Reglas técnicas importantes

### Temporada regular
- Cada usuario solo puede insertar su propia elección.
- Una elección por jornada.
- Un equipo solo puede usarse una vez por temporada.
- La hora `created_at` la asigna Postgres/Supabase, no el reloj del móvil.
- Los jugadores no pueden modificar sus resultados.
- El administrador marca victoria/derrota.

### Playoffs
- El usuario debe completar los 13 campos.
- Solo existe un cuadro por jugador y temporada.
- El cuadro se registra de una vez.
- La base de datos rechaza un cuadro nuevo si ya pasó `playoff_lock_at`.
- El jugador no puede modificarlo después.
- El administrador corrige aciertos/fallos después.

### Puntuación
Regular:
- victoria: 0
- derrota: 19 - jornada

Playoffs:
- Wild Card: -1 por acierto
- Divisional: -2
- Final de conferencia: -3
- Super Bowl: -4
- Si el campeón previsto cae en su primer partido de Playoffs: +5

## 7. Migración de los datos antiguos

La v4 local guardaba datos en `localStorage`; esta versión no los importa automáticamente porque ahora cada elección debe quedar asociada a un usuario real de Supabase. Lo recomendable es empezar la temporada online desde cero o hacer una importación administrativa controlada.

## Archivos

- `index.html` — app.
- `config.js` — configuración local (rellenar).
- `config.example.js` — plantilla.
- `supabase.sql` — tablas, triggers y seguridad RLS.
- `wordpress-embed.html` — iframe para WordPress.
- `README.md` — esta guía.
