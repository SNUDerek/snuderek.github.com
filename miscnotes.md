## using `pip` in `conda` virtual environments

for the primary guide to using `conda` environments, see the [official guide](https://conda.io/docs/user-guide/tasks/manage-environments.html)

however, i noticed that my system has the problem that even when in my environment, `pip` is run from the base evironment, as mentioned [in this issue](https://github.com/ContinuumIO/anaconda-issues/issues/1429)

in order to maintain different environments with different versions of e.g. `tensorflow`, `keras` and `keras-contrib`, i needed to use the solution given:

1. install `pip` to the environment as detailed in the official guide: `conda install -n myenv pip`  
2. get environments list with `conda info --envs`  
3. use the _full path to pip_ to install:  

`(oldkeras) derek@mymachine:~$ miniconda3/envs/oldkeras/bin/pip install --upgrade Keras==2.0.6`

...this seemed to help me maintain two versions of `Keras` for backwards compatibility

## managing `conda` virtual environments with `jupyter lab`

[this site](http://anbasile.github.io/programming/2017/06/25/jupyter-venv/) details the instructions i used

1. create a `conda env`
2. `pip install ipykernel` (using the full `pip` path if needed, as described above)
3. activate the environment with `source activate myenv`
4. `ipython kernel install --user --name=<kernelname>`

...this will require a restart of the `jupyter lab` server
