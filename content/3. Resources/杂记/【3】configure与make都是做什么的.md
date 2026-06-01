---
publish: true
---

## 1. configure 阶段（配置阶段）

### 作用：

- ​**系统检测和配置**​：检测当前系统的环境、编译器特性、依赖库等
    
- ​**生成定制化的构建配置**​：根据检测结果生成适合当前环境的 Makefile
    
- ​**用户选项处理**​：处理用户通过命令行参数指定的配置选项

```bash
# 检测系统架构、编译器、库文件等
./configure

# 带参数的配置示例
./configure \
    --prefix=/usr/local \          # 指定安装路径
    --enable-debug \               # 启用调试模式
    --disable-shared \             # 禁用共享库
    CC=afl-clang-fast \            # 指定编译器
    CFLAGS="-O0 -g"               # 设置编译标志
```
### 生成的输出：

- `Makefile`- 主要的构建规则文件
    
- `config.h`- 包含系统特定定义的头文件
    
- `config.status`- 配置状态脚本
    
- 各种 `config.log`文件

## 2. make 阶段（构建阶段）

### 作用：

- ​**实际编译源代码**​：根据 Makefile 中的规则编译源文件
    
- ​**链接目标文件**​：将编译后的对象文件链接成可执行文件或库
    
- ​**处理依赖关系**​：自动处理文件之间的依赖关系

``` bash
# 基本编译
make

# 并行编译（推荐）
make -j$(nproc)

# 编译特定目标
make all-binutils      # 只编译 binutils
make all-objdump       # 只编译 objdump 工具

# 主要子任务
make all              # 编译所有目标（默认）
make install          # 安装编译结果
make clean            # 清理编译生成的文件
make distclean        # 彻底清理（包括 configure 生成的文件）
make check            # 运行测试套件
```
