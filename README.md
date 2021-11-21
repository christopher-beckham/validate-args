

This is a a rather bare-bones implementation I have of dealing with arguments. There are libraries that exist for this kind of thing, but I wanted something that
specifically worked easily with the current workflow I'm using at Element AI. Define this method somewhere:

```
def insert_defaults(exp_dict, defaults):
    """Inserts default values into the exp_dict.

    Will raise an exception if exp_dict contains
    a key that is not recognised.

    Args:
        exp_dict (dict): dictionary to be added to
    """
    for key in exp_dict.keys():
        if key not in defaults:
            # Check if there are any unknown keys.
            raise Exception("Found key in exp_dict but is not recognised: {}".\
                format(key))
        else:
            if type(defaults[key]) == dict:
                # If this key maps to a dict, then apply
                # this function recursively
                insert_defaults(exp_dict[key], defaults[key])

    # insert defaults
    for k, v in defaults.items():
        if k not in exp_dict:
            exp_dict[k] = v
```

In whatever task launcher, when you have constructed the experiment json, invoke it like so, and also invoke an args validation method:

```
def main(exp_dict):
  insert_defaults(exp_dict, DEFAULTS)
  validate_args(exp_dict)
```

Example default args dict, note how some arguments are also (nested) dictionaries.

```
DEFAULTS = {
    'use_train_set': False,
    'mode': 'augment', # baseline, reconstruct, augment, or sample
    'epochs': 1000,
    'freeze_all_except': [], # TODO: deprecate??
    'from_scratch': False, # finetune resnet from scratch, i.e. unfreeze all layers
    'pretrained': None, # 2-tuple in format (clf_exp, ae_exp)
    'n_samples_per_class': 0, # number of augmented samples per class
    'dataset': None,
    'finetune': None, # kwargs for transform_clf
    'batch_size': 32, # batch size for training
    'aug_batch_size': 32, # batch size for generating new images
    'use_mixup': False, # enable mixup mode for classifier
    'mixup': {
        'dist': 'uniform', # do we use beta or uniform
        'alpha': 1.0, # U(0,alpha) for uniform, beta(alpha, alpha) for beta
        'mix_labels': True # (input mixup only) mix labels as well?
    },
    'temperature': None, # only applicable to mode==sample
    'ignore_dev_set': False, # if mode==augment, do not use original supports
    'wrong_label_p': 0.0,
    'optim': {
        'lr': 2e-4, 
        'beta1': 0.9, 
        'beta2': 0.999, 
        'weight_decay': 0.
    },
    'save_every': 1, # save model/metrics every this many epochs
}
```

Example validate_args, which makes heavy use of asserts:

```
def validate_args(dd):
    # NOTE: only validates top-level arguments
    assert type(dd['use_train_set']) is bool
    assert type(dd['mode']) is str and dd['mode'] in \
        ['baseline', 'reconstruct', 'augment', 'augment-v2', 'sample']
    assert type(dd['epochs']) is int
    assert type(dd['freeze_all_except']) is list
    assert type(dd['from_scratch']) is bool
    assert type(dd['pretrained']) in [tuple, list]
    assert type(dd['n_samples_per_class']) is int
    assert type(dd['dataset']) is dict
    assert type(dd['finetune']) is dict
    assert type(dd['batch_size']) is int
    assert type(dd['aug_batch_size']) is int
    assert type(dd['use_mixup']) is bool
    assert type(dd['mixup']) is dict
    assert dd['temperature'] is None or type(dd['temperature']) is float
    assert type(dd['ignore_dev_set']) is bool
    assert type(dd['wrong_label_p']) is float
    assert type(dd['optim']) is dict
    assert type(dd['save_every']) is int
```
