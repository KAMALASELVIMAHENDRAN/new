
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



