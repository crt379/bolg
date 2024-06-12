---
title: Podman学习
date: 2024-04-28 19:53:52
tags:
- Podman
category: 容器
---
Podman是一个无守护进程的容器引擎，它不需要在系统上运行任何守护进程或守护程序。它使用与OCI（Open Container Initiative）兼容的容器运行时（如runc、crun、runv等）与操作系统交互并创建正在运行的容器。

## podman 结构及相关组件
- 容器
    - conmon 是容器守护进程，负责一对一监控和管理每个 podman 容器
    - runc 负责提供容器运行时环境。用 Go 语言实现的 OCI

- 镜像
    - buildah 负责构建镜像
    - skopeo 负责与 images registry 交互。用来 pull/push/管理镜像

- 存储
    - storage 负责管理文件系统的 layers，包括容器 layer 和镜像 layer

- 网络
    - netavark 负责管理容器网络

### Podman、Buildah 和 Skopeo 的特点
Podman、Skopeo 和 Buildah 工具被开发来取代 Docker 命令功能。这些场景中的每个工具都是非常轻量级的，并专注于功能的子集。

Podman、Skopeo 和 Buildah 工具的主要优点包括：
- 以无根模式运行 - rootless 容器更安全，因为它们在运行时不需要添加任何特权
- 不需要守护进程 - 这些工具在空闲时对资源的要求要低，因为如果你没有运行容器，Podman 就不会运行。 另外，Docker 有一个守护进程一直在运行
- 引入systemd集成 - Podman 允许您创建systemd单元文件，放置容器作为系统服务运行

Podman、Skopeo 和 Buildah 的特点包括：
- Podman、Buildah 和 CRI-O 容器引擎都使用相同的存储目录，/var/lib/containers而不是默认使用 Docker 存储位置/var/lib/docker。
- 虽然Podman、Buildah和CRI-O共享相同的存储目录，但它们不能交互。这些工具可以共享镜像。
- 要以Smashing方式与Podman进行交互，您可以使用Podman v2.0 RESTful API，它可以在有根和无根的环境中工作。如需更多信息，请参阅[使用container-tools API](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/9/html-single/building_running_and_managing_containers/index#using-the-container-tools-api)。

[详情](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/9/html/building_running_and_managing_containers/assembly_starting-with-containers_building-running-and-managing-containers#doc-wrapper)