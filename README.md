<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Travis-CI Build Status](https://travis-ci.org/jaredlander/resumer.svg?branch=master)](https://travis-ci.org/jaredlander/resumer) [![Coverage Status](https://img.shields.io/codecov/c/github/jaredlander/resumer/master.svg)](https://codecov.io/github/jaredlander/resumer?branch=master) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/resumer)](http://cran.r-project.org/package=resumer)

resumer
=======

This package stores information for your CV in a CSV and compiles a nicely formatted LaTeX CV.

Data
----

Your jobs should be stored in a CSV with these names: JobName, Company, Location, Title, Start, End, Bullet, BulletName, Type, Description.

An example file can be found in [`inst/examples/Jobs.csv`](https://github.com/jaredlander/resumer/blob/master/inst/examples/Jobs.csv) or by running

``` r
data(jobs)
head(jobs)
```

| JobName      | Company               | Location     | Title  |  Start| End     | Bullet                                                              |  BulletName| Type | Description |
|:-------------|:----------------------|:-------------|:-------|------:|:--------|:--------------------------------------------------------------------|-----------:|:-----|:------------|
| Tech Startup | Pied Piper            | New York, NY | CTO    |   2013| Present | Set up company's computing platform                                 |           1| Job  |             |
| Tech Startup | Pied Piper            | New York, NY | CTO    |   2013| Present | Designed data strategy overseeing many datasources                  |           2| Job  |             |
| Tech Startup | Pied Piper            | New York, NY | CTO    |   2013| Present | Constructed statistical models for predictive analytics of big data |           3| Job  |             |
| Large Bank   | Goliath National Bank | New York, NY | Quant  |   2011| 2013    | Built quantitative models for derivatives trades                    |           1| Job  |             |
| Large Bank   | Goliath National Bank | New York, NY | Quant  |   2011| 2013    | Wrote algorithms using the R statistical programming language       |           2| Job  |             |
| Bank Intern  | Goliath National Bank | New York, NY | Intern |   2010|         | Got coffee for senior staff                                         |           1| Job  |             |

A helper function, `createJobFile`, creates a CSV with the correct headers.

Each row represents a detail about a job. So a job may take multiple rows.

The columns are:

-   `JobName`: Name identifying this job. This is identifying information used when selecting which jobs to display.
-   `Company`: Name of company.
-   `Location`: Physical location of job.
-   `Title`: Title held at job.
-   `Start`: Start date of job, usually represented by a year.
-   `End`: End date of job. This would ordinarily by a year, 'Present' or blank.
-   `Bullet`: The detail about the job.
-   `BulletName`: Identifier for this detail, used when selecting which details to display.
-   `Type`: Should be either `Job` or `Research`.
-   `Description`: Used for a quick blurb about research roles.

Usage
-----

In order to make this package as universal as possible it is designed for some information to be input in the yaml header and some in R code. Creating a new file using the template through RStudio will be easiest.

### yaml header

Here you put your name, address, the location of the jobs CSV, education information and any highlights. Remember, proper indenting is required for yaml.

The `name` and `address` fields are self explanatory. `output` takes the form of `package::function` which for this package is `resumer::resumer`.

The location of the jobs CSV is specified in the `JobFile` slot of the `params` entry. This should be the absolute path to the CSV.

These would look like this.

``` yaml
---
name: "Generic Name"
address: "New York"
output: resumer::resumer
params:
    JobFile: "examples/jobs.csv"
---
```

Supplying education information is done as a list in the `education` entry, with each school containing slots for `school`, `dates` and optionally `notes`. Each slot of the list is started with a `-`. The `notes` slot starts with a `|` and each line (except the last line) must end with two spaces.

For example:

``` yaml
---
education:
-   school: "Hudson University"
    dates: "2007--2009"
    notes: |
        GPA 3.955  
        Master of Arts in Statistics
-   school: "Smallville College"
    dates: "2000--2004"
    notes: |
        Cumulative GPA 3.838 Summa Cum Laude, Honors in Mathematics  
        Bachelor of Science in Mathematics, Journalism Minor  
        The Wayne Award for Excellence in Mathematics  
        Member of Pi Mu Epsilon, a national honorary mathematics society
---
```

Providing a `highlights` section and confirming that they should be displayed with \`doHighlights.

Each `bullet` in the `highlights` entry should be a list slot started by `-`. For example.

``` yaml
---
doHighlights: yes
highlights:
-   bullet: Author of \emph{Pulitzer Prize} winning article
-   bullet: Organizer of \textbf{Glasses and Cowl} Meetup
-   bullet: Analyzed global survey by the \textbf{Surveyors Inc}
-   bullet: Professor of Journalism at \textbf{Hudson University}
-   bullet: Thesis on \textbf{Facial Recognition Errors}
-   bullet: Served as reporter in \textbf{Vientiane, Laos}
---
```

The exact structure of the yaml section will change as tweaks are made to the underlying template.

### R Code

Jobs and details are selected for display by building a list of lists named `jobList`. Each inner list represents a job and should have three unnamed elements: - `CompanyName` - `JobName` - Vector of `BulletName`s

An example is:

``` r
jobList <- list(
    list("Pied Piper", "Tech Startup", c(1, 3)),
    list("Goliath National Bank", "Large Bank", 1:2),
    list("Goliath National Bank", "Bank Intern", 1:3),
    list("Surveyors Inc", "Survery Stats", 1:2),
    list("Daily Planet", "Reporting", 2:4),
    list("Hudson University", "Professor", c(1, 3:4)),
    list("Hooli", "Coding Intern", c(1:3))
)
```

Research is specified similarly in `researchList`.

``` r
# generate a list of lists of research that list the company name, job name and bullet
researchList <- list(
    list("Hudson University", "Oddie Research", 4:5),
    list("Daily Planet", "Winning Article", 2)
)
```

The job file is read into the `jobs` variable using `read.csv2`.

``` r
library(resumer)
jobs <- read.csv2(params$JobFile, header=TRUE, sep=',', stringsAsFactors=FALSE)
```

The jobs and details are written to LaTeX using a code chunk with `results='asis'`.

Same with research details.

### LaTeX

Regular LaTeX code can be used, such as in specifying an athletics section. Note that this uses a special `rSection` environment.

``` latex
\begin{rSection}{Athletics}
\textbf{Ice Hockey} \emph{Goaltender} | \textbf{Hudson University} | 2000--2004 \\
\textbf{Curling} \emph{Vice Skip} | \textbf{Hudson University} | 2000--2004
\end{rSection}
```

Resources
---------

### LaTeX Templates

-   <https://www.sharelatex.com/blog/2011/03/27/how-to-write-a-latex-class-file-and-design-your-own-cv.html>

### Pandoc

-   <http://pandoc.org/scripting.html>
-   <http://pandoc.org/README.html#header-identifiers-in-html-latex-and-context>
-   <https://github.com/jgm/pandocfilters/blob/master/pandocfilters.py>
-   <http://galahad.well.ox.ac.uk/repro/#latex-output>
