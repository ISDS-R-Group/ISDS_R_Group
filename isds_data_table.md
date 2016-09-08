isds_data_table
========================================================
author: Eric Bakota, HHD
date: 2/18/16

Overview of data.table
========================================================


- Created by Matt Dowle
- Very very fast (10x-500x faster than base R)
- Entirely in memory
- data.tables are also classed as data.frames

Overview of data.table (cont'd)
========================================================


- Syntax is less intuitive than dplyr
- Can't call against a SQL server without loading tables entirely into memory
- Reads data by using an index key that is sorted (similar to how phonebooks are sorted by last name)

Creating a DT
========================================================


```r
library(data.table)
library(dplyr)

(DT <- data.table(Disease = c("Ebola","Zika")
            ,Age = c(30,5,8,15,22,44)
            ,INV_TIME = c(21,21,15,51,23,6)
            ,State = "TX"))
```

```
   Disease Age INV_TIME State
1:   Ebola  30       21    TX
2:    Zika   5       21    TX
3:   Ebola   8       15    TX
4:    Zika  15       51    TX
5:   Ebola  22       23    TX
6:    Zika  44        6    TX
```


Syntax
========================================================


- Syntax parallels SQL syntax
- DT[i, j, by]
 + i = WHERE
 + j = SELECT
 + by = GROUP BY


```r
DT[Age > 10, .(INV_TIME = mean(INV_TIME)), by = .(Disease)]
```

```
   Disease INV_TIME
1:   Ebola     22.0
2:    Zika     28.5
```

- <i> Take DT, subset rows using i, then calculate j grouped by by.</i>

Syntax in dplyr
========================================================

```r
DT[Age > 10, .(INV = mean(INV_TIME)), by = .(Disease)]
```

```
   Disease  INV
1:   Ebola 22.0
2:    Zika 28.5
```


```r
DT %>% filter(Age > 10) %>% group_by(Disease) %>% summarize(INV = mean(INV_TIME))
```

```
Source: local data table [2 x 2]

  Disease   INV
    (chr) (dbl)
1   Ebola  22.0
2    Zika  28.5
```


Syntax DT[i,,]
========================================================
- i is pretty straightforward


```r
DT[Age > 10]
DT[2:3]
DT[c(1,3,5)]
DT[c(1,3,5),,]
DT[.N]
```

- .N is equal to nrow(DT)

```r
DT[.N]
```

```
   Disease Age INV_TIME State
1:    Zika  44        6    TX
```

Syntax DT[i,,] (cont'd)
========================================================

```r
DT[ Age > 10]
```

```
   Disease Age INV_TIME State
1:   Ebola  30       21    TX
2:    Zika  15       51    TX
3:   Ebola  22       23    TX
4:    Zika  44        6    TX
```

```r
DT %>% 
  filter(Age > 10)
```

```
   Disease Age INV_TIME State
1:   Ebola  30       21    TX
2:    Zika  15       51    TX
3:   Ebola  22       23    TX
4:    Zika  44        6    TX
```

Syntax DT[,j,]
========================================================
- somewhat analogous to base R snytax of DF[r,c]
- uses new syntax of .(), which is an alias to list()


```r
DT[,Age,] # returns a vector
```

```
[1] 30  5  8 15 22 44
```

```r
DT[,.(Age),] #returns a DF/DT
```

```
   Age
1:  30
2:   5
3:   8
4:  15
5:  22
6:  44
```

Syntax DT[,j,] (cont'd)
========================================================
- data.table will also 'summarize' columns
- structured syntax allows for easy commenting

```r
DT[,                      # unused i column
   .(INV = mean(INV_TIME)    # j1 
   ,Age)                    # j2     
   ,]                     # by column
```

```
        INV Age
1: 22.83333  30
2: 22.83333   5
3: 22.83333   8
4: 22.83333  15
5: 22.83333  22
6: 22.83333  44
```

Syntax DT[,j,] (cont'd)
========================================================
- you can pass functions into j

```r
DT[,plot(Age,INV_TIME),]
```
![plot of chunk unnamed-chunk-11](isds_data_table-figure/unnamed-chunk-11-1.png) 

Syntax DT[,,by] 
========================================================
- works very similar to the group_by() verb in dplyr

```r
### same output
DT[,.(INV = mean(INV_TIME)), by = .(Disease)]
DT %>% 
  group_by(Disease) %>% 
  summarize(INV = mean(INV_TIME))
```


```
   Disease      INV
1:   Ebola 19.66667
2:    Zika 26.00000
```

Syntax DT[,,by] (cont'd)
========================================================
- can also pass functions to by

```r
DT[,.(INV=mean(INV_TIME)), by = .(Age > 10)]
```

```
     Age   INV
1:  TRUE 25.25
2: FALSE 18.00
```

Intermediate and advanced topics
========================================================
- Chaining 
 + You can chain several DT operations together
  + DT[operation1][operation2]
- Subset Data (.SD)
 + DT[, lapply(.SD, mean), by = Disease]


```r
DT <- data.table(A = c("A","A","A","B"), B = 1:99, C = 101:199, D = -50:49)
DT[, lapply(.SD, length), by = A]
```

```
   A  B  C  D
1: A 75 75 75
2: B 25 25 25
```

Intermediate and advanced topics (cont'd)
========================================================
- using := to replace by reference

```r
head(DT,2)
```

```
   A B   C   D
1: A 1 101 -50
2: A 2 102 -49
```

```r
DT[,B := B/2]
head(DT,2)
```

```
   A   B   C   D
1: A 0.5 101 -50
2: A 1.0 102 -49
```








