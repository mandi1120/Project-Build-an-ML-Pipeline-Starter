# WGU Udacity Data Analyst Nanodegree <br>Machine Learning DevOps <br>Build an ML Pipeline for Short-Term Rental Prices in NYC  
### By: Amanda Hanway, 1/21/2024 

## Introduction 
The project was created as part of the Udacity Machine Learning DevOps course. 
The starter repository included some pre-implemented re-usable components in order to simulate
a real-world situation and to focus on the ML DevOps information covered in the course.

## Project Description
You are working for a property management company renting rooms and properties for short periods of 
time on various rental platforms. You need to estimate the typical price for a given property based 
on the price of similar properties. Your company receives new data in bulk every week. The model needs 
to be retrained with the same cadence, necessitating an end-to-end pipeline that can be reused.

## Table of contents
- [Preliminary steps](#preliminary-steps)
  * [Environment Setup](#environment-setup)
  * [The configuration](#the-configuration)
  * [Running the entire pipeline or just a selection of steps](#Running-the-entire-pipeline-or-just-a-selection-of-steps)
  * [Pre-existing components](#pre-existing-components)

## Preliminary steps
Fork the Started kit from: 
[https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter](https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter)

Clone the repository:
```
git clone https://github.com/[your github username]/Project-Build-an-ML-Pipeline-Starter.git
```

and go into the repository:
```
cd Project-Build-an-ML-Pipeline-Starter
```

Set up a Weights and Biases account: https://wandb.ai/

## Environment Setup
The first hurdle was getting the code to run on my personal computer. The following steps 
allowed me to set up the environment on a Windows 11 OS. 

- Install WSL: https://docs.microsoft.com/en-us/windows/wsl/install-win10
- Navigate to the repository folder
- Shift + right click on folder name
- Click "open linux shell here"
 
- Install anaconda:    
     - Reference: https://www.reddit.com/r/wsl2/comments/120f2uy/why_cant_we_use_conda_installed_in_windows_on_wsl2/
     - Run: wget https://repo.anaconda.com/archive/Anaconda3-2023.03-Linux-x86_64.sh
     - After download run: bash Anaconda3-2023.03-Linux-x86_64.sh
     - Close and reopen terminal after install

- Create udacity environment:
     - Run: conda create --name udacity python=3.8 mlflow jupyter pandas matplotlib requests -c conda-forge
     - Run: conda activate udacity (or source activate udacity)
     - See package versions run: conda list
     - If need to start over run: conda env remove -n udacity

- Install, log in and test weights and biases
     - Run: pip install wandb
     - Run: wandb login
     - Follow instructions to get and paste API key from https://wandb.ai/authorize
     - Run:
          - source activate udacity
          - echo "wandb test" > wandb_test
          - wandb artifact put -n testing/artifact_test wandb_test

- Test Mlflow
     - Run: mlflow --help

- Set up project environment
     - Run: conda env create -f environment.yml
     - Run: conda activate nyc_airbnb_dev

- Fix issue of being stuck on "Solving environment:" status
     - Reference: https://knowledge.udacity.com/questions/904948
          - specify conda-forge:: explicitly in front of all packages in environment.yml file
     - Reference: https://knowledge.udacity.com/questions/1009530
          - Update Conda by running: conda update -n base -c defaults conda
          - Clear the cache and recreate it by running: conda clean --all

- Fix error "Solving environment: / Killed"
     - References:
     - https://stackoverflow.com/questions/74566580/why-is-conda-create-env-killed
     - https://askubuntu.com/questions/178712/how-to-increase-swap-space
```
# Resize Swap to 8GB
# Turn swap off
# This moves stuff in swap to the main memory and might take several minutes
sudo swapoff -a

# Create an empty swapfile
# Note that "1G" is basically just the unit and count is an integer.
# Together, they define the size. In this case 20GB.
sudo dd if=/dev/zero of=/swapfile bs=1G count=20

# Set the correct permissions
sudo chmod 0600 /swapfile
sudo mkswap /swapfile  # Set up a Linux swap area
sudo swapon /swapfile  # Turn the swap on

# Check if it worked
grep Swap /proc/meminfo

# Make it permanent (persist on restarts)
# Add this line to the end of your /etc/fstab:
/swapfile swap swap sw 0 0
```

### The configuration
The parameters controlling the pipeline are defined in the ``config.yaml`` file defined in
the root of the starter kit. We will use Hydra to manage this configuration file. 
This file is only read by the ``main.py`` script (i.e., the pipeline) and its content is
available with the ``go`` function in ``main.py`` as the ``config`` dictionary. For example,
the name of the project is contained in the ``project_name`` key under the ``main`` section in
the configuration file. It can be accessed from the ``go`` function as 
``config["main"]["project_name"]``.

### Running the entire pipeline or just a selection of steps
In order to run the pipeline when you are developing, you need to be in the root of the starter kit, 
then you can execute as usual:

```bash
>  mlflow run .
```
This will run the entire pipeline.

When developing it is useful to be able to run one step at the time. Say you want to run only
the ``download`` step. The `main.py` is written so that the steps are defined at the top of the file, in the 
``_steps`` list, and can be selected by using the `steps` parameter on the command line:

```bash
> mlflow run . -P steps=download
```
If you want to run the ``download`` and the ``basic_cleaning`` steps, you can similarly do:
```bash
> mlflow run . -P steps=download,basic_cleaning
```
You can override any other parameter in the configuration file using the Hydra syntax, by
providing it as a ``hydra_options`` parameter. For example, say that we want to set the parameter
modeling -> random_forest -> n_estimators to 10 and etl->min_price to 50:

```bash
> mlflow run . \
  -P steps=download,basic_cleaning \
  -P hydra_options="modeling.random_forest.n_estimators=10 etl.min_price=50"
```

### Pre-existing components
In order to simulate a real-world situation, we are providing you with some pre-implemented
re-usable components. While you have a copy in your fork, you will be using them from the original
repository by accessing them through their GitHub link, like:

```python
_ = mlflow.run(
                f"{config['main']['components_repository']}/get_data",
                "main",
                parameters={
                    "sample": config["etl"]["sample"],
                    "artifact_name": "sample.csv",
                    "artifact_type": "raw_data",
                    "artifact_description": "Raw file as downloaded"
                },
            )
```
where `config['main']['components_repository']` is set to 
[https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter/tree/main/components](https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter/tree/main/components).
You can see the parameters that they require by looking into their `MLproject` file:

- `get_data`: downloads the data. [MLproject](https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter/blob/main/components/get_data/MLproject)
- `train_val_test_split`: segrgate the data (splits the data) [MLproject](https://github.com/udacity/Project-Build-an-ML-Pipeline-Starter/blob/main/components/train_val_test_split/MLproject)




## License

[License](LICENSE.txt)
