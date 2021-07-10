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


# Solution
I started parsing the given sample JSON file and inserting the words to the Elastic search. What did I do?
1. I have installed [Elastic Search (7.13)](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html) on Ubuntu 20.04. **To test the application you also need to install Elastic Search on your machine.**
2. I have created index to store Document, which includes word and examples. **To test the application you also need to create index** 
#### cURL PUT request to create index.
Index name is **words**. Input your Elastic search **server address(localhost)** and **portnumber(9200)**.
```
curl --location --request PUT 'localhost:9200/words?pretty'
```
