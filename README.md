# KhorzooKhan
List of useful notes for developers

# Package management tips
## pip 

You may encounter this error
```
pip3: bad interpreter: No such file or directory
```
Which means your pip is indicating to a python installation which does not exist. What you have to do is open pip bash script using

```
sudo gedit /home/USERNAME/.local/bin/pip
```
And change the first commented line which is indicating to the python installation. 

