<div align="center">

# 🚖 Goodcabs – Operations Analysis

### Tier-2 City Cab Performance | SQL + Power BI

<br/>

![SQL](https://img.shields.io/badge/SQL-003B57?style=for-the-badge&logo=postgresql&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-00758F?style=for-the-badge&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Status](https://img.shields.io/badge/Status-In%20Progress-f0883e?style=for-the-badge)

</div>

---

## 🚖 About Goodcabs

- 🗓️ **Founded in 2022**, Goodcabs is committed to transforming cab services in Tier-2 cities
- 🏙️ Currently operating in **10 Tier-2 cities** across India, with plans for expansion
- 🤝 Supports **local drivers** while delivering affordable, reliable rides
- 🌟 **Vision:** Become the leading cab service in Tier-2 cities while maintaining a balance between profitability and community support

---

## 🎯 Objective

- 📊 Analyze dataset to address key business questions on:
  - **City-Level Trip Performance**
  - **Passenger Trends**
  - **Revenue Patterns**
  - **Target Achievement**
- 🖥️ Present findings in a **clear and visually engaging manner**
- 💼 Assist the **Chief of Operations** in making informed strategic decisions

---

## 🗂️ Dataset Overview

| Table | Description |
|-------|-------------|
| `trips_db` | Trip-level records — city, fare, distance, passenger type, ratings |
| `targets_db` | Monthly city-wise targets for trips, new passengers & ratings |
| `dim_date` | Date dimension — month, fiscal year info |
| `dim_city` | City metadata |

---

## ❓ Business Questions & SQL Queries

---

### Q1 — Top 3 & Bottom 3 EV Makers for 2-Wheelers — FY 2023 & 2024

**🏆 Top 3**

```sql
SELECT s.maker, SUM(electric_vehicles_sold) AS sold_quantity
FROM electric_vehicle_sales_by_makers s
JOIN dim_date d ON d.date = s.date
WHERE d.fiscal_year IN (2023, 2024)
  AND s.vehicle_category = '2-wheelers'
GROUP BY s.maker
ORDER BY sold_quantity DESC
LIMIT 3;
```

**📉 Bottom 3**

```sql
SELECT s.maker, SUM(electric_vehicles_sold) AS sold_quantity
FROM electric_vehicle_sales_by_makers s
JOIN dim_date d ON d.date = s.date
WHERE d.fiscal_year IN (2023, 2024)
  AND s.vehicle_category = '2-wheelers'
GROUP BY s.maker
ORDER BY sold_quantity ASC
LIMIT 3;
```

---

### Q2 — Top 5 States by Penetration Rate in 2-Wheeler & 4-Wheeler EV Sales — FY 2024

**🛵 2-Wheelers**

```sql
SELECT e.state,
       (SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) * 100) AS penetration_rate
FROM electric_vehicle_sales_by_state e
JOIN dim_date d ON d.date = e.date
WHERE fiscal_year = 2024
  AND e.vehicle_category = '2-wheelers'
GROUP BY e.state
ORDER BY penetration_rate DESC
LIMIT 5;
```

**🚗 4-Wheelers**

```sql
SELECT e.state,
       (SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) * 100) AS penetration_rate
FROM electric_vehicle_sales_by_state e
JOIN dim_date d ON d.date = e.date
WHERE fiscal_year = 2024
  AND e.vehicle_category = '4-wheelers'
GROUP BY e.state
ORDER BY penetration_rate DESC
LIMIT 5;
```

---

### Q3 — States with Negative Penetration (Decline) in EV Sales — 2022 to 2024

```sql
WITH pct_change_previousyear AS (
    SELECT e.state,
           (SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) * 100) AS previoos_pent
    FROM electric_vehicle_sales_by_state e
    JOIN dim_date d ON d.date = e.date
    WHERE d.fiscal_year = 2023
    GROUP BY e.state
),
pct_change_currentyear AS (
    SELECT e.state,
           (SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) * 100) AS current_pent
    FROM electric_vehicle_sales_by_state e
    JOIN dim_date d ON d.date = e.date
    WHERE d.fiscal_year = 2024
    GROUP BY e.state
)
SELECT c.state,
       c.current_pent - p.previoos_pent AS pct_change
FROM pct_change_previousyear p
JOIN pct_change_currentyear c ON p.state = c.state
WHERE c.current_pent - p.previoos_pent < 0
ORDER BY pct_change;
```

---

### Q4 — Quarterly Trends for Top 5 EV Makers (4-Wheelers) — 2022 to 2024

```sql
WITH cte1 AS (
    SELECT e.maker, d.quarter,
           SUM(e.electric_vehicles_sold) AS ev_sales,
           ROW_NUMBER() OVER (PARTITION BY d.quarter 
                              ORDER BY SUM(e.electric_vehicles_sold) DESC) AS rnk
    FROM electric_vehicle_sales_by_makers e
    JOIN dim_date d ON d.date = e.date
    WHERE e.vehicle_category = '4-Wheelers'
    GROUP BY e.maker, d.quarter
)
SELECT maker, quarter, ev_sales
FROM cte1
WHERE rnk <= 5
ORDER BY quarter, ev_sales DESC;
```

---

### Q5 — EV Sales & Penetration Rate: Delhi vs Karnataka — FY 2024

```sql
SELECT state,
       SUM(electric_vehicles_sold) AS ev_sales,
       SUM(total_vehicles_sold)    AS total_vehicle_sales,
       ROUND(SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) * 100, 2) AS penetration_rate
FROM electric_vehicle_sales_by_state e
JOIN dim_date d ON d.date = e.date
WHERE state IN ('Delhi', 'Karnataka')
  AND fiscal_year = 2024
GROUP BY state
ORDER BY state;
```

---

### Q6 — CAGR in 4-Wheeler Units — Top 5 Makers — 2022 to 2024

```sql
WITH CAGR_FY2024 AS (
    SELECT m.maker, SUM(electric_vehicles_sold) AS EV_Sales2024
    FROM electric_vehicle_sales_by_makers m
    JOIN dim_date d ON m.date = d.date
    WHERE d.fiscal_year = 2024
    GROUP BY m.maker
),
CAGR_FY2022 AS (
    SELECT m.maker, SUM(electric_vehicles_sold) AS EV_Sales2022
    FROM electric_vehicle_sales_by_makers m
    JOIN dim_date d ON m.date = d.date
    WHERE d.fiscal_year = 2022
    GROUP BY m.maker
)
SELECT c.maker, c.EV_Sales2024, p.EV_Sales2022,
    CASE
        WHEN p.EV_Sales2022 > 0
        THEN ROUND((POWER(c.EV_Sales2024 / p.EV_Sales2022, 1/2) - 1) * 100, 2)
        ELSE NULL
    END AS CAGR_Maker_Percent
FROM CAGR_FY2024 c
JOIN CAGR_FY2022 p ON c.maker = p.maker
ORDER BY CAGR_Maker_Percent DESC
LIMIT 5;
```

---

### Q7 — Top 10 States by CAGR in Total Vehicles Sold — 2022 to 2024

```sql
WITH CAGR_FY2024 AS (
    SELECT st.state, SUM(total_vehicles_sold) AS TV_Sales2024
    FROM electric_vehicle_sales_by_state st
    JOIN dim_date d ON st.date = d.date
    WHERE d.fiscal_year = 2024
      AND st.vehicle_category = '2-Wheelers'
    GROUP BY st.state
),
CAGR_FY2022 AS (
    SELECT st.state, SUM(total_vehicles_sold) AS TV_Sales2022
    FROM electric_vehicle_sales_by_state st
    JOIN dim_date d ON st.date = d.date
    WHERE d.fiscal_year = 2022
      AND st.vehicle_category = '2-Wheelers'
    GROUP BY st.state
)
SELECT c.state, c.TV_Sales2024, p.TV_Sales2022,
    CASE
        WHEN p.TV_Sales2022 > 0
        THEN ROUND((POWER(c.TV_Sales2024 / p.TV_Sales2022, 1/2) - 1) * 100, 2)
        ELSE NULL
    END AS CAGR_State_Percent
FROM CAGR_FY2024 c
JOIN CAGR_FY2022 p ON c.state = p.state
ORDER BY CAGR_State_Percent DESC
LIMIT 10;
```

---

### Q8 — Peak & Low Season Months for EV Sales — 2022 to 2024

```sql
WITH New_Dim_Date AS (
    SELECT date, fiscal_year,
           MONTHNAME(STR_TO_DATE(date, '%d-%b-%y')) AS Month_Name
    FROM dim_date
)
SELECT n.Month_Name,
       SUM(m.electric_vehicles_sold) AS EV_Sales
FROM New_Dim_Date n
JOIN electric_vehicle_sales_by_makers m ON n.date = m.date
WHERE n.fiscal_year IN (2022, 2023, 2024)
GROUP BY n.Month_Name
ORDER BY EV_Sales DESC;
```

---

## 📊 Dashboard

> 🚧 **Coming Soon** — Power BI Dashboard is currently under development.  
> Will include: City-wise performance, passenger trends, revenue analysis & target tracking.

---

## 📁 Project Structure

```
Goodcabs-Transportation/
│
├── 📄 primary_and_secondary_quest...   ← SQL queries file
├── 📄 ad-hoc-requests.pdf              ← Business questions PDF
├── 📄 meta_data.txt                    ← Dataset metadata
└── 📄 README.md
```

---

## 📬 Connect

<div align="center">

[![LinkedIn](https://img.shields.io/badge/Madhuri_Padole-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/madhuri-padole-93b875259)
[![Gmail](https://img.shields.io/badge/madhuripadole390@gmail.com-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:madhuripadole390@gmail.com)
[![GitHub](https://img.shields.io/badge/madhuri808-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/madhuri808)

<br/>

> 🚖 *"Every trip tells a story — I find the patterns that drive smarter decisions."*

</div>
