# 使用pull request提交文档

本站部署基于 **SAST-Mathematics-SIG** 下的三个Github仓库

![alt text](pics/three_repos.png)

- `Mathematics_SIG_Blog`: `*.md` 文档全都置于这个仓库中
- `blog_deploy_source`: 一些 Blog 配置相关的仓库
- `sast-mathematics-sig.github.io`: github page部署使用的仓库

采用这样的方案主要是为了让配置文件和文档本身分离，当你需要提交你的文档时，你需要

1. fork `Mathematics_SIG_Blog`仓库
2. 在本地对这个仓库的内容进行修改
3. 向上游仓库提起 pull request ，等待我们的回应
4. pull request 合入仓库，你的文档就能正常的在网页上显示了

