# STM32+Cmake+Clangd+Ozone开发一篇通

特别鸣谢：[bin](https://space.bilibili.com/3493142393260061) ，[跃鹿战队](https://blog.csdn.net/NeoZng/article/details/127980949)，[xiaozhou](https://blog.csdn.net/qq1291917670/article/details/119395340).

- [x] 此教程适合已经对Cmake整套开源链熟悉的同学，不熟悉的同学可以先了解这套开源链再进行配置
- [x] 此方案已在F1和G4上运行过
- [x] 注意：此套方案是单独安装每个模块，不是msys2整体安装，清楚自己需要哪个教程

## 一. 四件套（在固定位置取名Toolchain文件）

- Cmake
- Ninja
- Clangd
- arm-none-eabi-gcc

### 1.安装Cmake

- 锚点：https://github.com/Kitware/CMake/releases/download/v4.0.1/cmake-4.0.1-windows-x86_64.zip
- 目前最新版本 4.0.1(2025.4.18)
- 下载好之后，放到一个稳定的路径下，来到`bin`目录下复制路径，添加到环境变量，类似这样

- ```
  C:\Toolchain\Cmake4_0\bin
  ```

### 2.安装ninja或者make（这里安装Ninja，可能需要魔法）

- 锚点：[https://github.com/ninja-build/ninja/releases/download/v1.12.1/ninja-win.zip](https://github.com/ninja-build/ninja/releases/tag/v1.12.1)（win版本）

- 目前最新版本 1.12.1(2025.4.18)

- 下载好之后，放到一个稳定的路径下，来到目录下复制路径，添加到环境变量，类似这样

- ```
  C:\Toolchain\Ninja_win
  ```

### 3.安装Clangd（可能需要魔法）

- 锚点：https://github.com/llvm/llvm-project/releases/download/llvmorg-20.1.3/clang+llvm-20.1.3-x86_64-pc-windows-msvc.tar.xz

- 目前最新版本 20.1.3(2025.4.18)

- 下载好之后，放到一个稳定的路径下，来到`bin`目录下复制路径，添加到环境变量，类似这样

- ```
  C:\Toolchain\clang+llvm-20.1.2-x86_64-pc-windows-msvc\bin
  ```

### 4.安装arm-none-eabi-gcc

- 锚点：https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi.zip
- 目前最新版本 14.2(2025.4.18)
- 下载好之后，放到一个稳定的路径下，来到`bin`目录下复制路径，添加到环境变量，类似这样

```
C:\Toolchain\arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi\bin
```

#### PS：验证是否添加环境变量起效（用Git Bash打开）

```
cmake --version
```

```
ninja --version
```

```
clangd --version
```

```
arm-none-eabi-gcc --version
```



## 二.安装vscode和Git

- 传送锚点：[`vscode`下载传送门](https://code.visualstudio.com/download#)，选择`system installer x64`版本下载并进行安装，位置任意选择。
- 锚点：[Git - 下载软件包](https://git-scm.com/downloads/win)

​	打开`vscode`，在插件市场搜索以下对应的插件

- clangd

- Material Icon Theme(切换主题图标)

- Task Buttons（创建按钮任务）

- Chinese (Simplified) (简体中文)

- Clang-Format（规范格式）

  
  
  


## 三.装`STM32CubeMX`

- [`STM32CubeMX`下载传送门](https://www.st.com/en/development-tools/stm32cubemx.html)，下载最新版本（写教程的时候是6.14）

## 四.打开cubemx生成Cmake工程（这里用STM32_Link命名工程名字）

-  生成代码时，`Toolchain`下拉选择`cmake`

## 五.用vscode打开生成的Cmake工程  ----->STM32_Link

#### 	我们会使用一些命令行来完成手动编译，主要是熟悉一下编译过程，也会给出全自动的脚本工具，一键运行即可完成编译（抄一抄bin佬的原话。QAQ）

- 手动配置部分
  - 编译部分

> 1. 使用vscode打开工程后，打开一个终端，这里推荐使用git bash
> 2. 在打开的终端里输入以下命令

```
mkdir build #存放编译的过程文件和elf文件以及之后会提到的bin和hex文件
cd build    #进入build目录
cmake -G Ninja ..     使用camke生成ninja 可以执行的编译文件build.ninja（可以对比理解成之前教程里提到的makefile）
ninja -j 8  ninja根据build.ninja执行构建，最后生成elf文件

# 输入并且执行（输入完命令后按下回车：Enter键）完以上四条命令后，build目录下就会生成一个led_blink.elf文件（这个文件可以用来调试，也可以直接烧录，还可以使用工具生成bin和hex文件供烧录调试）

arm-none-eabi-objcopy -O binary led_blink.elf led_blink.bin #转换为bin文件
arm-none-eabi-objcopy -O ihex led_blink.elf led_blink.bin   #转换为hex文件
```

##### 	PS：这里我取消了烧录和调试，转用ozone进行烧录和调试

#### 这里给出烧录脚本和按键配置

- 将以下脚本复制到工程的根目录，并且取名为`run.sh`，叫什么名字还是随意，这里只是作为示例（又抄抄bin佬的原话☆*: .｡. o(≧▽≦)o .｡.:*☆）

- 这里我对bin佬的源代码进行更改，加上了颜色和内存进度的条框，取消了调试和下载

- ```
  #!/bin/bash
  
  # 定义颜色代码
  COLOR_RED="\e[31m"
  COLOR_GREEN="\e[32m"
  COLOR_YELLOW="\e[33m"
  COLOR_BLUE="\e[34m"
  COLOR_BOLD_GREEN="\e[1;32m"
  COLOR_RESET="\e[0m"
  
  # 记录编译开始时间
  start_time=$(date +%s)
  
  # 创建并进入 build 目录，每次都会删除之前的文件
  mkdir -p build
  cd build
  rm -rf _*
  
  # 运行 CMake 和 Ninja 构建
  echo -e "${COLOR_BLUE}正在生成构建系统...${COLOR_RESET}"
  cmake -G Ninja .. && \
  echo -e "${COLOR_BLUE}开始编译项目...${COLOR_RESET}" && \
  cmake --build . --target all --config Release -- -j 16
  
  # 检查构建是否成功
  if [ $? -ne 0 ]; then
      echo -e "${COLOR_RED}构建失败！${COLOR_RESET}"
      exit 1
  fi
  
  # 获取当前工程所在的盘符
  drive_letter=$(pwd | cut -d'/' -f2 | tr '[:lower:]' '[:upper:]')
  
  # 修改 compile_commands.json 文件中的路径
  sed -i "s|/c/|C:/|g; s|/d/|D:/|g; s|/e/|E:/|g; s|/f/|F:/|g; s|/g/|G:/|g; s|/h/|H:/|g; s|/i/|I:/|g; s|/j/|J:/|g; s|/k/|K:/|g; s|/l/|L:/|g; s|/m/|M:/|g; s|/n/|N:/|g; s|/o/|O:/|g; s|/p/|P:/|g; s|/q/|Q:/|g; s|/r/|R:/|g; s|/s/|S:/|g; s|/t/|T:/|g; s|/u/|U:/|g; s|/v/|V:/|g; s|/w/|W:/|g; s|/x/|X:/|g; s|/y/|Y:/|g; s|/z/|Z:/|g" compile_commands.json
  
  # 获取 .ioc 文件的前缀
  ioc_file=$(basename ../*.ioc .ioc)
  elf_file="$ioc_file.elf"
  bin_file="$ioc_file.bin"
  hex_file="$ioc_file.hex"
  
  # 检查 ELF 文件是否存在
  if [ ! -f "$elf_file" ]; then
      echo -e "${COLOR_RED}错误：未找到 ELF 文件 $elf_file！${COLOR_RESET}"
      exit 1
  fi
  
  # 生成二进制文件和 HEX 文件
  echo -e "${COLOR_BLUE}生成二进制文件...${COLOR_RESET}"
  arm-none-eabi-objcopy -O binary "$elf_file" "$bin_file"
  echo -e "${COLOR_BLUE}生成 HEX 文件...${COLOR_RESET}"
  arm-none-eabi-objcopy -O ihex "$elf_file" "$hex_file"
  
  # 检查二进制文件是否生成成功
  if [ ! -f "$bin_file" ]; then
      echo -e "${COLOR_RED}错误：未生成二进制文件 $bin_file！${COLOR_RESET}"
      exit 1
  fi
  
  # 检查 HEX 文件是否生成成功
  if [ ! -f "$hex_file" ]; then
      echo -e "${COLOR_RED}错误：未生成 HEX 文件 $hex_file！${COLOR_RESET}"
      exit 1
  fi
  
  # 打印内存使用情况（带彩色进度条）
  print_memory_bar() {
      local used=$1
      local total=$2
      local label=$3
      
      # 转换为字节（输入单位KB）
      total_bytes=$((total * 1024))
      
      # 计算使用百分比
      if [ $total_bytes -eq 0 ]; then
          echo "错误：总大小不能为零！"
          return
      fi
      
      percent=$((used * 100 / total_bytes))
      ((percent > 100)) && percent=100
  
      # 根据百分比选择颜色
      if [ $percent -ge 90 ]; then
          color=$COLOR_RED
      elif [ $percent -ge 80 ]; then
          color=$COLOR_YELLOW
      else
          color=$COLOR_GREEN
      fi
  
      # 构建进度条
      bar_length=30
      filled=$((percent * bar_length / 100))
      empty=$((bar_length - filled))
      bar=$(printf "%${filled}s" | tr ' ' '█')
      space=$(printf "%${empty}s")
  
      # 转换使用量为KB并格式化输出
      used_kb=$(awk "BEGIN {printf \"%.2f\", $used/1024}")
      echo -e "${label}: ${color}${bar}${space}${COLOR_RESET} ${used_kb}KB/${total}KB (${percent}%)"
  }
  
  # 获取内存使用数据
  echo -e "\n${COLOR_BOLD_GREEN}内存使用分析：${COLOR_RESET}"
  size_info=$(arm-none-eabi-size "$elf_file" | tail -n1)
  text=$(awk '{print $1}' <<< $size_info)
  data=$(awk '{print $2}' <<< $size_info)
  bss=$(awk '{print $3}' <<< $size_info)
  
  # 计算各区域使用量（字节）
  rom_used=$((text + data))      # Flash使用量 = text + data
  ram_used=$((data + bss))       # RAM使用量 = data + bss
  ccmram_used=$(arm-none-eabi-objdump -h $elf_file | awk '/.ccmram/ {print "0x"$3}' | xargs printf '%d') 
  
  # 设置芯片内存参数（根据实际芯片修改以下数值）
  FLASH_SIZE=128   # KB
  RAM_SIZE=12     # KB
  CCMRAM_SIZE=64    # KB
  
  # 显示内存使用情况
  print_memory_bar $rom_used $FLASH_SIZE "FLASH"
  print_memory_bar $ram_used $RAM_SIZE "RAM"
  [ $CCMRAM_SIZE -gt 0 ] && print_memory_bar $ccmram_used $CCMRAM_SIZE "CCMRAM"
  
  # 计算编译耗时
  end_time=$(date +%s)
  duration=$((end_time - start_time))
  
  
  # 打印编译成功信息
  echo -e "\n${COLOR_RED}编译成功！！！${COLOR_RESET}"
  echo -e "${COLOR_GREEN}编译耗时: ${COLOR_GREEN}${duration}秒${COLOR_RESET}"
  ```

  ##### 添加Task Buttons任务

- 实现按键下载和添加自己的文件

1.在vscode的设置中的settings.json添加

```
"VsCodeTaskButtons.tasks": [
    {
        "label": "$(notebook-move-down) Build",
        "task": "Build", // 对应 tasks.json 中的 label: "Build"
        "tooltip": "🛠️ Start the \"Build\" task"
    }
]
```

2.在项目文件的.vscode文件夹创建tasks.json,并在其中添加以下代码（例如：C:\Users\X\Desktop\test\STM_Link\\.vscode\tasks.json）

```
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build",
            "type": "shell",
            "command": "./run.sh",         //这里是你的烧录脚本文件
            "options": {
                "cwd": "${workspaceFolder}",
                "shell": {
                    "executable": "E:/Git/bin/bash.exe",    //git Bash的位置
                    "args": ["-l", "-i"]
                }
            },
            "presentation": {
                "reveal": "always",
                "panel": "dedicated",
                "terminal": {
                    "name": "Git Bash",
                    "shellPath": "E:/Git/bin/bash.exe"   
                }
            }
        },
        
    ]
}
```

##### 如何添加自己的文件：在根目录的Cmakelists下

```
file(GLOB_RECURSE USER_SOURCES     //动态文件,包含当前文件夹下的c和cpp文件(注释要删除，不然报错)
${CMAKE_SOURCE_DIR}/Source/Src/*.c
${CMAKE_SOURCE_DIR}/Source/Src/*.cpp
)

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here
    ${USER_SOURCES}     //包含USER_SOURCES这个(注释要删除，不然报错)
)

# Add include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
    ${CMAKE_SOURCE_DIR}/Source/Inc     //头文件，包含当前文件夹下的所有.hpp和.h(注释要删除，不然报错)

)
```

### 添加好以上脚本，我们就实现了按钮编译和添加自己的文件

### 	一些小知识点（直接copy bin佬的 {{{(>_<)}}}）：具体参考[这里](https://github.com/threekb/cmake_ninja_clangd_stm32_dev_env)

#### 1.这个必须要添加，不然clangd会错误。

1. ```
   ./vscode/settings.json
   ```

   > 在这个文件里可以自行修改编译器路径，以下为示例（如果不懂如何配置，照搬即可，即在`C`盘下创建`Tooclchain`文件夹，再将解压后的编译器放到这个文件夹中即可）

> 

```
{
   "clangd.arguments": [
     "--query-driver=C:/Toolchain/arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi/bin/arm-none-eabi-gcc.exe",  // 指定编译器路径
     "--clang-tidy"
   ]
 }
```

#### 2.

1. ```
   .clangd
   ```

   > 配置这个文件可以修改工程编译时默认链接的标准库（例如`<math.h>`和`<stdio.h>`头文件），若工程里不做添加，则会默认链接到别的标准库，例如`msvc`或者`mingw64`
   >
   > 如果不会配置，就将编译器的位置与示例同步即可

```
CompileFlags:
  Add:
    - --target=arm-none-eabi          # 指定目标架构
    - -IC:/Toolchain/arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi/arm-none-eabi/include  # ARM-GCC头文件路径
    - -IC:/Toolchain/arm-gnu-toolchain-14.2.rel1-mingw-w64-i686-arm-none-eabi/lib/gcc/arm-none-eabi/14.2.1/include  # GCC库头文件
  CompilationDatabase: .              # 指向compile_commands.json
```

#### 3.

1. ```
   /cmake/gcc-arm-none-eabi.cmake
   ```

   > 其实这个文件中只需要添加以下命令到文件结尾处即可

```
# 有中文注释的部分，是cubemx不会自动生成的，需要手动添加
set(CMAKE_C_LINK_FLAGS "${CMAKE_C_LINK_FLAGS} -u _printf_float")  # 支持 printf 函数打印浮点数
set(CMAKE_C_LINK_FLAGS "${CMAKE_C_LINK_FLAGS} -lm")  # 链接数学库 libm
set(CMAKE_EXE_LINKER_FLAGS "-Wl,--gc-sections,--no-warn-rwx-segments")  # 取消 rwx 段的警告
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-parameter")  # 忽略 C 代码中未使用参数的警告
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-parameter")  # 忽略 C++ 代码中未使用参数的警告
```

#### 4.

1. `/core/src/usart.c`

   > 注意这个文件中的用户代码区中的以下代码，这段代码用于处理串口重定向

   > `main`函数里使用`printf`函数时，需要添加头文件`<stdio.h>`

```
int _write (int fd, char *pBuffer, int size)
{
  for (int i = 0; i < size; i++)
  {
    while((USART1->SR&0X40)==0);//等待上一次串口数据发送完成
    USART1->DR = (uint8_t) pBuffer[i];    //写DR,串口1将发送数据
  }
  return size;
}
```

## 六.ozone和Jlink安装(｡･∀･)ﾉﾞ嗨

###      安装Ozone3.24 32-bit和J-Link7.22b（[根据鹿鹿的文章](https://blog.csdn.net/NeoZng/article/details/127980949)）

- ##### 	锚点：[点这里](https://www.segger.com/downloads/jlink/#Ozone)(●'◡'●)

- #####         步骤：先安装Ozone，再安装jlink

- #####          安装ozone注意选择 install a new instance

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421155416838.png)

- ​       安装Jlink（选项和图片保持一致）

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421155529391.png)



## 七.调试器刷机教程（stlink --> Jlink）

- ##### ST-Link Reflash Utility 锚点：[点这里](https://www.segger.com/downloads/jlink#STLink_Reflash)，安装位置跟Jlink和ozone一个的位置

- ##### Winhex锚点：[点这里](https://www.ddooo.com/softdown/20756.htm)，安装位置跟Jlink和ozone一个的位置

- 位置放置![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421151730076.png)

##### 	步骤：1.先用winhex打开ST-Link Reflash.exe

##### 		2.修改ST-Link Reflash.exe![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152306143.png)

```
地址0x2566处的0x3C修改为0x38
地址0x2567处的0x40修改为0xC0
地址0x26B2处的0x3C修改为0x38
地址0x26B3处的0x4A修改为0xC0
然后ctrl+s保存
```

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152448343.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152625177.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152708217.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152744013.png)

##### 破解完成，开始愉快的刷机啦ヾ(≧▽≦*)o

- ###### 双击打开ST-Link Reflash.exe，然后双accpet，跳到以下界面

- ![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421152859907.png)

- 先插上我们的Stlink等待设备管理器有stlink出现，然后再命令行输入1：upgrate to J-link,等待进度跑完

- ![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153021495.png)

- ![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153102847.png)

- ##### ps：如果没有出现以上，建议关闭ST-Link Reflash.exe在重新开始刷机

## 八，完成以上操作，就开始我们的程序烧录和调试啦\^o^/,真不容易呀

### 步骤：cubemx生成的cmake工程（STM32_Link）,用vscode打开，然后编译得到elf文件，打开ozone，选择elf文件，下载调试

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153443683.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153331104.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153359573.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153517416.png)

##### 然后一路next😊

###### ps：因为ozone字体很小，需要自己在tool下拉的Preferences里面调节

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153550946.png)

连接开发板（我这里是stm32f103c8t6），运行

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153616570.png)

###### 简单写个sin代码，编译生成

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421153702801.png)

![](C:\Users\12715\Desktop\ALL_DOCUMENT\stm32-cmake-clangd-ozone\Picture\20250421154101669.png)

## 写在最后（抄抄bin佬的）

- 如果在使用脚本后，发现代码依旧爆红，请按下`ctrl+shift+p`，输入`clangd`，找到`clangd: Restart Language Server`，使用鼠标点击，或者使用键盘上的方向键选中后按下回车，此时你的代码应该就不会爆红了，而且代码提示和跳转也会恢复正常.
- 或者你想vx求助，这是神秘代码：XXyy123100
- 此教程的工程以及markdown文件会上传到github，这是[github](https://github.com/XXYY04)主页
- author：[Xy](https://space.bilibili.com/523214548?spm_id_from=..0.0)

##### 参考文献：(感谢开源o(*≧▽≦)ツ┏━┓)

bin：[threekb/cmake_ninja_clangd_stm32_dev_env](https://github.com/threekb/cmake_ninja_clangd_stm32_dev_env)

跃鹿战队：[CubeMX+VSCode+Ozone的STM32开发工作流（三）利用Ozone进行可视化调试和代码分析_ozone jlink-CSDN博客](https://blog.csdn.net/NeoZng/article/details/127980949)

xiaozhou.：[ST-Link刷成JLink来使用J-Scope工具_stlink刷jlink-CSDN博客](https://blog.csdn.net/qq1291917670/article/details/119395340)