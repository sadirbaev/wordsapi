# About the task
Nutella is studying English. Suddenly, he wanted to search some english words. He defined some requirement for the search engine.
* Should be able to search a word with typo.
* Should be able to search a sentence with missing / additional / unordered words

**Let's create API for him!**

## Data Source
[https://s3.amazonaws.com/wordsapi/wordsapi_sample.zip](https://s3.amazonaws.com/wordsapi/wordsapi_sample.zip)
(ref: [https://www.wordsapi.com](https://www.wordsapi.com))

## Requirements
* Implement CRUD REST Api (to add more words, or more sentences) to be searchable.
* Use Elastic Search (aggressively): the sample words should be searchable, and the example of sentences too.
* Explain or find multiple ways to improve the search engine. How could you improve the scoring (implement your own algorithm, or use multiple query etc)? How to measure the scoring of the search engine?
* Discuss the link between best search engine score and performance.
* Provide a kind and simple ReadMe about the overall project(run-command, structure, approach, new idea ...).
* Submit on Github without the data source file(s).

# User's Guide
1. Install [Elastic Search (7.13)](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html), if you have already installed, skip this step.
2. Create index with name (e.g. words).
cURL PUT request to create index.
Index name is **words**. Input your Elastic search **server address(localhost)** and **portnumber(9200)**.
```
curl --location --request PUT 'localhost:9200/words?pretty'
```
3. First upload all words from wordsapi_json.json.
<br />3.1. Download the application **upload json** folder. It contains 3 files, which are executable file **upload_json**, configuration file **config.json** and sample words **wordsapi_sample.json**
<br />3.2. Update **config.json** based on your configuration. **filename** is for sample json file, **elasticsearch.host**, **elasticseach.port**, **elasticsearch.index** are for elasticsearch server address, port number and the index name which you created in the 2nd step respectively.
```
{
  "filename": "wordsapi_sample.json",
  "elasticsearch": {
    "host": "localhost",
    "port": "9200",
    "index": "words"
  }
}
```
<br />&emsp;&emsp;3.3. Once you complate configuration process, go the folder and run the binary file using following command.
```
./upload_json
```
&emsp;&emsp;**Important: make sure all three files are in same folder. This binary file was build for Linux OS**
<br />
<br />&emsp;&emsp;3.4. If uploading process finishes, just close app and you can move to the next stage.
<br />&emsp;4. Run the web server.
<br />&emsp;&emsp;4.1. Download the application **web app** folder. It contains 2 files, which are executable file **app** and configuration file **config.json**.
<br />&emsp;&emsp;4.2. Update **config.json** based on your configuration. **server.address** is for port number to run the web application, **elasticsearch.host**, **elasticseach.port**, **elasticsearch.index** are for elasticsearch server address, port number and the index name which you created in the 2nd step respectively.
```
{
  "server": {
    "address": ":8000"
  },
  "context": {
    "timeout": 2
  },
  "elasticsearch": {
    "host": "localhost",
    "port": "9200",
    "index": "words"
  }
}
```
<br />&emsp;&emsp;4.3. Once you complate configuration process, go the folder and run the binary file using following command.
```
./app
```
&emsp;&emsp;**Important: make sure two files are in same folder. This binary file was build for Linux OS**
<br />
<br />&emsp;&emsp;4.4. If web application runs successfully, it is time to use API.
<br />&emsp;5. Testing API 
<br />&emsp;&emsp;5.1. `/search` `GET` api is for searching sentence or words.
<br />&emsp;&emsp;&emsp;5.1.1. To search by word you need to pass query parameter `word`. 
```
curl --location --request GET 'localhost:8000/api/?word=like'
```
<br />&emsp;&emsp;&emsp;5.1.2. To search by sentence from examples you need to pass query parameter `sentence`. 
```
curl --location --request GET 'localhost:8000/api/?sentence=I%20like'
```
&emsp;&emsp;&emsp;**Important: %20 is for space**
<br />&emsp;&emsp;5.2. `/search` `POST` api is for adding new words with examples. To add you need to send `word` as a string and `examples` as a string array in json body.
```
curl --location --request POST 'localhost:8000/api/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "word": "dog",
    "examples": [
        "I like dogs"
    ]
}'
```

# Discussions
For developing both above-mentioned apps, I have used **Golang**. The reason is task is not so big and for uploading 8 MB json file, I thought **Go** is the best option.  
I have used [Fuzzy Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html) for searching words with typo. Fuzzy query helps even there are 
* Changing a character (box → fox
* Removing a character (black → lack)
* Inserting a character (sic → sick)
* Transposing two adjacent characters (act → cat)<br />
cURL request for fuzzy query,
```
{
  "query": {
    "fuzzy": {
      "word": {
        "value": "like",
        "fuzziness": 2
      }
    }
  }
}
```

In my case fuziness is set as 2. It returns results, even there are typo mistake with 2 random characters.<br /><br />
For searching sentences, I have used [Span Near Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-span-near-query.html). In the task, it was asked searching unordered words so I have disabled `in_order` and slop equals to number of words in the entered sentence.<br />
cURL request for span near query,
```
{
    "query": {
        "span_near": {
            "clauses": [
                {
                    "span_multi": {
                        "match": {
                            "fuzzy": {
                                "examples": {
                                    "fuzziness": "2",
                                    "value": "the"
                                }
                            }
                        }
                    }
                },
                {
                    "span_multi": {
                        "match": {
                            "fuzzy": {
                                "examples": {
                                    "fuzziness": "2",
                                    "value": "lynched"
                                }
                            }
                        }
                    }
                }                   
            ],
            "slop": 12,
            "in_order": "false"
        }
    }
}
```
We can improve the app with searching words and sentences with one endpoint with join multiple queries. If we set less fuzziness to both **Fuzzy Query** and **Span Near Query** we get better score. However type suggestion may work unexpectively. Or for typo we can use [N-gram tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html) and we can implement some [Suggesters](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html) and so on.<br /><br />
I did not exactly understand what did they mean with **Discuss the link between best search engine score and performance.**. But performance and score are inversely proportional. For example in our case if we set laziness more we can get more result, but it affects neagtively to the server's performance. 

# Summary
- [x] Implement CRUD REST Api (to add more words, or more sentences) to be searchable.
- [x] Use Elastic Search (aggressively): the sample words should be searchable, and the example of sentences too.
- [x] Explain or find multiple ways to improve the search engine. How could you improve the scoring (implement your own algorithm, or use multiple query etc)? How to measure the scoring of the search engine?
- [x] Discuss the link between best search engine score and performance.
- [x] Provide a kind and simple ReadMe about the overall project(run-command, structure, approach, new idea ...).
- [x] Submit on Github without the data source file(s).
