Calendario Lineal en Obsidian  te recomiendo leer el fichero README.md para identificar el uso y seguir el video. 



```dataviewjs
const carpetaNotas = "/";    // Directorio que lee el script
const anioCalendario = 2026; // Año que se desea mostrar
const mesesNombres = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];

const todasLasNotas = dv.pages(`"${carpetaNotas}"`)
    .filter(p => p.fecha_inicio && p.fecha_fin)
    .map(p => ({
        name: p.titulo || p.file.name,
        path: p.file.path,
        start: window.moment(p.fecha_inicio.toString()),
        end: window.moment(p.fecha_fin.toString()),
        color: p.color || "#4a9eff",
        estado: p.estado || "Pendiente",
        hitos: p.hitos || {},
        bloqueado_por: p.bloqueado_por ? p.bloqueado_por.path : null
    }));

const estadosUnicos = ["Todos", ...new Set(todasLasNotas.map(n => n.estado))];
const coloresUnicos = ["Todos", ...new Set(todasLasNotas.map(n => n.color))];

if (!window.calState) {
    window.calState = {
        filtroEstado: "Todos",
        filtroColor: "Todos",
        busqueda: "",
        vista: "Anual"
    };
}


const style = document.createElement('style');
style.textContent = `
    .cal-main { font-family: 'Inter', system-ui, sans-serif; background: var(--background-primary); padding: 10px; border-radius: 12px; }
    
    /* Panel de Controles Superior */
    .cal-controls { display: flex; flex-direction: column; gap: 12px; background: var(--background-secondary-alt); padding: 15px; border-radius: 12px; border: 1px solid var(--background-modifier-border); margin-bottom: 15px; }
    .control-row { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px; }
    .filter-group { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
    .filter-label { font-size: 0.65rem; font-weight: bold; text-transform: uppercase; color: var(--text-faint); letter-spacing: 0.5px; }
    
    /* Botones y Inputs */
    .cal-btn { cursor: pointer; padding: 4px 12px; border-radius: 20px; border: 1px solid var(--background-modifier-border); background: var(--background-primary); color: var(--text-normal); font-size: 0.75rem; transition: 0.2s; }
    .cal-btn:hover { border-color: var(--interactive-accent); }
    .cal-btn.active { background: var(--interactive-accent); color: white; border-color: var(--interactive-accent); }
    .search-input { padding: 5px 12px; border-radius: 20px; border: 1px solid var(--background-modifier-border); background: var(--background-primary); font-size: 0.75rem; width: 180px; }

    .color-dot { width: 18px; height: 18px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; transition: 0.2s; }
    .color-dot.active { border-color: var(--text-normal); transform: scale(1.2); box-shadow: 0 0 5px rgba(0,0,0,0.3); }

    /* Estructura de Tabla */
    .cal-scroll { overflow-x: auto; border: 1px solid var(--background-modifier-border); border-radius: 8px; scroll-behavior: smooth; }
    .lin-table { border-collapse: collapse; table-layout: fixed; width: 100%; border-spacing: 0; }
    .lin-th { background: var(--background-secondary); font-size: 10px; border: 0.5px solid var(--background-modifier-border); width: 32px; height: 30px; color: var(--text-muted); }
    .month-label { position: sticky; left: 0; background: var(--background-secondary); z-index: 25; width: 90px; font-weight: bold; border-right: 3px solid var(--interactive-accent); text-align: center; font-size: 0.85rem; }
    
    .day-cell { border: 0.5px solid var(--background-modifier-border-soft); height: 55px; position: relative; padding: 0; vertical-align: top; }
    .weekend { background: var(--background-modifier-border-soft); opacity: 0.5; }
    .is-today { border: 2.5px solid var(--interactive-accent) !important; z-index: 10; background: rgba(var(--interactive-accent-rgb), 0.1); }

    /* Eventos y Carga */
    .event-wrapper { display: flex; flex-direction: column; gap: 2px; padding: 4px 0; }
    .event-bar { height: 15px; font-size: 9px; color: white; cursor: pointer; display: flex; align-items: center; padding: 0 5px; border-radius: 3px; position: relative; z-index: 5; margin: 0 1px; }
    .milestone-diamond { position: absolute; left: 50%; top: 50%; transform: translate(-50%, -50%) rotate(45deg); width: 6px; height: 6px; background: white; border: 1px solid black; }
    .load-tag { position: absolute; bottom: 2px; right: 2px; font-size: 9px; font-weight: bold; opacity: 0.4; }
    .high-load { background-color: rgba(255, 0, 0, 0.1) !important; }
`;
document.head.appendChild(style);

function render() {
    mainContainer.innerHTML = "";
    const hoy = window.moment();
    const wrapper = mainContainer.createEl("div", { cls: "cal-main" });
    const controls = wrapper.createEl("div", { cls: "cal-controls" });
    const row1 = controls.createEl("div", { cls: "control-row" });
    
    const filterSection = row1.createEl("div", { cls: "filter-group" });
    

    filterSection.createEl("span", { text: "Estado", cls: "filter-label" });
    estadosUnicos.forEach(est => {
        const btn = filterSection.createEl("button", { text: est, cls: `cal-btn ${window.calState.filtroEstado === est ? "active" : ""}` });
        btn.onclick = () => { window.calState.filtroEstado = est; render(); };
    });


    filterSection.createEl("span", { text: "Prioridad", cls: "filter-label", attr: { style: "margin-left:10px" } });
    coloresUnicos.forEach(col => {
        if (col === "Todos") {
            const btn = filterSection.createEl("button", { text: "Todos", cls: `cal-btn ${window.calState.filtroColor === "Todos" ? "active" : ""}` });
            btn.onclick = () => { window.calState.filtroColor = "Todos"; render(); };
        } else {
            const dot = filterSection.createEl("span", { cls: `color-dot ${window.calState.filtroColor === col ? "active" : ""}` });
            dot.style.backgroundColor = col;
            dot.onclick = () => { window.calState.filtroColor = col; render(); };
        }
    });


    const search = row1.createEl("input", { cls: "search-input", attr: { placeholder: "🔍 Buscar proyecto..." } });
    search.value = window.calState.busqueda;
    search.oninput = (e) => { window.calState.busqueda = e.target.value; render(); };

    const row2 = controls.createEl("div", { cls: "control-row" });
    
    const viewSection = row2.createEl("div", { cls: "filter-group" });
    viewSection.createEl("span", { text: "Enfoque", cls: "filter-label" });
    ["Anual", "Semestral", "Trimestral"].forEach(v => {
        const btn = viewSection.createEl("button", { text: v, cls: `cal-btn ${window.calState.vista === v ? "active" : ""}` });
        btn.onclick = () => { window.calState.vista = v; render(); };
    });

    const btnHoy = row2.createEl("button", { text: "📍 Ir a Hoy", cls: "cal-btn active" });
    btnHoy.onclick = () => {
        const target = document.querySelector(".is-today");
        if (target) target.scrollIntoView({ behavior: "smooth", block: "center", inline: "center" });
    };


    const filtradas = todasLasNotas.filter(n => {
        const matchEstado = window.calState.filtroEstado === "Todos" || n.estado === window.calState.filtroEstado;
        const matchColor = window.calState.filtroColor === "Todos" || n.color === window.calState.filtroColor;
        const matchBusqueda = n.name.toLowerCase().includes(window.calState.busqueda.toLowerCase());
        return matchEstado && matchColor && matchBusqueda;
    });

    let mesesAMostrar = Array.from({length: 12}, (_, i) => i);
    if (window.calState.vista === "Trimestral") {
        const q = Math.floor(hoy.month() / 3);
        mesesAMostrar = [q*3, q*3+1, q*3+2];
    } else if (window.calState.vista === "Semestral") {
        const s = Math.floor(hoy.month() / 6);
        mesesAMostrar = [s*6, s*6+1, s*6+2, s*6+3, s*6+4, s*6+5];
    }


    const scrollArea = wrapper.createEl("div", { cls: "cal-scroll" });
    const table = scrollArea.createEl("table", { cls: "lin-table" });
    

    const headRow = table.createEl("tr");
    headRow.createEl("th", { text: anioCalendario, cls: "month-label" });
    for(let i=1; i<=31; i++) headRow.createEl("th", { text: i.toString().padStart(2,'0'), cls: "lin-th" });

    mesesAMostrar.forEach(mIdx => {
        const row = table.createEl("tr");
        row.createEl("td", { text: mesesNombres[mIdx], cls: "month-label" });

        for (let dia = 1; dia <= 31; dia++) {
            const fecha = window.moment([anioCalendario, mIdx, dia]);
            const cell = row.createEl("td", { cls: "day-cell" });

            if (!fecha.isValid() || fecha.month() !== mIdx) {
                cell.style.background = "var(--background-modifier-border-dark)";
                cell.style.opacity = "0.2";
                continue;
            }

            if (fecha.day() === 0 || fecha.day() === 6) cell.addClass("weekend");
            if (fecha.isSame(hoy, 'day')) cell.addClass("is-today");

            const proyectosHoy = filtradas.filter(n => fecha.isBetween(n.start, n.end, 'day', '[]'));
            
            if (proyectosHoy.length > 0) {
                cell.createEl("span", { text: proyectosHoy.length, cls: "load-tag" });
                if (proyectosHoy.length >= 3) cell.addClass("high-load");
            }

            const wrapper = cell.createEl("div", { cls: "event-wrapper" });
            proyectosHoy.forEach(nota => {
                const bar = wrapper.createEl("div", { cls: "event-bar" });
                bar.style.backgroundColor = nota.color;
                
                const fKey = fecha.format("YYYY-MM-DD");
                if (nota.hitos[fKey]) bar.createEl("div", { cls: "milestone-diamond", title: nota.hitos[fKey] });
                
                if (fecha.isSame(nota.start, 'day')) {
                    const label = bar.createEl("span", { text: (nota.bloqueado_por ? "🔗 " : "") + nota.name });
                    label.style.fontWeight = "bold";
                }
                
                bar.onclick = () => app.workspace.getLeaf().openFile(app.vault.getAbstractFileByPath(nota.path));
            });
        }
    });
}

const mainContainer = dv.el("div", "");
render();

```

En el caso de que quieran usar tanto un icono o emoji se tiene esta opción que es una mejora sugerida por Reddit. 

```
const carpetaNotas = "/";  
const anioCalendario = 2026;
const mesesNombres = ["Enero", "Febrero", "Marzo", "Abril", "Mayo", "Junio", "Julio", "Agosto", "Septiembre", "Octubre", "Noviembre", "Diciembre"];
const todasLasNotas = dv.pages(`"${carpetaNotas}"`)
    .filter(p => p.fecha_inicio && p.fecha_fin)
    .map(p => ({
        name: p.titulo || p.file.name,
        path: p.file.path,
        start: window.moment(p.fecha_inicio.toString()),
        end: window.moment(p.fecha_fin.toString()),
        color: p.color || null,
        icono: p.icono || null, 
        estado: p.estado || "Pendiente",
        hitos: p.hitos || {},
        bloqueado_por: p.bloqueado_por ? p.bloqueado_por.path : null
    }));

const estadosUnicos = ["Todos", ...new Set(todasLasNotas.map(n => n.estado))];
const coloresUnicos = ["Todos", ...new Set(todasLasNotas.filter(n => n.color).map(n => n.color))];

if (!window.calState) {
    window.calState = {
        filtroEstado: "Todos",
        filtroColor: "Todos",
        busqueda: "",
        vista: "Anual"
    };
}
const style = document.createElement('style');
style.textContent = `
    .cal-main { font-family: 'Inter', system-ui, sans-serif; background: var(--background-primary); padding: 10px; border-radius: 12px; }
    .cal-controls { display: flex; flex-direction: column; gap: 12px; background: var(--background-secondary-alt); padding: 15px; border-radius: 12px; border: 1px solid var(--background-modifier-border); margin-bottom: 15px; }
    .control-row { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px; }
    .filter-group { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
    .filter-label { font-size: 0.65rem; font-weight: bold; text-transform: uppercase; color: var(--text-faint); letter-spacing: 0.5px; }
    .cal-btn { cursor: pointer; padding: 4px 12px; border-radius: 20px; border: 1px solid var(--background-modifier-border); background: var(--background-primary); color: var(--text-normal); font-size: 0.75rem; transition: 0.2s; }
    .cal-btn.active { background: var(--interactive-accent); color: white; border-color: var(--interactive-accent); }
    .search-input { padding: 5px 12px; border-radius: 20px; border: 1px solid var(--background-modifier-border); background: var(--background-primary); font-size: 0.75rem; width: 180px; }
    .color-dot { width: 18px; height: 18px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; transition: 0.2s; }
    .color-dot.active { border-color: var(--text-normal); transform: scale(1.2); }
    .cal-scroll { overflow-x: auto; border: 1px solid var(--background-modifier-border); border-radius: 8px; }
    .lin-table { border-collapse: collapse; table-layout: fixed; width: 100%; border-spacing: 0; }
    .lin-th { background: var(--background-secondary); font-size: 10px; border: 0.5px solid var(--background-modifier-border); width: 32px; height: 30px; color: var(--text-muted); }
    .month-label { position: sticky; left: 0; background: var(--background-secondary); z-index: 25; width: 90px; font-weight: bold; border-right: 3px solid var(--interactive-accent); text-align: center; font-size: 0.85rem; }
    .day-cell { border: 0.5px solid var(--background-modifier-border-soft); height: 55px; position: relative; padding: 0; vertical-align: top; }
    .weekend { background: var(--background-modifier-border-soft); opacity: 0.5; }
    .is-today { border: 2.5px solid var(--interactive-accent) !important; z-index: 10; background: rgba(var(--interactive-accent-rgb), 0.1); }
    .event-wrapper { display: flex; flex-direction: column; gap: 2px; padding: 4px 0; }
    
    .event-bar { height: 18px; font-size: 10px; color: white; cursor: pointer; display: flex; align-items: center; justify-content: center; padding: 0 4px; border-radius: 4px; position: relative; margin: 0 1px; }

    .event-bar.icon-only { background: transparent !important; color: var(--text-normal); border: none; font-size: 14px; }
    
    .milestone-diamond { position: absolute; left: 50%; top: -2px; transform: translateX(-50%) rotate(45deg); width: 7px; height: 7px; background: white; border: 1px solid black; z-index: 10; }
    .load-tag { position: absolute; bottom: 2px; right: 2px; font-size: 9px; font-weight: bold; opacity: 0.4; }
    .high-load { background-color: rgba(255, 0, 0, 0.1) !important; }
`;
document.head.appendChild(style);

function render() {
    mainContainer.innerHTML = "";
    const hoy = window.moment();
    const wrapper = mainContainer.createEl("div", { cls: "cal-main" });
    const controls = wrapper.createEl("div", { cls: "cal-controls" });
    const row1 = controls.createEl("div", { cls: "control-row" });
    

    const filterSection = row1.createEl("div", { cls: "filter-group" });
    filterSection.createEl("span", { text: "Estado", cls: "filter-label" });
    estadosUnicos.forEach(est => {
        const btn = filterSection.createEl("button", { text: est, cls: `cal-btn ${window.calState.filtroEstado === est ? "active" : ""}` });
        btn.onclick = () => { window.calState.filtroEstado = est; render(); };
    });

    filterSection.createEl("span", { text: "Prioridad", cls: "filter-label", attr: { style: "margin-left:10px" } });
    coloresUnicos.forEach(col => {
        if (col === "Todos") {
            const btn = filterSection.createEl("button", { text: "Todos", cls: `cal-btn ${window.calState.filtroColor === "Todos" ? "active" : ""}` });
            btn.onclick = () => { window.calState.filtroColor = "Todos"; render(); };
        } else {
            const dot = filterSection.createEl("span", { cls: `color-dot ${window.calState.filtroColor === col ? "active" : ""}` });
            dot.style.backgroundColor = col;
            dot.onclick = () => { window.calState.filtroColor = col; render(); };
        }
    });

    const search = row1.createEl("input", { cls: "search-input", attr: { placeholder: "🔍 Buscar..." } });
    search.value = window.calState.busqueda;
    search.oninput = (e) => { window.calState.busqueda = e.target.value; render(); };

    const row2 = controls.createEl("div", { cls: "control-row" });
    const viewSection = row2.createEl("div", { cls: "filter-group" });
    ["Anual", "Semestral", "Trimestral"].forEach(v => {
        const btn = viewSection.createEl("button", { text: v, cls: `cal-btn ${window.calState.vista === v ? "active" : ""}` });
        btn.onclick = () => { window.calState.vista = v; render(); };
    });

    const btnHoy = row2.createEl("button", { text: "📍 Ir a Hoy", cls: "cal-btn active" });
    btnHoy.onclick = () => {
        const target = document.querySelector(".is-today");
        if (target) target.scrollIntoView({ behavior: "smooth", block: "center", inline: "center" });
    };

    const filtradas = todasLasNotas.filter(n => {
        const matchEstado = window.calState.filtroEstado === "Todos" || n.estado === window.calState.filtroEstado;
        const matchColor = window.calState.filtroColor === "Todos" || n.color === window.calState.filtroColor;
        const matchBusqueda = n.name.toLowerCase().includes(window.calState.busqueda.toLowerCase());
        return matchEstado && matchColor && matchBusqueda;
    });

    let mesesAMostrar = Array.from({length: 12}, (_, i) => i);
    if (window.calState.vista === "Trimestral") {
        const q = Math.floor(hoy.month() / 3);
        mesesAMostrar = [q*3, q*3+1, q*3+2];
    } else if (window.calState.vista === "Semestral") {
        const s = Math.floor(hoy.month() / 6);
        mesesAMostrar = [s*6, s*6+1, s*6+2, s*6+3, s*6+4, s*6+5];
    }

    const scrollArea = wrapper.createEl("div", { cls: "cal-scroll" });
    const table = scrollArea.createEl("table", { cls: "lin-table" });
    const headRow = table.createEl("tr");
    headRow.createEl("th", { text: anioCalendario, cls: "month-label" });
    for(let i=1; i<=31; i++) headRow.createEl("th", { text: i.toString().padStart(2,'0'), cls: "lin-th" });

    mesesAMostrar.forEach(mIdx => {
        const row = table.createEl("tr");
        row.createEl("td", { text: mesesNombres[mIdx], cls: "month-label" });

        for (let dia = 1; dia <= 31; dia++) {
            const fecha = window.moment([anioCalendario, mIdx, dia]);
            const cell = row.createEl("td", { cls: "day-cell" });

            if (!fecha.isValid() || fecha.month() !== mIdx) {
                cell.style.background = "var(--background-modifier-border-dark)";
                cell.style.opacity = "0.2";
                continue;
            }

            if (fecha.day() === 0 || fecha.day() === 6) cell.addClass("weekend");
            if (fecha.isSame(hoy, 'day')) cell.addClass("is-today");

            const proyectosHoy = filtradas.filter(n => fecha.isBetween(n.start, n.end, 'day', '[]'));
            if (proyectosHoy.length > 0) {
                cell.createEl("span", { text: proyectosHoy.length, cls: "load-tag" });
                if (proyectosHoy.length >= 3) cell.addClass("high-load");
            }

            const evWrapper = cell.createEl("div", { cls: "event-wrapper" });
            proyectosHoy.forEach(nota => {
                const bar = evWrapper.createEl("div", { cls: "event-bar" });
                const esInicio = fecha.isSame(nota.start, 'day');
                
                if (nota.icono) {
                    bar.addClass("icon-only");
                    bar.createEl("span", { text: nota.icono });
                    
                    if (esInicio) {
                        bar.style.justifyContent = "flex-start";
                        bar.createEl("span", { text: " " + nota.name, attr: { style: "font-size: 9px; font-weight: bold; margin-left: 4px; white-space: nowrap;" } });
                    }
                } else if (nota.color) {
                    bar.style.backgroundColor = nota.color;
                    if (esInicio) {
                        bar.createEl("span", { text: (nota.bloqueado_por ? "🔗 " : "") + nota.name, attr: { style: "font-weight: bold; white-space: nowrap;" } });
                    }
                } else {
                    bar.style.backgroundColor = "var(--background-modifier-border)";
                    if (esInicio) bar.createEl("span", { text: nota.name, attr: { style: "white-space: nowrap;" } });
                }
                
                const fKey = fecha.format("YYYY-MM-DD");
                if (nota.hitos[fKey]) bar.createEl("div", { cls: "milestone-diamond", title: nota.hitos[fKey] });
                
                bar.onclick = () => app.workspace.getLeaf().openFile(app.vault.getAbstractFileByPath(nota.path));
            });
        }
    });
}

const mainContainer = dv.el("div", "");
render();
```

