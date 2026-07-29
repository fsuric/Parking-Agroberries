[README.md](https://github.com/user-attachments/files/30520505/README.md)
# Estacionamientos Agroberries — v4

Dos piezas: **Supabase** guarda los datos, **GitHub Pages** sirve la página.

| Archivo | Qué es |
|---|---|
| `index.html` | La aplicación completa. Un solo archivo, sin dependencias. |
| `migration_v4.sql` | Estructura de la base de datos. Se ejecuta una vez en Supabase. |

---

## 1. Base de datos (Supabase)

1. Entra a [supabase.com/dashboard](https://supabase.com/dashboard) y abre el proyecto.
   Si aparece como **Paused**, haz clic en **Restore** y espera ~2 minutos. Los
   proyectos del plan gratuito se pausan tras aproximadamente una semana sin uso;
   eso es lo que produce el error `Failed to fetch`.
2. **SQL Editor** → **New query** → pega todo `migration_v4.sql` → **Run**.
3. Al final debe listar tres tablas: `releases`, `requests`, `reservations`.

El script es idempotente y **no borra datos**: puedes correrlo sobre una base
vacía o sobre la versión anterior, y ejecutarlo dos veces no hace daño.

Las credenciales ya están dentro de `index.html`. Si algún día creas un proyecto
nuevo en Supabase, hay que actualizar `SB_URL` y `SB_KEY` al inicio del bloque
`<script>` (busca `CONFIG`).

---

## 2. Publicar en GitHub Pages

1. Entra a `github.com/fsuric/Parking-Agroberries`
2. Abre `index.html` → ícono del lápiz (**Edit**)
3. Selecciona todo (`Ctrl+A`) y bórralo
4. Pega el contenido del nuevo `index.html`
5. **Commit changes**

En 1-2 minutos queda publicado en `https://fsuric.github.io/Parking-Agroberries/`.

Si fuera un repositorio nuevo: **Settings → Pages → Source: Deploy from a branch
→ Branch `main`, carpeta `/ (root)` → Save**. El archivo debe llamarse
exactamente `index.html`.

---

## 3. Los 18 estacionamientos

Distribución según el plano, 7 columnas:

|  | c1 | c2 | c3 | c4 | c5 | c6 | c7 |
|---|---|---|---|---|---|---|---|
| arriba | | 7041 | Ascensores | 6002 | 3002 | | 3011 |
| | | 7040 | | 6003 | 3003 | | 3012 |
| línea B | 4030-B | 4024-B | 4023-B | | | | |
| línea A | 4030-A | 4024-A | 4023-A | 6021 | 6020 | 6018 | 6017 |

El mapa usa columnas fluidas (`1fr`), así que los 7 caben a lo ancho de un
celular sin scroll horizontal. Las tarjetas se encogen con `clamp()`.

### Asignaciones fijas

| Espacio | Asignado a | Regla |
|---|---|---|
| 3002 | Erwin Grune | siempre |
| 3003 | Juan Pablo Vogt | siempre |
| 3011 | Max Emden | siempre |
| 3012 | Juan Pablo Lavin | siempre |
| 6017 | Catalina Vives | siempre |
| 6018 | Francisco Suric | siempre |
| 6020 | Natalia Cwiklitzer | siempre |
| 6021 | Braulio Osorio | siempre |
| 6002, 6003, 7040, 7041 | Visitas | por hora, 9:00–18:00 |

Los únicos de reserva libre por rango de fechas son los seis tándem
(4023-A/B, 4024-A/B, 4030-A/B). De los 18 espacios: 8 fijos, 4 de visita por
hora y 6 libres.

### Iniciales

Sobre cada espacio ocupado se muestran las iniciales, y el nombre completo
aparece al pasar el mouse (o al mantener presionado en celular).

Nombres de dos palabras usan dos iniciales; de tres o más, tres. Eso es lo que
distingue a **JPV** (Juan Pablo Vogt, 3003) de **JPL** (Juan Pablo Lavin, 3012),
que con dos letras quedaban idénticos.

---

## 4. Uso

### Vista mapa
- Barra superior: rango de fechas.
- Espacio libre → reservar por rango de fechas.
- Espacio reservado → liberar.
- **6002**, **6003**, **7040** o **7041** → abre la vista horaria.
- Un espacio fijo muestra las iniciales del dueño. Si el administrador lo liberó,
  aparece con borde punteado y la palabra «liberado», y pasa a ser reservable.

### Vista horaria (visitas: 6002, 6003, 7040, 7041)
- Lunes a viernes, 9:00 a 18:00, bloques de una hora.
- Toca un bloque libre para seleccionarlo; toca un bloque **contiguo** para extender.
- **1 hora** → reserva directa.
- **2 horas o más** → genera una solicitud que el administrador aprueba o rechaza.
- Las horas ya pasadas quedan deshabilitadas.

### Vista administrador
Botón **⚙** abajo a la derecha.

- Usuario: `AdminAB`
- Clave: `EstacionamientosAB`

Adentro, el mismo botón alterna entre mapa y calendario. Arriba salen las
solicitudes pendientes con **Aprobar** / **Rechazar**. Abajo, el calendario
semanal con los 18 estacionamientos como filas y lunes a viernes como columnas:

- Celda vacía → nueva reserva, prellenada con ese espacio y día.
- Reserva existente → editar o eliminar.
- Etiqueta **«fijo»** → abre el diálogo para liberar ese espacio por un período.
  Ahí mismo se listan los períodos ya liberados, con opción de quitarlos.
- Horas vacías = reserva de día completo.

La sesión de administrador dura mientras la pestaña esté abierta.

### Cómo funciona la liberación temporal

Un espacio fijo se vuelve reservable **solo si el período liberado cubre todos
los días** del rango que el usuario seleccionó. Si alguien selecciona lunes a
viernes y la liberación es solo miércoles y jueves, el espacio sigue apareciendo
como fijo: así se evita que se reserve la semana completa pisando los días del
dueño. Para reservar el miércoles hay que seleccionar solo el miércoles.

Dos liberaciones consecutivas se encadenan (3–4 de agosto más 5–7 de agosto
cubren del 3 al 7). Si queda un día sin cubrir en el medio, no.

---

## 5. Cambiar asignaciones

Están en `index.html`, bloque `CONFIG`:

```js
const FIXED = {
  '3002':'Erwin Grune',
  '3003':'Juan Pablo Vogt',
  '3011':'Max Emden',
  '3012':'Juan Pablo Lavin',
  '6017':'Catalina Vives',
  '6018':'Francisco Suric',
  '6020':'Natalia Cwiklitzer',
  '6021':'Braulio Osorio'
};

const PARTIAL = {};
```

`PARTIAL` sirve para asignaciones válidas solo ciertos días. Está vacío porque
Braulio pasó a asignación permanente, pero el mecanismo sigue funcionando: para
asignar un espacio a alguien solo lunes, miércoles y jueves se agrega
`'7040':{name:'Nombre',days:[1,3,4]}`, con `1`=lunes … `5`=viernes.

Otros parámetros en el mismo bloque:

```js
const HOURLY = new Set(['6002','6003','7040','7041']);  // espacios por hora
const HOURS  = [9,10,11,12,13,14,15,16,17];             // bloques 9:00 a 18:00
const MAX_DIRECT_HOURS = 1;                             // más que esto pide aprobación
```

Los botones para elegir espacio en la vista horaria se generan a partir de
`HOURLY`, así que agregar o quitar uno ahí es suficiente: no hay que tocar el HTML.

Para mover un espacio en el mapa, edita `MAP_CELLS` (columna y fila de cada uno).

---

## Advertencia de seguridad

La clave de administrador está escrita en el HTML: cualquiera que abra el código
fuente de la página puede leerla. Para un estacionamiento de oficina el peor caso
es que alguien mueva una reserva, así que es un riesgo probablemente aceptable —
pero es una decisión consciente, no un descuido.

Lo mismo con la `anon key`: es pública por diseño, pero como las políticas RLS
son abiertas, cualquiera con esa clave puede escribir en las tablas.

Si en algún momento eso deja de ser tolerable, la solución correcta es Supabase
Auth con un rol de administrador y políticas RLS reales, y el frontend deja de
contener la clave.
