## 🐾基于Git的Palworld回合制离线联机

### 简介

本仓库用于幻兽帕鲁（Palworld）共享世界存档同步，实现：

- 在线模式：游戏内邀请码联机
- 离线模式：Git 接力游玩（pull → play → push）

目标：可接力游玩、存档回滚、不炸存档、不串角色

简介：

- 游玩前先pull对方最新的存档，游玩后再push。
- 两人不能同时游玩（存档无法merge），因此玩家A玩的时候要在Issue发布一个checkin，游玩完毕后resolve issue。Issue的意思是玩家A正在离线玩，玩家B看见则知道自己无法同时离线玩，而是要与A进行邀请码联机。

### 目录结构

位置：

```
C:\Users\{username}\AppData\Local\Pal\Saved\SaveGames\76561198807407615\<存档ID>
```

存档ID：C509AE174727EBFF56FEF3B3B6467391

#### 本地存档（主机侧）

```
存档ID/
  ├── backup           → 存档
  ├── Players
  |     ├──00000000000000000000000000000001.sav   → 默认玩家，即房主
  |     ├──2E0382F9000000000000000000000000.sav   → 玩家B（注意：玩家B存档保存在主机侧本地存档）
  ├── Level.sav        → 世界核心数据（建筑/地图/生物）
  ├── LevelMeta.sav    → 本地数据（最近连接的server、连接记录等）
  ├── LocalData.sav    → 世界索引与元信息
  ├── WorldOption.sav  → 世界规则设置
```

#### 本地存档（玩家侧）

位置相同。

```
存档ID/
  ├── backup           → 存档
  ├── LocalData.sav    → 世界索引与元信息
```



### 使用流程

#### 主机侧

**游玩前**

将本repository全部文件覆盖**本地存档**位置。

```
git fetch origin
git reset --hard origin/main
```

在Issues发布 [Checkin] 时间，表示正在游玩，其他玩家不能同时离线游玩。

**游玩结束后**

上传全部文件至本repository。

```
git add .
git commit -m "26MMDD存档"
git push origin main
```

在Issues resolve之前的issue，表示结束游玩，其他玩家可以离线游玩。



#### 玩家侧

因主机侧不在线，玩家侧需要在本地模拟主机侧开服务器，再加入。

**游玩前**

（1）将本repository文件覆盖**服务器存档**位置。

```
Steam\steamapps\common\PalServer\Pal\Saved\SaveGames\0\<存档ID>
```

存档ID：4E1572464FE2FFA06CCFDA86E647CE0D

（2）在Issues发布 [Checkin] 时间，表示正在游玩，其他玩家不能同时离线游玩。

（3）运行**Palworld Dedicated Server**。

```
Setting breakpad minidump AppID = 2394010
Game version is v0.7.3.90464
Running Palworld dedicated server on :8211
```

（4）进入Palworld，选择加入多人游戏（专用服务器），连接127.0.0.1:8211

**游玩结束后**

（1）上传全部文件至本repository。

（2）注意：开服务器生成的存档ID和主机侧的存档ID不同，因此**玩家侧**的存档会保存到

```
# 目录1
C:\Users\{username}\AppData\Local\Pal\Saved\SaveGames\76561198807407615\{主机侧本地存档ID}
# 目录2
C:\Users\{username}\AppData\Local\Pal\Saved\SaveGames\76561198807407615\{本地服务器ID}
```

因此需要将目录2全部文件覆盖目录1全部文件。

（3）在Issues resolve之前的issue，表示结束游玩，其他玩家可以离线游玩。



### 规则

|      | 规则                               | 原因                                     |
| ---- | ---------------------------------- | ---------------------------------------- |
| R1   | 要先pull再玩                       | 否则会覆盖别人的存档                     |
| R2   | 不能同时离线玩                     | 否则会产生冲突，无法合并                 |
| R3   | 玩之前要发Issue，玩完resolve Issue | 通知别人自己正在玩，防止两人同时离线游玩 |

