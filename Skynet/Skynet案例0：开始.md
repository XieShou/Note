# Skynet案例0：开始

1. 安装skynet，安装必要的库，进入目录执行`make linux`。

2. 执行如下命令，可以打开案例中的登陆服
   
   ```
   liaozuojia@DESKTOP-OUVU5AG:/mnt/d/UbuntuProjects/skynet$ ./skynet examples/config.login
   [:00000002] LAUNCH snlua bootstrap
   [:00000003] LAUNCH snlua launcher
   [:00000004] LAUNCH snlua cdummy
   [:00000005] LAUNCH harbor 0 4
   [:00000006] LAUNCH snlua datacenterd
   [:00000007] LAUNCH snlua service_mgr
   [:00000008] LAUNCH snlua main
   [:00000009] LAUNCH snlua logind
   [:0000000a] LAUNCH snlua logind
   [:0000000b] LAUNCH snlua logind
   [:0000000c] LAUNCH snlua logind
   [:0000000d] LAUNCH snlua logind
   [:0000000e] LAUNCH snlua logind
   [:0000000f] LAUNCH snlua logind
   [:00000010] LAUNCH snlua logind
   [:00000011] LAUNCH snlua logind
   [:00000009] login server listen at : 127.0.0.1 8001
   [:00000012] LAUNCH snlua gated 9
   [:00000012] Listen on 0.0.0.0:8888
   [:00000002] KILL self
   ```

上面的命令执行的第一个脚本是`examples/login/main.lua`。

```lua
local skynet = require "skynet"

skynet.start(function()
    local loginserver = skynet.newservice("logind") -- 开登陆服务器
    local gate = skynet.newservice("gated", loginserver) -- 开游戏服务器

    skynet.call(gate, "lua", "open" , {
        port = 8888,
        maxclient = 64,
        servername = "sample",
    })
end)
```

由于游戏服务器需要向登陆服务器注册游戏节点，所以需要登陆服务器在前先开启。
