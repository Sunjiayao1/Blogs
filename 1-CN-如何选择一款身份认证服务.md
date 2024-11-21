# 如何选择一款身份认证服务？

OAuth
2.0是允许用户授权客户端代表其访问受保护资源的协议，例如在使用某些微信小程序时会弹出对话框寻求你的授权，基于OAuth2.0协议授权后该小程序就能访问你允许其访问的资源，比如头像、用户名等。有很多公司基于此构建了身份认证管理系统让使用者能够快速构建认证服务，常见的有Amazon
Cognito, Auth0, Firebase Authentication, Ping Identity，Azure Active Directory等。

我曾经在项目中同时接触到了两个市场份额占比大的基于云的身份管理服务，Auth0和Cognito，两者都能提供快速搭建身份认证和授权的服务，包括创建客户端，维护用户池，提供多种登录和多重身份认证方式等。

但它们的区别在哪里，如果想要构建一个认证服务，应该如何选择供应商呢？本文将从用户使用的角度以及开发者的角度进行对比。

## 用户使用角度

### 自定义界面

当用户使用一个认证系统时，界面是否简洁容易操作，提供的登录方式是否符合需求是用户首先注意到的内容。并且客户端往往要求自行定制界面，例如添加公司logo，定义界面颜色样式等。在这方面，Cognito和Auth0都提供了基于不同客户端定制UI的功能。

[Cognito Hosted UI](./pictures/1-01.png)

Cognito的UI定制功能自定义范围有限，只能替换颜色，logo样式，按钮和输入框样式。

[Auth0 Hosted UI](./pictures/1-02.png)

Auth0的可定制范围广，在universal
login功能中可以定制登录、注册、忘记密码、MFA界面的颜色、字体、logo、排列方式，并且提供了liquid模版，可以根据客户端加载不同的界面样式。在此基础上如果有其他定制化的需求，Auth0还基于lock和auth0.js
SDK提供了三种模版：Lock, Lock(passwordless)和Custom Login Form方便进行自定义。

### 登录方式

在提供的登录方式方面，Cognito提供的有用户名和密码登录，社交账号登录以及企业登录，涵盖Google，Apple，Facebook，Amazon，SAML和OIDC等。

[Cognito支持的外部登录种类](./pictures/1-03.png)

在此基础上，Auth0提供了52种社交账号登录方式，在企业登录方面还提供了Google workspace, Okta，Azure, Ping, ADFS,
LDAP等方式，只需提供client
ID和secret就能完成配置，因此在利用社交账号登录和企业登录方面Auth0配置更方便，提供的选择更多。此外，Auth0还提供了passwordless登录和biometrics登录，这些无密码身份认证方式提升了用户账户的安全性，也为用户省去了管理密码的麻烦。

[Auth0支持的企业登录种类](./pictures/1-04.png)

[Auth0支持的社交账号登录种类](./pictures/1-05.png)

### 用户安全保障

除此之外，用户还会关心自己的账户是否安全。Auth0和Cognito都提供了多种方式保障用户安全，Auth0提供了Bot Detection，IP
throttling, Brute-force Protection, Breached Password Detection和多重身份验证（MFA），Cognito security提供了MFA和IP限制两种方式。

MFA是除了登录认证之外的另一种验证身份并保障账户安全的措施，在涉及到身份验证，特别是在进行支付，添加新设备，或者长时间不登录账户等情况下验证用户的合法性更是十分重要。

Cognito MFA可以提供了SMS与one-time-password（OTP）两种方式，可以选择通过短信或是邮件验证。Auth0 MFA提供了FIDO security keys,
FIDO device biometrics, OTP, auth0 guardian, 短信，邮件，recovery code，DUO security等方式。相比之下，Auth0提供了更多高安全性和高可用性的验证方式。

[Auth0支持的MFA种类](./pictures/1-06.png)

### 用户偏好

最后，在用户偏好方面，Auth0提供了更方便的属性定义方式。Auth0用户属性包括标准属性，user_metadata和app_metadata。用户可以在user_metadata内自定义各种属性而不需要更改userpool属性，例如基于metadata可以完成例如显示偏好颜色，添加用户昵称等各种自定义属性。在app_metadata中可以添加对系统产生影响的属性，例如，如果想对特定用户开启MFA，就可以在这些用户的app_metadata中添加MFA属性，从而达到个体化定制的效果。

Cognito的用户属性包括标准属性和自定义属性，如果某一用户需要添加某种新的自定义属性，则需要创建新的userpool并在属性中添加对应字段。并且由于cognito
userpool不支持数据导出，因此只能通过trigger将已经存在于旧userpool的用户在登录时导入到新的之中，为用户管理增添了麻烦。

## 开发者角度

从服务开发和维护者的角度，可以从以下几个方面来比较两个身份认证管理系统。

### 用户管理

在用户管理方面，用户在授权应用程序访问信息的过程中，可能会用不同的登录方式进行登录，例如使用同一个gmail邮箱进行google登录，并用该邮箱进行用户名密码登录，针对这种情况应该如何管理呢？

Auth0中，不同来源的用户通过connection区分，例如通过social登录，用户名密码登录和passwordless登录会被划分为三种connection。不同connection可以理解为不同数据库，同一个邮箱因注册方式不同进入不同数据库中，这些账户可以进行关联。

在用户登录时，如果该邮箱在其他connection中存在另外的用户，就会询问用户是否进行关联，用户在成功登录另外账号后，两个账号就能关联起来，系统将视两个账号为同一用户，这样能方便用户的使用和管理。Auth0提供了Auth0
Account Link官方插件完成账户关联功能。同样，在Cognito中通过外部登录的用户会被划分到不同的group中，但没有提供用户关联功能，需要使用者自己实现。

### 流程自定义

在实现登录或注册等流程中可能会遇到添加自定义功能的情况，比如针对部分用户开启MFA验证，或者给用户添加权限等，此时就会需要在特定阶段执行自己的逻辑。在Cognito中这种功能称为Trigger。Trigger的本质是lambda函数，可以通过event在身份认证的不同节点触发自定义的lambda函数达到自定义用户登录，注册，认证等流程的效果。因为是lambda函数，因此可以使用所有lambda支持的语言编写。

| cognito                        | auth0                  |
|--------------------------------|------------------------|
| Pre sign-up                    | Login                  |
| Pre authentication             | Machine to Machine     |
| Post authentication            | Pre User Registration  |
| Post confirmation              | Post User Registration |
| Custom message                 | Post Change Password   |
| Define Auth Challenge          | Send Phone Message     |
| Create Auth Challenge          |                        |
| Verify Auth Challenge Response |                        |
| User Migration                 |                        |
| Pre Token Generation           |                        |

Auth0添加自定义逻辑的方式有三种，Rules, Hooks和Actions，三者的实现方式都基于node.js，因此只能用js编写。Rules和Hooks是初始版本, 现在更推荐使用Actions。它可以在用户登录，注册，修改密码等阶段应用，并且还可以与外部库集成提升可扩展性。在编写逻辑时Auth0提供了模版方便自定义。另外它支持可视化的对不同逻辑的执行节点和顺序进行调整。此外，Actions的每一次部署都会创建一个新的version, 如果新的version出现问题，可以迅速切换回旧version。

[Auth0 Actions可视化编辑界面](./pictures/1-07.png)

### 系统监测

在系统维护中，监控对于保证一个系统可靠性和可用性以及监测系统性能方面有十分重要的作用。因为Cognito是AWS提供的服务，因此可以方便的和其他AWS服务连接，比如CloudTrail, CloudWatch等。Auth0监控方法包括logs和streams。Logs中记录了tenant中所有行为的日志，包括对client的操作和用户进行的操作等，可以通过filter筛选某一类型的logs。Streams可以将所有日志导出并在其他系统里进行进一步操作，例如可以与Amazon EventBridge，Datadog, Splunk，Sumo Logic等各种事件分析服务连接。另外，如果想查询某一用户的操作记录，Auth0中可以在用户的history中直接查询。

### 数据迁移

另外，随着不同供应商提供的产品发生变化，开发者可能会在不同产品间切换来满足需求。这就要求身份认证管理系统能够很好的支持客户端信息迁移，数据导入导出等。Auth0支持Client Id和Client Secret导入，这样即便是更换了平台，也能保证客户端不受影响正常使用。

用户数据迁移方面，Cognito不支持用户数据导出但Auth0支持，并且可以对密码进行加密。在数据导入中，Cognito和Auth0都提供了外部数据库导入的功能，Bulk migration指一次性将用户数据全部导入到新服务的数据库中，这可能会导致用户需重新登录。Lazy migration/ Automatic migration指的是用户在登录时将数据迁移到新系统中。两者都可以通过bulk migration或lazy migration的方式导入用户数据。

Auth0提供了插件帮助使用者进行数据导入导出，进行在导入数据时Auth0会对密码进行加密。Auth0还支持session migration, 因此用户在切换identity provider时不会感受到账号被登出。此外，Auth0还提供了外部数据库连接功能，即不使用Auth0提供的userpool，而是使用自己维护的数据库作为userpool。这为数据导入导出和切换认证服务供应商带来了很大的便捷。

[Auth0 lazy migration流程图](./pictures/1-08.png)

### SDK支持

最后，在开发过程中，Cognito提供了Amplify SDK，它提供了两种方式构建应用，一种是使用authentication UI components, 针对React, Vue, Angular和React Native都预置了UI组件。另一种是使用amplify的Auth进行sign up/sign in等身份验证流程。

Auth0同样提供了多种SDK，包括针对SPA、Web应用、API、移动端、Management API的ADK。并且针对不同语言和框架都有支持，例如Web应用的SDK包括Express, Java, Next.js, PHP, Python, Ruby等不同版本。此外Auth0有良好的社区支持，它提供了Auth0 Community供开发者交流，也有support team进行问题解答。

## 小结

综上，在用户体验方面，Auth0拥有高度定制化的UI并提供了丰富的认证方式，各种无密码登录提高了用户体验和账号安全性。

在开发者使用过程中，Auth0和Cognito都能完成基本功能的快速配置，但Auth0提供的功能和模版更多，在实现自定义逻辑方面更加快捷，同时在marketplace里提供了大量插件方便使用。而相比之下在Cognito中必须要自己实现，增加了开发时间。

此外，Auth0拥更为清晰的文档，支持团队反馈更加及时，新功能从提出到上线周期也更短。因此，如果仅需要基本的认证，Auth0和Cognito都能满足需求，Cognito因较便宜的价格更胜一筹。但如果需要较好的用户体验或需要实现丰富的功能，则Auth0是更好的选择。
