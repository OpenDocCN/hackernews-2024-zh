<!--yml
category: 未分类
date: 2024-05-27 12:58:47
-->

# Optuna - A hyperparameter optimization framework

> 来源：[https://optuna.org/](https://optuna.org/)

You can optimize LightGBM hyperparameters, such as boosting type and the number of leaves, in three steps:

1.  Wrap model training with an `objective` function and return accuracy
2.  Suggest hyperparameters using a `trial` object
3.  Create a `study` object and execute the optimization

```
import lightgbm as lgb

import optuna

# 1\. Define an objective function to be maximized.
def objective(trial):
    ...

    # 2\. Suggest values of the hyperparameters using a trial object.
    param = {
        'objective': 'binary',
        'metric': 'binary_logloss',
        'verbosity': -1,
        'boosting_type': 'gbdt',
        'lambda_l1': trial.suggest_float('lambda_l1', 1e-8, 10.0, log=True),
        'lambda_l2': trial.suggest_float('lambda_l2', 1e-8, 10.0, log=True),
        'num_leaves': trial.suggest_int('num_leaves', 2, 256),
        'feature_fraction': trial.suggest_float('feature_fraction', 0.4, 1.0),
        'bagging_fraction': trial.suggest_float('bagging_fraction', 0.4, 1.0),
        'bagging_freq': trial.suggest_int('bagging_freq', 1, 7),
        'min_child_samples': trial.suggest_int('min_child_samples', 5, 100),
    }

    gbm = lgb.train(param, dtrain)
    ...
    return accuracy

# 3\. Create a study object and optimize the objective function.
study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100)
```

[See full example on Github](https://github.com/optuna/optuna-examples/blob/main/lightgbm/lightgbm_simple.py)