# Best Practices for Writing R Code

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
For brevity, not every directory is "expanded", but we can glean some important takeaways from what we _do_ see:

1. All files and directories are numbered! This makes the jumble of alphabetized filenames much more coherent and places similar code and files next to one another. This also helps us understand how data flows from start to finish and allows us to easily map a script to its output (i.e. `2 - Analysis/1 - Absentee-Mean/1-absentee-mean-primary.R` => `5 - Results/1 - Absentee-Mean/1-absentee-mean-primary.RDS`). If you take nothing else away from this guide, this is the single most helpful suggestion to make your workflow more coherent.
2. Directories have capitalized letters and spaces but individual files do not.
3. Both `.gitignore` and `.Rproj` files are present. These are subtle but important additions.
    - There is a standardized `.gitignore` for `R` which you [can download](https://github.com/github/gitignore/blob/master/R.gitignore) and add to your project. This ensures you're not committing log files or things that would otherwise best be left ignored to GitHub.
    - An "R Project" can be created within RStudio by going to `File >> New Project`. Depending on where you are with your research, choose the most appropriate option. This will save preferences, working directories, and even the results of running code/data (though I'd recommend starting from scratch each time you open your project, in general). Then, ensure that whenever you are working on that specific research project, you open your created project to enable the full utility of `.Rproj` files.
4. Bash scripts are a useful component of a reproducible workflow. At many of the directory levels (i.e. in `3 - Analysis`), there is a bash script that runs each of the analysis scripts. This is exceptionally useful when data "upstream" changes -- you simply run the bash script. For big data workflows, the concept of "backgrounding" a Bash script allows you to start a "job" (i.e. run the script) and leave it overnight to run. At the top level, a bash script (``0-run-project.sh`) that simply calls the directory-level bash scripts (i.e. `0-prep-data.sh`,  `0-run-analysis.sh`, `0-run-figures.sh`, etc.) is a powerful tool to rerun every script in your project. See the included example bash scripts for more details.
5. Finally, you may have noticed the `0-config.R` file. This is the single most important file for your project. It will be responsible for a variety of common tasks, declare global variables, load functions, declare paths, and more. _Every other file in the project_ will begin with `source("0-config")`, and its role is to reduce redundancy and create an abstraction layer that allows you to make changes in one place (`0-config.R`) rather than 5 different files. To this end, paths which will be reference in multiple scripts (i.e. a merged_data_path) can be declared in `0-config.R` and simply referred to by its variable name in scripts. If you ever want to change things, rename them, or even switch from a downsample to the full data, all you would then to need to do is modify the path in one place and the change will automatically update throughout your project. See the example config file for more details.

### Comments

1. File Headers
2. File Structuring
3. Single-Line Comments

### Variables
