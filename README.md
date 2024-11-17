## Online/Offline coexist ideas
Minecraft 服务器实现正版离线互通的思路


## 服务端的实现

服务端区分正版与离线模式的核心是 **Yggdrasil 验证服务器** 的使用. 对于正版玩家, 服务端必须通过与 Mojang 的会话服务器验证玩家身份；而对于离线玩家, 服务端无需与 Mojang 通信, 直接生成离线 UUID. 

**正版验证流程：**

- **玩家连接正版服务器时**, 服务端会调用 `hasJoinedServer` 方法, 向 Mojang 的 Yggdrasil 验证服务器查询玩家信息 (例如 UUID、名称、皮肤等) . 
- 如果 Mojang 验证成功, 返回玩家的详细信息, 服务端将为玩家设置正确的 UUID 和其他相关信息. 
- **验证失败** (例如凭证无效、会话服务器不可用等) 时, 服务端拒绝玩家连接. 


**互通处理:**

![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/1.png)

- 上图代码中try块会捕获与验证服务器进行连接时抛出的异常, 当离线玩家进入时, 即便没有抛出那些异常, 也会因无法正确获取玩家信息而返回空对象.

- 所以只需要为离线玩家做离线服务端一样的事情即可, **故为玩家设置生成好的离线信息**.
    
![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/2.png)

这样就能够让离线玩家进入正版服务端.


## 客户端的实现
客户端判断服务器是否为正版服务器, 主要是通过与 **Yggdrasil 验证服务器** 通信来验证身份. 
如果服务器是正版服务器, 客户端将需要提供有效的登录凭证 (`accessToken` 和 `profileId`), 否则客户端会主动断开连接. 

如果是局域网列表中的服务器, 则不会主动断开连接.

如果是离线服务器, 将不显示TAB中的头像

**正版验证流程：**

- 客户端连接服务器时, 会检查服务器是否启用正版验证 (即 `online_mode=true`) . 
- 客户端会通过 `joinServer` 方法将玩家的凭证提交给 **Yggdrasil 验证服务器**, 如果凭证无效 (如过期或伪造), 客户端会断开连接. 

**互通处理:**

![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/3.png)

- 代码中调用`makeRequest`方法时当凭证无效后会抛出异常, 进而中断连接, 所以只需要将它放到try块中捕获异常即可.

![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/4.png)

- 但我们无法从服务端去更改客户端的java类, 这也正是为什么没有正版离线互通插件的原因.

- 而网易就不一样, 网易修改了客户端的`authlib.jar`, 客户端基本不会与 **mojang的Yggdrasil验证服务器** 进行通信, 并且默认允许加入全部的正版服务端.

- 难到就没有办法了吗? 有! 我们可以使用**外置验证服务器** (如`LittleSkin`) **拿到合法的登录凭证, 即可防止客户端主动断开连接**.
  
![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/5.png)

<br/>

***以上若有错误, 烦请各位指出!***

<br/>

***- 作者:** Koud_Wind*
