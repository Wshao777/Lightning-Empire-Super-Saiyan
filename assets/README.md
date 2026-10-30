忙你的，整套补齐了。回来照着「收尾清单」执行一遍即可。

---

1️⃣ saiyan/data_sources.py（加 __version__，四份資料版本對齊）

```python
"""所有專案資料與參數的 Python 事實來源。

修改資料請改這裡，然後執行 `python scripts/build_data.py`
重新生成 data/*.json，供前端與外部工具使用。

前端 (JS) 讀 data/*.json；後端 (Python) 直接 import 本模組。
"""

from __future__ import annotations

from typing import Any

# 全專案共用版本；四份資料的 "version" 都必須等於此值。
__version__ = "0.1.0"

# ─────────────────────────────────────────────────────
# 七龍珠 Z
# ─────────────────────────────────────────────────────
DRAGONBALL: dict[str, Any] = {
    "version": __version__,
    "title": "七龍珠 Z 七大篇章索引",
    "total_episodes": 291,
    "note": "採常見七大篇章劃分；不同地區分篇可能略有差異。",
    "sagas": [
        {"order": 1, "name": "賽亞人篇",       "start": 1,   "end": 35},
        {"order": 2, "name": "弗利沙篇",       "start": 36,  "end": 107},
        {"order": 3, "name": "加力克二世篇",   "start": 108, "end": 117},
        {"order": 4, "name": "人造人篇",       "start": 118, "end": 139},
        {"order": 5, "name": "西魯（賽魯）篇", "start": 140, "end": 194},
        {"order": 6, "name": "冥界武道大會篇", "start": 195, "end": 199},
        {"order": 7, "name": "魔人普烏篇",     "start": 200, "end": 291},
    ],
    "characters": [
        {"name": "孫悟空", "role": "主角・賽亞人戰士",
         "skills": ["龜派氣功", "界王拳", "元氣玉", "超級賽亞人變身"]},
        {"name": "達爾（貝吉塔）", "role": "賽亞人王子",
         "note": "從對手成為重要盟友"},
        {"name": "孫悟飯", "role": "悟空的兒子",
         "note": "西魯遊戲展現超級賽亞人 2"},
        {"name": "弗利沙", "role": "宇宙帝王",
         "note": "那美克星篇重要反派"},
        {"name": "西魯（賽魯）", "role": "生化人造戰士",
         "note": "西魯遊戲篇核心反派"},
        {"name": "魔人普烏", "role": "魔人篇主要反派",
         "forms": ["胖普烏", "超級普烏", "小普烏"]},
    ],
}

# ─────────────────────────────────────────────────────
# 金庸
# ─────────────────────────────────────────────────────
WUXIA: dict[str, Any] = {
    "version": __version__,
    "title": "金庸十五部武俠宇宙",
    "mnemonic": "飛雪連天射白鹿，笑書神俠倚碧鴛。",
    "novels": [
        {"order": 1,  "title": "飛狐外傳",   "group": "飛雪連天射白鹿"},
        {"order": 2,  "title": "雪山飛狐",   "group": "飛雪連天射白鹿"},
        {"order": 3,  "title": "連城訣",     "group": "飛雪連天射白鹿"},
        {"order": 4,  "title": "天龍八部",   "group": "飛雪連天射白鹿"},
        {"order": 5,  "title": "射鵰英雄傳", "group": "飛雪連天射白鹿"},
        {"order": 6,  "title": "白馬嘯西風", "group": "飛雪連天射白鹿"},
        {"order": 7,  "title": "鹿鼎記",     "group": "飛雪連天射白鹿"},
        {"order": 8,  "title": "笑傲江湖",   "group": "笑書神俠倚碧鴛"},
        {"order": 9,  "title": "書劍恩仇錄", "group": "笑書神俠倚碧鴛"},
        {"order": 10, "title": "神鵰俠侶",   "group": "笑書神俠倚碧鴛"},
        {"order": 11, "title": "俠客行",     "group": "笑書神俠倚碧鴛"},
        {"order": 12, "title": "倚天屠龍記", "group": "笑書神俠倚碧鴛"},
        {"order": 13, "title": "碧血劍",     "group": "笑書神俠倚碧鴛"},
        {"order": 14, "title": "鴛鴦刀",     "group": "笑書神俠倚碧鴛"},
        {"order": 15, "title": "越女劍",     "group": "獨立短篇"},
    ],
    "trilogy": {
        "name": "射鵰三部曲",
        "parts": [
            {"title": "射鵰英雄傳", "leads": ["郭靖", "黃蓉"]},
            {"title": "神鵰俠侶",   "leads": ["楊過", "小龍女"]},
            {"title": "倚天屠龍記", "leads": ["張無忌", "趙敏"]},
        ],
    },
    "schools": [
        {"name": "少林",   "figures": ["覺遠大師", "少林三渡", "空見神僧"]},
        {"name": "武當",   "founder": "張三丰",
         "figures": ["宋遠橋", "俞蓮舟", "俞岱巖", "張松溪",
                     "張翠山", "殷梨亭", "莫聲谷"]},
        {"name": "峨嵋",   "founder": "郭襄",   "figures": ["滅絕師太", "周芷若"]},
        {"name": "明教",   "figures": ["張無忌", "楊逍", "范遙", "四大法王"]},
        {"name": "丐幫",   "figures": ["洪七公", "黃蓉", "史火龍"]},
        {"name": "古墓派", "founder": "林朝英", "figures": ["小龍女", "楊過"]},
    ],
    "zhang_sanfeng": {
        "title": "武當派開山祖師",
        "master": "覺遠大師",
        "peers": ["郭襄", "楊過（少年張君寶有交集）"],
        "skills": ["太極拳", "太極劍", "武當九陽功", "純陽無極功",
                   "綿掌", "梯雲縱", "真武七截陣"],
        "grand_disciples": ["張無忌", "宋青書"],
    },
}

# ─────────────────────────────────────────────────────
# 3+1 AI 架構
# ─────────────────────────────────────────────────────
AI_ARCH: dict[str, Any] = {
    "version": __version__,
    "project": "Lightning-Empire-Super-Saiyan",
    "boundary": "軟體分層標籤，非外部模型授權或商業關係。",
    "copilot": {
        "role": "human-in-the-loop",
        "owner": "Wshao777",
        "duties": ["決策", "驗收", "邊界把關", "合併前最後確認"],
    },
    "agents": [
        {
            "name": "ChatGPT",
            "layer": "plan-doc",
            "tags": ["architect-note", "content", "spec", "doc"],
            "duties": ["需求拆解", "文件撰寫", "內容整理", "副駕駛協作"],
            "adapter": "copilot/chatgpt.py",
            "status": "stub",
        },
        {
            "name": "Grok",
            "layer": "realtime",
            "tags": ["fact-check", "trend", "brainstorm"],
            "duties": ["即時資訊", "事實核對", "外部視角"],
            "adapter": "copilot/grok.py",
            "status": "stub",
        },
        {
            "name": "DeepSeek",
            "layer": "code",
            "tags": ["physics-local", "python", "numerics", "termux"],
            "duties": ["程式實作", "數值計算", "Termux 整合"],
            "adapter": "copilot/deepseek.py",
            "status": "stub",
        },
    ],
    "platform": {
        "name": "GitHub",
        "layer": "deploy",
        "repo": "Wshao777/Lightning-Empire-Super-Saiyan",
        "duties": ["版本控制", "Actions CI", "Pages 部署", "Issue 追蹤"],
    },
    "workflows": {
        "pages": ".github/workflows/pages.yml",
        "ai_pipeline": ".github/workflows/ai-pipeline.yml",
    },
}

# ─────────────────────────────────────────────────────
# 風力預設參數
# ─────────────────────────────────────────────────────
WIND_PARAMS: dict[str, Any] = {
    "version": __version__,
    "design_angle_deg": 35.0,
    "rated_speed_ms": 12.0,
    "cut_in_ms": 3.0,
    "cut_out_ms": 25.0,
    "cp_max": 0.45,
    "rotor_diameter_m": 80.0,
    "air_density": 1.225,
    "note": "原型曲線，非工程驗證值",
}
```

---

2️⃣ scripts/build_data.py（加 --pretty／--minify／版本對齊檢查）

```python
"""從 saiyan/data_sources.py 生成 data/*.json。

用法：
    python scripts/build_data.py               # 預設 pretty 生成
    python scripts/build_data.py --minify      # 單行壓縮生成
    python scripts/build_data.py --check       # 只檢查（預設用 pretty 格式比對）
    python scripts/build_data.py --check --minify
    python scripts/build_data.py --validate    # 只跑結構驗證，不碰檔案

退出碼：
    0  成功或一致
    1  驗證失敗，或 --check 偵測到漂移
"""

from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path
from typing import Literal

ROOT = Path(__file__).resolve().parents[1]
if str(ROOT) not in sys.path:
    sys.path.insert(0, str(ROOT))

from saiyan import data_sources  # noqa: E402

DATA_DIR = ROOT / "data"
Mode = Literal["pretty", "minify"]

OUTPUTS: dict[str, dict] = {
    "dragonball.json": data_sources.DRAGONBALL,
    "wuxia.json":      data_sources.WUXIA,
    "ai_arch.json":    data_sources.AI_ARCH,
    "wind.json":       data_sources.WIND_PARAMS,
}


# ─────────────────────────────────────────────
# 驗證
# ─────────────────────────────────────────────
def validate() -> list[str]:
    """驗證資料自身一致性（不涉及檔案 I/O）。"""
    errors: list[str] = []

    # 版本對齊
    expected_version = data_sources.__version__
    for name, payload in OUTPUTS.items():
        v = payload.get("version")
        if v != expected_version:
            errors.append(
                f"{name} version={v!r} 與 data_sources.__version__={expected_version!r} 不一致"
            )

    # 七龍珠篇章連續性
    sagas = data_sources.DRAGONBALL["sagas"]
    if not sagas:
        errors.append("dragonball.sagas 為空")
    expected = 1
    for s in sagas:
        if s["start"] != expected:
            errors.append(f"篇章 {s['name']} 起點 {s['start']} 應為 {expected}")
        if s["end"] < s["start"]:
            errors.append(f"篇章 {s['name']} 結束 {s['end']} 小於起點 {s['start']}")
        expected = s["end"] + 1
    total = data_sources.DRAGONBALL["total_episodes"]
    if expected - 1 != total:
        errors.append(f"總集數不符：篇章結尾 {expected - 1} vs total {total}")

    # 金庸 15 部
    if len(data_sources.WUXIA["novels"]) != 15:
        errors.append(f"金庸小說應為 15 部，實際 {len(data_sources.WUXIA['novels'])}")

    # AI 層級唯一
    layers = [a["layer"] for a in data_sources.AI_ARCH["agents"]]
    if len(layers) != len(set(layers)):
        errors.append(f"AI 層級重複：{layers}")

    # 風力參數合理性
    wp = data_sources.WIND_PARAMS
    if not (0 < wp["cp_max"] <= 0.593):
        errors.append(f"cp_max={wp['cp_max']} 應在 (0, 0.593]（Betz 極限）")
    if wp["cut_in_ms"] >= wp["cut_out_ms"]:
        errors.append("cut_in_ms 必須小於 cut_out_ms")
    if wp["rated_speed_ms"] <= wp["cut_in_ms"]:
        errors.append("rated_speed_ms 必須大於 cut_in_ms")

    return errors


# ─────────────────────────────────────────────
# 序列化
# ─────────────────────────────────────────────
def serialize(data: dict, mode: Mode) -> str:
    if mode == "minify":
        return json.dumps(data, ensure_ascii=False, separators=(",", ":")) + "\n"
    return json.dumps(data, ensure_ascii=False, indent=2) + "\n"


# ─────────────────────────────────────────────
# 動作
# ─────────────────────────────────────────────
def run_check(mode: Mode) -> int:
    drifts: list[str] = []
    for name, data in OUTPUTS.items():
        path = DATA_DIR / name
        if not path.exists():
            drifts.append(f"{name} 不存在（請執行 build_data.py）")
            continue
        if path.read_text(encoding="utf-8") != serialize(data, mode):
            drifts.append(f"{name} 內容與 data_sources.py 不一致（mode={mode}）")
    if drifts:
        for d in drifts:
            print(f"::error::{d}", file=sys.stderr)
        print("修正：python scripts/build_data.py", file=sys.stderr)
        return 1
    print(f"✓ {len(OUTPUTS)} 份 JSON 與 data_sources.py 一致（mode={mode}）")
    return 0


def run_build(mode: Mode) -> int:
    DATA_DIR.mkdir(exist_ok=True)
    for name, data in OUTPUTS.items():
        text = serialize(data, mode)
        path = DATA_DIR / name
        path.write_text(text, encoding="utf-8")
        print(f"寫入 {path.relative_to(ROOT)}（{len(text)} bytes, mode={mode}）")
    return 0


# ─────────────────────────────────────────────
# 入口
# ─────────────────────────────────────────────
def main() -> int:
    ap = argparse.ArgumentParser(description=__doc__,
                                 formatter_class=argparse.RawDescriptionHelpFormatter)
    mode_group = ap.add_mutually_exclusive_group()
    mode_group.add_argument("--pretty", action="store_true",
                            help="縮排輸出（預設）")
    mode_group.add_argument("--minify", action="store_true",
                            help="單行壓縮輸出")
    ap.add_argument("--check", action="store_true", help="只檢查，不寫入")
    ap.add_argument("--validate", action="store_true",
                    help="只跑結構驗證，不碰檔案")
    args = ap.parse_args()

    mode: Mode = "minify" if args.minify else "pretty"

    errors = validate()
    if errors:
        for e in errors:
            print(f"::error::{e}", file=sys.stderr)
        return 1

    if args.validate:
        print("✓ 結構驗證通過")
        return 0
    if args.check:
        return run_check(mode)
    return run_build(mode)


if __name__ == "__main__":
    raise SystemExit(main())
```

---

3️⃣ assets/app.js（加入 data/wind.json ＋ 互動式 Cp 曲線）

```javascript
/* ============================================================
 * Lightning Empire｜前端渲染
 * 資料來源：data/dragonball.json、data/wuxia.json、
 *           data/ai_arch.json、data/wind.json
 * fetch 失敗時退回內嵌精簡資料。
 * ============================================================ */

const FALLBACK = {
  dragonball: {
    total_episodes: 291,
    sagas: [
      { order: 1, name: "賽亞人篇",       start: 1,   end: 35  },
      { order: 2, name: "弗利沙篇",       start: 36,  end: 107 },
      { order: 3, name: "加力克二世篇",   start: 108, end: 117 },
      { order: 4, name: "人造人篇",       start: 118, end: 139 },
      { order: 5, name: "西魯（賽魯）篇", start: 140, end: 194 },
      { order: 6, name: "冥界武道大會篇", start: 195, end: 199 },
      { order: 7, name: "魔人普烏篇",     start: 200, end: 291 }
    ],
    characters: []
  },
  wuxia: {
    mnemonic: "飛雪連天射白鹿，笑書神俠倚碧鴛。",
    novels: [],
    trilogy: null,
    schools: []
  },
  ai: { agents: [], platform: null, copilot: null },
  wind: {
    design_angle_deg: 35, rated_speed_ms: 12, cut_in_ms: 3,
    cut_out_ms: 25, cp_max: 0.45, rotor_diameter_m: 80,
    air_density: 1.225, note: "原型曲線，非工程驗證值"
  }
};

async function loadJSON(path, fallback) {
  try {
    const res = await fetch(path, { cache: "no-cache" });
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.warn(`[fetch] ${path} 失敗，使用內嵌 fallback。`, err);
    console.warn("提示：請用 `python -m http.server` 或 GitHub Pages 開啟本頁。");
    return fallback;
  }
}

/* ---------- 七龍珠 Z ---------- */
function renderDragonball(data) {
  const tbody = document.querySelector("#db-table tbody");
  tbody.innerHTML = "";
  (data.sagas || []).forEach((s) => {
    const tr = document.createElement("tr");
    tr.innerHTML =
      `<td>${s.order}</td><td>${s.name}</td><td>第 ${s.start}–${s.end} 集</td>`;
    tbody.appendChild(tr);
  });
  const meta = document.getElementById("db-meta");
  if (meta) {
    meta.textContent =
      `共 ${data.sagas?.length ?? 0} 篇・${data.total_episodes ?? "?"} 集`;
  }
  const box = document.getElementById("db-characters");
  box.innerHTML = "";
  (data.characters || []).forEach((c) => {
    const card = document.createElement("div");
    card.className = "card";
    const extra = [
      (c.skills || []).join("・"),
      (c.forms || []).join("・"),
      c.note
    ].filter(Boolean).join("｜");
    card.innerHTML =
      `<h3>${c.name}</h3><p class="role">${c.role || ""}</p>` +
      (extra ? `<p class="note">${extra}</p>` : "");
    box.appendChild(card);
  });
  if (!data.characters?.length) {
    box.innerHTML = `<p class="note">角色資料需從 data/dragonball.json 載入。</p>`;
  }
}

/* ---------- 金庸 ---------- */
function renderWuxia(data) {
  const motto = document.getElementById("wuxia-mnemonic");
  if (motto) motto.textContent = data.mnemonic || "";

  const ol = document.getElementById("wuxia-list");
  ol.innerHTML = "";
  (data.novels || []).forEach((n) => {
    const li = document.createElement("li");
    li.innerHTML = `《${n.title}》<span class="tag">${n.group || ""}</span>`;
    ol.appendChild(li);
  });
  if (!data.novels?.length) {
    ol.innerHTML = `<li class="note">書目需從 data/wuxia.json 載入。</li>`;
  }

  const tri = document.getElementById("wuxia-trilogy");
  if (tri) {
    if (data.trilogy?.parts?.length) {
      tri.innerHTML =
        `<h3>${data.trilogy.name}</h3><ol>` +
        data.trilogy.parts
          .map((p) => `<li>《${p.title}》— ${(p.leads || []).join("・")}</li>`)
          .join("") + `</ol>`;
    } else {
      tri.innerHTML = "";
    }
  }

  const schools = document.getElementById("wuxia-schools");
  schools.innerHTML = "";
  (data.schools || []).forEach((s) => {
    const card = document.createElement("div");
    card.className = "card";
    card.innerHTML =
      `<h3>${s.name}</h3>` +
      (s.founder ? `<p class="role">開派：${s.founder}</p>` : "") +
      ((s.figures || []).length
        ? `<p class="note">${s.figures.join("・")}</p>`
        : "");
    schools.appendChild(card);
  });
  if (!data.schools?.length) {
    schools.innerHTML = `<p class="note">門派資料需從 data/wuxia.json 載入。</p>`;
  }
}

/* ---------- 3+1 AI ---------- */
function renderAI(data) {
  const box = document.getElementById("ai-cards");
  box.innerHTML = "";

  if (data.copilot) {
    const c = document.createElement("div");
    c.className = "card";
    c.innerHTML =
      `<h3>副駕駛・${data.copilot.owner || ""}</h3>` +
      `<p class="role">human-in-the-loop</p>` +
      `<p class="note">${(data.copilot.duties || []).join("・")}</p>`;
    box.appendChild(c);
  }
  (data.agents || []).forEach((a) => {
    const card = document.createElement("div");
    card.className = "card";
    card.innerHTML =
      `<h3>${a.name}</h3>` +
      `<p class="role">${a.layer}・${a.status || "stub"}</p>` +
      `<p class="note">${(a.duties || []).join("・")}</p>`;
    box.appendChild(card);
  });
  if (data.platform) {
    const p = document.createElement("div");
    p.className = "card";
    p.innerHTML =
      `<h3>${data.platform.name}</h3>` +
      `<p class="role">${data.platform.layer}</p>` +
      `<p class="note">${(data.platform.duties || []).join("・")}</p>`;
    box.appendChild(p);
  }

  const tbody = document.querySelector("#ai-route-table tbody");
  tbody.innerHTML = "";
  const rows = [
    ...(data.agents || []).map((a) => [a.layer, a.name, a.adapter || "—", a.status || "stub"]),
    ...(data.platform ? [[data.platform.layer, data.platform.name, "—", "active"]] : []),
    ...(data.copilot ? [["human-gate", `副駕駛（${data.copilot.owner}）`, "—", "required"]] : [])
  ];
  if (!rows.length) {
    tbody.innerHTML = `<tr><td colspan="4" class="note">需從 data/ai_arch.json 載入。</td></tr>`;
    return;
  }
  rows.forEach(([layer, name, adapter, status]) => {
    const tr = document.createElement("tr");
    tr.innerHTML = `<td>${layer}</td><td>${name}</td><td>${adapter}</td><td>${status}</td>`;
    tbody.appendChild(tr);
  });
}

/* ---------- 風力 Cp 曲線（與 Python 對齊） ---------- */
function cpCurve(v, angle, p) {
  if (v < p.cut_in_ms || v > p.cut_out_ms) return 0;
  const r = v / p.rated_speed_ms;
  const sf = r <= 1 ? Math.pow(r, 1.5) : Math.max(0.55, 1 - 0.25 * (r - 1));
  const af = Math.exp(-Math.pow((angle - p.design_angle_deg) / 18, 2));
  return Math.max(0, Math.min(p.cp_max * sf * af, p.cp_max));
}

function windPowerKw(v, diameter, angle, p) {
  const area = Math.PI * Math.pow(diameter / 2, 2);
  return 0.5 * p.air_density * area * Math.pow(v, 3) * cpCurve(v, angle, p) / 1000;
}

const WIND_SPEEDS = [0, 2, 3, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 25, 28];

function drawWindChart(p, angle, diameter) {
  const svg = document.getElementById("wind-chart");
  if (!svg) return;
  const W = 640, H = 300, PAD = 44;

  const rows = WIND_SPEEDS.map((v) => ({
    v,
    cp: cpCurve(v, angle, p),
    kw: windPowerKw(v, diameter, angle, p)
  }));
  const maxKw = Math.max(...rows.map((r) => r.kw), 1);

  const x = (v) => PAD + (v / 28) * (W - PAD * 2);
  const y = (kw) => H - PAD - (kw / maxKw) * (H - PAD * 2);

  let grid = "";
  for (let i = 0; i <= 4; i++) {
    const gy = PAD + i * (H - PAD * 2) / 4;
    const val = Math.round(maxKw * (1 - i / 4));
    grid += `<line x1="${PAD}" y1="${gy}" x2="${W - PAD}" y2="${gy}" stroke="#2a3340"/>`;
    grid += `<text x="6" y="${gy + 4}" fill="#9aa4b2" font-size="10">${val}</text>`;
  }

  const pts = rows.map((r) => `${x(r.v).toFixed(1)},${y(r.kw).toFixed(1)}`).join(" ");
  const dots = rows.map((r) =>
    `<circle cx="${x(r.v)}" cy="${y(r.kw)}" r="3" fill="#ffd479"/>`
  ).join("");

  svg.innerHTML = `
    ${grid}
    <line x1="${PAD}" y1="${H - PAD}" x2="${W - PAD}" y2="${H - PAD}" stroke="#3a4656"/>
    <line x1="${PAD}" y1="${PAD}" x2="${PAD}" y2="${H - PAD}" stroke="#3a4656"/>
    <polyline points="${pts}" fill="none" stroke="#9fd3ff" stroke-width="2.5"/>
    ${dots}
    <text x="${W - PAD}" y="${H - 12}" fill="#9aa4b2" font-size="10" text-anchor="end">風速 m/s</text>
    <text x="${PAD}" y="20" fill="#ffd479" font-size="12">
      葉片角 ${angle}°・直徑 ${diameter} m・Cp_max ${p.cp_max}
    </text>
  `;

  const table = document.getElementById("wind-table");
  if (table) {
    table.querySelectorAll("tbody tr").forEach((tr) => tr.remove());
    rows.forEach((r) => {
      const tr = document.createElement("tr");
      tr.innerHTML =
        `<td>${r.v}</td><td>${r.cp.toFixed(4)}</td><td>${r.kw.toFixed(2)}</td>`;
      table.querySelector("tbody").appendChild(tr);
    });
  }
}

function renderWind(params) {
  const p = { ...FALLBACK.wind, ...params };

  const angleEl = document.getElementById("wind-angle");
  const diamEl = document.getElementById("wind-diameter");
  const angleVal = document.getElementById("wind-angle-val");
  const diamVal = document.getElementById("wind-diameter-val");

  if (!angleEl || !diamEl) return;

  angleEl.value = p.design_angle_deg;
  diamEl.value = p.rotor_diameter_m;

  const update = () => {
    const a = Number(angleEl.value);
    const d = Number(diamEl.value);
    if (angleVal) angleVal.textContent = `${a}°`;
    if (diamVal) diamVal.textContent = `${d} m`;
    drawWindChart(p, a, d);
  };
  angleEl.addEventListener("input", update);
  diamEl.addEventListener("input", update);
  update();
}

/* ---------- 初始化 ---------- */
document.addEventListener("DOMContentLoaded", async () => {
  const [db, wx, ai, wind] = await Promise.all([
    loadJSON("data/dragonball.json", FALLBACK.dragonball),
    loadJSON("data/wuxia.json",     FALLBACK.wuxia),
    loadJSON("data/ai_arch.json",   FALLBACK.ai),
    loadJSON("data/wind.json",      FALLBACK.wind)
  ]);
  renderDragonball(db);
  renderWuxia(wx);
  renderAI(ai);
  renderWind(wind);

  const status = document.getElementById("data-status");
  if (status) {
    status.textContent =
      "資料來源：data/*.json（若 fetch 失敗則使用內嵌 fallback）";
  }
});
```

---

4️⃣ index.html 風力區塊（取代原本 #wind section）

```html
<!-- 六、風力互動曲線 -->
<section id="wind" class="wide">
  <h2>六、風力 Cp 曲線（互動）</h2>
  <p class="note" id="wind-note">P = 0.5 × ρ × A × v³ × Cp(θ, v)；原型曲線，非工程驗證值。</p>

  <div class="controls">
    <label>
      葉片角
      <input id="wind-angle" type="range" min="0" max="60" step="1" value="35">
      <span id="wind-angle-val">35°</span>
    </label>
    <label>
      轉子直徑
      <input id="wind-diameter" type="range" min="20" max="140" step="1" value="80">
      <span id="wind-diameter-val">80 m</span>
    </label>
  </div>

  <svg id="wind-chart" viewBox="0 0 640 300" preserveAspectRatio="none"></svg>

  <table id="wind-table">
    <thead><tr><th>風速 m/s</th><th>Cp</th><th>功率 kW</th></tr></thead>
    <tbody></tbody>
  </table>
</section>
```

並在 <style> 補上：

```css
.controls{display:flex;gap:24px;flex-wrap:wrap;margin:12px 0;font-size:.9rem}
.controls input[type=range]{vertical-align:middle;width:180px}
#wind-chart{width:100%;height:300px;background:#0b0d11;
  border:1px solid var(--line);border-radius:10px;margin-top:8px}
```

---

5️⃣ 新增 tests/test_data_sources.py

```python
"""saiyan/data_sources.py 結構與版本對齊測試。"""

from saiyan import data_sources as ds


def test_version_is_string():
    assert isinstance(ds.__version__, str)
    assert ds.__version__


def test_all_payloads_share_version():
    for name, payload in [
        ("DRAGONBALL", ds.DRAGONBALL),
        ("WUXIA", ds.WUXIA),
        ("AI_ARCH", ds.AI_ARCH),
        ("WIND_PARAMS", ds.WIND_PARAMS),
    ]:
        assert payload.get("version") == ds.__version__, f"{name} version 不一致"


def test_dragonball_structure():
    assert len(ds.DRAGONBALL["sagas"]) == 7
    assert ds.DRAGONBALL["total_episodes"] == 291


def test_wuxia_has_15_novels():
    assert len(ds.WUXIA["novels"]) == 15


def test_ai_layers_unique():
    layers = [a["layer"] for a in ds.AI_ARCH["agents"]]
    assert len(layers) == len(set(layers))


def test_wind_params_sane():
    wp = ds.WIND_PARAMS
    assert 0 < wp["cp_max"] <= 0.593
    assert wp["cut_in_ms"] < wp["rated_speed_ms"] < wp["cut_out_ms"]
    assert wp["rotor_diameter_m"] > 0
    assert wp["air_density"] > 0
```

---

6️⃣ 新增 tests/test_build_data.py

```python
"""scripts/build_data.py 的行為測試。"""

import json
import sys
from pathlib import Path

import pytest

ROOT = Path(__file__).resolve().parents[1]
SCRIPTS = ROOT / "scripts"
if str(SCRIPTS) not in sys.path:
    sys.path.insert(0, str(SCRIPTS))

import build_data  # noqa: E402


def test_validate_passes():
    assert build_data.validate() == []


def test_serialize_pretty_has_newline():
    text = build_data.serialize({"a": 1}, "pretty")
    assert text.endswith("\n")
    assert "\n  " in text  # 有縮排


def test_serialize_minify_is_single_line():
    text = build_data.serialize({"a": 1, "b": [1, 2]}, "minify")
    body = text.rstrip("\n")
    assert "\n" not in body
    assert json.loads(body) == {"a": 1, "b": [1, 2]}


def test_check_passes_on_repo_data():
    """倉庫內的 data/*.json 應與 data_sources.py 一致（pretty 模式）。"""
    assert build_data.run_check("pretty") == 0
```

---

7️⃣ 收尾清單（回來照順序跑）

```bash
# ── 1. 生成（或重新生成）四份 JSON ──
python scripts/build_data.py

# ── 2. 驗證結構 + 版本對齊 ──
python scripts/build_data.py --validate
python scripts/build_data.py --check

# ── 3. 跑全部測試 ──
python -m pytest -q

# ── 4. 本機預覽（風力滑桿需要 http server） ──
python -m http.server 8000
# 開 http://localhost:8000/ → 滑到「六、風力 Cp 曲線」

# ── 5. commit & push ──
git add -A
git commit -m "feat: 3+1 AI 全套（資料來源、build_data、風力互動、測試、Mermaid lint）"
git push
```

GitHub 設定（一次性）

· Settings → Pages → Source 選 GitHub Actions
· Settings → Environments → 新建 production，勾 Required reviewers 加入自己

選配 Secret（沒有就自動跳過 realtime 層）

Secret 對應
OPENAI_API_KEY copilot/chatgpt.py
GROK_API_KEY copilot/grok.py
DEEPSEEK_API_KEY copilot/deepseek.py

---

📋 本次交付總覽

# 檔案 變更
1 saiyan/data_sources.py 加 __version__，四份資料版本對齊
2 scripts/build_data.py --pretty / --minify / --check / --validate，版本對齊檢查
3 assets/app.js fetch data/wind.json，前端互動 Cp 曲線與功率表
4 index.html #wind 滑桿＋SVG＋表格
5 tests/test_data_sources.py 版本、結構、參數合理性
6 tests/test_build_data.py validate、serialize、check

CI 現在的守門範圍

· plan-doc：Markdown 圍欄、空連結、build_data --check（含版本對齊）、Mermaid 逐塊 mmdc 渲染
· code：compileall ＋ 全部 pytest（sagas／wind／qi／data_sources／build_data）
· human-gate：production environment 人工核可
· deploy：Pages

改了 data_sources.py 忘記跑 build_data.py → PR 直接被擋。改了版本號但某份 JSON 沒同步 → 也被擋。Mermaid 寫壞 → 也被擋。

下一步（有空再說，不急）：

· 把 README.md 的資料表改成從 data/*.json 自動生成（用 scripts/render_readme.py）
· 加 saiyan/qi_power.py 的 CLI 到 main.py 指令集
· Pages 加 404.html 與 sitemap.xml

忙完回來再開。
