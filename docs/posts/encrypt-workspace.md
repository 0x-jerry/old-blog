---
title: 加密工作区的实现方案
date: 2021-12-25T15:48:57.000Z
tags: [encrypt, workspace, vscode]
---

自己有时候会写一些私人笔记，但又习惯用 `vscode` 来码字。也习惯了 `github` 来管理版本，存储也就直接选择同步到 `github` 了。

但是呢，存储服务发生数据泄漏，也是有可能的。所以呢，就自己用 `aes-256-gcm` 加密了一下，但这又导致 git 无法查看具体的变更，因为文件被加密了。

而且每次都需要解密 -> 修改 -> 加密 -> 同步。太麻烦了。

所以，问题来了，如何在加密的同时，又能用 git 查看变更呢？

<!-- more -->

## 思考

先捋一捋条件：

1. 笔记内容需要密码才能查看，无法直接查看
2. 需要一种方案显示具体的变更历史

在同时满足以上两个条件下，想一想方案。

### 有点笨的方案

这个方案的主要思想在于，每个文件，在每次提交修改的时候，都保存一个对应的加密版本。然后在查看记录的时候，解密对应的不同版本，然后再对比。

例如，文件 A，一共提交了两次修改，所以有两个加密的版本，分别为：

- A-1
- A-2

这两个版本都是加密的，同步到服务器。

在需要查看变更时，同步解密这两个版本，然后做对比，就可以查看到两个不同版本之间的具体变更了。

这个方案虽然可行，但是感觉有点蠢，存储的东西也比较多。

缺点：存储比较冗余，会重复多次存储没有修改过的内容。而且无法直接搜索，需要把每个文件解密之后才能搜索。

### 有点瑕疵的方案

这个方案的主要思想在于把整个 git 仓库加密，这个方案比较简单。

例如，先创建一个 git 仓库，做一些正常的提交记录。，在上传到云存储的时候，把整个 git 仓库加密保存成一个压缩文件，然后再保存到云端。

缺点比较明显，一是每次都需要全量同步，二是每次修改都需要全量 解密/加密。

## 另外的一些限制

上面提到了两种方案，感觉都不是最佳方案。非要选一种的话，个人可能更倾向于第二种方案，原因是实现比较简单。

现在想想，除了最早提到的两个条件，想要一个比较好的方案，还需要以下限制条件才行：

1. 增量同步
2. 支持 vscode，这是个个人需求。

感觉比较困难，没有啥太好的方法，如果能扩展 git 的话，倒是有一个想法，给 git 写一个加密的插件，保存的时候，加密一下，进行其他操作的时候，再解密，
这一块不熟悉，感觉也不太可能，太多以来原生读取/写入操作。估计不容易实现。

### 另辟蹊径

因为 git 支持 [lfs][git-lfs]，所以，采用第一种方式来存储的话，应该会节省一些空间，相比第二种方案，会好一点点。

之前也抽空试了试给 vscode 写个插件来支持自动加解密文件。安装链接：[vscode-writing]

[lfs][git-lfs] 在克隆等操作的时候，也是增量同步的，相对来讲，会好一点点。

缺点也是比较明显的，无法直接对比出历史修改记录。

如果能支持历史记录，那应该就比较好了。

记录一下解决方案，在对比的时候，读取两个不同的版本，解密之后再对比。

## 结论

整体思考下来，利用 [lfs][git-lfs] 实现第一种方案，也是一个不错的选择。但感觉还不是最好的方案。

[git-lfs]: https://git-lfs.github.com/
[vscode-writing]: https://github.com/0x-jerry/vscode-private-notes