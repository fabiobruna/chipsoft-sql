
```sql
Aantal bezoekers CY = 
    CALCULATE (
          [Aantal bezoekers],
          FILTER (
               ALL ( 'periode' ),
                     'periode'[jaar] = MAX ( 'DateTable'[jaar] )  
       )
    )

Aantal bezoekers CY = CALCULATE([Aantal bezoekers]; (('periode'[datum]));
          FILTER (
               ALL ( 'periode' ),
                     year('periode'[datum]) = year(now())
       )

)
```