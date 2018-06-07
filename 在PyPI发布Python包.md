# 在PyPI发布Python包

PyPI在2017年修改了发布规则，到2018年6月，许多发包攻略并未更新，因此在这里补充一点内容后，总结了一个简化的发布流程。[所有步骤的详细信息可在PyPI.org的官方发包指南中找到](https://packaging.python.org/guides/distributing-packages-using-setuptools)。简单地说，发布一个Python包到PyPI，需要编写代码，配置```setup.py```, 然后注册PyPI账号并打包上传。

#### 1. 基本包结构的说明，以及配置```setup.py```（介绍包结构要求。解释```setup.py```的内容）
#### 2. 注册PyPI以及TestPyPI，并配置 ```~/.pypirc```（发布准备工作）
#### 3. 打包+发布

---

## 1. 基本包结构的说明，以及配置```setup.py```

在发布Pyhton包之前，需要建立PyPI要求的包结构，并配置好```setup.py```。首先以发布一个名为```helloworld```的Python包为例，一个简单的包结构如下：

```
helloworld
    │
    ├── LICENSE  
    │       # 软件使用许可证，例如MIT
    │
    ├── MANIFEST.in 
    │       # 打包软件生成源码包时，通常只打包特定类型的文件，比如*.py
    │       # 如果需要打包csv等其他特异类型的文件，需要在这个文件中声明。
    │       # Python2.6之前的版本中，特异文件类型必须在这里声明。
    │       # Python2.7之后的版本则可以在setup.py中加入特异文件类型的声明。
    │
    ├── README.md 
    │       # 说明文档，可以显示在 PyPI Project Description
    │       # PyPI Project Description 支持Markdown格式，但需要使用新版的发布工具：
    │       #     setuptools > 38.6.0, twine > 1.11.0, wheel > 0.31.0
    │       # 并需要在setup()中增加字段：
    │       #     long_description="""# Markdown supported!""",
    │       #     long_description_content_type='text/markdown',
    │       # 升级发布工具 pip install -U twine
    │
    ├── helloworld
    │       │
    │       ├── __init__.py
    │       │       # 如无需求，留白即可。
    │       │
    │       └── helloworld.py
    │               # 实质的Python代码。
    │
    ├── setup.cfg 
    │       # setup.py 的配置信息，如果没有配置需求，留白即可。
    │
    └── setup.py 
            # 包含发布包所需的全部关键信息，将在下文进行说明。
```

基于上面介绍的基本包结构建立好之后，需要配置```setup.py```，这是发布过程中最关键的步骤，下面将提供一个简单的样板：

```python
from setuptools import setup, find_packages
from os import path
from io import open

here = path.abspath(path.dirname(__file__))
with open(path.join(here, 'README.md'), encoding='utf-8') as f:
    long_description = f.read()
    # 读取项目根目录下的 README.md 作为PyPI项目描述（long_description） 

setup(
    name='sampleproject', 
        # Required 包名称
    version='1.2.0', 
        # Required 包版本, 同一个版本号只能发布一次, 无论是否删除已发布包。
    description='A sample Python project', 
        # Required 简单的包说明，显示在pip search的说明中。
    long_description=long_description,  
        # Optional 可以将包的说明文档 Readme.md 显示在PyPI的项目描述中。
    long_description_content_type='text/markdown',  # Optional
        # Optional 如果使用 Markdown 格式编写 Readme, 需要在此声明
    url='https://github.com/someone/sampleproject',  
        # Optional 项目的主页
    author='The Python Packaging Authority',
        # Optional 项目的作者
    author_email='pypa-dev@googlegroups.com',
        # Optional 项目作者的邮箱
    packages=find_packages(),
        # Required 声明所需要发布的文件目录,一般使用 find_packages(), 也可手动声明。
    install_requires=['somepackagename'],
        # Optional 声明所有第三方依赖包。
    entry_points={
        'console_scripts': [
            'helloworld=helloworld:main',
        ],
        # 如果此包可以单独运行，例如发布的是一个命令行工具CLT(Command Line Tool)
        # 则需要在此指出 entry_points。例如在样板中申明了单独执行时运行 helloworld 时
        # 执行 helloworld.py 的 main()。
    },
)
```
配置完成后，使用 ```python setup.py check``` 来检测```setup.py```是否符合发布要求。如果没有任何反馈，则说明```setup.py```符合发布要求。然后请在项目的根目录用 ```pip install -e .``` 进行开发模式下的测试。如果测试达到设计要求，就可以进行发布前的准备工作了。

## 2. 注册PyPI以及TestPyPI, 配置 ```~/.pypirc```

* [PyPI](https://pypi.org): https://pypi.org/

* [TestPyPI](https://test.pypi.org): https://test.pypi.org/

TestPyPI是PyPI的一个测试实例, 用于发布的测试工作, 与PyPI的功能完全一致, 但不会对PyPI的发布造成任何影响。请分别注册[PyPI](https://pypi.org)和[TestPyPI](https://test.pypi.org)。二者可以使用相同的用户名和密码, 但无法使用相同的验证邮箱。注册完毕后，还需要配置 ```~/.pypirc```，以便在接下来的步骤中对正式发布与测试发布进行区分。


```
[distutils]
index-servers=
    pypi
    testpypi # 很多人设置成pypitest, 这里用testpypi是为了减少输入参数时的误操作几率。

[pypi]
repository: https://upload.pypi.org/legacy/ 
    # 这里不推荐标明repository, 更加保险的方式是交给twine自己解决。在过去的版本中, 这一地址曾数次更动, 会导致发布失败, 因此不指明地址, 交给twine, 并保持更新twine更可靠。
username: real_account 
    # pypi的注册用户名
password: 123456 
    # 密码字段可以不写, 在发布时再输入

[testpypi]
repository: https://test.pypi.org/legacy/ 
    # 测试包发布仓库。
username: test_account 
    # pypi的注册用户名
password: 654321 
    # 密码字段可以不写, 在发布时再输入
```

## 3. 打包+发布

在2017年6月以前, 需要先使用 ```python setup.py register``` 然后再进行发布。但2017年9月后，新的发布规则已经去掉了这一步骤。到2018年6月为止，官方手册推荐用```twine``` 进行发布，所以推荐用```pip instal twine```进行安装。在发布前，请先在[PyPI](https://pypi.org)中搜索包的名称, 以免因为同名而出现403错误。

1. 使用 ```python setup.py sdist``` 生成发布用的源码包。生成的源码包会出现在 ```dist/helloworld-0.0.1.tar.gz```
2. 进行测试发布：```twine upload -r testpypi dist/helloworld-0.0.1.tar.gz```
3. 在[TestPyPI](https://test.pypi.org)检验是否发布成功
4. 正式发布 ```twine upload -r pypi dist/helloworld-0.0.1.tar.gz```
5. 查看[PyPI](https://pypi.org)检验发布是否成功
6. 如果在[PyPI](https://pypi.org)搜索包名称, 或者使用 ```pip search yourpackagename``` 搜索包名称无果, 请稍安勿躁, 等待一段时间即可(数个小时)。

## 4. 参考资料

* [Packaging and distributing projects](https://packaging.python.org/guides/distributing-packages-using-setuptools/#setup-cfg)
* [Publishing your First PyPI Package by/for the Absolute Beginner](https://jonemo.github.io/neubertify/2017/09/13/publishing-your-first-pypi-package/)
* [Python发布包到PyPI的踩坑记录](http://www.cnblogs.com/rongpmcu/p/7662821.html)
* [PyPi description markdown doesn't work](https://stackoverflow.com/questions/26737222/pypi-description-markdown-doesnt-work?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
