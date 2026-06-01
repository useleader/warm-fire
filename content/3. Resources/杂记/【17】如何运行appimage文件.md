---
publish: true
---

## A. 什么是 AppImage？

![在 Linux 中使用 AppImage](https://i0.wp.com/itsfoss.com/wp-content/uploads/2017/07/use-appimage-linux.png?resize=800%2C450&ssl=1)

多年来，我们为基于 Debian/Ubuntu 的 Linux 发行版提供[DEB 包](https://en.wikipedia.org/wiki/Deb_\(file_format\))，为基于 Fedora /SUSE 的 Linux 发行版[提供 RPM 。](https://en.wikipedia.org/wiki/Rpm_\(software\))

虽然这些软件包为各自的分发用户提供了一种安装软件的便捷方式，但对于应用程序开发人员来说并不是最方便的。开发人员必须为多个发行版创建多个包。这就是 AppImage 发挥作用的地方。

[AppImage](http://appimage.org/)是一种通用的软件包格式。通过将软件打包在 AppImage 中，开发人员只需提供一个文件即可“统管所有”。最终用户，即您，可以在大多数（如果不是全部）现代 Linux 发行版中使用它

### AppImage 不以传统方式安装软件

一个典型的 Linux 软件会在不同的地方创建文件，需要 root 权限才能对系统进行这些更改。

AppImage 不这样做。事实上，AppImage 并没有真正安装软件。它是一个压缩映像，包含运行所需软件所需的所有依赖项和库。

你执行 AppImage 文件，你运行软件。没有提取，没有安装。你删除 AppImage 文件，软件就被删除了（我们稍后会看到）。您可以将其与 Windows 中的 .exe 文件进行比较，这些文件允许您在不实际执行安装过程的情况下运行软件。

让我列出 AppImage 的一些功能或优点。

### 应用图像功能

- 发行版无关：可以在各种不同的 Linux 发行版上运行
- 无需安装和编译软件：点击播放
- 无需root权限：不触及系统文件
- 便携性：可以在任何地方运行，包括活动磁盘
- 应用程序处于只读模式
- 只需删除 AppImage 文件即可删除软件
- 默认情况下，AppImage 中打包的应用程序不被[沙盒化。](https://en.wikipedia.org/wiki/Sandbox_\(computer_security\))

## B. 如何在 Linux 中使用 AppImage

使用 AppImage 相当简单。它通过以下 3 个简单的步骤完成：

- 下载 AppImage 文件
- 使其可执行
- 运行

不用担心，我将向您详细介绍如何运行 AppImage。我在本 AppImage 教程中使用的是 Ubuntu 16.04，但您也可以在其他 Linux 发行版上使用相同的步骤。毕竟，AppImage 的全部意义在于独立于发行版。

### 第一步：下载 .appimage 包

有很多 AppImage 格式的软件可用。GIMP、Krita、Scribus 和 OpenShot 只是其中的几个名称。您可以在[此处](https://github.com/AppImage/AppImageKit/wiki/AppImages)找到以 AppImage 格式提供的大量应用程序列表。

我将在本教程中使用 OpenShot 视频编辑器。你可以从它的[网站上](https://www.openshot.org/download/)下载它。

### 第 2 步：使其可执行

默认情况下，下载的 AppImage 文件没有执行权限。您必须更改文件的权限才能使其可执行。你不需要root权限来做到这一点。

如果您更喜欢图形方式，只需右键单击下载的 .appimage 文件并选择属性。

右键单击 AppImage 文件并选择属性

在下一个屏幕中，转到“权限”选项卡并选中“允许将文件作为程序执行”框。

使 AppImage 文件可执行

而已。您已使文件可执行。

或者，如果您更喜欢命令行，您可以简单地使用 chmod u+x <AppImage File> 使其可执行。

### 第 3 步：运行 AppImage 文件

使 AppImage 文件可执行后，只需双击它即可运行它。它将看到该软件正在运行，就像您在系统上安装它一样。酷，不是吗？

## C. 如何卸载 AppImage 软件

由于从未安装过该软件，因此无需“卸载”它。只需删除关联的 AppImage 文件，您的软件就会从系统中删除。

## D. 在 Linux 中使用 AppImage 时要记住的事项

您应该了解的有关 AppImage 的其他信息很少。

### 1. 打包不好的AppImage即使有执行权限也不会运行

AppImage 的概念是将所有依赖项都包含在包本身中。但是，如果开发人员认为他已经打包了所有依赖项但实际上并没有发生呢？

在这种情况下，您会看到即使授予 AppImage 执行权限也无济于事。您单击 AppImage 并没有任何反应。

您可以通过打开终端并像运行 shell 脚本一样运行 AppImage 来检查是否存在此类错误。这是一个例子：