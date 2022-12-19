# AcademicRoBERTa

We pretrained a RoBERTa-based Japanese masked language model on paper abstracts from the academic database CiNii Articles.

## Download
```

```
## Requirements
Python >= 3.8 <br>
fairseq <br>
sentencepiece <br>
tensorboardX (optional) <br>

## Preprocess
データ形式はtextとlabelの間にタブ区切りを想定しています。
```
python ./apply_sp.py $TRAIN_SRC $DATASET_DIR/train.src-tgt --bpe_model $SENTENCEPIECE_MODEL
python ./apply_sp.py $VALID_SRC $DATASET_DIR/valid.src-tgt --bpe_model $SENTENCEPIECE_MODEL
python ./apply_sp.py $TEST_SRC $DATASET_DIR/test.src-tgt --bpe_model $SENTENCEPIECE_MODEL
```
```
fairseq-preprocess \
    --source-lang "src" \
    --target-lang "tgt" \
    --trainpref "${DATASET_DIR}/train.src-tgt" \
    --validpref "${DATASET_DIR}/valid.src-tgt" \
    --testpref "${DATASET_DIR}/test.src-tgt" \
    --destdir "data-bin/" \
    --workers 60 \
    --srcdict ${DICT} \
    --tgtdict ${DICT}
```
## Finetune
```
fairseq-train data-bin/ \
    --restore-file $ROBERTA_PATH \
    --max-positions 512 \
    --batch-size $MAX_SENTENCES \
    --task sentence_prediction \
    --reset-optimizer --reset-dataloader --reset-meters \
    --required-batch-size-multiple 1 \
    --init-token 0 --separator-token 2 \
    --arch roberta_base \
    --criterion sentence_prediction \
    --classification-head-name $HEAD_NAME \
    --num-classes $NUM_CLASSES \
    --dropout 0.1 --attention-dropout 0.1 --weight-decay 0.01 \
    --lr-scheduler polynomial_decay --lr $LR --total-num-update $TOTAL_NUM_UPDATES --warmup-updates $WARMUP_UPDATES \
    --optimizer adam --adam-betas "(0.9, 0.98)" --adam-eps 1e-08 \
    --clip-norm 0.0 \
    --max-epoch 999 \
    --patience 10 \
    --no-epoch-checkpoints --seed 88 --log-format simple --log-interval $LOG_INTERVAL --save-interval-updates $SAVE_INTERVAL \
    --best-checkpoint-metric accuracy --maximize-best-checkpoint-metric \
    --shorten-method "truncate" \
    --find-unused-parameters
```
