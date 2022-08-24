/main

↳/data

↳  train_dataset.csv: {article, labels}

inference_dataset.csv: {'company', 'title', 'article', 'opinion', 'firm', 'date'}

ensemble_inferenced.csv

today_dataset.csv: {'company', 'title', 'article', 'opinion', 'firm', 'date'}

stock_master.csv

sell_backtranslated_papago_English.csv

buy_backtranslated_papago_English.csv

↳/models

↳ /kb-albert-char-base-v2

kbalbert_augment_epoch1_fold5_without_papago.pt

kbalbert_augment_epoch3_fold5_without_papago.pt

kbalbert_augment_epoch5_fold5_without_papago.pt

kbalbert_origin_epoch1_fold5.pt

kbalbert_origin_epoch3_fold5.pt

koBERT_train.pt

roBERTa_train.pt

## <modules>

### backtranslation_papago.py

in → train_dataset.csv

```python
# backtranslation_config.json
{
    "data_file_path": "./data/train_dataset.csv",
    "language": "English" # or Chinese
}
```

out **→** sell_backtranslated_papago_en.csv, buy_backtranslated_papago_en.csv

### DAPT.py

↳etc_modules.py

in → inference_dataset.csv

```python
# DAPT_config.json
{
    "model_name": "koBERT", # or roBERTa-large
    "batch": 4,
    "epochs": 20,
    "data_file": "./data/inference_dataset.csv"
}
```

out → 허깅페이스 업로드

### classification_train.py

↳data_control.py, model.py, etc_modules.py

in → train_dataset.csv,  sell_backtranslated_papago_en.csv, buy_backtranslated_papago_en.csv

```python
# classification_train_config.json
{
    "fold_length": 5,
    "batch": 16,
    "seed_val": 42,
    "epochs": 3,
    "data_path": "./data/",
    "data_file": [
        "train_dataset.csv",
        "sell_backtranslated_papago_English.csv",
        "buy_backtranslated_papago_English.csv"
    ],
    "model_path": "./models/",
    "pretrained_model": "kb-albert-char-base-v2", # or "klue/roberta-large" or "monologg/kobert"
    "save_model": "kbalbert_after_transfer_learning.pt"
}
```

out → pt

### inference.py

↳data_control.py, model.py, etc_modules.py

in → inference_dataset.csv, today_dataset.csv

```python
# inference_config.json
{
    "data_file_path": "./data/inference_dataset.csv",
    "save_file_path": "./data/ensemble_inferenced.csv",
    "pretrained_model": {
        "kbalbert": "./models/kb-albert-char-base-v2",
        "klue/roberta-large": "nile/roBERTa-large-finetuned-wholemasking20",
        "monologg/kobert": "nile/koBERT-finetuned-wholemasking20"
    },
    "model_path": "./models/",
    "inference_model_list": {
        "kbalbert": [
            "kbalbert_augment_epoch1_fold5_without_papago.pt",
            "kbalbert_augment_epoch3_fold5_without_papago.pt",
            "kbalbert_augment_epoch5_fold5_without_papago.pt",
            "kbalbert_origin_epoch1_fold5.pt",
            "kbalbert_origin_epoch3_fold5.pt"
        ],
        "roberta-large": [
            "roBERTa_train.pt"
        ],
        "kobert": [
            "koBERT_train.pt"
        ]
    }
}
```

out → ensemble_inferenced.csv

### **scraping_today_dataset.py**

out → today_dataset.csv

### **insertDB.py**

↳postgreSQL.py

in → ensemble_inferenced.csv

### <자동화>

**매일 업로드**

※ 절대 경로로 설정하기

```bash
#!/bin/bash

/home/piai/anaconda3/bin/python3 /home/piai/hustar/scraping_today_dataset.py
/home/piai/anaconda3/bin/python3 /home/piai/hustar/inference.py
/home/piai/anaconda3/bin/python3 /home/piai/hustar/insertDB.py
```

```bash
#!/bin/bash

/home/piai/anaconda3/bin/python3 /home/piai/hustar/backtranslation_papago.py
/home/piai/anaconda3/bin/python3 /home/piai/hustar/DAPT.py
/home/piai/anaconda3/bin/python3 /home/piai/hustar/classification_train.py
```

```bash
crontab -e

SHELL = /bin/bash
00 02 * * * /home/piai/hustar/automation.sh >> /home/piai/hustar/automation.log 2>&1
* * 31 12 * /home/piai/hustar/train.sh >> /home/piai/hustar/train.log 2>&1

chmod +x /home/piai/hustar/automation.sh
chmod +x /home/piai/hustar/train.sh
service cron restart
```
