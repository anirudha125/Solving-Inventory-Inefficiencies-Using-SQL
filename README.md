# Solving-Inventory-Inefficiencies-Using-SQL

This project presents a comprehensive inventory analysis system using SQL views, built on a normalized relational schema. It provides data-driven insights for stock control, turnover efficiency, pricing competitiveness, and product demand classification.

## Project Structure

- `0.schema_data.sql`: Defines the normalized relational database schema.
- `1.analysis_compiled.sql`: SQL views for ABC classification, inventory turnover, reorder calculations, and pricing analysis.

## Database Schema Overview

The database schema `inventory_project_sql` is constructed from the `inventory_forecasting.csv` dataset. It includes:

- **DimProduct**: Contains `ProductID`, `Category`.
- **DimStore**: Contains `StoreID`, `Region`, and `StoreSK`.
- **DimDate**: Contains `PDate`, `PYear`, `PMonth`, `PDay`, `PDayOfWeek`, `Seasonality`.
- **FactInventorySales**: Contains `InventoryLevel`, `UnitsSold`, `UnitsOrdered`, `DemandForecast`, `Price`, and `Discount`.

The data is initially loaded into `temp_inventory_data`, and then normalized into these tables.

## Key SQL Views

- `ViewABCAnalysis`: Categorizes products into A/B/C based on revenue contribution.
- `ViewInventoryTurnover`: Computes `InventoryTurnoverRatio` and `DaysInInventory`.
- `ViewCurrentStockLevels`: Provides current inventory levels.
- `ViewReorderPoints`: Calculates reorder point using average demand and safety stock.
- `ViewProductMovementClassification`: Classifies products as fast, medium, or slow moving.
- `ViewInventoryActions`: Recommends actions like "URGENT REORDER", "REDUCE", or "MAINTAIN".
- `ViewStockMovementTrends`: Analyzes average, minimum, and maximum stock trends over 30 days.
- `ViewProductDailySales`: Daily metrics on sales, inventory, and forecast.
- `ViewProductAverageDailySales`: Reports average sales and forecast accuracy.
- `ViewCompetitorPricingImpact`: Compares pricing with competitors and recommends adjustments.
- `ViewSeasonalDemandAnalysis`: Detects seasonal sales trends.
- `weather_stats`: Groups turnover and demand by weather condition.

## ABC Analysis

**Purpose**: Categorize products into A/B/C by cumulative revenue contribution.

**Thresholds**:
- A: Top 85%
- B: Next 15%
- C: Remaining

**Example Query**:
```sql
SELECT * FROM ViewABCAnalysis WHERE ProductID LIKE 'P0016';
```

**Suggested Actions**:
- **A items**: Tight control, frequent monitoring.
- **B items**: Standard control.
- **C items**: Minimal control or delisting.

## Inventory Turnover Analysis

**Purpose**: Evaluate how quickly products are sold relative to average inventory.

**Metrics**:
- Inventory Turnover Ratio = TotalUnitsSold / AvgInventoryLevel
- Days In Inventory = AvgInventoryLevel / (TotalUnitsSold / TimePeriod)

**Sample Query**:
```sql
SELECT * FROM ViewInventoryTurnover WHERE StoreID LIKE 'S001' AND Region LIKE 'North' LIMIT 5;
```

## Stock Recommendations

**Logic**:
- Safety Stock (SS): `SS = z * σ`, where `z = 1.65` and `σ = StdDev(DailyDemand)`
- Reorder Point (ROP): `ROP = D * L + SS`, simplified to `D + SS` for `L = 1`

**SQL Implementation**:
```sql
CEIL(COALESCE(1.65 * STDDEV(UnitsSold), 0)) AS SafetyStock
FLOOR(AVG(UnitsSold) + COALESCE(1.65 * STDDEV(UnitsSold), 0)) AS ReorderPoint
```

**Suggested Actions**:
- Immediately reorder items with "URGENT REORDER"
- Reduce stock for slow-movers flagged with "REDUCE"
- Review reorder logic periodically
- Use `DaysOfStockRemaining` and `MovementCategory` for action prioritization

## Competitor Pricing Impact

**Purpose**: Compare internal pricing with competitors to adjust strategy.

**Example Query**:
```sql
SELECT * FROM ViewCompetitorPricingImpact WHERE PricingRecommendation LIKE 'Reduce Price';
```

**Logic**:
Recommend price reduction if:
- Units sold when priced higher < 80% of when priced lower
- Price is above competitor average

**SQL Snippet**:
```sql
CASE
  WHEN AVG(CASE WHEN Price > CompetitorPricing THEN UnitsSold ELSE NULL END)
     < AVG(CASE WHEN Price <= CompetitorPricing THEN UnitsSold ELSE NULL END) * 0.8
     AND AVG(Price - CompetitorPricing) > 0
  THEN 'Reduce Price'
  ELSE 'Monitor Competitor Response'
END AS PricingRecommendation
```

**Suggested Actions**:
- Consider reductions when likely to boost volume
- Monitor and update pricing for items flagged as competitive or neutral

## Concluding Remarks

This project uses SQL-based analytics to guide inventory decisions across stores. While it enables thorough analysis of historical data, some limitations include:

- No real-time data integration
- Static date ranges requiring manual update
- Limited automation due to hardcoded thresholds
- No dashboard or visual frontend
- Recommendations are not validated against future data

## Author

Anirudha Thakur

