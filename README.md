# Analysis-of-Global-Unicorn-Growth-2019-2021-
Strategic analysis of global Unicorn growth (2019-2021). Utilizes multi-step SQL (CTEs) to identify top-performing industries, track yearly valuation trends, and advise investment firms on portfolio allocation. Skills demonstrated: SQL, Data Aggregation, Business Intelligence, Investment Analysis.
## **Data Source**
Data is taken from the Unicorns Database dataset, which includes four main tables:
### `dates`
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `date_joined` | The date that the company became a unicorn.  |
| `year_founded` | The year that the company was founded.       |

### `funding`
| Column           | Description                                  |
|----------------- |--------------------------------------------- |
| `company_id`       | A unique ID for the company.                 |
| `valuation`        | Company value in US dollars.                 |
| `funding`          | The amount of funding raised in US dollars.  |
| `select_investors` | A list of key investors in the company.      |

### `industries`
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `industry`     | The industry that the company operates in.   |

### `companies`
| Column       | Description                                       |
|------------- |-------------------------------------------------- |
| `company_id`   | A unique ID for the company.                      |
| `company`      | The name of the company.                          |
| `city`         | The city where the company is headquartered.      |
| `country`      | The country where the company is headquartered.   |
| `continent`    | The continent where the company is headquartered. |

## **Analysis Methodology (3 Logical Steps)**
This analysis is completed in a single SQL query structured into three CTEs, according to the project objectives:

### 1. Finding the Top Industries (Initial Selection Phase)
Objective: To identify all relevant industries, i.e., those with companies achieving unicorn status between 2019 and 2021.

SQL Query (relevant_industries CTE Section):
```sql
WITH most_valuable_company AS (
	-- 1. Finding the top industries:
	-- based on the number of unicorns that joined (2019-2021)
	SELECT 
		i.industry,
	    COUNT(d.company_id) AS total_unicorns
	FROM 
		dates AS d
	INNER JOIN 
		industries AS i
		ON d.company_id = i.company_id
	WHERE 
		EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
	GROUP BY 
		i.industry
	ORDER BY 
		total_unicorns DESC
	LIMIT 3
),
```
### **2. Gathering Yearly Rankings Data (Data Collection Phase)**
Purpose: To collect raw data (year, valuation) from all companies included in the relevant industry list, and filter based on the requested time period (2019-2021).

SQL Query (industry_yearly_data CTE Section):
```sql
industry_yearly_data AS (
	-- 2. Gathering yearly rankings data:
	-- Collecting data per year, specifically for companies from the top 3 industries
	SELECT 
		n.industry,
		EXTRACT(year FROM d.date_joined ) AS year,
		f.valuation
	FROM 
		dates AS d
	INNER JOIN 
		industries AS n
		ON d.company_id = n.company_id
	INNER JOIN 
		funding AS f
		ON d.company_id = f.company_id
	WHERE
		n.industry IN (SELECT industry FROM most_valuable_company)
		AND EXTRACT(YEAR FROM d.date_joined) IN (2019,2020,2021)
)
```
### **3. Returning the Final Results (Aggregation & Reporting Stage)
Purpose: Calculate the final metrics (num_unicorns, average_valuation_billions) and format the results according to client requirements (rounded and sorted).

SQL Query (Final SELECT Section):
```sql
SELECT 
	industry,
	year,
	COUNT(valuation) AS num_unicorns,
	ROUND(CAST(AVG(valuation) AS numeric) / 1000000000, 2) AS average_valuation_billions
FROM 
	industry_yearly_data
GROUP BY
	industry,
	year
ORDER BY
	year DESC,
	num_unicorns DESC;
  ```
## **Results and Key Findings**
Final Result (Example Query Output):
| index | industry | year | num_unicorns | average_valuation_billions |
|---|---|---|---|---|
| 0 | Fintech | 2021 | 138 | 2.75 |
| 1 | Internet Software & Services | 2021 | 119 | 2.15 |
| 2 | E-commerce & D2C | 2021 | 47 | 2.47 |
| 3 | Internet Software & Services | 2020 | 20 | 4.35 |
| 4 | E-commerce & D2C | 2020 | 16 | 4.00 |
| 5 | Fintech | 2020 | 15 | 4.33 |
| 6 | Fintech | 2019 | 20 | 6.80 |
| 7 | Internet Software & Services | 2019 | 13 | 4.23 |
| 8 | E-commerce & D2C | 2019 | 12 | 2.52 |

## **Key Insights:**

- Dramatic Growth Surge (Post-Pandemic): There was a massive increase in the number of unicorns in 2021 (a total of 304 new unicorns from the top 3 industries) compared to 2020 (a total of 51 unicorns). This increase indicates an acceleration of market liquidity and digitalization post-pandemic.

- Fintech Dominates in Quantity: Fintech was the most productive industry in 2021, creating 138 new unicorns, almost 7 times more than in 2020.

- Highest Valuations at the Beginning of the Period: Average valuations tended to be higher in 2019 and 2020 compared to 2021. The highest average valuation was recorded in Fintech 2019 ($6.8 Billion), although the number of unicorns was low (20). This may indicate that in 2021, more companies reached unicorn status with the minimum valuation ($1 Billion), thus lowering the average.


