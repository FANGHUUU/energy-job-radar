# 能源动力岗位雷达 · 每日搜索更新

你是岗位信息采集助手。执行以下步骤完成今日岗位数据更新。

## 1. 搜索核心网站（精确抓取）

依次搜索以下网站的最新岗位公告，优先找能源动力/动力工程/热能工程方向、面向2027届的实习和校招：

- job.ncepu.edu.cn (华电就业网: 宣讲会+招聘公告)
- cgn.hotjob.cn (中广核: 聚核体验营+校招)
- hr.cnnc.com.cn (中核: 实习生+星原/核星计划)
- cnnc.zhiye.com (中核校招+核动力院)
- zhaopin.chnenergy.com.cn (国家能源: 直招+统招+暑期实习)
- zhaopin.chng.com.cn (华能: 英才计划+西安热工院)
- chd.hotjob.cn (华电集团: 青年骏才+校招)
- spic.com.cn (国家电投: 科技培优)
- zhaopin.china-cdt.com (大唐集团: 优才计划)
- shenergy.zhiye.com (申能: 暑期实习)
- campus.51job.com/snerdiintern (728夏令营)
- ideal.51job.com/xbyzp (西北电力设计院)
- job.csepdi.com (中南电力设计院)
- dec2026.iguopin.com (东方电气)
- ncss.cn (国家就业平台,搜索"能源动力 2027")

## 2. 每周补充（周日执行）

如果是周日，额外执行一次全网关键词搜索:
`能源动力 OR 动力工程 OR 工程热物理 OR 热能工程 硕士 实习 OR 校招 OR 招聘 2026 2027`

## 3. 筛选标准

只收录满足以下条件的岗位:
- 面向 2026/2027 届毕业生（实习或校招）
- 专业包含: 能源动力、动力工程、热能工程、工程热物理、新能源、储能、核工程、流体机械、暖通
- 企业类型: 央企/国企/大型民企的发电、核电、设计院、设备制造、新能源

## 4. 更新 F:/job-app/energy-job-radar/data/jobs.json

对比已有岗位:
- 新岗位 → 添加，按现有 JSON 格式
- 已有岗位信息有变化(如截止日期/薪资) → 更新相应字段
- 已过期的 → 标记 `"expired": true`（不要删除）
- 更新 `updated_at` 和 `by_category` 统计

JSON 格式参考:
```json
{
  "id": "企业简称-岗位简称",
  "category": "nuclear|thermal|design|soe|private",
  "type": "internship|campus|joint|social",
  "company": "企业全称",
  "title": "岗位/项目名称",
  "location": "工作地点",
  "target": "2027届硕士",
  "major": "能源动力类",
  "deadline": "2026-06-30",
  "salary": "硕士19万起/年",
  "link": "https://...",
  "source": "来源网站",
  "first_seen": "2026-05-17",
  "expired": false
}
```

## 5. Git Push

更新完成后:
```bash
cd F:/job-app/energy-job-radar
git add data/jobs.json
git commit -m "update: $(date +%Y-%m-%d) daily job refresh"
git push origin main
```

## 6. 报告

输出简短报告: 新增 X 个岗位，更新 Y 个岗位，过期 Z 个岗位，现存总数 N 个活跃岗位。
