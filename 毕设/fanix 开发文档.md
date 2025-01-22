## 1.启动虚拟机

```bash
passwd `设置root密码
ping baidu.com `连接百度查看是否联网
```

## 2.打开cmd控制台：主机连接虚拟机

虚拟机：

```bash
ip add
ifconfig 
`这两个都是查看网络ip`
```

cmd:

```bash
ssh root@192.168......
```

## 3.新建分区并格式化、挂载

- `lsblk`

这会列出所有块设备的基本信息，包括设备名称、主/次设备号、大小、类型（磁盘或分区）、挂载点等。

> 可以看到有一个8G的磁盘sda。

- `fdisk /dev/sda`

  `fdisk` 是一个在 Linux 系统中用于分区管理的工具。作为一个交互式工具，用户可以通过它对磁盘进行分区操作。

  进入 `fdisk` 交互模式后，可以进行以下操作：

  - **`p`**：打印当前分区表。
  - **`n`**：创建新分区。
  - **`d`**：删除分区。
  - **`t`**：更改分区类型。
  - **`w`**：保存更改并退出。
  - **`q`**：退出而不保存更改。

> 这里我们输入n，创建新分区，并在接下来选择默认操作。

> 完成后，输入 w 退出。

这时再用`lsblk` 查看，发现sda下有一个sda1的分区。

- 格式化 `mkfs`

  `mkfs.ext4`+ `/dev/sda1` : 将磁盘格式化为ext4格式 

- 挂载：`mount`

  ```bash
  mount /dev/sda1 /mnt `将sda1挂载到mnt
  ```

## 4.更新软件包(镜像)

更新镜像

```bash
pacman -Sy `更新本地的软件包数据库`
pacman -Ss mirrorlist  `在远程仓库中搜索与“mirrorlist”相关的软件包`
pacman -S pacman-mirrorlist `安装或重新安装 pacman-mirrorlist 包。``安装镜像源列表文件`
```

这里收到一个信息：warning: /etc/pacman.d/mirrorlist installed as /etc/pacman.d/mirrorlist.pacnew

由此可知，mirrorlist 的文件位置。

```bash
cd /etc/pacman.d
cat mirrorlist.pacnew | grep China -A 25
`这会显示miroorlist.pacnew 文件中 , 包含“China”的行及其之后的 25 行。`
cat mirrorlist.pacnew | grep China -A 24 > mirrorlist
`这个命令的作用是从 /etc/pacman.d/mirrorlist.pacnew 文件中提取包含“China”的镜像源及其之后的24行内容，并将这些内容保存到一个名为 mirrorlist 的文件中。
```

>在 Arch Linux 中，`mirrorlist.pacnew` 是一个由 `pacman` 在更新 `pacman-mirrorlist` 包时生成的文件。它包含了最新的镜像源列表，但不会自动覆盖现有的 `/etc/pacman.d/mirrorlist` 文件。相反，它会被保存为 `/etc/pacman.d/mirrorlist.pacnew`，供用户手动合并或替换。

这样操作完成后，mirrorlist 文件里是各种各样的镜像源，前面都加了#号注释掉了。

接下来

```bash
vim mirrorlist
```

在里面随便取消一行的注释，作为我们的软件镜像源。

```bash
pacstrap -i /mnt base base-devel linux linux-firmware
`是一个用于安装 Arch Linux 基础系统的命令。将后面那4个软件包安装到目标挂载点 /mnt，并允许你交互式地确认每个软件包的安装。`
```

> **作用**：这个命令的作用是将 `base`、`base-devel`、`linux` 和 `linux-firmware` 这些软件包安装到 `/mnt` 挂载点上。这些软件包构成了一个最小化的 Arch Linux 系统，安装完成后，你可以通过 `arch-chroot` 进入新系统并进行进一步的配置。
>
> **命令**：`pacstrap` 会开始从远程仓库下载并安装这些软件包及其依赖项。
>
> 它将软件包安装到指定的挂载点（通常是新系统的根目录）。它会自动处理依赖关系，并将必要的文件安装到目标目录。
>
> **位置**：/mnt
>
> **后缀**：
>
> 1. **`base`**
>    `base` 是 Arch Linux 的基础软件包组，包含了运行一个最小化 Arch Linux 系统所必需的软件包。
> 2. **`base-devel`**
>    `base-devel` 是一个包含开发工具的软件包组，例如 `make`、`gcc`、`git` 等。如果你需要在新系统中进行软件开发或编译，这个组是很有用的。
> 3. **`linux`**
>    `linux` 是 Linux 内核的软件包，它是系统的核心组件，负责管理硬件资源和提供系统服务。
> 4. **`linux-firmware`**
>    `linux-firmware` 包含了各种硬件设备所需的固件文件。这些固件文件对于某些硬件设备的正常工作是必需的。

## 5.进入新系统并初始化

```bash
genfstab -U -p /mnt > /mnt/etc/fstab `生成文件系统表`
arch-chroot /mnt `进入新系统`
```

`arch-chroot` 是 Arch Linux 提供的一个增强版的 `chroot` 命令，用于在安装、修复或维护 Arch Linux 系统时切换到一个指定的根目录环境。它会自动挂载必要的文件系统（如 `/proc`、`/sys` 和 `/dev`），从而简化系统管理。



> 执行步骤
>
> 1. **确保分区已挂载**
>    在运行 `pacstrap` 之前，你需要确保目标分区已经挂载到 `/mnt`。例如：
>
>    ```bash
>    mount /mnt
>    ```
>
> 2. **运行 `pacstrap`**
>    执行以下命令安装基础软件包：
>
>    ```bash
>    pacstrap -i /mnt base base-devel linux linux-firmware
>    ```
>
>    这时，`pacstrap` 会开始从远程仓库下载并安装这些软件包及其依赖项。
>
> 3. **交互式确认**
>    由于使用了 `-i` 选项，`pacstrap` 会提示你确认每个软件包的安装。你可以选择安装或跳过某些软件包。
>
> 4. **生成 `fstab` 文件**
>    安装完成后，你需要生成 `/mnt/etc/fstab` 文件，以便新系统能够正确挂载分区：
>
>    ```bash
>    genfstab -U -p /mnt > /mnt/etc/fstab `生成文件系统表`
>    ```
>
> 5. **进入新系统**
>    使用 `arch-chroot` 进入新系统，进行进一步的配置：
>
>    ```bash
>    arch-chroot /mnt `进入新系统`
>    ```
>
> 6. **配置新系统**
>    在新系统中，你可以设置时区、语言、网络配置等，并安装其他需要的软件包。

**选择文字编码**

```bash
pacman -S vim `安装vim`
vim /etc/locale.gen  `编辑locale.gen 选择文字编码`
```

vim中找到zh_CN

```vim
:/zh_CN `找到后，把CN前缀的注释全部取消
:/en_US `找到后，把US前缀的utf8 注释也取消
```

```bash
locale-gen
```

在文件 `/etc/locale.conf` 中写入 `LANG=en_US.UTF-8` 保存

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
rm -f /etc/localtime   `删除原 UTC 时区
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime   `设置计算机系统时区为上海`
hwclock --systohc --localtime `设置硬件时间为本地时间`
echo fancy > /etc/hostname  `将主机名配置成 fancy，这个名称可以自行更改`
passwd `设置新系统的密码`
```

```bash
pacman -S ntfs-3g `安装 ntfs 文件系统，以便访问 Windows 磁盘
pacman -S dialog `安装终端对话框
```



**配置网络**

```bash
pacman -S networkmanager  `安装使用 NetworkManager 来管理网络`
```

```bash
systemctl enable NetworkManager `自启动 NetworkManager
`NetworkManager 会帮你自动把网络配置好`
pacman -S openssh `安装ssh'
systemctl enable sshd `设置开机启动ssh服务
```

新建用户名为 fancy，用户名可以自行修改，并且加入root用户

```bash
useradd -G root -m fancy
```

修改用户fancy的密码

```bash
passwd fancy
```

**配置sudo**

```bash
vim /etc/sudoers `如果没有sudoers，自己手动安装`
```

在里面通过`:/root` 找到 `root ALL=(ALL;ALL) ALL`

在其后添加 `fancy ALL=(ALL;ALL) ALL`



**配置引导**

```bash
pacman -S grub `安装引导程序`
grub-install --target=i386-pc /dev/sda `安装 BIOS 引导`
grub-mkconfig -o /boot/grub/grub.cfg  `grub.cfg 生成该文件`
```



安装完毕！

可以重启查看。`exit` `reboot`

---













## 关于locale

`locale-gen` 是一个用于生成和配置系统语言环境（locale）的工具，通常在 Linux 系统中使用，尤其是在安装或配置新系统时。它会根据 `/etc/locale.gen` 文件中的配置生成相应的语言环境文件，从而确保系统支持多种语言和字符集。

以下是关于 `locale-gen` 的详细解释和使用方法：

**1. 什么是 Locale？**

Locale 是一个系统设置，用于定义用户界面的语言、字符编码、日期格式、数字格式等。它确保系统能够根据用户的语言和地区偏好正确显示内容。

例如：

- `en_US.UTF-8` 表示美国英语，使用 UTF-8 编码。
- `zh_CN.UTF-8` 表示简体中文，使用 UTF-8 编码。

**2. `/etc/locale.gen` 文件**

`locale-gen` 的配置文件是 `/etc/locale.gen`。它包含了一系列支持的语言环境，每行定义了一个 locale，格式如下：

```
<locale_name> <encoding>
```

例如：

复制

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

默认情况下，许多 locale 是被注释掉的（以 `#` 开头）。要启用某个 locale，需要取消注释对应的行。

**3. 使用 `locale-gen`**

`locale-gen` 的主要作用是根据 `/etc/locale.gen` 文件中的配置生成 locale 文件。以下是常见操作：

**查看当前配置**

查看 `/etc/locale.gen` 文件的内容：

bash复制

```bash
cat /etc/locale.gen
```

你会看到类似以下的内容：

复制

```
# en_US.UTF-8 UTF-8
# zh_CN.UTF-8 UTF-8
```

**启用 Locale**

编辑 `/etc/locale.gen` 文件，取消注释需要启用的 locale：

bash复制

```bash
sudo nano /etc/locale.gen
```

例如，启用 `en_US.UTF-8` 和 `zh_CN.UTF-8`：

复制

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

**生成 Locale**

运行以下命令生成 locale 文件：

bash复制

```bash
sudo locale-gen
```

这个命令会根据 `/etc/locale.gen` 文件中的配置生成对应的 locale 文件，并将它们保存到 `/usr/lib/locale/` 目录中。

**查看已生成的 Locale**

运行以下命令查看当前系统已生成的 locale：

bash复制

```bash
locale -a
```

你会看到类似以下的输出：

复制

```
en_US.UTF-8
zh_CN.UTF-8
```

**4. 设置默认 Locale**

除了生成 locale，还需要设置系统的默认语言环境。这通常通过修改 `/etc/environment` 或 `/etc/default/locale` 文件来完成。

例如，设置默认语言为 `en_US.UTF-8`：

bash复制

```bash
echo "LANG=en_US.UTF-8" | sudo tee /etc/environment
```

或者编辑 `/etc/default/locale` 文件：

bash复制

```bash
sudo nano /etc/default/locale
```

添加以下内容：

```
LANG=en_US.UTF-8
```

**5. 常见问题**

**问题 1：为什么需要启用多个 Locale？**

启用多个 locale 可以让系统支持多种语言环境。例如，你可能希望系统默认使用英文，但同时支持中文输入和显示。

**问题 2：如何确定系统支持哪些 Locale？**

运行以下命令查看系统支持的所有 locale：

bash复制

```bash
locale -a
```

**问题 3：如何设置桌面环境的语言？**

某些桌面环境（如 GNOME、KDE）允许在设置中直接更改语言。这些设置通常会覆盖系统的默认 locale。

**6. 示例：配置多语言环境**

假设你想让系统支持英文和中文，并将默认语言设置为中文：

1. **编辑 `/etc/locale.gen` 文件**：

   bash复制

   ```bash
   sudo nano /etc/locale.gen
   ```

   启用以下两行：

   复制

   ```
   en_US.UTF-8 UTF-8
   zh_CN.UTF-8 UTF-8
   ```

2. **生成 locale**：

   bash复制

   ```bash
   sudo locale-gen
   ```

3. **设置默认语言**：

   bash复制

   ```bash
   echo "LANG=zh_CN.UTF-8" | sudo tee /etc/environment
   ```

4. **重启系统**： 为了使更改生效，建议重启系统：

   bash复制

   ```bash
   sudo reboot
   ```

**总结**

`locale-gen` 是一个用于生成和配置系统语言环境的工具。通过编辑 `/etc/locale.gen` 文件并运行 `locale-gen` 命令，你可以启用所需的语言环境，并通过设置环境变量来指定默认语言。这在安装新系统或配置多语言支持时非常有用。





## 关于挂载

关于挂载：

**挂载是什么？**

在 Linux 系统中，所有的文件和设备都被视为文件系统的一部分。挂载就是将一个存储设备（比如硬盘分区、USB 驱动器、光盘等）连接到文件系统中的某个目录，使其可以被访问的过程。简单来说，挂载就是让系统“看到”并使用某个存储设备。

**举个生活中的例子**

想象一下，你有一个书架（文件系统），书架上有很多书（文件和目录）。现在你又买了一个新的书架（存储设备，比如 USB 驱动器），但你希望将新书架上的书也放到原来的书架上。挂载的过程就像是把新书架上的书“搬”到原来的书架上，让你可以方便地找到和使用它们。

**挂载的步骤**

1. **插入设备**
   比如插入一个 USB 驱动器，或者连接一个外部硬盘。

2. **找到设备路径**
   使用命令 `lsblk` 或 `fdisk -l` 查看设备的路径。例如，USB 驱动器可能是 `/dev/sdb1`。

3. **创建挂载点**
   挂载点是一个目录，设备的内容会“挂”在这个目录下。你可以用以下命令创建一个挂载点：

   bash复制

   ```bash
   sudo mkdir /mnt/mydrive
   ```

   这里 `/mnt/mydrive` 就是挂载点。

4. **挂载设备**
   使用 `mount` 命令将设备挂载到挂载点：

   bash复制

   ```bash
   sudo mount /dev/sdb1 /mnt/mydrive
   ```

   这时，设备的内容就会出现在 `/mnt/mydrive` 目录下。

5. **访问设备内容**
   现在你可以通过挂载点访问设备的内容了：

   bash复制

   ```bash
   cd /mnt/mydrive
   ls
   ```

   你会看到 USB 驱动器中的文件。

6. **卸载设备**
   当你不再需要使用设备时，需要先“卸载”（umount）它，以确保数据安全：

   bash复制

   ```bash
   sudo umount /mnt/mydrive
   ```

**简单总结**

- **挂载**：将设备连接到文件系统中的某个目录，使其可以被访问。
- **挂载点**：设备内容被挂载的目录。
- **卸载**：将设备从文件系统中移除，确保数据安全。



## 关于pacman

`pacman` 是 Arch Linux 及其衍生发行版（如 Manjaro）的包管理工具。它是一个功能强大且设计简洁的命令行工具，用于管理软件包的安装、更新、查询和删除等操作。`pacman` 的名字来源于经典的街机游戏《Pac-Man》。

**Pacman 的主要功能**

1. **安装软件包**：从官方仓库或本地文件安装软件。
2. **更新软件包**：更新系统中已安装的软件包。
3. **查询软件包信息**：查看软件包的详细信息。
4. **删除软件包**：卸载不需要的软件。
5. **同步软件包数据库**：从远程仓库更新本地的软件包列表。

**常用命令**

**1. 更新系统**

更新所有已安装的软件包和软件包数据库：

bash复制

```bash
sudo pacman -Syu
```

- `-S`：同步软件包数据库。   sync
- `-y`：更新软件包数据库。   refresh
- `-u`：升级已安装的软件包。   update

**2. 安装软件包**

从官方仓库安装软件包：

bash复制

```bash
sudo pacman -S package_name
```

例如，安装 `vim` 编辑器：

bash复制

```bash
sudo pacman -S vim
```

**3. 查询软件包信息**

- 查看本地已安装的软件包：

  bash复制

  ```bash
  pacman -Q
  ```

- 查看软件包的详细信息：

  bash复制

  ```bash
  pacman -Qi package_name
  ```

- 查找仓库中可用的软件包：

  bash复制

  ```bash
  pacman -Ss package_name
  ```

**4. 删除软件包**

卸载软件包：

bash复制

```bash
sudo pacman -R package_name
```

如果需要同时删除依赖于该软件包的其他软件包，可以使用：

bash复制

```bash
sudo pacman -Rs package_name
```

**5. 清理缓存**

清理下载的软件包缓存：

bash复制

```bash
sudo pacman -Sc
```

清理所有缓存（谨慎使用）：

bash复制

```bash
sudo pacman -Scc
```

**6. 查看文件列表**

查看某个软件包安装的文件列表：

bash复制

```bash
pacman -Ql package_name
```

**7. 搜索本地已安装的软件包**

bash复制

```bash
pacman -Qs package_name
```

**8. 安装本地软件包**

安装本地下载的 `.pkg.tar.zst` 文件：

bash复制

```bash
sudo pacman -U /path/to/package.pkg.tar.zst
```

**Pacman 的特点**

1. **简单易用**：命令简洁，易于学习。
2. **强大的依赖管理**：自动处理软件包的依赖关系。
3. **安全**：所有软件包都经过签名验证，确保来源可靠。
4. **灵活性**：支持自定义仓库和本地编译安装。

**Arch Linux 的哲学**

Arch Linux 以“简单、现代、用户中心”为核心理念，`pacman` 作为其包管理工具，完美体现了这些理念。它提供了强大的功能，同时保持了简洁和高效。

**总结**

`pacman` 是 Arch Linux 的标志性工具之一，无论是新手还是资深用户，都能通过它轻松管理软件包。如果你正在使用 Arch Linux 或其衍生发行版，掌握 `pacman` 的基本命令将大大提高你的使用体验。

---



`-S`（sync）：同步软件包

`-S` 选项用于与软件包仓库（repositories）进行同步操作。它主要用于以下场景：

1. **安装软件包**：从远程仓库下载并安装软件包。

   bash复制

   ```bash
   sudo pacman -S package_name
   ```

   这会从配置的仓库中查找并安装指定的软件包，同时自动处理依赖关系。

2. **更新软件包数据库**：在某些情况下，`-S` 也可以用于更新本地的软件包数据库。不过，通常会结合 `-y` 选项一起使用。

`-y`（refresh）：更新软件包数据库

`-y` 选项的作用是刷新本地的软件包数据库，使其与远程仓库保持同步。具体来说：

- 它会从配置的仓库中下载最新的软件包元数据（如版本信息、依赖关系等）。
- 这是确保系统能够获取最新软件包信息的重要步骤。

`-Sy` 和 `-Syu` 的区别

1. **`-Sy`**：

   - **含义**：`-S` + `-y`，即更新软件包数据库（`-y`），但不升级软件包。

   - **用途**：通常用于手动更新软件包数据库，确保后续操作（如安装或查询）使用最新的信息。

   - **命令示例**：

     bash复制

     ```bash
     sudo pacman -Sy
     ```

2. **`-Syu`**：

   - **含义**：`-S` + `-y` + `-u`，即更新软件包数据库（`-y`），并升级所有已安装的软件包（`-u`）。

   - **用途**：这是最常用的系统更新命令，用于同步软件包数据库并升级系统中的所有软件包。

   - **命令示例**：

     bash复制

     ```bash
     sudo pacman -Syu
     ```

总结

- **`-S`**：用于同步操作，如安装软件包或更新软件包数据库。
- **`-y`**：专门用于更新软件包数据库，确保本地信息是最新的。
- **`-Sy`**：更新软件包数据库，但不升级软件包。
- **`-Syu`**：更新软件包数据库并升级所有已安装的软件包。