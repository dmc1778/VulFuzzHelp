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

# Compilation

## Build Pytorch from source

One major limitation of pytorch installtion using typical package managers such as pip and conda is that you don't have access to different components of pytorch. For example, one may want to run pytorch tests which is not possible in regular way. So, it is highly recommend to build from source.

In order to build from source, you need to follow the following instructions:

### Clone the repository
```
Git clone https://github.com/pytorch/pytorch.git
```
Then you have to change directory to pytorch repository:
```
Cd pytorch_dir
```

### Create build files

In this step, you need to create build files using:

```
python setup.py build --cmake-only
```
Then start configure the build files:

```
ccmake build 
```

### Installation

Finally, you can install pytorch using the following command:

```
MAX_JOBS=4 python setup.py install --user
```

Please note that, it is highly recommend that use ```MAX_JOBS``` which reduce the memory usage and most likely your compilation will not crashes. Also, you need to indicate python that you want to install for current user ```--user```.
