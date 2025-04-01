
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

### Aiden’s Work:

### FiveThirtyEight Statement

“I counted 89 total deaths — some unlucky Avengers are basically Meat
Loaf with an E-ZPass — and on 57 occasions the individual made a
comeback. Maybe they didn’t actually die and were secretly in the
Microverse, or they stayed on Franklin Richards‘s or the Scarlet Witch‘s
good side in life, or they were dragged back into Avenging by the Chaos
King or the Grim Reaper, or perhaps a colleague made a deal with time
travelers. Who knows!”

“But you can only tempt death so many times. There’s a 2-in-3 chance
that a member of the Avengers returned from their first stint in the
afterlife, but only a 50 percent chance they recovered from a second or
third death.8”

### Include the code

``` r
#Create a list of all the characters with at least one death
first_three_death_list <- av %>%
  select(URL, Name.Alias, Death1, Return1, Death2, Return2, Death3, Return3) %>%
  filter(Death1 == "YES")

#Filter out all the people that don't return after their first death
no_return_first_death <- first_three_death_list %>%
  filter(Return1 == "NO") %>%
  nrow()

#Calculate how many of the characters return after their first death
return_first_death_percent <- (1 - no_return_first_death / nrow(first_three_death_list)) * 100

#display the data
return_first_death_percent
```

    ## [1] 66.66667

``` r
#Characters who died twice
has_second_death <- first_three_death_list %>%
  filter(Death2 == "YES")

#Characters who died 3 times
has_third_death <- first_three_death_list %>%
  filter(Death3 == "YES")

#Number of characters who returned from their second death
second_return <- has_second_death %>%
  filter(Return2 == "YES") %>%
  nrow()

#Number of characters who returned from their third death
third_return <- has_third_death %>%
  filter(Return3 == "YES") %>%
  nrow()

#Number of characters who returned from a 2nd and/or 3rd death divided by the number of characters who had a 2nd or 3rd death
return_later_percent <- (second_return + third_return) / ((nrow(has_second_death) + nrow(has_third_death)))

#display the data
return_later_percent
```

    ## [1] 0.5

### Include your answer

Include at least one sentence discussing the result of your
fact-checking endeavor.

The first statement that there is a 2 in 3 chance that characters return
after their 3rd death checked out, my data also got 66.66% of characters
returning. The second statement that there is only a 50% chance of
recovery from a 2nd or 3rd death was also accurate in my findings, with
exactly 50% of 2nd or 3rd deaths being returned from. Overall in this
section they were accurate based on my findings.
