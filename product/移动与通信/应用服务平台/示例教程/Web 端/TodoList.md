本文档介绍如何快速搭建一个Todo List 应用，无需域名、无需服务器，通过网络同步您的 Todo List 数据，在多个设备之间实时共享。最终成型的应用展示如下：
![](https://main.qcloudimg.com/raw/e1bf378595fe93a37bbbcb5158a4e01a.png)

## 准备工作

1. 请 [注册腾讯云账号](https://cloud.tencent.com/register?s_url=https%3A%2F%2Fcloud.tencent.com%2F)，注册成功后即可使用腾讯云服务，已注册可忽略此步骤。
2. 登录腾讯云 [云开发控制台](https://console.cloud.tencent.com/tcb)，您在填写环境名称之后，单击【下一步】，直接提交授权并使用云开发服务。
![](https://main.qcloudimg.com/raw/a93584a3b38cbd035501b0f3b8ac2d56.png)
如果您之前创建过环境，可以继续使用已创建的**按量计费**环境，或者再次【新建环境】。
![](https://main.qcloudimg.com/raw/68f9e9836035f548aa840ad1c2a17a77.png)
3. <span id="step1.3"></span>开通成功之后，单击环境名称，进入环境总览页面，如下所示：
![](https://main.qcloudimg.com/raw/9f26a3f509467bad7c4f003099c61356.png)
> !请记住您的环境 ID，这个 ID 在后续步骤将被使用。您可单击**环境Id**右侧的【<img src="https://main.qcloudimg.com/raw/a06f957521023a64e977041f9181f251.jpg"  style="margin:0;">】图标进行复制。

## 步骤1：建立 Todo List 文件
1. 在本地新建文本文件，在文件中填入如下内容：
```js
<script src="https://acc.cloudbase.vip/todo.js?asa" charset="utf-8"></script>
<script src="https://acc.cloudbase.vip/login_util.js" charset="utf-8"></script>
<script src="https://imgcache.qq.com/qcloud/tcbjs/1.10.8/tcb.js"></script>
<script>
    let uid = null;                                         //用户唯一身份id
    const app = tcb.init({                                  //初始化云开发SDK
        env:"${envId}"                               //传入环境ID，
    })
    const auth = app.auth({                                 //加载权限对象
        persistence: "local"                                //类型：30天内不失效
    });
    const db = app.database();                              //加载数据库对象
    window.onload = function () {                           //页面加载完成，执行
        LO.init();                                          //引入的登录模块初始化加载
        TODO.init();                                        //引入的Todo应用加载
    }
    LO.Done = function(){                                   //登录成功后触发此函数
        uid = app.auth().hasLoginState().user.uid;          //获取用户的唯一身份id
        db.collection('todo').doc(uid).get().then(res=>{    //从数据库中查找该用户数据
            if(res.data.length==0){                         //如果没有找到数据，长度为0
                db.collection('todo').add({                 //数据库添加记录
                    _id:uid,                                //记录文档id为用户唯一身份id
                    list:TODO.todo,                         //TODO.todo为引入应用模块的待办事项数据
                    time:new Date()                         //取当前时间
                }).then(res=>{                              //文档添加成功
                    console.log(res);                       //打印返回数据
                    watchtodo();                            //执行数据库监听
                })
            }
            else{                                           //数据存在
                console.log(res);                           //打印返回数据
                TODO.todo = res.data[0].list;               //取出网络todo字段，更新本地应用数据
                TODO.todoinit();                            //重新构建待办事项的展示
                watchtodo();                                //执行数据库监听
            }
        });
    }
    TODO.itemChange = function(id,type,des){                //当待办事项有变化时触发
        db.collection('todo').doc(uid).update({             //更新数据库中文档记录
            list:db.command.set(TODO.todo),                 //重新设定todo为最新数据
            time:new Date()                                 //取当前时间
        }).then(res=>{
        }).catch(e=>{
            console.log(e);                                 //打印日志
        })
    };
    function watchtodo(){                                   //监听数据库
        db.collection('todo').where({ _id:uid }).watch({    //监听当前用户的记录
            onChange: (snapshot) => {                       //当数据变更时
                if(snapshot.msgType!="INIT_EVENT"){         //当数据不是监听初始加载
                    TODO.todo = snapshot.docs[0].list;      //取出网络todo字段，更新本地应用数据
                    TODO.todoinit();                        //重新构建待办事项的展示
                }
            },
            onError: (error) => {                           //当监听失败时
                alert('远端数据库监听失败！');                 //弹出提示框
            }
        });
    }
</script>
<html>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <head>
        <title>TODO应用</title>
    </head>
    <body>
        <div id="model">
            <input id="text-in" type="text" placeholder="写下你的待办事项…">
            <ul id="todo-list"></ul>
        </div>
    </body>
</html>
```
2. 保存文本文件，并将后缀改为`html`，命名为`index.html`。

## 步骤2：托管静态文件
为了更多人可以访问 Todo List 应用，可以使用云开发静态网站托管功能，云开发提供默认域名，可使用公网进行访问。
1. 替换本地 `index.html` 文件中的 `${envId}` 为您的云开发环境 ID，在文件第七行：`env:"${envId}"`。
2. 打开[云开发控制台](https://console.cloud.tencent.com/tcb/env/index)，进入在 [准备工作](#准备工作) 中已经创建好的**按量计费**环境。
3. 进入云开发控制台的 [静态网站托管](https://console.cloud.tencent.com/tcb/hosting)，点击上传文件，将 [步骤1](#步骤1建立-todo-list-文件)中的`index.html`文件上传。
    ![](https://main.qcloudimg.com/raw/88b1196eefce81d9c811b1f9b8f03df6.png)
4. 上传完毕后，点击**配置信息**中的**默认域名**，在浏览器中打开该链接，即可在公网环境下访问 Todo List网站。
   ![](https://main.qcloudimg.com/raw/8f65d2050d58281599fbc6e5b098ec37.png)
> ?默认域名可供您快速验证业务，如您需要对外正式提供网站服务，请前往【基础配置】绑定您已备案的自定义域名。

## 步骤3：创建数据库
完成以上步骤后，Todo List 应用内的数据还存储在本地，无法实现跨设备同步。接下来，您将使用云开发的**数据库**服务，实现数据同步功能。
1. 进入云开发控制台的 [数据库](https://console.cloud.tencent.com/tcb/db) 中，新建集合`todo`，如下图所示：
![](https://main.qcloudimg.com/raw/1711b25838642bd77d4637b186a91c98.png)
之后 Todo List 内的数据便会存储在这个集合中。

## 步骤4：配置邮箱登录
1. 为了实现邮箱验证登录功能，进入云开发控制台【环境】菜单内的[登录授权](https://console.cloud.tencent.com/tcb/env/login)中，开启**邮箱登录**，如下图所示：
![](https://main.qcloudimg.com/raw/4de7c150eb145458c50f29266c78c720.png)
2. 开启后点击【配置发件人】，参考[配置QQ邮箱](https://docs.cloudbase.net/authentication/email-login.html#shi-yong-qq-you-xiang-pei-zhi-you-xiang-deng-lu)进行配置。
   ![](https://main.qcloudimg.com/raw/45d61506ed0c1a5f8eab98a06827fee4.png)
3. 发件人配置成功后，点击【应用配置】，填写应用名称`Todo`。
   ![](https://main.qcloudimg.com/raw/03f9580539c2b88e6794dca9d32ce0f8.png)

## 步骤5：应用登录
1. 进入云开发控制台的[静态网站托管](https://console.cloud.tencent.com/tcb/hosting)，点击**配置信息**中的**默认域名**，在浏览器中打开该链接。打开链接后需要进行用户登录。
   ![](https://main.qcloudimg.com/raw/2f3deb55a5ccaa5d73950b5ad1bfc8c1.png)
2. 在登录模块中输入邮件地址和密码，如果邮箱未注册会发送注册邮件到邮箱，点击激活链接进行注册。
   ![](https://main.qcloudimg.com/raw/21807a6e1bf0965ee878c0f60de5cb9d.png)
3. 返回应用网站，10s后按钮变为可点击状态，点击登录，即可登录成功。之后，通过邮箱地址和密码即可完成之后登录。
   ![](https://main.qcloudimg.com/raw/287b3a222683daabbe3975a28f35e013.png)

## 步骤6：记录待办事项
至此，您已经成功创建了一个在线同步的 Todo List 网页应用，最终的效果图如下：
![](https://main.qcloudimg.com/raw/e1bf378595fe93a37bbbcb5158a4e01a.png)

## 补充说明

本实战项目通过模块化方式构建，主要突出使用云开发时的开发步骤，更加直观。如果您想探寻 Todo List 模块的内容，可以自行下载解读[代码](https://github.com/TCloudBase/WEB-TodoList)。

>? `login_util`模块，是一个简易登录插件，可以实现简单的登录操作，提供自定义方法，默认使用云开发的**邮件登录**方式，所以在无自定义登录方式时请保证邮件登录配置正确并打开。

1. `todo.js`暴露接口：
```js
TODO.todo;                          //待办事项内容json，可按照规则直接改变
TODO.todoinit();                    //刷新显示待办事项
TODO.itemChange(id,type,des);       //监听待办列表变化[id，类型，描述]
```
2. `login_util`暴漏接口：
```js
LO.close();                         //关闭登录框
LO.onClose = function(){};          //监听登录框关闭
LO.onLogin = function(obj){};       //监听登录按钮，需LO.custom=true才可生效
LO.setBtn(text,disable);            //设置登录按钮[text:文案,disable:是否可用，默认true]
LO.setDes(text,style);              //设置描述[text:文案,style:css样式]
LO.custom;                          //是否自定义登录操作，默认false为邮件登录
```
