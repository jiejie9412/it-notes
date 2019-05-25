.. Copyright 2019 LI,JIE-YING. All rights reserved.

Drone 搭建
============

目前支援的程式碼管理平台有

    - GitHub
    - GitLab
    - Gitea
    - Gogs
    - Bitbucket Cloud
    - Bitbucket Server

| 我選擇使用 Gitea 搭配 Multi-Machine 模式
| 分成 server 跟 agent 兩個 machine，agent machine 也可以進行擴充，適應不同平台。

`官方教學 <https://docs.drone.io/installation/gitea/multi-machine/>`_

- 創造 server 跟 agent 溝通的金鑰

使用在 DRONE_RPC_SECRET 變數。

::

    $ openssl rand -hex 16
    5ecb84be097f717315e7a97a705a0f12

Server
--------

1. 下載

`Docker Hub Of Drone <https://hub.docker.com/r/drone/drone/>`_

::

    $ docker pull drone/drone:1

2. 定義 docker-compose.yml

::

    version: '3.7'
    services:

        drone-server:
            image: drone/drone:1
            deploy:
                replicas: 1
                restart_policy:
                    condition: on-failure
            ports:
                # 這裡是內網使用，如果需要在用外部也能看到。推薦使用 traefik 做代理，並使用 tls
                - 80:80
            networks:

                # 跟 web-proxy 連接的
                - frontsite

                # 如果要使用額外 db 需要跟前端隔離。
                - backsite

                # drone net
                - drone_site

            volumes:

                # drone 額外需要的 volume
                - ./drone:/data

            environment:

                # 這兩項 SERVER 設定是用在自動設定 webhook 時的值。
                # PROTO 的實際運行的模式沒有關係，我試過就算用 'httpsssssss'，也 launch 的起來。
                # 然後 git server hook 的網址就變成 'httpsssssss://drone.domain.com'
                # HOST 不要加 'http://'，drone 會自動用 PROTO://HOST
                # 加了會變成 'http://http://drone.domain.com'，會 hook 不到，我為了這個浪費了半天。
                - DRONE_SERVER_HOST=drone.domain.com
                - DRONE_SERVER_PROTO=https

                # debug 級別日誌的 boolean 應該用於自動SSL證書生成和配置。 默認值為false。
                # 這個名稱很奇怪，但是官網上是這樣說的。
                - DRONE_TLS_AUTOCERT=false

                # database 項目，如果用 SQLite 的可以不用加，drone 會自動幫你配置好到 /date 底下。
                - DRONE_DATABASE_DRIVER=postgres
                - DRONE_DATABASE_DATASOURCE=postgres://<username>:<password>@<your_db_host>:<db_port>/<dbname>?sslmode=disable

                # RPC 設定，跟 agent 連接需要。
                # SECRET 使用上方生成的結果。
                - DRONE_RPC_SECRET=5ecb84be097f717315e7a97a705a0f12
                - DRONE_AGENTS_ENABLED=true

                # 要綁定的 git server
                - DRONE_GITEA_SERVER=https://gitea.domain.com

                # 以下兩項，drone 可以透過 oauth 直接連接到指定帳戶
                # 但是會直接登入，不適合用在開放可外連的環境
                #- DRONE_GITEA_CLIENT_ID=
                #- DRONE_GITEA_CLIENT_SECRET=

                # drone 會根據此項做權限操作，對 myuser 增加 admin 權限。
                # 只需要使用一次，drone 會自己對 db 持久化。
                #- DRONE_USER_CREATE=username:myuser,admin:true

                # 都要進行身份驗證，只有在庫是都需要註冊會員才可以看才需要
                # gitea 如果是要註冊才能觀看，這個 false 會 clone 會報錯
                # 要摸 gitea 改不須註冊，要摸這個改 true 試試看。
                - DRONE_GIT_ALWAYS_AUTH=false

                # 要不要顯示 log，如果用 -it 模式看得到
                #- DRONE_LOGS_DEBUG=true
                #- DRONE_LOGS_PRETTY=true

Agent
-------

1. 下載

`Docker Hub Of Drone <https://hub.docker.com/r/drone/drone/>`_

::

    $ docker pull drone/agent:1


2. 定義 docker-compose.yml

::

    drone-agent:
        image: drone/agent:1
        deploy:
            replicas: 1
            restart_policy:
                condition: on-failure
        depends_on:
            - "drone-server"
        networks:
            - drone_site
        volumes:
            # agent 要把 docker socket 引入，ci/cd 過程才可以使用 docker.
            # drone 是依賴 docker 所以一定要
            - /var/run/docker.sock:/var/run/docker.sock
        environment:
            # 你的 drone server web 網址
            - DRONE_RPC_SERVER=http://drone_drone-server
            # SECRET 使用上方生成的結果。
            - DRONE_RPC_SECRET=15a8d73efbcb0e32e608a9d93e94679b
            # agent 可以同時 ci/cd 的數量
            - DRONE_RUNNER_CAPACITY=3

.. warning::
    我用 docker swarm 就是為了可以用 secrets 結果 drone 只能用環境變數給值

.. TODO::
    看能不能想辦法讓結合 secret 跟 env
