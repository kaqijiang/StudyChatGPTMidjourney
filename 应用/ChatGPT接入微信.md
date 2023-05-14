> 在微信上迅速接入 ChatGPT，让它成为你最好的助手！

## **通过Docker使用**

```SQL
# 拉取镜像
docker pull holegots/wechat-chatgpt:latest
# 运行容器
docker run -it --name wechat-chatgpt \
    -e OPENAI_API_KEY=<YOUR OPENAI API KEY> \
    -e MODEL="gpt-3.5-turbo" \
    -e CHAT_PRIVATE_TRIGGER_KEYWORD="" \
    -v $(pwd)/data:/app/data/wechat-assistant.memory-card.json \
    holegots/wechat-chatgpt:latest
# 使用二维码登陆
docker logs -f wechat-chatgpt
```

> 如何获取 OPENAI API KEY？请参考 [OpenAI API Key](https://nhrvt0kw31.feishu.cn/wiki/UygJwAmVziSypLkoJ92clKtTnUh)。

## **通过docker compose使用**

```SQL
# 根据模板拷贝配置文件
cp .env.example .env
# 使用你喜欢的文本编辑器修改配置文件
vim .env
# 在Linux或WindowsPowerShell上运行如下命令
docker compose up -d
# 使用二维码登陆
docker logs -f wechat-chatgpt
```

## **使用NodeJS运行**

> 请确认安装的NodeJS版本为18.0.0以上

```SQL
# 克隆项目
git clone https://github.com/fuergaosi233/wechat-chatgpt.git && cd wechat-chatgpt
# 安装依赖
npm install
# 编辑配置
cp .env.example .env
vim .env # 使用你喜欢的文本编辑器修改配置文件
# 启动项目
npm run dev
# 如果您是初次登陆，那么需要扫描二维码
```

> 请确保您的账号可以登陆 [网页版微信](https://wx.qq.com/)。

## **环境变量设置**

| name                         | default                                           | example                                        | description                                                  |
| ---------------------------- | ------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| API                          | [https://api.openai.com](https://api.openai.com/) |                                                | 自定义ChatGPT API 地址                                       |
| OPENAI_API_KEY               | 123456789                                         | sk-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX | [创建你的 API 密钥](https://platform.openai.com/account/api-keys) |
| MODEL                        | gpt-3.5-turbo                                     |                                                | 要使用的模型ID, 目前仅支持gpt-3.5-turbo 和 gpt-3.5-turbo-0301 |
| TEMPERATURE                  | 0.6                                               |                                                | 在0和2之间。较高的数值如0.8会使 ChatGPT 输出更加随机，而较低的数值如0.2会使其更加稳定。 |
| CHAT_TRIGGER_RULE            |                                                   |                                                | 私聊触发规则                                                 |
| DISABLE_GROUP_MESSAGE        | TRUE                                              |                                                | 禁用在群聊里使用ChatGPT                                      |
| CHAT_PRIVATE_TRIGGER_KEYWORD |                                                   |                                                | 在私聊中触发ChatGPT的关键词, 默认是无需关键词即可触发        |
| BLOCK_WORDS                  | "VPN"                                             | "WORD1,WORD2,WORD3"                            | 聊天屏蔽关键词(同时在群组和私聊中生效, 避免 bot 用户恶意提问导致封号 |
| CHATGPT_BLOCK_WORDS          | "VPN"                                             | "WORD1,WORD2,WORD3"                            | ChatGPT回复屏蔽词, 如果ChatGPT的回复中包含了屏蔽词, 则不回复 |

## 📝 **使用自定义ChatGPT API**

> https://github.com/fuergaosi233/openai-proxy

```SQL
# 克隆项目
git clone https://github.com/fuergaosi233/openai-proxy
# 安装依赖
npm install && npm install -g wrangler && npm run build
# 部署到 CloudFlare Workers
npm run deploy
# 自定义域名(可选)
添加 `Route`` 到 `wrangler.toml`
routes = [
        { pattern = "Your Custom Domain", custom_domain = true },
]
```

## ⌨️ **命令**

> 在微信聊天框中输入

```SQL
/cmd help # 显示帮助信息
/cmd prompt <PROMPT> # 设置ChatGPT Prompt
/cmd clear # 清除WeChat-ChatGPT保存的会话记录
```

项目地址：https://github.com/fuergaosi233/wechat-chatgpt