# Energy Job Radar v2 — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace `index.html` with a full-featured job tracking dashboard: keyword search, multi-facet filtering, auto-priority scoring, application tracker, private notes, change timeline, card/table toggle.

**Architecture:** Single static HTML file. Fetches `data/jobs.json`, computes priority scores, diffs against cached snapshot for change detection, persists user data (status/notes) to localStorage. Three-column desktop layout, collapsible panels on mobile.

**Tech Stack:** HTML5, CSS3 (CSS variables, Grid, Flexbox), vanilla JS (ES2020+), no frameworks, no build step.

**Source file:** `F:\job-app\energy-job-radar\index.html`

---

## File Responsibilities

| File | Role |
|------|------|
| `index.html` | Everything — markup, styles, logic. Replaces v1 entirely. |
| `data/jobs.json` | Unchanged data source |
| `manifest.json`, `sw.js`, icons | Unchanged PWA assets |

---

### Task 1: Scaffold — HTML structure + CSS foundation + color system

**Files:**
- Replace: `F:\job-app\energy-job-radar\index.html`

Build the full HTML skeleton and CSS foundation. No JS logic yet — just static layout with hardcoded sample cards to verify the three-column grid and color system work.

- [ ] **Step 1: Write the HTML skeleton**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
  <meta name="theme-color" content="#1F3864">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="岗位雷达">
  <meta name="description" content="华电动力工程硕士专属岗位看板">
  <link rel="manifest" href="/manifest.json">
  <link rel="apple-touch-icon" href="/icon-192.png">
  <title>能源动力岗位雷达 v2</title>
  <style>
    /* CSS goes here — see next step */
  </style>
</head>
<body>

<!-- Header -->
<header class="header" id="header">
  <div class="header-top">
    <h1>⚡ 能源动力岗位雷达 <span class="version">v2</span></h1>
    <span class="updated" id="updateTime">加载中...</span>
  </div>
  <div class="header-bar">
    <input type="search" class="search-input" id="searchInput" placeholder="搜索公司、岗位、地点、专业...">
    <div class="change-summary" id="changeSummary"></div>
  </div>
  <div class="filter-quick" id="filterQuick">
    <!-- category pills, type pills, deadline range, view toggle -->
  </div>
</header>

<!-- Main: three columns -->
<div class="main" id="main">
  <!-- Left panel -->
  <aside class="panel-left" id="panelLeft">
    <div class="panel-section" id="priorityFilters"></div>
    <div class="panel-section" id="categoryFilters"></div>
    <div class="panel-section" id="statusFilters"></div>
    <div class="panel-section" id="cityFilters"></div>
  </aside>

  <!-- Center list -->
  <section class="panel-center" id="panelCenter">
    <div class="list-toolbar">
      <span class="result-count" id="resultCount"></span>
      <div class="view-toggle">
        <button class="view-btn active" data-view="card" id="viewCard">卡片</button>
        <button class="view-btn" data-view="table" id="viewTable">表格</button>
      </div>
    </div>
    <div class="active-filters" id="activeFilters"></div>
    <div class="job-list" id="jobList"></div>
  </section>

  <!-- Right panel -->
  <aside class="panel-right" id="panelRight">
    <div class="detail-empty" id="detailEmpty">← 点击岗位查看详情</div>
    <div class="detail-content" id="detailContent" style="display:none">
      <!-- dynamic fill -->
    </div>
  </aside>
</div>

<!-- Footer -->
<footer class="footer" id="footer">
  <span id="footerStats"></span>
</footer>

<!-- Mobile overlays -->
<div class="overlay" id="overlay"></div>

<script>
  /* JS goes here — see later tasks */
</script>

</body>
</html>
```

- [ ] **Step 2: Write the CSS — variables and reset**

```css
:root {
  --bg: #f1f5f9;
  --card: #ffffff;
  --text: #1e293b;
  --sub: #64748b;
  --primary: #1F3864;
  --accent: #3b82f6;
  --border: #e2e8f0;
  --shadow: 0 1px 3px rgba(0,0,0,0.06);

  /* Priority colors */
  --s-tier: #dc2626;
  --a-tier: #ea580c;
  --b-tier: #2563eb;
  --c-tier: #6b7280;

  /* Status colors */
  --st-none: #94a3b8;
  --st-applied: #3b82f6;
  --st-interview: #f59e0b;
  --st-offer: #22c55e;
  --st-rejected: #ef4444;

  /* Category accent (border-left) */
  --nuclear: #22c55e;
  --thermal: #f97316;
  --design: #8b5cf6;
  --soe: #3b82f6;
  --private: #ef4444;

  /* Layout */
  --header-h: auto;
  --footer-h: 36px;
  --left-w: 220px;
  --right-w: 300px;
  --gap: 10px;
  --radius: 10px;
}
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  background: var(--bg);
  color: var(--text);
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
```

- [ ] **Step 3: Write the CSS — header**

```css
.header {
  background: linear-gradient(135deg, var(--primary), #2d5aa0);
  color: #fff;
  padding: 12px 16px 10px;
  flex-shrink: 0;
}
.header-top {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 8px;
}
.header-top h1 { font-size: 1.1rem; font-weight: 700; }
.version { font-size: 0.65rem; opacity: 0.6; font-weight: 400; }
.updated { font-size: 0.7rem; opacity: 0.8; }
.search-input {
  width: 100%;
  padding: 8px 14px;
  border-radius: 20px;
  border: none;
  outline: none;
  font-size: 0.85rem;
  background: rgba(255,255,255,0.15);
  color: #fff;
  backdrop-filter: blur(4px);
}
.search-input::placeholder { color: rgba(255,255,255,0.5); }
.change-summary { font-size: 0.7rem; opacity: 0.8; margin-top: 6px; }
.filter-quick {
  display: flex; gap: 6px; margin-top: 8px;
  flex-wrap: wrap; align-items: center;
}
.pill {
  padding: 4px 12px; border-radius: 14px; border: 1px solid rgba(255,255,255,0.3);
  font-size: 0.7rem; cursor: pointer; user-select: none;
  white-space: nowrap; transition: all 0.15s;
  background: rgba(255,255,255,0.1); color: #fff;
  font-weight: 500;
}
.pill.active { background: #fff; color: var(--primary); border-color: #fff; font-weight: 700; }
.pill .count { font-size: 0.6rem; opacity: 0.7; margin-left: 2px; }
```

- [ ] **Step 4: Write the CSS — three-column grid and panels**

```css
.main {
  display: grid;
  grid-template-columns: var(--left-w) 1fr var(--right-w);
  gap: var(--gap);
  padding: 8px 10px;
  flex: 1;
  overflow: hidden;
}
.panel-left, .panel-right {
  background: var(--card);
  border-radius: var(--radius);
  box-shadow: var(--shadow);
  overflow-y: auto;
  padding: 10px;
}
.panel-center {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.panel-section { margin-bottom: 12px; }
.panel-section h3 {
  font-size: 0.72rem; color: var(--sub); text-transform: uppercase;
  letter-spacing: 0.5px; margin-bottom: 5px; font-weight: 600;
}
.filter-item {
  display: flex; align-items: center; justify-content: space-between;
  padding: 5px 8px; border-radius: 6px; cursor: pointer;
  font-size: 0.78rem; transition: background 0.1s;
}
.filter-item:hover { background: var(--bg); }
.filter-item.active { background: #dbeafe; color: var(--accent); font-weight: 600; }
.filter-item .badge-count { font-size: 0.65rem; color: var(--sub); }

.list-toolbar {
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 8px; flex-shrink: 0;
}
.result-count { font-size: 0.75rem; color: var(--sub); }
.view-btn {
  padding: 4px 12px; border-radius: 12px; border: 1px solid var(--border);
  background: var(--card); font-size: 0.7rem; cursor: pointer;
}
.view-btn.active { background: var(--primary); color: #fff; border-color: var(--primary); }

.active-filters {
  display: flex; gap: 4px; flex-wrap: wrap; margin-bottom: 6px; flex-shrink: 0;
}
.chip {
  display: inline-flex; align-items: center; gap: 3px;
  padding: 3px 10px; border-radius: 12px;
  background: var(--accent); color: #fff; font-size: 0.68rem;
  cursor: pointer;
}
.chip::after { content: '×'; font-size: 0.9rem; }

.job-list {
  flex: 1; overflow-y: auto;
  display: flex; flex-direction: column; gap: 6px;
  padding-right: 2px;
}
```

- [ ] **Step 5: Write the CSS — job cards and table view**

```css
.card {
  background: var(--card);
  border-radius: var(--radius);
  padding: 10px 12px;
  box-shadow: var(--shadow);
  border-left: 3px solid var(--border);
  cursor: pointer;
  transition: all 0.12s;
  position: relative;
}
.card:hover { transform: translateY(-1px); box-shadow: 0 2px 6px rgba(0,0,0,0.08); }
.card.selected { ring: 2px solid var(--accent); box-shadow: 0 0 0 2px var(--accent); }
.card.nuclear { border-left-color: var(--nuclear); }
.card.thermal { border-left-color: var(--thermal); }
.card.design  { border-left-color: var(--design); }
.card.soe     { border-left-color: var(--soe); }
.card.private { border-left-color: var(--private); }

.card-top { display: flex; justify-content: space-between; align-items: flex-start; gap: 6px; }
.card-company { font-size: 0.83rem; font-weight: 700; }
.card-badges { display: flex; gap: 3px; flex-shrink: 0; flex-wrap: wrap; }
.badge {
  padding: 1px 7px; border-radius: 8px; font-size: 0.6rem; font-weight: 600;
  white-space: nowrap; line-height: 1.5;
}
.badge.p-s { background: #fee2e2; color: #991b1b; }
.badge.p-a { background: #ffedd5; color: #9a3412; }
.badge.p-b { background: #dbeafe; color: #1e40af; }
.badge.p-c { background: #f1f5f9; color: #475569; }
.badge.internship { background: #dcfce7; color: #166534; }
.badge.campus     { background: #dbeafe; color: #1e40af; }
.badge.joint      { background: #fef3c7; color: #92400e; }
.badge.social     { background: #ffedd5; color: #9a3412; }
.badge.new     { background: #dcfce7; color: #166534; }
.badge.changed { background: #fef3c7; color: #92400e; }
.badge.urgent  { background: #fee2e2; color: #991b1b; }
.badge.st-none     { background: #f1f5f9; color: #64748b; }
.badge.st-applied  { background: #dbeafe; color: #1e40af; }
.badge.st-interview{ background: #fef3c7; color: #92400e; }
.badge.st-offer    { background: #dcfce7; color: #166534; }
.badge.st-rejected { background: #fee2e2; color: #991b1b; }
.badge.has-note { background: #fef3c7; color: #92400e; }

.card-title { font-size: 0.8rem; margin: 5px 0 3px; font-weight: 600; }
.card-meta { font-size: 0.68rem; color: var(--sub); display: flex; flex-wrap: wrap; gap: 2px 12px; }
.card-footer { display: flex; justify-content: space-between; align-items: center; margin-top: 7px; }
.card-deadline { font-weight: 700; font-size: 0.72rem; }
.card-deadline.urgent { color: #dc2626; }
.card-deadline.warn   { color: #ea580c; }
.card-deadline.ok     { color: #6b7280; }
.card-salary { font-size: 0.7rem; color: var(--primary); font-weight: 600; }

/* Table view */
.job-list.table-view { gap: 0; }
.job-list.table-view .card {
  display: grid;
  grid-template-columns: 30px 1fr 100px 90px 70px 60px;
  gap: 8px; align-items: center;
  border-left: none; border-bottom: 1px solid var(--border);
  border-radius: 0; box-shadow: none; padding: 8px 10px;
  font-size: 0.78rem;
}
.job-list.table-view .card:hover { background: #f8fafc; transform: none; }
.job-list.table-view .card-meta, .job-list.table-view .card-footer { display: none; }
.job-list.table-view .card-title { margin: 0; font-size: 0.78rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
```

- [ ] **Step 6: Write the CSS — right panel detail view**

```css
.detail-empty {
  height: 100%; display: flex; align-items: center; justify-content: center;
  color: var(--sub); font-size: 0.85rem;
}
.detail-content h2 { font-size: 0.95rem; margin-bottom: 8px; }
.detail-content .detail-row {
  display: flex; justify-content: space-between; align-items: baseline;
  padding: 5px 0; border-bottom: 1px solid var(--border);
  font-size: 0.78rem;
}
.detail-content .detail-label { color: var(--sub); flex-shrink: 0; }
.detail-content .detail-value { text-align: right; font-weight: 500; }

.detail-status { margin: 12px 0; }
.detail-status select {
  width: 100%; padding: 8px; border-radius: 8px;
  border: 1px solid var(--border); font-size: 0.82rem;
  background: var(--card);
}
.detail-notes { margin: 10px 0; }
.detail-notes textarea {
  width: 100%; height: 100px; padding: 8px; border-radius: 8px;
  border: 1px solid var(--border); font-size: 0.78rem;
  resize: vertical; font-family: inherit; line-height: 1.5;
}
.detail-link {
  display: block; text-align: center; padding: 10px; margin-top: 12px;
  border-radius: 20px; background: var(--primary); color: #fff;
  font-size: 0.82rem; font-weight: 600; text-decoration: none;
}
.detail-history { margin-top: 12px; }
.detail-history h3 { font-size: 0.72rem; color: var(--sub); margin-bottom: 4px; }
.detail-history .history-item { font-size: 0.7rem; color: var(--sub); padding: 2px 0; }
```

- [ ] **Step 7: Write the CSS — footer, overlay, responsive**

```css
.footer {
  background: var(--card);
  border-top: 1px solid var(--border);
  padding: 8px 16px;
  font-size: 0.72rem; color: var(--sub);
  flex-shrink: 0;
  display: flex; justify-content: space-between; gap: 16px; flex-wrap: wrap;
}
.overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.3); z-index: 90; }
.overlay.show { display: block; }

/* Responsive: mobile */
@media (max-width: 768px) {
  .main {
    grid-template-columns: 1fr;
    grid-template-rows: 1fr;
  }
  .panel-left {
    position: fixed; left: 0; top: 0; bottom: 0; width: 260px;
    z-index: 100; transform: translateX(-100%);
    transition: transform 0.2s;
    border-radius: 0; padding-top: 16px;
  }
  .panel-left.open { transform: translateX(0); }
  .panel-right {
    position: fixed; left: 0; right: 0; bottom: 0;
    z-index: 100; max-height: 50vh; border-radius: 16px 16px 0 0;
    transform: translateY(100%); transition: transform 0.2s;
  }
  .panel-right.open { transform: translateY(0); }

  /* Mobile filter toggle button */
  .mobile-filter-btn {
    display: inline-flex !important;
    padding: 5px 12px; border-radius: 14px;
    border: 1px solid rgba(255,255,255,0.3);
    background: rgba(255,255,255,0.1);
    color: #fff; font-size: 0.72rem; cursor: pointer;
  }
  .job-list.table-view .card {
    grid-template-columns: 30px 1fr 80px 60px;
  }
}
@media (min-width: 769px) {
  .mobile-filter-btn { display: none !important; }
}
```

- [ ] **Step 8: Test the scaffold**

Open `index.html` in browser. Verify:
- Three columns visible (left: filter panels, center: empty list, right: placeholder)
- Header with search bar visible
- Resize to mobile: columns collapse
- No JS errors (script tag is empty)

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "feat(v2): scaffold HTML structure + CSS foundation"
```

---

### Task 2: Data layer — fetch, localStorage, scoring, change detection

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (add JS inside `<script>` tag)

- [ ] **Step 1: Add localStorage read/write helpers**

```javascript
const LS = {
  get(key, fallback) {
    try { const v = localStorage.getItem('radar_'+key); return v ? JSON.parse(v) : fallback; }
    catch { return fallback; }
  },
  set(key, val) {
    try { localStorage.setItem('radar_'+key, JSON.stringify(val)); }
    catch(e) { console.warn('localStorage full', e); }
  }
};
```

- [ ] **Step 2: Add priority scoring function**

```javascript
const SCORE = {
  salary(s) {
    if (!s) return 12;
    const num = parseInt(s.replace(/[^0-9]/g, ''));
    if (num >= 40) return 30;       // 40万+
    if (num >= 25) return 25;       // 25-39万
    if (num >= 15) return 20;       // 15-24万
    if (num >= 8) return 15;        // 8-14万
    return 10;
  },
  platform(j) {
    const company = j.company || '';
    const title = j.title || '';
    // Top institutes
    if (/728|核动力|星原|核星|热工院/.test(title+company)) return 30;
    // Central SOE
    if (/中核|中广核|华能|大唐|华电|国家电投|国家能源|中石化|中海油|中国电建|中国能建|东方电气|上海电气|哈电/.test(company)) return 25;
    // Provincial SOE
    if (/申能|浙能|广东能源|深圳能源|皖能|京能|陕西/.test(company) && /集团|能源|电力/.test(company)) return 20;
    // Large private
    if (/宁德时代|比亚迪|远景|台积电|美的|潍柴|小鹏|欣旺达/.test(company)) return 20;
    return 10;
  },
  track(j) {
    const t = (j.title||'') + (j.major||'');
    if (/储能|新能源|电池|氢能/.test(t)) return 20;
    if (/核电|核工|核科学/.test(t)) return 17;
    if (/设计|CFD|仿真|研发/.test(t)) return 13;
    return 10;
  },
  location(j) {
    const loc = j.location || '';
    if (/上海|北京|深圳/.test(loc)) return 20;
    if (/广州|杭州|成都|武汉|南京|西安|苏州/.test(loc)) return 17;
    return 13;
  },
  bonuses(j) {
    let b = 0;
    if (j.deadline) {
      const days = (new Date(j.deadline) - new Date()) / 86400000;
      if (days <= 7) b += 5;
      else if (days <= 14) b += 3;
    }
    const t = (j.salary||'') + (j.title||'');
    if (/直通|转正|优先录用|签约/.test(t)) b += 5;
    if (/住宿|安家费|购房/.test(j.salary||'')) b += 3;
    return b;
  },
  tier(total) {
    if (total >= 70) return 'S';
    if (total >= 60) return 'A';
    if (total >= 45) return 'B';
    return 'C';
  },
  compute(j) {
    const s = this.salary(j.salary) + this.platform(j) + this.track(j) + this.location(j) + this.bonuses(j);
    return { total: s, tier: this.tier(s) };
  }
};
```

- [ ] **Step 3: Add change detection logic**

```javascript
function detectChanges(jobs) {
  const prev = LS.get('snapshot', {});
  const changes = { newIds: new Set(), changedIds: new Set(), urgentIds: new Set() };

  for (const j of jobs) {
    if (j.expired) continue;
    const prevJob = prev[j.id];
    if (!prevJob) {
      changes.newIds.add(j.id);
    } else if (prevJob.deadline !== j.deadline || prevJob.salary !== j.salary || prevJob.title !== j.title) {
      changes.changedIds.add(j.id);
    }
    if (j.deadline) {
      const days = (new Date(j.deadline) - new Date()) / 86400000;
      if (days >= 0 && days <= 7) changes.urgentIds.add(j.id);
    }
  }

  // Save snapshot for next visit
  const snap = {};
  for (const j of jobs) snap[j.id] = { deadline: j.deadline, salary: j.salary, title: j.title, expired: j.expired };
  LS.set('snapshot', snap);

  return changes;
}
```

- [ ] **Step 4: Add data loading and scoring pipeline**

```javascript
let jobs = [];            // raw data from json
let scoredJobs = [];     // with score and tier attached
let selectedId = null;
let viewMode = 'card';   // 'card' | 'table'

async function loadData() {
  try {
    const resp = await fetch('/data/jobs.json', { cache: 'no-cache' });
    if (!resp.ok) throw new Error('HTTP ' + resp.status);
    const data = await resp.json();
    jobs = data.jobs;

    // Attach scores
    scoredJobs = jobs.map(j => ({
      ...j,
      _score: SCORE.compute(j)
    }));

    // Detect changes
    const changes = detectChanges(jobs);
    window._changes = changes;

    // Update header
    document.getElementById('updateTime').textContent =
      '更新于 ' + (data.updated_at ? data.updated_at.slice(0,16).replace('T',' ') : '未知');
    document.getElementById('changeSummary').textContent =
      `🆕新增${changes.newIds.size} · ⚡变更${changes.changedIds.size} · ⚠️即将截止${changes.urgentIds.size}`;

    renderAll();
  } catch (err) {
    document.getElementById('jobList').innerHTML =
      '<div class="detail-empty">⚠️ 数据加载失败，请检查网络后刷新</div>';
  }
}
```

- [ ] **Step 5: Wire up init on DOM ready**

```javascript
document.addEventListener('DOMContentLoaded', loadData);

if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(v2): data layer — fetch, localStorage, scoring, change detection"
```

---

### Task 3: Header — keyword search + quick filter bar + render pipeline

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS `<script>`)

- [ ] **Step 1: Add filter state and search debounce**

```javascript
let filters = {
  keyword: '',
  categories: new Set(),
  types: new Set(),
  cities: new Set(),
  priorities: new Set(),
  statuses: new Set(),
  deadline: 'all',      // 'week' | 'month' | 'nextMonth' | 'all'
  onlyChanges: false,
};

const statusLabels = { none:'未投递', applied:'已投递', interview:'面试中', offer:'Offer', rejected:'已拒绝' };
const typeLabels = { internship:'实习', campus:'校招', joint:'联培', social:'社招' };
const catLabels = { nuclear:'核电', thermal:'水火电', design:'设计院', soe:'国央企', private:'民企' };

function getStatus(jobId) {
  const all = LS.get('status', {});
  return all[jobId] || 'none';
}

const searchInput = document.getElementById('searchInput');
let searchTimer;
searchInput.addEventListener('input', () => {
  clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    filters.keyword = searchInput.value.trim().toLowerCase();
    renderJobList();
  }, 200);
});
```

- [ ] **Step 2: Add filter function**

```javascript
function applyFilters() {
  let result = scoredJobs;

  // Keyword
  if (filters.keyword) {
    const kw = filters.keyword;
    result = result.filter(j =>
      (j.company||'').includes(kw) || (j.title||'').includes(kw) ||
      (j.location||'').includes(kw) || (j.major||'').includes(kw) ||
      (j.salary||'').includes(kw)
    );
  }

  // Expired filter
  result = result.filter(j => !j.expired);

  // Category
  if (filters.categories.size > 0) {
    result = result.filter(j => filters.categories.has(j.category));
  }

  // Type
  if (filters.types.size > 0) {
    result = result.filter(j => filters.types.has(j.type));
  }

  // City
  if (filters.cities.size > 0) {
    result = result.filter(j => {
      const loc = j.location || '';
      for (const c of filters.cities) {
        if (loc.includes(c)) return true;
      }
      return false;
    });
  }

  // Priority
  if (filters.priorities.size > 0) {
    result = result.filter(j => filters.priorities.has(j._score.tier));
  }

  // Status
  if (filters.statuses.size > 0) {
    result = result.filter(j => filters.statuses.has(getStatus(j.id)));
  }

  // Deadline
  if (filters.deadline !== 'all') {
    result = result.filter(j => {
      if (!j.deadline) return false;
      const days = (new Date(j.deadline) - new Date()) / 86400000;
      if (filters.deadline === 'week') return days >= 0 && days <= 7;
      if (filters.deadline === 'month') return days >= 0 && days <= 31;
      if (filters.deadline === 'nextMonth') return days > 31 && days <= 62;
      return true;
    });
  }

  // Only changes
  if (filters.onlyChanges && window._changes) {
    const c = window._changes;
    result = result.filter(j => c.newIds.has(j.id) || c.changedIds.has(j.id) || c.urgentIds.has(j.id));
  }

  // Sort by score descending
  result.sort((a,b) => b._score.total - a._score.total);

  return result;
}
```

- [ ] **Step 3: Add quick filter bar event handlers**

```javascript
function renderFilterQuick() {
  const container = document.getElementById('filterQuick');
  const cats = ['all','nuclear','thermal','design','soe','private'];
  const types = ['all','internship','campus','joint'];
  const deadlines = [
    { key:'all', label:'全部截止' },
    { key:'week', label:'本周截止' },
    { key:'month', label:'本月截止' },
    { key:'nextMonth', label:'下月截止' },
  ];

  let html = '';

  // Category pills
  for (const c of cats) {
    const label = c === 'all' ? '全部' : catLabels[c];
    const active = (c === 'all' && filters.categories.size === 0) || filters.categories.has(c);
    html += `<span class="pill${active?' active':''}" data-filter="category" data-val="${c}">${label}</span>`;
  }

  html += '<span style="width:8px"></span>';

  // Type pills
  for (const t of types) {
    const label = t === 'all' ? '全部类型' : typeLabels[t];
    const active = (t === 'all' && filters.types.size === 0) || filters.types.has(t);
    html += `<span class="pill${active?' active':''}" data-filter="type" data-val="${t}">${label}</span>`;
  }

  html += '<span style="width:8px"></span>';

  // Deadline pills
  for (const d of deadlines) {
    html += `<span class="pill${filters.deadline===d.key?' active':''}" data-filter="deadline" data-val="${d.key}">${d.label}</span>`;
  }

  html += '<span style="flex:1"></span>';

  // Only changes toggle
  html += `<span class="pill${filters.onlyChanges?' active':''}" data-filter="onlyChanges" data-val="toggle">🔔 仅看变化</span>`;

  // Mobile filter button
  html += '<span class="mobile-filter-btn" id="mobileFilterBtn">☰ 筛选</span>';

  container.innerHTML = html;
}

document.getElementById('filterQuick').addEventListener('click', e => {
  const el = e.target.closest('[data-filter]');
  if (!el) return;
  const ftype = el.dataset.filter;
  const val = el.dataset.val;

  if (ftype === 'category') {
    if (val === 'all') filters.categories.clear();
    else {
      filters.categories.has(val) ? filters.categories.delete(val) : filters.categories.add(val);
    }
  } else if (ftype === 'type') {
    if (val === 'all') filters.types.clear();
    else {
      filters.types.has(val) ? filters.types.delete(val) : filters.types.add(val);
    }
  } else if (ftype === 'deadline') {
    filters.deadline = val;
  } else if (ftype === 'onlyChanges') {
    filters.onlyChanges = !filters.onlyChanges;
  }
  renderAll();
});
```

- [ ] **Step 4: Add renderAll orchestrator**

```javascript
function renderAll() {
  renderFilterQuick();
  if (typeof renderLeftPanel === 'function') renderLeftPanel();
  renderJobList();
  if (typeof renderFooter === 'function') renderFooter();
}
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(v2): header — keyword search + quick filter bar + filter logic"
```

---

### Task 4: Left panel — filter groups with counts

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS)

- [ ] **Step 1: Add left panel render functions**

```javascript
function renderLeftPanel() {
  const filtered = applyFilters();
  renderPriorityFilters(filtered);
  renderCategoryFilters(filtered);
  renderStatusFilters(filtered);
  renderCityFilters(filtered);
}

function renderPriorityFilters(filtered) {
  const counts = { S:0, A:0, B:0, C:0 };
  for (const j of filtered) counts[j._score.tier]++;
  const tiers = ['S','A','B','C'];
  const labels = { S:'S 级 · 优先投递', A:'A 级 · 重点投递', B:'B 级 · 值得投递', C:'C 级 · 非优先' };
  let html = '<h3>优先级</h3>';
  for (const t of tiers) {
    const active = filters.priorities.has(t);
    html += `<div class="filter-item${active?' active':''}" data-filter="priority" data-val="${t}">
      <span>${labels[t]}</span><span class="badge-count">${counts[t]}</span></div>`;
  }
  document.getElementById('priorityFilters').innerHTML = html;
}

function renderCategoryFilters(filtered) {
  const counts = {};
  for (const j of filtered) counts[j.category] = (counts[j.category]||0) + 1;
  const cats = ['nuclear','thermal','design','soe','private'];
  let html = '<h3>分类</h3>';
  for (const c of cats) {
    const active = filters.categories.has(c);
    html += `<div class="filter-item${active?' active':''}" data-filter="category" data-val="${c}">
      <span>${catLabels[c]}</span><span class="badge-count">${counts[c]||0}</span></div>`;
  }
  document.getElementById('categoryFilters').innerHTML = html;
}

function renderStatusFilters(filtered) {
  const counts = { none:0, applied:0, interview:0, offer:0, rejected:0 };
  for (const j of filtered) counts[getStatus(j.id)]++;
  const sts = ['none','applied','interview','offer','rejected'];
  let html = '<h3>投递状态</h3>';
  for (const s of sts) {
    const active = filters.statuses.has(s);
    html += `<div class="filter-item${active?' active':''}" data-filter="status" data-val="${s}">
      <span>${statusLabels[s]}</span><span class="badge-count">${counts[s]}</span></div>`;
  }
  document.getElementById('statusFilters').innerHTML = html;
}

function renderCityFilters(filtered) {
  const cities = new Map();
  for (const j of filtered) {
    const loc = j.location || '';
    for (const part of loc.split(/[/,、，/]/)) {
      const c = part.trim().replace(/等$/,'');
      if (c && c.length <= 6) cities.set(c, (cities.get(c)||0) + 1);
    }
  }
  const sorted = [...cities.entries()].sort((a,b) => b[1]-a[1]).slice(0, 15);
  let html = '<h3>城市</h3>';
  for (const [city, count] of sorted) {
    const active = filters.cities.has(city);
    html += `<div class="filter-item${active?' active':''}" data-filter="city" data-val="${encodeURIComponent(city)}">
      <span>${city}</span><span class="badge-count">${count}</span></div>`;
  }
  document.getElementById('cityFilters').innerHTML = html;
}
```

- [ ] **Step 2: Add left panel click handler**

```javascript
document.getElementById('panelLeft').addEventListener('click', e => {
  const el = e.target.closest('[data-filter]');
  if (!el) return;
  const ftype = el.dataset.filter;
  const val = decodeURIComponent(el.dataset.val);

  if (ftype === 'priority') {
    filters.priorities.has(val) ? filters.priorities.delete(val) : filters.priorities.add(val);
  } else if (ftype === 'category') {
    filters.categories.has(val) ? filters.categories.delete(val) : filters.categories.add(val);
  } else if (ftype === 'status') {
    filters.statuses.has(val) ? filters.statuses.delete(val) : filters.statuses.add(val);
  } else if (ftype === 'city') {
    filters.cities.has(val) ? filters.cities.delete(val) : filters.cities.add(val);
  }
  renderAll();
});
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(v2): left panel — filter groups with live counts"
```

---

### Task 5: Center list — job cards with badges + active filter chips

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS)

- [ ] **Step 1: Add card render function**

```javascript
function renderJobList() {
  const filtered = applyFilters();
  const listEl = document.getElementById('jobList');
  const countEl = document.getElementById('resultCount');

  countEl.textContent = `共 ${filtered.length} 个岗位`;

  // Active filter chips
  const chips = [];
  for (const c of filters.categories) chips.push({ label: catLabels[c], clear: ()=> { filters.categories.delete(c); renderAll(); } });
  for (const t of filters.types) chips.push({ label: typeLabels[t], clear: ()=> { filters.types.delete(t); renderAll(); } });
  for (const c of filters.cities) chips.push({ label: c, clear: ()=> { filters.cities.delete(c); renderAll(); } });
  for (const p of filters.priorities) chips.push({ label: p+'级', clear: ()=> { filters.priorities.delete(p); renderAll(); } });
  for (const s of filters.statuses) chips.push({ label: statusLabels[s], clear: ()=> { filters.statuses.delete(s); renderAll(); } });
  if (filters.deadline !== 'all') {
    const dlLabels = { week:'本周截止', month:'本月截止', nextMonth:'下月截止' };
    chips.push({ label: dlLabels[filters.deadline], clear: ()=> { filters.deadline='all'; renderAll(); } });
  }
  document.getElementById('activeFilters').innerHTML = chips.map(c =>
    `<span class="chip" onclick="void(0)">${c.label}</span>`
  ).join('');

  // Render list
  if (filtered.length === 0) {
    listEl.className = 'job-list';
    listEl.innerHTML = '<div class="detail-empty">📭 无匹配岗位，试试调整筛选条件</div>';
    return;
  }

  listEl.className = 'job-list' + (viewMode === 'table' ? ' table-view' : '');

  if (viewMode === 'table') {
    listEl.innerHTML = `
      <div class="card" style="font-weight:700;border-bottom:2px solid var(--border);cursor:default;background:var(--bg)">
        <span></span><span>岗位</span><span>公司</span><span>地点</span><span>截止</span><span>薪资</span>
      </div>` +
      filtered.map(j => renderTableRow(j)).join('');
  } else {
    listEl.innerHTML = filtered.map(j => renderCard(j)).join('');
  }

  // Re-attach chip click handlers (since innerHTML replaces DOM)
  document.querySelectorAll('#activeFilters .chip').forEach((chip, i) => {
    chip.addEventListener('click', () => chips[i].clear());
  });
}

function renderCard(j) {
  const st = getStatus(j.id);
  const notes = LS.get('notes', {});
  const hasNote = notes[j.id] && notes[j.id].trim();
  const changes = window._changes;
  const tier = j._score.tier;
  const isUrgent = changes && changes.urgentIds.has(j.id);
  const isNew = changes && changes.newIds.has(j.id);
  const isChanged = changes && changes.changedIds.has(j.id);
  const isSelected = selectedId === j.id;

  return `
  <div class="card ${j.category}${isSelected?' selected':''}" data-id="${j.id}" onclick="selectJob('${j.id}')">
    <div class="card-top">
      <span class="card-company">${j.company}</span>
      <span class="card-badges">
        <span class="badge p-${tier.toLowerCase()}">${tier}</span>
        <span class="badge ${j.type}">${typeLabels[j.type]||j.type}</span>
        ${st !== 'none' ? `<span class="badge st-${st}">${statusLabels[st]}</span>` : ''}
        ${isNew ? '<span class="badge new">🆕</span>' : ''}
        ${isChanged ? '<span class="badge changed">⚡</span>' : ''}
        ${isUrgent ? '<span class="badge urgent">⚠️</span>' : ''}
        ${hasNote ? '<span class="badge has-note">📝</span>' : ''}
      </span>
    </div>
    <div class="card-title">${highlightMatch(j.title)}</div>
    <div class="card-meta">
      ${j.location ? '<span>📍 '+j.location+'</span>' : ''}
      ${j.target ? '<span>🎓 '+j.target+'</span>' : ''}
    </div>
    <div class="card-footer">
      <span class="card-deadline ${getDeadlineClass(j.deadline)}">⏰ ${formatDeadline(j.deadline)}</span>
      ${j.salary ? '<span class="card-salary">💰 '+j.salary+'</span>' : ''}
    </div>
  </div>`;
}

function renderTableRow(j) {
  const tier = j._score.tier;
  const st = getStatus(j.id);
  return `
  <div class="card ${j.category}" data-id="${j.id}" onclick="selectJob('${j.id}')">
    <span class="badge p-${tier.toLowerCase()}" style="font-size:0.6rem">${tier}</span>
    <span style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${j.title}</span>
    <span style="font-size:0.72rem;color:var(--sub);overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${j.company}</span>
    <span style="font-size:0.7rem;color:var(--sub)">${(j.location||'').split(/[/,、，/]/)[0]}</span>
    <span class="card-deadline ${getDeadlineClass(j.deadline)}" style="font-size:0.68rem">${j.deadline ? formatDeadline(j.deadline) : '待定'}</span>
    <span style="font-size:0.68rem;color:var(--primary);font-weight:600">${j.salary||'--'}</span>
  </div>`;
}

function getDeadlineClass(d) {
  if (!d) return 'ok';
  const days = (new Date(d) - new Date()) / 86400000;
  if (days < 0) return 'urgent';
  if (days <= 7) return 'urgent';
  if (days <= 14) return 'warn';
  return 'ok';
}

function formatDeadline(d) {
  if (!d) return '待定';
  const dt = new Date(d);
  const days = Math.ceil((dt - new Date()) / 86400000);
  const dateStr = (dt.getMonth()+1)+'月'+dt.getDate()+'日';
  return days < 0 ? dateStr+' 已截止' : days === 0 ? '今天截止' : dateStr+' ('+days+'天)';
}

function highlightMatch(text) {
  if (!filters.keyword || !text) return text;
  const kw = filters.keyword.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  return text.replace(new RegExp(`(${kw})`, 'gi'), '<mark style="background:#fef08a;padding:0 2px;border-radius:2px">$1</mark>');
}
```

- [ ] **Step 2: Add view toggle handler**

```javascript
document.getElementById('viewCard').addEventListener('click', () => { viewMode='card'; renderAll(); });
document.getElementById('viewTable').addEventListener('click', () => { viewMode='table'; renderAll(); });
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(v2): center list — job cards with badges + table view + highlight"
```

---

### Task 6: Right panel — detail view, status dropdown, notes, history

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS)

- [ ] **Step 1: Add selectJob and renderDetail**

```javascript
function selectJob(id) {
  selectedId = id;
  renderJobList(); // re-render to update selected state
  renderDetail(id);

  // Mobile: show right panel
  if (window.innerWidth <= 768) {
    document.getElementById('panelRight').classList.add('open');
    document.getElementById('overlay').classList.add('show');
  }
}

function renderDetail(id) {
  const j = scoredJobs.find(j => j.id === id);
  if (!j) return;

  document.getElementById('detailEmpty').style.display = 'none';
  const dc = document.getElementById('detailContent');
  dc.style.display = 'block';

  const st = getStatus(id);
  const notes = LS.get('notes', {});
  const note = notes[id] || '';

  dc.innerHTML = `
    <h2>${j.title}</h2>
    <div class="detail-row"><span class="detail-label">公司</span><span class="detail-value">${j.company}</span></div>
    <div class="detail-row"><span class="detail-label">地点</span><span class="detail-value">${j.location||'--'}</span></div>
    <div class="detail-row"><span class="detail-label">目标</span><span class="detail-value">${j.target||'--'}</span></div>
    <div class="detail-row"><span class="detail-label">专业</span><span class="detail-value">${j.major||'--'}</span></div>
    <div class="detail-row"><span class="detail-label">截止</span><span class="detail-value"><span class="card-deadline ${getDeadlineClass(j.deadline)}">${formatDeadline(j.deadline)}</span></span></div>
    <div class="detail-row"><span class="detail-label">薪资</span><span class="detail-value">${j.salary||'--'}</span></div>
    <div class="detail-row"><span class="detail-label">优先级</span><span class="detail-value"><span class="badge p-${j._score.tier.toLowerCase()}">${j._score.tier} 级 (${j._score.total}分)</span></span></div>

    <div class="detail-status">
      <label style="font-size:0.72rem;color:var(--sub);display:block;margin-bottom:4px">投递状态</label>
      <select id="detailStatus" onchange="updateStatus('${id}', this.value)">
        ${['none','applied','interview','offer','rejected'].map(s =>
          `<option value="${s}"${st===s?' selected':''}>${statusLabels[s]}</option>`
        ).join('')}
      </select>
    </div>

    <div class="detail-notes">
      <label style="font-size:0.72rem;color:var(--sub);display:block;margin-bottom:4px">📝 私人备注</label>
      <textarea id="detailNotes" placeholder="写点备注..." onblur="saveNote('${id}', this.value)">${note}</textarea>
    </div>

    ${j.link ? `<a class="detail-link" href="${j.link}" target="_blank" rel="noopener">立即投递 →</a>` : ''}

    <div class="detail-history">
      <h3>变更记录</h3>
      <div class="history-item">首次收录: ${j.first_seen||'未知'}</div>
    </div>
  `;
}

function updateStatus(jobId, newStatus) {
  const all = LS.get('status', {});
  all[jobId] = newStatus;
  LS.set('status', all);
  renderJobList();
}

let noteSaveTimer;
function saveNote(jobId, text) {
  clearTimeout(noteSaveTimer);
  noteSaveTimer = setTimeout(() => {
    const all = LS.get('notes', {});
    all[jobId] = text;
    LS.set('notes', all);
    renderJobList(); // update note indicator
  }, 800);
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat(v2): right panel — detail view, status dropdown, notes, history"
```

---

### Task 7: Mobile responsive — drawer, bottom sheet, overlay

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS)

- [ ] **Step 1: Add mobile interaction handlers**

```javascript
// Mobile filter drawer
document.getElementById('mobileFilterBtn').addEventListener('click', () => {
  document.getElementById('panelLeft').classList.add('open');
  document.getElementById('overlay').classList.add('show');
});

// Close panels on overlay click
document.getElementById('overlay').addEventListener('click', () => {
  document.getElementById('panelLeft').classList.remove('open');
  document.getElementById('panelRight').classList.remove('open');
  document.getElementById('overlay').classList.remove('show');
});

// Close right panel (detail) with close button on mobile
// The ensureMobileCloseBtn injects a close button into the right panel on mobile
function ensureMobileCloseBtn() {
  let btn = document.getElementById('detailCloseBtn');
  if (!btn && window.innerWidth <= 768) {
    btn = document.createElement('button');
    btn.id = 'detailCloseBtn';
    btn.className = 'detail-close';
    btn.textContent = '✕ 关闭';
    btn.style.cssText = 'position:absolute;top:8px;right:12px;background:none;border:none;font-size:1rem;cursor:pointer;color:var(--sub)';
    btn.addEventListener('click', () => {
      document.getElementById('panelRight').classList.remove('open');
      document.getElementById('overlay').classList.remove('show');
    });
    document.getElementById('panelRight').prepend(btn);
  }
}

// Extend selectJob to handle mobile bottom sheet
const _originalSelectJob = selectJob;
selectJob = function(id) {
  _originalSelectJob(id);
  ensureMobileCloseBtn();
};
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat(v2): mobile responsive — drawer, bottom sheet, overlay"
```

---

### Task 8: Footer + final polish + edge cases

**Files:**
- Modify: `F:\job-app\energy-job-radar\index.html` (extend JS)

- [ ] **Step 1: Add footer render**

```javascript
function renderFooter() {
  const active = scoredJobs.filter(j => !j.expired);
  const expired = scoredJobs.filter(j => j.expired);
  const c = window._changes || { newIds: new Set(), changedIds: new Set(), urgentIds: new Set() };

  const st = LS.get('status', {});
  let applied = 0, interview = 0, offer = 0;
  for (const j of active) {
    const s = st[j.id] || 'none';
    if (s === 'applied') applied++;
    else if (s === 'interview') interview++;
    else if (s === 'offer') offer++;
  }

  document.getElementById('footerStats').innerHTML =
    `共 <b>${scoredJobs.length}</b> 岗位 · <b>${active.length}</b> 活跃 · ` +
    `🆕${c.newIds.size} · ⚡${c.changedIds.size} · ⚠️${c.urgentIds.size} · ` +
    `📤 已投${applied} · 💬 面试${interview} · ✅ Offer${offer}`;
}
```

- [ ] **Step 2: Handle edge cases**

```javascript
// Edge case: no data at all
// Edge case: all jobs expired
// Edge case: localStorage full (already handled in LS.set with try/catch)
// Edge case: empty search results (already handled with "无匹配岗位")
// Edge case: very long company names — CSS handles with overflow hidden
// Edge case: mailto links — open normally

// Keyboard shortcut: Esc to clear search
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') {
    searchInput.value = '';
    filters.keyword = '';
    renderAll();
    searchInput.blur();
  }
});

// Keyboard shortcut: Ctrl+F focus search
document.addEventListener('keydown', e => {
  if ((e.ctrlKey || e.metaKey) && e.key === 'f') {
    e.preventDefault();
    searchInput.focus();
  }
});
```

- [ ] **Step 3: Full manual test**

Open `index.html` in browser and verify:
1. Page loads, jobs appear with S/A/B/C badges
2. Keyword search filters in real-time
3. Category/type pills toggle correctly
4. Left panel filters work, counts update
5. Clicking a job shows detail in right panel
6. Status dropdown changes persist after page reload
7. Notes autosave and persist
8. Table view toggle works
9. Mobile: filter drawer opens, detail bottom sheet opens
10. "仅看变化" shows only new/changed/urgent jobs
11. Footer stats are correct
12. Esc clears search
13. PWA: add to home screen still works

- [ ] **Step 4: Commit and push**

```bash
git add index.html
git commit -m "feat(v2): footer + edge cases + keyboard shortcuts"
git push origin main
```
