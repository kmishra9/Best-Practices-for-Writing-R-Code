# Best Practices & Style Guide for Writing R Code

##  Why Best Practices Are Important

Writing good code that adheres to many of the best practices listed here makes your code more digestible to anyone else who examines your code at any point. That could include collaborators, study reviewers, or even yourself, 6 months into the future. Writing best-practice code also makes the process of finding bugs, fixing them, comparing replicated code, and making large breaking changes much easier.

## List of Best Practices

### Project Structure

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
3. **Bash scripts** are a useful component of a reproducible workflow. At many of the directory levels (i.e. in `3 - Analysis`), there is a bash script that runs each of the analysis scripts. This is exceptionally useful when data "upstream" changes -- you simply run the bash script. For big data workflows, the concept of "backgrounding" a Bash script allows you to start a "job" (i.e. run the script) and leave it overnight to run. At the top level, a bash script (`0-run-project.sh`) that simply calls the directory-level bash scripts (i.e. `0-prep-data.sh`,  `0-run-analysis.sh`, `0-run-figures.sh`, etc.) is a powerful tool to rerun every script in your project. See the included example bash scripts for more details.
4. **Use a Config File** - This is the single most important file for your project. It will be responsible for a variety of common tasks, declare global variables, load functions, declare paths, and more. _Every other file in the project_ will begin with `source("0-config")`, and its role is to reduce redundancy and create an abstraction layer that allows you to make changes in one place (`0-config.R`) rather than 5 different files. To this end, paths which will be reference in multiple scripts (i.e. a `merged_data_path`) can be declared in `0-config.R` and simply referred to by its variable name in scripts. If you ever want to change things, rename them, or even switch from a downsample to the full data, all you would then to need to do is modify the path in one place and the change will automatically update throughout your project. See the example config file for more details.

### Comments

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
4. **Multi-Line Comments** - Occasionally, multi-line comments are necessary. Don't add line breaks manually to a single-line comment for the purpose of making it "fit" on the screen. Instead, in RStudio => Global Options => Code => “Soft-wrap R source files” to have lines wrap around. Format your multi-line comments like the
5. **Function Documentation** - Functions _need_ documentation. For any reproducible workflows, they are essential, because R is dynamically typed. This means, you can pass a string into an argument that is meant to be a `data.table`, or a list into an argument meant for a `tibble`. It is the responsibility of a function's author to document what each argument is meant to do and its basic type. This is an example for documenting a function (inspired by [JavaDocs](https://www.oracle.com/technetwork/java/javase/documentation/index-137868.html#format)):

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

### Variables & Function Calls

1. **Variable Names** - Try to make your variable names both more expressive and more explicit. Being a bit more verbose is not just _okay_ -- its prefferred! For example, instead of naming a variable `vaxcov_1718`, try naming it `vaccination_coverage_2017_18`. Similarly, `flu_res` could be named `absentee_flu_residuals` and you've already made significant progress in making your code more readable and explicit.
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

### Loading & Saving Intermediary Files

### Automated Styling Tools in RStudio

1. **Code Autoformatting**
2. **Assignment Aligner**

### Tidyverse vs. Base R
