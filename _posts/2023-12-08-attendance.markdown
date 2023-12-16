---
layout: posts
title:  "SP and Attendance"
date:   2023-12-08
categories: work
tags: r
author_profile: true
author: Keegan Johnson
---
<br>
```
{r Packages}

library('plotly')
library('odbc')
library('dplyr')
library('tidyverse')
library('lubridate')
library('DataEditR')
library('baseballr')
library('data.table')
library('arrow')

```


```
{r SQL Connect}

db <- DBI::dbConnect(odbc::odbc(), "SQL")

# Connect without a DSN
db <- DBI::dbConnect(odbc::odbc(),
                     Driver = 'ODBC Driver 17 for SQL Server',
                     Server = 'LAPTOP',
                     Database = "saber",
                     trusted_connection = 'yes',
                     Port = 1433
                     )


df <- DBI::dbGetQuery(db,"SELECT * 
                                    FROM sp_attendance")

head(df)


```

```
{r Data Wrangle}
# Select fields of interest:

df.1 <- df |>
    select(game_date
           , attendance
           , matchup_pitcher_fullName
           , attendance
           , fielding_team
           , venue_name
           , temperature
           , phase_of_day
           )

# df.fv <- df.1 |> select(game_date, attendance) |>
#         filter(matchup_pitcher_fullName=='Framber Valdez' & venue_name=='Minute Maid Park')
# 
# # Visualizations:  
# 
# p_fv <- ggplot(df.fv, aes(x=game_date, y=attendance)) +
#         geom_step(color='#EB6E1F') + 
#         labs(title = "Framber Valdez Attendance @MMP",
#              x='Starts',
#              y='Attendance') + 
#         theme_minimal() 
# 
# ggplotly(p_fv)

# Create 'ace' dummy variable:

# Create list of aces for 2023

aces_23 <- c('Patrick Corbin',
          'Kyle Muller',
          'Kyle Gibson',
          'Eduardo Rodriguez',
          'Chris Sale',
          'German Marquez',
          'Zack Greinke',
          'Pablo Lopez',
          'Hunter Greene',
          'Mitch Keller',
          'Miles Mikolas',
          'Marcus Stroman',
          'Shane Bieber',
          'Logan Webb',
          'Yu Darvish',
          'Luis Castillo',
          'Shane McClanahan',
          'Framber Valdez',
          'Alek Manoah',
          'Jacob deGrom',
          'Dylan Cease',
          'Julio Urias',
          'Max Fried',
          'Zac Gallen',
          'Aaron Nola',
          'Gerrit Cole',
          'Max Scherzer',
          'Shohei Ohtani',
          'Sandy Alcantara',
          'Corbin Burnes'
          )

df.1 <- df.1 |> 
    mutate(ace_23 = ifelse(matchup_pitcher_fullName %in% aces_23, 1, 0))

# Create list of aces for 2022

aces_22 <- c('Jose Quintana',
          'Patrick Corbin',
          'Madison Bumgarner',
          'Justin Verlander',
          'Zack Greinke',
          'Jon Gray',
          'Eduardo Rodriguez',
          'Sean Manaea',
          'Kyle Hendricks',
          'Shane McClanahan',
          'German Marquez',
          'Aaron Nola',
          'Yu Darvish',
          'Sonny Gray',
          'Tyler Mahle',
          'John Means',
          'Adam Wainwright',
          'Jose Berrios',
          'Shohei Ohtani',
          'Nathan Eovaldi',
          'Logan Webb',
          'Sandy Alcantara',
          'Max Fried',
          'Lance Lynn',
          'Shane Beiber',
          'Walker Buehler',
          'Gerrit Cole',
          'Robbie Ray',
          'Corbin Burnes',
          'Jacob deGrom'
          )

df.1 <- df.1 |> 
    mutate(ace_22 = ifelse(matchup_pitcher_fullName %in% aces_22, 1, 0))

# Create list of aces for 2021
aces_21 <- c('Tyler Anderson',
             'Matthew Boyd',
             'Kyle Gibson',
             'John Means',
             'Marco Gonzales',
             'Madison Bumgarner',
             'Nathan Eovaldi',
             'Tyler Glasnow',
             'Brad Keller',
             'Chris Bassitt',
             'Kevin Gausman',
             'Dylan Bundy',
             'Max Fried',
             'Sandy Alcantara',
             'Jack Flaherty',
             'German Marquez',
             'Zack Greinke',
             'Kyle Hendricks',
             'Brandon Woodruff',
             'Kenta Maeda',
             'Max Scherzer',
             'Luis Castillo',
             'Clayton Kershaw',
             'Yu Darvish',
             'Aaron Nola',
             'Lucas Giolito',
             'Hyun Jin Ryu',
             'Shane Bieber',
             'Gerrit Cole',
             'Jacob deGrom'
          )

df.1 <- df.1 |> 
    mutate(ace_21 = ifelse(matchup_pitcher_fullName %in% aces_21, 1, 0))

# Create columns to indicate if pitcher was an ace in previous 3 seasons:
df.1 <- df.1 |>
    rowwise() |> 
    mutate(ace_total = sum(ace_23,ace_22,ace_21))

df.1 <- df.1 |>
    rowwise() |> 
    mutate(ace_bin = ifelse(ace_total >= 1, 1, 0))

# Convert variables to factors:
df.1$phase_of_day <- as_factor(df.1$phase_of_day)
df.1$ace_bin <- as_factor(df.1$ace_bin)

```
```
{r EDA and Modeling}

tapply(df.1$attendance, df.1$ace_bin, summary)

barplot(table(df.1$ace_bin), main = "Aces")

boxplot(df.1$attendance, by=df.1$ace_bin)




# Model to run is just linear regression: 
model <- lm(attendance ~ ace_bin, data=df.1) 
summary(model)

# Scatter plot with regression line
plot(df.1$ace_bin, df.1$attendance, 
     main = "Attendance with Aces", 
     xlab = "Ace", ylab = "Attendance")
abline(model, col = "orange")

# Mean Squared Error (MSE)
mse <- mean(model$residuals^2)

# Root Mean Squared Error (RMSE)
rmse <- sqrt(mse)

# Display metrics
cat("Mean Squared Error:", mse, "\n")
cat("Root Mean Squared Error:", rmse, "\n")


```
