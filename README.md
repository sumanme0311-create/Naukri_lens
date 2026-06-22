<div align="center">

# 📊 Naukri Job Market — Data Visualization & Analysis

**Exploring ~30,000 job postings from India's largest job portal**

![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-Visualization-4C72B0?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/naukri-job-analysis/blob/main/naukri_job_analysis.ipynb)
&nbsp;&nbsp;
[![Kaggle Dataset](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?logo=kaggle)](https://www.kaggle.com/datasets/promptcloud/jobs-on-naukricom)

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Key Findings](#-key-findings)
- [Project Structure](#-project-structure)
- [Dataset](#-dataset)
- [Setup & Installation](#-setup--installation)
- [Analysis Sections](#-analysis-sections)
  - [1. Data Preparation & Cleaning](#1-data-preparation--cleaning)
  - [2. Geographic Distribution](#2-geographic-distribution--india-map)
  - [3. Industry Analysis](#3-industry-analysis)
  - [4. Experience Requirements](#4-experience-requirements)
  - [5. Industry × Experience Patterns](#5-industry--experience-patterns)
  - [6. Key Skills per Industry](#6-key-skills-per-industry)
  - [7. Job Title Analysis](#7-job-title-analysis)
- [Tech Stack](#-tech-stack)
- [License](#-license)

---

## 🔍 Overview

Job searching is one of the biggest concerns for many people. Understanding the job market — geographically, by experience level, and by required skills — helps job seekers better prepare for entering a new industry or switching careers.

This project analyses **~30,000 job postings** from [Naukri.com](https://www.naukri.com) (July–August 2019) and answers:

> 🗺️ **Where** are the jobs? &nbsp;|&nbsp; 🏭 **Which industries** are hiring? &nbsp;|&nbsp; 🎓 **What experience** is needed? &nbsp;|&nbsp; 🛠️ **What skills** are in demand?

---

## 💡 Key Findings

| # | Insight |
|---|---------|
| 🥇 | **IT / Software Services** accounts for the largest share of all job postings |
| 🗺️ | **Maharashtra, Karnataka & Uttar Pradesh** are the top 3 hiring states |
| 🎓 | **Semiprofessional (1–5 yrs)** roles dominate — nearly 50% of all postings |
| 🚨 | **Programming & Design** has the highest urgency in hiring |
| 🛠️ | **Java, Python, SQL** are the most demanded skills in the IT sector |
| 👶 | **Education & Training** industry recruits the highest proportion of Newbies |
| 🧑‍💼 | **Strategy / Consulting** firms recruit the most Experts (10+ years) |

---

## 📁 Project Structure

```
naukri-job-analysis/
│
├── 📓 naukri_job_analysis.ipynb    ← Main analysis notebook
├── 📄 README.md                    ← Project documentation (this file)
├── 📦 requirements.txt             ← Python dependencies
├── 🔒 .gitignore                   ← Ignored files (data, checkpoints, etc.)
│
└── 📊 images/                      ← Chart outputs
    ├── missing_data.png
    ├── top_industries.png
    ├── top_states.png
    ├── experience_pie.png
    ├── recruitment_pattern.png
    ├── top_job_titles.png
    ├── top_keywords.png
    └── urgent_jobs.png
```

---

## 🗄️ Dataset

| Property | Details |
|---|---|
| **Source** | [Kaggle — Jobs on Naukri.com](https://www.kaggle.com/datasets/promptcloud/jobs-on-naukricom) |
| **Period** | July 1, 2019 – August 30, 2019 |
| **Records** | ~30,000 job postings |
| **Key Columns** | `Job Title`, `Industry`, `Location`, `Job Experience Required`, `Key Skills`, `Role Category`, `Job Salary` |
| **Supplement Files** | India shapefile, city-state mapping, unmatched location dictionary |

> ⚠️ **Data files are NOT included** in this repo due to size. Download from Kaggle and place at the path referenced in the notebook.

---

## ⚙️ Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/YOUR_USERNAME/naukri-job-analysis.git
cd naukri-job-analysis
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Download the dataset
Go to the [Kaggle page](https://www.kaggle.com/datasets/promptcloud/jobs-on-naukricom) and download:
- `marketing_sample_for_naukri_com-jobs__20190701_20190830__30k_data.csv`
- Supplement files: `india-polygon.shp`, `Town_Codes_2001.xls`, `unmatchedLocations.npy`, `data_ecxel.xlsx`

### 5. Launch the notebook
```bash
jupyter notebook naukri_job_analysis.ipynb
```

---

## 📊 Analysis Sections

---

### 1. Data Preparation & Cleaning

**Load and inspect the raw data:**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

pd.set_option('display.max_columns', 15)
data_df = pd.read_csv('naukri_jobs.csv')
data_df.head()
```

**Check and visualize missing values:**

```python
selected_col = ['Job Title', 'Job Salary', 'Job Experience Required',
                'Key Skills', 'Role Category', 'Location',
                'Functional Area', 'Industry', 'Role']
for col in selected_col:
    print(f'Missing in {col}:', data_df[col].isna().sum())

sns.heatmap(data_df.isnull(), cbar=True, cmap='gnuplot')
```

![Missing Data Chart](images/missing_data.png)

> **Job Salary** had the most missing values and was excluded from further analysis. All rows with remaining nulls were dropped before proceeding.

**Drop missing rows:**
```python
data_df.dropna(axis=0, inplace=True)
```

---

### 2. Geographic Distribution — India Map

Jobs are mapped from city-level to state-level using a city-to-state lookup, then plotted on an India choropleth map.

```python
import geopandas as gpd

fp = 'india-polygon.shp'
map_df = gpd.read_file(fp)

def get_ind_city_cnt(ind):
    cityCnt = {}
    sub_data = data_df[data_df['Industry'].str.contains(ind)]
    for i in range(len(sub_data)):
        cities = sub_data.iloc[i, 7]
        if "(" in cities:
            cities = cities[:cities.find("(")]
        for city in cities.split(","):
            city = city.strip().lower()
            cityCnt[city] = cityCnt.get(city, 0) + 1
    return cityCnt
```

**Top 10 States by Job Count:**

![Top States](images/top_states.png)

> Maharashtra leads by a wide margin, followed by Karnataka and Uttar Pradesh. Job opportunities are **heavily concentrated in the west and south** of India.

---

### 3. Industry Analysis

```python
# Standardise duplicate industry names
data_df.loc[data_df['Industry'].str.contains('IT-Software / Software Services', case=False)] = \
    'IT-Software, Software Services'

top10 = data_df['Industry'].value_counts()[:10]
print(f"Top 10 industries represent {top10.sum()/len(data_df):.1%} of all postings")
```

![Top Industries](images/top_industries.png)

> **IT-Software & Software Services** dominates with nearly 4× the postings of the second-largest industry (Recruitment & Staffing). The top 10 industries together account for the vast majority of all job postings on the platform.

---

### 4. Experience Requirements

Experience values (e.g. `"3-7 yrs"`) are parsed and mapped into four clean categories:

| Category | Experience Range |
|---|---|
| 👶 Newbie | 0 – 1 year |
| 🧑 Semiprofessional | 1 – 5 years |
| 💼 Professional | 5 – 10 years |
| 🏆 Expert | 10+ years |

```python
CAT = {(0, 1): "Newbie", (1, 5): "Semiprofessional",
       (5, 10): "Professional", (10, 100): "Expert"}

def get_cat(yrs):
    if len(yrs) != 2: return ""
    start, end = int(yrs[0].strip()), int(yrs[1].strip())
    res = ''
    for key, val in CAT.items():
        if start <= key[1] and end > key[0]:
            res += ' ' + val
    return res.strip()

# Apply to all rows
for i in range(len(data_df)):
    job_req = data_df.iloc[i, 4].lower()
    yrs = job_req[:job_req.find('y')].split('-')
    data_df.iloc[i, -1] = get_cat(yrs)
```

![Experience Pie](images/experience_pie.png)

> Semiprofessional roles (1–5 yrs) dominate the market. Experts are comparatively rare — most companies are not urgently looking for 10+ year veterans.

---

### 5. Industry × Experience Patterns

For each of the top 10 industries, we compute the **percentage breakdown** across experience levels to reveal which industries favour newcomers vs veterans.

```python
# Merge experience-level counts per industry
merged = new_cnt.merge(semipro_cnt, on="index", how="left")
merged = merged.merge(pro_cnt,    on="index", how="left")
merged = merged.merge(exp_cnt,    on="index", how="left")
merged = merged.fillna(0)

# Convert to percentages
for i in range(len(merged)):
    total = merged[["Newbie","Semiprofessional","Professional","Expert"]].iloc[i].sum()
    merged.at[i, "Percent of New"]    = merged["Newbie"].iloc[i]          / total
    merged.at[i, "Percent of Semipro"]= merged["Semiprofessional"].iloc[i]/ total
    merged.at[i, "Percent of Pro"]    = merged["Professional"].iloc[i]    / total
    merged.at[i, "Percent of Exp"]    = merged["Expert"].iloc[i]          / total
```

![Recruitment Pattern](images/recruitment_pattern.png)

> - 🎓 **Education & Training** recruits the most Newbies
> - 🏦 **Banking & Finance** leans heavily Professional
> - 🧩 **Strategy / Consulting** has the highest Expert share

---

### 6. Key Skills per Industry

A bar chart + word cloud is generated for each of the top 10 industries.

```python
from wordcloud import WordCloud

def topSkills(Industry):
    ind = data_df[data_df['Industry'] == Industry]
    skillCnt = {}
    for row, cnt in ind['Key Skills'].value_counts().items():
        for skill in row.split('|'):
            skill = skill.strip().lower()
            skillCnt[skill] = skillCnt.get(skill, 0) + cnt

    skillCnt = pd.Series(skillCnt).sort_values(ascending=False)

    # Bar chart
    skillCnt[:10].plot(kind='bar', figsize=(9, 5), title=f"{Industry} — Top 10 Skills")
    plt.tight_layout(); plt.show()

    # Word cloud
    wordcloud = WordCloud(width=500, height=500, background_color='white').generate(
        ' '.join([f'{k} ' * v for k, v in skillCnt.items()])
    )
    plt.figure(figsize=(8, 8)); plt.imshow(wordcloud); plt.axis("off"); plt.show()

topSkills('IT-Software, Software Services')
topSkills('Banking, Financial Services, Broking')
# ... (called for all 10 industries in the notebook)
```

**Industries covered:**

| # | Industry |
|---|---|
| 1 | IT-Software / Software Services |
| 2 | Recruitment & Staffing |
| 3 | BPO / Call Centre / ITeS |
| 4 | Banking, Financial Services & Broking |
| 5 | Education, Teaching & Training |
| 6 | Medical, Healthcare & Hospitals |
| 7 | Strategy & Management Consulting |
| 8 | Internet & E-commerce |
| 9 | Media, Entertainment & Internet |
| 10 | Travel, Hotels, Restaurants, Airlines & Railways |

---

### 7. Job Title Analysis

#### Top 10 Most Common Job Titles

```python
jobTitle = data_df['Job Title']
dic = {}
for job in jobTitle:
    dic[job] = dic.get(job, 0) + 1

topTenJobTitle = sorted(dic.items(), key=lambda x: x[1], reverse=True)[:10]
```

![Top Job Titles](images/top_job_titles.png)

#### Top 15 Keywords in Job Titles

```python
keywords = {}
wordFilter = ["for", "in", "opening", ""]
for key, val in dic.items():
    for sub in key.split():
        if sub.lower() not in wordFilter:
            keywords[sub] = keywords.get(sub, 0) + val
```

![Top Keywords](images/top_keywords.png)

#### Urgent Job Roles

```python
jobTypeInUrgent = {}
for index, job in data_df.iterrows():
    if "urgent" in job['Job Title'].lower():
        role = job['Role Category']
        jobTypeInUrgent[role] = jobTypeInUrgent.get(role, 0) + 1
```

![Urgent Jobs](images/urgent_jobs.png)

> **Programming & Design** topped urgent postings — companies need developers fast. Surprisingly, **Teaching** roles also appeared in the top 10 urgent listings.

---

## 🛠️ Tech Stack

| Library | Version | Purpose |
|---|---|---|
| `pandas` | ≥ 1.3 | Data loading, wrangling, aggregation |
| `numpy` | ≥ 1.21 | Array operations |
| `matplotlib` | ≥ 3.4 | Base plotting |
| `seaborn` | ≥ 0.11 | Statistical visualizations |
| `geopandas` | ≥ 0.10 | Geospatial data handling |
| `geoplot` | ≥ 0.5 | India choropleth map |
| `wordcloud` | ≥ 1.8 | Skill word clouds per industry |
| `openpyxl` | ≥ 3.0 | Reading `.xlsx` supplement files |
| `xlrd` | ≥ 2.0 | Reading legacy `.xls` files |

---

## 📄 License

This project is open-source under the [MIT License](LICENSE).

---

<div align="center">

Made with ❤️ | Data from [Naukri.com via Kaggle](https://www.kaggle.com/datasets/promptcloud/jobs-on-naukricom)

⭐ **Star this repo if you found it useful!**

</div>
