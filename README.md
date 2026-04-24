# MIST4610 – GROUP PROJECT A5


## Group Information
**Group Name:** A-5
| Name | Role |
|------|------|
| Krish Sharma | Group Leader |
| Nila Karunakaran | Conceptual Modeler |
| Jude Zekra | SQL Writer |
| Winston Chung | Data Wrangler |
## Case Summary
Northern Outfitters is a rapidly growing online retail company that specializes in selling student-oriented lifestyle and technology accessories. Its product offerings include hoodies, water bottles, desk lamps, phone cases, keyboards, mouse pads, and backpacks. The company sources its inventory from external vendors and sells directly to consumers across the United States and Canada.

Currently, Northern Outfitters relies on Excel spreadsheets to manage its operational data, including inventory, sales, and vendor information. However, this approach has resulted in disorganized and inconsistent datasets, making it difficult to efficiently track operations, maintain data accuracy, and generate meaningful insights. As the company continues to expand, these limitations have become increasingly problematic, hindering scalability and decision-making.

## Conceptual Model
<img width="834" height="1319" alt="DM" src="https://github.com/user-attachments/assets/d2b39918-8ab1-41f5-ab0d-5ab68e316034" />

## Data Quality Assessment
### 1. SKU Case Standardization
**Issue:** SKUs appear in both uppercase and lowercase across both the Sales and Product Supplied sheets, leading to join failures.
| Raw Value | Cleaned Value |
|---|---|
| `sku-c-1012` | `SKU-C-1012` |
| `sku-u-1013` | `SKU-U-1013` |
| `sku-u-1001` | `SKU-U-1001` |

### 2. Sale Date Standardization
**Issue:** Dates are recorded in six different format including US and Canada date formats.
| Raw Value | Format | Cleaned Value |
|---|---|---|
| `10-11-2025` | MM-DD-YYYY (US) | `2025-10-11` |
| `24/09/2025` | DD/MM/YYYY (Canadian) | `2025-09-24` |
| `Oct 17 25` | Mon DD YY | `2025-10-17` |
| `October 5 25` | Month D YY | `2025-10-05` |
| `September 28 2025` | Month DD YYYY | `2025-09-28` |
| `10 Sep 2025` | DD Mon YYYY | `2025-09-10` |

### 3. Customer Information Parsing
**Issue:** The name, customer type, and country are combined into a single field, but need to be seperated using deliminators.
| Raw Value | customer_name | customer_type |
|---|---|---|
| `Grace Hall \| Student \| US` | `Grace Hall` | `Student` |
| `Emma Wilson; Loyalty? Y` | `Emma Wilson` | `Loyalty` |
| `Zoe Garcia / guest order` | `Zoe Garcia` | `Guest` |
| `Owen Clark` | `Owen Clark` | `Standard` |

### 4. Payment Method Standardization
**Issue:** Same payment methods appear with different casings and abbreviations.
| Raw Values | Cleaned Value |
|---|---|
| `VISA`, `visa` | `Visa` |
| `MC`, `Mastercard` | `Mastercard` |
| `Debit`, `debit` | `Debit` |
| `AMEX` | `Amex` |
| `Apple Pay`, `Interac`, `Cash` | unchanged |

### 5. Unit Price and Cost Split
**Issue:** The currency codes and numeric values are stored together in one field and some rows omit currency codes.
| Raw Value | price_amount | price_currency |
|---|---|---|
| `USD 18.99` | `18.99` | `USD` |
| `CAD 31.40` | `31.40` | `CAD` |
| `31.4`      | `31.40` | 'Unknown'|

### 6. Discount Standardizations 
**Issue:** Discounts are recorded in five differerent formats and require a uniform decimal rate.
| Raw Value | Meaning | Cleaned Value |
|---|---|---|
| `10%` | 10 percent | `0.10` |
| `5%` | 5 percent | `0.05` |
| `5` | 5 percent (no symbol) | `0.05` |
| `student 10%` | student promo 10% | `0.10` |
| `promo5` | promo code for 5% | `0.05` |
| `0` | no discount | `0.00` |
| `NULL` | no discount | `0.00` |

### 7. Tax Rate Standardization
**Issue:** Tax rates appear as percentages, decimals, and strings.
| Raw Value | Cleaned Value |
|---|---|
| `13%` | `0.13` |
| `8.25%` | `0.0825` |
| `7%` | `0.07` |
| `0.13` | `0.13` |
| `0.0825` | `0.0825` |
| `HST 13%` | `0.13` |
| `NULL` | `0.00` |

### 8. Line Total Cleaning 
**Issue:** The "line_total" is marked as a string, but sometimes includes values with dollar signs. There are also many "NULL" values in this column.
| Raw Value | Cleaned Value | Action |
|---|---|---|
| `$19.43` | `19.43` | Strip `$`, cast to decimal |
| `97.19` | `97.19` | Cast to decimal |
| `NULL` | computed | Calculate from price, quantity, discount, tax |

### 9. Ship Country Standardization
**Issue:** Two countries are represented by four different value.
| Raw Values | Cleaned Value |
|---|---|
| `US`, `USA` | `US` |
| `CA`, `Canada` | `CA` |
| `NULL` | inferred from `order_id` prefix |


### 10. Quantity Standardization
**Issue:** Some rows contain strings instead of integers
| Raw Value | Cleaned Value |
|---|---|
| `1` | `1` |
| `2` | `2` |
| `3` | `3` |
| `2 units` | `2` |

### 11. Format "Null" values
**Issue:** Rows conataining "NULL" values default to "N", which cause errors when running queries.
| Raw Value | Cleaned Value |
|---|---|
| `Y` | `Y` |
| `N` | `N` |
| `NULL` | `N` |

### 12. Duplicate SKU values
**Issue:** There are duplicant entries in different formats within the SKU column.
| sku (raw) | product_description | Action |
|---|---|---|
| `SKU-C-1002` | Aurora Mechanical Keyboard | Keep |
| `sku-c-1002` | Aurora Mechanical Keyboard Mini | Keep as variant |
| `SKU-C-1012` | Summit Laptop Stand | Keep |
| `sku-c-1012` | Summit Laptop Stand | Delete (true duplicate) |

### 13. Product Category Standardization 
**Issue:** There are 20 distinct category values that use incorrect and inconsistent delimiters.
| Raw Value | Cleaned Value |
|---|---|
| `Tech / Student`, `Tech & Student`, `Tech / Accessories` | `Tech` |
| `Audio / Student` | `Audio` |
| `Lifestyle , Student`, `Lifestyle / Student` | `Lifestyle` |
| `School / Student`, `School / Accessories` | `School` |
| `Apparel / Student`, `Student and apparels` | `Apparel` |
| `Desk Setup / Student`, `Desk Setup / Accessories` | `Desk Setup` |
| `Accessories / Student`, `Accessories / Accessories` | `Accessories` |

### 14. Vendor Phone and Representative Standardization
**Issue:** Phone numbers appear in three formats and the representative names contain notes and typos. 
| Raw Phone | Cleaned Phone |
|---|---|
| `404-555-0181` | `404-555-0181` |
| `604.555.0190` | `604-555-0190` |
| `(404)555-0181` | `404-555-0181` |

| Raw Rep Name | Cleaned Rep Name |
|---|---|
| `Jason Wu / email missing` | `Jason Wu` |
| `Ms. Anika Roy` | `Anika Roy` |
| `Mia Dia zFernandez` | `Mia Diaz` |

### 15. Reorder Level Standardization
**Issue:** The word "ten" is used instead of the integer "10" in some rows.
| Raw Value | Cleaned Value |
|---|---|
| `12` | `12` |
| `5` | `5` |
| `ten` | `10` |
| `NULL` | `NULL` |

### 16. Weight and Length Unit Standardization
**Issue:** Both columns contain mixed metric and imperial values with inconsistent formatting. The weights need to be converted to grams and the lengths to centimeters.
**Weight conversions:**

| Raw Value | Converted To (grams) |
|---|---|
| `8 ounces` | `226.80` |
| `1.1 pound` / `1.1 lb` / `1.1 lbs` | `498.95` |
| `0.22 kilograms` / `0.22kg` | `220.00` |
| `499 g` / `450g` / `340 grams` | unchanged |

**Length conversions:**

| Raw Value | Converted To (cm) |
|---|---|
| `10.2 in` / `10.2 inches` / `10.2"` | `25.91` |
| `12.6"` / `12.6 in` | `32.00` |
| `25.4 cm` / `25.4cm` | `25.40` |
| `26 centimetres` / `26cm` | `26.00` |

### 17. Country Code Extraction from Identifiers 
**Issue:** Country of origin for employees and orders is embedded in ID prefixes rather than stored as an explicit attribute.
| ID Value | Extracted country_code |
|---|---|
| `UORD-1041` | `US` |
| `CORD-1003` | `CA` |
| `EMU-202` | `US` |
| `EMC-403` | `CA` |
| `EMU-M03` | `US` |
| `EMC-M01` | `CA` |

## Data Cleaning Process
### 1. SKU Case Standardization
We standardized all SKU values by converting them to uppercase using Excel’s UPPER() function. This eliminated inconsistencies between lowercase and uppercase entries and ensured reliable joins across datasets. We also used conditional formatting to verify uniqueness.

### 2. Sale Date Standardization
We converted all date formats into a consistent ISO format (YYYY-MM-DD) using Excel’s date formatting tools. For ambiguous cases, we referenced country indicators to determine whether the format followed U.S. or Canadian conventions. This ensured consistency and accurate chronological analysis.

### 3. Customer Information Parsing
We split the customer_info field into separate attributes (customer name and customer type) using “Text to Columns” and delimiter-based formulas. Functions like TRIM() and LEFT() were used to clean and extract values. Missing classifications were assigned a default value of “Standard.”

### 4. Payment Method Standardization
We standardized payment methods by applying consistent naming conventions using PROPER() and find-and-replace logic. Variations such as “MC” and “Mastercard” were mapped to a single format. This ensured accurate grouping in payment analysis.

### 5. Unit Price and Cost Split
We separated currency codes from numeric values using functions like LEFT(), RIGHT(), and MID(). Numeric values were converted into decimal format, while missing currency codes were labeled as “Unknown.” This enabled proper calculations and standardization.

### 6. Discount Standardization
We converted all discount formats into a consistent decimal format using conditional formulas. Percentage symbols and text labels (e.g., “promo5”) were removed or interpreted accordingly. Missing values were standardized to 0.00 to ensure accurate pricing calculations.

### 7. Tax Rate Standardization
We standardized tax values by converting all formats into decimals using Excel formulas. Text entries such as “HST 13%” were parsed to extract the numeric portion. Null or missing values were replaced with 0.00 for consistency.

### 8. Line Total Cleaning
We removed symbols such as $ using SUBSTITUTE() and converted values into numeric format. For missing values, we recalculated line totals using quantity, unit price, discount, and tax. This ensured all rows had accurate financial data.

### 9. Ship Country Standardization
We standardized country values using find-and-replace (e.g., “USA” → “US”, “Canada” → “CA”). Missing values were inferred from order ID prefixes. This ensured consistency for geographic segmentation.

### 10. Quantity Standardization
We cleaned quantity values by removing text (e.g., “2 units”) using SUBSTITUTE() and converting all entries into integers. Invalid entries were corrected manually. This ensured quantities were usable in calculations.

### 11. Format "Null" Values
We standardized null values by replacing “NULL” entries using Excel’s find-and-replace function. Logical fields were assigned default values such as “N.” This prevented errors during import and analysis.

### 12. Duplicate SKU Values
We identified duplicates using Excel’s “Remove Duplicates” tool and conditional formatting. True duplicates were removed, while valid product variants were retained. This preserved data accuracy while eliminating redundancy.

### 13. Product Category Standardization
We consolidated inconsistent category labels using mapping logic and find-and-replace techniques. Variations with different delimiters or wording were grouped into a single standardized category. This improved reporting consistency.

### 14. Vendor Phone and Representative Standardization
We standardized phone numbers into a consistent format using text formatting and manual corrections. Representative names were cleaned by removing titles and notes using text functions. This ensured uniform vendor records.

### 15. Reorder Level Standardization
We converted text-based values such as “ten” into numeric format through manual replacement and validation. All values were stored as integers. Missing values were left as null where appropriate.

### 16. Weight and Length Unit Standardization
We converted all measurements into standard units (grams and centimeters) using Excel formulas and conversion factors. Text parsing was used to extract numeric values before applying conversions. This ensured consistency across product records.

### 17. Country Code Extraction from Identifiers
We extracted country codes from ID prefixes using functions such as LEFT(). These values were stored in separate columns for employees and orders. This allowed us to standardize geographic data independently of identifiers.

## Required Queries 
1. Which products generated the highest total sales revenue, by country
<img width="807" height="637" alt="image" src="https://github.com/user-attachments/assets/1c676e6c-5c9c-4ee1-b381-536a68f3fc32" />

2. Which employees handled the largest number of orders, and how do their results compare with other employees under the same manager
<img width="798" height="639" alt="image" src="https://github.com/user-attachments/assets/29501616-7dab-49f6-9c69-0bbd9c49cb8e" />

3. Which vendors supply products that appear in more than one category?
<img width="922" height="637" alt="image" src="https://github.com/user-attachments/assets/aed4b807-d8c2-49c8-9339-d97c1ca3b8be" />

## Creative Queries
4. What is the return rate by product category?
<img width="986" height="589" alt="image" src="https://github.com/user-attachments/assets/e131abf6-5888-49b5-893e-278459622c2f" />
Business Justification:

5. Which customers have spent the most and what is their loyalty status?
<img width="935" height="640" alt="image" src="https://github.com/user-attachments/assets/1e47d566-a85a-48b1-9da0-fbcc0de70565" />
Business Justification:

6. Which products have never been sold but are not discontinued?
7. <img width="786" height="444" alt="image" src="https://github.com/user-attachments/assets/8f36139e-9383-49b5-b33a-07f664dbdf54" />

Business Justification:
