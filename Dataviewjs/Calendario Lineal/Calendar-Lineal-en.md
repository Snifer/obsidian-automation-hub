


```dataviewjs
const notesFolder = "/";  
const calendarYear = 2026;
const monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];

const allNotes = dv.pages(`"${notesFolder}"`)
    .filter(p => p.start_date && p.end_date)
    .map(p => ({
        name: p.title || p.file.name || "Untitled",
        path: p.file.path,
        start: window.moment(p.start_date.toString()),
        end: window.moment(p.end_date.toString()),
        color: p.color || null,
        icon: p.icon || null, 
        status: p.status || "Pending",
        milestones: p.milestones || {},
        blocked_by: p.blocked_by ? p.blocked_by.path : null
    }));

const uniqueStatuses = ["All", ...new Set(allNotes.map(n => n.status))];
const uniqueColors = ["All", ...new Set(allNotes.filter(n => n.color).map(n => n.color))];

if (!window.calState) {
    window.calState = {
        statusFilter: "All",
        colorFilter: "All",
        searchQuery: "",
        view: "Yearly"
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
    const today = window.moment();
    const wrapper = mainContainer.createEl("div", { cls: "cal-main" });
    const controls = wrapper.createEl("div", { cls: "cal-controls" });
    const row1 = controls.createEl("div", { cls: "control-row" });


    const statusSection = row1.createEl("div", { cls: "filter-group" });
    statusSection.createEl("span", { text: "Status", cls: "filter-label" });
    uniqueStatuses.forEach(status => {
        const btn = statusSection.createEl("button", { text: status, cls: `cal-btn ${window.calState.statusFilter === status ? "active" : ""}` });
        btn.onclick = () => { window.calState.statusFilter = status; render(); };
    });


    const colorSection = row1.createEl("div", { cls: "filter-group" });
    colorSection.createEl("span", { text: "Priority", cls: "filter-label", attr: { style: "margin-left:10px" } });
    uniqueColors.forEach(color => {
        if (color === "All") {
            const btn = colorSection.createEl("button", { text: "All", cls: `cal-btn ${window.calState.colorFilter === "All" ? "active" : ""}` });
            btn.onclick = () => { window.calState.colorFilter = "All"; render(); };
        } else {
            const dot = colorSection.createEl("span", { cls: `color-dot ${window.calState.colorFilter === color ? "active" : ""}` });
            dot.style.backgroundColor = color;
            dot.onclick = () => { window.calState.colorFilter = color; render(); };
        }
    });


    const search = row1.createEl("input", { cls: "search-input", attr: { placeholder: "🔍 Search..." } });
    search.value = window.calState.searchQuery;
    search.oninput = (e) => { window.calState.searchQuery = e.target.value; render(); };

    const row2 = controls.createEl("div", { cls: "control-row" });
    

    const viewSection = row2.createEl("div", { cls: "filter-group" });
    ["Yearly", "Half-Year", "Quarterly"].forEach(v => {
        const btn = viewSection.createEl("button", { text: v, cls: `cal-btn ${window.calState.view === v ? "active" : ""}` });
        btn.onclick = () => { window.calState.view = v; render(); };
    });


    const btnToday = row2.createEl("button", { text: "📍 Go to Today", cls: "cal-btn active" });
    btnToday.onclick = () => {
        const target = document.querySelector(".is-today");
        if (target) target.scrollIntoView({ behavior: "smooth", block: "center", inline: "center" });
    };


    const filteredNotes = allNotes.filter(n => {
        const matchStatus = window.calState.statusFilter === "All" || n.status === window.calState.statusFilter;
        const matchColor = window.calState.colorFilter === "All" || n.color === window.calState.colorFilter;
        
        const noteName = n.name ? String(n.name).toLowerCase() : "";
        const searchTerms = window.calState.searchQuery ? window.calState.searchQuery.toLowerCase() : "";
        const matchSearch = noteName.includes(searchTerms);
        
        return matchStatus && matchColor && matchSearch;
    });


    let monthsToShow = Array.from({length: 12}, (_, i) => i);
    if (window.calState.view === "Quarterly") {
        const q = Math.floor(today.month() / 3);
        monthsToShow = [q*3, q*3+1, q*3+2];
    } else if (window.calState.view === "Half-Year") {
        const s = Math.floor(today.month() / 6);
        monthsToShow = [s*6, s*6+1, s*6+2, s*6+3, s*6+4, s*6+5];
    }


    const scrollArea = wrapper.createEl("div", { cls: "cal-scroll" });
    const table = scrollArea.createEl("table", { cls: "lin-table" });
    
    const headRow = table.createEl("tr");
    headRow.createEl("th", { text: calendarYear, cls: "month-label" });
    for(let i=1; i<=31; i++) headRow.createEl("th", { text: i.toString().padStart(2,'0'), cls: "lin-th" });

    monthsToShow.forEach(mIdx => {
        const row = table.createEl("tr");
        row.createEl("td", { text: monthNames[mIdx], cls: "month-label" });

        for (let day = 1; day <= 31; day++) {
            const date = window.moment([calendarYear, mIdx, day]);
            const cell = row.createEl("td", { cls: "day-cell" });

            if (!date.isValid() || date.month() !== mIdx) {
                cell.style.background = "var(--background-modifier-border-dark)";
                cell.style.opacity = "0.2";
                continue;
            }

            if (date.day() === 0 || date.day() === 6) cell.addClass("weekend");
            if (date.isSame(today, 'day')) cell.addClass("is-today");

            const projectsToday = filteredNotes.filter(n => date.isBetween(n.start, n.end, 'day', '[]'));
            if (projectsToday.length > 0) {
                cell.createEl("span", { text: projectsToday.length, cls: "load-tag" });
                if (projectsToday.length >= 3) cell.addClass("high-load");
            }

            const evWrapper = cell.createEl("div", { cls: "event-wrapper" });
            projectsToday.forEach(note => {
                const bar = evWrapper.createEl("div", { cls: "event-bar" });
                const isStart = date.isSame(note.start, 'day');
                
                if (note.icon) {
                    bar.addClass("icon-only");
                    bar.createEl("span", { text: note.icon });
                    if (isStart) {
                        bar.style.justifyContent = "flex-start";
                        bar.createEl("span", { text: " " + note.name, attr: { style: "font-size: 9px; font-weight: bold; margin-left: 4px; white-space: nowrap;" } });
                    }
                } else if (note.color) {
                    bar.style.backgroundColor = note.color;
                    if (isStart) {
                        bar.createEl("span", { text: (note.blocked_by ? "🔗 " : "") + note.name, attr: { style: "font-weight: bold; white-space: nowrap;" } });
                    }
                } else {
                    bar.style.backgroundColor = "var(--background-modifier-border)";
                    if (isStart) bar.createEl("span", { text: note.name, attr: { style: "white-space: nowrap;" } });
                }
                
                const fKey = date.format("YYYY-MM-DD");
                if (note.milestones[fKey]) bar.createEl("div", { cls: "milestone-diamond", title: note.milestones[fKey] });
                
                bar.onclick = () => app.workspace.getLeaf().openFile(app.vault.getAbstractFileByPath(note.path));
            });
        }
    });
}

const mainContainer = dv.el("div", "");
render();
```

