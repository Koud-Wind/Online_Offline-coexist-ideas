## Online/Offline coexist ideas
Minecraft 服务器实现正版离线互通的思路


## 对于服务端
  当玩家进入正版服务端时, 将调用`hasJoinedServer`方法, 
  服务端会主动与Yggdrasil验证服务器进行通信, 根据会话服务器尝试验证合法性并获取玩家信息 (UUID, 名称, 皮肤, 披风等).
  
  成功获取后为玩家设置这些信息并允许加入, 
  **如果`会话不合法/验证服务器中没有玩家信息/验证服务器宕机`, 那么服务端就拒绝此玩家加入.**

  而离线服务端就不看这些, 仅仅为玩家生成离线UUID并设置玩家名称, 故没有皮肤信息.

  ![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/1.png)

  上图代码中try块会捕获与验证服务器进行连接时抛出的异常, 当离线玩家进入时, 即便没有抛出那些异常, 
  也会因无法正确获取玩家信息而返回空对象.

  所以我们只需要为离线玩家做离线服务端一样的事情即可, 故为玩家设置生成好的离线信息.
    
  ![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/2.png)

  这样就能够让离线玩家进入正版服务端.


## 对于客户端

  在客户端眼中, 是会区分正版服务器与离线服务器的 (离线服务器不显示头像), 体现在与服务器建立连接初期, 将调用`joinServer`方法.
  
  与服务端进行初次握手时会判断服务端是否开启了正版验证, 
  若开启了就需要向mojang的Yggdrasil验证服务器提交登录凭证并拿到会话ID提交到正版服务器, 
  **若登录凭证不合法或没有凭证会主动断开连接** (而尝试加入局域网列表中的服务器就不会).

  ![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/3.png)

  代码中调用`makeRequest`方法时当凭证不合法后会抛出异常, 进而中断连接, 
  所以只需要将它放到try块中捕获异常即可.

  ![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/4.png)

  但我们无法从服务端去更改客户端的java类, 这也正是为什么没有正版离线互通插件的原因.

  而网易就不一样, 网易修改了客户端的authlib, 客户端基本不会与mojang的Yggdrasil验证服务器进行通信, 
  并且默认允许加入全部的正版服务端.

  难到就没有办法了吗? 有! 我们可以使用外置验证服务器 (如LittleSkin) 拿到合法的登录凭证, 
  即可防止客户端主动断开连接.
  
  ![](https://github.com/Koud-Wind/Online_Offline-coexist-ideas/blob/main/5.png)

<br/>

***以上若有错误, 烦请各位指出!***
























