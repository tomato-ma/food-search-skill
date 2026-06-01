# Food Search Skill — 美食雷达 🍜

Agent 美食搜索利器。用户说"想吃XXX"→ 3秒返回全城店铺排名表。

## 功能

- 高德POI API 实时搜索（免费，无需登录）
- 返回：店名 | 评分 | 人均 | 地址 | 电话 | 推荐菜
- 智能回退：无结果时自动拆词+扩城
- 补搜团购价格和评价

## 安装

### 一行安装

```bash
git clone https://github.com/tomato-ma/food-search-skill.git ~/.workbuddy/skills/food-search
```

### 手动安装

1. 下载 `SKILL.md`
2. 放到 `~/.workbuddy/skills/food-search/SKILL.md`
3. [注册高德API Key](https://lbs.amap.com)（免费5000次/天）→ 控制台 → Web服务
4. 在本 Skill 的 SKILL.md 第32行，把 `key=db21462a3b0b3a2f56fc3fce061d7823` 替换成你的 Key

> ⚠️ 或者用环境变量：`export AMAP_KEY=你的key`，然后把文件里的 key 改成 `$AMAP_KEY`

## 使用

直接在对话中说：
- "想吃螺蛳粉"
- "深圳 椰子鸡"
- "附近有什么好吃的火锅"

## 示例输出

| # | 店名 | ⭐ | 人均 | 推荐菜 | 地址 |
|---|------|------|------|--------|------|
| 1 | 潮牛匠潮汕牛肉火锅 | 4.6 | ¥65 | 鲜切牛肉 | 火炬开发区 |

## 限制

- 覆盖90%店铺（新店需1-3月同步）
- 完整菜单需打开美团App查看
- 每人需自备高德API Key

## 作者

tomato-ma - 反美团垄断第一线
