# Delta Force 战术伤害计算器

三角洲行动（Delta Force）的弹药伤害计算工具，单页 HTML 应用。

## 技术栈

- 纯前端单 HTML 文件（HTML + CSS + JS，无框架）
- 调用 **Claude API**（`claude-sonnet-4-6`）做截图识别（OCR 提取游戏参数）
- 无后端、无构建步骤，浏览器直接打开即可

## 核心功能

### 伤害模拟引擎
- 输入弹药属性（肉体伤害、护甲伤害、穿透等级、射速 RPM）
- 支持最多 10 发射击序列，每发可指定命中部位（头/胸/腹/手臂/腿）
- 部位系数：头部 1.9x、胸/腹 1.0x、手臂/腿 0.5x
- **独立护甲池**：身体护甲和头盔（固定 4 级 35 耐久）分开计算
- 护甲穿透机制：按弹级 vs 甲级差值计算肉伤系数（高两级=100%、高一级=75%、同级=50%、低级=0%）
- 护甲伤害衰减表 `DECAY_TABLE[穿透等级][甲级]`

### 距离衰减系统
- 支持近/中/远三段距离衰减
- 两个折点 `dist_threshold`（近→中）和 `dist_threshold2`（中→远）
- 各段独立衰减系数

### 截图识别（Claude Vision API）
- 粘贴/拖入游戏截图自动识别参数
- 支持三类截图：子弹属性页、距离衰减曲线图、武器属性页
- 识别结果自动填入计算器所有字段
- **注意**：API key 需要通过 `anthropic-dangerous-direct-browser-access` header 在浏览器直接调用

### 武器预设系统
- 识别后自动保存预设（localStorage via `window.storage`）
- 支持内置预设 `BUILTIN_PRESETS`（代码内写入）和用户预设（localStorage 优先）
- 导出/导入 JSON

### 护甲配置
- 支持 3/4/5/6 级甲切换
- 各级甲有预设耐久值（如 5 级甲：95/105/115/125）
- 可多选耐久值同时对比

## 关键变量和常量

```javascript
FLESH          // 当前弹药肉体伤害
ARMOR_DMG      // 当前弹药护甲伤害
RPM            // 射速（发/分）
PARTS          // 部位系数映射
DECAY_TABLE    // 护甲伤害衰减表 [穿透等级][甲级]
ARMOR_TIERS    // 各级甲预设耐久值
HEAD_ARMOR     // 头盔配置 {tier: 4, dur: 35}
armorDecayMap  // 截图识别覆盖的衰减系数（优先级高于 DECAY_TABLE）
BUILTIN_PRESETS // 内置武器预设（空对象，由更新代码时写入）
```

## 输出

- **击杀弹量表**：各距离 × 各护甲耐久的击杀弹数 + TTK
- **逐发详细表**：每发的肉伤、累计伤害、护甲剩余
- **击杀总结卡片**：近/远距分别的击杀弹数

## 文件

```
delta-force-calc-data/
├── CLAUDE.md                  # 本文件
├── README.md                  # 简介
└── tactical_damage_calc.html  # 计算器主体（~2047行，单文件应用）
```

## 开发注意事项

- 修改 CSS 变量可切换主题（`--bg`, `--panel`, `--accent` 等）
- `BUILTIN_PRESETS` 对象目前为空，可在代码中直接写入预置武器数据
- `simulate()` 是核心模拟函数，接收 `(armorDur, distDecay)` 返回完整逐发结果
- `SYSTEM_PROMPT` 包含截图识别的详细指令，修改识别逻辑时需同步更新
