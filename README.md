# Big Data Engineering (BDE) Honours Capstone Project
## Green Logistics: Carbon Footprint for E-Commerce


Capstone project for **COS 736 Big Data Engineering** (University of the Western Cape). It has two parts: a batch ETL pipeline on logistics emissions data, and a simulated real-time streaming pipeline for e-commerce transactions. This project was out of 35 and I receieved 84%. 

## Projects

| # | Project | Focus | Tools |
|---|---------|-------|-------|
| 1 | Green Logistics: Carbon Footprint for E-Commerce | ETL, analysis, visualisation | Python, pandas, MariaDB (`mariadb` connector), matplotlib |
| 2 | Streaming Data | Real-time simulation, message queues | Python, pandas, Apache Kafka (`confluent-kafka`), matplotlib |

---

## Project 1: Green Logistics

**Dataset:** `Logistics_Carbon_Emissions` (provided with the course; not included in this repo). 12,000 delivery records.
**Notebook:** `Question1A.ipynb` (covers 1A, 1B and 1C)

### 1A: ETL pipeline
- **Extract:** load the CSV with pandas, replace dashes with underscores in the column headers and `Route_Type` values, then create the `CarbonEmission` database in MariaDB and load all rows into a `combined` table.
- **Transform:**
  - Reclassify every `Mixed_Route` record by distance: `Inter_City` (over 100 km), `Inter_District` (20-100 km) or `Urban_Last_Mile` (under 20 km). No `Mixed_Route` records remain.
  - Recalculate `Carbon_Emission_kgCO2e` with `(Base Factor x Load Penalty x Traffic Multiplier) x Distance`, using the vehicle base factors and traffic multipliers from the brief. The load penalty is the package weight.
- **Load:** create one table per route type (`Inter_City`, `Inter_District`, `Urban_Last_Mile`), each ordered by vehicle type.

| Route type | Rows |
|------------|------|
| Inter_District | 4,709 |
| Urban_Last_Mile | 3,753 |
| Inter_City | 3,538 |

### 1B: Insights
- Total emissions per vehicle type for each route type
- Routes ranked by total distance travelled
- Eco-friendly vs non-eco-friendly distance for the year
- Total emissions per vehicle type per month

### 1C: Visualisations
- Grouped bar chart (log scale) of total emissions per vehicle type, by route type
- Pie chart of each route's share of total distance travelled
- Twelve column charts, one per month, showing emissions per vehicle type (log scale)

### Key findings
- **Inter-city routes dominate distance:** about 984,000 km, roughly 80% of the 1.23 million km travelled. Inter-district routes cover about 218,000 km and urban last-mile about 32,000 km.
- **Most distance is non-eco-friendly:** about 1.07 million km (around 87%) against about 165,000 km eco-friendly.
- **Heavy trucks drive emissions:** inter-city heavy trucks account for the overwhelming majority of total emissions, far above any diesel van category.

---

## Project 2: Streaming Data

**Dataset:** [Online Retail](https://doi.org/10.24432/C5BW33) (Chen, 2015), also on [Kaggle](https://www.kaggle.com/datasets/carrie1/ecommerce-data). 541,909 transactions.
**Notebooks:** `Question2A.ipynb` (2A) and `Question2B.ipynb` (2B, 2C and 2D)

### 2A: Preprocessing
- An `extract` function reads the `.xlsx` file with pandas.
- A `preprocess` function maps each `Country` to a `Region` (giving 9 columns) and splits the data into six regional dataframes: Europe, Asia Pacific, Middle East, North America, South America and Unspecified.
- Each regional dataframe is saved as a CSV file.

### 2B: Streaming with Kafka
- **Generator** reads the regional CSVs and `yield`s one row at a time.
- **Topics:** six Kafka topics, one per region (`Europe`, `Asia_Pacific`, `Middle_East`, `North_America`, `South_America`, `Unspecified`).
- **Producer:** a single producer takes each row from the generator and routes it to the topic matching its `Region`.
- **Consumer:** a single consumer subscribed to all six topics. For each message it counts messages per topic, calculates `Quantity x UnitPrice`, keeps a cumulative sales total per region and tracks purchases with a missing `CustomerID`.

Summary table produced by the consumer:

| Topic | Messages | Cumulative sales | Unknown customers |
|-------|----------|------------------|-------------------|
| Europe | 538,416 | 9,532,618.37 | 134,541 |
| Asia_Pacific | 2,431 | 199,563.14 | 335 |
| Middle_East | 142 | 4,275.73 | 2 |
| North_America | 442 | 5,397.30 | 0 |
| South_America | 32 | 1,143.60 | 0 |
| Unspecified | 446 | 4,749.79 | 202 |

### 2C: Insights
Answered with pandas.

- **Top 3 countries by number of purchases** (counting transaction lines):

| Country | Purchases | Total sales |
|---------|-----------|-------------|
| United Kingdom | 495,478 | 8,187,806.36 |
| Germany | 9,495 | 221,698.21 |
| France | 8,557 | 197,403.90 |

- **Most sold item per region (by quantity):**

| Region | Item | Quantity |
|--------|------|----------|
| Europe | World War 2 Gliders Asstd Designs | 52,742 |
| Asia Pacific | Rabbit Night Light | 5,285 |
| North America | Retro Coffee Mugs Assorted | 504 |
| Middle East | Ice Cream Sundae Lip Gloss | 96 |
| Unspecified | World War 2 Gliders Asstd Designs | 96 |
| South America | Small Heart Flowers Hook | 24 |

### 2D: Visualisations
Charts for the streaming summary (messages, sales and unknown customers per region), the top 3 countries (purchases and sales), and the most sold item per region.

### Key findings
- Europe accounts for about 99% of all transactions, and the UK alone for about 91%.
- About a quarter of Europe's transactions (134,541 of 538,416) have no customer ID.

---

## Repository structure

```
.
├── Project1_GreenLogistics/
│   └── Question1A.ipynb        # 1A ETL, 1B insights, 1C visuals
├── Project2_StreamingData/
│   ├── Question2A.ipynb        # extract, preprocess, persist
│   └── Question2B.ipynb        # Kafka pipeline (2B), insights (2C), visuals (2D)
├── requirements.txt
├── .gitignore
└── README.md
```

## Getting started

### Prerequisites
- Python 3.11+
- MariaDB server (Project 1)
- Apache Kafka broker running on `localhost:9092` (Project 2)

### Setup
```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install pandas openpyxl mariadb confluent-kafka matplotlib
```

Set your database credentials as environment variables rather than hard-coding them in the notebook.

### Running
1. Download the datasets (links above) into a local data folder.
2. **Project 1:** start MariaDB and run `Question1A.ipynb` top to bottom.
3. **Project 2:** run `Question2A.ipynb` to create the regional CSVs, start Kafka, then run `Question2B.ipynb` (topics, producer, consumer, then 2C and 2D).

## Acknowledgements
- Chen, D. (2015). *Online Retail* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33
