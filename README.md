# NCR Ride Bookings Analysis — Power BI Dashboard

## 1. Project Overview

This project analyzes ride-booking operations in the National Capital Region (NCR) using **Power BI**. The dashboard explores booking performance, ride status, vehicle types, revenue, payment methods, cancellations, and customer/driver ratings.

The report was created from the NCR ride-bookings dataset. The workflow included:
1. Importing the raw CSV into Power BI.
2. Cleaning and preparing the data in **Power Query Editor**.
3. Applying the prepared data to the Power BI data model.
4. Creating calculations/measures using **DAX**.
5. Building report pages and visuals.
6. Adding slicers and navigation buttons for interactive analysis.

> **Important:** This README describes the data fields and report pages visible in the uploaded dataset and PBIX. Where a custom DAX formula is not recoverable from the report layout alone, it is marked as an example/verification item rather than claimed as the exact formula from the PBIX.

---

## 2. Business Problem

Ride-hailing companies need to understand not only how many rides are booked, but also how many are completed, cancelled, or left incomplete. They need to monitor revenue, customer experience, driver experience, and operational issues.

This dashboard is intended to help answer questions such as:

- How many bookings were recorded over time?
- What proportion of bookings reached each booking status?
- Which vehicle types contribute to booking value and distance?
- Which payment methods are used, and how much booking value is associated with them?
- What are the main reasons customers and drivers cancel rides?
- How do customers and drivers rate their ride experience?
- How do results change when a date filter is applied?

### Business objectives

- Monitor overall booking performance.
- Identify cancellation and incomplete-ride problems.
- Compare vehicle categories.
- Analyze booking value and ride distance.
- Understand customer and driver satisfaction.
- Support data-driven operational decisions.

---

## 3. Tools and Technologies

| Tool | Purpose |
|---|---|
| Power BI Desktop | Report development, data model, dashboard visuals |
| Power Query Editor | Data cleaning, transformation, and preparation |
| DAX (Data Analysis Expressions) | Measures and analytical calculations |
| CSV | Source data format |
| Power BI data model | Stores prepared data and supports filtering/calculation |

---

## 4. Dataset Description

**Source file:** `ncr_ride_bookings.csv`

The source CSV contains **150,000 rows and 21 columns**.

Each row represents a ride-booking record. The dataset contains booking details, ride status, locations, cancellation/incomplete information, monetary value, distance, ratings, and payment information.

### Data dictionary

| Column | Description |
|---|---|
| `Date` | Date on which the booking was recorded |
| `Time` | Booking time |
| `Booking ID` | Unique or identifying booking reference |
| `Booking Status` | Status such as Completed, Incomplete, Cancelled, or No Driver Found |
| `Customer ID` | Customer identifier |
| `Vehicle Type` | Vehicle category selected for the ride |
| `Pickup Location` | Ride pickup area |
| `Drop Location` | Ride destination area |
| `Avg VTAT` | Average Vehicle Time of Arrival, as provided by the dataset |
| `Avg CTAT` | Average Customer Time of Arrival, as provided by the dataset |
| `Cancelled Rides by Customer` | Indicator/count associated with customer cancellations |
| `Reason for cancelling by Customer` | Customer cancellation reason |
| `Cancelled Rides by Driver` | Indicator/count associated with driver cancellations |
| `Driver Cancellation Reason` | Driver cancellation reason |
| `Incomplete Rides` | Indicator/count for incomplete rides |
| `Incomplete Rides Reason` | Reason for an incomplete ride |
| `Booking Value` | Booking amount/value |
| `Ride Distance` | Distance associated with the ride |
| `Driver Ratings` | Rating given to the driver |
| `Customer Rating` | Rating associated with the customer/ride experience |
| `Payment Method` | Payment mode used for the booking |

### Understanding missing values

The dataset contains many blank values. These are not necessarily errors. For example:
- A completed ride may not have a cancellation reason.
- A booking that never found a driver may not have a booking value or ride distance.
- Ratings may be unavailable for cancelled or incomplete rides.

The correct treatment depends on the meaning of each column. Do **not** replace all blanks with zero automatically: a missing rating is not the same as a rating of zero.

---

## 5. Data Preparation in Power Query Editor

Power Query is the **ETL (Extract, Transform, Load)** layer of Power BI:
- **Extract:** connect to the CSV file.
- **Transform:** clean, reshape, and standardize the data.
- **Load:** apply the prepared table to the Power BI model.

### Step 5.1 — Import the CSV

1. Open Power BI Desktop.
2. Select **Home → Get Data → Text/CSV**.
3. Browse to `ncr_ride_bookings.csv`.
4. Preview the data.
5. Choose **Transform Data** instead of immediately loading it.

This opens Power Query Editor, where data preparation can be performed before the model is loaded.

### Step 5.2 — Inspect the data

In Power Query Editor:
1. Review the column headers.
2. Check the first few rows and sample values.
3. Identify blank values and unusual values.
4. Check for inconsistent text formatting.
5. Confirm that each field has the correct data type.

This initial inspection helps distinguish genuine missing information from formatting problems.

### Step 5.3 — Set appropriate data types

Data types are important because they control sorting, aggregation, filtering, and calculations.

Recommended types:

| Column | Recommended type |
|---|---|
| `Date` | Date |
| `Time` | Time |
| `Booking ID` | Text |
| `Customer ID` | Text |
| `Booking Status` | Text |
| `Vehicle Type` | Text |
| Pickup/drop locations | Text |
| Cancellation/incomplete reasons | Text |
| `Avg VTAT`, `Avg CTAT` | Decimal number |
| `Booking Value` | Decimal number or fixed decimal/currency |
| `Ride Distance` | Decimal number |
| Ratings | Decimal number |
| Cancellation/incomplete indicators | Whole number, if they are numeric flags |
| `Payment Method` | Text |

**Why IDs should be text:** Booking IDs and Customer IDs are identifiers, not quantities. They should not be summed or treated as numeric values. Text also preserves leading zeros if any exist.

**Why Date should be a date type:** A date type supports date slicers, chronological sorting, time-intelligence calculations, and date hierarchies.

### Step 5.4 — Clean text fields

Text cleaning may include:
- Removing leading/trailing whitespace.
- Removing unwanted quotation marks around IDs, if present.
- Standardizing inconsistent capitalization or spelling where appropriate.
- Ensuring blank strings are treated as null values when appropriate.

For example, a value such as `"CNR5884300"` may contain quote characters in the CSV representation. If the quotes are actual data characters rather than CSV syntax, remove them so the ID is stored consistently.

**Power Query operations:**
- Select text columns.
- Use **Transform → Format → Trim**.
- Use **Clean** to remove non-printable characters when needed.
- Use **Replace Values** only when the replacement is justified.

Avoid changing category labels without checking whether they represent distinct categories.

### Step 5.5 — Handle missing values

Review nulls column by column.

Possible handling:
- Keep missing cancellation reasons for rides that were not cancelled.
- Keep missing ratings where no rating was provided.
- Keep missing booking value/distance for records where a ride did not take place.
- For analytical visuals, filter to relevant statuses or nonblank values when appropriate.

Do not indiscriminately delete rows with nulls. Doing so could remove cancelled or unsuccessful bookings and distort the booking-status analysis.

Do not fill a null rating with 0 unless the business definition explicitly treats 0 as a valid rating.

### Step 5.6 — Check duplicates

Use **Home → Remove Rows → Remove Duplicates** only if duplicate records are confirmed.

A possible check is to inspect `Booking ID`:
- If each booking ID should identify one booking, duplicate IDs may need investigation.
- If repeated rows are valid in the source, removing them could incorrectly reduce totals.

Keep the original row count and compare it with the cleaned row count so the impact of transformations is understood.

### Step 5.7 — Validate numeric values

Check for:
- Negative booking values or distances.
- Ratings outside the expected scale.
- Unexpected values in cancellation/incomplete flags.
- Invalid dates or times.
- Values that were imported as text instead of numbers.

Do not remove outliers automatically. First determine whether they are data errors or legitimate unusual records.

### Step 5.8 — Rename queries/columns if needed

Clear, consistent names make the model easier to understand. The uploaded report uses the table name `ncr_ride_bookings`.

### Step 5.9 — Apply the transformations

Once the data is ready:
1. Select **Home → Close & Apply**.
2. Power BI applies the Power Query steps.
3. The prepared table becomes available in the data model.

Power Query transformations are repeatable. When the source is refreshed, Power BI can rerun the recorded steps.

---

## 6. Data Modelling

Data modelling defines how tables and fields are organized for analysis. It affects filter behavior, aggregation, and report performance.

### 6.1 — Load the prepared table

The report uses a table named:

`ncr_ride_bookings`

The table contains booking-level information such as booking status, date, vehicle, booking value, distance, ratings, and payment method.

### 6.2 — Verify column properties

In **Model view** or **Data view**:
- Set the correct data type.
- Set `Date` as the date field.
- Set `Booking ID` and `Customer ID` as text.
- Set `Booking Value` to a currency/decimal format.
- Set `Ride Distance` to a suitable decimal format.
- Set ratings to a decimal format.
- Set the summarization of ID fields to **Do not summarize**.

This prevents Power BI from performing inappropriate operations such as summing IDs.

### 6.3 — Date hierarchy and date filtering

The report includes date slicers and visuals using a date hierarchy (Year, Quarter, Month, Day).

Power BI can automatically create a date hierarchy when Auto date/time is enabled. For a more robust production model, a dedicated Date table is recommended.

Example Date table DAX:

```DAX
DateTable =
CALENDAR(
    MIN(ncr_ride_bookings[Date]),
    MAX(ncr_ride_bookings[Date])
)
```

Additional Date table columns:

```DAX
Year = YEAR(DateTable[Date])

Month Number = MONTH(DateTable[Date])

Month Name = FORMAT(DateTable[Date], "MMM")

Quarter = "Q" & FORMAT(DateTable[Date], "Q")
```

Then:
1. Mark `DateTable` as a Date table.
2. Create a relationship from `DateTable[Date]` to `ncr_ride_bookings[Date]`.
3. Sort Month Name by Month Number.
4. Use the Date table in slicers and time-based visuals.

**Note:** This is a recommended model improvement. Verify whether the existing PBIX uses a dedicated Date table or Power BI's automatic hierarchy before claiming that it was implemented.

### 6.4 — Relationships

The report layout indicates the main visuals use fields from `ncr_ride_bookings`. If the model contains only this table, a relationship is not required for those visuals.

If adding dimension tables in a future version, a star schema is recommended:

- **Fact table:** ride bookings
- **Date dimension:** calendar attributes
- **Vehicle dimension:** vehicle categories
- **Location dimension:** pickup/drop locations
- **Payment dimension:** payment methods, if needed

A star schema separates descriptive categories from transactional records and can make the model easier to maintain.

---

## 7. DAX and Data Modelling Measures

**DAX (Data Analysis Expressions)** is the formula language used in Power BI for measures, calculated columns, and calculated tables.

### 7.1 — Measure vs calculated column

| Measure | Calculated column |
|---|---|
| Evaluated when a visual/query needs it | Calculated row by row during refresh |
| Responds to slicers and filters | Stored in the model |
| Usually used for totals, ratios, averages, KPIs | Usually used for row-level classifications or attributes |
| Does not create a stored value for every row | Adds a value for each row |

For dashboard KPIs, measures are generally preferable because they respond dynamically to report filters.

### 7.2 — Example measures

The PBIX includes a custom measure named `CanceledPercentage`. The following are illustrative DAX patterns for common booking metrics. Confirm their definitions against the formulas in your own PBIX before treating them as exact project measures.

#### Total Bookings

```DAX
Total Bookings =
COUNT(ncr_ride_bookings[Booking ID])
```

Counts booking IDs. If booking IDs can be duplicated, use `DISTINCTCOUNT` instead:

```DAX
Unique Bookings =
DISTINCTCOUNT(ncr_ride_bookings[Booking ID])
```

#### Completed Rides

```DAX
Completed Rides =
CALCULATE(
    [Total Bookings],
    ncr_ride_bookings[Booking Status] = "Completed"
)
```

`CALCULATE` modifies the filter context. This measure counts bookings only where the status is Completed.

#### Booking Value / Revenue

```DAX
Total Booking Value =
SUM(ncr_ride_bookings[Booking Value])
```

Adds booking values within the current filter context. For example, selecting a date or vehicle type changes the result.

> **Metric definition:** Booking Value is the source field. Calling it “revenue” assumes that the business defines this amount as recognized revenue. In a real business report, revenue may require additional adjustments.

#### Average Booking Value

```DAX
Average Booking Value =
AVERAGE(ncr_ride_bookings[Booking Value])
```

Calculates the average of nonblank booking values. It is not necessarily the average across all booking records if unsuccessful bookings have no value.

#### Total Ride Distance

```DAX
Total Ride Distance =
SUM(ncr_ride_bookings[Ride Distance])
```

Adds recorded ride distances.

#### Average Ride Distance

```DAX
Average Ride Distance =
AVERAGE(ncr_ride_bookings[Ride Distance])
```

Calculates average distance among records with a nonblank distance.

#### Cancellation Percentage — illustrative definition

```DAX
Cancellation Percentage =
DIVIDE(
    CALCULATE(
        [Total Bookings],
        ncr_ride_bookings[Booking Status] = "Cancelled"
    ),
    [Total Bookings],
    0
)
```

Format this measure as Percentage.

**Important:** This formula assumes the cancellation numerator should include only records with status `Cancelled`. If the report includes customer and driver cancellation flags separately, the numerator and denominator should match the project's agreed definition.

#### Average Customer Rating

```DAX
Average Customer Rating =
AVERAGE(ncr_ride_bookings[Customer Rating])
```

#### Average Driver Rating

```DAX
Average Driver Rating =
AVERAGE(ncr_ride_bookings[Driver Ratings])
```

These average only nonblank rating values.

### 7.3 — Filter context

A measure is calculated according to the filters active in the report.

For example:
- Selecting a date filters bookings to that date/range.
- Selecting a vehicle type filters bookings to that vehicle.
- Selecting a payment method filters booking value to that payment mode.

This behavior is what makes DAX measures useful for interactive dashboards.

### 7.4 — Format measures

In Power BI:
- Booking value: currency or decimal with suitable units.
- Distance: decimal with a distance unit if known.
- Ratings: decimal, typically one or two decimal places.
- Percentages: Percentage format.
- Booking counts: whole number.

Formatting makes KPI cards and charts easier to interpret.

---

## 8. Dashboard Pages and Visualizations

The uploaded PBIX contains these report pages:

1. Home Page
2. Overall
3. Vechical Type (report page spelling)
4. Renvenue (report page spelling)
5. Cancellation
6. Ratings

The page names below are retained as they appear in the PBIX. They can be renamed to `Vehicle Type` and `Revenue` for polished presentation.

### 8.1 — Home Page

The Home Page acts as a landing/navigation page.

Purpose:
- Introduce the report.
- Provide navigation to the analytical pages.
- Help users move between report sections.

The PBIX uses action buttons. Power BI buttons can be configured with page navigation, bookmarks, or other actions.

### 8.2 — Overall

This page provides a high-level overview.

Visuals detected in the PBIX:
- Line chart using booking count over the Date hierarchy.
- Pie chart showing Booking Status distribution.
- Date slicer.
- KPI/card visual.

Questions answered:
- How does booking volume change over time?
- What is the distribution of booking statuses?
- How do results change over the selected date range?

Interpretation:
- A rise in booking count indicates increased booking activity.
- A large share of cancelled/incomplete/no-driver bookings can indicate operational friction.
- Date filtering allows users to focus on a period and compare trends.

### 8.3 — Vehicle Type

This page compares booking value and ride distance across vehicle categories.

Visuals detected in the PBIX:
- Multiple KPI/card visuals using Booking Value.
- Multiple KPI/card visuals using Ride Distance.
- Date slicer.
- Navigation/action buttons.

Questions answered:
- How does booking value vary by vehicle type?
- Which vehicle categories contribute more recorded distance?
- Does vehicle performance change over time?

Important: The PBIX layout contains multiple cards tied to Booking Value and Ride Distance. Confirm the category-specific filters/interactions on each card in Power BI, because a card using the same field may show the same total if it is not filtered by vehicle type.

### 8.4 — Revenue

This page focuses on booking value and payment methods.

Visuals detected in the PBIX:
- Column chart using Booking Value by Payment Method.
- Column chart using Ride Distance by Date hierarchy.
- Table with Customer ID and Booking Value.
- Date slicer.
- Navigation/action buttons.

Questions answered:
- Which payment methods account for the largest booking value?
- How does ride distance change over time?
- Which customer records are associated with booking value?

Note: The table is at customer-level detail. For a business-facing dashboard, consider using aggregated customer metrics or anonymized IDs rather than exposing identifiers unnecessarily.

### 8.5 — Cancellation

This page explores cancellation behavior and its causes.

Visuals detected in the PBIX:
- Pie chart for customer cancellation reasons.
- Pie chart for driver cancellation reasons.
- Date slicer.
- KPI cards.
- A custom measure named `CanceledPercentage`.
- Navigation/action buttons.

Questions answered:
- What reasons are most frequently associated with customer cancellations?
- What reasons are most frequently associated with driver cancellations?
- What is the cancellation percentage under the selected filters?

Interpretation:
- Frequently occurring reasons may indicate opportunities to improve driver availability, pickup accuracy, wait time, or customer communication.
- Customer and driver cancellations should be analyzed separately because their causes and possible interventions differ.

### 8.6 — Ratings

This page analyzes customer and driver ratings.

Visuals detected in the PBIX:
- Multiple card visuals using Customer Rating.
- Multiple card visuals using Driver Ratings.
- Navigation buttons.

Questions answered:
- What rating values are present?
- How do customer and driver ratings compare?
- Do ratings change under the applied filters?

For analytical reporting, average ratings are generally more meaningful than summing rating values. Verify that rating cards use an Average aggregation or an explicit DAX average measure rather than Sum.

---

## 9. Dashboard Interactivity

### Date slicers

The report includes date slicers on several pages.

A slicer is a report-level visual control that filters other visuals on the same page, depending on the interaction settings.

Example:
1. Select a date range.
2. Booking counts, booking value, ride distance, and other visuals update.
3. Clear the slicer to return to the broader period.

### Cross-filtering and cross-highlighting

When users select a chart category, other visuals may update to show related records. This is controlled by **Format → Edit interactions**.

Verify the desired behavior for each visual:
- Filter: restrict other visuals to selected records.
- Highlight: emphasize the selected portion.
- None: prevent the visual from affecting another visual.

### Navigation buttons

The Home Page and report pages use buttons/actions. These can provide a guided navigation experience and make the report easier to use.

---

## 10. Suggested Analysis and Business Insights

Use the dashboard to investigate the following:

### Booking performance
- Compare booking volume over time.
- Identify periods with unusually high or low bookings.
- Compare completed bookings against unsuccessful booking statuses.

### Vehicle performance
- Compare booking value and ride distance across vehicle categories.
- Look for categories with strong booking activity but lower completion rates.
- Use date filters to evaluate changes.

### Revenue and payment behavior
- Compare booking value by payment method.
- Check whether booking value trends align with ride distance trends.
- Investigate unusually high or low values before making conclusions.

### Cancellation analysis
- Identify the most common customer cancellation reasons.
- Identify the most common driver cancellation reasons.
- Consider whether wait time, driver availability, or pickup-related problems may be contributing factors.

### Customer and driver experience
- Compare average customer and driver ratings.
- Investigate rating changes by date or vehicle type if the model supports those filters.
- Avoid drawing conclusions from very small sample sizes.

**Do not report specific numerical findings without checking the final dashboard values.** The README explains what the visuals are designed to investigate; it does not invent performance results.

---

## 11. Recommended Validation Checklist

Before publishing the dashboard, verify:

- [ ] Booking ID is treated as an identifier, not a numeric measure.
- [ ] Date is a Date data type.
- [ ] Booking Value and Ride Distance are numeric.
- [ ] Ratings use Average rather than Sum where appropriate.
- [ ] Cancellation percentages use a clearly defined numerator and denominator.
- [ ] Blank values are handled according to business meaning.
- [ ] Date slicers filter the intended visuals.
- [ ] Vehicle cards are correctly filtered by vehicle type.
- [ ] Payment-method chart uses the correct aggregation.
- [ ] Duplicate bookings have been investigated.
- [ ] Visual titles and axis labels are understandable.
- [ ] Report page spelling has been corrected if desired.
- [ ] Customer IDs are not exposed unnecessarily.
- [ ] Dashboard totals reconcile with the source data under the same filters.

---

## 12. Limitations and Future Improvements

Potential improvements:
- Create a dedicated Date dimension.
- Build a star schema if the model grows.
- Add explicit measures for total bookings, completed rides, cancellation rate, completion rate, average booking value, and average ratings.
- Add completion and cancellation KPIs to the Overall page.
- Add vehicle-type comparisons using charts rather than many similar cards.
- Add a map visual for pickup/drop locations if location data is suitable.
- Add time-of-day and weekday analysis.
- Add a decomposition tree or drill-through page for cancellation reasons.
- Add dynamic titles that reflect slicer selections.
- Improve accessibility with clear labels, alt text, and consistent color usage.
- Add a refresh date and report description.
- Use aggregated customer analysis instead of displaying customer identifiers.

---

## 13. Project Workflow Summary

```text
NCR Ride Bookings CSV
        |
        v
Power BI Desktop — Get Data
        |
        v
Power Query Editor
  - Inspect data
  - Set data types
  - Clean text
  - Review nulls
  - Validate records
        |
        v
Close & Apply
        |
        v
Power BI Data Model
  - Table/column properties
  - Date hierarchy
  - Measures using DAX
        |
        v
Report Development
  - Overall
  - Vehicle Type
  - Revenue
  - Cancellation
  - Ratings
        |
        v
Slicers + Navigation + Interactions
        |
        v
NCR Ride Bookings Dashboard
```

---

## 14. How to Open the Project

1. Download or clone this repository.
2. Open `power bi dashboard.pbix` using Power BI Desktop.
3. If Power BI asks for the source path, select `ncr_ride_bookings.csv`.
4. Select **Refresh** if necessary.
5. Review the report pages and interact with date slicers and navigation buttons.

### Suggested repository structure

```text
NCR-Ride-Bookings-PowerBI/
├── README.md
├── ncr_ride_bookings.csv
└── power bi dashboard.pbix
```

---

## 15. Conclusion

This Power BI project demonstrates a complete analytics workflow using a ride-bookings dataset: data preparation in Power Query, model configuration, DAX-based calculations, and interactive report design.

The dashboard organizes booking operations into overview, vehicle, revenue, cancellation, and ratings pages. It can support exploration of ride demand, booking value, cancellation causes, payment behavior, and customer/driver experience.

The most important analytical principle is to ensure that every KPI has a clear business definition and that filters, missing values, and aggregations are handled consistently.

---

## Author

**Project:** NCR Ride Bookings Analysis  
**Tool:** Microsoft Power BI  
**Dataset:** NCR Ride Bookings CSV
