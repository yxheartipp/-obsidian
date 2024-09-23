---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-09-24
sr-interval: 1
sr-ease: 250
---

原则上：能直接多卡训练，就不要用ZeRO；能用ZeRO-2就不要用ZeRO-3。
从左到右，越来越慢:
Stage 0 > Stage 1 > Stage 2 > Stage 2 + offload > Stage 3 > Stage3 + offloads
从做到右，所需GPU显存越来越少
Stage 0 < Stage 1 < Stage 2 < Stage2 + offload < Stage 3 < Stage3 + offloads
