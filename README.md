# Relation Extraction Database Based on Wikidata

This repository contains a lexical resource for English, sentence-level relation extraction (RE) based on Wikidata. The resource is distributed as an SQLite database for easy quering and storing. It includes more than 47 million examples collected from different corpora for RE, and it covers more than 1,000 unique Wikidata properties.

You can use, for example, this [DB browser](https://sqlitebrowser.org/) to read the database.

The database can be downloaded from [osf.io](https://osf.io/nr3km/files/osfstorage/62c53de5d202a120875c99e5) (4.5 G).

This database accompanies the paper "[Knowledge Extraction From Texts Based on Wikidata](https://aclanthology.org/2022.naacl-industry.33)" (Shimorina et al., NAACL 2022).

## Datasets

The following six datasets were preprocessed for **sentence-based** relation extraction: FewRel, T-REx, DocRED, WikiFact, Wiki20m, WebRED.

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


\* Some part of DocRED was verified by humans.

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

### Database

```
@inproceedings{shimorina-etal-2022-knowledge,
    title = "Knowledge Extraction From Texts Based on {W}ikidata",
    author = "Shimorina, Anastasia  and
      Heinecke, Johannes  and
      Herledan, Fr{\'e}d{\'e}ric",
    booktitle = "Proceedings of the 2022 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies: Industry Track",
    month = jul,
    year = "2022",
    address = "Hybrid: Seattle, Washington + Online",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2022.naacl-industry.33",
    pages = "297--304",
    abstract = "This paper presents an effort within our company of developing knowledge extraction pipeline for English, which can be further used for constructing an entreprise-specific knowledge base. We present a system consisting of entity detection and linking, coreference resolution, and relation extraction based on the Wikidata schema. We highlight existing challenges of knowledge extraction by evaluating the deployed pipeline on real-world data. We also make available a database, which can serve as a new resource for sentential relation extraction, and we underline the importance of having balanced data for training classification models.",
}
```

### DocRED

```
@inproceedings{yao-etal-2019-docred,
    title = "{D}oc{RED}: A Large-Scale Document-Level Relation Extraction Dataset",
    author = "Yao, Yuan  and
      Ye, Deming  and
      Li, Peng  and
      Han, Xu  and
      Lin, Yankai  and
      Liu, Zhenghao  and
      Liu, Zhiyuan  and
      Huang, Lixin  and
      Zhou, Jie  and
      Sun, Maosong",
    booktitle = "Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics",
    month = jul,
    year = "2019",
    address = "Florence, Italy",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/P19-1074",
    doi = "10.18653/v1/P19-1074",
    pages = "764--777",
    abstract = "Multiple entities in a document generally exhibit complex inter-sentence relations, and cannot be well handled by existing relation extraction (RE) methods that typically focus on extracting intra-sentence relations for single entity pairs. In order to accelerate the research on document-level RE, we introduce DocRED, a new dataset constructed from Wikipedia and Wikidata with three features: (1) DocRED annotates both named entities and relations, and is the largest human-annotated dataset for document-level RE from plain text; (2) DocRED requires reading multiple sentences in a document to extract entities and infer their relations by synthesizing all information of the document; (3) along with the human-annotated data, we also offer large-scale distantly supervised data, which enables DocRED to be adopted for both supervised and weakly supervised scenarios. In order to verify the challenges of document-level RE, we implement recent state-of-the-art methods for RE and conduct a thorough evaluation of these methods on DocRED. Empirical results show that DocRED is challenging for existing RE methods, which indicates that document-level RE remains an open problem and requires further efforts. Based on the detailed analysis on the experiments, we discuss multiple promising directions for future research. We make DocRED and the code for our baselines publicly available at https://github.com/thunlp/DocRED.",
}
```

### FewRel

```
@inproceedings{han-etal-2018-fewrel,
    title = "{F}ew{R}el: A Large-Scale Supervised Few-Shot Relation Classification Dataset with State-of-the-Art Evaluation",
    author = "Han, Xu  and
      Zhu, Hao  and
      Yu, Pengfei  and
      Wang, Ziyun  and
      Yao, Yuan  and
      Liu, Zhiyuan  and
      Sun, Maosong",
    booktitle = "Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing",
    month = oct # "-" # nov,
    year = "2018",
    address = "Brussels, Belgium",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/D18-1514",
    doi = "10.18653/v1/D18-1514",
    pages = "4803--4809",
    abstract = "We present a Few-Shot Relation Classification Dataset (dataset), consisting of 70, 000 sentences on 100 relations derived from Wikipedia and annotated by crowdworkers. The relation of each sentence is first recognized by distant supervision methods, and then filtered by crowdworkers. We adapt the most recent state-of-the-art few-shot learning methods for relation classification and conduct thorough evaluation of these methods. Empirical results show that even the most competitive few-shot learning models struggle on this task, especially as compared with humans. We also show that a range of different reasoning skills are needed to solve our task. These results indicate that few-shot relation classification remains an open problem and still requires further research. Our detailed analysis points multiple directions for future research.",
}
```

### T-REx

```
@inproceedings{elsahar-etal-2018-rex,
    title = "{T}-{RE}x: A Large Scale Alignment of Natural Language with Knowledge Base Triples",
    author = "Elsahar, Hady  and
      Vougiouklis, Pavlos  and
      Remaci, Arslen  and
      Gravier, Christophe  and
      Hare, Jonathon  and
      Laforest, Frederique  and
      Simperl, Elena",
    booktitle = "Proceedings of the Eleventh International Conference on Language Resources and Evaluation ({LREC} 2018)",
    month = may,
    year = "2018",
    address = "Miyazaki, Japan",
    publisher = "European Language Resources Association (ELRA)",
    url = "https://aclanthology.org/L18-1544",
}
```

### WebRED

```
@misc{ormandi2021webred,
      title={WebRED: Effective Pretraining And Finetuning For Relation Extraction On The Web}, 
      author={Robert Ormandi and Mohammad Saleh and Erin Winter and Vinay Rao},
      year={2021},
      eprint={2102.09681},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```

### Wiki20m

```
@inproceedings{han-etal-2020-data,
    title = "More Data, More Relations, More Context and More Openness: A Review and Outlook for Relation Extraction",
    author = "Han, Xu  and
      Gao, Tianyu  and
      Lin, Yankai  and
      Peng, Hao  and
      Yang, Yaoliang  and
      Xiao, Chaojun  and
      Liu, Zhiyuan  and
      Li, Peng  and
      Zhou, Jie  and
      Sun, Maosong",
    booktitle = "Proceedings of the 1st Conference of the Asia-Pacific Chapter of the Association for Computational Linguistics and the 10th International Joint Conference on Natural Language Processing",
    month = dec,
    year = "2020",
    address = "Suzhou, China",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2020.aacl-main.75",
    pages = "745--758",
    abstract = "Relational facts are an important component of human knowledge, which are hidden in vast amounts of text. In order to extract these facts from text, people have been working on relation extraction (RE) for years. From early pattern matching to current neural networks, existing RE methods have achieved significant progress. Yet with explosion of Web text and emergence of new relations, human knowledge is increasing drastically, and we thus require {``}more{''} from RE: a more powerful RE system that can robustly utilize more data, efficiently learn more relations, easily handle more complicated context, and flexibly generalize to more open domains. In this paper, we look back at existing RE methods, analyze key challenges we are facing nowadays, and show promising directions towards more powerful RE. We hope our view can advance this field and inspire more efforts in the community.",
}
```

### WikiFact

```
@inproceedings{goodrich2019wikifact,
author = {Goodrich, Ben and Rao, Vinay and Liu, Peter J. and Saleh, Mohammad},
title = {Assessing The Factual Accuracy of Generated Text},
year = {2019},
isbn = {9781450362016},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3292500.3330955},
doi = {10.1145/3292500.3330955},
abstract = {We propose a model-based metric to estimate the factual accuracy of generated text that is complementary to typical scori
ng schemes like ROUGE (Recall-Oriented Understudy for Gisting Evaluation) and BLEU (Bilingual Evaluation Understudy). We introduce an
d release a new large-scale dataset based on Wikipedia and Wikidata to train relation classifiers and end-to-end fact extraction mode
ls. The end-to-end models are shown to be able to extract complete sets of facts from datasets with full pages of text. We then analy
se multiple models that estimate factual accuracy on a Wikipedia text summarization task, and show their efficacy compared to ROUGE a
nd other model-free variants by conducting a human evaluation study.},
booktitle = {Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery \& Data Mining},
pages = {166–175},
numpages = {10},
keywords = {transformers, factual correctness, metric, deep learning, generative models},
location = {Anchorage, AK, USA},
series = {KDD '19}
}
```


