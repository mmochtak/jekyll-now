---
published: true
layout: post
title: 'SentenceR: (Almost) Language-Agnostic Sentence Tokenization Using R'
---

<img src="/images/logo_sentencer.png" style="max-width:100%;" height="139" align="right">
**sentenceR** is a language-agnostic utility designed for sentence tokenization of raw text. Using the UDPipe POS tagging pipeline, the package automatically extracts sentences with their appropriate indexes (hence the “crowbar” logo as a reference to extraction). The package works with any of the 100+ language models natively provided by UDPipe package.

## Overview
Working on languages other than English is for many programmers interested in NLP a real obstacle requiring investing a lot of precious time in developing language-specific pre-processing tools. This process of “reinventing wheel” is necessary especially when it comes to applications that need to be fast, efficient, and accurate. For many projects, however, this initial investment of time and resources does not make sense, especially if it comes to testing and development. SentenceR is a package that focuses on streamlining at least part of this hassle when it comes to sentence tokenization and lemmatization. The motivation for putting together this package has come from my own experience while working on text processing of textual data from low-resourced languages. Although resources available for R users working on textual data have become very popular in the recent years, their primary focus still remains on English with other languages being far behind in terms of development and implementation.   

The package is intended for use especially with non-English languages that are under-resourced in terms of standardized tools. The approach is not particularly fast as the pipeline relies on full POS tagging done by UDPipe package but it provides a reliable option for users working on small corpora of various origins (typically social scientists) who need a simple pre-processing tool for extracting sentence-level tokens and their higher n-grams. For the convenience, the main function provides several control arguments for cleaning the raw text as well as extracting its lemmatized form for further processing.

The package has been developed as a practice for me as I wanted to learn how to create packages to better organize my code. However, it might be useful to anybody working on NLP tasks requiring sentence tokenization and lemmatization in languages other than English without the complexity found in other, more advanced solutions. As I already mentioned, the approach is not the most efficient in terms of speed and accuracy but it is very versatile when it comes to languages that can be processed out of the box. Regarding accuracy, it is important to stress that sentence tokenization is based on accuracy of language models and may differ among models. Although I’ve never planned to submit it to CRAN, I am planning to maintain it as long as it is still useful via [GitHub]( https://github.com/mmochtak/sentenceR).

## Installation Instructions and Usage
The package can be simply installed from GitHub using *devtools*. All missing dependencies will be installed together with it.

~~~
devtools::install_github('mmochtak/sentenceR')
~~~

The package contains three general functions: `get_sentences()`; `sent_ngrams()`;  and `sent_ngrams_lem()`. 

`get_sentences()` is a general function for extraction sentences from raw text. The main input is a string vector. The only other required argument is the language model (here, for simplicity and clarity for broader audience, “english”; full list of available models can be found in [UDPipe repository]( https://github.com/bnosac/udpipe)). Additionally, if needed the output can be further cleaned off of numbers (add argument *remove_no = TRUE*) and punctuation (add argument *remove_punct = TRUE*), all characters can be transformed to lower cases (add argument *tolower = TRUE*), as well as the outcome sentence can be accompanied by its lemmatized version (add argument *lem = TRUE*; see example below). By default, all available cores are used for the processing.

~~~
library(sentenceR)
sample_text <- c("This is sentence number one. This is sentence number two. This is sentence number three.", "This is sentence number four. This is sentence number five. This is sentence number six.")
get_sentences(text = sample_text, language = "english", lem = TRUE)

  doc_id paragraph_id sentence_id                       sentence                   sentence_lem
1      1            1           1   This is sentence number one.   this be sentence number one.
2      1            1           2   This is sentence number two.   this be sentence number two.
3      1            1           3 This is sentence number three. this be sentence number three.
4      2            1           1  This is sentence number four.  this be sentence number four.
5      2            1           2  This is sentence number five.  this be sentence number five.
6      2            1           3   This is sentence number six.   this be sentence number six.
~~~

`sent_ngrams()` is a simple function that takes the result of `get_sentences()` (a data frame of class `sentenceR_df`) and creates higher sentence n-grams based on a specified *n*.

~~~
library(sentenceR)
sample_text <- c("This is sentence number one. This is sentence number two. This is sentence number three.", "This is sentence number four. This is sentence number five. This is sentence number six.")
result <- get_sentences(text = sample_text, language = "english", lem = TRUE)
sent_ngrams(sentences = result, n = 2)

  doc_id ngram_id                                                       ngram
1      1        1   This is sentence number one. This is sentence number two.
2      1        2 This is sentence number two. This is sentence number three.
3      2        1 This is sentence number four. This is sentence number five.
4      2        2  This is sentence number five. This is sentence number six.
~~~

`sent_ngrams_lem()` is an equivalent to `sent_ngrams()` but instead of regular sentences their lemmatized version are taken as an input (column sentence_lem must exist in a data frame of class `sentenceR_df`).

~~~
library(sentenceR)
sample_text <- c("This is sentence number one. This is sentence number two. This is sentence number three.", "This is sentence number four. This is sentence number five. This is sentence number six.")
result <- get_sentences(text = sample_text, language = "english", lem = TRUE)
sent_ngrams_lem(sentences = result, n = 2)

  doc_id ngram_id                                                       ngram
1      1        1   this be sentence number one. this be sentence number two.
2      1        2 this be sentence number two. this be sentence number three.
3      2        1 this be sentence number four. this be sentence number five.
4      2        2  this be sentence number five. this be sentence number six.
~~~

## Application on low-resourced language: *Czech*
Previous section has demonstrated the application of package on data that are general enough for broad audience. However, the real focus of **sentenceR** package is on low-resourced languages. Below you can find the application on Czech using a dummy text from Czech Wikipedia.

~~~
library(sentenceR)
Sys.setlocale(locale = "Czech_Czechia.1250") # Do not forget to set proper locale to avoid any problems with encoding (here is Czech_Czechia.1250 for Czech language)
sample_text <- enc2utf8("Česko, úředním názvem Česká republika, je stát ve střední Evropě. Samostatnost nabylo 1. ledna 1993 jako nástupnický stát Československa, předtím existovalo jako jedna ze dvou republik československé federace. Navazuje také na více než tisícileté dějiny české státnosti a kultury. Podle své ústavy je Česko parlamentní, demokratický právní stát s liberálním státním režimem a politickým systémem založeným na svobodné soutěži politických stran a hnutí. Hlavou státu je prezident republiky, vrcholným a jediným zákonodárným orgánem je dvoukomorový Parlament České republiky, na vrcholu moci výkonné stojí vláda České republiky.") # for testing purpose, the string with accented characters needs to have utf8 encoding (declaring strings like in this example returns encoding as “unknown”)
get_sentences(text = sample_text, language = "czech", lem = TRUE)

  doc_id paragraph_id sentence_id sentence                           sentence_lem                        
1 1                 1           1 Česko, úředním názvem Česká repub~ česko, úřední název český republika~
2 1                 1           2 Samostatnost nabylo 1. ledna 1993~ samostatnost nabýt 1. leden 1993 ja~
3 1                 1           3 Navazuje také na více než tisícil~ navazovat také na hodně než tisícil~
4 1                 1           4 Podle své ústavy je Česko parlame~ podle svůj ústav být Česko parlamen~
5 1                 1           5 Hlavou státu je prezident republi~ hlava stát být prezident republik, ~

result <- get_sentences(text = sample_text, language = "czech", lem = TRUE)
sent_ngrams(sentences = result, n = 2)

  doc_id ngram_id ngram                                                                                  
1 1      1        Česko, úředním názvem Česká republika, je stát ve střední Evropě. Samostatnost nabylo ~
2 1      2        Samostatnost nabylo 1. ledna 1993 jako nástupnický stát Československa, předtím existo~
3 1      3        Navazuje také na více než tisícileté dějiny české státnosti a kultury. Podle své ústav~
4 1      4        Podle své ústavy je Česko parlamentní, demokratický právní stát s liberálním státním r~

sent_ngrams_lem(sentences = result, n = 2)

  doc_id ngram_id ngram                                                                                  
1 1      1        česko, úřední název český republika, být stát v střední Evropa. samostatnost nabýt 1. ~
2 1      2        samostatnost nabýt 1. leden 1993 jako nástupnický stát Československo, předtím existov~
3 1      3        navazovat také na hodně než tisíciletý dějiny český státnost a kultura. podle svůj úst~
4 1      4        podle svůj ústav být Česko parlamentní, demokratický právní stát s liberální státní re~
~~~

## Final Remarks
The package is still under development (current version is 0.0.1) so feel free to report any bugs or weird behavior. Feel free to contact me if standard GitHub channels are not suitable for you via my [personal website](https://mochtak.com). If used in your research, please cite it as:

> Mochtak, Michal (2021): sentenceR: Language-Agnostic Sentence Tokenization for Low-Resourced Languages. URL: https://github.com/mmochtak/sentenceR/
