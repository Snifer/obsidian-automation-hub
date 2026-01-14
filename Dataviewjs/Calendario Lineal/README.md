#   Calendario Lineal (Linear Calendar)

English Versión: 

El **Calendario Lineal** es una herramienta visual diseñada para mostrar la cronología de tus proyectos y eventos de forma continua, evitando la fragmentación de los calendarios mensuales tradicionales. Permitiendo visualizar de un solo vistazo la carga de trabajo, hitos importantes y dependencias entre notas a lo largo de 365 días.

### Tutorial 
Si quieres ver cómo instalarlo, configurarlo y cómo funciona paso a paso, mira este video en mi canal:
**[Organiza tu año completo en Obsidian con el Calendario Lineal](https://www.youtube.com/watch?v=8iGNVyf15Mc)**

---

###  Requisitos

Para que este script funcione, necesitas tener instalado el plugin de comunidad **Dataview** y configurar lo siguiente:

1. Ve a `Settings` > `Dataview`.
2. Habilita la opción **Enable JavaScript Queries**.
3. Habilita la opción **Enable Inline JavaScript Queries**.
4. (Recomendado) Ajustar el **Refresh Interval** a `2500`ms para evitar sobrecarga del sistema.

---

###  Características Principales

* **Vistas Flexibles:** Botones interactivos para cambiar el enfoque entre vista **Anual, Semestral o Trimestral**.
* **Detector de Carga:** Si acumulas 3 o más proyectos en un mismo día, la celda se marcará automáticamente en rojo para alertar sobre saturación.
* **Interactividad:** * Haz clic en cualquier barra de proyecto para abrir la nota original.
 * Pasa el ratón sobre los diamantes de **Hitos** para ver la descripción.
 * Filtros rápidos por **Estado** y **Prioridad (Color)**.
* **Buscador:** Filtro de búsqueda en tiempo real por nombre de proyecto.

---

### Instalación

1. Crea una nueva nota en tu Obsidian.
2. Copia el código contenido en el archivo [`Calendario-Lineal.md`](Calendario-Lineal.md) de este repositorio.
3. Pégalo dentro de un bloque de código de tipo `dataviewjs`:

Recuerda que tus notas deben contener minimamente los siguientes campos en su frontmatter ( propiedades).


```
---
titulo: Calendario Lineal
fecha_inicio: 2026-01-01
fecha_fin: 2026-03-31
estado: En curso
color: #4a9e0f

---
```

Adicionalmente puedes registrar que es lo que esta bloqueando al proyecto e Hitos para que se visualicen. 

```
---
titulo: Calendario Lineal
fecha_inicio: 2026-01-01
fecha_fin: 2026-03-31
estado: En curso
color: #4a9e0f
bloqueado_por:
  - sniferl4bs
hitos:
  2026-01-15: Apoyar a SniferL4bs
  2026-02-20: mitad del curso
---
```

Si no deseas usar color como referencia y si un icono solo debes de usar la propiedad `icono: emoji`
