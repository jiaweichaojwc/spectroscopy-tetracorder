# Tetracorder 与 Davinci 环境变量配置及出图指南

本文档记录了在运行 Tetracorder 遥感矿物填图及后续可视化出图过程中，解决 `davinci` 相关命令缺失（`not found`）以及生成色彩叠加底图的完整步骤。

---

## 一、 问题背景
在执行 Tetracorder 结束阶段的脚本时，系统可能会提示：
> `davinci.image-cube-chan-to-gif: not found`

这是因为系统环境变量中未包含 `davinci` 的辅助脚本路径。通过将相关的命令路径加入到系统的 `PATH` 中，即可解决该问题。

---

## 二、 核心操作步骤

### 1. 把包含 davinci 辅助脚本的正确路径加入环境变量
在终端中执行以下命令，将 `davinci` 相关支持脚本的绝对路径临时导入到当前会话的 `PATH` 中：

    # 添加 davinci 核心及辅助命令路径
    export PATH=$PATH:/home/t1/tetracorder.cmds/tetracorder6.00a.cmds/davinci-cmds.for.usr.local.bin
    export PATH=$PATH:/home/t1/tetracorder.cmds/tetracorder6.00a.cmds/cmds.all.support
    export PATH=$PATH:/home/t1/tetracorder.cmds/tetracorder6.00a.cmds/cmds.color.support

> **提示**：如果需要永久生效，可以将上述 `export PATH` 语句写入到家目录下的 `~/.bashrc` 或 `~/.zshrc` 文件中。

---

### 2. 验证路径是否配置成功
可以通过 `which` 命令测试系统是否能够正确识别 `davinci` 的辅助脚本：

    which davinci.image-cube-chan-to-gif

*正常情况下，终端应当返回该脚本的完整绝对路径，即代表配置成功。*

---

### 3. 返回运行目录，生成底图并批量出图
完成环境变量配置后，切换到对应的色彩支持目录，开始生成底图并执行全套色彩渲染脚本：

    # 1. 进入指定的工作支持目录
    cd /workspace/1/prisma_run/cmds.color.support

    # 2. 生成丰度图像的 JPG 叠加底图（以 prismaCube 为例，半径/参数为 50）
    ../cmds.all.support/gen.fd.jpg.overlay+base.images prismaCube 50

    # 3. 执行最终的彩色结果全套渲染脚本
    ./make.color.results.all

---

## 三、 注意事项
* 路径 `/home/t1/tetracorder.cmds/...` 需要根据您服务器上 Tetracorder 的实际解压与存放位置进行微调。
* 确保在执行 `gen.fd.jpg.overlay+base.images` 时，对应的数据立方体名称（如示例中的 `prismaCube`）与当前工作目录下的文件保持一致。
