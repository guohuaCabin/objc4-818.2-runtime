# objc4-818.2

## 源码下载地址

[Apple开发源码](https://opensource.apple.com/)：根据对应的版本找到相应的源码，例如： macos --> 10.15 -->搜索 objc4

[MacOS 10.15源码](https://opensource.apple.com/release/macos-1015.html)：直接搜索想要下载的源码名称，例如：搜索objc --> objc4/ -->选择相应的objc版本下载

## 依赖资源文件

编译object4的时候需要添加一些依赖文件，这些依赖文件通过以下源码查找（文件名“-”后面的的是版本号）：

```
dyld-733.6
libauto-187
Libc-1353.41.1
libclosure-74
libdispatch-1173.40.5
libplatform-220
libpthread-416.40.3
xnu-6153.11.26
```

# 源码编译

## 1. `unable to find sdk 'macosx.internal'`

```
选择`target ->objc -> Build Setting -> Base SDK - > macOS
选择`target ->objc-trampolines -> Build Setting -> Base SDK - > macOS
```

## 2. `'sys/reason.h' file not found`

- 从下载的依赖资源文件中找`reason.h`，`reason.h`文件的目录`xnu/bsd/sys/reason.h`
- 在`objc4`目录下新建 `include`文件，将`reason.h`文件放到`include/sys`文件夹下
- 设置`reason.h`路径，`target -> objc -> Build Settings - > Header Search Paths`中 添加`$(SRCROOT)/include`

## 3. `mach-o/dyld_priv.h' file not found`

- 在`include`文件中创建`mach-o`文件
- 在下载的依赖资源文件中找到 `dyld -> include -> mach-o -> dyld_priv.h`
- 拷贝到 `include/mach-o`文件夹下
- 打开`dyld_priv.h`文件，在顶部添加

```
  #define DYLD_MACOSX_VERSION_10_11 0x000A0B00
  #define DYLD_MACOSX_VERSION_10_12 0x000A0C00
  #define DYLD_MACOSX_VERSION_10_13 0x000A0D00
  #define DYLD_MACOSX_VERSION_10_14 0x000A0E00
```

[![dyld_priv添加宏定义](https://camo.githubusercontent.com/79a06ee390c5ce68839c9b79f72a3cd8388cdc71eb7b91a6afba9d3d904c2048/687474703a2f2f626c6f672e67756f68756164656e2e636f6d2f64796c645f707269765f646566696e65253230322e706e67)](https://camo.githubusercontent.com/79a06ee390c5ce68839c9b79f72a3cd8388cdc71eb7b91a6afba9d3d904c2048/687474703a2f2f626c6f672e67756f68756164656e2e636f6d2f64796c645f707269765f646566696e65253230322e706e67)

## 4. `'os/lock_private.h' file not found`

- 在下载的资源文件`libplatform ->private -> os ->lock_private.h. base_private.h`两个文件，拷贝到`include/os`中

## 5. `'pthread/tsd_private.h' file not found`和`'pthread/spinlock_private.h' file not found`

- 在下载的资源文件`libpthread ->private ->tsd_private.h. spinlock_private.h`两个文件，拷贝到`include/pthread`中

## 6. `'System/machine/cpu_capabilities.h' file not found`

- 在下载的资源文件`xnu --> osfmk --> machine --> cpu_capabilities.h`文件，拷贝到`include/System/machine`中

## 7. `'os/tsd.h' file not found`

- 在下载的资源文件`xnu --> libsyscall --> os --> tsd.h`文件，拷贝到新建的`include/os`中

## 8. `'System/pthread_machdep.h' file not found`

- 在下载的资源文件`Libc --> pthreads --> pthread_machdep.h`文件，拷贝到`include/System`中

## 9. `'CrashReporterClient.h' file not found`

- 在[Libc-825.24](https://links.jianshu.com/go?to=https%3A%2F%2Fopensource.apple.com%2Fsource%2FLibc%2FLibc-825.24) 按路径`Libc/include/CrashReporterClient.h`找到对应文件导入`include`目录下
- 在`target -> Build Setting -> Preprocessor Macros`中加入`LIBC_NO_LIBCRASHREPORTERCLIENT`或者在`CrashReporterClient.h`文件中更改了里面的宏信息 `#define LIBC_NO_LIBCRASHREPORTERCLIENT`

## 10. `'objc-shared-cache.h' file not found`

- 在下载的资源文件`dyld --> include --> objc-shared-cache.h`文件，拷贝到`include`目录下中

## 11. `Mismatch in debug-ness macros`

- 将`objc-runtime.mm` 中 `#error mismatch in debug-ness macros`注释掉

## 12. `'_simple.h' file not found`

- 在下载的资源文件`libplatform --> private --> _simple.h`文件，拷贝到`include`目录下中

## 13. `'Block_private.h' file not found`

- 在下载的资源文件`llibclosure-74 --> Block_private.h`文件，拷贝到`include`目录下中

## 14. `'kern/restartable.h' file not found`

- 在下载的资源文件`xnu --> osfmk --> kern --> restartable.h`文件，拷贝到新建的`include/kern`中

## 15. `can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/AppleInternal/OrderFiles/libobjc.order`

- 在`target -> objc -> Build Setting -> Order File`中添加搜索路径`$(SRCROOT)/libobjc.order`

## 16. `/xcodebuild:1:1: SDK "macosx.internal" cannot be located.`

- 在`target -> objc -> Build Phases -> Run Script(markgc)`把脚本文本里的`macosx.internal`改成`macosx`

## 17. `library not found for -lCrashReporterClient`

- 在`target -> objc -> Build Settings -> Other -> Linker Flags -> Debug/Release -> Any macOS SDK`中删除`-lCrashReporterClient`

## 18. `Static_assert failed due to requirement 'bucketsMask >= ((unsigned long long)140737454800896ULL)' "Bucket field doesn't have enough bits for arbitrary pointers."`

```
MACH_VM_MAX_ADDRESS` 在arm64架构下对应的是： `0x1000000000
```

- 在`objc-runtime-new.h` 中将其替换：`static_assert(bucketsMask >= 0x1000000000, "Bucket field doesn't have enough bits for arbitrary pointers."); `直接注释掉这行也可以
- `objc-runtime-new.mm`中注释掉: `STATIC_ASSERT((~ISA_MASK & MACH_VM_MAX_ADDRESS) == 0 || ISA_MASK + sizeof(void*) == MACH_VM_MAX_ADDRESS);`

## 19. libobjc.order 路径问题

```
Can't open order file: /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.15.sdk/AppleInternal/OrderFiles/libobjc.order
```

- 选择 `target` -> `objc` -> `Build Settings`
- 在工程的 `Order File` 中添加搜索路径 `$(SRCROOT)/libobjc.order`

## 20.其他

其他相关报错，直接注释掉代码

以下是举例，但不是全部

```c
#include <os/feature_private.h> 
```

```c
    if (!os_feature_enabled_simple(objc4, preoptimizedCaches, true)) {
        DisablePreoptCaches = true;
    }
```

```c
    if (!dyld_program_sdk_at_least(dyld_platform_version_macOS_10_13)) {
        DisableInitializeForkSafety = true;
        if (PrintInitializing) {
            _objc_inform("INITIALIZE: disabling +initialize fork "
                         "safety enforcement because the app is "
                         "too old.)");
        }
    }
```

### 20.3 objc-cache.mm类

```
#include <Cambria/Traps.h>
#include <Cambria/Cambria.h>
```



```
if (oah_is_current_process_translated()) {
            kern_return_t ret = objc_thread_get_rip(threads[count], (uint64_t*)&pc);
            if (ret != KERN_SUCCESS) {
                pc = PC_SENTINEL;
            }
        } else {
            pc = _get_pc_for_thread (threads[count]);
        }
```

改为：

```
//        if (oah_is_current_process_translated()) {
//            kern_return_t ret = objc_thread_get_rip(threads[count], (uint64_t*)&pc);
//            if (ret != KERN_SUCCESS) {
//                pc = PC_SENTINEL;
//            }
//        } else {
            pc = _get_pc_for_thread (threads[count]);
//        }
```



# 编译调试

- 新建一个 `Target` : **objc-debug**

[![img](http://blog.guohuaden.com/objc-debug-target.png

![img](http://blog.guohuaden.com/objc-debug-target.png)

- 绑定二进制依赖关系

![依赖关系](http://blog.guohuaden.com/objc-debug-dependencies.png)

- 运行代码进入源码

![runtime调试](http://blog.guohuaden.com/runtime%E8%B0%83%E8%AF%95.png)

