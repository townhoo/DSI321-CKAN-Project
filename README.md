# Demo cronjob k8s
0. make docker image
 - edit python code and build to registry
 - edit conjob.yaml

1. download cronjob.yml
  - download
```
wget https://raw.githubusercontent.com/wasit7/dsi321_2023/main/week13/conjob.yaml
```
2. apply cronjob
```
kubectl --kubeconfig=kubeconfig.yaml apply -f conjob.yaml -n demo
```
3. check job 
```
kubectl --kubeconfig=kubeconfig.yaml get cronjobs -n demo
```
4. check pod of job
```
kubectl --kubeconfig=kubeconfig.yaml get pods -n demo
```
***

# ETL Real time Data and send to CKAN with K8S
- This project demonstates all processes about get data, ETL data, store data in CKAN.
- By using K8S, you will also learn how to use docker, docker hub and K8S with Cronjob 
- For easy to practice, all processes are divided into 2 big steps : coding  part and deployment part

# Coding part
In this part, the main objective is coding in python, jupyter notebook and use .env to store some secret parameters. When you run coding, you can run coding with Jupyter Notebook or Python script as usual. At these steps, no need of docker, docker hub, or K8S. Try to follow these step if you don't know where to start
1. Setting up
1. Coding and Test ETL on Jupyter Notebook
1. Coding and Test CKAN on Jupyter Notebook
1. Integrate both ETL and CKAN in scripy.py


## Setting Up
We try to set up all dependencies with requirement.txt and install them for your environment. Jupyter and python script require these dependencies. In addition, metadata.json needs some config for CKAN.

* Create requirement.txt, tell dependencies you need (docker file will read this file later)
```
pandas
requests
lxml
html5lib
beautifulsoup4
```
* Install dependencies for Jupyter
```bash
pip install -r requirement.txt 
```

* Create metadata.json  (see detail in link) : you have to modified information corresponding to your information - warning : name should be unique small letter and title should be big letter  
https://github.com/wasit7/dsi321_2023/blob/main/week13/metadata.json
```bash
# bash & wsl
wget https://raw.githubusercontent.com/wasit7/dsi321_2023/main/week13/metadata.json
```
* Edit dataset name and title to match your dataset and contain your team_id
```json
    "name": "thailand_gdp03",   # must change this line to weather_wsl01
    "title": "THAILAND_GDP_03", # must change this line to weather_wsl01
```

* Create .env (if you need)
```bash
# bash & WSL
wget https://github.com/wasit7/dsi321_2023/blob/main/week13/.env-template
cp .env-template .env      #ubuntu / mac
```

* Specify TOKEN , CKAN_URL, WEB_SCRIPY URL in .env  (if you need)

```bash
# .env
TOKEN=xxxxxxxxxxx   # ใส่ TOKEN (permission to access ckan, see in code week 12)
CKAN_URL=https://ckan.data.storemesh.com
WEB_SCRIPY=yyyyyyyyy # ใส่ URL web สำหรับ scrap (google sheet link)
```

## Data Source
* Weather Data Source : https://docs.google.com/spreadsheets/d/e/2PACX-1vQlEs3FxFPwm-dpvU1YdsfRgsbfT9WdiXJHZm9kJgGTziPnk-y3TWtftbSbxj6Fe_g0NxYgqyVHTVU5/pubhtml?gid=1397577608&amp;single=true&amp;widget=true&amp;headers=false
* Pandas can load HTML table tag with single function
```python
import pandas as pd
# from URL
dfs = pd.read_html("https://docs.google.com/spreadsheets/d/e/2PACX-1vQlEs3FxFPwm-dpvU1YdsfRgsbfT9WdiXJHZm9kJgGTziPnk-y3TWtftbSbxj6Fe_g0NxYgqyVHTVU5/pubhtml?gid=1397577608&amp;single=true&amp;widget=true&amp;headers=false")
```


## Data Preparation and Metadata in Jupyter

* Create your new Jupyter notebook file 
* Load data source from google sheet link above
* Use pandas to ETL data and transform into appropriate structure and export to df.CSV

## Send Data to CKAN Dataset in Jupyter

* This set step require metadata.json and function to send 
* See detail how to do in week 12

### Create dataset only one time
* You create dataset once, may be do it in Jupyter

### Upload data
* You can upload as many times as you need

## Integrate everything in scripy.py
* use scripy.py from template
```bash
wget https://raw.githubusercontent.com/wasit7/dsi321_2023/main/week13/scripy.py
```
* modify your ETL in scripy.py
* modify your ckan config in scripy.py
* modify your metadata.json
* Create Dockerfile
```Dockerfile
FROM python:3.8
WORKDIR /code
COPY . .
COPY requirement.txt  requirement.txt
RUN pip install -r /code/requirement.txt
# เหมือนจะขาดไปหนึ่งบรรทัด
```
* run with docker
```bash
docker build -t simple-scripy:latest .    # don't forget last dot which means the current directory
docker run simple-scripy:latest     # run without .env
docker run --env-file=.env simple-scripy:latest    # run with .env file
```
### load json file
- Your json file contains English only
```python
import json
ckan_meta = json.load(open('test.json'))
```
- Your json file contains utf8 character like Thai
```python
import json
ckan_meta = json.load(open('metadata.json', encoding="utf-8"))
```
### load value from .env
- if you change environment or machine, the correct configuration may be changed by admin. 
```python
# use os to get some values from env, getenv has 1 or 2 parameters : the first is key of value and the second is the default value in case value does not exist.
url_ckan = os.getenv("CKAN_URL") 
url_ckan = os.getenv("CKAN_URL","https://ckan.data.storemesh.com" ) 
```

At this step you should send your data to CKAN with no cronjob. Without cronjob you still send data to CKAN by manaul. With cronjob, it is more easier and more accurate to process routine tasks by programming. Cronjob is an operation that a part of OS. Linux called "crontab" while Windows called "task schedule".

# Deployment
Deployment is just like throwing your coding to process in another machine that operate while your laptap can shutdown. If you processes some instructions with cronjob, it allow you to run some processes more than 1 time on some schedule. 

## Push your image to Docker hub
* You must have docker hub account / activated with email
* Try to publish your image to docker hub. When building Docker image file, you must name image file as
```bash
docker login
# docker build -t <userName>/<repoName>:<tagName> .
docker build -t arc6828/simple-scripy:latest .    # don't forget last dot which means the current directory
docker push arc6828/simple-scripy    #
```

## Setting up for K8S Deployment file (.yaml)

* Create conjob.yaml (see details in the following link)
https://github.com/wasit7/dsi321_2023/blob/main/week13/conjob.yaml
```
# bash & wsl
wget https://raw.githubusercontent.com/wasit7/dsi321_2023/main/week13/conjob.yaml
```
Please change the "name" to your team_id 
```yml
name: scripy-web-wsl01 # change team-id to your team e.g. wsl01

```
* Modify schedule cronjob to every 6 hours
* Warning : this step requires publish your docker image to docker hub 
```yaml
schedule: "* * * * *"   # modify to every 6 hours
image: kran13200/simple-scripy:latest # use your docker hub image
```


## Kubernates CronJob
* download kubeconfig.yaml file by commmand provide in google classroom
* modified cronjob in conjob.yaml
* deploy to kubernetes server see detail in week 9 or see Demo cronjob k8s section above

# Data Visualization and Interpretation
* use you favorite tool to create a meaningful table or chart. Then use it in report to tell thesory and facts those you have discovered.

# Project References
* https://www.iqair.com/th/air-pollution-data-api
* [Air Quality Index (AQI)](https://en.wikipedia.org/wiki/Air_quality_index)
* [API Reference](https://api-docs.iqair.com/)
* [Crontab Editor](https://crontab.guru/#*_*/6_*_*_*)
* [Weather Data Source](https://docs.google.com/spreadsheets/d/e/2PACX-1vQlEs3FxFPwm-dpvU1YdsfRgsbfT9WdiXJHZm9kJgGTziPnk-y3TWtftbSbxj6Fe_g0NxYgqyVHTVU5/pubhtml?gid=1397577608&amp;single=true&amp;widget=true&amp;headers=false)
