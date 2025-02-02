# FBK-fairseq-ST

This repository is a fork of
https://github.com/pytorch/fairseq containing
additional code used for our papers.
Most of our code is in the `examples/speech_recognition` folder.

If you use this code, please consider citing the related paper(s).
The repository contains the code for:

<ul>
    <li>M. Gaido et al., "Is "moby dick" a Whale or a Bird? Named Entities and Terminology in Speech Translation", EMNLP 2021</li>
    <li>A. Karakanta et al., "Between Flexibility and Consistency: Joint Generation of Captions and Subtitles", IWSLT 2021</li>
    <li>M. Gaido, B. Savoldi et al., "How to Split: the Effect of Word Segmentation on Gender Bias in Speech Translation", Findings of ACL 2021</li>
    <li>M. Gaido et al., "CTC-based Compression for Direct Speech Translation", EACL 2021</li>
    <li>M. Gaido, B. Savoldi et al., "Breeding Gender-aware Direct Speech Translation Systems", COLING 2020</li>
    <li>M. Gaido et al., "Contextualized Translation of Automatically Segmented Speech", INTERSPEECH 2020</li>
    <li>M. Gaido et al., "On Knowledge Distillation for Direct Speech Translation", CliC-IT 2020</li>
</ul>

```bibtex
@inproceedings{gaido-etal-2021-is-moby-dick,
    title = {{Is ``moby dick'' a Whale or a Bird? Named Entities and Terminology in Speech Translation}},
    author = "Gaido, Marco and
      Rodríguez, Susana and
      Negri, Matteo  and
      Bentivogli, Luisa and
      Turchi, Marco",
    booktitle = "Proceedings of the 2021 Conference on Empirical Methods in Natural Language Processing (EMNLP)",
    month = nov,
    year = "2021",
    address = "Punta Cana, Dominican Republic",
    publisher = "Association for Computational Linguistics"
}
```
```bibtex
@inproceedings{karakanta-etal-2021-between,
    title = "Between Flexibility and Consistency: Joint Generation of Captions and Subtitles",
    author = "Karakanta, Alina  and
      Gaido, Marco and
      Negri, Matteo  and
      Turchi, Marco",
    booktitle = "Proceedings of the 18th International Conference on Spoken Language Translation",
    month = aug,
    year = "2021",
    address = "Online",
    publisher = "Association for Computational Linguistics"
}
```
```bibtex
@inproceedings{gaido-etal-2021-how-to-split,
    title = "How to Split: the Effect of Word Segmentation on Gender Bias in Speech Translation",
    author = "Gaido, Marco  and
      Savoldi, Beatrice  and
      Bentivogli, Luisa  and
      Negri, Matteo  and
      Turchi, Marco",
    booktitle = "Proceedings of the 59th Annual Meeting of the Association for Computational Linguistics",
    month = aug,
    year = "2021",
    address = "Online",
    publisher = "Association for Computational Linguistics"
}
```
```bibtex
@inproceedings{gaido-etal-2021-ctc,
    title = "{CTC}-based Compression for Direct Speech Translation",
    author = "Gaido, Marco  and
      Cettolo, Mauro  and
      Negri, Matteo  and
      Turchi, Marco",
    booktitle = "Proceedings of the 16th Conference of the European Chapter of the Association for Computational Linguistics: Main Volume",
    month = apr,
    year = "2021",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2021.eacl-main.57",
    pages = "690--696",
    abstract = "Previous studies demonstrated that a dynamic phone-informed compression of the input audio is beneficial for speech translation (ST). However, they required a dedicated model for phone recognition and did not test this solution for direct ST, in which a single model translates the input audio into the target language without intermediate representations. In this work, we propose the first method able to perform a dynamic compression of the input in direct ST models. In particular, we exploit the Connectionist Temporal Classification (CTC) to compress the input sequence according to its phonetic characteristics. Our experiments demonstrate that our solution brings a 1.3-1.5 BLEU improvement over a strong baseline on two language pairs (English-Italian and English-German), contextually reducing the memory footprint by more than 10{\%}.",
}
```
```bibtex
@inproceedings{gaido-etal-2020-breeding,
    title = "Breeding Gender-aware Direct Speech Translation Systems",
    author = "Gaido, Marco  and
      Savoldi, Beatrice  and
      Bentivogli, Luisa  and
      Negri, Matteo  and
      Turchi, Marco",
    booktitle = "Proceedings of the 28th International Conference on Computational Linguistics",
    month = dec,
    year = "2020",
    address = "Barcelona, Spain (Online)",
    publisher = "International Committee on Computational Linguistics",
    url = "https://www.aclweb.org/anthology/2020.coling-main.350",
    pages = "3951--3964",
}
```
```bibtex
@inproceedings{Gaido2020,
  author={Gaido, Marco and Di Gangi, Mattia A. and Negri, Matteo and Cettolo, Mauro and Turchi, Marco},
  title={{Contextualized Translation of Automatically Segmented Speech}},
  year=2020,
  month=oct,
  booktitle={Proc. of Interspeech 2020},
  pages={1471--1475},
  doi={10.21437/Interspeech.2020-2860},
  url={http://dx.doi.org/10.21437/Interspeech.2020-2860}
}
```

## CTC Compression

The script used to run the experiments for the EACL 2021 paper is the following:

```bash
save_dir=$1
lang=$2
source_t=$3
compress_layer=$4

python train.py /datadrive/data/corpora/ctccompress/en-$lang/ \
    -s $source_t -t $lang --skip-normalization --user-dir examples/speech_recognition \
    --clip-norm 20 \
    --ddp-backend=no_c10d \
    --max-sentences 8 \
    --max-tokens 12000 \
    --max-source-positions 2000 --max-target-positions 1000 \
    --save-dir $save_dir \
    --max-epoch 150 \
    --min-lr 1e-09 \
    --dropout 0.2 \
    --lr 5e-3 --min-lr 1e-07 --reset-optimizer \
    --lr-scheduler inverse_sqrt \
    --warmup-updates 4000 --warmup-init-lr 3e-4 \
    --update-freq 8 \
    --optimizer adam --adam-betas '(0.9, 0.98)' \
    --distance-penalty log \
    --no-attn-2d --encoder-layers 11 --decoder-layers 4 \
    --ctc-compress-out --ctc-encoder-layer $compress_layer \
    --arch conv_transformer_big2 --task speech_translation_with_transcription \
    --input-feat-per-channel 40 \
    --skip-invalid-size-inputs-valid-test \
    --sentence-avg \
    --specaugment --frequency-masking-pars 13 --time-masking-pars 20 --specaugment-rate 0.5 --frequency-masking-num 2 --time-masking-num 2 \
    --criterion ctc_multi_loss --underlying-criterion label_smoothed_cross_entropy --label-smoothing 0.1
```

## Preprocessing steps

### 1. Generate YAML / other files

You need to have a single file with all the transcripts and all the translations, one per line. In addition, you need to create a YAML file that contains for each line the corresponding audio segment. Eg. your `source1.txt` should be the first line of your `train.src` file, `target1.txt` the first line of your `train.tgt` and in `train.yaml` the first line should be:
```
- {duration: HOW_LONG_YOUR_FILE_IS_OR_A_VERY_LARGE_NUMBER, offset: 0.0, speaker_id: ID_OF_THE_SPEAKER_OR_NAME_OF_THE_FILE, wav: source1.wav}
```

### 2. Preprocessing text

Download [moses](https://github.com/moses-smt/mosesdecoder) and preprocess both transcripts and translations with:

```
${mosesdir}normalize-punctuation.perl -l $lang < $YOUR_INPUT_FILE | ${mosesdir}tokenizer.perl -l $lang | ${mosesdir}deescape-special-chars.perl > $YOUR_INPUT_FILE.tok
```

Then, learn BPE or other subword segmentation using SentencePience or any other tool you like. After this step, create the fairseq datasets containing the textual information with

```
python FBK-fairseq-ST/preprocess.py --trainpref $YOUR_TRAIN.bpe --validpref $YOUR_DEV.bpe --testpref $YOUR_TEST.bpe --destdir $THE_DIR_WHERE_YOU_WANT_YOUR_FAIRSEQ_DATASETS -s $src_lang -t $tgt_lang --workers 1 --dataset-impl cached
```

Finally, create symbolic links to have a name that does not contain the `src_lang-tgt_lang` pattern with:

```
for f in $THE_DIR_WHERE_YOU_WANT_YOUR_FAIRSEQ_DATASETS*.$src_lang-$tgt_lang.*; do ln -s $f $(echo $f | sed 's/\.$src_lang-'$tgt_lang'\./\./g'); done
```

### 3. Preprocess audio

First, extract with [XNMT](https://github.com/neulab/xnmt) the Mel filterbank features. You need to create a yaml config file like this:

```
extract-test-data: !Experiment
  preproc: !PreprocRunner
    overwrite: False
    tasks:
    - !PreprocExtract
      in_files:
      - $THE_YAML_YOU_GENERATED_IN_1
      out_files:
      - $YOUR_H5_OUTPUT_FILE.h5
      specs: !MelFiltExtractor {}
```
and then you can use it with this command:

```
python xnmt/xnmt/xnmt_run_experiments.py config.yaml
```

At the end of this process, you need to preprocess the h5 dataset into a fairseq dataset, which is done with

```
python FBK-fairseq-ST/examples/speech_recognition/preprocess_audio.py --destdir $THE_DIR_WHERE_YOU_WANT_YOUR_FAIRSEQ_DATASETS --format h5 --trainpref $YOUR_H5_TRAIN_OUTPUT_FILE --validpref $YOUR_H5_DEV_OUTPUT_FILE--testpref $YOUR_H5_TEST_OUTPUT_FILE
```

After this, in your target folder you should have files like these:

```
train.src_lang.idx
train.src_lang.bin
train.tgt_lang.idx
train.tgt_lang.bin
train.npz.idx
train.npz.bin
...
```

And you are done with your preprocessing!



Below, there is the original README file.


<p align="center">
  <img src="fairseq_logo.png" width="150">
  <br />
  <br />
  <a href="https://github.com/pytorch/fairseq/blob/master/LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue.svg" /></a>
  <a href="https://github.com/pytorch/fairseq/releases"><img alt="Latest Release" src="https://img.shields.io/github/release/pytorch/fairseq.svg" /></a>
  <a href="https://github.com/pytorch/fairseq/actions?query=workflow:build"><img alt="Build Status" src="https://github.com/pytorch/fairseq/workflows/build/badge.svg" /></a>
  <a href="https://fairseq.readthedocs.io/en/latest/?badge=latest"><img alt="Documentation Status" src="https://readthedocs.org/projects/fairseq/badge/?version=latest" /></a>
</p>

--------------------------------------------------------------------------------

Fairseq(-py) is a sequence modeling toolkit that allows researchers and
developers to train custom models for translation, summarization, language
modeling and other text generation tasks.

### What's New:


- April 2020: [Monotonic Multihead Attention code released](examples/simultaneous_translation/README.md)
- April 2020: [Quant-Noise code released](examples/quant_noise/README.md)
- April 2020: [Initial model parallel support and 11B parameters unidirectional LM released](examples/megatron_11b/README.md)
- March 2020: [Byte-level BPE code released](examples/byte_level_bpe/README.md)
- February 2020: [mBART model and code released](examples/mbart/README.md)
- February 2020: [Added tutorial for back-translation](https://github.com/pytorch/fairseq/tree/master/examples/backtranslation#training-your-own-model-wmt18-english-german)
- December 2019: [fairseq 0.9.0 released](https://github.com/pytorch/fairseq/releases/tag/v0.9.0)
- November 2019: [VizSeq released (a visual analysis toolkit for evaluating fairseq models)](https://facebookresearch.github.io/vizseq/docs/getting_started/fairseq_example)
- November 2019: [CamemBERT model and code released](examples/camembert/README.md)
- November 2019: [BART model and code released](examples/bart/README.md)
- November 2019: [XLM-R models and code released](examples/xlmr/README.md)
- September 2019: [Nonautoregressive translation code released](examples/nonautoregressive_translation/README.md)
- August 2019: [WMT'19 models released](examples/wmt19/README.md)
- July 2019: fairseq relicensed under MIT license
- July 2019: [RoBERTa models and code released](examples/roberta/README.md)
- June 2019: [wav2vec models and code released](examples/wav2vec/README.md)

### Features:

Fairseq provides reference implementations of various sequence-to-sequence models, including:
- **Convolutional Neural Networks (CNN)**
  - [Language Modeling with Gated Convolutional Networks (Dauphin et al., 2017)](examples/language_model/conv_lm/README.md)
  - [Convolutional Sequence to Sequence Learning (Gehring et al., 2017)](examples/conv_seq2seq/README.md)
  - [Classical Structured Prediction Losses for Sequence to Sequence Learning (Edunov et al., 2018)](https://github.com/pytorch/fairseq/tree/classic_seqlevel)
  - [Hierarchical Neural Story Generation (Fan et al., 2018)](examples/stories/README.md)
  - [wav2vec: Unsupervised Pre-training for Speech Recognition (Schneider et al., 2019)](examples/wav2vec/README.md)
- **LightConv and DynamicConv models**
  - [Pay Less Attention with Lightweight and Dynamic Convolutions (Wu et al., 2019)](examples/pay_less_attention_paper/README.md)
- **Long Short-Term Memory (LSTM) networks**
  - Effective Approaches to Attention-based Neural Machine Translation (Luong et al., 2015)
- **Transformer (self-attention) networks**
  - Attention Is All You Need (Vaswani et al., 2017)
  - [Scaling Neural Machine Translation (Ott et al., 2018)](examples/scaling_nmt/README.md)
  - [Understanding Back-Translation at Scale (Edunov et al., 2018)](examples/backtranslation/README.md)
  - [Adaptive Input Representations for Neural Language Modeling (Baevski and Auli, 2018)](examples/language_model/transformer_lm/README.md)
  - [Mixture Models for Diverse Machine Translation: Tricks of the Trade (Shen et al., 2019)](examples/translation_moe/README.md)
  - [RoBERTa: A Robustly Optimized BERT Pretraining Approach (Liu et al., 2019)](examples/roberta/README.md)
  - [Facebook FAIR's WMT19 News Translation Task Submission (Ng et al., 2019)](examples/wmt19/README.md)
  - [Jointly Learning to Align and Translate with Transformer Models (Garg et al., 2019)](examples/joint_alignment_translation/README.md )
  - [Multilingual Denoising Pre-training for Neural Machine Translation (Liu et at., 2020)](examples/mbart/README.md)
  - [Neural Machine Translation with Byte-Level Subwords (Wang et al., 2020)](examples/byte_level_bpe/README.md)
- **Non-autoregressive Transformers**
  - Non-Autoregressive Neural Machine Translation (Gu et al., 2017)
  - Deterministic Non-Autoregressive Neural Sequence Modeling by Iterative Refinement (Lee et al. 2018)
  - Insertion Transformer: Flexible Sequence Generation via Insertion Operations (Stern et al. 2019)
  - Mask-Predict: Parallel Decoding of Conditional Masked Language Models (Ghazvininejad et al., 2019)
  - [Levenshtein Transformer (Gu et al., 2019)](examples/nonautoregressive_translation/README.md)


**Additionally:**
- multi-GPU (distributed) training on one machine or across multiple machines
- fast generation on both CPU and GPU with multiple search algorithms implemented:
  - beam search
  - Diverse Beam Search ([Vijayakumar et al., 2016](https://arxiv.org/abs/1610.02424))
  - sampling (unconstrained, top-k and top-p/nucleus)
- large mini-batch training even on a single GPU via delayed updates
- mixed precision training (trains faster with less GPU memory on [NVIDIA tensor cores](https://developer.nvidia.com/tensor-cores))
- extensible: easily register new models, criterions, tasks, optimizers and learning rate schedulers

We also provide [pre-trained models for translation and language modeling](#pre-trained-models-and-examples)
with a convenient `torch.hub` interface:
```python
en2de = torch.hub.load('pytorch/fairseq', 'transformer.wmt19.en-de.single_model')
en2de.translate('Hello world', beam=5)
# 'Hallo Welt'
```
See the PyTorch Hub tutorials for [translation](https://pytorch.org/hub/pytorch_fairseq_translation/)
and [RoBERTa](https://pytorch.org/hub/pytorch_fairseq_roberta/) for more examples.

![Model](fairseq.gif)

# Requirements and Installation

* [PyTorch](http://pytorch.org/) version >= 1.4.0
* Python version >= 3.6
* For training new models, you'll also need an NVIDIA GPU and [NCCL](https://github.com/NVIDIA/nccl)
* **For faster training** install NVIDIA's [apex](https://github.com/NVIDIA/apex) library:
```bash
git clone https://github.com/NVIDIA/apex
cd apex
pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" --global-option="--deprecated_fused_adam" --global-option="--xentropy" --global-option="--fast_multihead_attn" ./
```

To install fairseq:
```bash
pip install fairseq
```

On MacOS:
```bash
CFLAGS="-stdlib=libc++" pip install fairseq
```

If you use Docker make sure to increase the shared memory size either with
`--ipc=host` or `--shm-size` as command line options to `nvidia-docker run`.

**Installing from source**

To install fairseq from source and develop locally:
```bash
git clone https://github.com/pytorch/fairseq
cd fairseq
pip install --editable .
```

# Getting Started

The [full documentation](https://fairseq.readthedocs.io/) contains instructions
for getting started, training new models and extending fairseq with new model
types and tasks.

# Pre-trained models and examples

We provide pre-trained models and pre-processed, binarized test sets for several tasks listed below,
as well as example training and evaluation commands.

- [Translation](examples/translation/README.md): convolutional and transformer models are available
- [Language Modeling](examples/language_model/README.md): convolutional and transformer models are available
- [wav2vec](examples/wav2vec/README.md): wav2vec large model is available

We also have more detailed READMEs to reproduce results from specific papers:
- [Training with Quantization Noise for Extreme Model Compression](examples/quant_noise/README.md)
- [Neural Machine Translation with Byte-Level Subwords (Wang et al., 2020)](examples/byte_level_bpe/README.md)
- [Jointly Learning to Align and Translate with Transformer Models (Garg et al., 2019)](examples/joint_alignment_translation/README.md )
- [Levenshtein Transformer (Gu et al., 2019)](examples/nonautoregressive_translation/README.md)
- [Facebook FAIR's WMT19 News Translation Task Submission (Ng et al., 2019)](examples/wmt19/README.md)
- [RoBERTa: A Robustly Optimized BERT Pretraining Approach (Liu et al., 2019)](examples/roberta/README.md)
- [wav2vec: Unsupervised Pre-training for Speech Recognition (Schneider et al., 2019)](examples/wav2vec/README.md)
- [Mixture Models for Diverse Machine Translation: Tricks of the Trade (Shen et al., 2019)](examples/translation_moe/README.md)
- [Pay Less Attention with Lightweight and Dynamic Convolutions (Wu et al., 2019)](examples/pay_less_attention_paper/README.md)
- [Understanding Back-Translation at Scale (Edunov et al., 2018)](examples/backtranslation/README.md)
- [Classical Structured Prediction Losses for Sequence to Sequence Learning (Edunov et al., 2018)](https://github.com/pytorch/fairseq/tree/classic_seqlevel)
- [Hierarchical Neural Story Generation (Fan et al., 2018)](examples/stories/README.md)
- [Scaling Neural Machine Translation (Ott et al., 2018)](examples/scaling_nmt/README.md)
- [Convolutional Sequence to Sequence Learning (Gehring et al., 2017)](examples/conv_seq2seq/README.md)
- [Language Modeling with Gated Convolutional Networks (Dauphin et al., 2017)](examples/language_model/conv_lm/README.md)

# Join the fairseq community

* Facebook page: https://www.facebook.com/groups/fairseq.users
* Google group: https://groups.google.com/forum/#!forum/fairseq-users

# License
fairseq(-py) is MIT-licensed.
The license applies to the pre-trained models as well.

# Citation

Please cite as:

```bibtex
@inproceedings{ott2019fairseq,
  title = {fairseq: A Fast, Extensible Toolkit for Sequence Modeling},
  author = {Myle Ott and Sergey Edunov and Alexei Baevski and Angela Fan and Sam Gross and Nathan Ng and David Grangier and Michael Auli},
  booktitle = {Proceedings of NAACL-HLT 2019: Demonstrations},
  year = {2019},
}
```
