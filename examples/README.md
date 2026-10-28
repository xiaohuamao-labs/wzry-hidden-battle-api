# 示例

> ⚠ 下面示例里的域名是**占位符**（`api.xiaohuamao.example`），换成正式地址再跑。

## curl

```bash
# 查一局的 10 人详情（含同队/对手、脱敏标记）
curl -s "https://api.xiaohuamao.example/api/battle?target=营地号&limit=1" \
  -H "Authorization: Bearer 你的卡密" | jq .
```

```bash
# 查固排统计（近 30 局窗口）
curl -s "https://api.xiaohuamao.example/api/mates?target=营地号&games=30" \
  -H "Authorization: Bearer 你的卡密" | jq '.mates[] | {name, mate, foe}'
```

## Python

```python
"""识别未授权号并统计固排 —— 最小可用示例。"""
import requests

BASE = "https://api.xiaohuamao.example"
KEY = "你的卡密"


def fetch_mates(target: str, games: int = 30) -> list[dict]:
    r = requests.get(
        f"{BASE}/api/mates",
        params={"target": target, "games": games},
        headers={"Authorization": f"Bearer {KEY}"},
        timeout=20,
    )
    r.raise_for_status()
    return r.json()["mates"]


for m in fetch_mates("营地号"):
    # ⚠ 软证据必须标出来，别把推断当事实卖
    mark = "☆疑似" if m["certainty"] == "low" else "  "
    if m["mate"] >= 3:
        print(f"{mark} {m['name']:<16} 同队{m['mate']:>3}局 "
              f"对手{m['foe']:>3}局  最近 {m['last']}")
```

## Python · 判断脱敏行的正确姿势

```python
def is_anonymized(player: dict) -> bool:
    """⚠ 只看 roleName。用 `not player_id` 判断会误伤。
    实测存在脱敏但 player_id 仍残留的行。"""
    return player.get("roleName") == "未授权游戏信息"


def is_flag_on(player: dict, key: str) -> bool:
    """⚠ 战报里有一批标记字段是字符串 '0'/'1'，不是布尔。
    '0' 是 truthy ⇒ 直接 `if p.get(key)` 会让条件永远成立。"""
    return str(player.get(key)) == "1"
```

## Node.js

```javascript
const BASE = "https://api.xiaohuamao.example";
const KEY = "你的卡密";

const r = await fetch(`${BASE}/api/battle?target=${encodeURIComponent("营地号")}`, {
  headers: { Authorization: `Bearer ${KEY}` },
});
const data = await r.json();

// 按阵营分组，脱敏行单独标出来
const blue = data.sides.filter((p) => p.camp === "blue");
const red = data.sides.filter((p) => p.camp === "red");

for (const p of [...blue, ...red]) {
  const name = p.anonymized ? "未授权游戏信息" : p.role_name;
  const doubt = p.certainty === "low" ? " ☆" : "";
  console.log(`[${p.camp}] ${name} ${p.hero_name} ${p.total_win}胜${doubt}`);
}
```

## MCP（让 AI 直接查）

```json
{
  "mcpServers": {
    "wzry-battle": {
      "url": "https://api.xiaohuamao.example/mcp",
      "headers": { "Authorization": "Bearer 你的卡密" }
    }
  }
}
```
