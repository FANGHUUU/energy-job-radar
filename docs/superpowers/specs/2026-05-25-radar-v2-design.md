# Energy Job Radar v2 — Design Spec

## Overview

Replace `index.html` on `energy-job-radar.netlify.app` with a full-featured job tracking dashboard. Same PWA, same `jobs.json`, same deployment pipeline — but with keyword search, multi-facet filtering, auto-priority scoring, application tracking, private notes, and change timeline.

## Architecture

Single static HTML file, zero build step, zero framework:

```
jobs.json (remote, git-pushed)
     │ fetch on load
     ▼
  memory state (jobs[])
     │
     ├─ filter/sort → render list
     │
     └─ localStorage
          ├─ appStatus:  { [jobId]: "none"|"applied"|"interview"|"offer"|"rejected" }
          ├─ appNotes:   { [jobId]: string }
          └─ timelineCache: { lastSeenSnapshot, changes[] }
```

All user data lives in `localStorage`. No backend, no auth.

## Layout: Three-Column Desktop, Stacked Mobile

```
┌──────────────────────────────────────────────────────────┐
│  Header: title, last-updated, keyword search bar         │
│  Filter bar: category pills, type pills, city dropdown,  │
│             priority pills, deadline range, "new only"   │
├──────────────┬───────────────────────┬───────────────────┤
│  Left Panel  │    Center List        │  Right Panel      │
│  (filters)   │    (job cards)        │  (detail/notes)   │
│              │                       │                   │
│  Priority    │  ┌─────────────────┐  │  Status dropdown  │
│   S (3) ✓    │  │ S 星原计划 40万  │  │  Notes textarea   │
│   A (8) ✓    │  │ 9/30截止   🆕   │  │  Change history   │
│   B (20) ✓   │  └─────────────────┘  │                   │
│   C (13) ✓   │  ┌─────────────────┐  │                   │
│              │  │ A 728夏令营     │  │                   │
│  Category    │  │ 上海 6/1 ⚠️     │  │                   │
│   核电 (7) ✓ │  └─────────────────┘  │                   │
│   ...        │                       │                   │
│              │                       │                   │
│  Status      │                       │                   │
│   未投递 (30)│                       │                   │
│   已投递 (8) │                       │                   │
│   面试中 (3) │                       │                   │
├──────────────┴───────────────────────┴───────────────────┤
│  Footer: total count, active count, new/changed/urgent   │
└──────────────────────────────────────────────────────────┘
```

On mobile (<768px): left panel collapses into a slide-out drawer; right panel becomes a bottom sheet.

## Feature Modules

### 1. Keyword Search
- Real-time filtering as user types (debounced 200ms)
- Matches against: company, title, location, major, salary, notes
- Highlight matching text in results

### 2. Multi-Facet Filters
- **Category**: 核电 / 水火电 / 设计院 / 国央企 / 民企 (same as v1)
- **Type**: 实习 / 校招 / 联培 / 社招
- **Location**: auto-extracted unique cities from data, multi-select
- **Priority**: S / A / B / C toggle
- **Deadline range**: "本周截止" / "本月截止" / "下月截止" / "不限"
- **Change only**: toggle to show only new/updated jobs since last visit (uses timelineCache)
- **Status**: 未投递 / 已投递 / 面试中 / Offer / 已拒绝
- Active filters shown as removable chips above the list

### 3. Auto Priority Scoring

```javascript
function score(job) {
  let s = 0;
  // Salary (0-30)
  if (salary > 300k) s += 30; else if (>200k) s += 25; else if (>150k) s += 20; else s += 10;
  // Platform (0-30)
  if (央企总部/顶尖院所) s += 30; else if (央企二级/省属) s += 25; else if (大型民企) s += 20; else s += 10;
  // Track (0-20)
  if (储能/新能源) s += 20; else if (核电) s += 17; else if (设计院) s += 13; else s += 10;
  // Location (0-20)
  if (一线城市) s += 20; else if (新一线) s += 17; else s += 13;
  // Bonuses
  if (daysUntilDeadline <= 7) s += 5;
  if (直通校招/转正) s += 5;
  if (有住宿/安家费) s += 3;
  return s; // S≥70 A≥60 B≥45 C<45
}
```

### 4. Application Tracker
- Status per job: 未投递 → 已投递 → 面试中 → Offer → 已拒绝
- Color-coded status badge on each card
- Status filter in left panel (count of each status)
- All data persisted to localStorage

### 5. Private Notes
- Per-job free-text notes
- Auto-save on blur (debounced 1s)
- Notes indicator (📝) on cards that have notes
- Stored in localStorage

### 6. Change Timeline
- On each load, diff current jobs.json against cached snapshot
- Detect: new jobs (🆕), updated fields (⚡), deadline approaching (⚠️ ≤7 days)
- Show change summary in header: "新增3 · 变更2 · 即将截止4"
- "仅看变化" toggle filters to changed jobs only
- Per-job change history in right panel detail view

### 7. Job Detail Panel (Right)
- Full job info (company, title, location, target, major, deadline, salary, link)
- Status dropdown
- Notes textarea
- Change history for this job
- "立即投递" CTA button (opens link in new tab)

### 8. Table/Card Toggle
- Card view (default): visual cards with priority badges
- Table view: compact rows for scanning, sortable columns

## Data Flow

```
页面加载
  ├─ fetch jobs.json
  ├─ 对比 timelineCache 生成变化列表
  ├─ 从 localStorage 加载 appStatus, appNotes
  ├─ 计算优先级评分
  ├─ 渲染筛选面板 + 岗位列表
  └─ 保存当前快照到 timelineCache

用户操作
  ├─ 搜索/筛选 → 实时过滤渲染
  ├─ 切换投递状态 → 写 localStorage + 更新UI
  ├─ 编辑备注 → debounce写 localStorage
  └─ 点击岗位 → 更新右侧详情面板
```

## PWA Compatibility
- Same `manifest.json`, `sw.js`, icons
- Offline: service worker caches `index.html` and `jobs.json`
- Add-to-home-screen unchanged

## What Stays
- `data/jobs.json` format unchanged
- `manifest.json`, `sw.js`, icons unchanged
- Netlify + GitHub Pages auto-deploy unchanged
- `scripts/daily-search.md` unchanged

## What Goes
- Old `index.html` completely replaced
- No migration needed — localStorage keys are new, old v1 had none
