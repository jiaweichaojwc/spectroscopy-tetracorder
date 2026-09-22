# Tetracorder 6.00 自动化运行与监控脚本

该脚本用于对 238 波段的 PRISMA 高光谱数据进行一键初始化、环境配置、后台启动地质矿物填图计算，并提供了实时监控运行状态的方法。

## 完整执行代码

```bash
cd /workspace/1

# 1. 彻底清理旧的运行目录
rm -rf /workspace/1/prisma_run

# 2. 使用 238 波段的新文件重新生成向导环境
/t1/tetracorder.cmds/tetracorder6.00a.cmds/cmd-setup-tetrun /workspace/1/prisma_run prisma01a cube /workspace/1/prisma_238_bil.dat 0.0001 -T 3 50 C -P 0.8 1.1 bar image png shortcubeid prismaCube longcubeid prisma_tetracorder6.00 geology

# 3. 完整补齐地质成因文件夹
cp -r /t1/tetracorder.cmds/tetracorder6.00a.cmds/geologic-origins /workspace/1/prisma_run/

# 4. 进入运行目录并修复底层程序路径
cd /workspace/1/prisma_run
sed -i 's|/usr/local/bin/tetracorder6.00|/usr/local/bin/tetracorder|g' cmd.runtet

# 5. 正式点火启动！
time ./cmd.runtet cube > tetracorder.out 2>&1 &

# 6. 实时查看后台计算日志（按 Ctrl+C 可退出查看，程序会继续在后台运行）
tail -f /workspace/1/prisma_run/tetracorder.out
