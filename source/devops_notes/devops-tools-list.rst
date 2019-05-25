.. Copyright 2019 LI,JIE-YING. All rights reserved.

Devops 工具列表與分析
========================

CI/CD Tools
------------------

1. Jenkins

    - 一個老牌的 CI/CD 工具。
    - 為人詬病難設定。
    - plugin 眾多。
    - `Jenkins github <https://github.com/jenkinsci/jenkins>`_


2. Drone

    - 年輕的 CI/CD 工具，感覺用了很潮。
    - 好設定，全部 dockerize。
    - 如果 app 也都 dockerize，這是個好選擇。
    - 但是因為年輕，plugin 較少，但大多新套件都有對應。
    - `Drone official website <https://drone.io/>`_
    - `Drone github <https://github.com/drone/drone>`_


3. GitLab CI

    - 只能用 GitLab。
    - 閉源。
    - 不考慮。


4. Travis CI

    - 跟 drone 類似。不對硬該說 drone 跟它類似。Travis 歷史比較長一點。
    - `Travis official website <https://travis-ci.org/>`_
    - `Travis github <https://github.com/travis-ci/travis-ci>`_



Git Server
---------------

1. Gogs

    - `Gogs official website <https://gogs.io/>`_
    - `Gogs github <https://github.com/gogs/gogs>`_
    - 一個輕量，而且易用的 git-Server
    - 但是擁有者把持專行，讓大家受不了，才有了 gitea

2. Gitea

    - `Gitea official website <https://gitea.io/zh-tw/>`_
    - `Gitea github <https://github.com/go-gitea/>`_
    - gogs 的分支，旨在讓更多人共同開發。
    - `簡述兩者差異的網站 <https://blog.wolfogre.com/posts/gogs-vs-gitea/>`_
