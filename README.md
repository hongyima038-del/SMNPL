# Self-paced Multi-view Clustering with Noisy Pseudo-Labels

This repository provides the implementations of Self-paced Multi-view Clustering with Noisy Pseudo-Labels, presented in the paper:


## Installation
Requires Python >= 3.7 (tested on 3.8)

To install the required packages, run:
```
pip install -r requirements.txt
```
from the root directory of the repository. Anaconda (or similar) is recommended.

## Datasets
### Included dataset
The following datasets are included as files in this project:

- `Caltech-2v`
### Generating datasets
To generate training-ready datasets, run:
```
python -m data.make_dataset <dataset_1> <dataset_2> <...> 
```
This will export the training-ready datasets to `data/processed/<datset_name>.npz`.

### Preparing a custom dataset for training
Create `<custom_dataset_name>.npz` in `data/processed/` with the following keys:
```
n_views: The number of views, V
labels: One-dimensional array of labels. Shape (n,)
view_0: Data for first view. Shape (n, ...)
  .
  .
  .
view_V: Data for view V. Shape (n, ...)
```
Alternatively, call
```Python
data.make_dataset.export_dataset(
    "<custom_dataset_name>",    # Name of the dataset
    views,                      # List of view-arrays
    labels                      # Label array
)
```
This will automatically export the dataset to an `.npz` file at the correct location.

## Experiment configuration
Experiment configs are nested configuration objects, where the top-level config is an instance of 
`config.defaults.Experiment`. 

The configuration object for the contrastive model on E-MNIST, for instance, looks like this:
```Python
CCV = Experiment(
    dataset_config=Dataset(name="ccv"),
    model_config=PGCLDCL(
        backbone_configs=(
            MLP(input_size=(5000,)),
            MLP(input_size=(5000,)),
            MLP(input_size=(4000,)),
        ),
        fusion_config=Fusion(method="weighted_mean", n_views=3),
        projector_config=None,
        cm_config=DDC(n_clusters=20),
        loss_config=Loss(
            funcs="ddc_1|ddc_2|ddc_3|Alignment|Clu|mi_fused",
            delta=20.0
        ),
        optimizer_config=Optimizer(scheduler_step_size=50, scheduler_gamma=0.1)
    ),
)
```

## Running an experiment
In the `src` directory, run:
```
python -m models.train -c <config_name> 
```
where `<config_name>` is the name of an experiment config from one of the files in `src/config/experiments/` or from 
'src/config/eamc/experiments.py' (for EAMC experiments).

### Overriding config parameters at the command-line
The configuration parameters can also be modified directly through the command line. For example, if we want to adjust the learning rate of the E-MNIST experiment from 0.001 to 0.0001, and increase the training epochs from 100 to 200, we can run:
```
python -m models.train -c Caltech-2v
```
Note the double underscores to traverse the hierarchy of the config-object.

## Evaluating an experiment
Run the evaluation script:
```Bash
python -m models.evaluate -c <config_name> \ # Name of the experiment config
                          -t <tag> \         # The unique 8-character ID assigned to the experiment when calling models.train
                          --plot             # Optional flag to plot the representations before and after fusion.
```
