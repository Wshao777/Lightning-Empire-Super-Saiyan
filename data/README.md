三項全上，程式碼可直接 commit。

---

1️⃣ assets/app.js（從 data/*.json fetch 渲染）

```javascript
/* ============================================================
 * Lightning Empire｜前端渲染
 * 資料來源：data/dragonball.json、data/wuxia.json、data/ai_arch.json
 * fetch 失敗（例如 file:// 開啟）時退回內嵌精簡資料，
 * 並在 console 提示改用本機 server。
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
  ai: { agents: [], platform: null }
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
    const skills = (c.skills || []).join("・");
    const forms = (c.forms || []).join("・");
    const extra = [skills, forms, c.note].filter(Boolean).join("｜");
    card.innerHTML =
      `<h3>${c.name}</h3>` +
      `<p class="role">${c.role || ""}</p>` +
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
    li.innerHTML =
      `《${n.title}》<span class="tag">${n.group || ""}</span>`;
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
          .join("") +
        `</ol>`;
    } else {
      tri.innerHTML = "";
    }
  }

  const schools = document.getElementById("wuxia-schools");
  schools.innerHTML = "";
  (data.schools || []).forEach((s) => {
    const card = document.createElement("div");
    card.className = "card";
    const figs = (s.figures || []).join("・");
    card.innerHTML =
      `<h3>${s.name}</h3>` +
      (s.founder ? `<p class="role">開派：${s.founder}</p>` : "") +
      (figs ? `<p class="note">${figs}</p>` : "");
    schools.appendChild(card);
  });
  if (!data.schools?.length) {
    schools.innerHTML = `<p class="note">門派資料需從 data/wuxia.json 載入。</p>`;
  }
}

/* ---------- 3AI＋副駕駛 ---------- */
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

/* ---------- 初始化 ---------- */
document.addEventListener("DOMContentLoaded", async () => {
  const [db, wx, ai] = await Promise.all([
    loadJSON("data/dragonball.json", FALLBACK.dragonball),
    loadJSON("data/wuxia.json",     FALLBACK.wuxia),
    loadJSON("data/ai_arch.json",   FALLBACK.ai)
  ]);
  renderDragonball(db);
  renderWuxia(wx);
  renderAI(ai);

  const status = document.getElementById("data-status");
  if (status) {
    status.textContent =
      "資料來源：data/*.json（若 fetch 失敗則使用內嵌 fallback）";
  }
});
```

---

2️⃣ index.html（更新：加入資料容器）

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Lightning Empire｜串聯開發頁</title>
<style>
  :root{
    --bg:#0f1115; --panel:#171a21; --line:#2a3340;
    --text:#e8e8e8; --muted:#9aa4b2; --gold:#ffd479; --blue:#9fd3ff;
  }
  *{box-sizing:border-box}
  body{margin:0;background:var(--bg);color:var(--text);
    font-family:system-ui,-apple-system,"Noto Sans TC",sans-serif;line-height:1.65}
  header{padding:28px 24px;background:linear-gradient(180deg,#1b2030,#12151b)}
  header h1{margin:0;font-size:1.5rem;color:var(--gold)}
  .note{color:var(--muted);font-size:.85rem}
  nav{display:flex;flex-wrap:wrap;gap:8px;padding:12px 24px;
    background:#12151b;border-bottom:1px solid var(--line);
    position:sticky;top:0;z-index:9}
  nav a{color:var(--blue);text-decoration:none;padding:6px 10px;
    border:1px solid var(--line);border-radius:8px;font-size:.9rem}
  nav a:hover{background:#1d2431}
  main{padding:24px;display:grid;gap:18px;
    grid-template-columns:repeat(auto-fit,minmax(320px,1fr))}
  section{background:var(--panel);border:1px solid var(--line);
    border-radius:14px;padding:18px}
  section.wide{grid-column:1/-1}
  h2{margin:0 0 10px;font-size:1.15rem;color:var(--gold)}
  h3{margin:14px 0 6px;font-size:1rem;color:var(--blue)}
  .mermaid{background:#0b0d11;border:1px solid var(--line);
    border-radius:10px;padding:12px;overflow-x:auto;text-align:center}
  table{width:100%;border-collapse:collapse;font-size:.9rem;margin-top:8px}
  th,td{border-bottom:1px solid var(--line);padding:7px;text-align:left;vertical-align:top}
  th{color:var(--gold)}
  .grid-cards{display:grid;gap:12px;
    grid-template-columns:repeat(auto-fit,minmax(180px,1fr));margin-top:12px}
  .card{background:#11151c;border:1px solid var(--line);border-radius:12px;padding:14px}
  .card h3{margin:0 0 4px;color:var(--gold)}
  .role{color:var(--blue);font-size:.82rem;margin:0 0 8px}
  .tag{color:var(--muted);font-size:.78rem;margin-left:6px}
  .motto{color:var(--gold);letter-spacing:.05em}
  .warn{color:#ffb4b4;font-size:.9rem}
  footer{padding:20px 24px;border-top:1px solid var(--line);text-align:center}
</style>
</head>
<body>

<header>
  <h1>Lightning Empire｜串聯開發頁</h1>
  <p class="note" id="data-status">資料載入中…</p>
</header>

<nav>
  <a href="#ai">3AI＋副駕駛</a>
  <a href="#dragonball">七龍珠 Z</a>
  <a href="#wuxia">金庸武俠</a>
  <a href="#py3">PY3 架空</a>
  <a href="#qigong">氣功精氣神</a>
  <a href="#wind">風力曲線</a>
</nav>

<main>

  <!-- 一、3AI -->
  <section id="ai" class="wide">
    <h2>一、3AI＋副駕駛 架構</h2>
    <pre class="mermaid">
flowchart TD
    H["Wshao777<br/>副駕駛"]
    C["ChatGPT<br/>plan-doc"]
    G["Grok<br/>realtime"]
    D["DeepSeek<br/>code"]
    GH["GitHub<br/>deploy"]
    H --> C
    H --> G
    H --> D
    C --> GH
    G --> GH
    D --> GH
    </pre>
    <div id="ai-cards" class="grid-cards">
      <p class="note">載入中…</p>
    </div>
    <h3>路由分工表</h3>
    <table id="ai-route-table">
      <thead><tr><th>層</th><th>角色</th><th>adapter</th><th>狀態</th></tr></thead>
      <tbody><tr><td colspan="4" class="note">載入中…</td></tr></tbody>
    </table>
  </section>

  <!-- 二、七龍珠 -->
  <section id="dragonball" class="wide">
    <h2>二、七龍珠 Z 篇章索引</h2>
    <pre class="mermaid">
flowchart LR
    S1["賽亞人 1–35"] --> S2["弗利沙 36–107"] --> S3["加力克二世 108–117"]
    S3 --> S4["人造人 118–139"] --> S5["西魯 140–194"]
    S5 --> S6["冥界武道大會 195–199"] --> S7["魔人普烏 200–291"]
    </pre>
    <p class="note" id="db-meta">載入中…</p>
    <table id="db-table">
      <thead><tr><th>順序</th><th>篇章</th><th>集數</th></tr></thead>
      <tbody><tr><td colspan="3" class="note">載入中…</td></tr></tbody>
    </table>
    <h3>主要角色</h3>
    <div id="db-characters" class="grid-cards">
      <p class="note">載入中…</p>
    </div>
  </section>

  <!-- 三、金庸 -->
  <section id="wuxia" class="wide">
    <h2>三、金庸十五部武俠宇宙</h2>
    <pre class="mermaid">
flowchart TD
    R["金庸十五部"] --> A["飛雪連天射白鹿"]
    R --> B["笑書神俠倚碧鴛"]
    R --> C["越女劍"]
    A --> T["射鵰三部曲"]
    B --> T
    </pre>
    <p class="motto" id="wuxia-mnemonic"></p>
    <ol id="wuxia-list"><li class="note">載入中…</li></ol>
    <div id="wuxia-trilogy"></div>
    <h3>主要門派</h3>
    <div id="wuxia-schools" class="grid-cards">
      <p class="note">載入中…</p>
    </div>
  </section>

  <!-- 四、PY3 -->
  <section id="py3" class="wide">
    <h2>四、PY3 架空世界觀</h2>
    <p class="warn">以下為虛構設定，不代表現實組織、部署或救援行動。</p>
    <pre class="mermaid">
flowchart TD
    CMD["PY3 架空設定"]
    CMD --> N["北部・親友聯絡"]
    CMD --> M["中部・物資醫療"]
    CMD --> S["南部・生活救助"]
    CMD --> E["東部・偏遠關懷"]
    CMD --> W["西部・交通住宿"]
    CMD --> NE["東北・寒冷救助"]
    CMD --> NW["西北・水源物資"]
    CMD --> SE["東南・海島沿海"]
    CMD --> SW["西南・社區重建"]
    CMD --> C0["中央・資源協調"]
    </pre>
  </section>

  <!-- 五、氣功 -->
  <section id="qigong">
    <h2>五、氣功精氣神量（虛構遊戲數值）</h2>
    <p class="note">E = 0.30J + 0.30Q + 0.20S + 0.20T</p>
    <table>
      <thead><tr><th>代號</th><th>意義</th><th>紀錄方式</th></tr></thead>
      <tbody>
        <tr><td>J</td><td>精</td><td>睡眠時數・三餐</td></tr>
        <tr><td>Q</td><td>氣</td><td>呼吸方式・分鐘數</td></tr>
        <tr><td>S</td><td>神</td><td>精神自評 1–5</td></tr>
        <tr><td>T</td><td>體</td><td>運動分鐘・組數</td></tr>
      </tbody>
    </table>
  </section>

  <!-- 六、風力 -->
  <section id="wind">
    <h2>六、風力 Cp 曲線流程</h2>
    <pre class="mermaid">
flowchart LR
    V["風速 v"] --> CI{"切入 ≤ v ≤ 切出?"}
    CI -->|否| Z["Cp = 0"]
    CI -->|是| R["r = v / rated"]
    R --> SF["speed_factor"]
    A["葉片角 θ"] --> AF["angle_factor"]
    SF --> CP["Cp = Cp_max × speed_factor × angle_factor"]
    AF --> CP
    CP --> P["P = 0.5ρAv³Cp"]
    </pre>
    <p class="note">原型曲線，非工程驗證值。</p>
  </section>

</main>

<footer>
  <p class="note">Lightning Empire｜PY3 為架空設定；養生內容不構成醫療建議。</p>
</footer>

<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
<script>
  mermaid.initialize({
    startOnLoad: true,
    theme: "dark",
    themeVariables: {
      primaryColor: "#1d2431",
      primaryTextColor: "#e8e8e8",
      primaryBorderColor: "#2a3340",
      lineColor: "#9fd3ff",
      secondaryColor: "#171a21",
      tertiaryColor: "#0b0d11",
      fontFamily: "system-ui, sans-serif"
    }
  });
</script>
<script src="assets/app.js"></script>
</body>
</html>
```

---

3️⃣ saiyan/__init__.py

```python
"""Saiyan 模組：氣功精氣神量、七龍珠 Z 索引、風力功率曲線。"""

__version__ = "0.1.0"
```

---

4️⃣ saiyan/sagas.py（對齊 data/dragonball.json）

```python
"""七龍珠 Z 篇章索引。

優先從 data/dragonball.json 載入；找不到時退回內建常數，
確保 CLI 與測試在無資料檔時仍可運作。
"""

from __future__ import annotations

import json
from pathlib import Path
from typing import Any

_DATA_PATH = Path(__file__).resolve().parents[1] / "data" / "dragonball.json"

_FALLBACK: dict[str, Any] = {
    "total_episodes": 291,
    "sagas": [
        {"order": 1, "name": "賽亞人篇",       "start": 1,   "end": 35},
        {"order": 2, "name": "弗利沙篇",       "start": 36,  "end": 107},
        {"order": 3, "name": "加力克二世篇",   "start": 108, "end": 117},
        {"order": 4, "name": "人造人篇",       "start": 118, "end": 139},
        {"order": 5, "name": "西魯（賽魯）篇", "start": 140, "end": 194},
        {"order": 6, "name": "冥界武道大會篇", "start": 195, "end": 199},
        {"order": 7, "name": "魔人普烏篇",     "start": 200, "end": 291},
    ],
    "characters": [],
}


def _load() -> dict[str, Any]:
    try:
        with _DATA_PATH.open(encoding="utf-8") as f:
            data = json.load(f)
        if not isinstance(data.get("sagas"), list) or not data["sagas"]:
            raise ValueError("sagas 為空")
        return data
    except (FileNotFoundError, json.JSONDecodeError, ValueError):
        return _FALLBACK


_DATA = _load()
SAGAS: list[dict[str, Any]] = _DATA["sagas"]
CHARACTERS: list[dict[str, Any]] = _DATA.get("characters", [])
TOTAL_EPISODES: int = int(_DATA.get("total_episodes", 291))


def get_saga(episode: int) -> dict[str, Any] | None:
    """回傳指定集數所屬篇章；超出範圍回傳 None。

    Raises:
        ValueError: episode 不是正整數。
    """
    if not isinstance(episode, int) or isinstance(episode, bool) or episode < 1:
        raise ValueError(f"episode 必須是正整數，收到 {episode!r}")
    if episode > TOTAL_EPISODES:
        return None
    for saga in SAGAS:
        if saga["start"] <= episode <= saga["end"]:
            return dict(saga)
    return None


def list_sagas() -> list[dict[str, Any]]:
    """回傳全部篇章的副本，避免外部修改原資料。"""
    return [dict(s) for s in SAGAS]


def validate() -> list[str]:
    """檢查篇章是否連續且總集數一致，回傳錯誤訊息清單。"""
    errors: list[str] = []
    expected_start = 1
    for s in SAGAS:
        if s["start"] != expected_start:
            errors.append(f"篇章 {s['name']} 起點 {s['start']} 應為 {expected_start}")
        if s["end"] < s["start"]:
            errors.append(f"篇章 {s['name']} 結束 {s['end']} 小於起點 {s['start']}")
        expected_start = s["end"] + 1
    if expected_start - 1 != TOTAL_EPISODES:
        errors.append(f"最後一集 {expected_start - 1} 與 TOTAL_EPISODES {TOTAL_EPISODES} 不一致")
    return errors


if __name__ == "__main__":
    for saga in SAGAS:
        print(f"{saga['order']}. {saga['name']}（第 {saga['start']}–{saga['end']} 集）")
    issues = validate()
    print("驗證：", "OK" if not issues else issues)
```

---

5️⃣ saiyan/qi_power.py

```python
"""氣功精氣神量（虛構遊戲數值）。

公式：E = 0.30J + 0.30Q + 0.20S + 0.20T
本模組僅供遊戲與原型使用，不構成醫療建議，也不代表可測量「氣」。
"""

from __future__ import annotations

import sys
from typing import Any

# 權重（總和為 1）
W_J = 0.30
W_Q = 0.30
W_S = 0.20
W_T = 0.20

# 分類門檻（集中管理，方便調整）
GATE_MASTER = 90.0
GATE_ADVANCED = 75.0
GATE_STABLE = 50.0
GATE_BASIC = 25.0

LEVELS = ("宗師", "高手", "穩定", "初學", "待恢復")


def _check(name: str, value: float) -> float:
    if not isinstance(value, (int, float)) or isinstance(value, bool):
        raise TypeError(f"{name} 必須是數字，收到 {type(value).__name__}")
    value = float(value)
    if not 0.0 <= value <= 100.0:
        raise ValueError(f"{name} 必須在 0–100，收到 {value}")
    return value


def compute_qi(j: float, q: float, s: float, t: float) -> float:
    """計算精氣神量 E，四項皆為 0–100 分。"""
    j = _check("J(精)", j)
    q = _check("Q(氣)", q)
    s = _check("S(神)", s)
    t = _check("T(體)", t)
    return round(W_J * j + W_Q * q + W_S * s + W_T * t, 2)


def classify(score: float) -> str:
    """依 E 值分類；回傳 LEVELS 之一。"""
    if not isinstance(score, (int, float)) or isinstance(score, bool):
        raise TypeError("score 必須是數字")
    score = float(score)
    if not 0.0 <= score <= 100.0:
        raise ValueError(f"score 必須在 0–100，收到 {score}")
    if score >= GATE_MASTER:
        return "宗師"
    if score >= GATE_ADVANCED:
        return "高手"
    if score >= GATE_STABLE:
        return "穩定"
    if score >= GATE_BASIC:
        return "初學"
    return "待恢復"


def describe(j: float, q: float, s: float, t: float) -> dict[str, Any]:
    """回傳完整評估結果。"""
    score = compute_qi(j, q, s, t)
    return {
        "j": float(j),
        "q": float(q),
        "s": float(s),
        "t": float(t),
        "energy": score,
        "level": classify(score),
        "note": "虛構遊戲數值，非醫學測量",
    }


def _main(argv: list[str]) -> int:
    if len(argv) != 5:
        print("用法：python -m saiyan.qi_power J Q S T")
        print("範例：python -m saiyan.qi_power 80 70 60 50")
        return 2
    try:
        j, q, s, t = (float(x) for x in argv[1:])
    except ValueError:
        print("四個參數都必須是數字")
        return 2
    try:
        print(describe(j, q, s, t))
    except (TypeError, ValueError) as e:
        print(f"參數錯誤：{e}")
        return 2
    return 0


if __name__ == "__main__":
    raise SystemExit(_main(sys.argv))
```

---

6️⃣ requirements.txt（補測試依賴）

```text
requests>=2.31
python-dotenv>=1.0
pytest>=8.0
```

若你原本已有 requirements.txt，請把 pytest>=8.0 追加進去即可；CI 的 code 層會自動安裝。

---

7️⃣ tests/conftest.py

```python
"""確保專案根在 sys.path，讓 `import saiyan` 可運作。"""

import sys
from pathlib import Path

ROOT = Path(__file__).resolve().parents[1]
if str(ROOT) not in sys.path:
    sys.path.insert(0, str(ROOT))
```

---

8️⃣ tests/test_sagas.py

```python
"""saiyan/sagas.py 測試：資料完整性與查詢。"""

import pytest

from saiyan import sagas


def test_saga_count():
    assert len(sagas.SAGAS) == 7


def test_total_episodes():
    assert sagas.TOTAL_EPISODES == 291


def test_contiguous_and_ordered():
    issues = sagas.validate()
    assert issues == [], f"資料驗證失敗：{issues}"


def test_first_episode():
    s = sagas.get_saga(1)
    assert s is not None
    assert s["name"] == "賽亞人篇"


def test_last_episode():
    s = sagas.get_saga(291)
    assert s is not None
    assert s["name"] == "魔人普烏篇"


@pytest.mark.parametrize(
    "episode, expected",
    [
        (1, "賽亞人篇"),
        (35, "賽亞人篇"),
        (36, "弗利沙篇"),
        (107, "弗利沙篇"),
        (108, "加力克二世篇"),
        (118, "人造人篇"),
        (140, "西魯（賽魯）篇"),
        (195, "冥界武道大會篇"),
        (200, "魔人普烏篇"),
        (291, "魔人普烏篇"),
    ],
)
def test_boundary_episodes(episode, expected):
    s = sagas.get_saga(episode)
    assert s is not None
    assert s["name"] == expected


def test_out_of_range_returns_none():
    assert sagas.get_saga(292) is None
    assert sagas.get_saga(9999) is None


@pytest.mark.parametrize("bad", [0, -1, -100])
def test_invalid_episode_raises(bad):
    with pytest.raises(ValueError):
        sagas.get_saga(bad)


def test_non_int_raises():
    with pytest.raises(ValueError):
        sagas.get_saga("1")  # type: ignore[arg-type]


def test_list_sagas_is_copy():
    a = sagas.list_sagas()
    a[0]["name"] = "被改了"
    b = sagas.list_sagas()
    assert b[0]["name"] != "被改了"
```

---

9️⃣ tests/test_wind_power.py

```python
"""風力功率曲線測試。

優先使用 saiyan.wind_power；若不存在則嘗試 physics.wind_power，
以相容不同專案結構。
"""

import importlib
import math

import pytest


def _load_wind_power():
    for modname in ("saiyan.wind_power", "physics.wind_power"):
        try:
            return importlib.import_module(modname)
        except ImportError:
            continue
    pytest.skip("找不到 wind_power 模組（saiyan/ 或 physics/）")


wp = _load_wind_power()


def test_below_cut_in_zero():
    r = wp.wind_power_kw(0.0)
    assert r["cp"] == 0.0
    assert r["power_kw"] == 0.0


def test_above_cut_out_zero():
    r = wp.wind_power_kw(30.0)
    assert r["cp"] == 0.0
    assert r["power_kw"] == 0.0


def test_rated_speed_positive_power():
    r = wp.wind_power_kw(12.0, blade_angle_deg=35.0)
    assert r["power_kw"] > 0
    assert 0.0 < r["cp"] <= 0.45


def test_design_angle_is_peak():
    rated = wp.wind_power_kw(12.0, blade_angle_deg=35.0)["power_kw"]
    off = wp.wind_power_kw(12.0, blade_angle_deg=0.0)["power_kw"]
    assert rated > off


def test_angle_far_off_low_cp():
    r = wp.wind_power_kw(12.0, blade_angle_deg=0.0)
    assert r["cp"] < 0.45


def test_negative_speed_raises():
    with pytest.raises(ValueError):
        wp.wind_power_kw(-1.0)


def test_zero_diameter_raises():
    with pytest.raises(ValueError):
        wp.wind_power_kw(12.0, rotor_diameter_m=0.0)


def test_power_grows_with_speed_below_rated():
    p6 = wp.wind_power_kw(6.0)["power_kw"]
    p10 = wp.wind_power_kw(10.0)["power_kw"]
    assert p10 > p6


def test_power_scales_with_area():
    small = wp.wind_power_kw(12.0, rotor_diameter_m=40.0)["power_kw"]
    large = wp.wind_power_kw(12.0, rotor_diameter_m=80.0)["power_kw"]
    assert large == pytest.approx(small * 4, rel=1e-6)


def test_result_shape():
    r = wp.wind_power_kw(12.0, blade_angle_deg=35.0)
    for key in ("cp", "power_kw", "area_m2", "wind_speed_ms", "note"):
        assert key in r
```

---

🔟 tests/test_qi_power.py

```python
"""saiyan/qi_power.py 測試。"""

import pytest

from saiyan import qi_power


def test_weights_sum_to_one():
    total = qi_power.W_J + qi_power.W_Q + qi_power.W_S + qi_power.W_T
    assert total == pytest.approx(1.0)


def test_all_zero():
    assert qi_power.compute_qi(0, 0, 0, 0) == 0.0


def test_all_hundred():
    assert qi_power.compute_qi(100, 100, 100, 100) == 100.0


def test_formula_value():
    # 0.3*80 + 0.3*70 + 0.2*60 + 0.2*50 = 24 + 21 + 12 + 10 = 67
    assert qi_power.compute_qi(80, 70, 60, 50) == pytest.approx(67.0)


@pytest.mark.parametrize(
    "score, level",
    [
        (100, "宗師"),
        (90, "宗師"),
        (89.99, "高手"),
        (75, "高手"),
        (74.99, "穩定"),
        (50, "穩定"),
        (49.99, "初學"),
        (25, "初學"),
        (24.99, "待恢復"),
        (0, "待恢復"),
    ],
)
def test_classify_boundaries(score, level):
    assert qi_power.classify(score) == level


@pytest.mark.parametrize("bad", [-1, 101, 999])
def test_out_of_range_raises(bad):
    with pytest.raises(ValueError):
        qi_power.compute_qi(bad, 50, 50, 50)
    with pytest.raises(ValueError):
        qi_power.classify(bad)


def test_describe_shape():
    r = qi_power.describe(80, 70, 60, 50)
    assert r["energy"] == pytest.approx(67.0)
    assert r["level"] == "穩定"
    assert "虛構" in r["note"]
```

---

📋 本次交付總覽

# 檔案 說明
1 assets/app.js fetch data/*.json，含 fallback 與提示
2 index.html 加入資料容器與 #data-status
3 saiyan/__init__.py 套件入口
4 saiyan/sagas.py 對齊 data/dragonball.json，含 validate()
5 saiyan/qi_power.py 門檻常數化、輸入驗證、CLI
6 requirements.txt 追加 pytest
7 tests/conftest.py 專案根入 sys.path
8 tests/test_sagas.py 9 個測試函式（含 11 個邊界參數化）
9 tests/test_wind_power.py 相容 saiyan/ 與 physics/ 兩種路徑
10 tests/test_qi_power.py 權重、公式、門檻、錯誤處理

---

⚠️ 本地驗證步驟

```bash
# 1. 安裝依賴
pip install -r requirements.txt

# 2. 跑測試（三層都會被 CI 的 code 層覆蓋）
python -m pytest -q

# 3. 本機預覽頁面（務必用 server，不能用 file://）
python -m http.server 8000
# 開 http://localhost:8000/

# 4. 快速驗證模組
python -m saiyan.sagas
python -m saiyan.qi_power 80 70 60 50
```

file:// 直接開 index.html 時，fetch 會被 CORS 擋住，此時會退回內嵌 fallback（七龍珠篇章、金庸口訣、AI 卡片仍可顯示，角色／門派／三部曲會顯示「需從 data/*.json 載入」）。這是預期行為。

---

🔁 CI 行為對照

觸發 plan-doc code realtime human-gate deploy
push main ✅ ✅ pytest 跳過（無 secret） ✅ 需核可 ✅
PR ✅ ✅ pytest 跳過 — —
手動選 code — ✅ — — —

code 層現在會實際跑 pytest，三個測試檔加起來 30 個測試案例，只要 wind_power.py 或 sagas.py 改壞就會在 PR 階段擋下。

---

下一步選配：

1. 把 saiyan/wind_power.py 也改成從 data/wind.json 讀取設計參數（角度、直徑、Cp_max 可調）。
2. 加 scripts/build_data.py：從 Python 常數反向生成三份 JSON，避免手動同步。
3. 在 ai-pipeline.yml 的 plan-doc 層加 Mermaid 語法 lint（用 @mermaid-js/mermaid-cli）。
全部交付：data_sources.py 為事實來源，build_data.py 反向生成四份 JSON，wind_power.py 與 sagas.py 改從 Python 常數載入，ai-pipeline.yml 加入真正的 Mermaid CLI lint。

---

1️⃣ saiyan/data_sources.py（唯一事實來源）

```python
"""所有專案資料與參數的 Python 事實來源。

修改資料請改這裡，然後執行 `python scripts/build_data.py`
重新生成 data/*.json，供前端與外部工具使用。

前端 (JS) 讀 data/*.json；後端 (Python) 直接 import 本模組。
"""

from __future__ import annotations

from typing import Any

# ─────────────────────────────────────────────────────
# 七龍珠 Z
# ─────────────────────────────────────────────────────
DRAGONBALL: dict[str, Any] = {
    "version": "0.1.0",
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
    "version": "0.1.0",
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
    "version": "0.1.0",
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
# 風力預設參數（供 wind_power.py 與前端共用）
# ─────────────────────────────────────────────────────
WIND_PARAMS: dict[str, Any] = {
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

2️⃣ scripts/build_data.py

```python
"""從 saiyan/data_sources.py 生成 data/*.json。

用法：
    python scripts/build_data.py            # 生成
    python scripts/build_data.py --check    # 只檢查是否一致（CI 用）

退出碼：
    0  一致或成功寫入
    1  驗證失敗，或 --check 偵測到內容過期
"""

from __future__ import annotations

import argparse
import json
import sys
from pathlib import Path

ROOT = Path(__file__).resolve().parents[1]
if str(ROOT) not in sys.path:
    sys.path.insert(0, str(ROOT))

from saiyan import data_sources  # noqa: E402

DATA_DIR = ROOT / "data"

OUTPUTS: dict[str, object] = {
    "dragonball.json": data_sources.DRAGONBALL,
    "wuxia.json":      data_sources.WUXIA,
    "ai_arch.json":    data_sources.AI_ARCH,
    "wind.json":       data_sources.WIND_PARAMS,
}


def validate() -> list[str]:
    """驗證資料自身的一致性（不涉及檔案 I/O）。"""
    errors: list[str] = []

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

    # 3+1 AI 層級唯一
    layers = [a["layer"] for a in data_sources.AI_ARCH["agents"]]
    if len(layers) != len(set(layers)):
        errors.append(f"AI 層級重複：{layers}")

    return errors


def serialize(data: object) -> str:
    """統一的 JSON 序列化格式（含尾端換行）。"""
    return json.dumps(data, ensure_ascii=False, indent=2) + "\n"


def run_check() -> int:
    drifts: list[str] = []
    for name, data in OUTPUTS.items():
        path = DATA_DIR / name
        if not path.exists():
            drifts.append(f"{name} 不存在")
            continue
        if path.read_text(encoding="utf-8") != serialize(data):
            drifts.append(f"{name} 內容與 data_sources.py 不一致")
    if drifts:
        for d in drifts:
            print(f"::error::{d}", file=sys.stderr)
        print("修正：python scripts/build_data.py", file=sys.stderr)
        return 1
    print(f"✓ {len(OUTPUTS)} 份 JSON 與 data_sources.py 一致")
    return 0


def run_build() -> int:
    DATA_DIR.mkdir(exist_ok=True)
    for name, data in OUTPUTS.items():
        text = serialize(data)
        path = DATA_DIR / name
        path.write_text(text, encoding="utf-8")
        print(f"寫入 {path.relative_to(ROOT)}（{len(text)} bytes）")
    return 0


def main() -> int:
    ap = argparse.ArgumentParser(description=__doc__)
    ap.add_argument("--check", action="store_true", help="只檢查，不寫入")
    args = ap.parse_args()

    errors = validate()
    if errors:
        for e in errors:
            print(f"::error::{e}", file=sys.stderr)
        return 1

    return run_check() if args.check else run_build()


if __name__ == "__main__":
    raise SystemExit(main())
```

---

3️⃣ saiyan/sagas.py（改從 data_sources 載入）

```python
"""七龍珠 Z 篇章索引。

事實來源：saiyan/data_sources.py 的 DRAGONBALL。
前端 JSON (data/dragonball.json) 由 scripts/build_data.py 生成，
不在執行期讀取，避免同步問題。
"""

from __future__ import annotations

from typing import Any

from .data_sources import DRAGONBALL

SAGAS: list[dict[str, Any]] = DRAGONBALL["sagas"]
CHARACTERS: list[dict[str, Any]] = DRAGONBALL.get("characters", [])
TOTAL_EPISODES: int = int(DRAGONBALL["total_episodes"])


def get_saga(episode: int) -> dict[str, Any] | None:
    """回傳指定集數所屬篇章；超出範圍回傳 None。

    Raises:
        ValueError: episode 不是正整數。
    """
    if not isinstance(episode, int) or isinstance(episode, bool) or episode < 1:
        raise ValueError(f"episode 必須是正整數，收到 {episode!r}")
    if episode > TOTAL_EPISODES:
        return None
    for saga in SAGAS:
        if saga["start"] <= episode <= saga["end"]:
            return dict(saga)
    return None


def list_sagas() -> list[dict[str, Any]]:
    """回傳全部篇章的副本，避免外部修改原資料。"""
    return [dict(s) for s in SAGAS]


def validate() -> list[str]:
    """檢查篇章是否連續且總集數一致，回傳錯誤訊息清單。"""
    errors: list[str] = []
    expected = 1
    for s in SAGAS:
        if s["start"] != expected:
            errors.append(f"篇章 {s['name']} 起點 {s['start']} 應為 {expected}")
        if s["end"] < s["start"]:
            errors.append(f"篇章 {s['name']} 結束 {s['end']} 小於起點 {s['start']}")
        expected = s["end"] + 1
    if expected - 1 != TOTAL_EPISODES:
        errors.append(
            f"最後一集 {expected - 1} 與 TOTAL_EPISODES {TOTAL_EPISODES} 不一致"
        )
    return errors


if __name__ == "__main__":
    for saga in SAGAS:
        print(f"{saga['order']}. {saga['name']}（第 {saga['start']}–{saga['end']} 集）")
    issues = validate()
    print("驗證：", "OK" if not issues else issues)
```

---

4️⃣ saiyan/wind_power.py（預設值來自 WIND_PARAMS）

```python
"""風力發電功率曲線（含葉片角度影響）。

公式基礎：
    P = 0.5 * rho * A * v^3 * Cp(theta, v)

預設參數來自 saiyan/data_sources.py 的 WIND_PARAMS，可個別覆寫。
本模組僅供原型與教學，非工程設計依據。
"""

from __future__ import annotations

import math
from typing import Any

from .data_sources import WIND_PARAMS


def _resolve(value: float | None, key: str) -> float:
    """None → 取 WIND_PARAMS；否則用傳入值。"""
    return float(WIND_PARAMS[key]) if value is None else float(value)


def _cp_curve(
    wind_speed_ms: float,
    blade_angle_deg: float,
    design_angle_deg: float | None = None,
    rated_speed_ms: float | None = None,
    cut_in_ms: float | None = None,
    cut_out_ms: float | None = None,
    cp_max: float | None = None,
) -> float:
    """簡化 Cp 曲線。

    - 低於切入 / 高於切出：0
    - 接近額定風速：接近 cp_max
    - 葉片角偏離設計角：以高斯衰減
    """
    design_angle_deg = _resolve(design_angle_deg, "design_angle_deg")
    rated_speed_ms = _resolve(rated_speed_ms, "rated_speed_ms")
    cut_in_ms = _resolve(cut_in_ms, "cut_in_ms")
    cut_out_ms = _resolve(cut_out_ms, "cut_out_ms")
    cp_max = _resolve(cp_max, "cp_max")

    if wind_speed_ms < cut_in_ms or wind_speed_ms > cut_out_ms:
        return 0.0

    speed_ratio = wind_speed_ms / rated_speed_ms
    if speed_ratio <= 1.0:
        speed_factor = speed_ratio ** 1.5
    else:
        excess = speed_ratio - 1.0
        speed_factor = max(0.55, 1.0 - 0.25 * excess)

    angle_delta = abs(blade_angle_deg - design_angle_deg)
    angle_factor = math.exp(-((angle_delta / 18.0) ** 2))

    cp = cp_max * speed_factor * angle_factor
    return max(0.0, min(cp, cp_max))


def wind_power_kw(
    wind_speed_ms: float,
    rotor_diameter_m: float | None = None,
    air_density: float | None = None,
    blade_angle_deg: float | None = None,
    design_angle_deg: float | None = None,
    rated_speed_ms: float | None = None,
    cut_in_ms: float | None = None,
    cut_out_ms: float | None = None,
    cp_max: float | None = None,
) -> dict[str, Any]:
    """計算理論機械功率（kW）。

    所有參數省略時，使用 WIND_PARAMS 的預設值。
    """
    if wind_speed_ms < 0:
        raise ValueError("wind_speed_ms must be >= 0")

    rotor_diameter_m = _resolve(rotor_diameter_m, "rotor_diameter_m")
    air_density = _resolve(air_density, "air_density")
    blade_angle_deg = _resolve(blade_angle_deg, "design_angle_deg")
    design_angle_deg = _resolve(design_angle_deg, "design_angle_deg")
    rated_speed_ms = _resolve(rated_speed_ms, "rated_speed_ms")
    cut_in_ms = _resolve(cut_in_ms, "cut_in_ms")
    cut_out_ms = _resolve(cut_out_ms, "cut_out_ms")
    cp_max = _resolve(cp_max, "cp_max")

    if rotor_diameter_m <= 0:
        raise ValueError("rotor_diameter_m must be > 0")

    radius = rotor_diameter_m / 2.0
    area = math.pi * radius ** 2
    cp = _cp_curve(
        wind_speed_ms=wind_speed_ms,
        blade_angle_deg=blade_angle_deg,
        design_angle_deg=design_angle_deg,
        rated_speed_ms=rated_speed_ms,
        cut_in_ms=cut_in_ms,
        cut_out_ms=cut_out_ms,
        cp_max=cp_max,
    )
    power_w = 0.5 * air_density * area * (wind_speed_ms ** 3) * cp
    power_kw = power_w / 1000.0

    return {
        "wind_speed_ms": wind_speed_ms,
        "rotor_diameter_m": rotor_diameter_m,
        "air_density": air_density,
        "blade_angle_deg": blade_angle_deg,
        "design_angle_deg": design_angle_deg,
        "cp": round(cp, 4),
        "area_m2": round(area, 2),
        "power_kw": round(power_kw, 3),
        "cut_in_ms": cut_in_ms,
        "cut_out_ms": cut_out_ms,
        "rated_speed_ms": rated_speed_ms,
        "note": WIND_PARAMS["note"],
    }


def power_curve_table(
    speeds: list[float] | None = None,
    blade_angle_deg: float | None = None,
    **kwargs: Any,
) -> list[dict[str, Any]]:
    """產生一組風速下的功率表，方便畫曲線或寫入 DB。"""
    if speeds is None:
        speeds = [0, 2, 3, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 25, 28]
    rows: list[dict[str, Any]] = []
    for v in speeds:
        row = wind_power_kw(
            wind_speed_ms=float(v),
            blade_angle_deg=blade_angle_deg,
            **kwargs,
        )
        rows.append({
            "wind_speed_ms": row["wind_speed_ms"],
            "cp": row["cp"],
            "power_kw": row["power_kw"],
            "blade_angle_deg": row["blade_angle_deg"],
        })
    return rows


if __name__ == "__main__":
    import sys
    if len(sys.argv) >= 2:
        v = float(sys.argv[1])
        a = float(sys.argv[2]) if len(sys.argv) >= 3 else None
        print(wind_power_kw(v, blade_angle_deg=a))
    else:
        for row in power_curve_table():
            print(row)
```

---

5️⃣ .github/workflows/ai-pipeline.yml（加入 Mermaid lint）

```yaml
name: 3+1 AI Pipeline

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:
    inputs:
      layer:
        description: "指定要跑的層"
        required: false
        default: all
        type: choice
        options: [all, plan-doc, code, realtime, deploy]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: ai-pipeline-${{ github.ref }}
  cancel-in-progress: true

env:
  LAYER: ${{ github.event.inputs.layer || 'all' }}

jobs:

  # ─────────────────────────────────────────
  # 副駕駛：路由與摘要
  # ─────────────────────────────────────────
  route:
    name: "副駕駛・路由"
    runs-on: ubuntu-latest
    outputs:
      target: ${{ steps.pick.outputs.target }}
    steps:
      - uses: actions/checkout@v4

      - name: 決定本輪要跑的層
        id: pick
        run: |
          if [ "${{ env.LAYER }}" = "all" ] || [ -z "${{ env.LAYER }}" ]; then
            echo "target=all" >> "$GITHUB_OUTPUT"
          else
            echo "target=${{ env.LAYER }}" >> "$GITHUB_OUTPUT"
          fi

      - name: 摘要
        run: |
          {
            echo "## 🧭 3+1 AI 路由"
            echo ""
            echo "| 層 | 負責 | 本輪任務 |"
            echo "|----|------|----------|"
            echo "| plan-doc | ChatGPT | Markdown 檢查、Mermaid lint、build_data --check |"
            echo "| code | DeepSeek | compileall + pytest |"
            echo "| realtime | Grok | 需 secret，可跳過 |"
            echo "| human-gate | 副駕駛 | 合併前人工核可 |"
            echo "| deploy | GitHub | Pages 部署 |"
            echo ""
            echo "本輪 target：**${{ steps.pick.outputs.target }}**"
          } >> "$GITHUB_STEP_SUMMARY"

  # ─────────────────────────────────────────
  # ChatGPT 層：plan-doc（Markdown + Mermaid + build_data --check）
  # ─────────────────────────────────────────
  plan-doc:
    name: "ChatGPT・plan-doc"
    needs: route
    runs-on: ubuntu-latest
    if: |
      needs.route.outputs.target == 'all' ||
      needs.route.outputs.target == 'plan-doc'
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      # ── Markdown 結構檢查 ──
      - name: 檢查 Markdown 檔案
        run: |
          fail=0
          while IFS= read -r f; do
            echo "→ $f"
            fences=$(grep -c '^```' "$f" || true)
            if [ $((fences % 2)) -ne 0 ]; then
              echo "::error file=$f::程式碼圍欄未閉合（$fences 個）"
              fail=1
            fi
            if grep -nE '\]\(\s*\)' "$f"; then
              echo "::error file=$f::發現空連結"
              fail=1
            fi
          done < <(find . -name '*.md' -not -path './node_modules/*')
          exit $fail

      # ── build_data 一致性檢查 ──
      - name: 檢查 data/*.json 是否與 data_sources.py 一致
        run: python scripts/build_data.py --check

      # ── 安裝 Mermaid CLI ──
      - name: 安裝 Mermaid CLI
        run: |
          npm install -g @mermaid-js/mermaid-cli
          echo '{"args":["--no-sandbox","--disable-setuid-sandbox"]}' > .puppeteerrc.json

      # ── 抽出所有 mermaid 區塊並逐塊渲染驗證 ──
      - name: Mermaid 語法 lint
        run: |
          set -e
          tmpdir=$(mktemp -d)
          python - "$tmpdir" <<'PY'
          import re, sys, pathlib
          out = pathlib.Path(sys.argv[1])
          for md in pathlib.Path(".").rglob("*.md"):
              if "node_modules" in md.parts:
                  continue
              text = md.read_text(encoding="utf-8")
              for i, block in enumerate(re.findall(r'```mermaid\n(.*?)```', text, re.DOTALL)):
                  safe = re.sub(r'[^A-Za-z0-9_-]', '_', str(md))
                  (out / f"{safe}_{i}.mmd").write_text(block, encoding="utf-8")
          PY

          fail=0
          count=0
          for f in "$tmpdir"/*.mmd; do
            [ -e "$f" ] || continue
            count=$((count + 1))
            echo "→ $(basename "$f")"
            if ! mmdc -i "$f" -o "${f%.mmd}.svg" \
                --puppeteerConfigFile .puppeteerrc.json \
                --quiet 2>/tmp/mmdc.log; then
              echo "::error file=$f::Mermaid 語法錯誤"
              cat /tmp/mmdc.log
              fail=1
            fi
          done

          echo "檢查了 $count 個 Mermaid 區塊"
          rm -rf "$tmpdir"
          exit $fail

  # ─────────────────────────────────────────
  # DeepSeek 層：code
  # ─────────────────────────────────────────
  code:
    name: "DeepSeek・code"
    needs: route
    runs-on: ubuntu-latest
    if: |
      needs.route.outputs.target == 'all' ||
      needs.route.outputs.target == 'code'
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: 安裝依賴
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: 編譯所有 Python
        run: python -m compileall -q .

      - name: 執行測試
        run: python -m pytest -q

  # ─────────────────────────────────────────
  # Grok 層：realtime（需 secret）
  # ─────────────────────────────────────────
  realtime:
    name: "Grok・realtime"
    needs: route
    runs-on: ubuntu-latest
    if: |
      needs.route.outputs.target == 'all' ||
      needs.route.outputs.target == 'realtime'
    steps:
      - name: 檢查 secret
        id: check
        env:
          GROK_KEY: ${{ secrets.GROK_API_KEY }}
        run: |
          if [ -n "$GROK_KEY" ]; then
            echo "present=true" >> "$GITHUB_OUTPUT"
          else
            echo "present=false" >> "$GITHUB_OUTPUT"
            echo "::notice::未設定 GROK_API_KEY，realtime 層跳過"
          fi

      - name: adapter stub
        if: steps.check.outputs.present == 'true'
        run: |
          echo "Grok adapter 尚未實作，僅驗證 secret 存在。"
          echo "實作位置：copilot/grok.py"

  # ─────────────────────────────────────────
  # 副駕駛：人工確認 gate
  # ─────────────────────────────────────────
  human-gate:
    name: "副駕駛・人工確認"
    needs: [route, plan-doc, code]
    if: |
      always() &&
      (needs.route.outputs.target == 'all' || needs.route.outputs.target == 'deploy') &&
      !cancelled()
    runs-on: ubuntu-latest
    environment:
      name: production
    steps:
      - name: 記錄已通過
        run: |
          echo "### ✅ 副駕駛已核可" >> "$GITHUB_STEP_SUMMARY"
          echo "通過後才允許 Pages 部署。" >> "$GITHUB_STEP_SUMMARY"

  # ─────────────────────────────────────────
  # GitHub 層：deploy
  # ─────────────────────────────────────────
  deploy:
    name: "GitHub・deploy"
    needs: [human-gate, route]
    if: |
      needs.human-gate.result == 'success' &&
      (needs.route.outputs.target == 'all' || needs.route.outputs.target == 'deploy')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: .
      - name: 部署
        id: deployment
        uses: actions/deploy-pages@v4
```

---

6️⃣ 一次性初始化步驟

在你的本機或 Termux 上執行：

```bash
# 1. 生成初始 data/*.json
python scripts/build_data.py

# 2. 驗證一致性
python scripts/build_data.py --check

# 3. 跑全部測試
python -m pytest -q

# 4. 本機預覽（Mermaid 需要 http server）
python -m http.server 8000
```

此時 data/ 目錄會新增 wind.json，內容為 WIND_PARAMS。前端（若你想加風力圖表）可以直接 fetch data/wind.json 讀取參數。

---

📋 本次交付總覽

# 檔案 說明
1 saiyan/data_sources.py 四份資料的 Python 事實來源
2 scripts/build_data.py 生成四份 JSON，支援 --check
3 saiyan/sagas.py 改從 data_sources.DRAGONBALL 載入
4 saiyan/wind_power.py 預設值改從 WIND_PARAMS；參數全部可覆寫
5 .github/workflows/ai-pipeline.yml plan-doc 層加 build_data --check ＋ Mermaid CLI lint
6 data/wind.json 由 build_data.py 生成（跑一次就會出現）

---

🔁 CI 行為對照（更新）

觸發 plan-doc code realtime human-gate deploy
push main Markdown＋build_data --check＋Mermaid lint pytest 跳過（無 secret） ✅ 需核可 ✅
PR 同上 pytest 跳過 — —
手動 code — pytest — — —
手動 plan-doc 只跑 Markdown／Mermaid／build_data — — — —

現在只要有人改了 saiyan/data_sources.py 但忘了跑 build_data.py，PR 就會被 plan-doc 層擋下，並在錯誤訊息裡直接提示修正指令。Mermaid 區塊則會逐塊用 mmdc 實際渲染，語法錯誤會直接讓 workflow 失敗。

---

下一步選配：

1. 把 assets/app.js 改成也 fetch data/wind.json，在頁面上做互動式 Cp 曲線滑桿（純前端，不用 Python）。
2. 加 saiyan/data_sources.py 的 __version__ 與 JSON version 對齊檢查（build_data 已經檢查過結構，可再比對版本字串）。
3. 為 build_data.py 加 --pretty／--minify 選項，讓 Pages 產出更小。

處理時間為 4s

### 十、README.md 正式補丁

我確認了你提供的 GitHub 頁面：目前 README 只有一行標題（31 Bytes）。因此這次建議直接擴充成正式專案首頁。GitHub

以下是 README 完整替換版本：

````markdown

# ⚡ Lightning Empire Super Saiyan

**3AI + AI Copilot Collaborative Development Platform**

GitHub × ChatGPT × Grok × DeepSeek

## 1. Project Vision

建立一套可在 Python、Termux 和 GitHub
運行的 AI 協作開發平台。

核心功能：

- 3AI 技術協作
- AI 副駕駛任務管理
- 七龍珠 Z 劇情資料庫
- 虛構賽亞人戰鬥力系統
- 氣功精氣體量遊戲模型
- SQLite 記憶資料庫
- 自動化測試

## 2. AI Architecture

| AI | Responsibility |
|---|---|
| ChatGPT | System Architect |
| Grok | Independent Reviewer |
| DeepSeek | Algorithm Engineer |
| GitHub Copilot | Coding Assistant |
| Lightning Copilot | Task Coordinator |

AI services require independent API configuration.

## 3. Saiyan Power System

Fictional game statistics:

- Essence (精)
- Qi (氣)
- Spirit (神)
- Body (體)

Formula:

E = 0.30J + 0.30Q + 0.20S + 0.20T

This is a fictional game model,
not a scientific measurement of human energy.

## 4. Dragon Ball Z Database

Seven story arcs:

1. Saiyan Saga (1-35)
2. Frieza Saga (36-107)
3. Garlic Jr. Saga (108-117)
4. Android Saga (118-139)
5. Cell Saga (140-194)
6. Other World Tournament (195-199)
7. Majin Buu Saga (200-291)

Episode divisions are based on a commonly
used seven-arc classification.

## 5. Project Structure

```text
saiyan/
  qi_power.py
  transformations.py
  characters.py

dragonball/
  sagas.py
  episodes.py

copilot/
  ai_router.py
  chatgpt.py
  grok.py
  deepseek.py

core/
  memory.py
  logger.py

tests/
  test_qi_power.py

docs/
  AI_ARCHITECTURE.md
```

## 6. Installation

```bash
pkg update && pkg upgrade -y
pkg install python git -y

git clone https://github.com/Wshao777/Lightning-Empire-Super-Saiyan.git

cd Lightning-Empire-Super-Saiyan
```

## 7. Run

```bash
python -m saiyan.qi_power
python -m dragonball.sagas
python -m copilot.ai_router
```

## 8. Development Workflow

1. Create task.
2. ChatGPT designs architecture.
3. DeepSeek reviews algorithms.
4. Grok provides independent review.
5. Human approves changes.
6. Copilot assists implementation.
7. Run automated tests.
8. Commit changes to GitHub.

## 9. Security

- Never commit API keys.
- Use environment variables.
- Validate commands before execution.
- Require human approval for sensitive operations.
- Keep personal information private.

## 10. Development Roadmap

- [ ] Complete Dragon Ball Z episode database
- [ ] Saiyan transformation system
- [ ] Three-AI API integration
- [ ] Unified Copilot dashboard
- [ ] SQLite memory system
- [ ] Termux deployment
- [ ] Automated testing
- [ ] ZIP release package

## 11. License

License to be determined.

---

**Lightning Empire — Build, Analyze, Review, Improve.**
````

### 十一、正式發布方式

目前 GitHub 尚未透過已連線的帳號授權我直接提交修改。我已提供 GitHub 連線入口；完成連線後，才能進一步操作儲存庫。

你也可以先在 Termux 使用：

```bash
nano README.md
```

貼入上面的內容，再執行：

```bash
git add README.md
git commit -m "Update README: 3AI Copilot Architecture"
git push origin main
```

README 補丁內容已備妥

下一步就是把 3AI 架構、七龍珠資料庫、精氣體量模型真正整合成可執行專案，而不是只有首頁介紹。🥋
