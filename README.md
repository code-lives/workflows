# Workflows 如何使用

> 触发的分支全部是 release 分支,用哪个自己改。不然我这 push 一次运行一堆 action 😄

> 需要创建.github/workflows 文件夹存放

> 关于 uses 使用文档[查看](https://github.com/marketplace?type=actions)

# yml 文件中 secrets 变量，类似于模版赋值。

> 在当前项目的设置【Secrets and variables】->【Actions】->【New repository secret】创建

![secre Image](images/secret.png)

# 开始介绍不同文件的作用

## docker-build.yml 镜像打包上传镜像库

> 根据 Dockerfile 进行打包，生成镜像最后上传到镜像库。这里使用的阿里云镜像库。

> Dockerfile 就在根目录存放即可。如果存在其他目录如 docker 文件夹

```
docker build -t ${{ secrets.IMG_ADDRESS }}:$GITHUB_RUN_NUMBER -f release/Dockerfile .
```

## docker-build-new.yml 镜像打包上传镜像库

> 对比 docker-build.yml。这个案例的 docker 登录、镜像打包、上传。使用了更多的 uses 动作

## docker-deploy-linux.yml 打包部署到 linux

> 打包成镜像，上传镜像库，登录 Linux，拉取镜像，部署

## github-issues-feishu-bot 有问题飞书通知

> Github 有问题创建和关闭 or 评论 的话就，就飞书通知

1、飞书创建群聊

2、加入自定义机器人

3、把 webhook 复制到自己的 secrets 变量 FEISHU_WEBHOOK_URL

### 飞书机器人

![secre Image](images/feishu-bot.png)

### 接收效果

![secre Image](images/feishu.png)

## php composer 的 release 版本打包

# 相关文档链接

### issues types 有哪些类型 比如 opened、edited、deleted、closed 等状态

[issues.types 的事件文档查看](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#issues)

### issues 有哪些类型的字段可以使用传递

[github.event.issue.html_url 变量文档查看](https://docs.github.com/en/rest/issues/issues?apiVersion=2022-11-28)

## 成定时每天 9 点 30 发送消息

```yml
name: 定时飞书通知
on:
  schedule:
    - cron: "30 9 * * *" # 每天UTC时间的早上9点30

jobs:
  send_notification:
    runs-on: ubuntu-latest

    steps:
      - name: 发送消息
        env:
          FEISHU_WEBHOOK_URL: ${{ secrets.FEISHU_WEBHOOK_URL }}
        run: |
          curl -X POST -H 'Content-Type: application/json' -d '{
            "msg_type": "interactive",
            "card": {
              "elements": [
                {
                  "tag": "div",
                  "text": {
                    "tag": "lark_md",
                    "content": "这是每天定时通知的内容。"
                  }
                },
                {
                  "tag": "action",
                  "actions": [
                    {
                      "tag": "button",
                      "text": {
                        "tag": "plain_text",
                        "content": "快速查看"
                      },
                      "url": "https://github.com/code-lives/punch-card" # 你的仓库链接
                    }
                  ]
                }
              ]
            }
          }' $FEISHU_WEBHOOK_URL
```
