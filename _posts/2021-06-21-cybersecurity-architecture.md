---
layout: post
title:  "cybersecurity"
comments: true
categories: security
---
本文分两个部分，安全知识体系一章以CIA Triad为核心演绎构建安全知识体系；具体应用一章详述一些流行的安全协议。

# 1 安全知识体系(cybersecurity knowledge architecture)
安全方面的技术内容涉及众多数学理论、通讯协议、技术应用，为方便澄清，我们围绕CIA Triad，结合IATF(Information assurance technical framework)的部分观点，构建一个概要性的安全知识体系。

## 1.1 CIA Triad
CIA Triad是Confidentiality, Integrity, Availability的合称：

![CIA-triad](/assets/2021-06-21-cybersecurity-architecture/CIA-triad.png)

三者即有一定的独立性，又相互影响：比如私密性中的源认证（source authentication）是实现完整性的前提；只有保证了私密性和完整性，才可能保证系统的可用性。

具体实践上，贯穿CIA始终的核心议题是访问控制（access control），即对访问access涉及的所有对象进行定义表示（representing）和认证（authenticating），其中最关键对象为身份（identity）和授权（authentication），其他对象还包括访问的contextual information，比如time，place，status等。

下面我们先从最具象的Access control着手，继而分析Confidentiality，Integrity，Availability，以及Non-repudiation，绘制出相关理论知识体系。

## 1.2 Access control
Access can actually be composed of a complicated set of parameters. For example, a particular access might be, “007 Execute a file labeled top secret at 3:15 p.m. during a time of war.”换句话说，access实际上是a set of information that must be represented, communicated, and authenticated, including contextual information, such as time, places, status, or current conditions. 本例包括多个方面的访问控制：
1. 需要对编号为007的人的身份identity进行认证
2. 需要对编号为007的authorization进行认证，确保其权限privilege不低于所访问实体（即a file）的安全等级（即top secret）
3. 需要对活动的时间进行确认（3:15 p.m. during a time of war）

为此，我们将access control分为四个阶段：
1. 对身份的定义和认证，即identity & Authentication
2. 对authorization的定义和认证，即authorization & Authentication
3. 基于身份或者authorization做出decision，是approve还是deny
4. 真正的执行访问，即enforce access，这个阶段要考虑到除了identity和authorization之外的其他contextual information。

下面重点详述前三阶段。

### 1.2.1 Identity & Authentication
Identity & Authentication是对身份进行认证，确保通讯的参与者符合预期。这涉及到两方面议题，一是identity的表示和绑定，即我们如何表示通讯实体entity的identity，以及如何将其与该entity关联起来；二是通讯communication和认证authenication，即将identity传输给对端peer后，对端如何进行验证authenticating。

可以针对三类对象实施认证：你所知道的（sth you know），你所有的（sth you have），以及你所是的（sth you are)。如果认证涉及多种类型，我们就称其为多因子认证，比如2FA（2-factor authentication）。

![authentication-category](/assets/2021-06-21-cybersecurity-architecture/authentication-category.png)

#### 1.2.1.1 静态密码
静态密码是单因子认证对象，即sth you know。

静态密码根据你所知道，但其他人不知道这个特性，来验证你的identity。你在网站上注册账户时录入的密码，使用物理token时需要提供的PIN（即Personal Identification Number）都是静态密码。

静态密码需要人的大脑记忆。为图方便，很多人在多个网站上都使用相同的静态密码，从而增加了密码泄露的风险。

#### 1.2.1.2 动态密码
动态密码是单因子认证对象，即sth you have。

动态密码就是OTP（One-Time Password），每个密码只使用一次，因此原则上来说比静态密码安全。

显然，人的大脑记不住OTP，只能**由特定的设备实时生成**。实践上，OTP有两种生成和使用方式：
1. 同步方式。客户端的物理token和authentication server共享同一个secret key。双方都使用这个key与timestamp按照特定算法生成OTP。显然，这种模式要求客户端和服务端的时间保持同步，至少漂移值在限定范围内。
2. 异步方式。客户端的物理token和authentication server共享同一个secret key。使用challenge/response机制：服务端先随机发送一个challenge给客户端，用户将这个challenge输入到物理token中，将该challenge与token中的key一起计算得到一个OTP，客户端再将这个OTP作为response发送给服务端进行验证。

无论哪种情况：

1. 掌握该物理token（sth you have）的用户即可生成有效OTP，从而实现身份上的认证。
2. 物理token需要与authentication server共享同一个secret key，因此**物理token是站点特定的**，也即是说，在单纯的OTP模式下，物理token的scalability存在限制。这一点可比对第二部分FIDO2中的authenticator进行理解。

#### 1.2.1.3 公私钥
公私钥是一个密钥对，两者不同但配对：采用私钥加密的密文，经公钥解密可以得到明文。

一种典型的应用场景采用challenge/response机制：
1. server端发送一个随机串S作为challenge
2. 用户使用私钥对其进行签名（先计算S的摘要d，然后用私钥加密d得到签名）然后作为response返回给server端
3. server端使用用户的公钥进行解密得到d', 与本地计算的S的摘要d''进行比较，实现authentication。

> **说明**
> 
> 基于私密性confidentiality的考虑，我们一般将私钥封存在密钥库中保护，密钥库即可能是一个密码保护的加密文件，也可能是一个PIN码或者指纹等生物特征保护的物理封禁的加密硬件设备（比如USB，NFC近场通讯设备，或者BLE蓝牙设备），注意这里实际用到了两类identity信息，一个是PIN码或指纹，对应sth you know或sth you are，一个是密钥库文件或硬件设备，对应sth you have。因此这种情况本质上是2FA（2-Factor Authentication）。
> 
> 用户的公私钥对生成和分发有两种途径：
> 1. 用户生成密钥对，然后通过注册流程将公钥传递给Server端保存。比如FIDO2中的注册。
> 2. 服务端生成密钥对，然后将私钥封装在物理token中，再通过线下途径分发给用户。类似银行提供的U盾。

下面简单介绍公私钥体系中的两个常见概念：摘要和签名。

the message digest is the hash of the message，a fixed size numeric representation of the contents of a message，目的就是确保消息的完整性integrity，防止恶意的篡改，其特点有二：
1. 单向。One-way，not be possible to reverse the function to find the message corresponding to a particular message digest
2. 碰撞几率足够小。must be computationally infeasible to find two messages that hash to the same digest.

Message digests are encrypted with private keys creating a digital signature. 使用自己的私钥对消息摘要进行加密即称为数字签名。
> 签名的目的并不是为了保证私密性confidentiality，因为使用私钥加密的成本（比如CPU消耗）很高，如果对一个很长的文本进行加密签名，将耗费大量的CPU时间，不具有实践意义。保证私密性的一般方法是使用对称密钥进行加密，它在安全性和计算成本之间保持了平衡。
> 签名的目的是为了实现身份认证以及支持不可抵赖性non-repudiation，因为只有拥有私钥才能进行签名，而其他人无法给出有效的消息签名。

一个实际的案例如下：
1. Question。Alice和Bob各有一对密钥，记为$A_{pub}/A_{priv}$, $B_{pub}/B_{priv}$, Alice和Bob都知道对方的公钥，但私钥只有自己知道。Alice想向Bob秘密的传递一个消息，需要解决两个问题，首先消息要加密，其次Bob要确认消息确实是Alice发送的。
2. 初始设想：
   1. Alice想发送消息m给Bob。一般来说就是Alice用$B_{pub}$直接加密m。这样Bob就用能他自己的$B_{priv}$解密得到m
   2. Alice想向Bob证明是她发送了m。一般来说就是Alice用$A_{priv}$加密m，这样Bob就能用$A_{pub}$解密得到m
   > 显然，Alice和Bob之间只需要实施b即可，但实际上，使用非对称的密钥加密的计算量非常大。所以人们就想了一点办法，结合密钥交换、对称加密和消息摘要来降低计算量。
3. 改进流程
   1. Alice先对m做摘要，得到固定长度的$d_A$。
   2. Alice用$A_{priv}$加密$d_A$，得到签名sig
   3. Alice生成一个规定长度的随机密钥$k_{sym}$，并交换给Bob。以便实施对称加密
      - Alice用Bob的$B_{pub}$加密$k_{sym}$, 将结果$E_{k_{sym}}$发送给Bob
      - Bob用自己的私钥$B_{priv}$加密$E_{k_{sym}}$，还原得到$k_{sym}$
   4. Alice用$k_{sym}$对称加密`m|sig`，得到c
   5. Bob用$k_{sym}$解密c，得到`m|sig`
   6. Bob用$A_{pub}$解密sig，得到消息摘要$d_A$，然后本地计算m的摘要得到$d_B$，如果$d_A = d_B$, 则说明m没有被篡改。
   > 实际上，为了减少传输，Alice在生成$E_{k_{sym}}$后，并不会立刻发送给Bob，而是在步骤4中得到c后，将两者(c和$E_{k_{sym}}$一并传给Bob。Bob会先解开$E_{k_{sym}}$得到$k_{sym}$,再用$k_{sym}$解密c得到`m|sig`。

#### 1.2.1.4 HMAC
Hash-based Message Authentication Code，或者keyed-hash message authentication code，即基于哈希的消息认证码，它要求通信双方共享密钥、约定算法、对报文进行Hash运算，形成固定长度的编码。

其通讯过程如下：
1. 通讯双方提前通过其他机制交换共享密钥k。
2. 客户端结合消息m和密钥k，进行hash运算得到h，然后将`m|h`发送给服务端
3. 服务端收到消息，结合消息m和密钥k重新进行hash运算得到h'。如果h'和h一致，则表明消息是完整的，没有被篡改。

从上述过程可以看出，因为非法的对端并不知道共享密钥k，所以通过验证某个消息HMAC值的一致性，即可验证对端的身份。这是HMAC的一个重要派生用途，所以它直接就叫做验证码（Message Authentication Code）。

HMAC can provide message authentication using a shared secret instead of using digital signatures with asymmetric cryptography. It trades off the need for a complex public key infrastructure by delegating the key exchange to the communicating parties, who are responsible for establishing and using a trusted channel to agree on the key prior to communication.也就是说，HMAC没有使用非对称加密算法，而是使用共享密钥，这避免了对复杂的PKI基础设施的依赖，但通讯双方必须提前解决好共享密钥的交换问题。

### 1.2.2 Authorization & Authentication
authorization的本质是**主体subject对客体object的操作集合**。人们往往把这种操作集合称为subject对object的rights，或者说subject被grant的privileges。

和identity类似，authorization也涉及到：
1. authorization信息的表示representing，assign和bind到entity（包括subject和object）。assigning authorization的过程形成了a level of trust，即按级别逐级分配trust。
2. 认证authentication过程，即确信authorization是合法有效的。

比如
1. OS system的系统目录访问权限，由OS administrator管理
2. data owner指定数据或文件的read/write/execute，比如data owner指定文件的read-only权限。
3. network的ACL，由network administrator管理
4. version 3 X.509 certificates containing extensions。 这些extension indicates how the certificate should be used。

这些authorization信息被存放在某个地方，比如文件系统的inode，或者存储系统的元数据管理服务，或者ACL（一般与entity存放在一起），或者CA系统中。换句话说，Authenticity of authorization information is provided either by its trusted relationship with identity information (local binding) or because it is carried in cryptographically verifiable certificates.即这些授权信息的真实性，或者由身份信息的信任关系提供（这是本地绑定的情况，比如上面提到的文件系统inode授权信息，就与文件实体本身绑定在一起，或者与entity存放在一起的ACL授权信息），或者这些信息被存放在可验证的证书体系中（这是第三方，比如CA认证的情况）

### 1.2.3 Decision
根据先前认证通过的identity和authorization information，来make an access control decision：prove或者deny。

access decisions通常基于access control policy来实施，而access control policy可大致分为两类：
1. RBAC。Rule-based Access Control，也被称为mandatory policy。checks an entity's authorizations against an established rule set。其基本原理：
   1. 将敏感信息打上数据标签，然后针对这些标签设定各种操作的rights。
   2. 为entity赋予一定的privilege（a special right or advantage that only one person or group has）
   3. 所谓的rule，就是Access is granted to sensitive information if an entity’s privileges are appropriate for the sensitivities of the data. 
2. IBAC。Identity-based Acccess Control，也被称为discretionary policy。即bases decisions on an identity。其基本原理是grants or denies a access based on the presence of an entity on an ACL. If an entity is on the ACL, access to the requested information or resource is permitted; otherwise, access is denied. 

具体实践上，我们经常使用它们的组合策略，被称为**role-based policy**，基于角色的访问策略, can be considered a variant of both IBAC and RBAC. In this case, an identity is assigned to a group that has been granted authorizations. Identities can be members of one or more groups. 

## 1.3 confidentiality
防止敏感信息泄露实际上是access control的subset：providing access control also provides confidentiality, or conversely, that providing confidentiality is a type of access control.

有几种私密性保护策略：
1. Data Protection： 即对data进行加密encryption，防止已经接触到数据的无关人无法获取其中的私密信息。有对称加密和非对称加密两种实现方式
2. Data Separation：即采取某种机制，防止data from flowing into undesired areas，避免无关人接触到数据。
3. Traffic Flow Protection：对传输中的data或地址进行混淆隐藏。比如Data padding，向数据流中添加随机数据从而隐藏数据传输速率，或者使用NAT（Network Address translation）隐藏内部IP地址，或者使用IPSec建立通讯的加密通道。
4. 其他机制，比如spread-spectrum 扩频技术，frequency hopping 跳频技术

## 1.4 Integrity
prevention of unauthorized modification of data (both stored and communicated), detection and  notification of unauthorized modification of data, and recording of all changes to data.

完整性的前提是**origin/source authentication**。integrity protection is of no value unless it is combined with a mechanism that provides authentication of the source. Without source authentication, anyone could tamper with the original data and then just reapply an integrity mechanism.

被保护的数据可分为单数据集a Single Data unit和流式数据集a stream of data units。

对a Single Data unit，一般由originating entity计算check value并随数据发送，常用的算法比如Checksum（CRC），hash值（a message digest）。在中间人攻击情况下，原始信息可以被修改并重新计算check value，因此需要引入认证因子信息：
1. 使用私钥来**签名**Checksum或者digest
2. 使用共享密钥连同message进行hash，比如**HMAC**

对a stream of data units，为了防范reordering, losing, replaying and inserting, or modifying data，some type of ordering information must be provided within the communications protocol（比如sequence numbers and timestamps），常用的办法是encrypting the sequence depends on the encryption of all previous sequences，比如**Cipher block chaining (CBC)**

## 1.5 Availability
所谓可用性availabitity，即为认证并授权的用户提供timely, reliable access to data and information services。先前提到的Access control, integrity, and confidentiality都有助于实现the availability security service.

可用性包括：
1. Protection from Attack，比如**防病毒**
2. Protection from Unauthorized Use，比如**DDOS**，**nondelivery notice floods**
3. 避免服务Failures，比如**high reliability**, **redundancy**(equipment, multiple network routes)

## 1.6 Nonrepudiation
所谓repudiation，是指denial by one of the entities involved in a communication that it participated in that communication. 我们主要利用the authenticating characteristics of digital signatures来实现对entity的身份认证，因此证书的有效性是关键。

X.509 证书规范包含**CRL**（certificate revocation list），另外一种IETF认可的证书验证标准是**OSCP**（the Online Certificate Status Protocol）

# 2 具体应用
## 2.1 HTTP authentication framework & OAuth2.0
### 2.1.1 HTTP authentication framework
HTTP authentication framework, which can be used by a server to challenge a client request, and by a client to provide authentication information.流程如下：

![HTTP-authentication-flow](/assets/2021-06-21-cybersecurity-architecture/HTTP-authentication-flow.png)

当客户端访问站点server时， Server首先响应401发起challenge：provides information on how to authorize with a *WWW-Authenticate* response header containing at least one challenge. 随后客户端按照要求，通过including an *Authorization* request header with the credentials并发送给server，注意这个**credential包括authentication scheme以及相应的authentication information两个部分**。最后server进行authenticating。上图对应Basic scheme。

HTTP认证框架支持多种authentication scheme，比如：
1. Basic：直接将用户名和密码串接后进行base64编码，然后传递给服务端进行验证。
2. Digest：依据一定规则，对用户名或密码，使用SHA-512-256计算摘要。
3. Bearer：bearer tokens to access **OAuth** 2.0-protected resources

其他scheme还有HOBA，Mutual和AWS4-HMAC-SHA256。

就Basic和Digest而言，它们都不够安全，因此SHOULD be over a secure channel like HTTPS。

Bearer是一种特别的scheme，其后的credentials为一个token，具体认证流程对应于The OAuth 2.0 Authorization Framework。简单说，OAuth2.0 是基于HTTP authentication framework之上构建的一个新的协议，以便enables a third-party application to obtain limited access to an HTTP service。

### 2.1.2 OAuth 2.0
OAuth 2.0主要是为解决第三方应用（Third-Party Application）访问资源服务的授权问题而定义，OAuth is an open standard for access delegation（委托访问，第三方应用代表用户访问资源服务）, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords。注意：
1. 用户向第三方应用授权的主要目的，是让第三方应用能代表用户来访问资源服务站点上的服务资源。获取详细的用户信息并不是必须，只有访问服务资源时需要使用用户信息的情况下，第三方应用才会获取资源服务器上的用户信息。
2. 让第三方应用代表用户，并不是用户把自己的私密信息（比如物理token，密码等）直接提供给第三方应用，而是采用token机制，即授权服务器authorization server在获得用户（即资源所有者）批准的情况下，向第三方应用提供access token，然后第三方应用就可以使用这个token访问资源服务器，如同用户自己亲自访问资源服务器一样。

OAuth 2.0 的核心思想是用token代替密码，避免私密信息泄露，控制授权时间，降低安全攻击的可能性。

OAuth2.0有四种授权模式，其中比较常见的为Authorization Code Grant和Implicit Grant两种，其中前者对应The authorization code flow，也被称为the server-side flow，而后者对应the Implicit Flow, aka as the Client-Side Flow。我们先看Server-side flow如下：

![OAuth-authorization-code-flow](/assets/2021-06-21-cybersecurity-architecture/OAuth-authorization-code-flow.png)

换个视角如下：

![OAuth-authorization-code-flow1](/assets/2021-06-21-cybersecurity-architecture/OAuth-authorization-code-flow1.png)

再来看Client-side flow：

![OAuth-implicit-flow](/assets/2021-06-21-cybersecurity-architecture/OAuth-implicit-flow.png)

换个视角如下（此图去掉了从Web-Hosted client resource服务器上获取脚本的D、E、F三步，因为第三方应用一般会直接将这个脚本内置在代码中，当授权服务器通过fragment返回access token后，第三方应用会直接剥离并获取到这个token）：

![OAuth-implicit-flow1](/assets/2021-06-21-cybersecurity-architecture/OAuth-implicit-flow1.png)

那么server-side和Client-Side是什么意思呢？首先要将它们与OAuth2.0中的client区别开，OAuth2.0中的Client是指第三方应用，而Server-side和Client-Side都是指这个第三方应用：对于一个web application来说，the code resides on both sides, client-side code除了负责用户交互，也负责一些业务逻辑处理，server-side code负责数据持久化，以及一些业务逻辑处理。在过去，绝大部分业务逻辑都在服务端实现，而随着前端技术的飞速发展，前端也即客户端的库越来越丰富，客户端实现的业务逻辑占整个系统的比重也越来越高。

OAuth2.0中区分client-side web application和server-side web application的关键，或者说在OAuth2.0中区分server-side（authorization code grant type）和client-side（implicit grant type）认证流程的关键，在于获取token执行特权指令的业务逻辑代码是在client端还是在server端实现：如果执行特权指令的代码在client-side端，那么获取token的代码逻辑就运行在用户本地的user-agent中；如果执行特权指令的代码在Server-side端，就意味着获取token的代码逻辑运行在第三方应用的server远端。

基于安全性的考虑，对于server-side端实现，token不会在用户本地的user-agent落地，安全性由授权服务器对第三方应用的注册、认证和其他线下管理来保证；对于client-side端实现，token只能在本地落地并由第三方应用部署在本地user-agent中的代码来使用。

## 2.2 Session-id vs JWT
Session-id和JWT都用来识别用户，并为无状态的HTTP提供状态通讯能力。

Session-id是cookie的演绎发展：将原来保存在客户端cookie中的用户相关信息迁移到服务端内存中维护，并将其映射为一个唯一的session-id，掌握该session-id的客户端即被认证为合法用户。在这种模式下，the Server does all the heavy lifting server-side. Broadly speaking a client authenticates with its credentials and receives a session_id (which can be stored in a cookie) and attaches this to every subsequent outgoing request. So **this could be considered a "token" as it is the equivalent of a set of credentials**. There is however **nothing fancy about this session_id string. It is just an identifier and the server does everything else**. It is stateful. It associates the identifier with a user account (e.g. in memory or in a database). It can restrict or limit this session to certain operations or a certain time period and can invalidate it if there are security concerns. More importantly it can do and change all of this on the fly. Furthermore it can log the user's every move on the website(s). Possible disadvantages are bad scale-ability (especially over more than one server farm) and extensive memory usage. 注意，**Session_id只是一个普通的唯一性标记，其业务含义完全由服务端来解释，因此服务端必须负责维护session_id所对应的所有用户信息和状态信息**：这一方面导致站点服务需要消耗大量的内存来保存session信息，另一方面也限制了站点服务集群的扩展性和容错能力，因为集群中每个服务器上的session信息，在其他服务器的内存中并不存在。当这台服务器因异常宕机时，连接到该服务上的所有session将失效（一种可能的解决方案是使用分布式缓存来集中存放所有session）。

JWT（JSON Web Token）是一种token。实际上，token大致分为两类：一类是access token（比如OAuth2.0中的token），被用来进行单纯的访问控制；另外一类是session-token，含有session和状态信息，由于session和状态信息也包含用户信息，因此session-token也能实施认证和访问控制。JWT是一种session-token。在session-token模式下，**no session is persisted server-side (stateless)**. The initial steps are the same. Credentials are exchanged against a token which is then attached to every subsequent request (**it can also be stored in a cookie，也可以由客户端应用自己管理，比如放在请求头中发送给服务端**). However for the purpose of decreasing memory usage, easy scale-ability and total flexibility (tokens can be exchanged with another client) **a string with all the necessary information is issued (the token)** which is checked after each request made by the client to the server. 

JWT由head，payload，signature/Encryption三部分构成，其中head指明token的类型和后面第三部分的签名或加密散列算法，payload是存放session信息的地方，在实际使用时默认采用base64编码，所以前两部分在token中相当于是明文，最后一部分是验证签名。

![session-token-JWT](/assets/2021-06-21-cybersecurity-architecture/session-token-JWT.png)

对于签证签名部分，主要有三种构造机制：
1. **hash机制**：Using a hash mechanism e.g. HMAC-SHA1
```
token = user_id|expiry_date|HMAC(user_id|expiry_date, k)
where user_id and expiry_date are sent in plaintext with the resulting hash attached (k is only know to the server).
```
2. **对称加密机制**：Encrypting the token symmetrically e.g. with AES
```
token = AES(user_id|expiry_date, x)
where x represents the en-/decryption key.
```

3. **非对称加密机制**：Encrypting it asymmetrically e.g. with RSA
```
token = RSA(user_id|expiry_date, private key)
```
## 2.3 可扩展的OTP
如1.2.1.2节所述，单纯OTP机制下，物理OTP token是站点特定的，这就导致其scalability受到限制：我们需要为不同的站点准备不同的token。

我们可以结合PKI的公私钥机制来实现可扩展的OTP系统。具体流程如下：
1. 支持OTP的服务器，生成OTP（one-time password），然后使用用户的公钥加密得到$E_{OTP}$
2. 用户使用私钥解密$E_{OTP}$得到OTP，然后将OTP发送给服务器
3. 服务器验证收到的OTP是否为先前生成的OTP

这里有几点需要注意：
1. 用户需要提前将其公钥**注册**到支持OTP的服务站点，以便服务站点能根据用户名找到对应的用户公钥。
2. 使用一个用户私钥来访问多个服务站点。
3. 客户端根本无需支持OTP机制，比如计算OTP。实际上，客户端只需要有一个存储私钥的物理token。
4. 非标准协议，需要服务站点的支持，且无法抵御钓鱼攻击（因为登录过程中，客户端并没有对站点和相关域名的验证流程）。可参照下面FIDO2。

## 2.4 FIDO2
FIDO2利用了PKI非对称的公私钥加解密机制，加上对物理token的保护性访问（PIN码，或者指纹），FIDO2是一个2FA协议。

FIDO2(Fast IDentity Online 2)，是一个**phishing proof**, **passwordless** authentication protocol。

FIDO2的数据流如下：

![FIDO2-flow](/assets/2021-06-21-cybersecurity-architecture/FIDO2-flow.png)

它包括三个通讯的主体和两个通讯协议：
1. 三个通讯主体为Client(通常是Browser)，Relying Party（即提供服务和实施authencation的Server端），以及Authenticator（指内部或外部提供签名的物理token设备）
2. 两个通讯协议，一是client与Authenticator之间通过CTAP协议和相关API接口进行通讯，实施签名；二是client与Relying Party之间通过WebAuthn协议进行通讯，实施注册和authentication。

支持FIDO2的服务端站点要求用户先注册，注册流程如下：

![WebAuthn-registering-flow](/assets/2021-06-21-cybersecurity-architecture/WebAuthn-registering-flow.png)

注意:

1. 上面的device（即authenticator）生成的key-pair是针对当前注册站点（即relying server）的域名（比如example.com）, 这在后续authenticating时是一个非常重要的要素。这里我们只要知道，**authenticator device中存储了很多个私钥，每个私钥针对不同的特定站点**。
2. authenticator device在生成public-private key pair并将the public key返回给browser client，browser再将这个public key发送给relying server。实际上，为了方便进行源端身份认证，authenticator device返回给browser client的信息，是使用authenticator device的初始私钥（我们称其为attestation private key）加密为该特定站点生成的密钥对中的公钥（我们称其为assertion public key），串接这个初始私钥对应的公钥（我们称其为attestation public key）。browser client将其发送给Server，而Server首先会验证attestation public key的合法性，如果合法则完成注册。
> authenticator devcie的初始私钥以及公钥的验证数据体系是如何建立的呢？FIDO2的规范没有说明，但FIDO alliance提供了一个Metadata Service，称为MDS，任何FIDO authenticator vendors can publish metadata statements，然后任何一个遵循FIDO规范的站点服务器就可以下载这个metadata信息。也就是说，The FIDO Alliance Metadata Service (MDS)提供了a centralized and trusted source of information about FIDO authenticators. Vendors are frequently releasing new authenticators or updating existing ones. In addition, vulnerabilities may be discovered in existing authenticators, requiring that their use be limited or phased out. 而各个使用FIDO MDS服务的组织应该keep its metadata database up-to-date to ensure it has the latest information about new authenticators, including their certification status, and protect itself against vulnerabilities in trusted authenticators. 简言之，FIDO alliance充当了一个中立的CA。每个authenticator device在制造时，即内置了自己的私钥，而公钥则被制造商录入到了这个中立的CA中。这就是上面FIDO2的数据流图中，Relying Server和FIDO Server之间的数据访问链路：Felying Server会从FIDO alliance的Metadata Servcie获取autenticator的公钥信息，以验证用户注册使用的authenticator的身份。

此后，用户登录一个已注册站点的流程如下：

![WebAuthn-authenticating-flow](/assets/2021-06-21-cybersecurity-architecture/WebAuthn-authenticating-flow.png)

注意：
1. browser client收到后，会基于CTAP协议访问authenticator device，并提供当前站点域名参数。authenticator device根据站点域名参数查找到对应key pair，并然后用私钥对challenge签名，再随同公钥一并发给browser client，browser client将其作为resposne返回给relying server。relying server首先会检查公钥是否已注册，然后用这个公钥解密签名串，如果能得到原始的challenge，说明用户掌握有私钥，从而识别验证了这个用户的身份。
2. 当服务器发送challenge时，browser client会**将服务站点的域名随同challenge一并传递给authenticator device**中，如果当前服务域名是一个非法的钓鱼站点，则authenticator就不会返回正确的信息（因为只有正确注册过的合法站点域名才有对应的私钥信息）。这是FIDO2保证**phising proof**的关键所在。
3. 在用户登录的整个过程中，只有在browser client使用CTAP协议访问authenticator device时，需要用户介入授权访问（比如输入device的PIN码，或者使用指纹认证授权），其他流程全部自动，避免了用户输入站点的用户名和密码，因此FIDO2是**passwordless**的，换句话说，用户至多只需要记住authenticator devcie的PIN码（如果是指纹认证的话则连PIN码也无需记忆），就能利用其轻松登录各个不同的服务站点。


参考
- [1] National Security Agency. Information assurance technical framework, release 3.1. Sep, 2002
- [2] Dawid Czagan. [One-time passwords with token. Aug, 2013](https://resources.infosecinstitute.com/topic/one-time-passwords-with-token/)
- [3] [FIDO2: WebAuthn & CTAP](https://fidoalliance.org/fido2/)
- [4] Kevin Chandra. [What is WebAuthn](https://medium.com/cotterapp/what-is-webauthn-logging-in-with-face-id-and-touch-id-on-the-web-d71e8d53933b)
- [5] Hoax. [Session Authentication vs Token Authentication](https://security.stackexchange.com/questions/81756/session-authentication-vs-token-authentication)
- [6] 周志明. [凤凰架构](https://icyfenix.cn/)
