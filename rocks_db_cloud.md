---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-04-02
sr-interval: 1
sr-ease: 250
---
### Cloud Manifest
#### DumpToRandomFile:
- 新建writableFile
- 调用WriteToLog：里面放进了Header以及Record 人工构建
- LoadFromLog， 从log中加载到manifest中。
#### LoadFromFile
- 从log中恢复到Manifest中。
#### GetCurrentEpoch
- 从Cloud中Manifest中获取当前的Epoch_


### db_cloud_impl
#### DBCloud::Open
如何从本地恢复？
