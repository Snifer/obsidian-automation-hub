#  Linear Calendar

Spanish Version: [`README.md`](README.md) 

The **Linear Calendar** is a visual tool designed to display the chronology of your projects and events continuously, avoiding the fragmentation of traditional monthly calendars. It allows you to visualize workload, milestones, and dependencies across 365 days at a single glance.

### Tutorial
If you want to see how to install, configure, and use it step-by-step, watch this video on my channel:
**[Organize your entire year in Obsidian with the Linear Calendar](https://www.youtube.com/watch?v=8iGNVyf15Mc)**

Enable auto dubbed English in YouTube.

---

### Requirements

To run this script, you need the **Dataview** community plugin installed and the following settings enabled:

1. Go to `Settings` > `Dataview`.
2. Enable **Enable JavaScript Queries**.
3. Enable **Enable Inline JavaScript Queries**.
4. (Recommended) Set **Refresh Interval** to `2500`ms to avoid system overhead.

---

###  Key Features

* **Flexible Views:** Interactive buttons to switch focus between **Annual, Semi-annual, or Quarterly** views.
* **Workload Detector:** If 3 or more projects overlap on the same day, the cell automatically turns red to warn of saturation.
* **Interactivity:** * Click any project bar to open the original note.
    * Hover over **Milestone** diamonds to see descriptions.
    * Quick filters by **Status** and **Priority (Color)**.
* **Search:** Real-time search filter by project name.

---

### Installation

1. Create a new note in your Obsidian vault.
2. Copy the code from the [`Linear-Calendar.md`](Linear-Calendar.md) file in this repository.
3. Paste it inside a `dataviewjs` code block.

#### Note Properties (Frontmatter)
Your notes must contain at least the following fields in English:

```yaml
---
title: Linear Calendar Project
start_date: 2026-01-01
end_date: 2026-03-31
status: In Progress
color: #4a9e0f
---
```

##### Advanced Options (Dependencies and Milestones):
You can also track what is blocking a project and specific milestones:

---
title: Linear Calendar Project
start_date: 2026-01-01
end_date: 2026-03-31
status: In Progress
color: #4a9e0f
blocked_by:
  - [[Other Note]]
milestones:
  2026-01-15: "Initial Setup"
  2026-02-20: "Mid-project Review"
---

