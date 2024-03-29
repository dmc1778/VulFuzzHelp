# VulFuzz
Fuzzing Deep Learning Libraries Using Vulnerability Knowledge from Open Source

# How instrumentation in FreeFuzz works?

## Issue in the instrumentation of tensorflow APIs called in unit tests

I found the issues. The issue is that tensorflow APIs called via tensorflow unit tests have different API names compared to APIs listed in the official documentation. For example when you run:

```
python broadcast_to_ops_test.py
```

Your code hijacks the class name inside this test file which is BroadcastToTest and records main as the parent module due to this piece of code:

```
if __name__ == "__main__":
  test_lib.main()
```

## When calling function APIs

When we import tensorflow by writing ````import tensorflow as tf``` or ``` import TensorFlow``, inside a script or through the command line, all high-level
modules as well as functions in each module are set inside the tensorflow object. 

When calling an API, e.g. class or function, the API information is set into the object, for example by calling:

```
import tensorflow as tf
print(tf.math.floor(1.5))
```
We set an instantiation of ```floor``` function from ```math``` object into root tensorflow object.

Once we have all attributes and functions inside the tensorflow object, we can get or set them. For example, we can get ```floor``` from ```math``` module by running:

```
import tensorflow as tf
s=tf.math.floor(5.5)
moduleobj = getattr(tf, 'math')
function_obj = getattr(moduleobj, 'floor')
```
You can get all information you need for further processing. 

# Package management tips
## pip 

You may encounter this error
```
pip3: bad interpreter: No such file or directory
```
That is, your pip is pointing to a Python installation that does not exist. You must first launch the pip bash script:

```
sudo gedit /home/USERNAME/.local/bin/pip
```
And change the first commented line which is indicating to the Python installation. 

# FSE23
## Get statistics
We need to count the overlaps among covered APIs by each source. As a result, every time we run API calls for each source, we have to
change the __init__.py file in the DL library root patch. For example, for PyTorch:

```
import pymongo

"""
You should configure the database
"""
torch_db = pymongo.MongoClient(host="localhost", port=27017)["Torch"]

def write_fn(func_name, params, input_signature, output_signature):
    params = dict(params)
    out_fname = "torch." + func_name
    params['input_signature'] = input_signature
    params['output_signature'] = output_signature
    params['source'] = 'docs'
    torch_db[out_fname].insert_one(params)
   
	
```

# Database
## MongoDB
### Change data dir to home (normal installation)
In order to change Mongodb data directory, please follow:
First, run:
```
sudo nano /etc/mongodb.conf
```
Then, change:

```
# Where and how to store data.
storage:
  dbPath: /home/nimashiri/mongodb
#  engine:
#  wiredTiger:
```
You need to give mongodb permission to the new path:

```
sudo chown mongodb:mongodb /new/path
```

### Use binaries

I tried several ways to change the data directory of MongoDB using standard installation, but since I am using a mounted drive with insufficient privileges, still I can't write to the new data directory. Hence, I had to use pre-compiled binaries since I can define data and log manuals from scratch with all RWX permissions. 

# Compilations

## Build Pytorch from source

One significant limitation of installing Pytorch using standard package managers such as pip and conda is that you cannot access all of Pytorch's components. For example, one may want to run PyTorch tests, which is not normally possible. As a result, building from a source is strongly advised. 

In order to build from the source, you need to follow the following instructions:

### Clone the repository
```
Git clone https://github.com/pytorch/pytorch.git
```
Then you have to change the directory to the Pytorch repository:
```
Cd pytorch_dir
```

Now check out the version of Pytorch you want to use. We are using ```v1.8.0```:

```
git checkout v1.7.0
```

Now, you have to run the following commands to clone third-party libraries:

```
git submodule sync
git submodule update --init --recursive --jobs 0
```

### Create build files

In this step, you need to create build files using:

```
python setup.py build --cmake-only
```
Then start configuring the build files:

```
cmake-gui build
```
Please note that you have to configure and generate the configurations. 

### Installation

Finally, you can install PyTorch using the following command:

```
MAX_JOBS=4 python setup.py install --user
```
Please note that it is highly recommended that use ```MAX_JOBS``` which reduces memory usage and most likely your compilation will not crash. Also, you need to indicate the python that you want to install for the current user ```--user```.
Then run the following command to make the executables:
```
python setup.py develop
```

# Query Stackoverflow
```
select p.id,
p.PostTypeId,
p.AcceptedAnswerId,
p.ParentId, 
p.CreationDate,
p.LastEditDate,
p.Score,
p.ViewCount,
p.Title,
p.Body,
p.tags,
p2.CreationDate,
p2.LastEditDate,
p2.Score,
p2.Body
from Posts p
join Posts p2 
on p.PostTypeId=1 
and p2.PostTypeId=2
and p.AcceptedAnswerId=p2.id
where p.tags LIKE '%python%' and p.AcceptedAnswerId !='' 
and p.ViewCount < XXXX
and p.Score >=0
order by p.ViewCount DESC;
```
