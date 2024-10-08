# Python Interface

:::{tip}
Silo's python interface is designed to compile and work with either Python 2 or Python 3
:::

Silo's configure logic looks for `python`, not `python3`.
If you need it to look instead for `python3`, this works...

```
env PYTHON=python3 PYTHON_CPPFLAGS=-I<path-to-python-headers> ./configure --enable-pythonmodule
```

where `path-to-python-headers` can often be obtained by...

```
% python3
Python 3.8.9 (default, May 17 2022, 12:55:41) 
[Clang 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sysconfig
>>> sysconfig.get_config_var("INCLUDEPY")
'/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.8/Headers'
```

:::{warning}
Configuring with `--disable-shared` or `--enable-shared=no` will disable the python module
:::

It is probably easiest to understand the Python interface to Silo by examining some examples and tests.
In the source code distribution, you can find some examples in tools/python and tests in tests directories.
Here, we briefly describe Silo's Python interface.

The Python interface will be in the `lib` dir of the Silo installation, named `Silo.so`.
To use it, Python needs to be told where to find it.
You can do this a couple of ways; through the `PYTHONPATH` environment variable or by explicitly adding the Silo installation `lib` dir to Python's path using `sys.path.append()`.

For example, if Silo is installed to /foo/bar, this works...

```
% env PYTHONPATH=/foo/bar/lib python
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple `LLVM` 7.0.0] on darwin
Type "help", "copyright", "credits" or "license" for more info.
>>> import Silo
```

Or, if you prefer to use `sys.path.append`...

```
% python
Python 2.7.10 (default, Oct 23 2015, 19:19:21)
[GCC 4.2.1 Compatible Apple `LLVM` 7.0.0] on darwin
Type "help", "copyright", "credits" or "license" for more info.
>>> import sys
>>> sys.path.append("/foo/bar/lib")
>>> import Silo
```

{{ EndFunc }}

## `Silo.Open()`

* **Summary:** Open a Silo file (See [`DBOpen`](./files.md#dbopen))

* **C Signature:**

  ```
  DBfile Silo.Open(filename, flags);
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `filename` | Name of the Silo file to open
  `flags` | Pass either `Silo.DB_READ` if you will only read objects from the file or `Silo.DB_APPEND` if you need to also write data to the file.

* **Description:**

  Returns a `DBfile` object as a Python object

{{ EndFunc }}

## `Silo.Create()`

* **Summary:** Create a new silo file (See [`DBCreate`](./files.md#dbcreate))

* **C Signature:**

  ```
  DBfile Silo.Create(filename, info, driver, clobber)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `filename` | [required string] name of the file to create
  `info` | [required string] comment to be stored in the file
  `driver` | [optional int] which `driver` to use. Pass either `Silo.DB_PDB` or `Silo.DB_HDF5`. Note that advanced `driver` features are not available through the Python interface. Default is `Silo.DB_PDB`.
  `clobber` | [optional int] indicate whether any existing file should be clobbered. Pass either `Silo.DB_CLOBBER` or `Silo.DB_NOCLOBBER`. Default is `Silo.DB_CLOBBER`.

* **Description:**

  Returns a `DBfile` object as a Python object

{{ EndFunc }}

## `<DBfile>.GetToc()`

* **Summary:** Get the table of contents

* **C Signature:**

  ```
  DBtoc <DBfile>.GetToc()
  ```

* **Description:**

  Returns a `DBToc` object as a Python object.
  This probably should really be a Python dictionary object but it is not presently.
  There are no methods defined for a `DBToc` object but if you print it, you can get the list of objects in the current working directory in the file.

{{ EndFunc }}

## `<DBfile>.GetVarInfo()`

* **Summary:** Get metadata and bulk data of any object (See [`DBGetObject`](./generic.md#dbgetobject))

* **C Signature:**

  ```
  dict <DBfile>.GetVarInfo(name, flag)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of object to read
  `flag` | [optional int] `flag` to indicate if object bulk/raw data should be included. Pass 0 to **not** also read object bulk/raw data. Pass non-zero to also read object bulk/raw data. Default is 0.

* **Description:**

  Returns a Python dictionary object for a Silo high level object (e.g. not a primitive array).
  This method cannot be used to read the contents of a primitive array.
  It can be used for any object the Silo C interface's `DBGetObject()` would also be used.
  If object bulk data is not also read, then the dictionary members for those sub-objects will contain a string holding the path of either a sub-object or a primitive array.
  Note that on the HDF5 driver, if friendly HDF5 names were not used to create the file, then the string paths for these sub-objects are often cryptic references to primitive arrays in the hidden `/.silo` directory of the Silo file.

  This method is poorly named.
  A better name is probably `GetObject`.

{{ EndFunc }}

## `<DBfile>.GetVar()`

* **Summary:** Get a primitive array (See [`DBReadVar`](./generic.md#dbreadvar))

* **C Signature:**

  ```
  tuple <DBfile>.GetVar(name)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of primitive array to read

* **Description:**

  This method returns a primitive array as a Python tuple

{{ EndFunc }}

## `<DBfile>.SetDir()`

* **Summary:** Set current working directory of the Silo file (See [`DBSetDir`](./files.md#dbsetdir))

* **C Signature:**

  ```
  NoneType <DBfile>.SetDir(name)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of directory to set

* **Description:**

  Sets the current working directory of the Silo file

{{ EndFunc }}

## `<DBfile>.Close()`

* **Summary:** Close the Silo file

* **C Signature:**

  ```
  NoneType <DBfile>.Close()
  ```

* **Description:**

  Close the Silo file

{{ EndFunc }}

## `<DBfile>.WriteObject()`

* **Summary:** Write a Python dictionary as a Silo object (See [`DBWriteObject`](./generic.md#dbwriteobject))

* **C Signature:**

  ```
  NoneType <DBfile>.WriteObject(name, obj_dict)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of the new object to write
  `obj_dict` | [required dict] Python dictionary containing object data

* **Description:**

  This method will write any Python dictionary object to a Silo file as a Silo object.
  Here's the rub.
  Readers employing Silo's high level interface (e.g. `DBGetUcdmesh`, `DBGetQuadvar`, etc.) will be able recognize an object so written if and only if the dict object's structure *matches* a known high-level Silo object.

  So, you can use this method to write objects that can be read later via Silo's high-level object methods such `DBGetUcdmesh` and DBGetMaterial, etc.
  as long as the Python dictionary's members match what Silo expects.

  Often, the easiest way to interrogate how a given Python dict object should be structured to match a Silo object is to find an example object in some file and read it into Python with `GetVarInfo()`.

  It is fine to create a dict object with additional members too.
  For example, if you create a dict object that is intended to be a Silo material object, you can add additional members to it and readers will still be able to read it via `DBGetMaterial`.
  Of course, such readers will not be aware of any additional members so handled.

  It is also fine to create wholly new kinds of Silo objects for which there are no corresponding high-level interface methods such as `GetUcdmesh` or `GetQuadvar` in the C language interface.
  Such an object can be read by the generic object, `DBGetObject()` C language interface method.

{{ EndFunc }}

## `<DBfile>.Write()`

* **Summary:** Write primitive array data to a Silo file (see DBWrite)

* **C Signature:**

  ```
  NoneType <DBfile>.Write(name, data)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of the primitive array
  `data` | [required tuple] the `data` to write

* **Description:**

  This method will write a primitve array to a Silo file.
  However, it presently handles only one dimensional tuples.
  Furthermore, the tuples must be consistent in type (e.g. all floats or all ints).

{{ EndFunc }}

## `<DBfile>.MkDir()`

* **Summary:** Make a directory in a Silo file

* **C Signature:**

  ```
  NoneType <DBfile>.MkDir(name)
  ```

* **Arguments:**

  Arg name | Description
  :---|:---
  `name` | [required string] `name` of the directory to create

* **Description:**

  Creates a new directory in a Silo file

{{ EndFunc }}
