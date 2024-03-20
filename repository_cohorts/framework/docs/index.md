---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 2rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: none;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

p {
  max-width:2000px;
}

</style>


# Repository Cohorts

<b style="color:red">Please note that this page is in active development</b>

## What is this page?
*This page is a very brief demonstration of the concept of repository cohorts.* It is designed 
to act as a companion to a talk that will be given at Open Source Summit North America 2024 
titled 
["Repository Cohorts: How OSPO's Can Programmatically Categorize All Their Repositories"](https://ossna2024.sched.com/event/1aBPX/repository-cohorts-how-ospos-can-programmatically-categorize-all-their-repositories-justin-gosses-microsoft-natalia-luzuriaga-remy-decausemaker-isaac-milarsky-centers-for-medicare-medicaid-services?iframe=no).

## What to know about the data being shown here?
Data on this page is loaded from a CSV file and is not updated post 2024-03-19. 
It is from a fixed snapshot in time that is now out of date.

It shows data from from real repositories. 
These repositories are all from GitHub organizations monitored by Microsoft either owns or has a hand in the 
governance of and therefore collects data on.
All the data shown is public data that we collect and harvest via the GitHub API, just like anyone else could.


```js
// IMPORT LIBRARIES

import Tabulator from "https://cdn.jsdelivr.net/npm/tabulator-tables/+esm";

import {sql} from "npm:@observablehq/duckdb";

import * as duckdb from "npm:@duckdb/duckdb-wasm";
import {DuckDBClient} from "npm:@observablehq/duckdb";

// IMPORT DATA 
const repos = FileAttachment("./data/microsoft_repos_public_20240319.csv").csv();

```
## How to adapt this for your own purposes

To see the code, look at
the [cohort.js](https://github.com/microsoft/OSPO/blob/main/repository_cohorts/framework/docs/components/cohorts.js) file 
and [index.md](https://github.com/microsoft/OSPO/blob/main/repository_cohorts/framework/docs/index.md) file in the repository that builds this page. The open source [Observable Framework](https://observablehq.com/framework/) is used to generate the data visualization static site.

## Data transformation steps

Internally, Microsoft uses [Kusto Query Language](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
to generate the cohort columns. The logic used on this page is identical but in JavaScript. 

The initial data processing steps are load the CSV file, 
rename several of the columns as needed, then calculate additional columns, including cohort columns. 
The data is then ready for further analysis to generate insights.

Unprocessesd raw data

```js
display(repos)
```

```js

//  Data utilities for plotting
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(repos, (D) => -D.length, (d) => d.owner).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});

const colorLanguage = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(repos, (D) => -D.length, (d) => d.language).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});
```

```js

// rename column names
import { renameKeys} from "./components/cohorts.js";

const changeKeyNamesObject = {
    "FullName": "full_name",
    "OwnerLogin":"owner" ,
    "Size":"size" ,
    "StargazersCount": "stargazers_count" ,
    "OpenIssuesCount": "open_issues_count",
    "ForksCount":"forks_count" ,
    "SubscribersCount":"subscribers_count",
    "Homepage": "homepage" ,
    "HasIssues": "has_issues",
    "count_unique_contributor_Ids": "commit_stats_total_committers",
    "Description": "description",
    "UpdatedAt":"updated_at" ,
    "CreatedAt":"created_at"
}

const reposReName = renameKeys(repos, changeKeyNamesObject)

```

Keys are renamed to mirror ecosyste.ms key names, so this example code can be reused more easily

The renamed key mapping:
```js
display(changeKeyNamesObject)
```

The data table with renamed keys:
```js

display(reposReName )
```

```js

import {countsCohortGroup} from "./components/cohorts.js"

// CREATING COHORTS 

import {createCohortColumns} from "./components/cohorts.js";

import {repos_cohort_processed_BaseCohorts} from "./components/cohorts.js";

const repos_cohort_processedSemi = repos_cohort_processed_BaseCohorts(reposReName )

const dataCohortsWithTrueInGroup = countsCohortGroup(repos_cohort_processedSemi,[ "cohort_sample","cohort_age","cohort_committers","cohort_Nadia"])


```

Calculated columns are created, including cohort columns. These are added on to the existing columns.

```js

display(dataCohortsWithTrueInGroup)
```

### Table showing all repositories with calculated cohort columns.
Scroll to the far right to see cohort columns

```js
const table_all2 = 
  view(Inputs.table(dataCohortsWithTrueInGroup, {
   rows:16
  }))

```
--------------------------
## What problems do repository cohorts solve?

### No wants to read 3,000 READMEs

OSPOs often have to manage, enforce compliance, and recommend best practice for 
many hundreds or thousands of repositories. 
This page shows over 10,000 repositories. That is way too many to read, so 
there's a strong tendancy for OSPOs to treat every repository the same and make 
policy and best practive recommendations for an average repository or more 
accurately an average top of mind repository.

### Metadata but make it more easily reusable

Metadata reduces the need to read thousands of repositories by letting OSPOs 
understand repositories according to their easily measurable characteristics.

This typically requiress OSPOs, or a centralized data team that OSPO is working with, to collect 
code platform metadata (GitHub, Azure DevOps, etc.) on a reoccurring basis and then 
provide it for reuse to others internally from a database and/or API.

Working with raw repository metadata fields, however, requires thought about how to combine 
raw data, where to make thresholds, etc. If this is done again and again for each potential 
use case of the metadata, it imposes time and cognitive burdens that limit how often 
the metadata is used to make data informed decisions. Additionally, small differences 
in where to make a cut off for categories such as highly forked repositories versus normally forked 
repositories can lead to inefficiencies in applying the learnings from one project to another project.

Repository cohorts attempts to solve these problems by being standardized labels for repostories. 
They have meanings that are easy to remember and can be reused with very little effort as they become 
additional columns in the repository metadata table.

### Repository cohort structure
Repository cohorts are either true or false. There can be groups of cohorts that split a dimension. 
For exampple, there can be cohorts of repository age of baby, toddler, teenager, adult, and senior. 
Each repository, or row in the table, will be true for only one of these cohorts and false for all the 
other cohorts in the group. Additionally, there will be a column that has the column name of the 
cohort that is true for each cohort group.

### Benefits of repository cohorts
These characteristics of repository cohorts makes it easier to generate counts of cohorts from a single cohort group
for any filtered group of repositories. Additionally, you can combine cohorts using 
simple AND OR statements without having to remember thresholds or rewrite extensive queries.
The idea is that this makes it faster and easier to gain insights into the communities building each repository
across thousands of repositories compared with reading READMEs & issues or working with raw metadata fields. 

--------------------------

## Initial visualizations of main repository cohorts
*Note that the tool tips have a limit of how many repositories they will show information for.*

```js

Plot.plot({
  title: "Cohort analysis age",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barY(
  dataCohortsWithTrueInGroup,
  Plot.groupX({ y: "count" }, { x: "cohort_age_trueValueInGroup", title: "full_name",lineWidth: 74, marginBottom: 40})
)
  ]
})

```

```js
Plot.plot({
  title: "Cohort analysis age vs. committers size",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barY(
  dataCohortsWithTrueInGroup,
  Plot.groupX({ y: "count" }, { x: "cohort_age_trueValueInGroup", fill: "cohort_committers_trueValueInGroup", title: "full_name", lineWidth: 74, marginBottom: 40, sort: { y: "y", reverse: true }})
)
  ]
})
```

```js
Plot.plot({
  title: "Cohort analysis cohort_Nadia vs. age in days",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barY(
  dataCohortsWithTrueInGroup,
  Plot.groupX({ y: "count" }, { x: "cohort_Nadia_trueValueInGroup", fill: "age_in_days" , title: "full_name",  lineWidth: 74, marginBottom: 40, sort: { y: "y", reverse: true }})
)
  ]
})
```

```js
Plot.plot({
  title: "Nadia community cohorts vs. days since last update",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barY(
  dataCohortsWithTrueInGroup,
  Plot.groupX({ y: "count" }, { x: "cohort_Nadia_trueValueInGroup", title: "full_name", fill: "daysSinceUpdated", lineWidth: 74, marginBottom: 40, sort: { y: "y", reverse: true }})
)
  ]
})
```
-----------------

## User inputs are used in a SQL query that filters data in tables and plots below

```js

const dbRepos = DuckDBClient.of({reposSQL: dataCohortsWithTrueInGroup});

// need to create a const for organizationList

```

```js

//const organization = view(Inputs.select(organizationList, {multiple: true, label: "select organizations"}))
const sizeMin = view(Inputs.range([0, 5000000], {label: "greater than this size in bytes",value:1000}));
const sizeMax = view(Inputs.range([10, 100000000], {label: "less than this size in bytes",value:5000000}));
const stargazer_count_min = view(Inputs.range([0, 10000], {label: "more than this many stargazers",value:20}));
const fork_count_min = view(Inputs.range([0, 4000], {label: "more than this many forks",value:5}));
const max_days_since_updated = view(Inputs.range([0, 10000], {label: "less than this many days since updated",value:30}));
const min_days_since_updated = view(Inputs.range([0, 10000], {label: "more than this many forks",value:0}));
const archived = view(Inputs.select([null].concat(["true","false"]), {label: "archived", value:"false"}));
const limitNumberRowsToShow = view(Inputs.range([0, 15000], {label: "max number of rows to show",value:200,step:1}));
console.log("archived ", archived )
```

```
SELECT * FROM reposSQL WHERE size > ${sizeMin} AND size < ${sizeMax} AND stargazers_count > ${stargazer_count_min} AND daysSinceUpdated < ${max_days_since_updated} AND daysSinceUpdated > ${min_days_since_updated} AND Archived == ${archived} LIMIT ${limitNumberRowsToShow}
```

```js

const dbRepos_FilteredBySQL = dbRepos.sql`SELECT * FROM reposSQL WHERE size > ${sizeMin} AND size < ${sizeMax} AND stargazers_count > ${stargazer_count_min} AND daysSinceUpdated < ${max_days_since_updated} AND daysSinceUpdated > ${min_days_since_updated} AND Archived == ${archived} LIMIT ${limitNumberRowsToShow}`

//console.log("dbRepos_FilteredBySQL",dbRepos_FilteredBySQL)

//const lengthOfdbRepos_FilteredBySQL = await dbRepos_FilteredBySQL.length
//display(lengthOfdbRepos_FilteredBySQL)
```

```js

const table_SQLunfiltered = 
  view(Inputs.table(dbRepos_FilteredBySQL))

```

```js
Plot.plot({
    title: "Nadia community cohorts vs. Organization",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 140,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barX(
      dbRepos_FilteredBySQL,
      Plot.groupY(
        { x: "count" },
        { fill: "cohort_Nadia_trueValueInGroup", y: "owner", title: "full_name",  sort: { x: "x", reverse: true }}
      )
    )
  ]
})
```


```js
Plot.plot({
  title: "Nadia community cohorts vs. days since last update",
  marginTop: 20,
  marginRight: 20,
  marginBottom: 30,
  marginLeft: 40,
  grid: true,
  width: 1000,
  color: { legend: true } ,
  marks: [
    Plot.barY(
  dbRepos_FilteredBySQL,
  Plot.groupX({ y: "count" }, { x: "cohort_Nadia_trueValueInGroup", title: "full_name", fill: "daysSinceUpdated", lineWidth: 74, marginBottom: 40, sort: { x: "x", reverse: true }})
)
  ]
})
```


### What is a Nadia Cohort?

The Nadia cohort group attemps to capture a description of community user / contributor patterns that reveal something about the structure of a community that builds a repository. This idea comes from [Nadia Asparouhova's](https://nadia.xyz/) book ["Working in Public: The Making and Maintenance of Open Source Software"](https://press.stripe.com/working-in-public) Stripe Matter Incorporated, 2020, pp59-65, which  categorizes open source project communities as being federations, clubs, stadiums, or toys. No exact metrics are given 
for defining the boundaries, but rather they are described using a matrix of high or low contributor growth and high or low user growth.

|                              | HIGH USER GROWTH | LOW USER GROWTH |
|------------------------------|------------------|-----------------|
| **HIGH CONTRIBUTOR GROWTH**  | Federations      | Clubs           |
| **LOW CONTRIBUTOR GROWTH**   | Stadiums         | Toys            |  

For repository cohorts, we have taken these ideas and modified them to work with easily available repository metadata.
We have also added a category of "middle" or "mid" repositories that are not quite one of those four but some where 
in between them. There is also a "missing data" Nadia cohort for when we lack the repository metadata needed to calculate a Nadia cohort.

#### Metadata thresholds for Nadia community cohorts
The metadata thresholds used for what we are calling Nadia cohorts is based on a combination of 
community size, stargazer count, and ratio of stargazers vs. committer count.


|                              | ratioStargazersVsCommitters > 2 | ratioStargazersVsCommitters < 2 |
|------------------------------|------------------|-----------------|
| **> 60 contributors**       | Federation cohort      | Club cohort         |

|       |    | |
|------------------------------|------------------|-----------------|
| **6 < contributors < 60**      | Mid cohort   |  |

|                              | stargazers_count > 100 | stargazers_count < 100 |
|------------------------------|------------------|-----------------|
| **< 6 contributors**       | Stadium cohort             | Toy cohort                  |  

These thresholds seem to work well for Microsoft repositories. We would be interested in learning what 
other organizations use if they model similar cohorts, so please add an issue with any feedback or comments.

To describe these cohorts in plain english: 
1. **Federations** are built by a large community with a much larger silent group of watchers.
2. **Clubs** are built but a large community with the ratio of silent watchers closer to the size of the contributor community. 
3. **Stadium** is built by a small community with a larger proportion of silent watchers.
4. **Toy** is built by a small community with a small number of silent watchers. 
5. **mid** is built by a moderately sized community that sits between the other cohorts in terms of community size.
