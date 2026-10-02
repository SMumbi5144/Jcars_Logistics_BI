# JCARS LOGISTICS ANALYSIS

A Power BI business intelligence solution built from a raw, unprepared vehicle sales dataset for JCars Logistics, a company that imports, sells and delivers vehicles across Kenya. The project covers data cleaning, currency standardization, data modelling, DAX measures and a multi-page interactive dashboard designed to support management decision-making.

## Project Objective

Management needed a way to understand overall business performance and identify areas that require attention. The solution needed to cover sales and revenue, costs and profitability, vehicle performance, branch and regional performance, sales representatives, lead sources, payments, deliveries and logistics, returns and cancellations, customer experience and unusual transactions worth investigating.

## Dataset

The source data was a single raw flat file with 32 columns and 276 rows. One row represents one vehicle sold in one sales transaction. The data had not been prepared for analysis and contained a wide range of quality issues that needed to be investigated and resolved before any reliable reporting could be built on top of it.



## Data Quality Issues Investigated

More than ten significant data quality issues were found and resolved, including:

- Order IDs recorded in four different formats, standardized to `ORD-####`
- Dates in more than five formats, including Excel serial numbers and invalid calendar dates such as `31/02/2026`
- Implausible customer ages such as `0`, `5` and `121`
- Five different currencies mixed across the monetary columns, including a corrupted symbol that had to be investigated and confirmed as USD
- Ten or more spelling, casing and abbreviation variations across most categorical columns
- Zero values in monetary columns that meant different things depending on the column and the transaction's payment or delivery status
- Invalid Vehicle Year entries, including an Excel date artifact (`1899`) and a typo (`202A`)
- An outlier discount of `120%`
- Ratings recorded as text (`"4.7 out of 5"`), words (`"Excellent"`) and out-of-range values


---

## Currency Standardization

All monetary values were converted to Kenya Shillings (KES), the company's home currency, using the following exchange rates:

| Currency | Rate to KES |
|---|---|
| USD | 129.54 |
| EUR | 147.84 |
| ZAR | 7.93 |

Values without a stated currency were treated as KES. A corrupted currency symbol found in some rows was investigated by comparing the affected values against the Unit Cost on the same row and was confirmed to represent USD before conversion.

---

## Data Model

The raw flat file was restructured into a star schema with one fact table and eight dimension tables.

**FactSales** (grain: one row per vehicle sold in one transaction) relates to:

- `DimDate` (twice, via Order Date and Delivery Date)
- `DimCustomer`
- `DimVehicle`
- `DimLocation` (Region, County, City, Branch)
- `DimSalesRep`
- `DimPayment` (Payment Method and Payment Status)
- `DimLeadSource`
- `DimDeliveryStatus`

All relationships are One-to-Many with single-direction cross-filtering.

**Known limitation:** the dataset has no unique customer identifier, so `DimCustomer` was built from distinct combinations of Customer Name, Type and Age. Customer-level findings should be read with this in mind.

---

## Key DAX Measures

```dax
Total Revenue = SUM(FactSales[RevenueRecorded])

Total Gross Profit =
SUMX(FactSales, FactSales[RevenueRecorded] - (FactSales[UnitsSold] * FactSales[UnitCost]))

Gross Profit Margin % = DIVIDE([Total Gross Profit], [Total Revenue])

Return Rate % =
DIVIDE(
    CALCULATE(COUNTROWS(FactSales), FactSales[Returned] = "Yes"),
    COUNTROWS(FactSales)
)

Logistics Cost % of Revenue = DIVIDE([Total Logistics Cost], [Total Revenue])
```

Gross Profit is defined as Revenue minus (Units Sold times Unit Cost). Delivery Fee and Logistics Cost are treated as separate operating expenses rather than part of the vehicle's direct cost.

---

## Dashboard Structure

| Page | Purpose |
|---|---|
| Executive Dashboard | KPIs, revenue trend, branch and make comparison, payment mix and a logistics cost attention indicator |
| Vehicle/Branch Performance | Make, Model, Vehicle Type and Year performance, branch revenue share and a Region to Branch drill-down |
| SalesRep/LeadSource Performance | Sales representative ranking and lead source value |
| Revenue/Logistics | Payment status and method, delivery performance and logistics cost efficiency |
| TopCustomers | Top 10 customers by revenue and the relationship between revenue and customer ratings |

The report includes synced Region and Year slicers, a Region to County to City to Branch drill-down and cross-filtering between visuals.


---

## Key Insights

1. Revenue is concentrated in a small number of vehicle makes and regions, with Toyota and the Rift Valley and Central regions contributing a disproportionate share of the KSh 1.48bn total.
2. The top 10 customers out of 276 account for roughly 20% of total revenue, a moderate but meaningful concentration.
3. Trucks, Vans, Sedans and Crossovers are being sold at negative gross margins despite meaningful sales volumes, while SUVs generate the highest revenue with a positive margin.
4. No clear relationship was found between customer rating and transaction value.
5. Nairobi HQ has the highest logistics cost share and the longest average delivery time of any branch, despite being the company's headquarters.

---

## Management Recommendations

1. Investigate pricing and cost structure for Trucks, Vans, Sedans and Crossovers before continuing to sell at current terms.
2. Review the 10 transactions marked as Paid where the delivery was cancelled, for possible unresolved refunds.
3. Investigate the operational cause of Nairobi's slower delivery times and higher logistics cost share.


## Tools Used

- Power Query for data cleaning and transformation
- Power BI Desktop for data modelling, DAX and dashboard design
- Git for project management

---

## Notes

Monetary values, exchange rates and category standardizations reflect assumptions made during this project and are documented in full, along with their reasoning, in the project documentation article. Where the available data did not provide enough evidence to confidently resolve an issue, the value was left as missing rather than estimated.

##  Project Article

Read the full project walkthrough on Dev.to:

[From Messy CSV to Boardroom-Ready: Cleaning, Modelling and Visualizing Data in Power BI]( https://dev.to/smumbi_/from-messy-csv-to-boardroom-ready-cleaning-modelling-and-visualizing-jcars-logistics-sales-data-11ni)
