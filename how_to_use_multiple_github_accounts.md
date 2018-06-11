# 如何同时使用多个Github账号

简单的说，在一台mac上同时使用多个Github账号，就是通过创建多对密钥，配置```ssh```，使得在本地可以```push```代码到不同的Github账户。


## 1. 创建 RSA 密钥

同时使用多个Github账号首先需要分别为其创建RSA密钥，并添加到各自账号的 SSH Keys 中。创建RSA密钥的过程可以查询[官方手册](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)。例如为账号```first_account@email.com```创建一个符合github要求的RSA密钥：

```bash
ssh-keygen -t rsa -b 4096 -C "first_account@email.com" 
    # 为Github账号first_account@email.com创建密钥。
```

创建过程：

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/id_rsa_1 
    # 询问密钥目录名称。默认目录及名称为 ~/.ssh/id_rsa
    # 可以自定义名称，但不能与其他密钥重名。
    # 这里使用 ~/.ssh/id_rsa_1，以便区分不同账号对应的密钥。
Enter passphrase (empty for no passphrase): 
    # 询问是否要为该密钥设置密码，如果不需要密码，回车即可跳过。
Enter same passphrase again: 
    # 重复密码，如果不需要密码，回车即可跳过。
```

生成RSA密钥后，请在对应账号的 Github Settings 里新增 SSH Key，并复制粘贴对应账号的公钥作为新的 SSH key。[相关步骤可以查询官方手册](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account)。然后重复上面的步骤，为另一个Github账号```second_account@email.com```创建密钥：

```bash
ssh-keygen -t rsa -b 4096 -C "second_account@email.com" 
    # 为Github账号first_account@email.com创建密钥。

Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa): ~/.ssh/id_rsa_2 
    # 询问密钥目录名称。默认目录名称为 ~/.ssh/id_rsa
    # 可以自定义名称，但不能与其他密钥重名。
    # 这里使用 ~/.ssh/id_rsa_2，以便区分不同账号对应的密钥
Enter passphrase (empty for no passphrase): 
    # 询问是否要为该密钥设置密码，如果不需要密码，回车即可跳过。
Enter same passphrase again: 
    # 重复密码，如果不需要密码，回车即可跳过。
```

此时在默认目录~/.ssh/下，有两对密钥，分别对应 first_account@email.com 和 second_account@email.com。在他们各自的Github Settings里，也已经添加了对应的公钥。

## 2. 配置ssh

完成了上面的步骤后，需要配置```~/.ssh/config```，用来指引```git```找到正确的Github账户。

```bash
touch ~/.ssh/config # 如果还没哟config文档，则需要创建一个新的
```

添加或修改如下内容到 ```~/.ssh/config```

```bash
# github.com: first_account@email.com
Host git1 # 此处可以自定义，用来在git参数中与其他账户做区分，因而不能与其他账户相同。
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_1 # 该账户对应的私钥

# github.com: second_account@email.com
Host git2 # 此处可以自定义，用来在git参数中与其他账户做区分，因而不能与其他账户相同。
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_2 # 该账户对应的私钥
```

配置完成后，通过在```git```命令中使用不同的参数来对相应的Github账号进行操作。

## 3. 使用示例

下文的示例使用```git```命令```push```代码到各自的Github账号：

```bash
cd ~/project_1
git push git1:first_account_username/project_1 master

cd ~/project_2
git push git2:second_account_username/project_2 master
```

其他的```git```命令，以此类推，使用参数来区分Github账号即可。

## 4. 坑点重申

* 使用多个Github账号时，不宜使用```--global```对```user.email```和```user.name```进行全局配置。请在各自的```repo```目录下使用```--local```指定参数，例如：

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

## 5. 参考资料

* [GithubHelp](https://help.github.com)
* [一台电脑绑定两个github帐号教程](https://www.jianshu.com/p/3fc93c16ad2d)
* [Multiple github accounts on the same computer?](https://stackoverflow.com/questions/3860112/multiple-github-accounts-on-the-same-computer?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)