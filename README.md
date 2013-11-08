MultiRun v 0.0.9
==========

A Matlab tool for facilitating parameter calibration of DETIM/DEBaM.
Given a valid ```input.txt```, MultiRun can take arbitrary ranges of 
arbitrary numbers of model
parameters, and then execute DEBaM or DETIM for every permutation
of these parameters. Model output is directed into folders with
unique alphanumeric names to ensure that output data is not overwritten,
and that lengthy model runs do not need to be repeated. 

As of version 0.0.8, MultiRun is compatible with versions 2.x.x of DEBaM and DETIM,
and incompatible with earlier versions.  Version 0.0.7, which is compatible
with earlier versions on the models, is still available
[here](https://github.com/fmuzf/matlab_hk_MultiRun/releases).


### What you need:
  1. Compiled versions of DEBaM or DETIM.
  2. A valid ```input.txt``` file for the models. In this file, you
     should set all of the parameters you would like the model
     to use, with the exceptions of those MultiRun will change for each model run.
  3. Model inputs to go along with the model. See the model documentation for
     setting these up.
 

Installation
------------
1. Download and compile the latest version of
    [DEBaM and DETIM](https://github.com/regine/meltmodel).
    MultiRun supports versions 2.x.x of the models.
    If you already have a copy on your computer, there is no
    need the re-download the model, just use the copy you
    currently have.
2. Download MultiRun, either with ```git```, or download the
    [zipball](https://github.com/fmuzf/matlab_hk_MultiRun/archive/master.zip).
3. *Important*: Move the folder ```+MultiRun``` into your Matlab Path; it's
    not enough to navigate to the containing folder with Matlab, as the script
    changes the working directory. Mathworks has helpful documentation 
    [about the Matlab Path](http://www.mathworks.com/help/matlab/matlab_env/what-is-the-matlab-search-path.html)
    and [how to change it](http://www.mathworks.com/help/matlab/ref/pathtool.html).
4. Check that Matlab can find MultiRun by trying:

    ```matlab
    MultiRun.modelMultiRun()
    ```
    in Matlab.  Your *should* get an error of the
    form: ```??? Input argument "config" is undefined.```
    If you get a different error, reinstall MultiRun.
5. You're done! MultiRun is now available to Matlab, and you can access
    its functions and classes via ```MultiRun.<function/classname>```.


Quick Start
-----------
1. Install MultiRun.
2. Configure an ```input.txt``` parameter file for DEBaM/DETIM. Set all of the
    parameters for the model run to the values you want them to be when the model
    is run, except for the values you will be change using MultiRun, these can be
    set to anything. 
3. Open Matlab.
4. Run the ```MultiRun.modelMultiRun``` command:

    ```matlab
    modelName = '/home/luser/meltmodel/bin/debam';
    configName = '/home/luser/my_glacier_folder/input.txt';
    [hashes, status, err, changes, runs] = MultiRun.modelMultiRun(modelName, configName, 'icekons', [5:0.1:6], 'rockkons', [0:0.1:0.5])
    ```
    Where you set ```modelName``` to the full file name of debam (detim, if you're
    going to run detim), on your machine, an ```configName``` to the full file name
    of your ```input.txt```. Also, replace ```'icekons', [5:0.1:6], 'rockkons', [0:0.1:0.5]```,
    with the parameters you want
    changed in your base ```input.txt```. The syntax should be familiar
    from Matlab's ```plot``` function, moreover you may assign to arbitrary number
    of parameters, over arbitrary ranges of values. In our example ```icekons```
    will take on every value in ```[5:0.1:6]``` and ```rockkons``` will take on
    every value in ```[0:0.1:0.5]```.
5. The output of each model run is located in a subdirectory of the ```outpath```
    specified in the base ```input.txt```. if ```outpath``` was set
    to ```/home/luser/modeloutput/```, you will find subdirectories
    of ```/home/luser/modeloutput/``` with long alpha-numeric names.
    Each directory contains:
    * The ```input.txt``` for *this* run of the model, with parameters changed from
    the base parameter file.
    * A ```changes.txt``` file which describes the changes made to the original ```input.txt```.
    * A directory ```outpath```, containing the output from this run of the model.
    This is ```outpath``` in ```input.txt``` for this model run.
    * A file ```runstatus.lock```. This assists ```MultiRun``` to prevent re-running
    this parameter configuration more than once. If you need to re-run this
    configuration, delete ```runstatus.lock``` before doing so.
8. In Matlab, ```modelMultiRun``` has returned quite a bit of information to you
     which you can use to further manipulate your finished runs. These are outlined
     in the API below.
9. Have Fun! 


API (Available Functions/Objects and How to Use them)
------
MultiRun provides a method for running the models over a range of
parameter values, as well as a helper class which manages each
individual run of the model, and ensures that the model is not
run more times than necessary.


## function modelMultiRun

To run the model multiple times the function ```modelMultiRun``` is available.

```[hashes, status, err, changes, runs] = MultiRun.modelMultiRun(modelpath, basefile, varargin)```

### Arguments: 

* ```modelpath``` - fully qualified path to the executable of the model you want to run (debam/detim).
* ```basefile``` - fully qualified path to a valid parameter file for the model, this will be modified based upon the the list of key-value
  pairs passed to varargin
* ```varargin``` - a list of key-value pairs which are modified, e.g.
 ```modelMultiRun('debam', 'input.txt', 'icekons', [5:0.1:6])```
 will run the model with ```icekons``` set to each value in ```[5:0.1:6]```.

### Returns:
* ```hashes``` - Cell array of hashes of each run
* ```status``` - status(i) = Array of return status of run with hash hashes{i}
* ```err``` - Error messages associated by incomplete runs
* ```changes``` - array of changes made to input.txt
* ```runs``` - a container.Maps indexed by hashes of HashedRun objects,
   each corresponding to a single model run.

### E.g.
Let ```base_input.txt```, be a valid parameter file for DEBaM/DETIM (aside from the file name),
and a copy of ```detim``` be located at ```/home/luser/local/bin/detim```.
In Matlab, we run:
```matlab
basefile = '/home/luser/base_input.txt';
modelpath = '/home/luser/local/bin/detim';

 [hashes, status, err, changes, runs] = MultiRun.modelMultiRun(modelpath, basefile, 'icekons', [5, 6.0], 'firnkons', [350, 351]);
```
at the command line. MultiRun will take the parameter file
from ```/home/luser/base_input.txt``` and generate ```input.txt```
 files which contain every combination of the parameters
```icekons``` and ```firnkons```, given the values you've assigned them
(in this case there are four combinations).
Each generated file is placed in a directory whose name is the SHA-1 hash
of the modified parameter file, and the model's output is directed to that folder as well.
If ```outpath``` is set to ```/home/luser/mytest``` in ```base_input.txt```,
this model run will result in a directory structure which looks like:

```
mytest/
└── output
    ├── a729f3b8529edf74e2d57cf64ba8cc91fc64907e
    │   └── outpath
    ├── cdad489ead2ec45681e9a5ec91516623c88433ce
    │   └── outpath
    ├── d5c3a35650dcde462cc5eaea301cfbb62cd050d9
    │   └── outpath
    └── fbf399de8a6ac388aad1647ad918d37f9a3d81f1
        └── outpath
```
After generating the parameter file, Matlab changes its
current working directory to the folder containing the parameter
file, and executes the command in ```modelpath``` (in our example ```detim```).
Model output is put into the folder ```<hash>/outpath```.

```modelMultiRun``` also writes a file ```changes.txt```
which lists the changes made to to the original parameter file for that run, i.e.:
```bash
$ cd mytest/output/a729f3b8529edf74e2d57cf64ba8cc91fc64907e/
$ ls
changes.txt     input.txt      outpath        runstatus.lock
$ cat changes.txt
Base parameter file: /home/luser/mytest/input.txt
New parameter file: /home/luser/mytest/output/a729f3b8529edf74e2d57cf64ba8cc91fc64907e/index.txt
Changes made:
  icekons = 6
  firnkons = 350
```

### What happens:
For each run, MultiRun uses the helper class ```MultiRun.HashedRun``` to
generate a nearly-unique alpha-numeric identifier for that particular model run.
This is done by:
- Generating an ```input.txt``` file from the passed parameters,
- Taking the SHA-1 hash of the text in this file
- Modifying the ```outpath``` parameter of the ```containers.Map``` containing
  the configuration to ```<original-outpath>/<SHA hash>/outpath/```.

multiModelRun returns several cell-arrays; entries with the same index correspond to the same model run
  1. ```hashes```: the SHA-1 hash of the ```input.txt``` parameter files
  2. ```status```: Returned status of the model run.
  3. ```err```: Sting detailing any errors encountered; is empty if none occur
  4. ```changes```: Strings listing any changes made to ```base_input``` for this run
  5. ```runs```: a ```containers.Map``` object, whose keys are the hashes of each run, and whose
    values are the HashedRun objects of each run.

- HashedRun checks to see whether or not there is already a parameter file in the new output path.
  This is done by checking a lockfile ```runstatus.lockfile```, the contents of which
  tell whether or not this parameter file has been run, is currently running, or is waiting to
  be run.
- If no lockfile is found, the new configuration file is written
  to disk at ```<original-outpath>/<SHA hash>/input.txt```.

- If a lockfile is found, it's status is returned and an error is posted.
- The model is run as a system subprocess, if any errors occur ```status[i]``` is
 set, and an error message is returned.
- In particular, if every entry in ```status``` is ```1```, we know that
 every model run exited successfully.


Each run is managed by a helper class called ```HashedRun```. The ```HashedRun``` class
manages the run; building a directory structure for that run alone, and ensuring that
each configuration is only run once.

## class HashedRun

Each run is managed by a ```HashedRun``` class, which makes the
appropriate directories, checks to see if the run has been completed
previously 


### Methods: 
  - ```hr = MultiRun.HashedRun(config, model)```
       Object constructor function
       __Arguments__: 
      * ```config```: The text of a valid Model 'input.txt'
      * ```model```: the fully-qualified path for the model executable

  - ```[success, err] = genConfig(self)```
       Generate this run's input.txt, and write it to disk.
       __Returns__: 
       * ```success``` : success code is
          - 0 something has gone wrong
          - 1 parameter file has been generated
          - 2 parameter file already existed
       * ```err```: error message, if empty everything is fine

  - ```[success, err] = runModel(self)```
       Execute the model, checking to make sure that the model hasn't run
       already.
       __Returns__:
       * ```success```: codes for completion are:
          - 0 : an error has occurred
          - 1 : The model has been run successfully
          - 2 : the lockfile indicates the model has
          already run
       * ```err```: error message.  

### Properties: 
  - ```configMap```: Map container containing info for the model run
  - ```hash```:  SHA-1 hash of input.txt corresponding to configMap
  - ```model```: Fully qualified path to model executable
  - ```outPath```L Path where model will be outputting
    


License
-------
BSD
