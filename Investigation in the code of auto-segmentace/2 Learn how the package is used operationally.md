### Plan  

Read:
- ~~`auto_segmentace.wiki/Package-Setup.md` + run~~
- ~~`auto_segmentace.wiki/How-to-Use-the-Package.md`~~
	- ~~config.yaml file description: [config.yaml](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/home/How-to-Use-the-Package/config.yaml)~~
	- ~~tuning_config.yaml file description: [tuning_config.yaml](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/home/How-to-Use-the-Package/tuning_config.yaml)~~
- ~~`pyproject.toml`~~
- ~~`Makefile`~~
- ~~`README.md`~~

Read code:
- ~~`monai_framework/cli/train_model.py`~~
- ~~`monai_framework/cli/tune_hp.py`~~
- ~~`monai_framework/cli/test_model.py`~~
- ~~`monai_framework/cli/export_model.py`~~
- ~~`monai_framework/cli/report_experiment.py`~~
- ~~`monai_framework/cli/fine_tune.py`~~
- `monai_framework/cli/cross_validation.py`

---

### Questions to focus on   
- What commands does the package expose?
- Which commands are normal workflow and which are optional?
- Which outputs matter to the rest of the team, especially export for the fullstack side?
##### Expected outcome:
	- you can draw the high-level workflow: train -> test -> export -> report,
	- you know which CLI file starts each workflow.*

### Notes from documentation review
##### Questions from documentation review
1. auto_segmentace.wiki\How-to-Use-the-Package.md row 34: I did not get  what means 'evaluate on more samples, since the mask does not have to contain all the labels for given patient.'
2. In the file auto_segmentace.wiki\How-to-Use-the-Package.md row 57: How the Experiment report works and what was it aimed for?
3. What is the goal of **inferer_overlap** and **background** in [config.yaml](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/home/How-to-Use-the-Package/config.yaml)?
	1. What smoothing is?
	2. Why such postprocessing params were selected?
##### UV, poetry, choco, make, pylint, mypy,  isort, black**

- `uv`: a fast Python package and environment manager used to create virtual environments, install dependencies, and keep installs reproducible. Connected output file: uv.lock
- `poetry`: a Python project manager used to define dependencies in [pyproject.toml](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html), manage environments, and package the project(creates **distributable files**).
- `choco` (Chocolatey): a Windows package manager used to install command-line tools like `make`.
- `make`: a task runner used to execute predefined project commands such as formatting, linting, and dependency export.
- `mypy`: a static type checker used to catch type-related errors before running the code.
	- Alternative: ruff
- `pylint`: a linter used to detect code quality problems, style issues, and some likely bugs.
- `black`: an automatic Python formatter used to enforce consistent code style with minimal configuration.
- `isort`: an import formatter used to sort and group Python imports into a consistent order.
##### UV sync 
 When both [pyproject.toml](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html) and [uv.lock](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html) exist, `uv sync` does not ignore the lockfile, but it also does not use the lockfile alone.

It uses them together:
- [pyproject.toml](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html) defines what the project wants: dependencies, extras, groups, Python version.
- [uv.lock](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html) provides the pinned resolved versions to install.
##### What clipping is?
A common example is image intensity clipping. If pixel values are supposed to stay in [0,255][0,255] or [0,1][0,1], anything below the minimum is set to the minimum, and anything above the maximum is set to the maximum. So if you clip to [0,255][0,255], then −10→0−10→0 and 300→255300→255.

In practice, clipping is mostly about preventing invalid or extreme values from dominating later steps. The tradeoff is that once values are clipped, information outside the range is lost.

### Notes from code review

##### train_model.py`
**Questions:**
1. config_dir - which data are optional and which ones required?
2.  rlimit = resource.getrlimit(resource.RLIMIT_NOFILE)
    resource.setrlimit(resource.RLIMIT_NOFILE, (2048, rlimit[1]))
    torch.multiprocessing.set_sharing_strategy("file_system")
    I am not sure why is that needed
###### resource.RLIMIT_NOFILE & set_sharing_strategy
TODO: investigate resource.RLIMIT_NOFILE 
When PyTorch uses multiprocessing, different processes need to **share tensors** (your data batches).

There are two main strategies:

1. "file_descriptor" (default)
Uses OS-level shared memory tracked by file descriptors
Fast
BUT: uses a lot of descriptors

2. "file_system"
torch.multiprocessing.set_sharing_strategy("file_system")
Uses temporary files on disk (or /tmp)
Fewer file descriptors needed
Slightly slower, but more robust
****
###### Flag --resume
`--resume` is a boolean flag that tells the program to continue an existing (training, hyperparameter tuning, etc) study instead of starting a new one.

Without `--resume`:
- `resume` is `False`
- the command is treated as a fresh tuning run
With `--resume`:
- `resume` becomes `True`
- `run_tuning(...)` is called with `resume_study=True`
- the code is expected to reconnect to an existing study, usually identified by `--study_name`
###### Flag --freeze_depth
I am not sure what was the puprose of the freezing in the training pipeline. But here is the possible use cases:
1. Training in stages: you might train different parts of a model at different times.
2. Stabilizing training: in complex systems (like GANs or multi-part models), you often freeze one component while training another.
But as the freezing was added 2 month ago it seems that it was done in order to use the fine_tuning with it only.
In the[ mlflow files](http://51.21.22.93/#/experiments/70/runs/a9f283615e654ee3a8f061630ebe3308) and [log files](https://cloud.ujp.cz/apps/files/files/10102532?dir=/AI_Segmentace/bundle_onnx/finetuned_cads/1e-4) for finetuning or training on CADS dataset I did not found any commands for calling .py files, so the exact command are not reproducible, so the experiments are barely reproducible.
##### `report_experiment.py`
It is a rapper on  run_report() which gathers the outputs that already exist after training and testing, such as TensorBoard logs, generated plots, configs, logs, and optional test result files, and turns them into a structured markdown report and supporting artifacts for either the local experiment folder or the wiki. The main goal of [run_report](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html) is to convert raw experiment results into a shareable, human-readable summary so the team can review, compare, and document experiments consistently from the CLI entry point in [report_experiment.py:60](vscode-file://vscode-app/c:/Users/aleksiutenko/AppData/Local/Programs/Microsoft%20VS%20Code/560a9dba96/resources/app/out/vs/code/electron-browser/workbench/workbench.html).

##### `fine_tune.py`
I am a bit confused that the fine_tune script do not have freeze_depth param(because it is not updated and the parameter is relatively new).

### Correction notes from documentation review
1. ~~Why the How-to-Use-the-Package.md not in the How-to-Use-the-Package dir?~~
2. ~~Add info that for installing pylint mypy black and isort run not uv sync row 14 auto_segmentace.wiki\Package-Setup.md but uv sync --extra dev.~~
3. ~~Adam said that we are not using poetry, so everywhere where there is a poetry is should be uv, probably(???).~~
4. ~~Package-Setup.md and Readme are having the duplicated info. Better to do a link on the setup file than duplicate the same info.~~
5. ~~In Package-Setup.md: Add to the Make installation that both commands should be runned from the Powershell runned as admin, not just basic powershell.~~
6. ~~In auto_segmentace.wiki\How-to-Use-the-Package.md: Hyper parameter -> hyperparameters, dataset cleaning -> Dataset cleaning(maybe preprocessing? I will check it)~~
7. In the file auto_segmentace.wiki\How-to-Use-the-Package.md row 18: I am not sure that the config contain postprocessing. UPD: https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/How-to-Use-the-Package/config.yaml?redirected_from=home/How-to-Use-the-Package/config.yaml contain postprocessing BUT https://gitlab.ujp.cz/ddud/auto_segmentace/-/blob/master/monai_framework/templates/config.yaml?ref_type=heads do not.
8.  In the file auto_segmentace.wiki\How-to-Use-the-Package.md row 27: example command do not include tuning_config.yaml, in row 24 config.yaml template is maintained already known info from the previous example on the row 17, I would delete it.
9. ~~In the file auto_segmentace.wiki\How-to-Use-the-Package.md: fine tuning -> finetuning~~
10. ~~In the file auto_segmentace.wiki\How-to-Use-the-Package.md ro 46: fine_tuned_model.pt in the example are missleading name as it seems that by the name it should be the output path where checkpoints are aimed to be setted.~~
11.  ~~In the file auto_segmentace.wiki\How-to-Use-the-Package.md: 'Fine tuning will provide a further training (usually on an end-user dataset to optimize the model's prediction according to the end-user's standards)' should be after example command, or should not be here.~~
12.  In the file auto_segmentace.wiki\How-to-Use-the-Package.md: '- set the dataset_path and structures (lines 671 and 672) before running the script.' should be unified with args and arparse as itis for the different cli scripts. 
13.  ~~In the file auto_segmentace.wiki\How-to-Use-the-Package.md row 62: hyperparametr -> hyperparameter~~
14. in [config.yaml](https://gitlab.ujp.cz/ddud/auto_segmentace/-/wikis/home/How-to-Use-the-Package/config.yaml)'List of structures to segment. This automaticaly configurates the pipeline as multiclass segmentation task if the lenght of the list is > 1. If single channel keep only one structure in the list.' I am not sure that the word "channel" is right here. Maybe class?
15. In the README.md file what submit job on helious' means?
16. 
### Correction notes from code review
1. In the file monai_framework\cli\train_model.py:
	1. Row  26 wheather ->whether
	2. ~~Row 38 MOANI-> Monai~~
	3. It seems that row 35 freeze_depth should be defined in monai_framework\cli\fine_tune.py. 
	   TODO: investigate
	4. def train and run_training should be probaly changed vice versa by names, as it is missleading. 
	   TODO: rename myb after deeper investigation in the current run_training
2. In the monai_framework\cli\test_model.py:
	1. It seems that the flag name single_test_structure does not properly displays the thing it does. It seems that it lets you limit testing to one structure, or a specific list of structures, instead of testing everything the model can evaluate.
	   TODO: rename it to test_structures, check if internaly it is a list(I saw a split after)
3. In the file monai_framework\cli\export_model.py:
	1. ~~It seems that 'Target deployment directory' it is nt a proper description. Probably it is Target exporting directory~~
	2. Flag '--checkpoint' name is not displaying the definition. TODO: better rename to --save_checkpoint