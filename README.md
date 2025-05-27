# Different Speech Translation Models Encode and Translate Speaker Gender Differently

This repository contains the code associated with the ACL2025 paper  
[_**Different Speech Translation Models Encode and Translate Speaker Gender Differently**_](link_to_be_added).

## 📦 Getting Started

These instructions will help you set up the environment and run the core experiments.

### 1. Clone the Repository

```
git clone https://github.com/dennisfcc/speech-translation-gender.git
cd speech-translation-gender
```

### 2. Install Dependencies

```
pip install -r requirements.txt
```

## 🚀 Example Usage

The following examples demonstrate how to extract hidden states from speech translation models, train a probing classifier, and evaluate its performance. 

Below is a description of the required parameters:
- `${*_data_tsv}` refers to the TSV files containing the training, validation, or test datasets.
- `${*_embeddings_h5}` is the path to the HDF5 file where the extracted hidden states will be stored.
- `${lang}` specifies the language code of the input data (es, fr, it).
- `${model}` is the name of the model hosted on the Hugging Face Hub.
- `${saved_probe}` defines the path where the trained probe will be saved during training, and from which 
it will be loaded during evaluation.

### Extract States

```
python /path/to/speech-translation-gender/cli/extract_embeddings.py \
  --tsv-path ${*_data_tsv} --output-file ${*_embeddings_h5} \
  --lang ${lang} --model-name ${model_name} \
  --layer post_adapter --num-workers 0 --max-seq-len 60
```

### Train Probe

```
python /path/to/speech-translation-gender/cli/train_probe.py \
  --dataframe-train ${train_data_tsv} --embeddings-train ${train_embeddings_h5} \
  --dataframe-val ${validation_data_tsv} --embeddings-val ${validation_embeddings_h5} \
  --probe attention --level sequence --attention-type scaled_dot \
  --save-probe ${saved_probe} \
  --batch-size 32 --update-frequency 1 --tol 0.00001 --max-iter 40000 --early-stopping 20 \
  --num-layers 1 --num-heads 1 --dropout 0.0 --dropout-att 0.0 --learning-rate 0.0001
```

### Evaluate Probe

```
python /path/to/speech-translation-gender/cli/evaluate_probe.py \
  --dataframe ${data_tsv_file} --embeddings ${embeddings_h5_file} \
  --output-tsv ${output_tsv_file} \
  --probe attention --attention-type scaled_dot --num-layers 1 \
  --level sequence --pretrained-probe ${saved_probe} --batch-size 32
```

The following examples demonstrate how to generate translations and evaluate them. For gender accuracy evaluation, 
the official MuST-SHE script is used, which also requires `mosesdecoder` for tokenization.

Translations are saved in the `${output_tsv}` TSV file. 
The scripts print evaluation results, which include both general translation quality and 
gender accuracy and coverage.



### Evaluate Translations

```
# Generate translations
python /path/to/speech-translation-gender/cli/transcribe_data.py \
  --tsv-path ${test_data_tsv} --lang ${lang} --model-name ${model_name} --output-file ${output_tsv} \
  --num-workers 0 --batch-size 1 --max-seq-len 60

# Evaluate gender accuracy
cut -f4 ${output_tsv} | tail -n+2 > __h__
python /path/to/mustshe-directory/MuST-SHE-v1.1-eval-script/mustshe_acc_v1.1.py \
  --input <(perl /path/to/mosesdecoder/scripts/tokenizer/tokenizer.perl -l ${lang} -q -no-escape < __h__) \
  --tsv-definition ${output_tsv}
rm __h__

# Evaluate translation quality
python /path/to/speech-translation-gender/cli/evaluate_translation.py --file-path ${output_tsv} 
```


## 📄 Citing the Paper

```
@inproceedings{fucci-et-al-2025-different,
title = "Different Speech Translation Models Encode and Translate Speaker Gender Differently",
author = {Fucci, Dennis and Gaido, Marco and Negri, Matteo and Bentivogli, Luisa and 
Martins, André F. T. and Attanasio, Giuseppe},
booktitle = "Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics",
year = "2025",
address = "Vienna, Austria",
publisher = "Association for Computational Linguistics"
}
```
