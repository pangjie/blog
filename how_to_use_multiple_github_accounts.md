# 如何同时使用多个Github账号

简单的说，在一台mac上同时使用多个Github账号，就是通过创建多个密钥，配置```ssh```，使得在本地可以```push```代码到不同的Github账户。下文将按照这一步骤进行说明：

[1. 创建RSA密钥](#1-创建rsa密钥)

[2. 配置```ssh```和```git```](#2-配置ssh和git)

[3. 使用示例](#3-使用示例)

[4. 坑点重申](#4-坑点重申)

[5. 参考资料](#5-参考资料)


## 1. 创建RSA密钥

同时使用多个Github账号首先需要分别为其创建RSA密钥，并将密钥中的公钥添加到各自账号的 SSH Keys 中。创建RSA密钥的过程可以查询[官方手册](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。例如为账号```first_account@email.com```创建一个符合Github要求的RSA密钥：

```bash
ssh-keygen -t rsa -b 4096 -C "first_account@email.com" 
    # 为Github账号first_account@email.com创建密钥。
```

创建过程中请注意，为了区分密钥与账号的对应关系，不要使用默认目录及名称，请**手动指定密钥的名称**：

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/id_rsa_1 
    # 默认目录及名称为 ~/.ssh/id_rsa，可以自定义目录及名称，但不能与其他密钥重名。
    # 这里使用 ~/.ssh/id_rsa_1，以便区分不同账号对应的密钥。
Enter passphrase (empty for no passphrase): 
    # 询问是否要为该密钥设置密码，如果不需要密码，回车即可跳过。
Enter same passphrase again: 
    # 重复密码，如果不需要密码，回车即可跳过。
```
创建完成后，生成的密钥分为一个私钥```id_rsa_1```和一个公钥```id_rsa_1.pub```。再次使用```ssh-keygen```为另一个Github账号```second_account@email.com```创建密钥：

```bash
ssh-keygen -t rsa -b 4096 -C "second_account@email.com" 
    # 为Github账号first_account@email.com创建密钥。

Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/id_rsa_2 
    # 默认目录及名称为 ~/.ssh/id_rsa，可以自定义目录及名称，但不能与其他密钥重名。
    # 这里使用 ~/.ssh/id_rsa_2，以便区分不同账号对应的密钥
Enter passphrase (empty for no passphrase): 
    # 询问是否要为该密钥设置密码，如果不需要密码，回车即可跳过。
Enter same passphrase again: 
    # 重复密码，如果不需要密码，回车即可跳过。
```

此时在默认目录```~/.ssh/```下，至少有两对密钥，分别对应```first_account@email.com```和```second_account@email.com```。请在对应账号的 Github Settings 里新增 SSH Key，并复制粘贴对应账号的公钥：```id_rsa_1.pub```和```id_rsa_2.pub```。[相关步骤可以查询官方手册](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account)。


## 2. 配置```ssh```和```git```

完成了上面的步骤后，需要配置```~/.ssh/config```，以便通过映射关系来指引```git```找到正确的Github账号。

```bash
touch ~/.ssh/config # 如果还没有config文档，则需要创建一个新的
```

添加或修改如下内容到 ```~/.ssh/config```

```bash
# github.com: first_account@email.com
Host git1 # 可以自定义，此名称用来在git参数中与其他账户做区分，因而不能与其他账户相同。
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_1 # 该账户对应的私钥

# github.com: second_account@email.com
Host git2 # 可以自定义，此名称用来在git参数中与其他账户做区分，因而不能与其他账户相同。
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_2 # 该账户对应的私钥
```

配置```ssh```后，也需要对```git```进行简单的配置。使用多个Github账号时，不宜使用```--global```对```user.email```和```user.name```进行全局配置。请在各自的```repo```目录下使用```--local```指定参数，例如：

```bash
git config –global –unset user.name # 取消user.name全局配置
git config –global –unset user.email # 取消user.email全局配置

cd ~/project_1
git config --local user.email 'first_account@email.com'
git config --local user.name 'john'

cd ~/project_2
git config --local user.email 'second_account@email.com'
git config --local user.name 'tom'
```

## 3. 使用示例

下文的示例使用```git```命令```push```代码到不同的Github账号：

```bash
cd ~/project_1
git push git1:first_account_username/project_1 master

cd ~/project_2
git push git2:second_account_username/project_2 master
```

以此类推，其他的```git```命令，如```remote```同样可以使用相应参数来区分Github账号。

## 4. 坑点重申

* 创建新密钥前，请查看```~/.ssh/```下是否有其他密钥，以免因误操作导致重名密钥被覆盖。

## 5. 参考资料

* [GithubHelp](https://help.github.com)
* [一台电脑绑定两个github帐号教程](https://www.jianshu.com/p/3fc93c16ad2d)
* [Multiple github accounts on the same computer?](https://stackoverflow.com/questions/3860112/multiple-github-accounts-on-the-same-computer?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)