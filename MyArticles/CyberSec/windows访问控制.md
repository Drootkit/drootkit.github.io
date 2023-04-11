# 令牌

## Windows Access Token

访问令牌，描述进程或者线程安全上下文的一个对象。

每个用户登录计算机都会产生一个**AcessToken**以用于创建进程和线程，用户注销以后会将主令牌切换成模拟令牌，也就是授权令牌和模拟令牌，不会清除令牌，只有重启才会。

## 分类

令牌都是在登录时产生的，就像是自助餐入场券一样，你在里面睡觉就会从授权令牌变成虚拟令牌，只有当你离开店铺的时候，才会失效，重新进来又需要新的入场券。

具有Delegation token的用户在注销后，该Token将变成Impersonation token，依旧有效。

### 授权令牌

Delegation token(授权令牌):用于交互会话登录(例如本地直接登录、rdp login 、psexec )

### 模拟令牌

Impersonation token(模拟令牌):用于非交互登录(利用net use访问共享文件夹，或者wmi、winrm等等）

## TOKEN产生

每个进程创建时都会根据登录会话权限由LSA(Local Security Authority)分配一个Token。

如果CreaetProcess时自己指定了Token, LSA会用该Token，否则就继承父进程Token进行运行

### 组成

- 用户（User）。用户账号的SID。若用户登录到本地计算机上的一个账号，则他的 SID来自于本地SAM维护的账号数据库；若用户登录到一个域账号，则他的SID来自于活动目录里用户对象的Object-SID属性。
- 组（Groups）。包含该用户的安全组的SID列表，表中也包含代表活动目录里用户 账号的用户对象的SID-History属性里的SID。
- 特权（Privileges）。用户和用户的安全组在本地计算机上拥有的特权列表。
- 所有者（Owner）。特定用户或安全组的SID，这些用户或安全组默认成为用户所 创建或拥有的任何对象的所有者。
- 主组（Primary Group）。用户的主安全组的SID。这个信息只由POSIX子系统使用， Windows 2000的其他部分对其忽略。
- 默认任意访问控制表（Default Discretionary Access Control List, DACL）。一组内置 许可权。在没有其他访问控制信息存在时操作系统将其作用于用户所创建的对象。默认DACL向创建所有者和系统赋予完全控制（Full Control）权限。
- 源（Source）。导致访问令牌被创建的进程，例如会话管理器、LAN管理器或远程 过程调用（RPC）服务器。
- 类型（Type）。指示访问令牌是主（primary）令牌还是模拟（impersonation）令牌。 主令牌代表一个进程的安全上下文；模拟令牌是服务进程里的一个线程，用来临时接受一个不同的安全上下文（如服务的一个客户的安全上下文）的令牌。
- 模拟级别（Impersonation Level）。指示服务对该访问令牌所代表的客户的安全上下 文的接受程度。
- 统计信息（Statistics）。关于访问令牌本身的信息。操作系统在内部使用这个信息。
- 限制SID（Restricting SID）。由一个被授权创建受限令牌的进程添加到访问令牌里 的可选的SID列表。限制SID可以将线程的访问限制到低于用户被允许的级别。
- 会话ID（Session ID）。指示访问令牌是否与终端服务（Terminal Services）客户会 话相关。

token具体的实例可以用windbg查看 `!exts.token -n d1bed030`





# 权限提升