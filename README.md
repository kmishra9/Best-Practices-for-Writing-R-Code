# Best Practices & Style Guide for Writing R Code

##  Why Best Practices Are Important

Writing good code that adheres to many of the best practices listed here makes your code more digestible to anyone else who examines your code at any point. That could include collaborators, study reviewers, or even yourself, 6 months into the future. Writing best-practice code also makes the process of finding bugs, fixing them, comparing replicated code, and making large breaking changes much easier.

## Project Structure

Many data science projects, especially those with multiple contributors, would benefit from following a standardized workflow that is consistent across _all_ projects. One way to do this is by using a directory structure that looks like this:
```
0-run-project.sh
0-config.R
1 - Data-Management/
    0-prep-data.sh
    1-prep-cdph-fluseas.R
    2a-prep-absentee.R
    2b-prep-absentee-weighted.R
    3a-prep-absentee-adj.R
    3b-prep-absentee-adj-weighted.R
2 - Analysis/
    0-run-analysis.sh
    1 - Absentee-Mean/
        1-absentee-mean-primary.R
        2-absentee-mean-negative-control.R
        3-absentee-mean-CDC.R
        4-absentee-mean-peakwk.R
        5-absentee-mean-cdph2.R
        6-absentee-mean-cdph3.R
    2 - Absentee-Positivity-Check/
    3 - Absentee-P1/
    4 - Absentee-P2/
3 - Figures/
    0-run-figures.sh
    ...
4 - Tables/
    0-run-tables.sh
    ...
5 - Results/
    1 - Absentee-Mean/
        1-absentee-mean-primary.RDS
        2-absentee-mean-negative-control.RDS
        3-absentee-mean-CDC.RDS
        4-absentee-mean-peakwk.RDS
        5-absentee-mean-cdph2.RDS
        6-absentee-mean-cdph3.RDS
    ...
.gitignore
.Rproj
```
For brevity, not every directory is "expanded", but we can glean some important takeaways from what we _do_ see.

1. **Order Files and Directories** - This makes the jumble of alphabetized filenames much more coherent and places similar code and files next to one another. This also helps us understand how data flows from start to finish and allows us to easily map a script to its output (i.e. `2 - Analysis/1 - Absentee-Mean/1-absentee-mean-primary.R` => `5 - Results/1 - Absentee-Mean/1-absentee-mean-primary.RDS`). If you take nothing else away from this guide, this is the single most helpful suggestion to make your workflow more coherent.
    - **Note**: Directories have capitalized letters and spaces but individual files do not.
2. **Use`.gitignore` and `.Rproj` files** - There is a standardized `.gitignore` for `R` which you [can download](https://github.com/github/gitignore/blob/master/R.gitignore) and add to your project. This ensures you're not committing log files or things that would otherwise best be left ignored to GitHub.
    - An "R Project" can be created within RStudio by going to `File >> New Project`. Depending on where you are with your research, choose the most appropriate option. This will save preferences, working directories, and even the results of running code/data (though I'd recommend starting from scratch each time you open your project, in general). Then, ensure that whenever you are working on that specific research project, you open your created project to enable the full utility of `.Rproj` files.
3. **Bash scripts** - these are useful components of a reproducible workflow. At many of the directory levels (i.e. in `3 - Analysis`), there is a bash script that runs each of the analysis scripts. This is exceptionally useful when data "upstream" changes -- you simply run the bash script. For big data workflows, the concept of "backgrounding" a Bash script allows you to start a "job" (i.e. run the script) and leave it overnight to run. At the top level, a bash script (`0-run-project.sh`) that simply calls the directory-level bash scripts (i.e. `0-prep-data.sh`,  `0-run-analysis.sh`, `0-run-figures.sh`, etc.) is a powerful tool to rerun every script in your project. See the included example bash scripts for more details.
    - **Running Bash Scripts in Background**: Running a long bash script is not trivial. Normally you would run a bash script by opening a terminal and typing something like `./run-project.sh`. But what if you leave your computer, log out of your server, or close the terminal? Normally, the bash script will exit and fail to complete. To run it in background, type `./run-project.sh &; disown`. You can see the job running (and CPU utilization) with the command `top` and check your memory with `free -h`.
4. **Use a Config File** - This is the single most important file for your project. It will be responsible for a variety of common tasks, declare global variables, load functions, declare paths, and more. _Every other file in the project_ will begin with `source("0-config")`, and its role is to reduce redundancy and create an abstraction layer that allows you to make changes in one place (`0-config.R`) rather than 5 different files. To this end, paths which will be reference in multiple scripts (i.e. a `merged_data_path`) can be declared in `0-config.R` and simply referred to by its variable name in scripts. If you ever want to change things, rename them, or even switch from a downsample to the full data, all you would then to need to do is modify the path in one place and the change will automatically update throughout your project. See the example config file for more details.

## Comments

1. **File Headers** - Every file in a project should have a header that allows it to be interpreted on its own. It should include the name of the project and a short description for what this file (among the many in your project) does specifically. You may optionally wish to include the inputs and outputs of the script as well, though the next section makes this significantly less necessary.
  ```
  ################################################################################
  # Master's Thesis - Spatial Epidemiology of Absenteeism
  # Shoo the Flu Evaluation
  # 2011-2018 Spatial Epidemiology Analysis
  # Input management file for creating aggregated, publishable datasets of statistical inputs
  ################################################################################
  ```
2. **File Structure** - Just as your data "flows" through your project, data should flow naturally through a script. Very generally, you want to 1) source your config => 2) load all your data => 3) do all your analysis/computation => save your data. Each of these sections should be "chunked together" using comments. See [this file](https://github.com/kmishra9/Flu-Absenteeism/blob/master/Master's%20Thesis%20-%20Spatial%20Epidemiology%20of%20Influenza/2a%20-%20Statistical-Inputs.R) for a good example of how to cleanly organize a file in a way that follows this "flow" and functionally separate pieces of code that are doing different things.
    - **Note**: If your computer isn't able to handle this workflow due to RAM or requirements, modifying the ordering of your code to accomodate it won't be ultimately helpful and your code will be fragile, not to mention less readable and messy. You need to look into high-performance computing (HPC) resources in this case.
3. **Single-Line Comments** - Commenting your code is an important part of reproducibility and helps document your code for the future. When things change or break, you'll be thankful for comments. There's no need to comment excessively or unnecessarily, but a comment describing what a large or complex chunk of code does is always helpful. See [this file](https://github.com/kmishra9/Flu-Absenteeism/blob/master/Master's%20Thesis%20-%20Spatial%20Epidemiology%20of%20Influenza/1b%20-%20Map-Management.R) for an example of how to comment your code and notice that comments are always in the form of:

  ```# This is a comment -- first letter is capitalized and spaced away from the pound sign```

4. **Multi-Line Comments** - Occasionally, multi-line comments are necessary. Don't add line breaks manually to a single-line comment for the purpose of making it "fit" on the screen. Instead, in RStudio => Global Options => Code => “Soft-wrap R source files” to have lines wrap around. Format your multi-line comments like the file header from above.
5. **Function Documentation** - Functions _need_ documentation. For any reproducible workflows, they are essential, because R is dynamically typed. This means, you can pass a `string` into an argument that is meant to be a `data.table`, or a `list` into an argument meant for a `tibble`. It is the responsibility of a function's author to document what each argument is meant to do and its basic type. This is an example for documenting a function (inspired by [JavaDocs](https://www.oracle.com/technetwork/java/javase/documentation/index-137868.html#format)):
  ```
  calculate_KSSS = function(centroids, statistical_input, time_column = "schoolyr", location_column = "school_dist", value_column = "absences_ill", population_column = "student_days", k_nearest_neighbors = 5, nsim = 9999, heat_map = TRUE, heat_map_title = NULL, heat_map_caption = NULL) {
    # @Description: Calculates the population-based Kulldorff Spatial Scan Statistic
    # @Arg: centroids: an SPDF from which the centroids of each school catchment area can be drawn, along with a uniquely identifying location_column (such as an ID or School-District combo)
    # @Arg: statistical_input: a tibble of at least 4 columns containing a location, time, value, and population size, ordered by (location_column, time_column)
    # @Arg: time_column: an integer or string column on which values can be clustered temporally
    # @Arg: location_column: an integer or string column on which values can be clustered spatially (must be a unique key)
    # @Arg: value_column: an integer or string column containing the count data for the given space-time
    # @Arg: population_column: an integer column containing the size of the population for the given space-time
    # @Arg: heat_map: a boolean variable dictating whether to plot a heat_map based on how likely a school catchment is to be part of a cluster
    # @Arg: heat_map_title: a string used as the title for a heat_map if one is drawn
    # @Arg: heat_map_caption: a string used as the caption for a heat_map if one is drawn
    # @Output: plots a heatmap local clustering and prints the total number of significant clusters if heat_map = TRUE
    # @Return: a list containing the results of running KSSS and a tibble of all clusters

    ...
    Some code here
    ...
  }
  ```
  Even if you have no idea what a `KSSS` is, you have some way of understanding what the function does, its various inputs, and how you might go about using the function to do what you want. Also notice that the function is defined in one line at the top (which will soft-wrap around) and all optional arguments (i.e. ones with pre-specified defaults) follow arguments that require user input.
    - **Note**: As someone trying to call a function, it is possible to access a function's documentation (and internal code) by `CMD-Left-Click`ing the function's name in RStudio

## Variables & Function Calls

1. **Variable Names** - Try to make your variable names both more expressive and more explicit. Being a bit more verbose is useful and easy in the age of autocompletion! For example, instead of naming a variable `vaxcov_1718`, try naming it `vaccination_coverage_2017_18`. Similarly, `flu_res` could be named `absentee_flu_residuals`, making your code more readable and explicit.
    - For more help, check out [Be Expressive: How to Give Your Variables Better Names](https://spin.atomicobject.com/2017/11/01/good-variable-names/)

2. **Snake_Case** - Base R allows `.` in variable names and functions (such as `read.csv()`), but this goes against best practices in many other coding languages. For consistencies sake, across all data science languages, `snake_case` has been adopted and modern packages and functions typically use it (i.e. `readr::read_csv()`). As a very general rule of thumb, if a package you're using doesn't use `snake_case`, there may be an updated version or more modern package that _does_, bringing with it the variety of performance improvements and bug fixes inherent in more mature and modern software.
    - **Note**: you may also see `camelCase` throughout the R code you come across. This is _okay_ but not ideal -- try to stay consistent across all your code with `snake_case`.

3. **Assignment** - _Please_ use the `=` operator instead of  `<-`. Please! Similarly, in a function call, definitely use "named arguments" and separate arguments by to make your code more readable. Here's an example of what a function call for `calculate_KSSS` (documented above) might look like without named arguments or any separation:
  ```
  input_1_KSSS_ill = calculate_KSSS(all_study_school_shapes, input_1, 5, "Local Clustering of Illness-Specific\n Absence Rates in all years during Flu Season")
  ```
  And here it is again using the best practices we've outlined:
  ```
  input_1_KSSS_ill = calculate_KSSS(
    centroids           = all_study_school_shapes,
    statistical_input   = input_1,
    k_nearest_neighbors = 5,
    heat_map_title      = "Local Clustering of Illness-Specific\n Absence Rates in all years during Flu Season"
  )
  ```
  I'll let you be the judge of which is more coherent.

## Automated Styling Tools (in RStudio)

1. **Code Autoformatting** - RStudio includes a fantastic built-in utility (keyboard shortcut: `CMD-Shift-A`) for autoformatting highlighted chunks of code to fit many of the best practices listed here. It generally makes code more readable and fixes a lot of the small things you may not feel like fixing yourself. Try it out as a "first pass" on some code of yours that _doesn't_ follow many of these best practices!

2. **Assignment Aligner** - A [cool R package](https://www.r-bloggers.com/align-assign-rstudio-addin-to-align-assignment-operators/) allows you to very powerfully format large chunks of assignment code to be much cleaner and much more readable. Follow the linked instructions and create a keyboard shortcut of your choosing (recommendation: `CMD-Shift-Z`). Here is an example of how assignment aligning can dramatically improve code readability:
  ```
  # Before
  OUSD_not_found_aliases = list(
    "Brookfield Village Elementary" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Brookfield"),
    "Carl Munck Elementary" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Munck"),
    "Community United Elementary School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Community United"),
    "East Oakland PRIDE Elementary" = str_subset(string = OUSD_school_shapes$schnam, pattern = "East Oakland Pride"),
    "EnCompass Academy" = str_subset(string = OUSD_school_shapes$schnam, pattern = "EnCompass"),
    "Global Family School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Global"),
    "International Community School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "International Community"),
    "Madison Park Lower Campus" = "Madison Park Academy TK-5",
    "Manzanita Community School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Manzanita Community"),
    "Martin Luther King Jr Elementary" = str_subset(string = OUSD_school_shapes$schnam, pattern = "King"),
    "PLACE @ Prescott" = "Preparatory Literary Academy of Cultural Excellence",
    "RISE Community School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Rise Community")
)
  ```
  ```
  # After
  OUSD_not_found_aliases = list(
    "Brookfield Village Elementary"      = str_subset(string = OUSD_school_shapes$schnam, pattern = "Brookfield"),
    "Carl Munck Elementary"              = str_subset(string = OUSD_school_shapes$schnam, pattern = "Munck"),
    "Community United Elementary School" = str_subset(string = OUSD_school_shapes$schnam, pattern = "Community United"),
    "East Oakland PRIDE Elementary"      = str_subset(string = OUSD_school_shapes$schnam, pattern = "East Oakland Pride"),
    "EnCompass Academy"                  = str_subset(string = OUSD_school_shapes$schnam, pattern = "EnCompass"),
    "Global Family School"               = str_subset(string = OUSD_school_shapes$schnam, pattern = "Global"),
    "International Community School"     = str_subset(string = OUSD_school_shapes$schnam, pattern = "International Community"),
    "Madison Park Lower Campus"          = "Madison Park Academy TK-5",
    "Manzanita Community School"         = str_subset(string = OUSD_school_shapes$schnam, pattern = "Manzanita Community"),
    "Martin Luther King Jr Elementary"   = str_subset(string = OUSD_school_shapes$schnam, pattern = "King"),
    "PLACE @ Prescott"                   = "Preparatory Literary Academy of Cultural Excellence",
    "RISE Community School"              = str_subset(string = OUSD_school_shapes$schnam, pattern = "Rise Community")
)
  ```

3. **StyleR** - Another [cool R package from the Tidyverse](https://www.tidyverse.org/articles/2017/12/styler-1.0.0/) that can be powerful and used as a first pass on entire projects that need refactoring. The most useful function of the package is the `style_dir` function, which will style all files within a given directory. See the [function's documentation](https://www.rdocumentation.org/packages/styler/versions/1.1.0/topics/style_dir) and the vignette linked above for more details.
    - **Note**: The default Tidyverse styler is subtly different from some of the things we've advocated for in this document. Most notably we differ with regards to the assignment operator (`<-` vs `=`) and number of spaces before/after "tokens" (i.e. Assignment Aligner add spaces before `=` signs to align them properly). For this reason, we'd recommend the following: `style_dir(path = ..., scope = "line_breaks", strict = FALSE, `. You can also customize StyleR [even more](http://styler.r-lib.org/articles/customizing_styler.html) if you're really hardcore.
    - **Note**: As is mentioned in the package vignette linked above, StyleR modifies things _in-place_, meaning it overwrites your existing code and replaces it with the updated, properly styled code. This makes it a good fit on projects _with version control_, but if you don't have backups or a good way to revert back to the intial code, I wouldn't recommend going this route.

## Outdated Base R Practices to Avoid

### Reading/Saving Data
1. **`.RDS` vs `.RData` Files** - One of the most common ways to load and save data in Base R is with the `load()` and `save()`  functions to serialize multiple objects into a file. The biggest problems with this practice include an inability to control the names of things getting loaded in, the inherent confusion this creates in understanding older code, and the inability to load individual elements of a saved file. For this, we recommend using the RDS format to save R objects.
    - **Note**: `saveRDS` and `loadRDS` are Base R functions to do this but we recommend using `save_rds` and `load_rds` from the `readr` package for the sake of consistency and performance improvements.
    - **Note**: when you use `load_rds` you must assign the thing being loaded to a variable (i.e. `some_descriptive_variable_name = load_rds(...)`). Then, use the variable as you would. This makes your code less fragile and it doesn't need to rely on an `.RData` file to load in a particular name for the code to run properly.
    - **Note**: if you have many related R objects you would have otherwise saved all together using the `save` function, the functional equivalent with `RDS` would be to create a (named) list containing each of these objects, and saving it.

2. **CSVs** - Once again, the `readr` package as part of the Tidvyerse is great, with a much faster `read_csv()` than Base R's `read.csv()`. For massive CSVs (> 5 GB), you'll find `data.table::fread()` to be the fastest CSV reader in any data science language out there. For writing CSVs, `readr::write_csv()` and `data.table::fwrite()` outclass Base R's `write.csv()` by a significant margin as well.

3. **Feather** - If you're using both R and Python, you may wish to check out the [Feather package](https://www.rdocumentation.org/packages/feather/versions/0.3.3) for exchanging data between the two languages [extremely quickly](https://blog.rstudio.com/2016/03/29/feather/).

### Tidyverse

Throughout this document there have been references to the Tidyverse, but this section is to explicitly show you how to transform your Base R tendencies to Tidyverse (or Data.Table, Tidyverse's performance-optimized competitor). Tidyverse is quickly becoming [the gold standard](https://rviews.rstudio.com/2017/06/08/what-is-the-tidyverse/) in R data analysis and modern data science packages and code should use Tidyverse style and packages unless there's a significant reason not to (i.e. big data pipelines that would benefit from Data.Table's performance optimizations).

The package author has published a [great textbook on R for Data Science](https://r4ds.had.co.nz/), which leans heavily on many Tidyverse packages and may be worth checking out.

The following list is not exhaustive, but is a compact overview to begin to translate Base R into something better:

Base R | Better Style, Performance, and Utility
--- | ---
_|_
`read.csv()`| `readr::read_csv()` or `data.table::fread()`
`write.csv()` | `readr::write_csv()` or `data.table::fwrite()`
`readRDS` | `readr::read_rds()`
`saveRDS()` | `readr::write_rds()`
_|_
`data.frame()` | `tibble::tibble()` or `data.table::data.table()`
`rbind()` | `dplyr::bind_rows()`
`cbind()` | `dplyr::bind_cols()`
`df$some_column` | `df %>% dplyr::pull(some_column)`
`df$some_column = ...` | `df %>% dplyr::mutate(some_column = ...)`
`df[get_rows_condition,]` | `df %>% dplyr::filter(get_rows_condition)`
`df[,c(col1, col2)]` | `df %>% dplyr::select(col1, col2)`
`merge(df1, df2, by = ..., all.x = ..., all.y = ...)` | `df1 %>% dplyr::left_join(df2, by = ...)` or `dplyr::full_join` or `dplyr::inner_join` or `dplyr::right_join`
_|_
`str()` | `dplyr::glimpse()`
`grep(pattern, x)` | `stringr::str_which(string, pattern)`
`gsub(pattern, replacement, x)` | `stringr::str_replace(string, pattern, replacement)`
`ifelse(test_expression, yes, no)`| `if_else(condition, true, false)`
Nested: `ifelse(test_expression1, yes1, ifelse(test_expression2, yes2, ifelse(test_expression3, yes3, no)))` | `case_when(test_expression1 ~ yes1,  test_expression2 ~ yes2, test_expression3 ~ yes3, TRUE ~ no)`
`proc.time()` | `tictoc::tic()` and `tictoc::toc()`

For a more extensive set of syntactical translations to Tidyverse, you can check out [this document](https://tavareshugo.github.io/data_carpentry_extras/base-r_tidyverse_equivalents/base-r_tidyverse_equivalents.html#reshaping_data).


Working with Tidyverse within functions can be somewhat of a pain due to non-standard evaluation (NSE) semantics. If you're an avid function writer, we'd recommend checking out the following resources:

- [Tidy Eval in 5 Minutes](https://www.youtube.com/watch?v=nERXS3ssntw) (video)
- [Tidy Evaluation](https://tidyeval.tidyverse.org/index.html) (e-book)
- [Data Frame Columns as Arguments to Dplyr Functions](https://www.brodrigues.co/blog/2016-07-18-data-frame-columns-as-arguments-to-dplyr-functions/) (blog)
- [Standard Evaluation for *_join](https://stackoverflow.com/questions/28125816/r-standard-evaluation-for-join-dplyr) (stackoverflow)
- [Programming with dplyr](https://dplyr.tidyverse.org/articles/programming.html) (package vignette)

### Optimizing for Performance

It may also be the case that you're working with very large datasets. Generally I would define this as 10+ million rows. As is outlined in this document, the 3 main players in the data analysis space are Base R, `Tidvyerse` (more specificially, `dplyr`), and `data.table`. For a majority of things, Base R is inferior to both `dplyr` and `data.table`, with concise but less clear syntax and less speed. `Dplyr` is architected for medium and smaller data, and while its very fast for everyday usage, it trades off maximum performance for ease of use and syntax compared to `data.table`. An overview of the `dplyr` vs `data.table` debate can be found in [this stackoverflow post](https://stackoverflow.com/questions/21435339/data-table-vs-dplyr-can-one-do-something-well-the-other-cant-or-does-poorly/27840349#27840349) and all 3 answers are worth a read.

You can also achieve a performance boost by running `dplyr` commands on `data.table`s, which I find to be the best of both worlds, given that a `data.table` is a special type of `data.frame` and fairly easy to convert with the `as.data.table()` function. The speedup is due to `dplyr`'s use of the `data.table` backend and in the future this coupling should become even more natural.

## Miscellaneous Best Practices

- For `ggplot` calls and `dplyr` pipelines, do not crowd single lines. Here are some nontrivial examples of "beautiful" pipelines, where beauty is defined by coherence:
  ```
  # Example 1
  school_names = list(
    OUSD_school_names = absentee_all %>%
      filter(dist.n == 1) %>%
      pull(school) %>%
      unique %>%
      sort,

    WCCSD_school_names = absentee_all %>%
      filter(dist.n == 0) %>%
      pull(school) %>%
      unique %>%
      sort
  )
  ```
  ```
  # Example 2
  absentee_all = fread(file = raw_data_path) %>%
    mutate(program = case_when(schoolyr %in% pre_program_schoolyrs ~ 0,
                               schoolyr %in% program_schoolyrs ~ 1)) %>%
    mutate(period = case_when(schoolyr %in% pre_program_schoolyrs ~ 0,
                              schoolyr %in% LAIV_schoolyrs ~ 1,
                              schoolyr %in% IIV_schoolyrs ~ 2)) %>%
    filter(schoolyr != "2017-18")
  ```
  And of a complex `ggplot` call:
  ```
  # Example 3
  ggplot(data=data,
         mapping=aes_string(x="year", y="rd", group=group)) +
    geom_point(aes_string(col=group,shape=group),
               position=position_dodge(width=0.2),size=2.5) +
    geom_errorbar(aes_string(ymin="lb", ymax="ub", col=group),
                  position=position_dodge(width=0.2),
                  width=0.2) +
    geom_point(position=position_dodge(width=0.2),size=2.5) +
    geom_errorbar(aes(ymin=lb, ymax=ub),
                  position=position_dodge(width=0.2),
                  width=0.1) +
    scale_y_continuous(limits=limits,
                       breaks=breaks,
                       labels=breaks) +
    scale_color_manual(std_legend_title,values=cols,labels=legend_label) +
    scale_shape_manual(std_legend_title,values=shapes, labels=legend_label) +
    geom_hline(yintercept=0, linetype="dashed") +
    ylab(yaxis_lab) +
    xlab("Program year") +
    theme_complete_bw() +
    theme(strip.text.x = element_text(size = 14),
          axis.text.x = element_text(size = 12)) +
    ggtitle(title)
  ```
  Imagine (or perhaps mournfully recall) the mess that can occur when you don't strictly style a complicated `ggplot` call. Trying to fix bugs and ensure your code is working can be a nightmare. Now imagine trying to do it with the same code 6 months after you've written it. Invest the time now and reap the rewards as the code practically explains itself, line by line.
