# Several Recent Approaches (2018) on VQA v2

The project is based on [Cyanogenoid/vqa-counting][0]. Most of the current VQA2.0 projects are based on [https://github.com/KaihuaTang/bottom-up-attention-vqa][1], while I personally prefer the Cyanogenoid's framework, because it's very clean and clear. So I reimplement several recent approaches including [bottom-up top-down][2], [bilinear attention network][3], [Intra- and Inter-modality Attention][4] together with [learning to count][5] into this project.  


## Dependencies

- Python 3.6
  - torch > 0.4
  - torchvision 0.2
  - h5py 2.7
  - tqdm 4.19

## Prepare dataset (Follow[Cyanogenoid/vqa-counting][0])
- In the `data` directory, execute `./download.sh` to download VQA v2.
  - For experimenting, using 36 fixed proposals is faster, at the expense of a bit of accuracy. Uncomment the relevant lines in `download.sh` and change the paths in `config.py` accordingly. Don't forget to set `output_size` in there to 36 to actually get the speed-up.
- Prepare the data by running
```
python preprocess-images.py
python preprocess-vocab.py
```
This creates an `h5py` database (95 GiB) containing the object proposal features and a vocabulary for questions and answers at the locations specified in `config.py`.

## How to Train

All the models are named as XXX_model.py, and most of the parameters is under config.py. To change the model, simply change model_type in config.py. Then train your model with:
```
python train.py [optional-name]
```
- To evaluate accuracy (VQA accuracy and balanced pair accuracy) in various categories, you can run
```
python eval-acc.py <path to .pth log> [<more paths to .pth logs> ...]
```
Currently the framework is only support test on validation set, to train with the full train&val splite and get the test-std/test-val results, I will support it later. 

## Model Details

I didn't fully test all the models I reimplement, sorry about that. And I didn't implement tfidf embedding of BAN model, only Glove Embedding is provided. About Intra- and Inter-modality Attention, Although I implement all the details provided by the paper, it still seems not as good as the paper reported, I will chech what I miss when authors release their code.

To Train [Counting Model][5]

Set following parameters in config.py:
```
v_feat_norm = True
initial_lr = 1.5e-3
model_type = 'counting'
optim_method = 'Adam'
```

To Train [Bottom-up Top-down][2]
```
model_type = 'baseline' 
optim_method = 'Adamax' 
```

To Train [Bilinear attention network][3]
```
batch_size = 64
model_type = 'ban' 
optim_method = 'Adamax'
```
Note that BAN is very Memory Comsuming, so please ensure you got enough GPUs and run main.py with CUDA_VISIBLE_DEVICES=0,1,2,3

To Train [Intra- and Inter-modality Attention][4]
```
batch_size = 64
model_type = 'inter_intra' 
optim_method = 'Adamax'
```
You may need to change the learning rate decay strategy as well from gradual_warmup_steps and lr_decay_epochs in config.py 

[0]: https://github.com/Cyanogenoid/vqa-counting
[1]: https://github.com/hengyuan-hu/bottom-up-attention-vqa
[2]: https://arxiv.org/abs/1707.07998
[3]: https://arxiv.org/abs/1805.07932
[4]: https://arxiv.org/abs/1812.05252
[5]: https://arxiv.org/abs/1802.05766
