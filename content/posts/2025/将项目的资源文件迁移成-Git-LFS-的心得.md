+++
date = '2025-09-17T10:34:00+08:00'
title = '将项目的资源文件迁移成 Git LFS 的心得'
categories = ["编程技术"]
tags = []
+++

最近，我遇到了一个棘手的问题：我的 Git 仓库因为包含了大量的资源文件（如图片、音视频等）而变得异常臃肿。每一次 `clone`、`pull` 或 `push` 操作都变得非常缓慢，严重影响了开发效率。在寻找解决方案时，我了解到了 Git LFS (Large File Storage)，并借助它成功地为我的仓库“瘦身”。

这篇文章记录了我将现有仓库迁移到 Git LFS 的完整过程和一些心得体会，内容主要参考了我与 Gemini 的一次技术探讨。希望对遇到同样问题的朋友有所帮助。

## 问题：当大文件已经提交到 Git 历史中

如果你和我一样，在意识到需要使用 Git LFS 之前，就已经将许多大文件直接提交到了版本历史中，那么简单的 `git lfs track` 是不够的。因为这些大文件已经成为了仓库历史的一部分，即使你在最新的提交中删除了它们，它们依然存在于 `.git` 目录中，占据着大量空间。

核心思路是：**重写 Git 仓库的历史记录**，将历史提交中的大文件“抽取”出来，用 Git LFS 的指针（pointer）文件替代它们。

## 解决方案：使用 `git lfs migrate`

Git LFS 官方提供了一个强大的工具 `git lfs migrate`，专门用于解决这个问题。以下是详细的操作步骤。

### 第 1 步：备份你的仓库（极其重要！）

在进行任何历史重写操作之前，**请务必备份你的整个项目**！这是一个有风险的操作，一旦出错，可能会导致数据丢失。

最简单的方式是完整地复制一份项目文件夹：

```bash
cp -R your-repo your-repo-backup
```

更可靠的方式是创建一个完整的镜像备份（Bare Clone）：

```bash
git clone --mirror git@github.com:your/repo.git your-repo.git.backup
```

### 第 2 步：安装并初始化 Git LFS

如果你的本地环境中还没有安装 Git LFS，请先安装它。安装完成后，在你的仓库目录中执行一次初始化命令：

```bash
git lfs install
```

### 第 3 步：识别需要迁移的大文件

你需要确定哪些类型的文件或者哪些目录下的文件应该被 LFS 追踪。通常是根据文件扩展名来确定。

一个很好的实践是先检查一下你的仓库里到底哪些文件最大。你可以使用 `find` 命令来查找大于特定体积（例如 10MB）的文件：

```bash
# 在项目根目录运行，查找大于 10MB 的文件
find . -type f -size +10M
```

根据查找结果，结合你的项目类型，确定一个需要迁移的文件类型列表。

### 第 4 步：执行迁移命令

这是整个流程的核心。使用 `git lfs migrate import` 命令来重写历史。

```bash
git lfs migrate import --everything --include="*.webp,*.png,*.jpg,*.jpeg,*.gif,*.ico,*.mp3,*.wav,*.mp4"
```

这里的参数非常关键：

* `--everything`: 这个参数至关重要！它会重写**所有**的本地分支和标签。如果省略它，将只会重写你当前所在的分支，导致你的仓库历史出现分叉，后续的合并操作会变得极其困难，并且旧的大文件对象也无法被真正清理，仓库体积不会减小。所以，对于修复现有仓库的场景，**必须使用 `--everything`**。
* `--include`: 指定要迁移的文件模式，可以用逗号分隔多个模式。

这个过程可能会花费一些时间，具体取决于你的仓库大小和历史记录的复杂程度。

### 第 5 步：提交 `.gitattributes` 文件

迁移命令执行成功后，你会发现 Git LFS 自动创建或修改了 `.gitattributes` 文件。这个文件告诉 Git 哪些文件应该由 LFS 来处理。你不需要手动执行 `git lfs track`，`migrate` 命令已经帮你完成了。

你需要做的就是将这个文件提交到仓库：

```bash
git add .gitattributes
git commit -m "chore: Configure Git LFS tracking"
```

### 第 6 步：强制推送到远程仓库

因为你已经重写了历史记录，所以无法通过常规的 `git push` 来更新远程仓库。你需要进行强制推送。

**警告**：强制推送会覆盖远程仓库的历史。在执行此操作前，必须确保所有团队成员都已经提交并推送了他们的本地工作，并且知晓你将要进行历史重写。

推荐使用 `--force-with-lease`，它比 `--force` 更安全，因为它会检查远程分支在你上次拉取后是否有新的提交，如果有，则推送会失败，防止覆盖他人的工作。

```bash
# 推送所有被重写的分支
git push --force-with-lease --all origin

# 推送所有被重写的标签
git push --force-with-lease --tags origin
```

### 第 7 步：通知所有协作者同步

这是团队协作中至关重要的一步。在你完成强制推送后，所有协作者的本地仓库都基于旧的历史，需要进行同步。

* **对于没有本地未推送修改的成员（推荐）**：
    最简单、最安全的方式是让他们删除本地仓库，然后重新从远程 `clone` 一份新的。

* **对于有本地未推送修改的成员（高级操作）**：
    他们需要将自己的本地分支变基（rebase）到新的（被重写后的）上游分支上。例如，如果他们的功能分支是 `my-feature`，主分支是 `main`：

    ```bash
    git fetch origin
    git checkout my-feature
    git rebase origin/main
    ```

    这个过程可能会产生冲突，需要手动解决。

### 第 8 步：清理本地的 `.git` 目录（可选）

完成上述步骤后，虽然历史记录被重写了，但你本地的 `.git` 目录可能仍然包含那些被移除的大对象的“幽灵”副本。可以通过 Git 的垃圾回收机制来清理它们，以减小本地 `.git` 文件夹的体积。

```bash
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

协作者在重新克隆仓库后，也会自动获得一个干净、小体积的 `.git` 目录。

## 附：一个更全面的文件类型列表

除了常见的图片和视频，根据项目类型的不同，你可能还需要追踪其他类型的大文件。以下是一些补充建议，你可以按需添加到你的 `--include` 参数中：

* **设计与图形资源**: `*.psd`, `*.ai`, `*.svg`, `*.fig`, `*.sketch`, `*.tga`, `*.tiff`
* **文档与字体**: `*.pdf`, `*.pptx`, `*.docx`, `*.otf`, `*.ttf`, `*.woff`, `*.woff2`
* **压缩包与磁盘映像**: `*.zip`, `*.rar`, `*.7z`, `*.tar`, `*.gz`, `*.iso`, `*.dmg`
* **开发相关二进制文件**: `*.dll`, `*.so`, `*.a`, `*.lib`, `*.jar`, `*.exe`, `*.pdb`
* **游戏开发资源**: `*.unitypackage`, `*.fbx`, `*.obj`, `*.blend`, `*.uasset`, `*.umap`

一个更全面的迁移命令示例如下：

```bash
git lfs migrate import --everything --include="\
*.webp,*.png,*.jpg,*.jpeg,*.gif,*.ico,\
*.mp3,*.wav,*.mp4,*.mov,*.avi,*.ogg,\
*.psd,*.ai,*.svg,\
*.pdf,*.pptx,*.docx,\
*.otf,*.ttf,*.woff,*.woff2,\
*.zip,*.rar,*.7z,\
*.dll,*.so,*.a,*.lib"
```

## 注意事项：迁移后的常见现象

在执行完 `git lfs migrate` 命令后，你可能会观察到一些“奇怪”但完全正常的现象，这通常和 Git LFS 的工作原理有关。

### 现象一：文件变成了文本指针

你可能会发现，一些刚刚被迁移的图片文件，用 `file` 命令查看时，竟然显示为 `ASCII text`。

```bash
$ file static/images/about.webp
static/images/about.webp: ASCII text
```

**核心原因**：这不是文件损坏，而是 Git LFS 正常工作的表现。`git lfs migrate` 命令会将大文件的实际内容从 Git 的历史记录中移除，并在原来的位置放置一个轻量的“指针文件”（Pointer File）。你看到的 `ASCII text` 就是这个指针文件的内容，它通常长这样：

```text
version https://git-lfs.github.com/spec/v1
oid sha256:de36aba129a393955675f859f977c4c37521c161962381283e78b10c598b554c
size 13824
```

这个指针告诉 Git LFS 去哪里找到真实的、体积较大的文件。

### 现象二：`git lfs ls-files` 的输出包含 `*` 和 `-`

当你运行 `git lfs ls-files` 时，会看到文件前面有 `*` 或 `-` 两种符号。

```text
ec4cf991b0 * static/avatar/avatar.webp
de36aba129 - static/images/about.webp
```

它们的含义如下：

* `-` (hyphen): 表示该文件的 LFS 对象**仅存在于你的本地**，还没有被推送到远程 LFS 服务器。这是 `migrate` 之后的默认状态。
* `*` (asterisk): 表示该文件的 LFS 对象**已经被成功推送**到了远程 LFS 服务器。

### 如何解决与后续操作

迁移完成后，你需要将所有本地的 LFS 对象（标记为 `-` 的文件）推送到远程服务器：

```bash
git lfs push --all origin
```

如果你在本地看到的是文本指针而不是实际文件，可以尝试执行以下命令来强制 Git LFS 重新检出文件，将指针替换为实际内容：

```bash
git lfs checkout
# 或者更强制的
git lfs pull
```

## 总结

将一个已经存在大文件的仓库迁移到 Git LFS，虽然过程稍显复杂，但只要遵循以上步骤，就能顺利完成。关键在于：**备份**、使用 `git lfs migrate import --everything` **彻底重写历史**、**强制推送**并**通知所有协作者**。

完成这些步骤后，你的仓库就会回到一个健康、轻量的状态，所有指定的大文件都将由 Git LFS 在幕后高效管理。
