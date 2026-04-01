
step 1: create Quarter and Year column in main table


Step 2: create 'quater-year table'
```
Quarter-year Table = DISTINCT (
    SELECTCOLUMNS (
        Sheet1,
        "quarter_year", Sheet1[quarter_year],
        "Year", Sheet1[year],
        "Quarter", Sheet1[Quarter]
    )
)
```

Step 3: Create New Column in 'quater-year table'
```
Quarter Sort Order = VALUE('Quarter-year Table'[Year]) * 10 +
SWITCH(
    'Quarter-year Table'[Quarter],
    "Q1", 1,
    "Q2", 2,
    "Q3", 3,
    "Q4", 4
)
```

Step 4: create parameter for changing 'year' and ;quarter-yr' from 'quater-year table' in slicer and add new slicer which contains only year and quarter-yr which changes when parametee applied.

Quarter parameter
```
Quarter_parameter = { 
    ("Yearly", NAMEOF('Quarter-year Table'[Year]), 0),
    ("Quarterly", NAMEOF('Quarter-year Table'[quarter_year]), 1)
}
```


aggsavings adjusted measure
```
Filtered_AggSavings = 
CALCULATE(
    SUM(gt_plus_oa_data[aggsavings]),
    FILTER(
        gt_plus_oa_data,
        (
            gt_plus_oa_data[dk_key] = "DK1" &&
            gt_plus_oa_data[rule] IN {1500,1600,1700,1800,1900}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK2" &&
            gt_plus_oa_data[rule] IN {1100,1200,1300,1400,1500}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK3" &&
            gt_plus_oa_data[rule] IN {1689,1717,1852,1885}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK5" &&
            gt_plus_oa_data[rule] IN {1200,1400,1600,1800,2000}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK10" &&
            gt_plus_oa_data[rule] IN {1124,1300,1500,1700}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK15" &&
            gt_plus_oa_data[rule] IN {2705,2600,2500}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK18" &&
            gt_plus_oa_data[rule] IN {2000,2200,2400,2600}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK20" &&
            gt_plus_oa_data[rule] IN {1900,2100,2300,2500,2780}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK23" &&
            gt_plus_oa_data[rule] IN {2000,2200,2400,2574}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK25" &&
            gt_plus_oa_data[rule] IN {1066,1200,1400,1600}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK30" &&
            gt_plus_oa_data[rule] IN {1500,1560,1700,1900}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK34" &&
            gt_plus_oa_data[rule] IN {2200,2400,2560}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK36" &&
            gt_plus_oa_data[rule] IN {1198,1238,1400}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK39" &&
            gt_plus_oa_data[rule] IN {1160,1300,1500,1534}
        )
        ||
        (
            gt_plus_oa_data[dk_key] = "DK40" &&
            gt_plus_oa_data[rule] IN {2000,2200,2400,2456}
        )
    )
)
```
cpt matching Measure
```
Filtered_AggSavings_CPT = 
CALCULATE(
    [Filtered_AggSavings],
    KEEPFILTERS(gtplus_cpt_list)
)
```


```
Selected_Qtr_AggSavings = 
VAR SelectedQuarter =
    SELECTEDVALUE(gt_plus_oa_data[quarter_year])
RETURN
IF(
    NOT ISBLANK(SelectedQuarter),
    CALCULATE(
        [Filtered_AggSavings-CPT],
        gt_plus_oa_data[quarter_year] = SelectedQuarter,
        REMOVEFILTERS(gt_plus_oa_data[Year])
    )
)
```

new column
```
Quarter_Index = 
VALUE(LEFT(gt_plus_oa_data[quarter_year],4)) * 10 +
SWITCH(
    RIGHT(gt_plus_oa_data[quarter_year],2),
    "Q1", 1,
    "Q2", 2,
    "Q3", 3,
    "Q4", 4
)
```
```
Prev_Qtr_AggSavings_Final =
VAR CurrentIndex =
    MAX(gt_plus_oa_data[Quarter_Index])
RETURN
CALCULATE(
    [Filtered_AggSavings_CPT],   -- uses rules + CPT filter

    -- shift to previous quarter
    FILTER(
        ALL(gt_plus_oa_data),
        gt_plus_oa_data[Quarter_Index] = CurrentIndex - 1
    )
)
```
difference percentage 
```
QoQ_% = 
IF(
    NOT ISBLANK([Prev_Qtr_Filtered_AggSavings]),
    DIVIDE(
        [Filtered_AggSavings] - [Prev_Qtr_Filtered_AggSavings],
        [Prev_Qtr_Filtered_AggSavings]
    )
)
```
or
```
QoQ_Difference =
VAR Curr = [Filtered_AggSavings_CPT]
VAR Prev = [Prev_Qtr_AggSavings_Final]
RETURN
IF(
    NOT ISBLANK(Prev),
    Curr - Prev
)
```



<img width="573" height="288" alt="image" src="https://github.com/user-attachments/assets/f7d3703a-b14a-41c7-86c4-84d364a40caf" />

<img width="612" height="285" alt="image" src="https://github.com/user-attachments/assets/014f69bd-ebba-4d8b-8e72-c241dc396db7" />

