---
name: food-search
description: "Agent美食搜索。用户说想吃XXX→秒出全城店铺对比(店名/评分/人均/地址/电话/推荐菜)。Trigger: 想吃、找餐厅、有什么好吃的、附近美食、美食推荐、螺蛳粉、烤鱼、火锅、鱼杂煲、奶茶等。基于高德POI API+搜索补漏。"
agent_created: true
version: "2.0.0"
---

## 准备工作

**需要高德API Key（免费，5000次/天）：**
1. https://lbs.amap.com → 注册 → 控制台 → 创建应用 → Web服务 
2. 设置环境变量：`export AMAP_KEY=你的key`

# Food Search Skill v2 — 美食雷达

## 一句话

用户说"想吃XXX"，3秒内返回全城店铺排名表（高德API）+ 补搜评价/团购。

## 你的行为准则

**做**：
- 1次高德API调用搞定基础数据（位置/评分/人均/电话）
- 店铺<5家时自动扩城到周边
- 用布尔逻辑补搜：`"{店名}" 推荐菜 OR 团购 OR 测评`
- 搜不到菜单就诚实说，不编造

**不做**：
- 不编造价格、菜单、评分
- 不同时打开浏览器（纯API+搜索就够了）
- 不对"搜了也没用"的店铺追着补搜（高德评分<3.5的跳过）

## 执行流程

### S1: 解析 → 查高德

```bash
curl -s "https://restapi.amap.com/v3/place/text?keywords={URLENCODED}&city={CODE}&key=$AMAP_KEY&extensions=all&types=050000&offset=15" \
  | python -X utf8 -c "
import json,sys,os
d=json.load(sys.stdin)
code=os.environ.get('EXIT_CODE','0')
if d.get('status')!='1' or not d.get('pois'):
  print('NO_RESULTS:'+str(d.get('count',0)))
  sys.exit(1)
for i,p in enumerate(d['pois'][:12]):
    b=p.get('biz_ext',{})
    t=p.get('tel','-')
    if isinstance(t,list): t=t[0] if t else '-'
    r=b.get('rating','?'); c=b.get('cost','?')
    print(f'ROW|{p[\"name\"]}|{r}|{c}|{p[\"address\"][:50]}|{t}')
"
```

### S2: 失败回退链

```
NO_RESULTS → 去types限制再搜
仍无结果 → 拆词分别搜（"螺蛳粉烤鱼"→"螺蛳粉"+"烤鱼"）
仍无结果 → 扩大城市(440100广州/440600佛山)
仍无结果 → 诚实告知+建议美团
```

### S3: 补搜评价（并行，Top 5）

对评分最高的5家店，用 WebSearch 补搜：
```
"{店名} {城市} 推荐菜 OR 团购 OR 测评"
```

提取模式：
- 价格：`¥\d+` `\d+元` `团购.*\d+\.?\d*元`
- 推荐菜：出现≥2次的菜名（排除"好吃""推荐"等通用词）
- 没找到就标注 `菜单未索引`

### S4: 输出

```markdown
## 🍜 "{食物}" — {城市}

| # | 店名 | ⭐ | 人均 | 推荐菜/团购 | 地址 |
|---|------|------|------|------------|------|
| 1 | ... | 4.5 | ¥18 | 炸蛋螺蛳粉·团购¥7.7 | ... |

> 💡 评分>4.0优先 | 菜单详情→美团App | 数据源：高德+搜索补漏
```

---

## 城市编码

442000=中山 | 440100=广州 | 440300=深圳 | 440400=珠海 | 440600=佛山

## 边界规则

- API status≠1 → 立即报错，不重试
- count=0 → 触发S2回退
- 编码乱码 → 加 `-X utf8`
- 电话为空 → 显示"无"
- 人均为空 → 显示"-"

## 不做的事

- 不打开 agent-browser（太慢+被盾）
- 不为评分<3.5的店补搜评价
- 不编造任何数据
