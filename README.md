---
name: formula-mixcut-animation
description: 公式混剪爆款 Manim 定稿技能 — 壁纸幽灵层、金橙渐变、快速形变转场（transform_mismatches 灵魂参数）、1秒/卡节奏、公式+动态彩色函数图双层卡片（复变函数 46 连发验证）、资料搜集路径。Use when making fast-paced formula showcase/mixcut videos.
metadata:
  type: reference
  tags: [manim, formula, mixcut, transform, fast-paced, wallpaper, plot-layer]
---

# Manim 公式混剪：代码即特效

这是一个用于制作高节奏公式混剪视频的 Manim Skill。它用代码直接实现公式到公式之间的炫酷形变转场、金橙渐变视觉与动态彩色函数图层，无需进入剪辑软件做进一步后期，也能得到连贯、有冲击力的“混剪”效果。

## 案例效果

仓库中的 [`formula-mixcut-demo.mov`](./formula-mixcut-demo.mov) 是约一分钟的案例视频，展示了公式卡片、动态绘图和连续形变转场的最终效果。

# 公式混剪 Manim Skill（成品定稿版）

以成品《概率统计考试神公式》（22 卡+2 绘图页/29.3s）逆向定稿；
由《复变函数邪修进阶之路》（46 卡+14 动态图/50.6s）升级出**公式+绘图双层卡片**架构。
节奏基准来自爆款样本实测：**14 卡 / 14.4s ≈ 1.03s/卡**。慢速讲解版节奏已废弃。

---

## 1. 全局配置

```python
from manim import *
import numpy as np

config.pixel_width  = 1920
config.pixel_height = 1080
config.frame_width  = 16.0
config.frame_height = 9.0
# 背景近黑 #050505（壁纸层垫底，见 §3）
```

## 2. 节奏（铁律中的铁律）

```python
T_RUN  = 0.45   # 形变时长
T_HOLD = 0.55   # 停留时长
# 卡片循环 = 1.0s，与样本 1.03s/卡一致
```

- 开场 ≤ 2s（大标题一闪 + 一句副标题，直接融入第一张卡）
- 结尾 ≤ 2s（含 0.4s 黑场淡出）
- **总时长公式：卡数 × 1.0s + 绘图页数 × 2.5s + 4s**

## 3. 画面三层结构 + 壁纸层

每张卡只有三层，**同屏永远只允许一张卡**（护眼核心）：

| 层 | 内容 | 规格 |
|----|------|------|
| 标题 | 固定前缀 + 主题名 | 顶部居中 `to_edge(UP, buff=0.45)`，楷体 60 加粗；前缀黄绿 `#CDDC39`，主题名金橙渐变 `#FFD166→#F59E62` |
| 公式 | 单条 LaTeX | 居中 `UP*0.55`，`MathTex(font_size=64, stroke_width=1.2)`，橙金渐变 `#FF8A50→#FFD166`，超宽则 `scale_to_fit_width(14.4)` |
| 副标题 | 两行短句 | `DOWN*2.35`，第一行绿 `#66D19E` 34 加粗，第二行青 `#4DB6AC` 26 |

**壁纸幽灵层**（最先 add，全程静止）：
- 25~30 条灰色 `MathTex`，字号 34~48 随机，`set_opacity(0.10)`，随机散布 `[-7.1,7.1]×[-4,4]`、微旋转 ±0.12rad
- 用固定随机种子保证可复现；内容取卡片公式的"变体"（积分、极限、分布记号等）
- **opacity ≤ 0.12、绝不参与形变、绝不移动**

## 4. 转场（灵魂，实测踩坑结论）

```python
def morph_to(self, new_t, new_f, new_s, run_time=T_RUN):
    self.play(
        ReplacementTransform(self.cur_t, new_t),                  # 中文标题整族字形拉伸
        TransformMatchingTex(self.cur_f, new_f,
                             transform_mismatches=True),          # ★灵魂参数
        ReplacementTransform(self.cur_s, new_s),                  # 副标题同法
        run_time=run_time,
    )
```

- ★ `transform_mismatches=True` **必须加**：默认行为会把对不上的碎片淡出淡入，观感是"一条公式弹出另一条"；加上后所有碎片互相形变——旧字形拉伸、旋转、飞向新位置，新旧公式中途共存叠影，与爆款样本逐帧一致
- 中文 `Text` 用 `ReplacementTransform` 整族形变（字符数不同自动复制拉伸，产生拉丝感）；`TransformMatchingShapes` 会淡掉不匹配字形，效果偏温和，弃用
- 上一卡的 mob 直接作为 transform 源，**绝不 FadeOut 再 Write**
- 标题、公式、副标题三路**同一拍**形变，节奏才连得上
- 首卡公式用 `FadeIn(shift=DOWN*0.3)` 进场（与标题形变同拍）；末卡淡出收黑

## 5. 数据驱动（加卡 = 加一行）

```python
# 双层版：五元组 (标题, 公式, 副标题1(境界·标签), 副标题2, 绘图键)
CARDS = [
    ("欧拉公式", r"e^{i\theta}=\cos\theta+i\sin\theta",
     "炼气·天人合一", "复指数就是旋转", "euler"),   # ← 有图卡
    ("三角不等式", r"|z_1+z_2|\le|z_1|+|z_2|",
     "炼气·三角不等式", "模的次可加性", None),      # ← 纯公式卡
]
PLOT_BUILDERS = {"euler": SceneClass.p_euler, ...}  # 键 → 构图方法
```

- LaTeX 禁止 `\text{中文}`（中文一律 Text）；卡片可带双式，超宽靠 fit 缩放
- 数学符号（λ、σ、²、±、→）实测楷体可直出；但 √ 这类必须 MathTex，别放 Text（只显示半截）

## 6. 绘图层：双层卡片架构（复变函数 46 连发验证，本次升级）

**新架构**：绘图不再是独立"绘图页"，而是成为**任何一张公式卡的第四层**。
每卡五元组多一个 `绘图键`：有键则该卡 = 标题 + 公式(上移) + **公式下方动态彩色函数图** + 单行副标题；无键则回归标准三层卡。

```python
PC = DOWN * 1.75   # 函数图像面板中心（公式上移到 UP*1.05，图占下半屏）

def go_card(self, idx):
    name, latex, s1, s2, key = CARDS[idx]
    has_plot = key is not None
    new_t = self.make_title(name)
    new_f = self.make_formula(latex, pos=UP * 1.05 if has_plot else UP * 0.55)
    new_s = self.make_sub1(s1) if has_plot else self.make_subs(s1, s2)
    new_plot = PLOT_BUILDERS[key](self) if has_plot else None
    self.morph_to(new_t, new_f, new_s, new_plot)   # 图随形变拍同进同出
    self.wait(T_HOLD)

def morph_to(self, new_t, new_f, new_s, new_plot=None, run_time=T_RUN):
    anims = [ReplacementTransform(self.cur_t, new_t),
             TransformMatchingTex(self.cur_f, new_f, transform_mismatches=True),
             ReplacementTransform(self.cur_s, new_s)]
    if self.cur_plot is not None:            # 旧图同拍散场（不单独占时）
        anims.append(FadeOut(self.cur_plot))
    if new_plot is not None:
        anims.append(FadeIn(new_plot, lag_ratio=0.06))   # lag 产生"生长感"
    ...
```

### 6.1 网格面板（每张图的底座，颜色不单一的秘诀）

```python
def make_plane(self):
    return NumberPlane(
        x_range=[-5.4, 5.4, 1], y_range=[-1.45, 1.45, 1],
        background_line_style={"stroke_color": "#1F3A57", "stroke_width": 1.2},
        axis_config={"stroke_color": "#4A7DB5", "stroke_width": 2.5},
        faded_line_ratio=0).move_to(PC)
```

### 6.2 面板坐标绘图辅助（横轴过面板中心的函数曲线）

```python
def axes_plot(fn, x0, x1, color, dash=False, sw=3.5):
    pl = ParametricFunction(lambda t: PC + np.array([t, fn(t), 0.0]),
                            t_range=[x0, x1], color=color, stroke_width=sw)
    return DashedVMobject(pl, num_dashes=60) if dash else pl
```

### 6.3 十四张图的方向库（复变函数成品实测）

| 图型 | 做法 | 彩色要点 |
|------|------|----------|
| 单位根 | 圆内接正 n 边形 | 顶点六色彩虹点 |
| e^z 网格 | 同心圆×放射线正交网 | 蓝圆+红射线（CR 正交直观） |
| 调和函数 | 数值上色的方格热力图 | `interpolate_color(RED,YELLOW,BLUE)` |
| 多值函数 | 支割线+多色螺线 | 分支换色（蓝绿红黄） |
| 积分围道 | 圆/摇摆曲线 + 奇点红叉 | 黄色围道+绿色公式角标 |
| 级数逼近 | 部分和逐阶叠画 | 每阶一色（蓝→绿→橙） |
| 洛朗圆环 | 大圆套小圆+中心红叉 | 环内绿色填充 |
| 保角映射 | 左源域彩格 → 右像域曲线网 | 网格线调色盘轮换 |
| 方波分解 | 方波+1/3/5/7 阶叠加曲线 | 阶数递进配色 |
| 变换对 | 左右双面板+中间箭头 | 时域红/频域蓝+黄 F |
| 色散关系 | 洛伦兹峰+色散尾翼 | Re 蓝 Im 红 |

### 6.4 绘图层铁律

- **图必须在面板内**：y 幅度 ≤1.4（面板半高 1.45），参数曲线的 t 范围按 max|y| 收紧
- 图内标注放**面板空白角**（如 [±4.5, ±1.0]），严禁与公式/副标题碰撞
- 每图 ≤ 25 个 mobject，FadeIn(lag_ratio=0.06) 控制在 0.45s 内完成
- 奇点画红叉（两条交叉短线）是统一视觉语言
- `set_stroke(dashes=...)` **不存在**——虚线用 `DashedVMobject(mob, num_dashes=)`
- 热力图标注放面板**下方**（PC+[0,-1.68]），别放上方（会撞公式）

## 7. 制作流水线（检查清单）

1. 写 `CARDS` 数据 → 组装代码（§5 模板）
2. `python3 -c "compile(...)"` 语法检查；正则查 `\text{中文}` 必须为 0
3. `-ql` 低清渲染（带缓存）→ ffprobe 核对时长 ≈ 卡数×1.0 + 4
4. ffmpeg 抽帧：**停留期帧**查排版/遮挡/越界，**形变中点帧**查转场炫度
5. 修完 → `-pqh` 1080p60 高清 → 直接拷到 `~/Desktop/`（用户自行归档）
6. **中间产物默认保留**：`media/`（partial_movie_files、texts、旧成片）与 `__pycache__`
   不删——缓存让下次改片只渲染新增/修改的部分；用户明确说"清理"才清理

## 8. 资料搜集路径（公式清单怎么找，供 AI/人类复用）

**五步法**（复变函数 46 连发实测流程）：

1. **自备草案先行**：先按主流教材大纲手写一份 30~45 条的候选清单（别等搜索）。
   例如复变函数按"复数→解析→积分→级数→留数→定理→变换"章节轴铺开，
   概率论按"概率→分布→数字特征→大数定律→抽样分布"铺开。章节轴保证"从低级到高级"的叙事感。
2. **WebSearch 交叉核对**：一次搜 5~8 个关键词（中英混着搜），
   例：`复变函数 核心公式 汇总 柯西黎曼 留数定理 儒歇定理`。
   用搜索结果**校正草案**：证实记忆 + 补漏。复变函数这次就补出了
   柯西积分公式、高阶导数公式、m 阶极点留数三张卡。
3. **优先信源**（可信度排序）：
   - 知乎专栏专题帖（zhuanlan.zhihu.com，搜"XX 公式汇总/核心知识"）
   - CSDN 期末复习向总结（搜"XX 期末 公式 总结"）
   - 维基百科对应词条（定理标准表述）
   - 高校课程页（faculty.*.edu.cn，系统大纲）
   - 贴吧/洛谷等社区合集（捡漏用）
4. **必须逐条数值验证**（数学准确性的最后一道闸）：写 numpy 脚本对每条
   拿得出数值的结论做数值验证——两个具体数字代入、看等式是否成立；
   记忆模糊的公式（尤其定点坐标、系数）**先验证再上屏**。
   实测案例：圆锥曲线定点结论靠数值验证纠正了两处"想当然"。
5. **卡片文案包装**：每条配"境界/标签式"副标题（如"金丹·一点决定全域"），
   8~15 字内，为爆款节奏服务。主题化包装（修仙境界/游戏等级）实测能显著提升观感。

## 9. 渲染与交付（含缓存策略）

```bash
~/venvs/manim/bin/manim f.py ClassName -pqh          # 不要加 --disable_caching！
cp media/videos/f/1080p60/ClassName.mp4 ~/Desktop/   # 成片直接放桌面（用户自行归档）
```

- **增量渲染**：Manim 对每个 `play()` 按内容哈希缓存 partial movie file；改片后重渲，
  未变的动画直接命中缓存，只渲染新增/修改的拍——追加卡片时几乎秒出
- 局部重渲可用 `--from_animation_number N [--to_animation_number M]`（调试某一拍时用）
- **注意**：`\underbrace` 会让 `TransformMatchingTex` 子物件错位报 IndexError，公式卡禁用
- 环境：`~/venvs/manim`（Python 3.12 + Manim CE 0.21）、TeX Live 2026、ffmpeg；中文楷体用 `"Kaiti SC"`
- 中文字体必须先渲染冒烟测试（Windows 下 KaiTi / macOS 下 Kaiti SC，跨平台要换名）
