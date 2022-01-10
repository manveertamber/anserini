# Anserini: Regressions for [DL20 (Doc)](https://trec.nist.gov/data/deep2020.html) w/ docTTTTTquery

This page describes experiments, integrated into Anserini's regression testing framework, for the TREC 2020 Deep Learning Track (Document Ranking Task) on the MS MARCO document collection using relevance judgments from NIST.

Note that the NIST relevance judgments provide far more relevant documents per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).
For additional instructions on working with MS MARCO document collection, refer to [this page](experiments-msmarco-doc.md).

Note that there are four different regression conditions for this task, and this page describes the following:

+ **Indexing Condition:** each MS MARCO document is treated as a unit of indexing
+ **Expansion Condition:** doc2query-T5

All four conditions are described in detail [here](https://github.com/castorini/docTTTTTquery), in the context of doc2query-T5.

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/dl20-doc-docTTTTTquery.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/dl20-doc-docTTTTTquery.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

Note that in November 2021 we discovered issues in our regression tests, documented [here](experiments-msmarco-doc-doc2query-details.md).
As a result, we have had to rebuild all our regressions from the raw corpus.
These new versions yield end-to-end scores that are slightly different, so if numbers reported in a paper do not exactly match the numbers here, this may be the reason.

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection JsonCollection \
  -input /path/to/msmarco-doc-docTTTTTquery \
  -index indexes/lucene-index.msmarco-doc-docTTTTTquery/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 7 -storePositions -storeDocvectors -storeRaw \
  >& logs/log.msmarco-doc-docTTTTTquery &
```

The directory `/path/to/msmarco-doc-docTTTTTquery/` should be a directory containing the expanded document corpus in Anserini's jsonl format.
See [this page](experiments-msmarco-doc-doc2query-details.md) for how to prepare the corpus.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/).
The regression experiments here evaluate on the 45 topics for which NIST has provided judgments as part of the TREC 2020 Deep Learning Track.
The original data can be found [here](https://trec.nist.gov/data/deep2020.html).

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-docTTTTTquery/ \
  -topics src/main/resources/topics-and-qrels/topics.dl20.txt -topicreader TsvInt \
  -output runs/run.msmarco-doc-docTTTTTquery.bm25-default.topics.dl20.txt \
  -bm25 -hits 100 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-docTTTTTquery/ \
  -topics src/main/resources/topics-and-qrels/topics.dl20.txt -topicreader TsvInt \
  -output runs/run.msmarco-doc-docTTTTTquery.bm25-default+rm3.topics.dl20.txt \
  -bm25 -rm3 -hits 100 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-docTTTTTquery/ \
  -topics src/main/resources/topics-and-qrels/topics.dl20.txt -topicreader TsvInt \
  -output runs/run.msmarco-doc-docTTTTTquery.bm25-tuned.topics.dl20.txt \
  -bm25 -bm25.k1 4.68 -bm25.b 0.87 -hits 100 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-docTTTTTquery/ \
  -topics src/main/resources/topics-and-qrels/topics.dl20.txt -topicreader TsvInt \
  -output runs/run.msmarco-doc-docTTTTTquery.bm25-tuned+rm3.topics.dl20.txt \
  -bm25 -bm25.k1 4.68 -bm25.b 0.87 -rm3 -hits 100 &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -c -m map -c -m ndcg_cut.10 -c -m recip_rank -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl20-doc.txt runs/run.msmarco-doc-docTTTTTquery.bm25-default.topics.dl20.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -m map -c -m ndcg_cut.10 -c -m recip_rank -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl20-doc.txt runs/run.msmarco-doc-docTTTTTquery.bm25-default+rm3.topics.dl20.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -m map -c -m ndcg_cut.10 -c -m recip_rank -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl20-doc.txt runs/run.msmarco-doc-docTTTTTquery.bm25-tuned.topics.dl20.txt

tools/eval/trec_eval.9.0.4/trec_eval -c -m map -c -m ndcg_cut.10 -c -m recip_rank -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl20-doc.txt runs/run.msmarco-doc-docTTTTTquery.bm25-tuned+rm3.topics.dl20.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

MAP                                     | BM25 (default)| +RM3      | BM25 (tuned)| +RM3      |
:---------------------------------------|-----------|-----------|-----------|-----------|
[DL20 (Doc)](https://trec.nist.gov/data/deep2020.html)| 0.4230    | 0.4229    | 0.4099    | 0.4104    |


nDCG@10                                 | BM25 (default)| +RM3      | BM25 (tuned)| +RM3      |
:---------------------------------------|-----------|-----------|-----------|-----------|
[DL20 (Doc)](https://trec.nist.gov/data/deep2020.html)| 0.5885    | 0.5407    | 0.5852    | 0.5743    |


MRR                                     | BM25 (default)| +RM3      | BM25 (tuned)| +RM3      |
:---------------------------------------|-----------|-----------|-----------|-----------|
[DL20 (Doc)](https://trec.nist.gov/data/deep2020.html)| 0.9369    | 0.8147    | 0.9439    | 0.8701    |


R@100                                   | BM25 (default)| +RM3      | BM25 (tuned)| +RM3      |
:---------------------------------------|-----------|-----------|-----------|-----------|
[DL20 (Doc)](https://trec.nist.gov/data/deep2020.html)| 0.6414    | 0.6555    | 0.6178    | 0.6127    |

Explanation of settings:

+ The setting "default" refers the default BM25 settings of `k1=0.9`, `b=0.4`.
+ The setting "tuned" refers to `k1=4.68`, `b=0.87`, tuned using the MS MARCO document sparse judgments to optimize for recall@100 (i.e., for first-stage retrieval) on 2019/12.

Settings tuned on the MS MARCO document sparse judgments _may not_ work well on the TREC dense judgments.

Note that retrieval metrics are computed to depth 100 hits per query (as opposed to 1000 hits per query for DL20 passage ranking).
Also, remember that we keep qrels of _all_ relevance grades, unlike the case for DL20 passage ranking, where relevance grade 1 needs to be discarded when computing certain metrics.

Some of these regressions correspond to official TREC 2020 Deep Learning Track submissions by team `anserini`:

+ `d_d2q_bm25` = BM25 (default), `k1=0.9`, `b=0.4`
+ `d_d2q_bm25rm3` = BM25 (default) + RM3, `k1=0.9`, `b=0.4`