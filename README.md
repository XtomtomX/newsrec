# Welcome to newsrec 👋

In this repository we work on Neural News Recommendation methods.
Currently our repository contains following models:
- [LSTUR](https://www.aclweb.org/anthology/P19-1033/)
- [NAML](https://arxiv.org/abs/1907.05576)
- [NRMS](https://www.aclweb.org/anthology/D19-1671/)
- [SentiRec](https://www.aclweb.org/anthology/2020.aacl-main.6.pdf)
- RobustSentiRec

### Table of Contents
- **[Models](#models)**
    - **[LSTUR](#LSTUR%20%28An%20et%20al.%2C%202019%29)**
    - **[NAML](#NAML%20%28WU%20et%20al.%2C%202019%29)**
    - **[NRMS](#NRMS%20%28Wu%20et%20al.%2C%202019%29)**
    - **[SentiRec](#SentiRec%20%28Wu%20et%20al.%2C%202020%29)**
    - **[RobustSentiRec](#robustsentirec)**
- **[How to Train and Test?](#How%20to%20Train%20and%20Test%3F)**
    - **[Requirements](#requirements)**
    - **[Data](#data)**
    - **[Train-Run](#train-run)**
    - **[Test-Run](#test-run)**
    - **[Monitoring](#monitoring)**

# Models
The general architecture of the models looks as following:

![](figures/genreal_recommendation_framework.png)

- Items are encoded based on their content.
- Users are encoded based on their pervious interactions with items.
- For capturing personalized attention or long-short term interests a user-embedding might be used in the user- or item-encoder.
- Candidate items are scored considering candidate items encoding and the target user's encoding

## LSTUR (An et al., 2019)
![](figures/lstur_framework.png)
> Figure taken from the LSTUR paper.

LSTUR aptures short-term interest of users by applying GRU on recently clicked items and long-term interest by considering a user’s whole history track. 

Please referr to [the paper](https://www.aclweb.org/anthology/P19-1033/) for more details. 

## NAML (WU et al., 2019)
![](figures/naml_framework.png)
> Figure taken from the NAML paper.

NAML uses attention networks to incorporate different views of a news article (e.g., title, abstract, category, etc.) into the news representaiton.

Please referr to [the paper](https://arxiv.org/abs/1907.05576) for more details. 

## NRMS (Wu et al., 2019)
![](figures/nrms_framework.png)
> Figure taken from the NRMS paper.

NRMS uses multi-head self-attention in combination with additive-attention to model news articles and in turn users. 

Please referr to [the paper](https://www.aclweb.org/anthology/D19-1671/) for more details. 

## SentiRec (Wu et al., 2020)
![](figures/sentirec_framework.png)

SentiRec builds upon the NRMS mode and learns through an auxiliary sentiment-prediction task in the news encoding sentiment-aware news representations and in turn penalizes recommendations, which have a similar sentiment orientation as the user’s history track.

Please referr to [the paper](https://www.aclweb.org/anthology/2020.aacl-main.6.pdf) for more details. 

## RobustSentiRec
![](figures/robust_sentirec_framework.png)

We adapt SentiRec by omitting the auxiliary sentiment-prediction task. Instead, we directly incorporate the sentiment-polarity scores determined by the sentiment-analyzer into the news representation by concatenating it with the output of the news encoder and applying a linear transformation.


# How to Train and Test?
## Requirements
In our implementations we use the combo pytorch - pytorchlighnting - torchmetrics. Thus, a cuda device is suggested and more are even better! We are using conda and have exported the conda-environment we are working with, thus: 
1. Install [conda](https://docs.conda.io/en/latest/)
2. Create an environment with the provided environment.yml:

    ```
    conda env -n _env_name_of_your_choice_ -f environment.yml
    ```
3. Activate the environment as following: 
    ```
    conda activate _env_name_of_your_choice_
    ```
## Data
In our experiments we use the Microsoft News Dataset, i.e., MIND. In particular we have used MINDsmall_train in a 9:1 split for trainig and validation; and the MINDsmall_dev as our holdout. The datasets and more detail are provided [here](https://msnews.github.io/index.html). Furthermore, we use 300-dimensional Glove embeddings, which can be downloaded [here](http://nlp.stanford.edu/data/glove.840B.300d.zip). 

In order to train our models, you need to pre-preprocess the data. Therefore, we provide ``./project/data/data_preprocess.py`` and an example config ``./project/config/data/mind_recsys2021.yaml`` (as we use it). 

Please, change the config according to your needs. You may want to change source paths (i.e., paths training data, test data, and wordembedding) and target paths (i.e, dir where the pre-processed data should be persisted). 

Run the prepreocessing as following:
```
./project/data/data_preprocess.py --config ./project/config/data/mind_recsys2021.yaml
```

## Config
We provide for each model example configs under ``./project/config/model/_model_name_``. The config files contain general information, hyperparameter, meta-information (logging, checkpointing, ...). All hyperparemters are either trained on the validation set or referr to the corresponding papers. The generall structure of the config is as follwing: 

- GENERAL: General information about the model like the name of the model to be trained (e.g., ``name: "sentirec"``). 
- DATA: Paths to (pre-processed) train and test data files and information about the underlying data (might be changed according to the pre-processing); Features to be used in the model, e.g. title, vader_sentiment, distillbert_sst2_sentiment, etc. 
- MODEL: Hyperparams of the model and model-architecture, e.g., learning_rate, dropout_probability, num_attention_heads, sentiment_classifier, etc. 
- TRAIN: Config for the Checkpointing, Logging, Earlystop; Dataloader information (e.g., batch_size, train_val_split ,etc.); and Config  for the trainer (e.g., max_epochs, fast_dev_run, etc.)

Please, change the config according to your needs. You may want to change the paths to: i) train/test - data; ii) directory for the checkpoints; iii) directory for the logging.

## Train-Run
You can train now the models using ``./project/train.py``. Here an example for running SentiRec: 
```
CUDA_VISIBLE_DEVICES=0,1 ./project/train.py --config ./project/config/model/sentirec/vader_lambda0p3_mu10.yaml
```
The name parameter in the config defines which model to train, here it is set to "sentirec". With ``CUDA_VISIBLE_DEVICES=ids`` you can define on which CUDA devices to train. If you do not define any device and run just ``./project/train.py`` all visible devices will be used. 

Currently ``train.py`` saves the last checkpoint and the one of best three epochs (can be configured) based on validation-AUC. In case of an interruption it attempts a gracefull shutdown and saves the checkpoint aswell. You can resume training by providing a path to a checkpoint with  ``--resume`` like: 
```
CUDA_VISIBLE_DEVICES=0,1 ./project/train.py --config ./project/config/model/sentirec/vader_lambda0p3_mu10.yaml --resume _path_to_checkpoint_
```

## Test-Run
For Testing use the same config as you have trained, since it contains all necessary information, e.g., path to test data, config of test-dataloader, logging info, etc. It may take longer, but use only ONE GPU for testing (e.g., ``CUDA_VISIBLE_DEVICES=_one_gpu_id_of_your_choice_``). Only test once, right before you report/submit (after validation, tuning, ...). You have to provide a checkpoint of your choice (e.g., best performing hyperparam setting according to val, etc.) and can run a test as following:
```
./project/test.py --config project/config/model/sentirec/vader_lambda0p3_mu10.yaml --ckpt path_to_ckpt_of_your_choice
```

## Monitoring
We use the default logger of pytorchlightning, i.e., tensorboard. You can monitor your traning, validation, and testing via tensorboard. You have to provide the path of your logs and can start tensorboard as following: 
```
tensorboard --logdir _path_to_tensorboard_logdir_
```


# Acknowledgments
We build upon [@yusanshi's news-recommendation repo](https://github.com/yusanshi/NewsRecommendation) and [Microsoft's recommenders repo](https://github.com/microsoft/recommenders). 
