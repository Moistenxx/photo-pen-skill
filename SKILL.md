---
name: photo-pen-skill
description: Use when the user uploads one or more photos and wants each turned into a separate high-end design poster — 3:4 vertical, top half preserves the original photo (light editorial grade), bottom half becomes a 3×3 naïve hand-drawn doodle "memory grid" with handwritten title and caption. One poster per photo, never a multi-photo collage.
---

# Photo Memory Poster

## Overview
把照片做成"上半原图 + 下半手绘记忆清单"的高级海报。难点是**精确控制版式**：上半必须保留原图、下半必须是 3×3 涂鸦、两区严格 1:1。一次图像生成很难守住边界（模型常把整图重画或比例失控），所以用**程序化合成**最稳：原图放上半、单独生成涂鸦层放下半、垂直拼合。

## When to Use
- 用户上传照片，要求"做成海报 / 设计海报 / 视觉日记 / collection sheet / 手帐风 / memory poster"
- 明确要求 3:4 竖版、上下 1:1、上半原图、下半涂鸦或图标网格
- 要求"每张照片单独一张、不拼接"
- 关键词：doodle grid, hand-drawn, naïve illustration, 视觉日记, 手帐, 记忆清单, 9-grid, collection sheet

When NOT to use
- 用户要的是多图拼贴 / 九宫格整图排版
- 用户要写实插画或完整还原原图

## Core Workflow（程序化合成，强制）
**绝不**一次生成整张 3:4 就交差。整图 3:4，上下各占 50%，所以每半都是 **3:2** 矩形。严格走 4 步：

1. **准备上半（原图调色）**
   - 将原图裁切/缩放适配为 **3:2** 区域（为适配画幅可自然扩展环境背景，但不得拉伸/扭曲/改变主体）。
   - **照片非 3:2 时（常见竖版 9:16 / 3:4）**：默认做 3:2 智能横裁，垂直偏移略偏向主体（人 / 车 / 关键物），保证主体不丢；若用户要求保留完整照片，改用等比缩放 + 纸底留白（letterbox），绝不拉伸变形。
   - 轻微高级调色（可选）：用 ImageGen 对原图做 image-to-image，喂「上半调色提示词」，保主体、结构、姿态、光影、原色不变。
   - 用户要"纯保留"则跳过调色，直接用原图。

2. **生成下半（涂鸦层）**
   - 用 ImageGen 基于原图语义生成一张 **3:2** 插画层（白底 / 纸纹底，透明更佳）。
   - 喂「下半涂鸦提示词」。

3. **程序化拼合**
   - 用 Python PIL（或等价）把上半（3:2）与下半（3:2）垂直拼为 3:4。
   - 两区之间可留极细纸纹分隔线，保持边界清晰。
   - 建议输出尺寸：宽 1080 × 高 1440。

4. **逐张输出**
   - 每张上传照片 → 一张独立海报，文件名含原始照片名。**绝不**把多张塞进一图。

## 上半调色提示词（可选，喂 ImageGen image-to-image）
> subtle high-end editorial color grade, art magazine / independent publication / exhibition still quality; preserve subject identity, structure, pose, authentic texture, natural light and original color mood; only light grading, do not change the subject; DO NOT stretch, distort or alter the subject.

## 下半涂鸦提示词（喂 ImageGen，直接套用）
> Based on the semantic content of the SAME photo above, compress the scene into 9 most memorable visual units — the subject itself, a partial feature, a personal item, a plant, food, a transport mode, a pose, an emotion symbol, or a tiny scene memory point. Each unit is a semantic extension of the upper photo, so they read as one photo's "memory checklist".
> Style: Naïve Hand-drawn Doodle Illustration. Forms simple with slight childlike awkwardness; outlines lightly jittery, uneven thickness, not fully closed, deliberately imperfect. Fill not packed solid — let paper grain, white grain and rough edges show. Mix black thin-line outlines + crayon/colored-pencil fill + a few local color blocks; build relationships with flat color / spot color, NOT realistic light/shadow.
> Composition: implicit loose 3×3 grid (organic grid). The nine cells are only an underlying order, not rigid — each icon's position, size, angle and landing point shifts slightly, orderly yet not mechanical. Top: one short handwritten title with a few small wave-lines, stars or symbols. Bottom: one tiny handwritten caption. Overall like a private journal page / visual diary / collection sheet. Generous whitespace, relaxed, cute, restrained, not messy. If grid guide lines appear, make them extremely faint — like a pencil underdrawing, NOT a computer table.
> Output ONLY the bottom half: white or transparent background, 3:2 aspect ratio.

## Common Mistakes
- ❌ 一次生成整张 3:4 → 上半被重画、比例失控。**必须程序化拼合。**
- ❌ 下半做写实插画或完整描摹原图 → 违规，要压缩成 9 个图标单元。
- ❌ 多张照片拼进一张 → 违例，逐张输出。
- ❌ 网格画成工整电脑表格 → 要 loose / organic。
- ❌ 涂鸦铺色太满、无纸纹颗粒 → 要留白 + 颗粒感。
- ❌ 拉伸/扭曲主体适配画幅 → 只可扩展环境背景。

## Quick Reference
| 要素 | 规格 |
|------|------|
| 画幅 | 3:4 竖版 |
| 分区 | 上下 1:1，各 3:2 |
| 上半 | 原图 + 轻调色（可选） |
| 下半 | 3×3 naïve doodle，9 单元 |
| 输出 | 每图一张，不拼接 |
| 工具 | ImageGen（GPT Image 2.5 类）+ PIL 拼合 |

## Implementation（拼合示意）
```python
from PIL import Image
W, H = 1080, 1440
top = top_img.resize((W, H // 2))      # 上半：原图裁为 3:2
bot = bot_img.resize((W, H // 2))       # 下半：ImageGen 涂鸦层 3:2
canvas = Image.new("RGB", (W, H), "white")
canvas.paste(top, (0, 0))
canvas.paste(bot, (0, H // 2))
canvas.save(out_path)
```
