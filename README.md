# VulFuzz
Fuzzing Deep Learning Libraries using Vulnerability Knowledge from Open Source

# How instrumentation in FreeFuzz works?

## When calling function APIs

When we import tensorflow by writing ````import tensorflow as tf``` or ```import tensorflow```, inside a script or through command line, all high-level
modules as well as functions in each module are set inside tensorflow object. 

When calling an API, e.g. class or function, the API information is set into the object, for example by calling:

```
import tensorflow as tf
print(tf.math.floor(1.5))
```
We set an instatiation of ```floor``` function from ```math``` object into root tensorflow object.

Once we have all attributes and functions inside tensorflow object, we can get or set them. For example, we can get ```floor``` from ```math``` module by running:

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
And change the first commented line which is indicating to the python installation. 

# FSE23
## Get statistics
We need to count the overlapp among covered APIs by each sources. As a result, everytime we run API calls for each source, we have to
change the __init__.py file in DL library root patch. For example, for pytorch:

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
## Mongodb
### change data dir to home (normal installation)
In order to change mongodb data directory, please follow:
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

I tried several ways to change data directory of mongodb using normal installation, but since I am using a mounted drive with insufficient priviligaes, still I can't write to new data directory. Hence, I had to use pre compiled binaries since I can define data and log directories from scratch with all RWX permissions. 

# Compilations

## Build Pytorch from source

One significant limitation of installing Pytorch using standard package managers such as pip and conda is that you do not have access to all of Pytorch's components. For example, one may want to run pytorch tests, which is not normally possible. As a result, building from source is strongly advised. 

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
# Query stackoverflow
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
