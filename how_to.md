# macOS Tips

## It's better to use Homebrew's python even there is an apple python 2018/08/16
```pip install python``` for python3
```pip install python@2``` for python2

## zsh: non-standard shell error 2018/08/16
Getting the error when trying to change mac terminal to use zsh:
```chsh: /usr/local/bin/zsh: non-standard shell```
Open ```/etc/shells``` by ```sudo vim /etc/shells```
Added  ```/usr/local/bin/zsh``` to the end.

## pandas: RuningTimeWarning about numpy.dtype size changed 2018/08/16
```pip install --no-binary all```