---
title: hijack_mac
date: 2024-06-17 01:17:51
tags:
---

# mac地址欺骗

## 系统调用劫持

通过劫持系统调用，可以拦截和修改应用程序请求的硬件信息，如 MAC 地址。

#### 使用 `LD_PRELOAD` 劫持 `getifaddrs` 系统调用

1. **编写劫持库**：

   - 创建一个共享库，劫持 `getifaddrs` 系统调用以返回伪造的 MAC 地址。

   ```c
   #define _GNU_SOURCE
   #include <dlfcn.h>
   #include <ifaddrs.h>
   #include <netpacket/packet.h>
   #include <net/if.h>
   #include <stdio.h>
   
   int getifaddrs(struct ifaddrs **ifap) {
       // 获取原始的 getifaddrs 函数
       int (*original_getifaddrs)(struct ifaddrs **);
       original_getifaddrs = dlsym(RTLD_NEXT, "getifaddrs");
   
       // 调用原始 getifaddrs 函数
       int result = original_getifaddrs(ifap);
       if (result != 0) {
           return result;
       }
   
       // 打印调试信息
       printf("getifaddrs called\n");
   
       // 遍历接口列表并修改 MAC 地址
       struct ifaddrs *ifa = *ifap;
       while (ifa) {
           if (ifa->ifa_addr && ifa->ifa_addr->sa_family == AF_PACKET) {
               struct sockaddr_ll *s = (struct sockaddr_ll *)ifa->ifa_addr;
               // 修改 MAC 地址
               s->sll_addr[0] = 0x02;
               s->sll_addr[1] = 0x42;
               s->sll_addr[2] = 0xac;
               s->sll_addr[3] = 0x11;
               s->sll_addr[4] = 0x00;
               s->sll_addr[5] = 0x02;
               // 打印调试信息
               printf("Modified MAC address for interface %s\n", ifa->ifa_name);
           }
           ifa = ifa->ifa_next;
       }
   
       return result;
   }
   ```

2. **编译共享库**：

   ```bash
   gcc -shared -fPIC -o libspoof_mac.so spoof_mac.c -ldl
   ```

3. **使用 `LD_PRELOAD` 运行目标程序**：

   ```bash
   LD_PRELOAD=./libspoof_mac.so ./target_program
   ```

如果出现mac地址欺骗未成功，可以尝试劫持其他相关系统调用，例如 `ioctl`

`SIOCGIFHWADDR` 是一个特定的 `ioctl` 请求码，用于获取网络接口的硬件地址（MAC 地址）。`ioctl` 是一个通用的输入/输出控制接口，可以对设备文件进行各种操作，而 `SIOCGIFHWADDR` 则专门用于网络设备。

### `SIOCGIFHWADDR` 的使用

当一个程序想要获取某个网络接口的 MAC 地址时，会使用 `ioctl` 系统调用并传递 `SIOCGIFHWADDR` 请求码。这个请求码指示内核返回指定网络接口的硬件地址。

### 请求的结构

在使用 `SIOCGIFHWADDR` 请求时，通常需要一个 `struct ifreq` 结构体作为参数。该结构体定义在 `<net/if.h>` 头文件中，主要包含网络接口的名称和相关信息。

```c
#define _GNU_SOURCE
#include <dlfcn.h>
#include <sys/ioctl.h>
#include <net/if.h>
#include <netpacket/packet.h>
#include <string.h>
#include <stdio.h>
#include <stdarg.h>

// 定义劫持的 ioctl 函数
int ioctl(int fd, unsigned long request, ...) {
    static int (*original_ioctl)(int, unsigned long, ...);
    if (!original_ioctl) {
        original_ioctl = dlsym(RTLD_NEXT, "ioctl");
    }

    va_list args;
    va_start(args, request);

    int ret;

    if (request == SIOCGIFHWADDR) {
        struct ifreq *ifr = va_arg(args, struct ifreq *);
        ret = original_ioctl(fd, request, ifr);
        if (ret == 0) {
            // 打印调试信息
            printf("Intercepted SIOCGIFHWADDR for interface: %s\n", ifr->ifr_name);
            // 修改 MAC 地址
            ifr->ifr_hwaddr.sa_data[0] = 0x02;
            ifr->ifr_hwaddr.sa_data[1] = 0x42;
            ifr->ifr_hwaddr.sa_data[2] = 0xac;
            ifr->ifr_hwaddr.sa_data[3] = 0x11;
            ifr->ifr_hwaddr.sa_data[4] = 0x00;
            ifr->ifr_hwaddr.sa_data[5] = 0x02;
            // 打印调试信息
            printf("Modified MAC address for interface %s\n", ifr->ifr_name);
        }
    } else {
        ret = original_ioctl(fd, request, va_arg(args, void *));
    }

    va_end(args);
    return ret;
}
```



