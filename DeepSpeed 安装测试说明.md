---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-09-21
sr-interval: 1
sr-ease: 250
---
1. 执行nvidia-smi  查看支持显卡支持的CUDA版本，要求CUDA版本至少>11.8
![[Pasted image 20240920143114.png]]
2. cd deepspeed
3. 在没有任何虚拟环境被激活的情况下创建虚拟环境 
```
conda env create -n deepspeed -f environment.yml --force
```
4. conda activate deepspeed // 激活虚拟环境
5. 执行如下命令，进行deepspeed的安装
```
TORCH_CUDA_ARCH_LIST="8.6" DS_BUILD_OPS=1 pip install . --no-cache
```
6.  ./bin/ds_report // 查看是否安装成功, 成功输出如下
![[Pasted image 20240920143635.png]]
8.  cd /test/small_model_debugging/
9.  ds test_model.py //可以直接执行简易的模型训练
![[Pasted image 20240920143846.png]]
