README
================

Authors: Reigo
Hendrikson

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://choosealicense.com/licenses/mit/)

### Use `purrr` to feed four cats

In this example we will show you how to go from a ‘for loop’ to purrr.
Use this as a cheatsheet when you want to replace your for loops.

Imagine having 4 cats. Four real cats who need food, care and love to
live a happy life. They are starting to meow, so it’s time to feed them.
Our real life algorithm would be:

  - give food to cat 1
  - give food to cat 2
  - …

But life is not that simple. Some of your cats are good cats and some
are bad. You want to spoil the good cats and give normal food to the bad
cats (Clearly the authors do not have cats in real life because Cats
will complain no matter what).

## Elements of the four cats problem

Let’s work out this ‘problem’ in R. In real life we would have to have
some place for the cats and their bowls. In R the place for cats and
their bowls can be a data.frame (or tibble).

Let’s start with the cats. The cats have a name and a temperament: they
are good or bad. Let’s call that goodness\[1\].

``` r
library(tibble)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(purrr)
cats <- 
    tribble(
    ~Name, ~Good,
    "Suzie", TRUE,
    "Margaret", FALSE,
    "Donald", FALSE,
    "Tibbles", TRUE
    )
cats$Name <- paste(emo::ji("cat"), cats$Name)
cats$Good <- ifelse(cats$Good, emo::ji("heart"), emo::ji("lightning"))
cats
```

    ## # A tibble: 4 x 2
    ##   Name        Good 
    ##   <chr>       <chr>
    ## 1 🐱 Suzie    ❤️   
    ## 2 🐱 Margaret 🌩   
    ## 3 🐱 Donald   🌩   
    ## 4 🐱 Tibbles  ❤️

Now we have to feed the cats. Only good cats get premium quality
super-awesome-food. Bad cats get normal food.

If you have used dplyr or ifelse statements before, this example is
super silly and inefficient. But for people coming from different
programming languages and people who started out with for loops, this is
the approach you might use.

To feed the cat, we have to determine if the cat is Good or not. We
don’t want to type to much, we are lazy data scientist after all. So
we create a function to do our work.

``` r
good_kitty <- function(x) {
    if(x == emo::ji("heart")) {
        paste0(emo::ji("cake")," Premium food")
    } else {
        paste0(emo::ji("bread"), " Normal food")
    }
}
good_kitty(cats$Good[1])
```

    ## [1] "\U0001f370 Premium food"

We also want to cuddle our cats, unfortunately some of the cats don’t
want to be pet. Margareth and Donald don’t want to be cuddled. So let’s
create a function that takes care of that.

``` r
pet_cat <- 
    function(name){
    if(name %in% c("Margaret", "Tibbles")){
        "pet"
    }else{
        "don't pet, stay away!"
    }
}
pet_cat("Donald")
```

    ## [1] "don't pet, stay away!"

``` r
pet_cat("Tibbles")
```

    ## [1] "pet"

## Assigning food and love to kitties

There’s two things we want to do. Give food and give cuddles.

For every row we want to do two things:

  - give food based on Goodness
  - pet based on name

This is something we would want to use a ‘for loop’ for. If you have
programmed with C(++) or python, this is a natural approach. And it is
not wrong to do this in R too.

``` r
# First we create place for the Food to go into
# and for the petting to take place. 
cats$Food <- NA
cats$Petting <- NA
for (cat in 1:nrow(cats)){
    cats$Food[cat] <- good_kitty(cats$Good[cat])
    cats$Petting[cat] <- pet_cat(cats$Name[cat])
}
cats
```

    ## # A tibble: 4 x 4
    ##   Name        Good  Food            Petting              
    ##   <chr>       <chr> <chr>           <chr>                
    ## 1 🐱 Suzie    ❤️    🍰 Premium food don't pet, stay away!
    ## 2 🐱 Margaret 🌩    🍞 Normal food  don't pet, stay away!
    ## 3 🐱 Donald   🌩    🍞 Normal food  don't pet, stay away!
    ## 4 🐱 Tibbles  ❤️    🍰 Premium food don't pet, stay away!

A ‘for loop’ is you telling your computer: ‘I will only tell you this
once, but you must do it to everything in the collection’. To make that
work the computer needs to know what the iterator part is: `(something
in collection)` in this case `(cat in 1:nrow(cats))`, what needs to
happen inside the loop: `{actions}`, and the use of a variable that
takes a different value every iteration `something` or in this case
`cat`. People often use `i` for this value `for (i in list)` but as you
can see, it really doesn’t matter.

Let’s get into a bit more details

### index

inside the loop we see `cats$Food[cat]`, we’re taking the `Food` column
in the `cats` data.frame. And then we index `[]` it. `cats$Food[1]`
would take the first item in this collection, `cats$Food[2]` the second,
etc.

Were does the index come from?

For loops are dumb. They don’t know and don’t care about the collection
they are moving through. The only thing they know is that they move
through every element of the collection until the end. In our case the
collection is `1:nrow(cats)`, a sequence from 1 to the number of rows in
the data.frame.

When your ‘for loop’ doesn’t behave as you expect the best way to debug
it is to print the index at the start of the loop.

## Functional approach

For loops are very often used and very handy. But they do require a bit
of mental overhead, you need to think about the collection you are
iterating over and use the correct indices to put stuff in. It’s easy to
get lost.

With the `purrr` package we approach the problem from another direction:
we think about small steps. We want to execute the pet and feeding
functions on every row, we do not care about the specifics of the row
number.

In the purrr style of thinking (functional programming) we want to ‘map’
the function to the data. We do care about the outcome of every
execution, we want that to be character.

Let’s use `map_chr`

``` r
# reset the Food and Petting
cats$Food <- NA
cats$Petting <- NA
# execute on every element of cats$Good the good kitty function
cats$Food <- map_chr(cats$Good, good_kitty)
# execute on every element of cats$Name the pet_cat function
cats$Petting <- map_chr(cats$Name, pet_cat)
cats
```

    ## # A tibble: 4 x 4
    ##   Name        Good  Food            Petting              
    ##   <chr>       <chr> <chr>           <chr>                
    ## 1 🐱 Suzie    ❤️    🍰 Premium food don't pet, stay away!
    ## 2 🐱 Margaret 🌩    🍞 Normal food  don't pet, stay away!
    ## 3 🐱 Donald   🌩    🍞 Normal food  don't pet, stay away!
    ## 4 🐱 Tibbles  ❤️    🍰 Premium food don't pet, stay away!

The results are the same.

## Resources

Just when we were finishing up this post there was an excellent [tweet
thread](https://mobile.twitter.com/apreshill/status/1036986558887342080)
on purrr resources\!

Complete list at moment of writing

  - <https://mobile.twitter.com/WeAreRLadies/status/1034817323922804737>
  - <https://github.com/jenniferthompson/RLadiesIntroToPurrr>
  - <https://jennybc.github.io/purrr-tutorial/>
  - <https://github.com/cwickham/purrr-tutorial>
  - <https://github.com/jennybc/row-oriented-workflows#readme>
  - <https://amber.rbind.io/blog/2018/03/26/purrr_iterations/>

<!-- end list -->

1.  Incidently this is a great way to label things. Don’t create a
    gender column with male or female (or other), but make the label the
    description: female TRUE/FALSE
