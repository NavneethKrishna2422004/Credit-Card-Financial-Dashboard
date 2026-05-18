# Credit Card Analytics Dashboard

I built this project to analyze credit card performance, customer behavior, and transaction trends using PostgreSQL and Power BI. The data covers 667K+ transactions worth $57M in total revenue.

---

## What this project does

Takes raw credit card and customer data, loads it into a PostgreSQL database, and visualizes it across 3 Power BI dashboards — the kind of reporting a bank or fintech team would actually use.

---

## Tools used

- **PostgreSQL** — to set up the database and load the data
- **Power BI** — to build the dashboards
- **DAX** — for calculated measures and KPIs

---

## The Dashboards

### 1. Credit Card Performance Report
Tracks how revenue is moving week over week. Week 53 saw a 28.8% spike — current week hit $1M vs $933K the previous week. Also breaks down revenue by card type (Blue, Silver, Gold, Platinum) and shows which customer jobs have the highest delinquency rates.

### 2. Credit Card Transaction Report
Big picture view of the business — $57M revenue, $46M interest earned, 667K transactions. Breaks it all down by quarter, card category, how customers paid (swipe/chip/online), what they spent on (bills, fuel, grocery, etc.), education level, and job type.

### 3. Credit Card Customer Report
Focuses on who the customers actually are — age groups, income levels, gender, marital status, top states (TX, NY, CA, FL, NJ). Also tracks monthly revenue through all of 2023 and customer satisfaction scores.

---

## SQL

### Setting up the database

```sql
CREATE DATABASE ccdb;

-- Credit card transactions table
CREATE TABLE cc_detail (
    Client_Num INT,
    Card_Category VARCHAR(20),
    Annual_Fees INT,
    Activation_30_Days INT,
    Customer_Acq_Cost INT,
    Week_Start_Date DATE,
    Week_Num VARCHAR(20),
    Qtr VARCHAR(10),
    current_year INT,
    Credit_Limit DECIMAL(10,2),
    Total_Revolving_Bal INT,
    Total_Trans_Amt INT,
    Total_Trans_Ct INT,
    Avg_Utilization_Ratio DECIMAL(10,3),
    Use_Chip VARCHAR(10),
    Exp_Type VARCHAR(50),
    Interest_Earned DECIMAL(10,3),
    Delinquent_Acc VARCHAR(5)
);

-- Customer details table
CREATE TABLE cust_detail (
    Client_Num INT,
    Customer_Age INT,
    Gender VARCHAR(5),
    Dependent_Count INT,
    Education_Level VARCHAR(50),
    Marital_Status VARCHAR(20),
    State_cd VARCHAR(50),
    Zipcode VARCHAR(20),
    Car_Owner VARCHAR(5),
    House_Owner VARCHAR(5),
    Personal_Loan VARCHAR(5),
    Contact VARCHAR(50),
    Customer_Job VARCHAR(50),
    Income INT,
    Cust_Satisfaction_Score INT
);
```

### Loading the data

```sql
-- Initial load
COPY cc_detail FROM 'D:\credit_card.csv' DELIMITER ',' CSV HEADER;
COPY cust_detail FROM 'D:\customer.csv' DELIMITER ',' CSV HEADER;

-- Added more data later (incremental load)
COPY cc_detail FROM 'D:\cc_add.csv' DELIMITER ',' CSV HEADER;
COPY cust_detail FROM 'D:\cust_add.csv' DELIMITER ',' CSV HEADER;
```

---

## DAX Measures

### Revenue

```dax
Revenue = 
    'public cc_detail'[annual_fees] 
    + 'public cc_detail'[total_trans_amt] 
    + 'public cc_detail'[interest_earned]
```

### Week-over-Week tracking

```dax
Week_num2 = WEEKNUM('public cc_detail'[week_start_date])

Current_week_revenue = 
    CALCULATE(
        SUM('public cc_detail'[Revenue]),
        FILTER(
            ALL('public cc_detail'),
            'public cc_detail'[Week_num2] = MAX('public cc_detail'[Week_num2])
        )
    )

Previous_week_revenue = 
    CALCULATE(
        SUM('public cc_detail'[Revenue]),
        FILTER(
            ALL('public cc_detail'),
            'public cc_detail'[Week_num2] = MAX('public cc_detail'[Week_num2]) - 1
        )
    )

wow_revenue = 
    DIVIDE(
        ([Current_week_revenue] - [Previous_week_revenue]),
        [Previous_week_revenue]
    )
```

### Customer segmentation

```dax
AgeGroup = 
    SWITCH(
        TRUE(),
        'public cust_detail'[customer_age] < 30, "20-30",
        'public cust_detail'[customer_age] >= 30 && 'public cust_detail'[customer_age] < 40, "30-40",
        'public cust_detail'[customer_age] >= 40 && 'public cust_detail'[customer_age] < 50, "40-50",
        'public cust_detail'[customer_age] >= 50 && 'public cust_detail'[customer_age] < 60, "50-60",
        'public cust_detail'[customer_age] >= 60, "60+",
        "unknown"
    )

IncomeGroup = 
    SWITCH(
        TRUE(),
        'public cust_detail'[income] < 35000, "low",
        'public cust_detail'[income] >= 35000 && 'public cust_detail'[income] < 70000, "med",
        'public cust_detail'[income] >= 70000, "high",
        "unknown"
    )
```

---

## Key findings

- Blue card alone brings in ~$47M out of $57M total — it's carrying the whole portfolio
- Businessmen are the most valuable customer segment at $18M
- Most customers still swipe ($36M) — chip and online are way behind
- Graduate-level customers spend the most at $23M
- Delinquency is only at 6.06% overall, government employees have the lowest rate (0.61%)
- Week 53 had a big 28.8% revenue jump — worth investigating what drove it

---

## How to run it

1. Clone the repo:
   ```bash
   git clone https://github.com/NavneethKrishna2422004/credit-card-analytics.git
   ```
2. Run the SQL in pgAdmin to create the database and load the CSVs
3. Open the `.pbix` file in Power BI Desktop
4. Connect it to your PostgreSQL database and hit refresh

---

## Author

**Navneeth Krishna**  
[GitHub](https://github.com/NavneethKrishna2422004) · [LinkedIn](https://linkedin.com/in/navneethkrishna2004)
