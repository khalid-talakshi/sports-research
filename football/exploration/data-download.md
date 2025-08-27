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

with open('./football/data/2024/PlayerSeasonStatsRaw.json', 'w+') as f:
    json.dump(data, f)
```
```python
df = pd.read_json('./football/data/2024/PlayerSeasonStatsRaw.json')
df.columns 
```
```python
identity_cols = ['PlayerID', 'SeasonType', 'Season', 'Team', 'Number', 'Name', 'Position', 'PositionCategory']
passing_cols = ['PassingYards', 'PassingTouchdowns', 'TwoPointConversionPasses', 'PassingInterceptions']
rushing_cols = ['RushingYards', 'RushingTouchdowns', 'TwoPointConversionRuns']
receiving_cols = ['Receptions', 'ReceivingYards', 'ReceivingTouchdowns', 'TwoPointConversionReceptions']
misc_cols = ['Fumbles', 'FumbleReturnTouchdowns']

points_df = df[identity_cols + passing_cols + rushing_cols + receiving_cols + misc_cols]

def applyPoints(df, pointmap):
    total_points = 0
    for key, value in pointmap.items():
        if key not in df.columns:
            raise ValueError(f"Key {key} not in columns list")
        total_points += df[key] * value
    return total_points

point_map = {
    'PassingYards': 0.05,
    'PassingTouchdowns': 4,
    'TwoPointConversionPasses': 2,
    'PassingInterceptions': -1,
    'RushingYards': 0.1,
    'RushingTouchdowns': 6,
    'TwoPointConversionRuns': 2,
    'Receptions': 0.5,
    'ReceivingYards': 0.1,
    'ReceivingTouchdowns': 6,
    'TwoPointConversionReceptions': 2,
    'Fumbles': -2,
    'FumbleReturnTouchdowns': 6
}

points_df['Points'] = applyPoints(points_df, point_map)
points_df.sort_values(by='Points', ascending=False)

points_df.to_csv('football/data/2024/SeasonFantasyPoints.csv')
```
```python
endpoint = "https://api.sportsdata.io/api/nfl/fantasy/json/PlayerGameStatsByWeek/2024REG/1"

res = requests.get(endpoint, headers=headers)
data = res.json()

with open('football/data/2024/SeasonWeek1StatsRaw.json', 'w+') as f:
    f.write(json.dumps(data))
    
```
```python
df = pd.read_json('football/data/2024/SeasonWeek1StatsRaw.json')
df.columns

week_identity_cols = identity_cols + ['GameKey', 'Week', 'Opponent', 'HomeOrAway']
week_points_df = df[week_identity_cols + passing_cols + rushing_cols + receiving_cols + misc_cols]
week_points_df['Points'] = applyPoints(week_points_df, point_map)
week_points_df.sort_values(by='Points', ascending=False)
week_points_df.to_csv('football/data/2024/SeasonWeek1FantasyPoints.csv')
```
```python
parent_df = df
for i in range(2, 18):
    endpoint = f"https://api.sportsdata.io/api/nfl/fantasy/json/PlayerGameStatsByWeek/2024REG/{i}"
    res = requests.get(endpoint, headers=headers)
    data = res.json()
    
    with open(f'football/data/2024/SeasonWeek{i}StatsRaw.json', 'w+') as f:
        f.write(json.dumps(data))

    df = pd.read_json(f'football/data/2024/SeasonWeek{i}StatsRaw.json')
    week_points_df = df[week_identity_cols + passing_cols + rushing_cols + receiving_cols + misc_cols]
    week_points_df['Points'] = applyPoints(week_points_df, point_map)
    week_points_df.to_csv(f'football/data/2024/SeasonWeek{i}FantasyPoints.csv')
    parent_df = pd.concat([parent_df, df])
```
```python
parent_df.to_csv('football/data/2024/SeasonAllWeeksFantasyPoints.csv')
```

