<p align="center"><img src="./img/enai_logo.png"></p>

# Pre-Trained Korean Language Model
NLP 발전을 위해 한글 Corpus로 Pre-train한 Language Model을 공개합니다.

* Pre-Train Model Download
  * [Small Download (18M Params)](https://drive.google.com/open?id=13D9Fnnl0ra1qjPgtSWdp1-xIs6DfJ7Zg)
    * [Small with LMHead](https://drive.google.com/file/d/1QXwQ8dg4p7Xhr2GLN4joREYNgfL86trP/view?usp=sharing)
  * [Large-V1 Download (330M Params)](https://drive.google.com/file/d/1n0B3pK8DkkBvEpEXnjUX4a523LfPtumx/view?usp=sharing)
    * [Large-V1 with LMHead](https://drive.google.com/file/d/1uPZ0LeXsxMmzfDNiZIJxBOc1XDEpV1nr/view?usp=sharing)
  * Large-V2 Download (In Progress)

Large Model의 경우 Fine-Tuning Step에서도 많은 Computational resource가
필요하기 때문에 고사양 Machine이 없을 시 Fine-Tuning이 어렵습니다. 이에
따라 Benchmark를 진행한 3가지 Task(KorQuAD1.0, KorNLI, KorSTS)에 대한
Fine-Tuning Model도 공개합니다.
* Fine-Tuning Model Download
  * V1
    * [KorQuAD1.0 (EM:85.61/F1:93.89)](https://drive.google.com/file/d/1kanzo9DkHfxjXGtjq62C-ZKpsPrmoE3l/view?usp=sharing)
    * [KorNLI (spearman: 81.68)](https://drive.google.com/file/d/18QP4lpoqM46PLTBHJxGdzzVSqrLT9inC/view?usp=sharing)
    * [KorSTS (acc: 83.9)](https://drive.google.com/file/d/1nVsSXnRrr6xJjkECe9tkUptt8ynnkiAz/view?usp=sharing)
  * V2
    * In Progress

## Pre-train Corpus
* Small: 한국어 Wikipedia
* V1: 한국어 Wikipedia + News (88M Sentences)
* V2: 한국어 Wikipedia + News (174M Sentences)


## Model Detail
* Masking Strategy: Dynamic
  Masking([RoBERTa](https://arxiv.org/abs/1907.11692)) + n-gram
  Masking([ALBERT](https://arxiv.org/abs/1909.11942))
* Additional Task: SOP(Sentence Order
  Prediction)([ALBERT](https://arxiv.org/abs/1909.11942))
* Optimizer:
  * Small: Adam Optimizer
  * Large: [Lamb Optimizer](https://arxiv.org/abs/1904.00962)
* Scheduler:
  * Small: LinearWarmup
  * Large:
    [PolyWarmup](https://github.com/NVIDIA/DeepLearningExamples/blob/master/PyTorch/LanguageModeling/BERT/schedulers.py)
* Mixed-Precision Opt Level "O1"
  ([Nvidia Apex](https://nvidia.github.io/apex/amp.html))
* Hyper-parameters

| Hyper-parameter       | Small Model | Large Model       |
|:----------------------|:------------|:------------------|
| Number of layers      | 12          | 24                |
| Hidden Size           | 256         | 1024              |
| FFN inner hidden size | 1024        | 4048              |
| Attention heads       | 8           | 16                |
| Attention head size   | 32          | 64                |
| Mask percent          | 15          | 15                |
| Learning Rate         | 0.0001      | 0.00125           |
| Warmup Proportion     | 0.1         | 0.0125            |
| Attention Dropout     | 0.1         | 0.1               |
| Dropout               | 0.1         | 0.1               |
| Batch Size            | 256         | 2048              |
| Train Steps           | 500k        | 125k(V1) 250k(V2) |


## Model Benchmark

|                               | KorQuAD1.0 (EM/F1) | KorNLI (acc) | KorSTS (spearman) |
|:-----------------------------:|:------------------:|:------------:|:-----------------:|
| multilingual-BERT (Base Size) |    70.42/90.25     |    76.33     |       77.90       |
|      KoBERT (Base Size)       |    52.81/80.27     |    79.00     |       79.64       |
|     KoELECTRA (Base Size)     |    61.10/89.59     |    80.85     |       83.21       |
|      HanBERT (Base Size)      |    78.74 / 92.02   |    80.89     |       83.33       |
|       Ours (Small Size)       |    78.98/88.20     |    74.67     |       74.53       |
|       Ours (Large Size)       |     85.61/93.89    |    81.68     |       83.90       |
| This repository (Small Size)  |  **78.68/88.59**   |      -       |          -        |

* **Fine-tuning Setting (Ours Model)**
  * Optimizer: Adam
  * Scheduler: LinearWarmup
  * Mixed Precision Opt Level "O2"
  * KorQuAD1.0
    * lr: 5e-5
    * epochs: 4
    * batch size: 16
  * KorNLI
    * lr: 2e-5
    * epochs: 3
    * batch size: 32
  * KorSTS
    * Fine-tuning Step에서 분산은 상당히 클 수 있으므로 상대적으로
      Dataset의 크기가 작은 KorSTS Task는 Random Seed에 대해 Grid
      Search({1~10})를 사용하여 가장 성능이 좋은 Model을 사용하였습니다.
      ([Reference](https://arxiv.org/abs/2002.06305))
    * lr: 3e-5
    * epochs: 10
    * batch size: 16(Large) 32(Small)
    * best random seed: 9(Large) 7(Small)

* **[AI NLP 대회 참여] BERT-Dep (single) (Virssist)**
  * Colab Notebook https://colab.research.google.com/drive/1LpkZLmfsZz5LHCh0ivaR9TCr7CxOPYE-?usp=sharing
  * Dev set score: 78.68/88.59 (EM/F1) 
  * Test set score: 77.30/87.45 (EM/F1)
  * 답변의 start과 end사이에 의존관계가 있다는 아이디어를 이용하여 입력문자열에 의해 생성된 start와 마지막 히든 상태 H을 concatenating 시켜 end를 예측 하게 하였습니다. 아래 단락은 저희가 참조한 논문의 일부분입니다. 논문의 구현체는 https://github.com/cooelf/AwesomeMRC 에서 구할 수 있었습니다.
  * [Machine Reading Comprehension: The Role of Contextualized Language Models and Beyond](https://arxiv.org/pdf/2005.06249.pdf)
    * 5.6.4 Answer Dependency. Recent studies separately use H to predict the start and end
spans for the answer, neglecting the dependency of the start and end representations.
We model the dependency between start and end logits by concatenating the start logits
and H through a linear layer to obtain the end logits where [; ] denotes concatenation: 
<p align="center"><img src="./img/linear.png" width=200px></p>

## Example Scripts
**KorQuAD1.0**
* Train
```shell
python3 run_qa.py \
  --checkpoint $MODEL_FILE \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --train_file data/korquad/KorQuAD_v1.0_train.json \
  --max_seq_length 512 \
  --doc_stride 128 \
  --max_query_length 64 \
  --max_answer_length 30 \
  --per_gpu_train_batch_size 16 \
  --learning_rate 5e-5 \
  --num_train_epochs 4.0 \
  --adam_epsilon 1e-6 \
  --warmup_proportion 0.1
```
* Eval
```shell
python3 eval_qa.py \
  --checkpoint $MODEL_FILE \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --predict_file data/korquad/KorQuAD_v1.0_dev.json \
  --max_seq_length 512 \
  --doc_stride 64 \
  --max_query_length 64 \
  --max_answer_length 30 \
  --batch_size 16 \
  --n_best_size 20
```
**KorNLI**
* Train
```shell
python3 run_classifier.py \
  --data_dir data/kornli \
  --task_name kornli \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --checkpoint $MODEL_FILE \
  --do_eval \
  --max_seq_length 128 \
  --train_batch_size 32 \
  --eval_batch_size 32 \
  --learning_rate 3e-5 \
  --num_train_epochs 3.0 \
  --warmup_proportion 0.1
```
* Eval
```shell
python3 eval_classifier.py \
  --data_dir data/kornli \
  --task_name kornli \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --checkpoint $MODEL_FILE \
  --max_seq_length 128 \
  --eval_batch_size 32
```

**KorSTS**
* Train
```shell
python3 run_classifier.py \
  --data_dir data/korsts \
  --task_name korsts \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --checkpoint $MODEL_FILE \
  --do_eval
  --max_seq_length 128 \
  --train_batch_size 16 \
  --eval_batch_size 32 \
  --learning_rate 3e-5 \
  --num_train_epochs 10.0 \
  --warmup_proportion 0.1
```
* Eval
```shell
python3 eval_classifier.py \
  --data_dir data/korsts \
  --task_name korsts \
  --config_file $CONFIG_FILE \
  --vocab_file $VOCAB_FILE \
  --checkpoint $MODEL_FILE \
  --max_seq_length 128 \
  --eval_batch_size 32
```

## Acknowledgement
본 연구는 과학기술정보통신부 및 정보통신산업진흥원의 ‘고성능 컴퓨팅 지원’ 사업으로부터 지원받아 수행하였음  
Following(or This research) was results of a study on the "HPC Support" Project, supported by the ‘Ministry of Science and ICT’ and NIPA.

## Reference
* [Google BERT](https://github.com/google-research/bert)
* [Huggingface Transformers](https://github.com/huggingface/transformers)
* [Nvidia BERT](https://github.com/NVIDIA/DeepLearningExamples/blob/master/PyTorch/LanguageModeling/BERT)
* [Nvidia Apex](https://nvidia.github.io/apex/index.html)
* [RoBERTa](https://arxiv.org/abs/1907.11692)
* [ALBERT](https://arxiv.org/abs/1909.11942)
* [KoBERT](https://github.com/SKTBrain/KoBERT)
* [KoELECTRA](https://github.com/monologg/KoELECTRA)
* [KorNLUDatasets](https://github.com/kakaobrain/KorNLUDatasets)


---
* 추가적으로 궁금하신점은 해당 repo의 issue를 등록해주시거나 ekjeon@enliple.com으로 메일 주시면 답변 드리겠습니다.
