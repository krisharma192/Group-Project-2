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
<img width="864" height="1319" alt="MIST DM PNG GitHub" src="https://github.com/user-attachments/assets/a8222fa2-be02-4071-931d-41179ad70ef7" />


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

## Queries
