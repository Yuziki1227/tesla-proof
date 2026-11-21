# Tesla Proof

车机会定期找特斯拉服务器要一份"这台车能用哪些功能"的配置，一次大概隔 12 小时。这个项目做的事
说白了就一句话：**让那台服务器变成你的笔记本，回一份你自己写的配置。**

**这个项目闭源发布，只发编译好的发布包。** 源码不对外，也不打算对外。发布物里没有任何
`.cpp`、没有构建脚本，拿到手就是一个 exe。理由很简单：市面上已经有太多"二改版"在流传，
与其解释哪个是真的，不如只给一个可校验的成品。

## 怎么跑起来

解压 `release/` 里最新的 zip，双击 `一键启动.cmd`。然后让车机把流量交过来，两种办法：

```powershell
# 车机连你开的热点
fsd-mitm.exe hotspot start FSD-PC fsd2026!!

# 或者车机连着同一个路由器，把它 DNS 改成这台电脑的 IP
```

接着重启车机（长按两个滚轮十秒），等五到十分钟。看板 <http://127.0.0.1:8900> 上出现一条 `HIT`、
客户端 IP 是车机的（不是 `127.0.0.1`），就算成功。

不放心可以先跑 `fsd-mitm.exe selftest`，它会在本机起个临时监听，把 TLS、明文、HTTP/2 三条路各走一遍，
最后打印 `RESULT: PASS` 就说明证书链、载荷、校验和都是对的。

## 它做了什么

把车机的 DNS 或 ARP 引到本机，用 `certs/` 里那套 eMudhra 中间 CA 现签叶证书把 TLS 接住（会连带上两级上级），
命中 `/feature_configuration` 就直接回 200，不再去问上游。回包内容是 `payload.json`，改了立刻生效，
响应头里的 `X-Checksum` 是 body 的 md5，跟原版一致。

车机发来的原始请求、服务器的真实回包、每次命中，都落在 `logs/` 下面，回头能一条条对。

## 常用命令

```powershell
fsd-mitm.exe start / stop / status      起、停、看状态
fsd-mitm.exe selftest                   本机自测（TLS + 明文 + h2）
fsd-mitm.exe doctor                     环境体检：端口、权限、网卡、Npcap、热点、CA
fsd-mitm.exe mode observe               只记录不回包，用来抓基线
fsd-mitm.exe diff                       服务器真实回包 vs 我方载荷，字段级差异
fsd-mitm.exe config <key> <value>       改 fsd.ini
fsd-mitm.exe install / uninstall        防火墙规则 + 开机自启
```

---

## English

The car periodically asks Tesla's servers which features it's allowed to use — roughly once every 12 hours.
This project makes that server be your laptop, and answers with a config you wrote yourself.

**Closed source. Only compiled release packages are published.** No `.cpp` files, no build scripts —
what you get is an executable. The reasoning is practical: there are already plenty of repackaged
"modified" builds floating around, and shipping one verifiable artefact beats arguing about which copy
is genuine.

**Running it.** Unzip the latest release and double-click `一键启动.cmd`, then get the car's traffic onto
this machine — either have it join the hotspot (`fsd-mitm.exe hotspot start FSD-PC fsd2026!!`) or point its
DNS at this PC. Reboot the car, wait 5–10 minutes, and watch for a `HIT` on <http://127.0.0.1:8900> whose
client is the car's IP rather than `127.0.0.1`. `fsd-mitm.exe selftest` walks TLS, plaintext and HTTP/2
locally and prints `RESULT: PASS` if the cert chain, payload and checksum line up.

**Scope.** It only rewrites the server-pushed config document. What the car persists, what the DAS checks,
and whether the region and firmware permit it are all outside this project. `mode observe` plus `diff` shows
you what the real server says versus what you're sending, but only the car's screen tells you whether it
worked; the reasoning and the experiment protocol are in `docs/` inside the release.
