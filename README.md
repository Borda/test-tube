# Test tube: Easily log, track and optimize Deep Learning experiments

<div style="text-align: center">
<img src="https://raw.githubusercontent.com/williamfalcon/test_tube/master/imgs/test_tube_logo.png">
</div>
<br>

[![PyPI version](https://badge.fury.io/py/test_tube.svg)](https://badge.fury.io/py/test_tube)    [![Documentation Status](https://readthedocs.org/projects/test-tube/badge/?version=latest)](https://williamfalcon.github.io/test_tube/)
     [![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/williamFalcon/test_tube/blob/master/LICENSE)    
[DOCS](https://williamfalcon.github.io/test_tube/)

---
Test tube is a python library to track and optimize Deep Learning experiments. It's framework agnostic and is built on top of the python argparse API for ease of use.

Test tube stores logs in csv files on your machine for easy analysis.

```bash
pip install test_tube
```

---
Use Test Tube if you need to:

- Track multiple [Experiments](experiment_tracking/experiment/) across models.
- [Optimize your hyperparameters](hyperparameter_optimization/HyperOptArgumentParser/) using grid_search or random_search.
- Visualize experiments without uploading anywhere, logs store as csv files.
- Automatically track ALL parameters for a particular training run.
- Automatically snapshot your code for an experiment using git tags.    
- Save progress images inline with training metrics.    

Compatible with:   
- Python 2, 3
- Tensorflow   
- Keras  
- Pytorch   
- Caffe, Caffe2   
- Chainer   
- MXNet   
- Theano   
- Any python based ML or DL library   
- Runs seamlessly on CPU and GPU.   

## Example use:

### Log experiments

```python
from test_tube import Experiment

exp = Experiment(name='dense_model', save_dir='../some/dir/')

exp.add_meta_tags({'learning_rate': 0.002, 'nb_layers': 2})

training_steps = 10
for step in range(1, training_steps):
    tng_err = 1.0 / step # model.eval(tng_x, tng_y)

    exp.add_metric_row({'tng_err': tng_err})

# training complete!
# load the resulting metrics.csv into your favorite viz tool   
import pandas as pd    
import matplotlib   
df = pd.read_csv('../some/dir/test_tube_data/dense_model/version_0/metrics.csv')
df.tng_err.plot()
```

### Optimize hyperparameters
```python
from test_tube import HyperOptArgumentParser

# subclass of argparse
parser = HyperOptArgumentParser(strategy='random_search')
parser.add_argument('--learning_rate', default=0.002, type=float, help='the learning rate')

# let's enable optimizing over the number of layers in the network
parser.add_opt_argument_list('--nb_layers', default=2, type=int, tunnable=True, options=[2, 4, 8])

# and tune the number of units in each layer
parser.add_opt_argument_range('--neurons', default=50, type=int, tunnable=True, start=100, end=800, nb_samples=10)

# compile (because it's argparse underneath)
hparams = parser.parse_args()

# run 20 trials of random search over the hyperparams
for hparam_trial in hparams.trials(20):
    train_network(hparam_trial)
```    

### Log images inline with metrics    
```python
# name must have either jpg, png or jpeg in it
img = np.imread('a.jpg')
exp.add_metric_row('test_jpg': img, 'val_err': 0.2)

# saves image to ../exp/version/media/test_0.jpg  
# csv has file path to that image in that cell   
```

## Visualize and access your data

[Example test_tube output](https://github.com/williamFalcon/test_tube/tree/master/examples/saved_logs/example_test_tube_data)    
Test tube saves experiments directly to csv files along with relevant metadata. An experiment is a folder structure like:
```
test_tube_data
    /my_exp_name_A
        /version_0
        /version_1
            meta.experiment
            meta_tags.json
            metrics.csv
            media/
                test_img_0.jpg
                test_img_1.jpg
         /version_2

    /my_exp_name_B
        ...
```

#### Media files

Any saved images are saved to `/media`:

**example**
```python
img = np.imread('a.jpg')
exp.add_metric_row('test_jpg': img, 'val_err': 0.2)

# saves image to /media/test_0.jpg
```


#### Metrics file
metrics.csv is a standard csv with rows for each `add_metric_row` call and columns for each key across all rows

**example**
```python
exp.add_metric_row('val_err': 0.2, 'epoch': 1)
exp.add_metric_row('test_err': 0.2, 'epoch': 1)
```

`metrics.csv`
*created_at is added automatically*

|  val_err |  epoch | test_err | created_at |
|---|---|---| --- |
| 0.2  |  1 | - | 2017-10-13 01:34:14 |
| -  | 1  | 0.2| 2017-10-13 01:34:18 |


#### Meta tags file
`meta_tags.json` contains a json file with the information for your experiment

**example**
```python
exp.add_meta_tags({'learning_rate': 0.002, 'nb_layers': 2})
```

`meta_tags.json`
```json
{
    "learning_rate": 0.002,
    "nb_layers": 2
}
```

## Demos
- [Experiments and hyperparameter optimization for tensorflow across 4 GPUs simultaneously](https://github.com/williamFalcon/test_tube/blob/master/examples/tensorflow_example.py)
