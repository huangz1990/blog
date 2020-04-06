使用 V2Ray 代理访问外网
=======================================

随着墙的更新与升级，
之前一直使用的 Shadowsocks 代理最近开始无法访问，
给日常的工作和生活带来了很大的麻烦。
为了解决这个问题，
近日将自己的网络代理从 Shadowsocks 换成了 V2Ray 。

本文将简单地介绍在 Manjaro 系统上使用 V2Ray （图形界面为Qv2ray）代理访问外网的方法，
此方法也适用于其他 Linux 发行版以及 Windows 、macOS 等系统。

.. image:: image/v2ray_logo.png

步骤
----------

1. 访问 https://github.com/v2ray/v2ray-core/releases ，
   根据你使用的系统下载并解压相应的 v2ray-core （本例中为 ``v2ray-linux-64.zip`` ）。

2. 同样地，
   访问 https://github.com/Qv2ray/Qv2ray/releases ，
   根据你使用的系统下载并解压相应的 Qv2ray 
   （本例中为 ``Qv2ray.v2.1.2.linux-x64.qt5.13.2.AppImage`` ）。

3. 执行命令 ``chmod +x Qv2ray.v2.1.2.linux-x64.qt5.13.2.AppImage`` ，
   为 Qv2ray 赋予执行权限。
   （macOS 和 Windows 可能也需要执行类似的操作，未测试，不确定。）
   
4. 运行 ``Qv2ray.v2.1.2.linux-x64.qt5.13.2.AppImage`` ，
   看到下图所示的界面，
   点击上方的“Preferences”按钮打开配置菜单。

   .. image:: image/v2ray-main.png 

5. 在“General Setting”标签页的“V2ray Settings“设置中指定 v2ray-core 文件的路径及其目录，
   如下图所示（具体细节根据你的实际情况进行修改）。
   设置完毕之后请通过点击“Check V2ray Core Settings“按钮来检查设置是否成功。

   .. image:: image/v2ray-general.png

6. 在“Inbound Settings”标签页设置代理选项，
   如下图所示（具体细节根据你的实际情况进行修改）。

   .. image:: image/v2ray-proxy.png

7. 修改完配置之后，
   点击“OK”保存并回到主界面。

8. 点击主界面的“Subscriptions”按钮，
   在弹出界面的左下方点击“Add Subscription”，
   新建一个订阅。
   接着在右半边订阅界面的“Subscription Address”框中填入你的 V2Ray 服务商提供的订阅地址，
   然后点击下方的“Update Subscription Data”以取得最新的服务器地址。
   最后再次点击“OK”按钮，
   回到主界面。

   .. image:: image/v2ray-sub-data.png

9. 在主界面的“Host List”中选择一个距离你较近的服务器，
   然后点击上方的“Connect”按钮来连接服务器
   （也可以右键点击服务器然后选择“Connect to this”）。

   .. image:: image/v2ray-connect.png
 
10. 配置你的软件或者浏览器，
    让它们通过 V2Ray 而不是默认的网络来进行访问
    （比如为Chrome浏览器安装并设置SwitchyOmega插件）。

11. 大功告成！

| 因为无法正常访问GitHub.com而苦恼不已的黄健宏
| 2020.2.15
