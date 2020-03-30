---
title: "Assignment 3"
author: "Mohammad Shehab"
data: "30-March-2020"
output: html_document
---

### Introduction

This assignment is developed using ``R`` language and the RStudio IDE. 
This project uses `tidyverse`

### Dependency:

  - [xquartz](https://formulae.brew.sh/cask/xquartz) viewer
  - [tidyverse](https://www.tidyverse.org/) library
  - [brew](https://brew.sh/) Package manager for Mac

### Getting Started

```{r message=FALSE,cache=FALSE}
library("tidyverse")
```

Assignment Description:  This Assignment involves the following questions from Chapter 7: Tibbles with tibble and chapter 9: Tidy Data with tidyr:


Before we start answering the questions we load some data for testing

```{r}
library(datasets)
  data(cars)
    summary(cars)
```


### Part I
 
<b>Q</b> : 1 - What does `tibble::enframe()` do? When might you use it?
 
<b>A</b> : Works like a dictionary the  `tibble::enframe()` function converts named vectors to a data frame with keys and values for example:

```{r}
enframe(c(a = 1, b = 2, c = 3))
```

<b>Q</b> : 2 - What option controls how many additional column names are printed at the footer of a tibble

<b>A</b>: The help page for the `print()` method of tibble objects is discussed in `?print.tbl`.
The `n_extra` argument determines the number of extra columns to print information for.

```{r}
?(print.tbl)
```

---

### Part II

##### Working with Tidy Data with tidyr

<b>Q</b> 3 - Compute the `rate` for `table2`, and `table4a` + `table4b`. 
You will need to perform four operations:

1.  Extract the number of TB cases per country per year.
1.  Extract the matching population per country per year.
1.  Divide cases by population, and multiply by 10000.
1.  Store back in the appropriate place.

Which representation is easiest to work with? 
Which is hardest? 
Why?

<b>A</b>

To calculate cases per person, we need to divide cases by population for each country and year.
This is easiest if the cases and population variables are two columns in a data frame in which rows represent (country, year) combinations.


```{r}
## create separate tables for cases 
## and population and ensure that they are sorted in the same order

t2_cases <- filter(table2, type == "cases") %>%
  rename(cases = count) %>%
    arrange(country, year)

# Display The Cases
t2_cases;

t2_population <- filter(table2, type == "population") %>%
  rename(population = count) %>%
    arrange(country, year)

# Display t2 populations    
t2_population;
```
```{r}
## Create the population and cases columns, and calculate the cases per capita in a new column.[^ex-12.2.2]

t2_cases_per_cap <- tibble(year = t2_cases$year,
  country = t2_cases$country,
  cases = t2_cases$cases,
  population = t2_population$population) %>%
    mutate(cases_per_cap = (cases / population) * 10000) %>%
      select(country, year, cases_per_cap)

## Display type 2 cases per capita
t2_cases_per_cap
```

```{r}

## To store this new variable in the appropriate location, we will add new rows to `table2`.

t2_cases_per_cap <- t2_cases_per_cap %>%
  mutate(type = "cases_per_cap") %>%
  rename(count = cases_per_cap)
```

```{r}
bind_rows(table2, t2_cases_per_cap) %>%
  arrange(country, year, type, count)
```

Note that after adding the `cases_per_cap` rows, the type of `count` is coerced to `numeric` (double) because `cases_per_cap` is not an integer.

```{r}
## For `table4a` and `table4b`, create a new table for cases per capita which ## we'll name `table4c`, with country rows and year columns.

table4c <-
  tibble(
    country = table4a$country,
    `1999` = table4a[["1999"]] / table4b[["1999"]] * 10000,
    `2000` = table4a[["2000"]] / table4b[["2000"]] * 10000
  )
table4c
```

<b>Analysis</b>

Neither table is particularly easy to work with.
Since `table2` has separate rows for cases and population we needed to generate a table with columns for cases and population where we could
calculate cases per capita.
`table4a` and `table4b` split the cases and population variables into different tables which
made it easy to divide cases by population.
However, we had to repeat this calculation for each row.

The ideal format of a data frame to answer this question is one with columns `country`, `year`, `cases`, and `population`.
Then problem could be answered with a single `mutate()` call.

---

## Part III
<b>Q</b> 4 - Recreate the plot showing change in cases over time using `table2 instead of `table1`. What do you need to do first ?

```{r}
table2 %>%
  filter(type == "cases") %>%
    ggplot(aes(year, count)) +
      geom_line(aes(group = country), colour = "grey50") +
        geom_point(aes(colour = country)) +
          scale_x_continuous(breaks = unique(table2$year)) +
  ylab("cases")
```

--- 
## Part IV

<b>Q</b> 5 - Tidy the simple tibble below. Do you need to make it wider or longer?  What are the variables?

```{r}
preg <- tribble(
  ~pregnant, ~male, ~female,
  "yes", NA, 10,
  "no", 20, 12
)
```

To tidy the `preg` tibble, we need to use `gather()`. 
The variables in this data are:

*   `sex` ("female", "male")
*   `pregnant` ("yes", "no")
*   `count`, which is a non-negative integer representing the number of observations.

The observations in this data are unique combinations of sex and pregnancy status.

```{r}
preg_tidy <- preg %>%
  gather(male, female, key = "sex", value = "count")
preg_tidy
```

We can simplify the tidied data frame by removing the (male, pregnant) row with the missing value of `NA`.

```{r}
preg_tidy2 <- preg %>%
  gather(male, female, key = "sex", value = "count", na.rm = TRUE)
preg_tidy2
```

Although we have already done enough to make the data tidy, there's some other changes that can be made to clean this data.
If a variable takes two values, like `pregnant` and `sex`, it is often preferable to store them as logical vectors.

```{r}
preg_tidy3 <- preg_tidy2 %>%
  mutate(
    female = sex == "female",
    pregnant = pregnant == "yes"
  ) %>%
  select(female, pregnant, count)
preg_tidy3
```

In the previous data frame, we named the logical variable representing the sex `female`, not `sex`.

This makes the meaning of the variable self-documenting.
If the variable were named `sex` with values `TRUE` and `FALSE`, without reading the documentation, we wouldn't know whether `TRUE` means male or female.

Finally compare the `filter()` calls to select non-pregnant females from `preg_tidy2` and `preg_tidy`.

```{r}
filter(preg_tidy2, sex == "female", pregnant == "no")
filter(preg_tidy3, female, !pregnant)
```

