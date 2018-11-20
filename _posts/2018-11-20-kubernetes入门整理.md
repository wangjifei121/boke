- 简单描述

  -  k8s是一个编排容器的工具，其实也是管理应用的全生命周期的一个工具，从创建应用，应用的部署，应用提供服务，扩容缩容应用，应用更新，都非常的方便，而且可以做到故障自愈，例如一个服务器挂了，可以自动将这个服务器上的服务调度到另外一个主机上进行运行，无需进行人工干涉。

- 优势：

  - k8s可以更快的更新新版本，打包应用。
  - 更新的时候可以做到不用中断服务，服务器故障不用停机。
    - 可以做到故障自愈，例如一个服务器挂了，可以自动将这个服务器上的服务调度到另外一个主机上进行运行，无需进行人工干涉。
  - 从开发环境到测试环境到生产环境的迁移极其方便，一个配置文件搞定，一次生成image，到处运行。

- 在k8s进行管理应用的时候，基本步骤是：

  - 1、创建集群

    - 为什么要使用集群？

      - 使用集群，create cluster是为了掩盖底层的无能，在各种环境中，底层的硬件各不相同，有的是各种低廉的服务器，有的各种云环境，有的是各种vm，有的各种host machine，要想屏蔽底层的细节，增强可靠性和稳定性，从而需要创建集群。
      - 创建集群的好处
        - 统一对外提供接口，无须进行各种复杂的调用；
        - 提供更好的可靠性，服务器宕机那么频繁，物理磁盘那么容易损坏，无须担心，集群统一进行调配；
        - 提供更好的性能，组合集群中各个机器的计算存储网络资源，提供更好的TPS和PS；
        - 提供横向扩容的能力，在进行横向扩容的时候，性能基本上能呈线性增长。

    - 创建集群

      - 在k8s只要使用两条指令就可以创建一个集群

        - 1、kubectl init # 进行初始化，创建一个master节点
        - 2、kubectl join xxx # 创建一个node节点，加入这个集群

      - 集群两种类型的主机说明

        - k8s在物理上进行划分的时候，划分了两种类型的主机

          - master节点，主要用来调度，控制集群的资源等功能；
            - 用来控制v,用来存储各种元数据
          - node节点，主要是用来运行容器的节点，也就是运行服务的节点。
            - 真正来干活的,node节点定时与master进行通信，通过kubelet进程来汇报信息。

        - 

          ![img](https://img.mubu.com/document_image/9b924fc6-41a4-49c9-91e5-efe654a88488-983162.jpg)

    - 查看集群指令

      - kubectl cluster-info # 查看主节点信息

        ![img](https://img.mubu.com/document_image/f683d22a-728a-4df7-b106-af10600ee2c2-983162.jpg)

      - kubectl get nodes -o wide # 查看node节点信息

        ![img](https://img.mubu.com/document_image/a54a7b08-e7ec-40ed-8936-c1cf92957ca4-983162.jpg)

  - 2、部署应用

    - 一条指令就能运行一个服务，有了image之后就是这么简单。所以，在开发完成程序之后，需要将程序打包成image，然后放到registry中，然后就能够运行应用了。

      - kubectl run kel --image=nginx:1.7.9 --port=80

        ![img](https://img.mubu.com/document_image/9b975fa8-e506-4887-9356-7077eb3c4295-983162.jpg)

      - kubectl get deployments/kel

        ![img](https://img.mubu.com/document_image/e083d4ad-2ffe-4dde-baef-5f68fc4438b3-983162.jpg)

      - kubectl get pods -w wide

        ![img](https://img.mubu.com/document_image/2875183f-f1d3-4e5f-ab1c-a804f1ece51b-983162.jpg)

    - 部署完成应用之后，就可以看到应用的名称，期望状态是运行一个pod，当前有一个pod，活动的也是一个，还有启动的时间，那么什么是pod呢？

      - 在k8s里面，集群调度的最小单元就是一个pod，一个pod可以是一个容器，也可以是多个容器，例如你运行一个程序，其中使用了nginx，使用mysql了，使用了jetty，那么可以将这三个使用在同一个pod中，对他们提供统一的调配能力，一个pod只能运行在一个主机上，而一个主机上可以有多个pod。

      - 为什么要使用pod，为什么不能直接使用容器呢？使用pod，相当与一个逻辑主机，还记得创建一个vm，在vm上运行几个进程么，其实道理是一样的，pod的存在主要是让几个紧密连接的几个容器之间共享资源，例如ip地址，共享存储等信息。如果直接调度容器的话，那么几个容器可能运行在不同的主机上，这样就增加了系统的复杂性。

      - 

        ![img](https://img.mubu.com/document_image/fe96f36f-903e-4f0f-82b9-99ce45613030-983162.jpg)

        ![img](https://img.mubu.com/document_image/c14a9d3b-511b-4e46-a56b-86885d58a941-983162.jpg)

    - 

  - 3、发布应用

    - 发布应用主要就是对外提供服务，可能会有人提出疑问，我都运行了服务，为什么还不能提供服务：

      - 这是因为在集群当中，创建的ip地址等资源，只有在同一个集群中才能访问，每个pod也有独一的ip地址，当有多个pod提供相同的服务的时候，就需要有负载均衡的能力，从而这里就涉及到一个概念就是service，专门用来提供服务的。

    - kubectl expose deployment kel --type="NodePort" --port=80 # 将kel应用的 80端口 以节点端口方式暴露

      ![img](https://img.mubu.com/document_image/f6c8796c-6a67-4246-a86a-0ce60d7fcce2-983162.jpg)

    - kubectl get services/kel

      ![img](https://img.mubu.com/document_image/3d7c3923-0470-4ea4-9aa6-a13223fabcad-983162.jpg)

    - 服务主要是用来提供外界访问的接口，服务可以关联一组pod，这些pod的ip地址各不相同，而service相当于一个复杂均衡的vip，用来指向各个pod，当pod的ip地址发生改变之后，也能做到自动进行负载均衡，在关联的时候，service和pod之间主要通过label来关联，也就是标签（-l表示为label）。

    - kubectl get services -l run=kel

      ![img](https://img.mubu.com/document_image/fb33299b-b31d-4525-87fe-07e030247244-983162.jpg)

    - 外界就可以访问此应用

      - kubectl describe services/kel

        ![img](https://img.mubu.com/document_image/6f2f9a11-ad53-4376-9402-635c4db40dc2-983162.jpg)

    - 

      ![img](https://img.mubu.com/document_image/abb741e4-3f8e-4285-89c9-f6972f3fa5f1-983162.jpg)

  - 4、扩展应用 (扩容缩容  )

    - 横向扩展的能力

    - 双十一怎么办? 扩容

      - kubectl scale deployments/kel --replices=2

        ![img](https://img.mubu.com/document_image/44e968eb-c23b-4cc2-a938-2e776f380923-983162.jpg)

    -  过了双十一怎么办? 缩容

      - kubectl scale deployments/kel --replices=1

        ![img](https://img.mubu.com/document_image/8d03fe3f-1f54-4197-889b-ef688a24161f-983162.jpg)

  - 5、更新应用

    - 有新版本了，我要发布

      - kubectl set image deployments/kel kel=nginx:1.8

        - 滚动更新。根据新的image创建一个pod，分配各种资源，然后自动负载均衡，删除老的pod，然后继续更新,不会中断服务

          ![img](https://img.mubu.com/document_image/b54b1092-7c5a-488c-932c-f26ea4ff3f85-983162.jpg)

      - 更新错了怎么办，不会影响生产业务，回滚就好了

        - kubectl rollout undo deployments/kel

          ![img](https://img.mubu.com/document_image/2991a462-c0e2-422e-9e89-d846635fe935-983162.jpg)

- 后话

  - k8s的基本入门，其实算是一种用户视角，只是用来演示如何使用k8s，怎么提高了生产力而已。如何提高了发布的效率，更新版本的效率，更方便更快捷的上线新版本。
  - 但是在运维关注的视角下，这些远远不够：
    - master？存储了哪些元数据，存储在etcd中？
    - 如何来进行监控？
    - 在很多很多系统情况下，怎么来部署k8s，是一个项目一个k8s还是一个k8s多个项目？等等一系列的问题。