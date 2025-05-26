# Hotel Revenue
This is the GitHub repository for storing Hotel Revenue Dashboards created using **Power BI**. Project is based on the [Hotel Revenue Dataset](https://www.kaggle.com/datasets/govindkrishnadas/hotel-revenue). A static PDF export of the dashboards is available [here](./Hotel%20Revenue%20Dashboards.pdf). Please note that the PDF version does not retain interactive features.

Polish version of the document can be [found here](README-PL.md)/Polską wersję dokumentu można [znaleźć tutaj](README-PL.md).

## Contents
* [Summary Dashboard](#summary-dashboard)
* [Meals & Special Requests](#meals--special-requests)
* [Demography](#demography)
* [Marketing & Operations](#marketing--operations)
* [Data Loading](#data-loading)
* [DAX Measures and Columns](#dax-measures-and-columns)

### Summary Dashboard
![Summary Dashboard](./images/Summary%20Dashboard.png)

A summary dashboard highlighting key financial metrics for hotels. Features a donut chart comparing revenue distribution by hotel type (City vs. Resort), interactive date filters (2018–2020), and insights into discounts and parking space utilization. A supplementary table provides granular details on annual revenue trends, parking rights, and discount percentages for deeper analysis.

For details on the DAX measures and calculated columns used in this dashboard, refer to the [DAX Measures and Columns](#summary-dashboard-1).

### Meals & Special Requests
![Meals & Special Requests](./images/Meals%20&%20Special%20Requests.png)

The "Meals & Special Requests" dashboard focuses on analyzing meal-related revenue and guest special requests. Key metrics include total meal revenue, average revenue per stay, and special request statistics. It features a yearly data table and a breakdown of revenue by meal type (e.g., Bed & Breakfast, Full Board). Interactive timeline visualizations track revenue and request trends across selected periods (e.g., 2019–2020).

For details on the DAX measures and calculated columns used in this dashboard, refer to the [DAX Measures and Columns](#meals--special-requests-1).

### Demography
![Demography](./images/Demography.png)

The "Demography" dashboard is an advanced analytical tool providing in-depth insights into hotel guest demographics. Its interactive geographical map features comprehensive tooltips displaying key metrics like total bookings revenue, guest count and average stay duration. The dashboard includes age group analysis and smart traveler type classification (solo, couple, family) based on sophisticated DAX measures. This solution enables multidimensional data analysis with quick switching between different perspectives.

For details on the DAX measures and calculated columns used in this dashboard, refer to the [DAX Measures and Columns](#demography-1).

### Marketing & Operations
![Marketing & Operations](./images/Marketing%20&%20Operations.png)

The "Marketing & Operations" dashboard focuses on analyzing the effectiveness of marketing strategies and operational processes in the hotel industry. It evaluates key performance indicators such as direct booking rate and returning guests rate to assess customer loyalty and campaign success. For operations, the dashboard explores booking lead time through a custom histogram, ingeniously created by modifying a clustered column chart in Power BI, which reveals patterns in how far in advance guests plan their stays. By integrating data on market segmentation, customer types, and temporal trends, this tool enables a holistic approach to optimizing business decisions.

For details on the DAX measures and calculated columns used in this dashboard, refer to the [DAX Measures and Columns](#marketing--operations-1).

### Data Loading
The dataset was loaded into **Power BI** from an **SQL Server** using the following query:
```WITH hotels AS (
SELECT * FROM dbo.[2018$]
UNION ALL
SELECT * FROM dbo.[2019$]
UNION ALL
SELECT * FROM dbo.[2020$])

SELECT * 
FROM hotels
LEFT JOIN dbo.market_segment$
ON hotels.market_segment = market_segment$.market_segment
LEFT JOIN dbo.meal_cost$
ON meal_cost$.meal = hotels.meal
WHERE is_canceled = 0
```

This query combines hotel booking data from three consecutive years (2018-2020) using the UNION ALL operator. It then enriches the data through:

1. LEFT JOIN with the market_segment$ table - to add market segment details
2. LEFT JOIN with the meal_cost$ table - to include meal cost information

The key element is results filtering (WHERE is_canceled = 0) which:
- Excludes canceled reservations (where is_canceled = 1)
- Keeps only completed stays (where is_canceled = 0)
- Ensures revenue analysis is based on actually delivered services

### DAX Measures and Columns
#### **Summary Dashboard**
* **Revenue Column**

```=([stays_in_week_nights]+[stays_in_weekend_nights])*([adr]-([adr]*[Discount]))```

This DAX measure calculates the total revenue per booking. Since the dataset didn't contain direct revenue figures but only included Average Daily Rate (ADR), the formula computes revenue by: 
1. Calculating the total stay duration (week nights + weekend nights)
2. Adjusting the ADR by applying any applicable discounts
3. Multiplying the adjusted ADR by the total nights stayed

Key components:

1. stays_in_week_nights: Number of weekday nights booked
2. stays_in_weekend_nights: Number of weekend nights booked  
3. adr: Average Daily Rate (base price before discounts)
4. Discount: Percentage discount applied to the rate (expressed as decimal)

* **Total Nights**

```= SUM(Query1[stays_in_week_nights]) + SUM(Query1[stays_in_weekend_nights])```

This simple DAX measure calculates the total number of nights stayed across all bookings by summing:
stays_in_week_nights: weekday stays (Monday–Friday),

stays_in_weekend_nights: weekend stays (Saturday–Sunday).

* **Parking Space Nights**

```
= SUMX(
    Query1,
    Query1[required_car_parking_spaces] * (Query1[stays_in_week_nights] + Query1[stays_in_weekend_nights]))
```

This DAX measure calculates total nights requiring parking spaces by:

1. Multiplying the number of required parking spaces (required_car_parking_spaces) for each booking
2. By the total length of stay (sum of weekday and weekend nights)

Use Cases:

1. Estimates total demand for parking spaces
2. Provides foundation for hotel parking capacity analysis
3. Can be used to calculate parking occupancy rate (when combined with actual parking capacity data)


* **Parking Percentage**

```= [Parking Space Nights] / [Total Nights] ```

This DAX measure calculates the percentage of stays that required parking spaces relative to all stays. It leverages two previously defined measures:

Parking Space Nights - number of stays with declared parking space requirement

Total Nights - total number of all stay nights

*Context:*
Since the dataset doesn't contain information about the actual number of parking spaces available at the hotel (no relevant columns or supporting documentation), this measure provides the best available approximation of guests' parking needs.

Use Cases:
1. Tracking changes in parking demand over time
2. Analyzing seasonality of parking requirements
3. Comparing different guest segments

#### **Meals & Special Requests**
* **Effective Daily Cost**

```
= SUMX(
    Query1,
    VAR TotalNights = Query1[stays_in_week_nights] + Query1[stays_in_weekend_nights]
    RETURN
        SWITCH(
            TRUE(),
            Query1[meal] = "Self catering", Query1[Cost],
            Query1[meal] = "Undefined", 0,
            Query1[Cost] * TotalNights
        )
)
 ```

 The measure calculates effective meal revenue based on meal type policies:
1. For Self Catering (e.g., kitchen access) – applies a one-time fee (value from [Cost] column),
2. For Undefined meal type – revenue is 0,
3. For other meal plans (e.g., Bed & Breakfast) – revenue is calculated proportionally to stay duration (daily cost × total nights: stays_in_week_nights + stays_in_weekend_nights).

* **Average Meal Cost per Reservation**

```
= AVERAGEX(
    Query1,
    VAR TotalNights = Query1[stays_in_week_nights] + Query1[stays_in_weekend_nights]
    RETURN
        SWITCH(
            TRUE(),
            Query1[meal] = "Self catering", Query1[Cost],
            Query1[meal] = "Undefined", 0,
            Query1[Cost] * TotalNights
        )
) 
```

The measure calculates the average meal revenue per reservation based on meal type policies:
1. For Self Catering (e.g., kitchen access) – applies a one-time fee (value from [Cost] column),
2. For Undefined meal type – revenue is 0,
3. For other meal plans (e.g., Bed & Breakfast) – revenue is scaled by stay duration (daily cost × total nights).

#### **Demography**

* **Guest Combination**

```
= VAR Adults = [adults]
VAR Children1 = [children]
VAR Babies = [babies]
RETURN
SWITCH(
TRUE(),
Adults = 1 && Children1 = 0 && Babies = 0, "Solo Traveler",
Adults = 2 && Children1 = 0 && Babies = 0, "Couple",
Adults >= 2 && Children1 > 0, "Family with Children",
Babies > 0, "Family with Baby",
"Other"
) 
```

Calculated column categorizing bookings by guest composition:

1. Solo Traveler – 1 adult, no children,
2. Couple – 2 adults, no children,
3. Family with Children – 2+ adults with children,
4. Family with Baby – includes infants,
5. Other – remaining combinations.

Note: Classification uses [adults], [children], and [babies] columns.

* **Family-Friendly Index**
```
= DIVIDE(
    COUNTROWS(FILTER(Query1, Query1[Children] > 0 || Query1[Babies] > 0)),
    COUNTROWS(Query1),
    0
)
```

Calculates the percentage of bookings involving children relative to total bookings:
1. Counts bookings where either Children or Babies count is greater than zero,
2. Returns a percentage value (e.g., 0.25 = 25%),
3. The 0 parameter guards against edge cases (e.g., empty filtered tables).

* **Average Length of Stay**
```
= DIVIDE(
    [Total Nights],
    COUNTROWS(Query1),
    0
)
```

Calculates the average guest stay duration by dividing total nights (Total Nights) by the number of bookings.

Total Nights sums stays_in_week_nights and stays_in_weekend_nights,

The trailing 0 prevents division-by-zero errors.

#### **Marketing & Operations**

* **Direct Booking Rate**
```
= DIVIDE(
    COUNTROWS(FILTER(Query1, Query1[distribution_channel] = "Direct")),
    COUNTROWS(Query1),
    0
)
```

Calculates the percentage of bookings made directly with the hotel (without intermediaries) using the distribution_channel column:

1. Counts records where distribution_channel = "Direct",
2. Divides by total bookings,
3. Returns a percentage value (e.g., 0.25 = 25%).

The 0 parameter safeguards against theoretical edge cases with empty datasets.

* **Returning Guests Rate**

```= DIVIDE(COUNTROWS(FILTER(Query1, Query1[is_repeated_guest] = 1)), COUNTROWS(Query1))```

Calculates the percentage of bookings made by returning guests using the binary column is_repeated_guest:
1. Counts records marked as 1 (True) - indicating returning guests,
2. Divides by total bookings,
3. Returns a percentage value (e.g., 0.15 = 15%).