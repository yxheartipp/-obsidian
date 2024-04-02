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
1. 制作ColmnFamilyDescriptor 。
2. sst_file_manager生成或者是从CloudFileSystem生成。
3. 如果是新数据库就新建一个CloudManifest。
4. 每一次对MANIFEST的更新都会上传到S3中。
5. 对cfs定义了一个manifest一个循环滚动更新，每次更新本地manifest就更新云端manifest
#### DBCloud::Savepoint
- 从所有db中获取所有的sst
- 将所有sst复制到云上路径
#### DBCloudImpl::CheckpointToCloud
- 先暂时停止删除
- 创建新的Manifest
- 将当前所有file放入cloud
- 更新Manifest
- 恢复删除功能


### CloudFileSystemImpl
#### GetChildren
- 从cloud中获取给定目录下的所有文件
- 获取本地给定目录下所有的文件
- 在结果中删除所有非sst以及anifestFile的文件。
#### CopyLocalFileToDest
- 将云端删除的规划取消
- 将云端文件复制到给定的目录

