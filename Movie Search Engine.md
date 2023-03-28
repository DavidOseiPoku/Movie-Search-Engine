## 1. Tasks and Steps




## 1.1 WHY:
The project aims to build a customised search engine for movies and TV shows using ElasticSearch and related tools. We will use the IMDB dataset of the top 1000 movies and TV shows from Kaggle as our data source.
The search engine will enable users to search for movies by title, genre, year of release, actors, directors, and other relevant information.




## 1.2 WHAT:
The Domain for our search engine is Movies.




The Dataset houses 1000 movies of various genre from 1920 - 2020




Link to Data Source: https://www.kaggle.com/datasets/harshitshankhdhar/imdb-dataset-of-top-1000-movies-and-tv-shows




Data Field Description:
Poster_Link: Link of the poster that imdb using
Series_Title: Name of the movie
Released_Year: Year at which that movie released
Certificate: Certificate earned by that movie
Runtime: Total runtime of the movie
Genre: Genre of the movie
IMDB_Rating: Rating of the movie at IMDB site
Overview: mini story/ summary
Meta_score: Score earned by the movie
Director: Name of the Director
Star1,Star2,Star3,Star4: Name of the Stars
No_of_votes: Total number of votes
Gross: Money earned by that movie


Example Data:
```json
{
  "Poster_Link":"https://m.media-amazon.com/images/M/MV5BMDFkYTc0MGEtZmNhMC00ZDIzLWFmNTEtODM1ZmRlYWMwMWFmXkEyXkFqcGdeQXVyMTMxODk2OTU@._V1_UX67_CR0,0,67,98_AL_.jpg",
  "Series_Title":"The Shawshank Redemption",
   "Released_Year":1994,
   "Certificate": "A",
   "Runtime": "142 min",
   "Genre": "Drama",
   "IMDB_Rating": 9.3,
   "Overview": "Two imprisoned men bond over a number of years, finding solace and eventual redemption through acts of common decency.",
   "Meta_score": "80",
   "Director": "Frank Darabont",
   "Star1": "Tim Robbins",
   "Star2" : "Morgan Freeman",
   "Star3" : "Bob Gunton",
   "Star4" : "William Sadler",
   "No_of_Votes" : 2343110,
  "Gross":28341469
}
```




## 1.3 WHO:


## 1.3.1 Information Needs and Identification of Use Cases


## Use Case 1:
Keywords: ```Tom Hardy```
Brief description of information need: ```To find movies with Tom Hardy that earned more than $100 million```


## Use Case 2:
Keywords: ```crime war revolution```
Brief description of information need: ```To find the movies about crime, war and revolutions```


## Use Case 3:
Query Keywords: ```Quentin Tarantino```
Brief description of information need: ```Highly rated movies directed by Quentin Tarantino with more than a million votes```


## 1.4 HOW:
## Setup for Custom Similarity:


```Json
PUT /do447_info624_202202_imdb
{
"settings": {
 "index": {
   "similarity": {
     "my_bm25": {
       "type": "BM25",
       "k1": 2.0,
       "b":1.0
     },
     "my_dfr": {
       "type": "DFR",
       "basic_model": "g",
       "after_effect": "l",
       "normalization": "h2",
       "normalization.h2.c": "3.0"
     }
   }
 }
}
}
```
Impact:
For "my_bm25" similarity: The "K1" and "b" parameters were increased to 2.0 and 1.0 respectively meaning a higher value of "k1" will increase the impact of term frequency on the score, making documents with more occurrences of the search terms more relevant. A higher value of "b" will increase the impact of document length on the score, making longer documents more relevant.


For "my_dfr" similarity:
- The default type "DFR" was used which means relevant documents should be more informative than non-relevant documents, and that different parts of a document may be more or less informative depending on the query.
- The "basic_model": "g" means that the occurrences of the query term in the document follow a geometric distribution.
- The "after_effect": "l" means that the information gain should be smoothed to avoid zero term frequencies.
- The "normalization": "h2" This parameter controls the "saturation point" of the normalization function, which is a way to adjust the score of a document based on its length.
- The "normalization.h2.c": "3.0"  means that the function saturates at a document length that is three times the average length of documents in the index.




## Mapping:
```Json
PUT /do447_info624_202202_imdb/_mapping
{
 "properties" : {
   "Poster_Link": {
     "type":"keyword"
   },
   "Series_Title": {
     "type":"text",
     "analyzer": "english",
     "similarity" : "my_dfr"
   },
   "Released_Year" : {
     "type" : "integer"
   },
   "Certificate" : {
     "type" : "keyword"
   },
   "Runtime": {
     "type": "text",
     "analyzer": "english"
   },
   "Genre": {
     "type": "text",
     "analyzer": "english"
   },
   "IMDB_Rating": {
     "type" : "rank_feature",
     "positive_score_impact": true
   },
   "Overview": {
     "type":"text",
     "analyzer": "english",
     "similarity" : "my_bm25"
   },
   "Meta_score" : {
     "type" : "rank_feature",
     "positive_score_impact": true
   },
   "Director" : {
     "type" : "text",
     "analyzer": "english"
   },
   "Star1": {
     "type" : "text",
     "analyzer": "english",
     "similarity" : "boolean"
   },
   "Star2": {
     "type" : "text",
     "analyzer": "english",
     "similarity" : "boolean"
   },
   "Star3": {
     "type" : "text",
     "analyzer": "english",
     "similarity" : "boolean"
   },
   "Star4": {
     "type" : "text",
     "analyzer": "english",
     "similarity" : "boolean"
   },
   "No_of_Votes": {
     "type" : "rank_feature",
     "positive_score_impact": true
   },
   "Gross": {
     "type" : "rank_feature",
     "positive_score_impact": true
   }
 }
}
```


Table showing Data Field, data types, analyzers and Rationale:


| Data Field    |  Data Type   | Analyzer       |                                               Rationale                                                          |
|---------------|--------------|----------------|------------------------------------------------------------------------------------------------------------------|
| Poster_Link   |  Keyword     |                |                                                                                                                  |
| Series_Title  |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| Runtime       |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| Genre         |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| IMDB_Rating   | rank_feature |                |Used to positively influence the relevance score of the documents.                                                |
| Overview      |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| Meta_Score    |  rank_feature|                |Used to positively influence the relevance score of the documents.                                                |
| Director      |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| Stars         |  text        | english        |Analyzer is designed for text and will tokenize them in a way that allows for accurate search and retrieval       |
| No_of_Votes   | rank_feature |                |Used to positively influence the relevance score of the documents.                                                |
| Gross         | rank_feature |                |Used to positively influence the relevance score of the documents.                                                |




## Description of search queries in terms of the use cases (needs) And Evaluation:


Use Case 1:
Query Keywords: ```Tom Hardy```
Brief description of information need: ```To find high grossing movies with Tom Hardy```


```json
GET /do447_info624_202202_imdb/_search
{"from": 0, "size": 5,
"query": {
  "bool": {
    "must": [
      {
        "multi_match": {
          "query": "Tom Hardy",
          "fields": ["Star1", "Star2", "Star3", "Star4"]
        }
      },
      {
        "bool": {
          "should":
            {
      "rank_feature": {
        "field": "Gross",
        "saturation": {
          "pivot": 1000000
        }
      }
    }
        }
      }
    ]
  }
}
}
```


```Results```
| Doc ID |    Series_Title      |    Gross  |Relevant?  |Relevance Score|
|--------|----------------------|-----------|-----------|---------------|
|   65   |The Dark Knight Rises |448,139,099|     1     |       3       |
|   575  |Dunkirk               |188,373,161|     1     |       2       |
|   345  |The Revenant          |183,637,894|     1     |       2       |
|   225  |Mad Max: Fury Road    |154,058,340|     1     |       2       |
|   146  |Warrior               | 13,657,115|     0     |       0       |
```Precision = 4/4+1 = ⅘ = 0.8```
```DCG(5) = 3 + 2/log(2) + 2/log(3) + 2/log(4) + 0/log(5) = 1 + 1/1 + 1/1.58 + ½ = 3.13 ```
```IDCG(5) = 3 + 2/1 + 2/1.58 + 2/2 = 3.13```
```nDCG(5) = DCG(5)/IDCG(5) = 3.13/3.13 = 1.00```






Use Case 2:
Query Keywords: ```crime war revolution```
Information Need: ```To find the movies about crime, war and revolutions```


```json
GET /do447_info624_202202_imdb/_search
{
"from" : 0, "size" : 5,
"query": {
  "multi_match" : {
    "query":    "crime war revolution",
    "fields": ["Genre", "Overview", "Series_Title"]
  }
}
}
```
```Results```
| Doc ID |    Series_Title      |           Genre             | Relevant? | Relevance Score |
|--------|----------------------|-----------------------------|-----------|-----------------|
|   184  |Judgment at Nuremberg |          Drama, War         |     1     |        3        |
|   360  |Persepolis            | Animation, Biography, Drama |     1     |        2        |
|   432  |Doctor Zhivago        |      Drama, Romance, War    |     1     |        2        |
|   35   |Joker                 |    Crime, Drama, Thriller   |     1     |        3        |
|   829  |Miller's Crossing     |    Crime, Drama, Thriller   |     1     |        3        |
```Precision = 5/5+0 = 5/5 = 1.0```
```DCG(5) = 3 + 2/1 + 2/1.58 + 3/2 + 3/2.32 = 9.05```
```IDCG(5) = 3 + 3/1 + 3/1.58 + 2/2 + 2/2.32 = 9.75```
```nDCG(5) = DCG(5)/IDCG(5) = 9.05/9.75 = 0.93```




Use Case 3:
Query Keywords: ```Quentin Tarantino```
Information Need: ```Highly rated movies directed by Quentin Tarantino with more than a million votes```


```json
GET /do447_info624_202202_imdb/_search
{ "from": 0, "size": 5,
"query": {
  "bool": {
    "must": [
      {
        "match": {
          "Director": "Quentin Tarantino"
        }
      }
    ],
    "should": [{
      "rank_feature": {
        "field": "IMDB_Rating",
        "saturation": {
          "pivot": 0.1
        }
      }
    },
    {
      "rank_feature": {
        "field": "Meta_Score",
        "saturation": {
          "pivot": 1
        }
      }
    },
    {
      "rank_feature": {
        "field": "No_of Votes",
        "saturation": {
          "pivot": 100
        }
      }
    }]
  }
}
}
```
```Results```
|Doc ID|  Series_Title      | IMDB_Rating | Meta_score | No_of_Votes | Relevant? | Relevance Score |
|------|--------------------|-------------|------------|-------------|-----------|-----------------|
|   8  |Pulp Fiction        |     8.9     |     94     | 1,826,188   |     1     |        3        |       
|   64 |Django Unchained    |     8.4     |     81     | 1,357,682   |     1     |        3        |
|   95 |Inglourious Basterds|     8.3     |     69     | 1,267,869   |     1     |        2        |
|   105|Reservoir Dogs      |     8.9     |     79     | 918,562     |     0     |        0        |
|   243|Kill Bill: Vol. 1   |     8.1     |     69     | 1,000,639   |     1     |        2        |
```Precision = 4/4+1 = ⅘ = 0.8```
```DCG(5) = 3 + 3/1 + 2/1.58 + 0/2 + 2/2.32 = 8.13```
```IDCG(5) = 3 + 3/1 + 2/1.58 + 2/2 + 0/2.32 = 8.26```
```nDCG(5) = DCG(5)/IDCG(5) = 8.13/8.26 = 0.98```


Interpretation of Results:


Precision:
This is the Fraction of retrieved documents that are relevant. P = tp (true positive) /(tp (true positive) + fp (false positive)
For Use case 1: our precision is 0.8
For Use Case 2: Our precision is 1.0
For Use case 3: Our precision is 0.8


nDCG:
The nDCG (Normalised Discounted Cumulative Gain) values range between 0 and 1, where 1 represents a perfect ranking of the search results and 0 represents the worst ranking.


For Use case 1: Our nDCG is 1.00 which means all the top search results are highly relevant and the ranking is perfect.
For Use case 2: Our nDCG is 0.93 which means the search engine has returned a very good set of highly relevant search results at the top of the ranking.
For Use case 3: Our nDCG is 0.98 which means the search engine has returned a very good set of highly relevant search results at the top of the ranking.




## WHERE:
The data was uploaded from a csv file into Kabina on our individual system. It is located in our indices below:
David: do447_info624_202202_imdb
Christian: cce49_info624_202202_imdb
Manisha: mj844_info624_202202_imdb


## Experience:
+ For Use case 3, we could have boosted the number of vote field in our scoring to get a more accurate ranking in our resulting
+ As a team, we learnt how to build and evaluate a search engine based on various parameters and how to manipulate those parameters to get more accurate information from our search engine.
+ For can use more in-depth information-need or use cases and better evaluation methods for Large datasets.
+ We changed the Gross column in the original file to general to remove the commas (',') so we could import the file into kibana as an integer value.
+ The Released_Year value for the movie Apollo was PG which gave an error when importing so we got the year the movie was released which was 1995 and edited it in the original file before importing it into Kibana.
+ Due to the large size of the data and high number hits, we excluded recall and F1 score as part of our evaluation because it will be too tasking to evaluate the number of documents that were not retrieved to know if they were relevant or not.







