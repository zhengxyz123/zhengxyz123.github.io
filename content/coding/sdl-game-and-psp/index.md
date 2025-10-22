+++
title = "使用SDL开发PSP游戏时需要注意的地方"
date = 2025-09-29T21:49:22+08:00
tags = ["闲话", "游戏开发"]
+++

最近在使用SDL2开发PSP游戏，遇到了一些奇奇怪怪的小问题，在这里分享一下。

<!--more-->

首先是SDK相关的问题。我使用的是PSPDEV提供的[一整套工具链](https://github.com/pspdev)。

`psp-pacman`是一个好东西，我们可以直接安装一些编译起来非常麻烦的依赖项。不过它提供的`pacman`需要`libgpgme.so.11`，这在Arch Linux上早就已经不复存在了，所以我直接使用了系统提供的`pacman`。

想使用`setlocale`？当然可以！不过第二个参数只能是`"C"`或`"POSIX"`，这是因为newlib（PSPDEV使用的C库，通常用于嵌入式系统）[默认只提供了最小化的实现](https://github.com/pspdev/newlib/blob/master/newlib/libc/locale/locale.c#L29)。这也同时意味着类似`mbtowc`等依赖区域设置的函数用了相当于没用。

为了能够同时编译PSP应用，`CMakeLists.txt`需要修改一下。

加载依赖包的时候应该这样做：
```CMakeLists.txt
if(NOT PSP)
    find_package(SDL2 REQUIRED)
    find_package(SDL2_image REQUIRED)
    find_package(SDL2_mixer REQUIRED)
    find_package(SDL2_ttf REQUIRED)
else()
    include(FindPkgConfig)
    pkg_search_module(SDL2 REQUIRED sdl2) # 这里是小写
    pkg_search_module(SDL2_IMAGE REQUIRED SDL2_image)
    pkg_search_module(SDL2_MIXER REQUIRED SDL2_mixer)
    pkg_search_module(SDL2_TTF REQUIRED SDL2_ttf)
endif()
```

链接的时候应该这样做：
```CMakeLists.txt
if(NOT PSP)
    target_link_libraries(${PROJECT_NAME}
        SDL2::SDL2 SDL2_image::SDL2_image
        SDL2_mixer::SDL2_mixer SDL2_ttf::SDL2_ttf
        ...
    )
else()
    target_include_directories(${PROJECT_NAME} PRIVATE
        ${SDL2_INCLUDE_DIRS}
        ${SDL2_IMAGE_INCLUDE_DIRS}
        ${SDL2_MIXER_INCLUDE_DIRS}
        ${SDL2_TTF_INCLUDE_DIRS}
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE
        ${SDL2_LIBRARIES}
        ${SDL2_IMAGE_LIBRARIES}
        ${SDL2_MIXER_LIBRARIES}
        ${SDL2_TTF_LIBRARIES}
        ...
    )
    create_pbp_file(
        TARGET ${PROJECT_NAME} # 只有这一项是必须的
        ICON_PATH NULL # 大小为 144x82 的 .png 文件
        BACKGROUND_PATH NULL # 大小为 480x272 的 .png 文件
        PREVIEW_PATH NULL # 大小为 480x272 的 .png 文件
        TITLE ${PROJECT_NAME}
        VERSION 1.0
    )
endif()
```

一定要记得是使用`psp-cmake`而不是`cmake`！

- - -

接下来就是有关PSP的一系列问题，这些问题主要归功于其孱弱的硬件。

PSP最大只支持大小为512×512的贴图。这个限制可能在各种意想不到的地方坑你一下，比如使用`TTF_Render*_Wrapped`的时候，`wrapLength`参数必须小于512（屏幕宽度才480呢）。

PSP的CPU（MIPS32）不支持SIMD指令。好巧不巧的是`SDL_ttf`又需要SIMD指令来加速，导致绘制文字的时候非常慢。所以我给出了以下解决方案：

- 从贴图加载并绘制文字
  - 优点：真的非常快；缺点：大概率只能绘制英文字符、半角符号及数字
- 使用[libintrafont](https://github.com/pspdev/libintraFont)加载PGF字体并绘制文字
  - 优点：至少比`SDL_ttf`快；缺点：只适用于PSP而且PGF格式几乎失传
- 使用`SDL_ttf`绘制文字并缓存
  - 优点：没有优点；缺点：全是缺点

> 一个Noto CJK字体的大小就有60MB了，然而PSP的最大内存才64MB（其中PSP1000只有32MB），一个字体就占用了90%以上的内存，那别的事情就可以不要干了。

不过Vita并没有这方面的问题，它支持大小为4096×4096的贴图（虽然足够大，但仍然比PC上的16384×16384小不少）,内存大小达到512MB，且其使用的ARM CPU支持Neon。完美解决PSP的一系列痛点。
