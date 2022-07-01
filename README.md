# Relation Extraction Database Based on Wikidata

This repository contains a lexical resource for English, sentence-level relation extraction (RE) based on Wikidata. The resource is distributed as an SQLite database for easy quering and storing. It includes more than 47 million examples collected from different corpora for RE, and it covers more than 1,000 unique Wikidata properties.

You can use, for example, this [DB browser](https://sqlitebrowser.org/) to read the database.

## Datasets

Datasets were preprocessed for **sentence-based** relation extraction. 

* \# of R types: how many unique Wikidata properties are in the dataset
* Negative examples are sentences with no relation detected or with an unknown relation between entities.

| dataset                               |# of instances|# of R types|% of neg. examples| human checks | license      |
|---------------------------------------|--------------|------------|---------|--------------|--------------|
| FewRel ([Han et al., 2018](https://aclanthology.org/D18-1514/)) [[data](https://github.com/thunlp/FewRel)]   | 56,000       | 80         | 0\%     | yes          | MIT          |
| T-REx ([Elsahar et al., 2018](https://aclanthology.org/L18-1544/)) [[data](https://hadyelsahar.github.io/t-rex/downloads/)]   | 12,081,023   | 652        | 0\%     | no           | CC BY-SA 4.0 |
| DocRED ([Yao et al., 2019](https://aclanthology.org/P19-1074/)) [[data](https://github.com/thunlp/DocRED)]  | 778,914      | 96         | 0\%     | no (yes*)    | MIT          |
| WikiFact ([Goodrich et al., 2019](https://dl.acm.org/doi/10.1145/3292500.3330955)) [[data](https://github.com/google-research-datasets/wikifact/)] | 33,628,338   | 934        | 92\%    | no           | CC BY 4.0    |
| Wiki20m ([Han et al., 2020](https://aclanthology.org/2020.aacl-main.75/)) [[data](https://github.com/thunlp/OpenNRE/blob/master/benchmark/download_wiki20m.sh)]| 738,463      | 81         | 60\%    | no           | MIT          |
| WebRED ([Ormandi et al., 2021](https://arxiv.org/abs/2102.09681)) [[data](https://github.com/google-research-datasets/WebRED)]| 107,819      | 385        | 54\%    | yes          | CC BY 4.0    |
| our database (DB)                     | 47,390,557   | 1,022      | 66\%    | yes/no       | CC BY-SA 4.0 |


* Some part of DocRED was verified by humans.

## What's inside the DB?

The DB has one table `corpora` where all dataset instances are stored.

### Columns

|column name         | explanation | example | comment |
|--------------------|--------------|------------|---------|
| id                 | ID in db          |  INTEGER           |         |         
| relation_id        | property ID from Wikidata (*)          |  P131           |         |
| relation_name      | property label from Wikidata (*)         | located in the administrative territorial entity        |         |
| sentence           | a sentence where subject and object are marked          |  SUBJ{Magnificent Mile}, a SUBJ{neighborhood} in OBJ{Chicago}, Illinois, U.S.           |         |
| dataset            | name of the dataset ( webred/docred/wikifact)          |  trex          |         |
| human_checks       | Was this sentence manually verified by humans?          | True/False            |         |
| dataset_part       | The dataset split the instance belongs to (train or dev). Test data is not included.          | train/dev            |         |
| source_name        | subject text          |  Magnificent Mile           |         |
| target_name        | object text          |  Chicago           |         |
| source_wikidata_id | subject ID from Wikidata          | Q3056359            | only in FewRel, Wiki20m        |
| target_wikidata_id | object ID from Wikidata                                                                   | Q1331049            | only in FewRel, Wiki20m         |
| sentence_tokenised | tokenised sentence                                                                        | The Lakes is a locality in OBJ{Western Australia} within the SUBJ{Shire of Mundaring} .            |   only in FewRel, Wiki20m, DocRED |
| title              | title of the document where the sentence comes from                                       |  The Lakes, Western Australia    | only in T-REx, DocRED        |
| sent_id            | sentence ID from the document                                                             | INTEGER             | only in T-REx, DocRED        |
| source_entity_type | subject entity type (LOC/PER/TIME/MISC)                                                   | ORG            | only in DocRED        |
| target_entity_type | object entity type (LOC/PER/TIME/MISC)                                                    |  LOC           | only in DocRED        |
| triple_annotator   | aligner that was used for aligning the triple with text (see T-REx docs)                  | Simple-Aligner         | only in T-REx        |
| source_annotator   | aligner that was used for aligning the subject with text (see T-REx docs)                 | Wikidata_Spotlight_Entity_Linker            | only in T-REx        |
| target_annotator   | aligner that was used for aligning the object with text (see T-REx docs)                  | Date_Linker            | only in T-REx        |
| num_pos_raters     | number of unique human raters who thought that the sentence expresses the given relation. | INTEGER            |  only in WebRED       |
| num_raters         | number of unique human raters whouannotated the sentence-fact pair                        | INTEGER            |  only in WebRED       |
| url                | document url where the sentence comes from                                                |   https://en.wikipedia.org/wiki/Miracle_Mile          |  only in WebRED    |

(*) Relation IDs and names include `P0` (absence of relation; name: "no_relation") and `NA` (name: "unknown").

## Preprocessing Details for Datasets

### WebRED

* Calculate rater confidence and assign P0 to the cases where it was low.
* Separate into train and dev (90/10).

### WikiFact

* Delete instances where SUBJ and OBJ cover the same token(s).
* Delete instances with properties which does not exist in Wikidata anymore (e.g., P5130, P134, P1432, P1962, P1773).

### FewRel

* Create detokenised sentences with MosesDetokenizer from `sacremoses`.

### Wiki20m

* Create detokenised sentences with MosesDetokenizer from `sacremoses`.
* Don't include sentences that are present in FewRel.
* Add sentences that are present in FewRel but have a different relation (66 instances).
* Delete duplicate entries.

### DocRED

* Extract relations that have subject and object present in the same sentence.
* Create detokenised sentences with MosesDetokenizer from `sacremoses`.
* For DocRED-human: discard some examples, e.g., with missing subjects/objects, no evidence support (see [this issue](https://github.com/thunlp/DocRED/issues/19)).

### T-REx

* Take all the alignments where word boundaries for subject/object are present.
* Delete duplicates among all the alignments for a text.

### Other Remarks
 
We do train/dev division only for WebRED, since it is human-annotated. We don't do it for T-REx. All other corpora provide a validation set.

## Other Datasets

There exist other datasets that use Wikidata for RE. They were not included in the DB.

* KELM/TEKGEN ([url](https://github.com/google-research-datasets/KELM-corpus), [paper](https://aclanthology.org/2021.naacl-main.278/))
* REBEL ([url](https://github.com/Babelscape/rebel), [paper](https://aclanthology.org/2021.findings-emnlp.204/))

## Citing

If you use the database, please cite every used dataset.
