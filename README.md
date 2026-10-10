# 流放继承人 / Exile Heir to a March

给 **Europa Universalis IV 1.37** 加两个决议：把看不顺眼的继承人打发到属国去，让他去那边当君主。

> 灵感与数据结构参考了 EU4 原版外交行动 `promote_appanage_heir`（把属国继承人过继给宗主国）——本 mod 是反过来的方向。

---

## 功能

| 决议 | 目标 | 出现条件 | 消耗 |
|---|---|---|---|
| **Exile Heir to a March** | 一个卫戍国（任何 `is_march = yes` 的属国类型，含自定义类型） | 你有继承人，且存在卫戍国 | 50 威望 |
| **Exile Heir to a Vassal** | 一个普通附庸国 | 你有继承人，且存在附庸国 | 50 威望 |

执行效果：

1. 你的**继承人被移除**，保存为流放者
2. 随机选定一个目标属国：若其已有继承人则先废黜
3. 你的继承人成为**该属国的新君主**（原君主死亡）
4. 属国 **+1 稳定度**（抵消君主死亡带来的稳定度损失）
5. 你 **-50 威望**

AI **永远不会**执行这两个决议（`ai_will_do = { factor = 0 }`）。

---

## 目的地选择规则

```
① 跳过与我国同王朝的属国        （dynasty = ROOT）
② 在剩下的里轮换                （接收过的属国本轮跳过）
③ 一轮轮完后清空标记，重新开始
④ 若所有属国都与我国同王朝 → 任选其一
```

每一层的随机选择之前，都会先用 `any_subject_country` 确认"这类属国确实存在"，
所以**不会出现"继承人已经从宫廷移除、却没有国家接收"的情况**。

> 为什么不直接随机？—— EU4 的 `random_subject_country` 在同一个游戏日内随机种子是固定的，
> 暂停状态下连点决议会一直选中同一个国家。轮换是唯一可靠的办法。

---

## 安装

1. 下载本仓库（或 `git clone`）
2. 把 **`exile_heir_mod.mod`** 和 **`exile_heir_mod/`** 文件夹一起放进：
   ```
   Documents\Paradox Interactive\Europa Universalis IV\mod\
   ```
3. 启动游戏 → 启动器的**播放集**里勾选「流放继承人」

> 新增 mod 默认不会自动启用，必须手动加入播放集，否则游戏不会加载它。

---

## 兼容性

- 游戏版本：**1.37.\***（实测 1.37.5.0）
- 与自定义属国类型兼容：任何 `is_march = yes` 的类型都能被识别
  （例如「简易中华都护府」mod 的 `cnpro_duhufu`，它 `copy_from = march` 因而继承了 `is_march = yes`）
- 只新增决议与 scripted effects，不改动任何原版文件，**存档安全**

---

## 已知问题：中文显示

本 mod **只提供英文本地化**。在中文汉化 mod 环境下，自定义文本无法显示中文，原因如下（已实测）：

| 文件写法 | 游戏内结果 |
|---|---|
| UTF-8 **with BOM** + 中文 | 文件被加载，但中文渲染为 `????????` |
| UTF-8 **无 BOM** | **文件不被加载** → 显示 key 原名 |
| GBK 无 BOM | **文件不被加载** → 同上 |

汉化字体（`gfx/fonts/zh-hans-*.fnt`，BMFont 位图，face 名为 `方正屏显雅宋_GBK`）
的字形表**按标准 Unicode 码位编号**（如 `char id=22825` = U+5929「天」），
但汉化 mod 自己的文本采用的是**专有 3 码位编码**（`[0x10|0x11|0x12][低字节][高字节]`），
标准 UTF-8 中文无法命中字形，因此显示为问号。

**欢迎有汉化经验的贡献者接手这个问题**——详细分析（含字节样本、已知错例、素材路径）见
[`docs/中文显示问题.md`](docs/中文显示问题.md)。

---

## 文件结构

```
exile_heir_mod.mod                                mod 描述符（放在 mod/ 根目录）
exile_heir_mod/
├── descriptor.mod                                目录内描述符（供创意工坊发布用）
├── decisions/
│   └── exile_heir_decisions.txt                  两个决议的定义与选择逻辑
├── common/scripted_effects/
│   └── exile_heir_effects.txt                    转移继承人的 scripted effects
└── localisation/
    └── exile_heir_l_english.yml                  英文本地化（UTF-8 with BOM）
```

---

## 开发笔记

- `exile_heir_as` / `set_heir` 是 EU4 **1.12** 就加入的效果，可以跨国家传递继承人
  （原版 `common/new_diplomatic_actions/00_diplomatic_actions.txt` 的 `promote_appanage_heir` 就是这么用的）
- 1.37 **没有** `is_monarchy` / `is_republic` 触发器，判断政体只能用 `government = <组>`
- 属国作用域必须写 `any_subject_country` / `random_subject_country`，**不能**写 `any_subject` / `random_subject`
- localisation 文件**必须带 UTF-8 BOM**，否则游戏不加载
- 决议定义的改动需要**重启游戏**；只改文本可以用控制台 `reloadloc` 热重载

---

## License

MIT
