---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.15.2
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

```python
import requests
import pandas as pd
import json
import configparser
```
```python
config = configparser.ConfigParser()
config.read('sports-research.cfg')
api_key = config['Football']['sportsdataio']
```
```python
endpoint = "https://api.sportsdata.io/api/nfl/fantasy/json/PlayerSeasonStats/2024"

headers = {
    'Ocp-Apim-Subscription-Key': api_key
}

response = requests.get(endpoint, headers=headers)
data = response.json()

with open('./football/data/2024/PlayerSeasonStats.json', 'w+') as f:
    json.dump(data, f)
```
```python
pd.read_json('./football/data/2024/PlayerSeasonStats.json')
```
