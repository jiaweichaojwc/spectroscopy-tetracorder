# Tetracorder 6.00 适配 EnMAP 高光谱数据全流程指南

**文档说明**：本指南用于指导如何在极度依赖老旧 Fortran 架构的 USGS Tetracorder 6.00 中，手动适配并运行 219 波段的 EnMAP 卫星高光谱数据。

---

## 阶段一：输入数据准备与清洗

Tetracorder 对输入文件的命名、路径长度和头文件格式有极其严苛的限制。

### 1. 整理波长与分辨率文件
准备两个纯文本文件，代表 EnMAP 的 219 个波段信息。
*   **`waves.txt`**：中心波长（单位：微米）
*   **`resol.txt`**：半高全宽 FWHM（单位：微米）
*   **⚠️ 避坑**：必须是严格的单列纯数字文本，共 219 行，**绝不能包含任何不可见的制表符（Tab）或表头**。

### 2. 精简 ENVI 头文件（极其重要）
老旧的头文件解析器无法处理现代卫星数据庞大的科学计数法数组，会导致程序死锁。
1. 将影像命名为简短的名称，例如 `cc.dat`（绝对路径长度必须 **<73个字符**）。
2. 将头文件严格命名为 **`cc.dat.hdr`**，与数据放在同级目录。
3. 打开头文件，**删除冗长的数据块**：
   找到并彻底删除 `data gain values = {...}` 和 `data offset values = {...}` 整个括号及中间的所有内容。仅保留行列数、波长数组和基础元数据。

---

## 阶段二：建立专属光谱库（字典卷积）

Tetracorder 无法直接使用其自带的实验室光谱，必须先将其“降维/卷积”成与 EnMAP 完全一致的 219 波段光谱库。

### 1. 卷积主光谱库（包含 7000+ 光谱的大库）
进入基础转换目录：
```bash
cd /sl1/usgs/library06.conv/
```

执行一键生成脚本：
```bash
# 注意：库名必须严格限制为 7 个字符，例如 s06enmp
./AAA.make.new.instrument.convolved.spectral.library.sh s06enmp a 219 ENMAP 12 'Convolved ENMAP 219 ch library' 'Wavelengths in microns 219ch' 'Resolution in microns 219ch' -waves /您的路径/waves.txt -fwhm /您的路径/resol.txt noX
```
*等待 1-3 分钟，执行完毕后生成大库文件 `s06enmpa`。*

### 2. 卷积研究级光谱库（247 个核心矿物的小库）
进入研究库目录：
```bash
cd /sl1/usgs/rlib06
```

执行研究库卷积：
```bash
./AAA.make.new.instrument.convolved.rlib-spectral.library.sh r06enmap a 219 ENMAP 12 s06enmp a 1
```
*执行完毕后生成研究库文件 `r06enmapa`。*

---

## 阶段三：配置 Tetracorder 运行环境

在此阶段，我们需要告诉主程序如何读取刚才生成的 EnMAP 光谱库。

进入 Tetracorder 主配置目录：
```bash
cd /t1/tetracorder.cmds/tetracorder6.00a.cmds/
```

### 1. 配置重启文件 (Restart File)
以系统模板为基础建立 EnMAP 专属配置：
```bash
cp restart_files/r1-emitc restart_files/r1_enmapc
vi restart_files/r1_enmapc
```

修改以下关键变量，指向我们第二阶段生成的库文件：
```text
iwfl=/sl1/usgs/rlib06/r06enmapa
iyfl=/sl1/usgs/library06.conv/s06enmpa
irfl=r1_enmapc
iwdgt= r06enmapa
inmy=  s06enmpa
nchans= 219
```

### 2. 配置四大数据规则小文件
依次在主配置目录下的子文件夹中建立 EnMAP （代号设为 `enmap_c`）的支持文件：

**① DATASETS 注册数据**
```bash
cp DATASETS/emit_c DATASETS/enmap_c
vi DATASETS/enmap_c
```
内容改为：
```text
data=    ENMAPc
restart= r1_enmapc
```

**② DELETED.channels 声明坏波段**
```bash
echo "c   # enmap_c" > DELETED.channels/delete_enmap_c
```

**③ COLOR.channels 配置假彩色底图波段**
```bash
cp COLOR.channels/color-aviris_1995 COLOR.channels/color-enmap_c
vi COLOR.channels/color-enmap_c
```
挑选影像中的红绿蓝波段序号（1~219），例如：
```text
BASE    46 46 46  base-image.jpg
COLOR1  46 28 12  color-visRGB.jpg
```

**④ DISABLE 复制掩膜开关**
```bash
cp DISABLE/emit_c DISABLE/enmap_c
```

---

## 阶段四：生成环境与执行匹配

### 1. 生成独立工作区
确保身在 `/t1/tetracorder.cmds/tetracorder6.00a.cmds/` 目录，执行终极设置命令：
```bash
# 参数依次为：工作区文件夹名, 仪器代号, 数据类型, 影像绝对路径, 反射率缩放因子(如果是万分一则填 0.0001), 基础运行参数...
./cmd-setup-tetrun map_enmap_run enmap_c cube /workspace/1/cc.dat 0.0001 -T -3 50 C -P 0.8 1.1 bar shortcubeid ENMP longcubeid EnMAP-Test geology
```

### 2. 一键跑数据
设置成功后，进入生成的工作区并启动主程序：
```bash
cd map_enmap_run
./cmd.runtet
```
*此时屏幕将滚动输出 `Processing line 1...` 等日志。运行结束后，生成的矿物丰度图、分类影像及相关文本结果将全数保存在该目录下的 `results/` 和 `results.masses/` 文件夹中。*

---

## 💡 常见报错总结与避坑
*   **报错 `invalid input` 并卡死**：通常是因为 `restart` 文件中 `iyfl` 指向的库为空或错误。检查阶段二的“主光谱库”是否成功生成。
*   **报错 `ERROR on opening ENVI header file`**：头文件命名不符合规范，确保影像叫 `xxx.dat`，头文件必须叫 `xxx.dat.hdr`。
*   **读取头文件后程序死锁（敲回车无反应）**：进入 `.hdr` 文件中，删掉所有极长的大括号数组（如 gain/offset），拯救老旧解析器。
