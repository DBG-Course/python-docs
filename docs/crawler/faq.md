# FAQ

> 此处会收集整理同学们常问的问题，给出统一的解答

## Python编程实验：爬虫与信息系统


### [Python 环境] 如何安装 Conda 虚拟环境

1. 安装 MiniConda。
    - MacOS 

        ```bash
        mkdir -p ~/miniconda3

        curl https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o ~/miniconda3/miniconda.sh

        bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3

        rm -rf ~/miniconda3/miniconda.sh

        ~/miniconda3/bin/conda init zsh
        ```

        摘自 [MacOS miniconda安装方法](https://zhuanlan.zhihu.com/p/707270703)

    - Windows：可参考 [Miniconda 安装](https://www.quanxiaoha.com/conda/install-miniconde.html)


2. 为 Conda 换源，可参考 [https://mirror.tuna.tsinghua.edu.cn/help/anaconda/](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)


### [爬虫] 爬到的歌曲没有歌词怎么办？

Web 系统中的歌曲详情页需要展示歌词，因此要求展示的 2000 首歌曲都应有歌词。

<del>（否则爬 2000 首无歌词的歌曲就不需要爬歌词了 😭 ）</del>


### [爬虫] 歌曲在爬取后下架了 / 链接失效了怎么办？

网站中展示的歌曲或歌手原始网站 URL 失效是允许的。但是展示的其他信息，如名称、图片、歌词等仍需要正常展示。


### [爬虫] 请求难以分析 / 请求加密了 该怎么办？

1. 使用 Selenium 控制浏览器进行请求，这样可以避免对复杂的请求进行分析。学习 Selenium 的参考资料：
    - [Selenium文档 - 入门指南](https://www.selenium.dev/zh-cn/documentation/webdriver/getting_started/)
    - [2023 酒井科协暑培 - 爬虫](https://summer23.net9.org/backend/crawler/)
    - [2024 酒井科协暑培 - 后端 Track](https://summer24.net9.org/backend/index_backend/)

2. 搜索并参考网上其他人写的同类爬虫所使用的接口和实现方式。

    实际开发中通常会避免“造轮子”，因此参考网络上的代码是允许的，但是不能直接复制粘贴，建议在看懂思路后自行完成实现。对于参考了网上代码的部分，需要在提交代码的注释中明确给出参考代码的来源。


### [Web系统设计] 一定要使用 Django 吗？可以使用其他框架 / 静态页面吗？

Web 系统不一定要用 Django 框架搭建，但是必须使用 Python 完成。使用静态网页、把所有信息全部传到前端处理、主体用其他语言实现都是不可以的。


### [Web系统设计] 爬取的歌曲有多个创作者时，需要都在歌曲详情页中展示并提供链接吗？

可以实现为歌曲有多个创作者；也可以忽略多个歌手，仅在歌曲详情页中展示并允许跳转到其中某一个歌手。

### [数据分析] “给出3个有意义的结论” 是指什么？有没有例子？


- 往年参考样例（作业为视频网站爬虫）
    - 收集到的鬼畜区视频评论中，热词 top 5 是 xx，占比分别为 xx%（附词云图）
    - 视频播放量和视频的投币数呈明显的线性相关性（附散点图，拟合直线和相关系数）
    - 在收集到的视频中5-7时发布的视频数量最少，占比仅为 xx%（附频率分布直方图）

- 这一任务的目的是练习使用 `numpy` 等 Python 库对爬取的数据进行数据统计、分析和展示，发现一些有趣或有价值的信息，对结论的类型没有明确要求。大家可以自行选择分析的话题与思路，无需囿于以上的案例。

- 但是需要注意，统计与分析需要有一定深度与启发意义，结论不可太过简单浅显，不可直接对数据进行粗略的描述，比如仅给出个别统计量。不合理的例子：
    - 所有视频的平均播放量为xxx
    - xx相关视频在所有视频占10%
