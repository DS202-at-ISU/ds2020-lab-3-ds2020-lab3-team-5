
<!-- README.md is generated from README.Rmd. Please edit the README.Rmd file -->

# Lab report \#3 - instructions

Follow the instructions posted at
<https://ds202-at-isu.github.io/labs.html> for the lab assignment. The
work is meant to be finished during the lab time, but you have time
until Monday evening to polish things.

Include your answers in this document (Rmd file). Make sure that it
knits properly (into the md file). Upload both the Rmd and the md file
to your repository.

All submissions to the github repo will be automatically uploaded for
grading once the due date is passed. Submit a link to your repository on
Canvas (only one submission per team) to signal to the instructors that
you are done with your submission.

# Lab 3: Avenger’s Peril

## As a team

Extract from the data below two data sets in long form `deaths` and
`returns`

``` r
av <- read.csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/avengers/avengers.csv", stringsAsFactors = FALSE)
head(av)
```

    ##                                                       URL
    ## 1           http://marvel.wikia.com/Henry_Pym_(Earth-616)
    ## 2      http://marvel.wikia.com/Janet_van_Dyne_(Earth-616)
    ## 3       http://marvel.wikia.com/Anthony_Stark_(Earth-616)
    ## 4 http://marvel.wikia.com/Robert_Bruce_Banner_(Earth-616)
    ## 5        http://marvel.wikia.com/Thor_Odinson_(Earth-616)
    ## 6       http://marvel.wikia.com/Richard_Jones_(Earth-616)
    ##                    Name.Alias Appearances Current. Gender Probationary.Introl
    ## 1   Henry Jonathan "Hank" Pym        1269      YES   MALE                    
    ## 2              Janet van Dyne        1165      YES FEMALE                    
    ## 3 Anthony Edward "Tony" Stark        3068      YES   MALE                    
    ## 4         Robert Bruce Banner        2089      YES   MALE                    
    ## 5                Thor Odinson        2402      YES   MALE                    
    ## 6      Richard Milhouse Jones         612      YES   MALE                    
    ##   Full.Reserve.Avengers.Intro Year Years.since.joining Honorary Death1 Return1
    ## 1                      Sep-63 1963                  52     Full    YES      NO
    ## 2                      Sep-63 1963                  52     Full    YES     YES
    ## 3                      Sep-63 1963                  52     Full    YES     YES
    ## 4                      Sep-63 1963                  52     Full    YES     YES
    ## 5                      Sep-63 1963                  52     Full    YES     YES
    ## 6                      Sep-63 1963                  52 Honorary     NO        
    ##   Death2 Return2 Death3 Return3 Death4 Return4 Death5 Return5
    ## 1                                                            
    ## 2                                                            
    ## 3                                                            
    ## 4                                                            
    ## 5    YES      NO                                             
    ## 6                                                            
    ##                                                                                                                                                                              Notes
    ## 1                                                                                                                Merged with Ultron in Rage of Ultron Vol. 1. A funeral was held. 
    ## 2                                                                                                  Dies in Secret Invasion V1:I8. Actually was sent tto Microverse later recovered
    ## 3 Death: "Later while under the influence of Immortus Stark committed a number of horrible acts and was killed.'  This set up young Tony. Franklin Richards later brought him back
    ## 4                                                                               Dies in Ghosts of the Future arc. However "he had actually used a hidden Pantheon base to survive"
    ## 5                                                      Dies in Fear Itself brought back because that's kind of the whole point. Second death in Time Runs Out has not yet returned
    ## 6                                                                                                                                                                             <NA>

Get the data into a format where the five columns for Death\[1-5\] are
replaced by two columns: Time, and Death. Time should be a number
between 1 and 5 (look into the function `parse_number`); Death is a
categorical variables with values “yes”, “no” and ““. Call the resulting
data set `deaths`.

Similarly, deal with the returns of characters.

Based on these datasets calculate the average number of deaths an
Avenger suffers.

``` r
# Create the deaths dataset in long format
deaths <- av %>%
  select(URL, Name.Alias, Death1, Death2, Death3, Death4, Death5) %>%
  pivot_longer(
    cols = starts_with("Death"),
    names_to = "Time",
    values_to = "Death"
  ) %>%
  mutate(
    Time = readr::parse_number(Time)
  )

# Display the first few rows of the deaths dataset
head(deaths)
```

    ## # A tibble: 6 × 4
    ##   URL                                                Name.Alias       Time Death
    ##   <chr>                                              <chr>           <dbl> <chr>
    ## 1 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonatha…     1 "YES"
    ## 2 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonatha…     2 ""   
    ## 3 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonatha…     3 ""   
    ## 4 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonatha…     4 ""   
    ## 5 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonatha…     5 ""   
    ## 6 http://marvel.wikia.com/Janet_van_Dyne_(Earth-616) "Janet van Dyn…     1 "YES"

## Restructuring Returns Data to Long Format

Similarly, let’s reshape the Return columns into a long format dataset:

``` r
# Create the returns dataset in long format
returns <- av %>%
  select(URL, Name.Alias, Return1, Return2, Return3, Return4, Return5) %>%
  pivot_longer(
    cols = starts_with("Return"),
    names_to = "Time",
    values_to = "Return"
  ) %>%
  mutate(
    Time = readr::parse_number(Time)
  )

# Display the first few rows of the returns dataset
head(returns)
```

    ## # A tibble: 6 × 4
    ##   URL                                                Name.Alias      Time Return
    ##   <chr>                                              <chr>          <dbl> <chr> 
    ## 1 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonath…     1 "NO"  
    ## 2 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonath…     2 ""    
    ## 3 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonath…     3 ""    
    ## 4 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonath…     4 ""    
    ## 5 http://marvel.wikia.com/Henry_Pym_(Earth-616)      "Henry Jonath…     5 ""    
    ## 6 http://marvel.wikia.com/Janet_van_Dyne_(Earth-616) "Janet van Dy…     1 "YES"

## Calculating Average Number of Deaths per Avenger

Now, let’s calculate the average number of deaths an Avenger suffers,
correctly accounting for those with zero deaths:

``` r
# Count deaths for each character
death_counts <- deaths %>%
  filter(Death == "YES") %>%
  group_by(URL, Name.Alias) %>%
  summarise(death_count = n(), .groups = "drop")

# Get the full list of all Avengers
all_avengers <- av %>%
  select(URL, Name.Alias) %>%
  distinct()

# Join the death counts with all avengers to include those with zero deaths
complete_death_counts <- all_avengers %>%
  left_join(death_counts, by = c("URL", "Name.Alias")) %>%
  mutate(death_count = replace_na(death_count, 0))

# Calculate the average number of deaths per Avenger
avg_deaths <- mean(complete_death_counts$death_count)

# Print the result
cat("The average number of deaths per Avenger is:", round(avg_deaths, 2))
```

    ## The average number of deaths per Avenger is: 0.51

## Individually

For each team member, copy this part of the report.

Each team member picks one of the statements in the FiveThirtyEight
[analysis](https://fivethirtyeight.com/features/avengers-death-comics-age-of-ultron/)
and fact checks it based on the data. Use dplyr functionality whenever
possible.

### Harsh’s Work:

### FiveThirtyEight Statement

“Out of 173 listed Avengers, my analysis found that 69 had died at least
one time after they joined the team.5 That’s about 40 percent of all
people who have ever signed on to the team. Let’s put it this way: If
you fall from four or five stories up, there’s a 50 percent chance you
die. Getting a membership card in the Avengers is roughly like jumping
off a four-story building”

### Include the code

``` r
# Count the number of Avengers with each number of deaths
death_distribution <- complete_death_counts %>%
  count(death_count) %>%
  mutate(percentage = n / sum(n) * 100)

# Display the distribution
death_distribution
```

    ##   death_count   n percentage
    ## 1           0 104 60.1156069
    ## 2           1  53 30.6358382
    ## 3           2  14  8.0924855
    ## 4           3   1  0.5780347
    ## 5           5   1  0.5780347

### Include your answer

Include at least one sentence discussing the result of your
fact-checking endeavor.

60% of the Avengers haven’t died once, directly oposing what they have
said that 69% have died at least once.

Upload your changes to the repository. Discuss and refine answers as a
team.
