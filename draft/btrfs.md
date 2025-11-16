### 分区

列出分区的情况：

```bash
lsblk -pf
```

进行分区：
```
cfdisk /dev/分区名
```
这里需要创建 EFI 分区，以及 linux 文件分区，如果有多块磁盘，都需要创建
### 格式化 EFI 分区

```bash
mkfs.fat -F32 /dev/分区名
```

### 单设备文件系统 

在分区 `/dev/分区名` 上创建一个 Btrfs 文件系统：

```bash
mkfs.btrfs -L mybtrfs /dev/分区名
```

> [!important]
> 这里 `mybtrfs` 是自定义标签，**这里的分区和 EFI 分区不一样！**

### 多设备文件系统

```bash
mkfs.btrfs -L mybtrfs -m raid1 -d raid1 /dev/分区1 /dev/分区2
```

+ `-m raid 1`：元数据 (Metadata) 镜像。元数据是文件系统的“地图”，记录着哪个文件存放在哪里、目录结构、权限等信息。元数据损坏会导致整个文件系统崩溃。raid 1 意味着元数据会在两块硬盘上各存一份。
+ `-d raid 1`：数据 (Data) 镜像。这是你实际存储的文件内容（文档、视频、程序等）。raid 1 意味着你的每个文件也会在两块硬盘上各存一份。

如果不指定 `-m` 和 `-d` 参数，默认元数据为 `raid1`，数据为 `single`

如果不想备份两份，而要追求性能：

```
mkfs.btrfs -L mybtrfs -m raid0 -d raid0 /dev/分区1 /dev/分区2
```
### 挂载顶层 Btrfs 卷

```bash
mount /dev/分区名 /mnt
```

> [! Important]
> 如果有多个分区，例如 `/dev/sda2`, `/dev/sdb1`，也仅需挂载任意一个即可

> [!warning] 
> 这里不是挂载 efi 分区！

### 创建子卷

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@swap
```


> [!tip]
> `@var` (可变数据目录)
> 
> 用途：这个子卷将作为 `/var` 目录。这个目录存放着日志、系统缓存（比如 pacman 的软件包缓存）、数据库等频繁变动的文件。
> 
> 为什么独立？：这些文件变化太快，而且大多是临时或日志数据。如果不把它分离出去，那么每次对系统 (@) 创建快照时，都会包含进大量无用的、频繁变动的 @var 数据，导致快照体积急剧增大，失去意义。将它分离，可以保持系统快照的纯净和高效。
> 
> `@snapshots` (快照存放目录)
> 
> 用途：这个子卷专门用来挂载，以便存放您为 @ 创建的所有快照。
> 
> 为什么独立？：为了避免“嵌套”。如果您把快照存放在 @ 子卷内部（比如 `/.snapshots `），那么当您下一次对 @ 创建快照时，就会把之前的快照也包含进去，形成“快照套娃”，这会造成管理上的混乱和空间计算的困难。把它独立出来，您的快照管理会变得非常清晰、整洁。

### 对 swap 子卷禁止 CoW

Linux 内核要求 Swap 文件必须具有静态、固定的物理地址，以便能快速、直接地进行读写。而 Btrfs 的写时复制 (CoW) 机制，其本质就是“写入时数据会改变物理地址”。这两者是根本上冲突的。如果在启用了 CoW 的文件或子卷上创建 Swap 文件，系统将无法激活它，或者会导致严重的性能问题和不稳定。因此，为 Swap 文件所在的整个子卷禁用 CoW 是最干净、最可靠的做法。

```bash
chattr +C /mnt/@swap
```

### 卸载顶层卷并重新挂载子卷

```bash
umount /mnt
# 挂载根目录子卷，如果有两个磁盘/dev/sda2和/dev/sdb1，这里分区名可以填写sda2
# sda1为efi分区!
mount -o noatime,compress=zstd,subvol=@ /dev/分区名 /mnt

mkdir -p /mnt/home
# 分区名通常为sda2
mount -o noatime,compress=zstd,subvol=@home /dev/分区名 /mnt/home

mkdir -p /mnt/boot
# 分区名通常为sda1!
mount /dev/分区名 /mnt/boot

mkdir -p /mnt/swap
# 分区名同查维sda2
mount -o noatime,nodatacow,subvol=@swap /dev/分区名 /mnt/swap

mkdir -p /mnt/var
# 分区名同查维sda2
mount -o noatime,compress=zstd,subvol=@var /dev/分区名 /mnt/var

mkdir -p /mnt/.snapshots
# 分区名同查维sda2
mount -o noatime,compress=zstd,subvol=@snapshots /dev/分区名 /mnt/.snapshots
```


> [!Important]
> 后续指定目录为 `/mnt` !

### swap 文件设置

创建 swap 文件：
```bash
btrfs fi mkswapfile /mnt/swap/swapfile --size 8G
```
如果需要休眠，那么 swapfile 文件的大小应约为内存的 1.1 倍

激活 swap 文件：
```
swapon /mnt/swap/swapfile
```

### 快照命令管理

创建一个以当前时间命名的只读快照：

```bash
sudo btrfs subvolume snapshot -r / /. snapshots/root_$(date +%F_%H-%M-%S)
```

列出快照：快照就是子卷，所以用列出子卷的命令即可。

```bash
sudo btrfs subvolume list /
```

删除快照：

```bash
sudo btrfs subvolume delete /.snapshots/root_2025-09-04_07-30-00
```


> [!Tips] 
> 推荐使用 timeshift 自动管理快照
