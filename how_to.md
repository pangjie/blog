# macOS Tips

## It's better to use Homebrew's python even there is an apple python. 2018/08/16

```pip install python``` for python3

```pip install python@2``` for python2

## zsh: non-standard shell error 2018/08/16

Getting the error when trying to change mac terminal to use zsh:

```chsh: /usr/local/bin/zsh: non-standard shell```

Open ```/etc/shells``` by ```sudo vim /etc/shells```

Added  ```/usr/local/bin/zsh``` to the end.

## pandas: RuntimeWarning: numpy.dtype size changed, may indicate binary incompatibility. 2018/08/16

```pip install --no-binary all```

[StarOverflow Reference](https://stackoverflow.com/questions/40845304/runtimewarning-numpy-dtype-size-changed-may-indicate-binary-incompatibility)

It's only on Py2. Py3 doesn't have this warning.

Reinstall with previous version also works. ```numpy<=1.14.5``` or ```pandas<=0.23.0``` 

## Tensorflow: pip install tensorflow 2018/08/16

Note the version of python. Python2 works fine with ```pip install tensorflow```. It doesn't work for Python3.7 for now.

## Visual Studio Code with oh-my-zsh style Terminal. 2018/08/16

Add font [**Menlo for Powerline**](https://github.com/abertsch/Menlo-for-Powerline)

Config the user setting:

```json
{
  "terminal.integrated.shell.osx": "/usr/local/bin/zsh",
  "terminal.integrated.fontFamily": "Menlo for Powerline",
}
```