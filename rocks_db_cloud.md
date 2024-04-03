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
如何从云端恢复？
1. 提前获取当前配置下的sst_file_size
2. 在本地文件系统新建目录
3. 判断云端上是否存在Manifest
4. 获得Manifest后，同步dbpath目录
5. 
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
- 在结果中删除所有非sst以及ManifestFile的文件。
#### CopyLocalFileToDest
- 将云端删除的规划取消
- 将云端文件复制到给定的目录
#### LoadLocalCloudManifest
- 生成Cloud上Manifest文件名
- 创建文件
- 从log中读取，并写入Manifest
#### ResyncDir
- 重新加载两端所有文件。
#### LoadCloudManifest
- [[#FetchCloudManifest]]
- [[#LoadLocalCloudManifest]]
- [[#FetchManifest]]

#### FetchCloudManifest
- 从目的地址中获取CloudManifest文件
 
#### FetchManifest
- 利用版本号获取manifest文件
