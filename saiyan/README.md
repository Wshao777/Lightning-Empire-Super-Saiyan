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

全套交付：三份資料檔 ＋ Pages 部署 ＋ 3+1 AI 流水線 ＋ copilot adapter stub。所有內容可直接 commit。

---

1️⃣ data/dragonball.json

```json
{
  "version": "0.1.0",
  "title": "七龍珠 Z 七大篇章索引",
  "total_episodes": 291,
  "note": "採常見七大篇章劃分；不同地區分篇可能略有差異。",
  "sagas": [
    { "order": 1, "name": "賽亞人篇",       "start": 1,   "end": 35  },
    { "order": 2, "name": "弗利沙篇",       "start": 36,  "end": 107 },
    { "order": 3, "name": "加力克二世篇",   "start": 108, "end": 117 },
    { "order": 4, "name": "人造人篇",       "start": 118, "end": 139 },
    { "order": 5, "name": "西魯（賽魯）篇", "start": 140, "end": 194 },
    { "order": 6, "name": "冥界武道大會篇", "start": 195, "end": 199 },
    { "order": 7, "name": "魔人普烏篇",     "start": 200, "end": 291 }
  ],
  "characters": [
    { "name": "孫悟空", "role": "主角・賽亞人戰士", "skills": ["龜派氣功", "界王拳", "元氣玉", "超級賽亞人變身"] },
    { "name": "達爾（貝吉塔）", "role": "賽亞人王子", "note": "從對手成為重要盟友" },
    { "name": "孫悟飯", "role": "悟空的兒子", "note": "西魯遊戲展現超級賽亞人 2" },
    { "name": "弗利沙", "role": "宇宙帝王", "note": "那美克星篇重要反派" },
    { "name": "西魯（賽魯）", "role": "生化人造戰士", "note": "西魯遊戲篇核心反派" },
    { "name": "魔人普烏", "role": "魔人篇主要反派", "forms": ["胖普烏", "超級普烏", "小普烏"] }
  ]
}
```

---

2️⃣ data/wuxia.json

```json
{
  "version": "0.1.0",
  "title": "金庸十五部武俠宇宙",
  "mnemonic": "飛雪連天射白鹿，笑書神俠倚碧鴛。",
  "novels": [
    { "order": 1,  "title": "飛狐外傳",   "group": "飛雪連天射白鹿" },
    { "order": 2,  "title": "雪山飛狐",   "group": "飛雪連天射白鹿" },
    { "order": 3,  "title": "連城訣",     "group": "飛雪連天射白鹿" },
    { "order": 4,  "title": "天龍八部",   "group": "飛雪連天射白鹿" },
    { "order": 5,  "title": "射鵰英雄傳", "group": "飛雪連天射白鹿" },
    { "order": 6,  "title": "白馬嘯西風", "group": "飛雪連天射白鹿" },
    { "order": 7,  "title": "鹿鼎記",     "group": "飛雪連天射白鹿" },
    { "order": 8,  "title": "笑傲江湖",   "group": "笑書神俠倚碧鴛" },
    { "order": 9,  "title": "書劍恩仇錄", "group": "笑書神俠倚碧鴛" },
    { "order": 10, "title": "神鵰俠侶",   "group": "笑書神俠倚碧鴛" },
    { "order": 11, "title": "俠客行",     "group": "笑書神俠倚碧鴛" },
    { "order": 12, "title": "倚天屠龍記", "group": "笑書神俠倚碧鴛" },
    { "order": 13, "title": "碧血劍",     "group": "笑書神俠倚碧鴛" },
    { "order": 14, "title": "鴛鴦刀",     "group": "笑書神俠倚碧鴛" },
    { "order": 15, "title": "越女劍",     "group": "獨立短篇" }
  ],
  "trilogy": {
    "name": "射鵰三部曲",
    "parts": [
      { "title": "射鵰英雄傳", "leads": ["郭靖", "黃蓉"] },
      { "title": "神鵰俠侶",   "leads": ["楊過", "小龍女"] },
      { "title": "倚天屠龍記", "leads": ["張無忌", "趙敏"] }
    ]
  },
  "schools": [
    { "name": "少林",   "figures": ["覺遠大師", "少林三渡", "空見神僧"] },
    { "name": "武當",   "founder": "張三丰", "figures": ["宋遠橋", "俞蓮舟", "俞岱巖", "張松溪", "張翠山", "殷梨亭", "莫聲谷"] },
    { "name": "峨嵋",   "founder": "郭襄",   "figures": ["滅絕師太", "周芷若"] },
    { "name": "明教",   "figures": ["張無忌", "楊逍", "范遙", "四大法王"] },
    { "name": "丐幫",   "figures": ["洪七公", "黃蓉", "史火龍"] },
    { "name": "古墓派", "founder": "林朝英", "figures": ["小龍女", "楊過"] }
  ],
  "zhang_sanfeng": {
    "title": "武當派開山祖師",
    "master": "覺遠大師",
    "peers": ["郭襄", "楊過（少年張君寶有交集）"],
    "skills": ["太極拳", "太極劍", "武當九陽功", "純陽無極功", "綿掌", "梯雲縱", "真武七截陣"],
    "grand_disciples": ["張無忌", "宋青書"]
  }
}
```

---

3️⃣ data/ai_arch.json

```json
{
  "version": "0.1.0",
  "project": "Lightning-Empire-Super-Saiyan",
  "boundary": "軟體分層標籤，非外部模型授權或商業關係。",
  "copilot": {
    "role": "human-in-the-loop",
    "owner": "Wshao777",
    "duties": ["決策", "驗收", "邊界把關", "合併前最後確認"]
  },
  "agents": [
    {
      "name": "ChatGPT",
      "layer": "plan-doc",
      "tags": ["architect-note", "content", "spec", "doc"],
      "duties": ["需求拆解", "文件撰寫", "內容整理", "副駕駛協作"],
      "adapter": "copilot/chatgpt.py",
      "status": "stub"
    },
    {
      "name": "Grok",
      "layer": "realtime",
      "tags": ["fact-check", "trend", "brainstorm"],
      "duties": ["即時資訊", "事實核對", "外部視角"],
      "adapter": "copilot/grok.py",
      "status": "stub"
    },
    {
      "name": "DeepSeek",
      "layer": "code",
      "tags": ["physics-local", "python", "numerics", "termux"],
      "duties": ["程式實作", "數值計算", "Termux 整合"],
      "adapter": "copilot/deepseek.py",
      "status": "stub"
    }
  ],
  "platform": {
    "name": "GitHub",
    "layer": "deploy",
    "repo": "Wshao777/Lightning-Empire-Super-Saiyan",
    "duties": ["版本控制", "Actions CI", "Pages 部署", "Issue 追蹤"]
  },
  "workflows": {
    "pages": ".github/workflows/pages.yml",
    "ai_pipeline": ".github/workflows/ai-pipeline.yml"
  }
}
```

---

4️⃣ .github/workflows/pages.yml（Pages 部署）

```yaml
name: Deploy Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    name: 建置靜態站
    runs-on: ubuntu-latest
    steps:
      - name: 取出原始碼
        uses: actions/checkout@v4

      - name: 設定 Pages
        uses: actions/configure-pages@v5

      - name: 上傳 artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .

  deploy:
    name: 部署至 Pages
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: 部署
        id: deployment
        uses: actions/deploy-pages@v4
```

啟用：Repo → Settings → Pages → Source 選 GitHub Actions。

---

5️⃣ .github/workflows/ai-pipeline.yml（3+1 AI 自動化流水線）

```yaml
name: 3+1 AI Pipeline

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:
    inputs:
      layer:
        description: "指定要跑的層（all / plan-doc / code / realtime / deploy）"
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
  # ─────────────────────────────────────────────
  # 副駕駛：路由與摘要（永遠先跑）
  # ─────────────────────────────────────────────
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

      - name: 印出 3+1 AI 分工
        run: |
          {
            echo "## 🧭 3+1 AI 路由"
            echo ""
            echo "| 層 | 負責 | 本輪任務 |"
            echo "|----|------|----------|"
            echo "| plan-doc | ChatGPT | Markdown／Mermaid 檢查 |"
            echo "| code | DeepSeek | Python 編譯／測試／煙霧測試 |"
            echo "| realtime | Grok | 外部資訊（需 secret，可跳過） |"
            echo "| deploy | GitHub | Pages 部署 |"
            echo "| human-gate | 副駕駛 | 合併前人工確認 |"
            echo ""
            echo "本輪 target：**${{ steps.pick.outputs.target }}**"
          } >> "$GITHUB_STEP_SUMMARY"

  # ─────────────────────────────────────────────
  # ChatGPT 層：plan-doc
  # ─────────────────────────────────────────────
  plan-doc:
    name: "ChatGPT・plan-doc"
    needs: route
    runs-on: ubuntu-latest
    if: |
      needs.route.outputs.target == 'all' ||
      needs.route.outputs.target == 'plan-doc'
    steps:
      - uses: actions/checkout@v4

      - name: 檢查 Markdown 檔案
        run: |
          fail=0
          while IFS= read -r f; do
            echo "→ 檢查 $f"
            # 檢查未閉合的程式碼圍欄
            fences=$(grep -c '^```' "$f" || true)
            if [ $((fences % 2)) -ne 0 ]; then
              echo "::error file=$f::程式碼圍欄未閉合（共 $fences 個）"
              fail=1
            fi
            # 檢查空連結
            if grep -nE '\]\(\s*\)' "$f"; then
              echo "::error file=$f::發現空連結"
              fail=1
            fi
          done < <(find . -name '*.md' -not -path './node_modules/*')
          exit $fail

      - name: 檢查 Mermaid 區塊
        run: |
          found=$(grep -rl '```mermaid' --include='*.md' . | wc -l)
          echo "含 Mermaid 的 Markdown 檔：$found 個"
          if [ "$found" -eq 0 ]; then
            echo "::warning::README 尚無 Mermaid 圖表"
          fi

  # ─────────────────────────────────────────────
  # DeepSeek 層：code
  # ─────────────────────────────────────────────
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

      - name: 安裝依賴（若存在）
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: 編譯所有 Python
        run: python -m compileall -q .

      - name: 風力功率煙霧測試
        run: |
          if [ -f saiyan/wind_power.py ]; then
            python - <<'PY'
          import importlib.util, pathlib
          p = pathlib.Path("saiyan/wind_power.py")
          spec = importlib.util.spec_from_file_location("wp", p)
          m = importlib.util.module_from_spec(spec); spec.loader.exec_module(m)
          r = m.wind_power_kw(12, blade_angle_deg=35)
          assert r["power_kw"] >= 0, r
          print("wind_power_kw(12, 35) =", r["power_kw"], "kW")
          PY
          else
            echo "::notice::找不到 saiyan/wind_power.py，跳過煙霧測試"
          fi

      - name: 執行測試（若存在）
        run: |
          if [ -d tests ]; then
            python -m pytest -q || exit 1
          else
            echo "::notice::尚無 tests/ 目錄，跳過 pytest"
          fi

  # ─────────────────────────────────────────────
  # Grok 層：realtime（需 secret，否則跳過）
  # ─────────────────────────────────────────────
  realtime:
    name: "Grok・realtime"
    needs: route
    runs-on: ubuntu-latest
    if: |
      needs.route.outputs.target == 'all' ||
      needs.route.outputs.target == 'realtime'
    steps:
      - name: 檢查 secret 是否設定
        id: check
        env:
          GROK_KEY: ${{ secrets.GROK_KEY }}
        run: |
          if [ -n "$GROK_KEY" ]; then
            echo "present=true" >> "$GITHUB_OUTPUT"
          else
            echo "present=false" >> "$GITHUB_OUTPUT"
            echo "::notice::未設定 GROK_KEY，realtime 層跳過（adapter stub）"
          fi

      - name: 即時資訊層（stub）
        if: steps.check.outputs.present == 'true'
        run: |
          echo "Grok adapter 尚未實作，僅驗證 secret 可用。"
          echo "實作位置：copilot/grok.py"

  # ─────────────────────────────────────────────
  # 副駕駛：人工確認 gate（需設定 environment: production）
  # ─────────────────────────────────────────────
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
          echo "通過後才允許進入 Pages 部署。" >> "$GITHUB_STEP_SUMMARY"

  # ─────────────────────────────────────────────
  # GitHub 層：deploy
  # ─────────────────────────────────────────────
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

這條流水線的行為

觸發 行為
push 到 main 跑全部層，最後進 human-gate → deploy
PR 跑 plan-doc ＋ code，跳過 deploy
手動 workflow_dispatch 選 code 只跑 DeepSeek 層
沒設 GROK_KEY realtime 層自動跳過，不失敗
human-gate 需在 repo 建立 production environment 並開啟 Required reviewers

---

6️⃣ copilot/ adapter stub

copilot/base.py

```python
"""3AI adapter 的共同介面。

本模組僅定義「分層呼叫介面」，不代表任何外部模型授權或商業關係。
實際啟用哪個 adapter 由副駕駛（人類）決定。
"""

from __future__ import annotations

import os
from dataclasses import dataclass


class NotConfiguredError(RuntimeError):
    """缺少 API key 或環境設定。"""


@dataclass
class AgentReply:
    layer: str
    agent: str
    content: str
    stub: bool = True


class BaseAgent:
    layer: str = "base"
    agent_name: str = "base"
    env_key: str = ""

    def __init__(self) -> None:
        self._key = os.getenv(self.env_key, "").strip()

    @property
    def configured(self) -> bool:
        return bool(self._key)

    def ask(self, prompt: str) -> AgentReply:
        """回傳回覆。目前為 stub，未實作外部呼叫。"""
        if not self.configured:
            raise NotConfiguredError(
                f"{self.agent_name}: 缺少環境變數 {self.env_key}"
            )
        return AgentReply(
            layer=self.layer,
            agent=self.agent_name,
            content=f"[stub] {self.agent_name} 尚未實作外部 API 呼叫。prompt={prompt!r}",
        )

    def describe(self) -> dict[str, object]:
        return {
            "layer": self.layer,
            "agent": self.agent_name,
            "env_key": self.env_key,
            "configured": self.configured,
            "status": "stub",
        }
```

copilot/chatgpt.py

```python
"""ChatGPT adapter（plan-doc 層）。

用途：需求拆解、文件撰寫、內容整理、副駕駛協作。
狀態：stub，未實作外部 API 呼叫。
"""

from __future__ import annotations

from .base import BaseAgent


class ChatGPTAgent(BaseAgent):
    layer = "plan-doc"
    agent_name = "ChatGPT"
    env_key = "OPENAI_API_KEY"
```

copilot/grok.py

```python
"""Grok adapter（realtime 層）。

用途：即時資訊、事實核對、外部視角。
狀態：stub，未實作外部 API 呼叫。
"""

from __future__ import annotations

from .base import BaseAgent


class GrokAgent(BaseAgent):
    layer = "realtime"
    agent_name = "Grok"
    env_key = "GROK_API_KEY"
```

copilot/deepseek.py

```python
"""DeepSeek adapter（code 層）。

用途：程式實作、數值計算、Termux 整合。
狀態：stub，未實作外部 API 呼叫。
"""

from __future__ import annotations

from .base import BaseAgent


class DeepSeekAgent(BaseAgent):
    layer = "code"
    agent_name = "DeepSeek"
    env_key = "DEEPSEEK_API_KEY"
```

copilot/__init__.py

```python
"""3AI adapter 匯出。

注意：所有 adapter 目前皆為 stub，僅提供分層介面。
"""

from .base import BaseAgent, AgentReply, NotConfiguredError
from .chatgpt import ChatGPTAgent
from .grok import GrokAgent
from .deepseek import DeepSeekAgent

AGENTS = {
    "plan-doc": ChatGPTAgent,
    "realtime": GrokAgent,
    "code": DeepSeekAgent,
}


def get_agent(layer: str) -> BaseAgent:
    """依層級取得 adapter 實例；未知層回傳 NotImplementedError。"""
    cls = AGENTS.get(layer)
    if cls is None:
        raise NotImplementedError(f"未支援的層：{layer}")
    return cls()


__all__ = [
    "BaseAgent", "AgentReply", "NotConfiguredError",
    "ChatGPTAgent", "GrokAgent", "DeepSeekAgent",
    "AGENTS", "get_agent",
]
```

可選：copilot/selftest.py

```python
"""快速驗證 adapter 分層是否正確。"""

from . import get_agent

for layer in ("plan-doc", "realtime", "code"):
    a = get_agent(layer)
    print(a.describe())
```

執行：python -m copilot.selftest

---

📋 交付總覽

# 檔案 用途
1 data/dragonball.json 七龍珠 Z 七大篇章＋角色
2 data/wuxia.json 金庸十五部＋三部曲＋門派＋張三丰
3 data/ai_arch.json 3+1 AI 分工與 workflow 路徑
4 .github/workflows/pages.yml Pages 自動部署
5 .github/workflows/ai-pipeline.yml 3+1 AI 流水線（route → plan-doc／code／realtime → human-gate → deploy）
6 copilot/base.py adapter 介面
7 copilot/chatgpt.py／grok.py／deepseek.py 三個 stub
8 copilot/__init__.py 匯出與 get_agent()
9 copilot/selftest.py 本機自測

---

⚠️ 啟用前必做兩件事

1. Settings → Pages → Source 選「GitHub Actions」（不是 branch）
2. Settings → Environments → 新建 production，勾選 Required reviewers，加入你自己 → human-gate 才會真的擋在部署前

Secret 對照表（可選，沒有就自動跳過）

Secret 對應 adapter
OPENAI_API_KEY copilot/chatgpt.py
GROK_API_KEY copilot/grok.py
DEEPSEEK_API_KEY copilot/deepseek.py

---

下一步選配：

1. 我幫你把 index.html 改成從 data/*.json fetch 渲染（資料與前端分離）
2. 我幫你補 tests/test_sagas.py、tests/test_wind_power.py，讓 code 層真的跑 pytest
3. 我幫你把 saiyan/sagas.py、saiyan/qi_power.py 對齊 data/dragonball.json

