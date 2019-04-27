<!-- TOC -->

- [AWS IAM 实践小结](#aws-iam-实践小结)
  - [最佳实践原则](#最佳实践原则)
    - [不要使用根账户的秘钥](#不要使用根账户的秘钥)
    - [创建 IAM 用户](#创建-iam-用户)
    - [权限分配](#权限分配)
    - [为特权用户启用 MFA](#为特权用户启用-mfa)
  - [举例实践](#举例实践)
    - [创建 Admins 组和 admin 用户](#创建-admins-组和-admin-用户)
    - [创建 Dvelopers 组 和 dennis 用户](#创建-dvelopers-组-和-dennis-用户)
    - [为 Admin 用户启用控制台 MFA](#为-admin-用户启用控制台-mfa)
    - [对 AWS CLI 启用 MFA 认证](#对-aws-cli-启用-mfa-认证)
    - [配置受 MFA 保护的 API 访问](#配置受-mfa-保护的-api-访问)
  - [参考](#参考)

<!-- /TOC -->

<a id="markdown-aws-iam-实践小结" name="aws-iam-实践小结"></a>

## AWS IAM 实践小结

[AWS Identity and Access Management(IAM)](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/introduction.html) 是一种 Web 服务，可以用来安全地控制对 AWS 资源的访问，以及控制对用户进行身份验证（登录）和授权（具有权限）来使用资源。  
首次注册 AWS 账户（account）时，这个账户为`根用户`，具有所有 AWS 服务和资源的所有权限。可以用注册的根用户凭证(`邮箱`和`密码`)来登录。建议不要使用该账户来进行日常任务，应该为每个使用者创建 IAM 用户，并根据不同需求分配不同粒度的权限。

<a id="markdown-最佳实践原则" name="最佳实践原则"></a>

### 最佳实践原则

关于 IAM 的使用，建议阅读官网的 [IAM 最佳实践](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/IAMBestPracticesAndUseCases.html)。

<a id="markdown-不要使用根账户的秘钥" name="不要使用根账户的秘钥"></a>

#### 不要使用根账户的秘钥

在 AWS 管理控制台的【我的安全凭证】页面，可以看到自己的秘钥。但是根用户，不要创建这个秘钥。如果创建，请删除。

<a id="markdown-创建-iam-用户" name="创建-iam-用户"></a>

#### 创建 IAM 用户

可以为账户里的每一个人创建 IAM `用户(user)`，也可以创建`用户组(group)` 来管理一组相同职能的用户，并为组分配不同的权限。根用户登录之后，先创建一个【管理员】组，分配 `AdministratorAccess 托管策略`。将管理员的用户分配到这个组里。其它的用户和组，全由管理员来管理。  
创建用户和组的方法，参考这篇[【创建您的第一个 IAM 管理员用户和组】](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/getting-started_create-admin-group.html)。

> 只有少数情况下，需要用到根账户。[见这里](https://docs.aws.amazon.com/zh_cn/general/latest/gr/aws_tasks-that-require-root.html)。

<a id="markdown-权限分配" name="权限分配"></a>

#### 权限分配

权限分配遵循【最小权限原则】。需要执行相应的任务，才由管理员分配对应的权限。  
AWS 提供一些`托管策略(policy)`，可以在创建用户时直接选择。用户可以创建一些自定义的托管策略。

<a id="markdown-为特权用户启用-mfa" name="为特权用户启用-mfa"></a>

#### 为特权用户启用 MFA

为增强安全性，应为持有特权的 IAM 用户 (获准访问敏感资源或 API 操作的用户) 启用[多重验证 (Multi-Factor Authentication, MFA)](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa.html)。启用 MFA 后, 用户在使用普通凭证登录之后，需要通过 MFA 设备提供的验证码做额外验证。  
MFA 支持多种不同形式的[验证方法](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa.html)。接下来我们主要介绍 `虚拟 MFA 设备`。虚拟 MFA 设备，是手机或其他移动设备上运行的一个应用程序。AWS 需要可产生六位数 OTP 的虚拟 MFA 应用程序。一些支持的应用程序列表，[见这里](https://aws.amazon.com/cn/iam/details/mfa/)。我们选择 [Authy](https://play.google.com/store/apps/details?id=com.authy.authy&hl=en)，大家可以自行选择相应的版本下载。具体的启用 MFA 的步骤请[参考官网](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html)，或者按照下面实践中的方法。

<a id="markdown-举例实践" name="举例实践"></a>

### 举例实践

目标：创建一个 `Admins` 组和一个 `Developers`组。创建用户 `admin`, 分配到 Admins 组。用户 `dennis` 分配到 Developers 组。

<a id="markdown-创建-admins-组和-admin-用户" name="创建-admins-组和-admin-用户"></a>

#### 创建 Admins 组和 admin 用户

下面使用控制台创建 Admins 组和 admin 用户。

- 使用根用户信息登录 [IAM 控制台](https://console.aws.amazon.com/iam/)。
- 选择 `Users (用户)`，然后选择`Add user (添加用户)`, 输入用户信息。
- 选中 `AWS 管理控制台 access (AWS 管理控制台访问)` 旁边的复选框，选择 `Custom password (自定义密码)`，然后在文本框中键入新密码。
- 选择 `Next: Permissions (下一步: 权限)`。
- 在`设置权限`页面上，选择`将用户添加到组`。选择 `Create group`。
- 对于 `Group name (组名称)`，键入 `Admins`。
- 选择 `Policy Type (策略类型)`，然后选择 `Job function (作业功能)` 以筛选表内容。
- 在`策略列表`中，选中 `AdministratorAccess` 的复选框。然后选择 `Create group`。
- 选择 `Next: Tagging (下一步: 标记)`,可选操作。
- 选择 `Next: Review` 以查看创建用户的信息。选择 `Create user`完成创建。

完成创建之后，得到 IAM 的登录地址。

```
https://AWS-account-ID or alias.signin.aws.amazon.com/console
```

可以用创建的用户名 `admin` 和相应的密码登录。在控制台的【我的安全凭证】，可以看到用户信息。

- 用户名
- 用户 ARN
- AWS 账户 ID

在【访问秘钥】中，点击生成秘钥。生成页只会显示一次，可以下载 CSV 保存，或者自行记录。

<a id="markdown-创建-dvelopers-组-和-dennis-用户" name="创建-dvelopers-组-和-dennis-用户"></a>

#### 创建 Dvelopers 组 和 dennis 用户

上面使用控制台创建了 admin 用户。现在介绍一下，使用 AWS CLI 来创建用户和组。

- 创建 Developers 组

```
$ aws iam create-group --group-name Developers
```

- 列出所有的组，确保创建成功

```
$ aws iam list-groups
```

- 将策略 `SystemAdministrator` 附加到 Developers 组

```
$aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/job-function/SystemAdministrator
```

- 查看组的策略

```
$ aws iam list-attached-group-policies --group-name Developers
```

- 创建用户 dennis

```
$ aws iam create-user --user-name dennis
```

- 创建用户的访问类型  
  控制台访问，使用 `aws iam create-login-profile`。这里我们使用`编程访问`。

```
$ aws iam create-access-key --user-name dennis
```

- 添加用户到组

```
$ aws iam add-user-to-group --user-name dennis --group-name Developers
```

至此，创建用户完成，可以通过 `aws configure` 在本地添加 profile，使用 AWS CLI 来进行操作服务。

<a id="markdown-为-admin-用户启用控制台-mfa" name="为-admin-用户启用控制台-mfa"></a>

#### 为 Admin 用户启用控制台 MFA

对具有特殊权限的 Admin 用户增加 MFA 验证。

- 登录 AWS 管理控制台, https://console.aws.amazon.com/iam。
- 点击右上角的【Security Credential】选项卡。
- 点击【分配 MFA 设备】，选择【虚拟 MFA 设备】。
  此时会显示 MFA 设备配置的二维码。
- 手机打开下载的 Authy 应用程序，扫描二维码。
- 然后将手机上的 MFA 码，输入到 【MFA code】。需要输入两个连续的码。
- 点击【分配 MFA】完成配置。

分配完之后，会得到一个 MFA 设配的 ARN。

```
arn:aws:iam::{your account id}:mfa/{your user name}
```

这样每次登陆控制台，都需要使用额外的 MFA 码做二次校验。

<a id="markdown-对-aws-cli-启用-mfa-认证" name="对-aws-cli-启用-mfa-认证"></a>

#### 对 AWS CLI 启用 MFA 认证

但是到这一步，只是在控制台登陆需要 MFA 验证。通过 AWS CLI 还是可以访问 AWS 资源，所以我们需要对 AWS CLI 做 MFA 验证，可以[参考这里](https://aws.amazon.com/cn/premiumsupport/knowledge-center/authenticate-mfa-cli/)。
上面我们已经给 admin 用户分配了一个 `AdministratorAccess` 策略。我们可以查看这个策略如下，即有所有服务的所有权限。

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

要启用对 AWS CLI 的访问，需要创建包含 `Condition` 元素的策略，如下：

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}

```

可以将上面的内容创建一条新的策略，如 `AMDIN_WITH_MFA` 策略，然后附加给用户 admin。此时，通过 AWS CLI 访问服务，会收到一个 `AccessDenied` 的错误，
完成了对 AWS CLI 的验证。

<a id="markdown-配置受-mfa-保护的-api-访问" name="配置受-mfa-保护的-api-访问"></a>

#### 配置受 MFA 保护的 API 访问

用 AWS CLI 来访问带 MFA 的资源，就需要通过 `AWS STS` 来获取一个临时凭证，可以通过 [GetSessionToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetSessionToken.html) 和 [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) 两种 API 操作。具体查看官网[这篇文章](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html)。
我们使用【通过命名配置文件使用临时凭证】方法来设置。这里可以使用一个工具 [aws-mfa](https://github.com/broamski/aws-mfa) 来方便管理。  
安装 aws-mfa:

```
$ brew tap chef/chefops-tools
$ brew install aws-mfa
```

编辑用户主目录中的 `.aws 文件夹`下的 `credentials` 文件。使用 aws-mfa 生成 profile, 一般需要先创建一个 `<profile name>--longterm`的 profile。例如创建一个 `admin`的 profile, 需要在 `credentials` 文件中，加入名为 `admin-long-term`的 profile。

```
[admin-long-term]
aws_access_key_id = ***
aws_secret_access_key = ***
aws_mfa_device = arn:aws:iam::<your account id>:mfa/youraws.username
```

使用 aws-mfa 生成 profile， 中间需要输入一次 `MFA Code`。

```
$ aws-mfa --profile admin
```

我们还可以使用上面的指令来刷新 token。 也可以通过可选项 设置不同的参数。参看[这里](https://github.com/broamski/aws-mfa#usage)。

至此，我们就可以通过 AWS CLI 访问带 MFA 的服务了。如查看 S3 的 Bucket。

```
$ aws s3 ls --profile admin
```

<a id="markdown-参考" name="参考"></a>

### 参考

[1][配置受 mfa 保护的 api 访问](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/id_credentials_mfa_configure-api-require.html?utm_source=drip&utm_medium=email&utm_campaign={{%20email.subject%20|%20url_encode%20}}&utm_content=ami-mfa)  
[2][using mfa with the aws api](https://mharrison.org/post/aws_mfa/)  
[3][iam 最佳实践](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/best-practices.html)
