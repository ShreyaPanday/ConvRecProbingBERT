# Probing LMs for Conversational Recommendation

In our paper *"What does BERT know about books, movies and music? Probing BERT for Conversational Recommendation"* we devise probing tasks to evaluate language models knowledge already stored in its parameters. We probe LMs (without any finetunning) for three types of knowledge: genre, search and recommendation.

## Running probes

1. Clone repo and install rec_probing in a python (>=3.6) virtual env:  
```
git clone https://github.com/Guzpenha/ConvRecProbingBERT.git
cd ConvRecProbingBERT

python3 -m venv env
source env/bin/activate
pip install -r requirements.txt

cd rec_probing
pip install -e .
```

2. Download required datasets and use scripts to preprocess them:
```
./download_data.sh
./run_datasets_creation.sh
```

This will download and preprocess a few datasets:
  
|| Recommendation | Search | Conversational Recommendation |
|-------------|-------------|------------|------------|
|Movies | ML25M: 25m movie ratings | Reviews crawled from IMDB | Conversations crawled from /r/moviesuggestions/
| Books | GoodReads: 200m book interactions | Reviews from GoodReads | Conversations crawled from /r/booksuggestions/ |
| Music | Amazon-Music: 2.3m ratings/reviews | Reviews from Amazon-Music | Conversations crawled from /r/musicuggestions/ | 

As well as categories information for items of the 3 domains.

3. Use our python scripts to run probes:

```
# Search and Recommendation
python run_probes.py \
    --task $TASK \
    --probe_type ${PROBE_TYPE} \
    --input_folder $REPO_DIR/data/${PROBE_TYPE}/ \
    --output_folder $REPO_DIR/data/output_data/probes/ \
    --number_queries $NUMBER_PROBE_QUERIES \
    --number_candidates 5 \
    --batch_size 64 \
    --probe_technique ${PROBE_TECHNIQUE} \
    --bert_model 'bert-base-cased' 
```

Where PROBE_TYPE can be ['recommendation', 'search'], PROBE_TECHNIQUE can be ['mean-sim', 'cls-sim', 'nsp'] and TASK can be ['ml25m' 'gr' 'music'] for the domains of movies, books and music respectivelly.

```
# Genres
python run_mlm_probe.py \
    --task $TASK \
    --input_folder $REPO_DIR/data/recommendation/ \
    --output_folder $REPO_DIR/data/output_data/probes/ \
    --number_queries $NUMBER_PROBE_QUERIES \
    --batch_size 32 \
    --sentence_type ${SENTENCE_TYPE} \
    --bert_model 'roberta-large'
```
Where SENTENCE_TYPE can be ['no-item', 'type-I', 'type-II'] and TASK can be ['ml25m' 'gr' 'music'] for the domains of movies, books and music respectivelly.

## Running response ranking for reddit conv. recommendation data

In order to get the results from Table 7 of the paper, regarding models conversation response ranking results on the conversational recomendation reddit data, use:


```
cd list_wise_reformer
pip install -e .
cd list_wise_reformer/scripts
./run_all_dialogue_baselines.sh
```

Ignore that the package is named list_wise_reformer. It contains several baselines for dialogue, search and recommendation, including a prototype of a list wise Reformer model.

## Infusing knowledge
We simply train the models for the probing tasks before fine-tunning for the final task, and use this model as input to the previous code.

```
python pre_train_BERT.py \
    --task $TASK \
    --probe_type ${PROBE_TYPE} \
    --input_folder $REPO_DIR/data/recommendation/ \
    --output_folder $REPO_DIR/data/output_data/probes/ \
    --number_queries $NUMBER_PROBE_QUERIES \
    --number_candidates 1 \
    --batch_size 32 \
    --num_epochs 5 \
    --bert_model "bert-base-cased"
```

## Experiments with ReDial Data

They use the same framework from other tasks, the difference is that we need to create the adversarial test data. For that we use the script data/genereate_adversarial_test.py.

