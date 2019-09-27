---
title: 饥荒联机版Linux专用服务器快读搭建向导
date: 2019-09-27 09:43:31
tags:
---

> 原文章地址：https://forums.kleientertainment.com/forums/topic/64441-dedicated-server-quick-setup-guide-linux/
>
> 本文章为原文章的翻译，并添加了部分额外内容。

这篇指南将会帮助你快速在Ubuntu上搭建属于你的饥荒联机版专用服务器。

# 一、安装依赖

```bash
对于64位的系统需要执行：
sudo apt install libstdc++6:i386 libgcc1:i386 libcurl4-gnutls-dev:i386 lib32gcc1
对于32位的系统需要执行：
sudo apt install libstdc++6 libgcc1 libcurl4-gnutls-dev
```

# 二、安装Steamcmd

你可以依照以下网址的指引下载并安装Steamcmd

https://developer.valvesoftware.com/wiki/SteamCMD#Linux

这篇指南会默认在你当前所在目录安装Steamcmd，你可以跳过里面有关创建用户的步骤。

或者是直接执行以下命令

```bash
mkdir -p ~/steamcmd/
cd ~/steamcmd/
wget "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz"
tar -xvzf steamcmd_linux.tar.gz
```

# 三、配置并下载服务器设置

1、从Steam中打开Don't Starve Together，点击 **开始游戏** 。

2、点击 **账户信息** 。

3、在 **账号信息** 界面点击上方的游戏，然后点击 **Don’t Starve Together Servers** 按钮。

4、如果你还没有配置过服务器，则点击下方的 **添加新服务器** 。

5、点击 **配置服务器** 按钮，你可以修改一些服务器相关的配置。然后点击 **下载设置** 。如果无法下载，右键点击 **复制网页URL** 并在浏览器中进行操作。

6、在服务器的用户根目录创建 **.klei/DoNotStarveTogether** 路径， 并将下载的 **MyDediServer** 压缩包解压到该文件夹中。此时目录结构应该为**.klei/DoNotStarveTogether/MyDediServer**，该文件夹下有cluster.ini、cluster_token.txt、Master文件夹和Caves文件夹。

# 四、创建安装和启动服务器脚本

下载该脚本

https://accounts.klei.com/assets/gamesetup/linux/run_dedicated_servers.sh

或新建文件

```bash
sudo vi run_dedicated_servers.sh
```

并复制以下内容

```sh
#!/bin/bash

steamcmd_dir="$HOME/steamcmd" # Steamcmdd的安装路径
install_dir="$HOME/dontstarvetogether_dedicated_server" # 饥荒联机版的安装路径
cluster_name="MyDediServer" # 存档的名称，即在.klei中创建的文件夹名称
dontstarve_dir="$HOME/.klei/DoNotStarveTogether" # 存档存放的位置

function fail()
{
	echo Error: "$@" >&2
	exit 1
}

function check_for_file()
{
	if [ ! -e "$1" ]; then
		fail "Missing file: $1"
	fi
}

cd "$steamcmd_dir" || fail "Missing $steamcmd_dir directory!"

check_for_file "steamcmd.sh"
check_for_file "$dontstarve_dir/$cluster_name/cluster.ini"
check_for_file "$dontstarve_dir/$cluster_name/cluster_token.txt"
check_for_file "$dontstarve_dir/$cluster_name/Master/server.ini"
check_for_file "$dontstarve_dir/$cluster_name/Caves/server.ini"

./steamcmd.sh +force_install_dir "$install_dir" +login anonymous +app_update 343050 validate +quit

check_for_file "$install_dir/bin"

cd "$install_dir/bin" || fail

run_shared=(./dontstarve_dedicated_server_nullrenderer)
run_shared+=(-console)
run_shared+=(-cluster "$cluster_name")
run_shared+=(-monitor_parent_process $$)

"${run_shared[@]}" -shard Caves  | sed 's/^/Caves:  /' &
"${run_shared[@]}" -shard Master | sed 's/^/Master: /'
```

为下载或新建的 **run_dedicated_servers.sh** 文件添加当前用户的运行权限

```sh
sudo chmod u+x run_dedicated_servers.sh
```

# 五、运行脚本自动安装并启动服务器

```sh
./run_dedicated_servers.sh
```

# 六、如何下载并启用mod

在执行完以上步骤后，你得到了一个纯净的饥荒联机版专用服务器，你可能还想为这个服务器添加一些mod。

## 1、下载mod

服务器的mod下载设置文件 **dedicated_server_mods_setup.lua** 在安装路径的mods文件夹中，本例的路径为 **~/dontstarvetogether_dedicated_server/mods** ，添加mod的有两种方式

### 通过添加创意工坊mod的ID进行单个添加

如 **Combined Status** 状态显示 的Steam创意工坊地址为 **https://steamcommunity.com/sharedfiles/filedetails/?id=376333686**，则该mod的ID为 **376333686**。

在 **dedicated_server_mods_setup.lua** 文件尾部添加 **ServerModSetup("350811795")** 即可下载该mod。

### 通过添加创意工坊mod合集的ID进行批量添加

你也可以通过创建合集来批量下载mod

如  **Kantai Collection** 舰娘合集 的创意工会合集地址为 **https://steamcommunity.com/sharedfiles/filedetails/?id=728810004**。

在 **dedicated_server_mods_setup.lua** 文件尾部添加 **ServerModCollectionSetup("728810004")** 即可下载该合计内的所有mod。

**NOTE：**该方法只对合集内包含的mod有效，对于软连接的其他mod合集无效



## 2、启用和设置mod

对于如何设置和启动mod，建议在本地电脑上直接搭建一个只含有且调整完mod选项服务器mod的本地服务器，创建完后即可关闭。

在 **我的电脑\文档\klei\DoNotStarveTogether\[数字]\Cluster_1\Master** 中复制 **modoverrides.lua** 文件到服务器对于的 **Master** 和 **Cave** 存档文件中，就可以完成服务器mod的启用和配置。其中Cluster_1是你创建的本地服务器存档。

# 七、如何将地表服务器和洞穴服务器运行在两台服务器上

饥荒联机版专用服务器对服务器性能要求不高，但如果在一台服务器上同时运行地表服务器和洞穴服务器难免还是会增加服务器的负担和延迟，对于一些只有1核1G或2G内存的云服务器则完全不能同时运行。

所以将地表服务器和洞穴服务器分别运行在两台服务器上是一个比较好的选择。

只需修改存档下的cluster.ini文件

```
[SHARD]
shard_enabled = true
bind_ip = 0.0.0.0
master_ip = 地表服务器的IP
master_port = 地表服务器的端口，一般为10900或10899
cluster_key = 随意设定
```

同时修改  **run_dedicated_servers.sh** 文件最下面的

```
"${run_shared[@]}" -shard Caves  | sed 's/^/Caves:  /' &
"${run_shared[@]}" -shard Master | sed 's/^/Master: /'
```

如果是地表服务器：

```
"${run_shared[@]}" -shard Master | sed 's/^/Master: /'
```

如果是洞穴服务器：
```
"${run_shared[@]}" -shard Caves  | sed 's/^/Caves:  /'
```

# 八、Q&A

## 1、服务器的存档位置在哪？

 **~/.klei/DoNotStarveTogether/MyDediServer** ，其中 **.klei** 是个隐藏文件夹，ls命令无法显示，直接 **cd ~/.klei** 即可。

## 2、没有 **.klei** 和 **DoNotStarveTogether** 文件夹

手动在当前用户根目录下创建

```sh
mkdir -p .klei/DoNotStarveTogether
```

## 3、怎么设置世界属性？

同启用和设置mod，在本地电脑创建好世界后，将存档内的 **leveldataoverride.lua** 和 **worldgenoverride.lua** 复制到服务器对应的存档位置中。