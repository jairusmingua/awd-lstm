# AWD-LSTM and ULMFiT Toolkit
**Note: This is a work in progress!**

This repository contains partial (so far) reproductions of the following papers:
* [Regularizing and Optimizing LSTM Language Models](https://arxiv.org/abs/1708.02182) (Merity et al., 2017).
* [Universal Language Model Finetuning for Text Classification](https://arxiv.org/abs/1801.06146) (Howard & Ruder, 2018) 

Code in this repository allows you to:
1. Train AWD-LSTM language models;
2. Finetune language models on other datasets; and
3. Finetune language models for text classification (ULMFiT)

This repository is a work in progress and so not all techniques have been added. Please see the **To-do** section below to see what has not been added yet.

# Requirements
Libraries you need:
* PyTorch - At least v.1.0.0
* Transformers - The learning rate scheduling comes from there
* spacy - We use spacy tokenizers for ULMFiT
* numpy and pandas - For random numbers and data manipulation
* tqdm - Progress bars to keep our sanity intact :)

Hardware you need:
* A GPU with at least 11 GB - All the results in this repository have been produced on machines with NVIDIA GTX 1080Ti and servers with NVIDIA Tesla P100 GPUs. A lot of the models when training do take about 10 GB of memory. You *could* get away with a smaller and slower GPU but do note that this affects your speed and performance.

# Using the Training Scripts for Language Modeling
The scripts can be used for language modeling, following the ideas proposed in Merity et al. (2017). Here are a few examples on WikiText-2:

Train an AWD-LSTM using SGD with variable BPTT lengths (Valid PPL: 77.8227 / Test PPL: 74.2074)
```
python awd_lstm/main.py --path=data/wikitext-2 --train=wiki.train.tokens --valid=wiki.valid.tokens --test=wiki.test.tokens --output=pretrained_wt2 --bs=80 --bptt=80 --epochs=150 --use_var_bptt --tie_weights --save_vocab --vocab_file=vocab.pth --gpu=0
```

Train an AWD-LSTM using Adam+LinearWarmup with variable BPTT lengths (Valid PPL: 85.3083 / Test PPL: 80.9872)
```
python awd_lstm/main.py --path=data/wikitext-2 --train=wiki.train.tokens --valid=wiki.valid.tokens --test=wiki.test.tokens --output=pretrained_wt2 --bs=80 --bptt=80 --epochs=150 --use_var_bptt --tie_weights --optimizer=adam --lr=3e-4 --warmup_pct=0.1 --save_vocab --vocab_file=vocab.pth --gpu=0
```

Alternatively, you can test language modeling by training a simple LSTM baseline. Here is an example:

Train a basic LSTM using SGD without variable BPTT lengths (Valid PPL: 101.1345 / Test PPL: 95.7615)
```
python awd_lstm/main.py --path=data/wikitext-2 --train=wiki.train.tokens --valid=wiki.valid.tokens --test=wiki.test.tokens --output=basic_wt2 --bs=80 --bptt=80 --epochs=100 --tie_weights --encoder=lstm --decoder=linear --lr=20 --save_vocab --vocab_file=vocab.pth --gpu=0

```

This is also how language model pretraining works. Be sure to use the ```--save_vocab``` argument to save your vocabularies.

# Generation
You can use your pretrained models to generate text. Here is an example:
```
python awd_lstm/generate.py --path data/wikitext-2 --tie_weights --vocab_file=vocab.pth --pretrained_file=pretrained_wt103.pth --temp=0.7 --nwords=100
```

# Finetuning Language Models
To finetune on a dataset, you'll need the saved vocabulary file and the pretrained weights. For text datasets, you will need to preprocess them such that each sample is separated by a blank line (the code replaces this with an ```<eos>``` token.) Here's an example finetuning the iMDB dataset on a pretrained model trained using WikiText-103
```
python awd_lstm/main.py --path=data/imdb --train=train.txt --valid=valid.txt --test=test.txt --output=imdb_finetuned --bs=60 --bptt=60 --epochs=10 --use_var_bptt --tie_weights --load_vocab --vocab_file=vocab.pth --use_pretrained --pretrained_file=pretrained_wt103.pth --gpu=0
```

# ULMFiT / Finetuning Classifiers
To finetune a classifier, make sure you have a finetuned language model at hand. Load the ```ULMFiT.ipynb``` notebook and follow the instructions there.

# Changelog
Version 0.3
* Added ULMFiT and related code
* Added finetuning and pretraining capabilities
* Added a way to load vocabularies
* Fixed the generation script

Version 0.2
* Added basic LSTM Encoder support and modularity
* Added an option to train with Adam with Linear Warmups
* Fixed parameters and reduced required parameters
* AR/TAR only activates when using an AWD-LSTM

Version 0.1
* Added basic training functionality

# To-do and Current Progress
As said, this repository is a work in progress. There will be some features missing. Here are the missing features:

For AWD-LSTM training:
* NT-ASGD not implemented

For ULMFiT:
* Discriminative learning rates missing
* STLR (although I personally added Linear Warmup Scheduling for ULMFiT)
* Learning rate sweeping like in FastAI

Miscellany
* Distributed training

For now, ULMFiT achieves a validation accuracy of 91.93%, which is 3.47 points below the paper's original result of 95.4%. I surmise that this score will get closer once I add in all the missing features. For now, the repo is a partial reproduction.

The pretrained WikiText-103 model used in the results was also adapted from FastAI. I will update with newer scores once I add in distributed pretraining to train my own language model.

# Credits
Credits are due to the following:
* The contributors at FastAI where I adapted the dropout code from.
* The people at HuggingFace responsible for their amazing Transformers library.

# Issues and Contributing
If you find an issue, please do let me know in the issues tab! Help and suggestions are also very much welcome.
