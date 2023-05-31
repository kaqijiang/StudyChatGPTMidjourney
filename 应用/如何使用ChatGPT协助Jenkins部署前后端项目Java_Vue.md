## 如何使用ChatGPT协助Jenkins部署前后端项目Java_Vue

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080843969.(null))

现在主流方向应该是Git服务器 + Jenkins服务器 + Docker持续化集成，但是因为Docker有一定的学习成本很多人也就没有增加Docker，我们先使用非Docker方式部署一套，然后再加入Docker对比一下方便程度。

### **依赖**

Git 、Jenkins 、Docker

> 1-6都是通用配置，这些应该没有什么问题，不再讲述，可以从网上找到很多图文步骤，具体从7配置项目开始

### **1-6通用配置**

- 登录服务器
- 服务器安装Git
- 服务器安装Jenkins
- 初始化Jenkins
- Jenkins安装插件
- Jenkins中创建项目

### **7.配置项目**

##### **丢弃旧构建**

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844112.(null))

##### **设置Git参数** 

- 需要提前安装插件git parameterized

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080843983.(null))

##### **设置git地址 这里需要注意\*/master 还是main**

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null))

##### **凭证设置 Credentials**

Git设置中 Credentials 选择Add添加凭证 选择SSH Username with private Key

输入公钥即可

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844024.(null))

##### **构建方式(前端忽略Maven,前端直接使用shell)**

使用maven 直接构建 clean后 package

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844112-5059724.(null))

然后再运行脚本将52服务器的打包后数据scp到51服务器，并运行

```Bash
starttime=$(date +%Y-%m-%d\ %H:%M:%S)
echo "star Time: ${starttime}"
echo sync jar

scp ruoyi-admin/target/ruoyi-admin.jar root@192.168.0.11:/home/crm/

ssh root@192.168.0.51  "cd /home/crm;  sh start.sh"

# 获取结束时间
endtime=$(date +%Y-%m-%d\ %H:%M:%S)
echo "End Time: ${endtime}"

# 计算使用时间差
start_seconds=$(date --date="${starttime}" +%s)
end_seconds=$(date --date="${endtime}" +%s)
total_seconds=$((end_seconds - start_seconds))

# 输出总共使用的时间
echo "Total Time: ${total_seconds} seconds"
```

前端打包脚本，在Jenkins机器上打包完后将dist.zip文件传输到测试服务器上，运行脚本解压。

```Bash
starttime=$(date +%Y-%m-%d\ %H:%M:%S)
echo "star Time: ${starttime}"
echo "开始构建"

npm install

npm run build:dev

# 传输
scp dist.zip root@192.168.0.51:/home/web

# 运行脚本
ssh root@192.168.0.51  "cd /home/crm_web;  sh start.sh"

# 获取结束时间
endtime=$(date +%Y-%m-%d\ %H:%M:%S)
echo "End Time: ${endtime}"

# 计算使用时间差
start_seconds=$(date --date="${starttime}" +%s)
end_seconds=$(date --date="${endtime}" +%s)
total_seconds=$((end_seconds - start_seconds))

# 输出总共使用的时间
echo "Total Time: ${total_seconds} seconds"
```

解压脚本start.sh

```Bash
rm -rf ./crm-admin-vue/*
rm -rf ./crm-admin-vue/.*

unzip dist.zip -d ./crm-admin-vue/

#重启nginx

# /usr/local/nginx/sbin/nginx -s reload
```

### **飞书消息通知**

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844133.(null))

文档地址：https://open.feishu.cn/document/uAjLw4CM/ukTMukTMukTM/bot-v3/bot-overview

#### **使用机器人推送**

步骤

1.创建自己应用机器人，获取 **AppID** 和 **App Secret**。

2.通过自建机器人，获取tenant_access_token。

3.发送消息，使用tenant_access_token鉴权

##### **自建应用获取 tenant_access_token**

1. 登录[开发者后台](https://open.feishu.cn/app/)。
2. 在企业自建应用列表中，点击指定应用，进入应用详情页。
3. 进入 **凭证与基础信息** 页面，在 **应用凭证** 处，获取 **AppID** 和 **App Secret**。
   1. ![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080843992-5059723.(null))
4. 调用[自建应用获取 tenant_access_token](https://open.feishu.cn/document/ukTMukTMukTM/ukDNz4SO0MjL5QzM/auth-v3/auth/tenant_access_token_internal)接口，获取自建应用的 
   1.  `tenant_access_token`。 https://open.feishu.cn/document/ukTMukTMukTM/ukDNz4SO0MjL5QzM/auth-v3/auth/tenant_access_token_internal

   2. ```Bash
      POST
      /open-apis/auth/v3/tenant_access_token/internal
      请求
      {
        "app_id": "123",
        "app_secret": "12131"
      }
      响应
      {
        "code": 0,
        "expire": 4549,
        "msg": "ok",
        "tenant_access_token": "t-12121"
      }
      CURL方式
      
      curl 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal' \
        -H 'Content-Type: application/json; charset=utf-8' \
        -d '{
         "app_id": "123",
        "app_secret": "123"
        }'
      {"code":0,"expire":3949,"msg":"ok","tenant_access_token":"t-123"}%
      ```
5. 发送消息
   1. ```Bash
      POST
      /open-apis/im/v1/messages
      
      {
        "receive_id": "发送给谁",
        "msg_type": "interactive",
        "content": "{\"type\": \"template\", \"data\": { \"template_id\": \"模版ID\", \"template_variable\": {\"project_name\": \"项目名称\",\"time\": \"2020-02-02 09:09\",\"person\": \"at测试<at id=xxx></at>\"} } }"
      }
      ```

#### **使用webhook机器人**

##### **邀请自定义机器人入群**

​    进入你的目标群组，打开**会话设置**，找到**群机器人**，并点击**添加机器人**，选择**自定义机器人**加入群聊。

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080843992.(null))

​    为你的机器人输入一个合适的名字和描述，也可以为机器人设置一个合适的头像，然后点击下一步。

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844075.(null))

##### **配置 webhook**

​    你会获取该机器人的 webhook 地址，格式如下：

```Bash
https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxxxxxxxxxxx
```

**请妥善保存好此 webhook 地址**，不要公布在Gitlab、博客等可公开查阅的网站上，避免地址泄露后被恶意调用发送垃圾消息

![img](images/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8ChatGPT%E5%8D%8F%E5%8A%A9Jenkins%E9%83%A8%E7%BD%B2%E5%89%8D%E5%90%8E%E7%AB%AF%E9%A1%B9%E7%9B%AEJava_Vue/(null)-20230526080844137.(null))

##### **调用webhook发送消息**

​    用任意方式向该 webhook 发起 HTTP POST 请求，即可向这个自定义机器人所在的群聊发送消息。

注意： 你需要一定的服务端开发基础，通过服务端请求方式调用webhook地址。 以curl指令为例，请求示例如下：

```Bash
curl -X POST -H "Content-Type: application/json" \
-d '{"msg_type":"text","content":{"text":"request example"}}' \
https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxxxxxxxxxxx 
```

你可以把上述指令复制到 macOS系统的“终端”应用(或Windows系统的“控制台”应用)中进行测试。

请将上述代码中 `https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxxxxxxxxxxx` 更换为真实webhook的地址。 若测试出错，请先检查复制的指令是否和测试指令结构一致。

如**请求成功**，返回体为：

```Bash
{
        "code": 0,
        "msg": "success",
        "data": {},
        "Extra": null,//冗余字段，用于兼容存量历史逻辑，不建议使用
        "StatusCode": 0,//冗余字段，用于兼容存量历史逻辑，不建议使用
        "StatusMessage": "success"//冗余字段，用于兼容存量历史逻辑，不建议使用
}
```

如**请求体格式错误**，返回体如下。请检查：

- 请求体内容格式是否与各消息类型的示例代码一致
- 请求体大小不能超过20k

```Bash
{
"code": 9499,
"msg": "Bad Request",
"data": {}
}
```

#### **完整脚本python**

```Bash
import sys
import requests
import datetime

# 其他方法的定义...
APP_ID = '13242323'
APP_SECRET = '23232323'
TEMPLATE_ID = '23232323'
time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")


# 获取用户
def get_user_at(build_user, is_group):
    at = ""

    if is_group:
        if build_user == "张三":
            user = "张三,李四,王五"
        elif build_user == "李四":
            user = "张三,李四,王五"
        elif build_user == "王五":
            user = "张三,李四,王五"
        else:
            user = "张三,李四,王五"  # 设置默认值为空字符串
        at = format_user_at(user)
    else:
        at = format_user_at(build_user)

    return at


# 格式化
def format_user_at(user_string):
    user_list = user_string.split(',')
    at = ""

    for user in user_list:
        openid = get_user_openid(user)
        at += f"<at id={openid}></at> "

    return at.strip()

# openID
def get_user_openid(build_user):
    if build_user == "张三":
        at = "xxx"
    elif build_user == "李四":
        at = "xdddd"
    return at

# 获取token
def get_tenant_access_token():
    url = 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal'
    headers = {'Content-Type': 'application/json; charset=utf-8'}
    data = {
        "app_id": APP_ID,
        "app_secret": APP_SECRET
    }

    response = requests.post(url, headers=headers, json=data)
    response_json = response.json()

    if response_json['code'] == 0:
        tenant_access_token = response_json['tenant_access_token']
        return tenant_access_token
    else:
        error_message = response_json['msg']
        raise Exception(f"Failed to retrieve tenant_access_token. Error: {error_message}")

# 发送消息
def send_message(token, receive_id, project_name, at_body ,is_group):
    type = 'chat_id' if is_group else 'open_id'
    url = 'https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type='+ type
    headers = {
        'Content-Type': 'application/json; charset=utf-8',
        'Authorization': 'Bearer ' + token
    }
    data = {
        "receive_id": receive_id,
        "msg_type": "interactive",
        "content": "{\"type\": \"template\", \"data\": { \"template_id\": \"" + TEMPLATE_ID + "\", \"template_variable\": {\"project_name\": \"" + project_name + "\",\"time\": \"" + time + "\",\"person\": \"" + at_body + "\"} } }",
        "uuid": ""
    }

    response = requests.post(url, headers=headers, json=data)
    response_json = response.json()
    print(response_json)
    if response_json.get('code') == 0:
        print("Message sent successfully.")
    else:
        error_message = response_json.get('msg')
        raise Exception(f"Failed to send message. Error: {error_message}")


def main():

    # 设置相关变量...
    create_user = sys.argv[1]
    project_name = sys.argv[2]
    
    # 调用函数获取 tenant_access_token
    try:
        # 获取TOKEN
        token = get_tenant_access_token()
        print("tenant_access_token:", token)

        # 1 发送到群 chatID
        receive_id = 'oc_1212121'#群
        AT_BODY = get_user_at(create_user, True)
        send_message(token, receive_id, project_name, AT_BODY, True)
        # 2 发送到个人 openID
        receive_id = get_user_openid(create_user)
        AT_BODY = get_user_at(create_user, False)
        send_message(token, receive_id, project_name, AT_BODY, False)
    
    except Exception as e:
        print("Error:", str(e))

if __name__ == "__main__":
    main()
python message_sender.py 张三 项目名称
```

### **步骤总结**

```Bash
1. 安装Jenkins：您需要在您的服务器或本地机器上安装Jenkins。您可以从Jenkins官方网站（https://www.jenkins.io/）下载适合您操作系统的安装程序，并按照说明进行安装。

2. 配置Jenkins：一旦安装完成，您可以通过在浏览器中输入Jenkins的URL来访问Jenkins的Web界面。配置账号密码权限。

3. 安装所需插件：通用插件和特定插件例如Git Git Parameter Plug-In maven build user vars plugin

4. 创建一个新的Jenkins任务

5. 配置：丢弃旧构建，设置Git参数插件参数 GIT_BRANCH ${GIT_BRANCH} 源代码管理
        - 注意事项main master git凭证公钥还是私钥，公开的仓库用公钥，私有仓库用私钥
6. 配置构建步骤：使用Maven构建并指定您要执行的Maven命令（例如clean package）。这里是后端 前端不需要这些东西
         - 需要配置全局配置地址         
7. 配置构建后操作：
        -        方式一：scp把压缩包放到服务器上，运行服务器脚本解压运行项目
        - 方式二：使用Docker镜像传递，再去服务器拉取镜像运行
8.飞书通知
        添加机器人，通知消息，设置消息模版
        机器人分webhook和应用机器人，应用机器人需要用key secret去获取token webhook直接post请求即可
        脚本解析，通过创建人发送到脚本，根据创建人去获得openID，使用机器人发送到个人，同时发不到群并且 AT
```

### **以下是我遇到问题后使用ChatGPT解决的方式**

#### **1.服务器连接没有设置公钥，询问如何设置公钥访问**

**问:用账号密码链接上之后 如何添加公钥到服务器**

**答一旦通过用户名和密码连接到服务器，您可以按照以下步骤将您的公钥添加到服务器上：**

1. 在您连接到服务器的终端中，运行以下命令来创建SSH目录（如果尚不存在）：
2. mkdir -p ~/.ssh
3. 运行以下命令将您的公钥内容添加到`~/.ssh/authorized_keys`文件中：
4. echo "YOUR_PUBLIC_KEY_CONTENT" >> ~/.ssh/authorized_keys
5. 在上面的命令中，将`YOUR_PUBLIC_KEY_CONTENT`替换为您的公钥内容。您可以直接粘贴整个公钥内容，并确保整个公钥都被引号包围。
6. 例如：
7. echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCx9Ck0... [公钥的其余部分]" >> ~/.ssh/authorized_keys
8. 运行以下命令以确保`~/.ssh/authorized_keys`文件具有正确的权限：
9. chmod 600 ~/.ssh/authorized_keys
10. 这将设置`authorized_keys`文件的权限，以确保只有所有者（您）具有读写权限。
11. 现在，您的公钥已成功添加到服务器的`authorized_keys`文件中，允许您使用SSH密钥进行身份验证。
12. 您可以尝试从另一个终端或窗口使用SSH密钥连接到服务器，这次将不再需要使用用户名和密码。
13. ssh username@server_ip_address
14. 替换`username`为您的用户名，`server_ip_address`为服务器的IP地址或主机名。
15. 如果您的SSH密钥配置正确，并且服务器上的`authorized_keys`文件包含您的公钥，您将无需输入密码即可连接到服务器。

请注意，添加公钥到服务器时，请确保您在控制台的安全环境下进行操作，并确保仅将正确的公钥添加到服务器上。

#### **2.没有安装Git，安装Git**

**问：centos 安装git**

**答：在CentOS上安装Git，您可以按照以下步骤进行操作：**

1. 打开终端。
2. 运行以下命令以更新系统软件包：
3. sudo yum update
4. 运行以下命令以安装Git：
5. sudo yum install git
6. 在安装过程中，您可能需要输入您的用户密码以授予管理员权限。然后，系统将自动下载和安装Git软件包。
7. 完成安装后，运行以下命令以验证Git安装：
8. git --version
9. 如果一切顺利，您将看到安装的Git版本号。

#### **3.查看服务器是否安装Jenkins**

**问：如何查看服务器是否安装Jenkins**

**答：运行以下命令来检查是否安装了Jenkins**

systemctl status jenkins

如果Jenkins已经安装在服务器上，命令的输出将提供有关Jenkins服务的状态信息，包括正在运行的进程信息和服务状态。

如果Jenkins未安装，则会显示相应的错误信息或提示。

如果您无法使用 `systemctl` 命令，您可以尝试以下替代方法之一：

service jenkins status

或

ps -ef | grep jenkins

这些命令将提供类似的信息，以判断Jenkins是否已安装并正在运行。

请注意，上述命令需要具有足够的权限才能执行。如果您无法检查Jenkins的安装状态，请与服务器管理员或负责维护该服务器的人员联系以获取支持。

另外，您还可以尝试通过在Web浏览器中访问服务器的Jenkins地址（默认为 `http://server_ip_address:8080`）来确认Jenkins是否安装在服务器上。如果成功访问Jenkins的Web界面，则表明Jenkins已经安装并运行。

#### **4.获取服务器公钥**

`cat ~/.ssh/id_rsa.pub`查询后发现没有生成SSH密钥对。

**问：CentOS服务器上如何生成SSH密钥对**

**答：要在服务器上生成SSH密钥对，您可以按照以下步骤操作：**

1. 连接到您的服务器。例如，在命令行中运行以下命令：
2. ssh user@server_ip_address
3. 将 "user" 替换为您的用户名，"server_ip_address" 替换为服务器的IP地址或主机名。根据您的服务器配置和访问权限，您可能需要提供用户密码或SSH密钥来进行身份验证。
4. 连接成功后，在服务器上运行以下命令来生成SSH密钥对：
5. ssh-keygen -t rsa -b 4096
6. 这将提示您选择密钥文件的保存路径和名称。如果您不需要更改默认路径和名称，可以直接按Enter键继续。否则，根据需要提供新的路径和名称。
7. 在提示符下，您可以选择是否为密钥对设置密码。设置密码可以增加安全性，但在每次使用密钥时都需要输入密码。如果您选择设置密码，请提供一个安全且易于记住的密码，并确保妥善保管。
8. 完成上述步骤后，将会生成SSH密钥对。私钥文件的默认路径是 `~/.ssh/id_rsa`，而公钥文件的默认路径是 `~/.ssh/id_rsa.pub`。
9. 您可以使用以下命令查看公钥的内容：
10. cat ~/.ssh/id_rsa.pub
11. 将公钥内容复制到剪贴板，并在需要的地方使用它，例如添加到其他服务器的 "authorized_keys" 文件中。

请注意，上述步骤中的 `~/.ssh/id_rsa` 和 `~/.ssh/id_rsa.pub` 是默认的SSH密钥路径和文件名。如果您希望使用不同的密钥文件或自定义路径，请相应地更改命令中的文件路径和名称。

希望这可以帮助您在服务器上生成SSH密钥对！