---
aliases: 
Type: 
tags:
  - "#review"
sr-due: 2023-01-01
sr-interval: 1
sr-ease: 250
---

本文目的在于讨论在obsidian中模仿supermemo进行渐进阅读的可行性。obsidian作为一款现代双链笔记软件，可以有效避免supermemo中卡片重复、数学公式无法编辑和图片丢失等问题。请下载我提供的obsidian仓库按下文进行操作。

## 概念介绍

本文中，obsidian中的笔记相当于supermemo中的摘录卡片。在这里我建议大家可以按KG笔记法的理念进行笔记的制作。

## 渐进阅读步骤

1. 通过快捷键`Shift + O`开始当日的渐进阅读。
2. 通过快捷键`Alt + W`对当日需复习的笔记进行遍历。我在`Spaced Repeatition`插件中设置了`复习后自动打开下一个笔记`，所以快捷键`Alt + W`会设置好当前笔记的下一个复习日期，并打开下一个需要复习的笔记。单个笔记的历史间隔是逐渐增大的，这一点和supermemo对摘录卡片的处理相似。
3. 通过快捷键`Alt + X`对选中的文本进行摘录和命名。我制作好的模版[[TP-KG-Link]]和核心插件`笔记重组`会自动将选中文本设为一篇笔记并设置好其下次出现的时间，即笔记元语言区的`sr-due:`字段。
4. 当你对笔记完成问答/填空卡片制作后，可通过快捷键`Ctrl + D`将笔记的`tags: ["#review"]`改为`tags: ["#Dismiss"]`，笔记将不会被插件`Spaced Repeatition`识别，从而不再出现在后续的复习中。你也可以将`Dismiss`改回`review`重新开始复习。

以上就是在obsidian中进行渐进阅读的所需要的关键操作，你可以根据渐进阅读的理念在每次`Alt + W`之前，对笔记进行修改和摘录。

## 制作问答/填空卡的三种方法：
- 使用obsidian的插件`Spaced Repeatition`，在obsidian中进行制卡和背诵；
- 使用obsidian另一款插件`Obsidian Anki Sync`制卡并同步到anki中背诵；
- 使用quicker动作将obsidian中选中的内容传输到supermemo中进行制卡和背诵。


