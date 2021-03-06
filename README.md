## Distilling the Knowledge of BERT for Sequence-to-Sequence ASR

### Requirements
Python  3.7.4  
Pytorch 1.3.0  
subword-nmt https://github.com/rsennrich/subword-nmt    
transformers https://github.com/huggingface/transformers

### Data preparation

We used two corpus:
the Corpus of Spontaneous Japanese (CSJ) and the Balanced Corpus of Contemporary Written Japanese (BCCWJ).
CSJ is for training of ASR and BERT, and BCCWJ is for training of BERT.

1. Prepare CSJ-APS and CSJ-SPS data in the same format as `./data/csj/csj.example`.
They should be put as `./data/csj/csj.aps` and `./data/csj/csj.sps`.

2. Prepare BCCWJ-LB and BCCWJ-PB data in the same format as `./data/bccwj/bccwj.example`.
They should be put as `./data/csj/bccwj.lb` and `./data/csj/bccwj.pb`.

3. run `./prep-bccwj.sh` (cd: `./prep`)

4. run `./prep-csj.sh` (cd: `./prep`)

### Pre-train BERT

We trained BERT on BCCWJ-LB and BCCWJ-PB first, then on the transcriptions of CSJ-APS and CSJ-SPS.
We used TITAN X (12GB) 3 GPUs for pre-training (about 4 days).

```
(cd: ./bert)
python train.py -conf bert-bccwj.config
python train.py -conf bert-csj.config
```

### Soft label preparation

```
(cd: ./bert)
python prep_soft_labels.py -conf bert-csj.config -model checkpoints/bert.csj.network.epoch50 -ctx utt
python prep_soft_labels.py -conf bert-csj.config -model checkpoints/bert.csj.network.epoch50 -ctx full
```

### Train seq2seq ASR

baseline seq2seq ASR
```
(cd: ./asr)
python train.py -conf base.config
```
seq2seq ASR with soft labels from BERT (utterance-level)
```
(cd: ./asr)
python train.py -conf bert-utt.config
```
seq2seq ASR with soft labels from BERT (full-length)
```
(cd: ./asr)
python train.py -conf bert-full.config
```

### Test ASR

Prepare test data in the same format as `./data/csj/csj.eval1.example` and be put as `./data/csj/csj.eval1`

```
(cd: ./asr)
python test.py -conf {base.config or bert-utt.config or bert-full.config} \
-model checkpoints/{base, distill.bert.utt, distill.bert.full}.network.epoch{}
```

### Result

|  | WER(eval1) |
|:---:|:---:|
| baseline | 10.31 |
| BERT(utterance) | 9.53 |
| BERT(full-length) | **9.19** |
