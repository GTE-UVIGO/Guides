# Code packaging with Conda
This guide shows how to use **Conda** to package code projects and distribute them 
through a private conda channel built locally by the user.

**This guide assumes a Windows OS**.

## Preparing conda
Create a conda environment (named `Build` in this example) including the `conda-build` 
required package. It will be used for creating and managing the channel, as well as 
for code packaging:
```shell
conda create -n Build conda-build
```
Now choose a local directory where to create the custom channel. In this example, the 
channel will be named `custom-channel`, and it will be located at 
`C:\Users\ExampleUser\Conda`.

**Note:** Make sure the channel's main directory is not included inside a Git local 
repository or inside any IDE's project path. Make sure the main directory is not too 
deeply nested.

**Note:** the channel's main directory will store all the `tar.bz2` compressed 
packages you build, as well as some `JSON`, `HTML` and other files used by Conda to 
manage the channel's data. The memory requirements should not be too large (i.e. 1 MB 
approx for a channel storing 16 compressed packages).

**Optional:** Add **conda-forge** to the list of default channels in the `.condarc` 
configuration file:
```shell
conda config --add channels conda-forge
```

## Building the local channel
Now activate the environment, and register the channel's main directory using the 
`index` method:
```shell
conda activate Build
conda index C:\Users\ExampleUser\Conda\custom-channel
```
Now register this newly indexed directory as a custom channel in the `.condarc` 
configuration file:
```shell
conda config --set custom_channels.custom-channel file:///C:/Users/ExampleUser/Conda
```
**Note:** In the previous step only, make sure the `/` _slash_ character is used in 
the path definition, instead of the _backslash_ one.

Add your local channel to the list of default channels. **Make sure it is listed first** 
of all in the `.condarc` file, so conda will look there first when you ask to install 
one of your own packages.
```shell
conda config --add channels custom-channel
```
Finally, set your local channel's main path as the output folder for the `conda-build` 
tool:
```shell
conda config --set conda-build.output_folder: C:\Users\ExampleUser\Conda\custom-channel
```
**Note:** Sometimes, trying to make changes to the `.condarc` configuration file 
using the `conda config` command in a terminal may not work. For these cases, the 
`.condarc` may be manually edited using any plain-text editor. This configuration is 
usually found in the user's main directory (as a hidden file).

After applying all these previous steps, the contents of the `.condarc` file should look 
like this (assuming the optional conda-forge-channel-setting step was also applied):
```text
channels:
  - custom-channel
  - conda-forge
  - defaults
custom_channels:
  custom-channel: file:///C:/Users/ExampleUser/Conda
conda-build:
  output_folder: C:\Users\ExampleUser\Conda\custom-channel
```

## Code packaging
Once the local conda channel is operative, local code projects may be 
packaged (_distributed_) and added to the channel, so conda can install them different 
environments.

We will assume the code project to distribute has the required structure and 
distribution files. Different options are available for distributing code packages 
with conda, but this guide will assume the following files are located in the code 
project's main directory (all filled as required):
  - `conda.recipe/meta.yaml`
  - `conda.recipe/run_tests.py`
  - `pyproject.toml`
  - `setup.cfg`
  - `setup.py`

Assuming the working directory is set to the main directory of the code project to 
package (where the `src` and `conda.recipe` paths are):
```shell
conda activate Build
conda build conda.recipe
```
Conda will then start the packaging process. Depending on the concrete tasks to do 
(building a test environment, running code tests, uploading the package to a public 
remote repo, etc.) this may take several minutes. After conda finishes its run, the 
package should be ready to use.

**Optional:** The following command can be executed to clean debugging and temporal 
files created during the packaging:
```shell
conda build purge
```
Finally, run the following line to ensure the package was correctly created and 
registered in the local channel:
```shell
conda search PAQUETE -c custom-channel --override-channels
```

## Manually adding packages to the channel
Packages available in a local channel may be updated by manually adding or removing 
the compressed `tar.bz2` files of the target packages. After manual changes, the 
channel must be re-indexed:
```shell
conda index C:\Users\ExampleUser\Conda\custom-channel
```
Now the manually added/removed packages should appear in the `index.html` file.