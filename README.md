ganalytics
==========

Interact with **Google Analytics** using **R**

S4 classes and methods for interactive use of the Google Analytics core reporting API using R.

Updates
-------

There is now support for user and session level condition based segments as well as sequential segmentation.

Also recently added is Google Management API support (still a work in progress) - so now you can get details in R about what goals you have defined, accounts, properties, views, etc. There are *write* methods in there too, but these have not been fully tested so perhaps try them under a test login if you want, otherwise just avoid using the "INSERT", "UPDATE" and "DELETE" methods.

There is also some basic support for the Google Tag Manager API, but again, still work in progress.

Installation
------------
### 1. Install the necessary packages into R

#### Warning
* The ganalytics package is currently under development.
* The installation procedure below installs directly from the respective GitHub repositories.

#### Prerequisites
* Ensure you have installed the latest version of [R](http://cran.r-project.org/)
* If using Windows, you will also need the latest version of [RTools](http://cran.r-project.org/bin/windows/Rtools/)
* For other operating systems, please refer to [installation instructions for devtools](https://github.com/hadley/devtools/blob/master/README.md)

#### Execute the following statements in R to install ganalytics:

```r
# Install the latest version of devtools via CRAN
install.packages("devtools", dependencies = TRUE)
# Install ganalytics via the GitHub repository.
devtools::install_github("jdeboer/ganalytics")
# End
```

#### Now, restart R.
* This is important to ensure you have a clean workspace to avoid possible errors.

### 2. Prepare your Google API application _(you only need to do this once)_
* Browse to [**Google API Console**] (https://code.google.com/apis/console/)
* Check you are **signed into Google** with the account you wish to use.
* Choose **Create Project** from the Google API Console and give your project a name (or choose an existing project if you have one already).
* From the **APIs** page, enable the **Analytics API**. You may also want to enable the **Tag Manager API** if you wish to try that.
* You will need to **agree** and **accept** the Google APIs and Analytics API Terms of Service to proceed.
* Go to the **Credentials** page and click the blue button **Create New Client ID**.
* Select **Installed Application** as the application type.
* Select **Other** as the Installed Application Type.
* Note your **Client ID** and **Client Secret** and download the JSON file to your R working directory.

_Note: For further information about Google APIs please refer to the [References section](https://github.com/jdeboer/ganalytics/blob/master/README.md#useful-references) at the end of this document._

### 3. Set your system environment variables _(this is optional but recommended)_
* Add the following two user variables:

  |     | Variable name                 | Variable value         |
  | --- | ----------------------------- | ---------------------- |
  |   1 | `GOOGLE_APIS_CONSUMER_ID`     | `<Your client ID>`     |
  |   2 | `GOOGLE_APIS_CONSUMER_SECRET` | `<Your client secret>` |

* To do this in Windows:
  * Search for and select **"Edit Environment Variables For Your Account"** from the Start menu.
  * Within the **Environment Variables** window, add the above **User Variables** by selecting **New** and entering the **Variable Name** and **Variable Value**, then click **OK**. Do this for both variables listed in the above table.
  * Click **OK**.
  * **Restart** your computer for the new environment variables to take effect.
* To do this in Mac OS, there is a free open source utility called [EnvPane](https://github.com/hschmidt/EnvPane)

  _Note: For other operating systems please refer to the Reference section at the end of this document._

### 4. Authenticate and attempt your first query with ganalytics
* To perform a query you will need to obtain your **Google Analytics view ID**. This can be accessed in a number of ways:
  * Using the [Google Analytics Query Explorer tool](http://ga-dev-tools.appspot.com/explorer/)
  * The **Admin page** in Google Analytics under **View Settings**, or
  * The browser's address bar when viewing a report in Google Analytics - look for the digits between the letter **'p'** and trailing **'/'**, e.g. `.../a11111111w22222222p33333333/` shows a view ID of `33333333`.

* **If you do not provide a view ID then:**
  * If you have access to only one Google Analytics account, with one property, then ganalytics will automatically select the default view for you.
  * Otherwise it will select the first property from the first account it finds in the list of accounts you have access to.

* Return to R and execute the following to load the ganalytics package:

  ```r
  library(ganalytics)
  ```

* If you successfully set your system environment variables in step 3 above, then execute the following, optionally providing the email address you use to sign-in to Google Analytics:

  ```r
  my_creds <- GoogleApiCreds("you@domain.com")
  ```

* Otherwise do one of the following:
  * If you downloaded the JSON file containing your Google API app credentials, then provide the filename:

    ```r
    my_creds <- GoogleApiCreds("you@domain.com", "client_secret.json")
    ```
  * Instead of a filename you may supply the `client_id` and `client_secret` directly:

    ```r
    my_creds <- GoogleApiCreds(
      "you@domain.com",
      list(client_id = "<client id>", client_secret = "<client secret>")
    )
    ```

* Now formulate and run your Google Analytics query, remembering to substitute `view_id` with the view ID you wish to use:

  ```r
  myQuery <- GaQuery( view_id, my_creds )
  GetGaData(myQuery)
  ```

* You should then be directed to *http://accounts.google.com* within your default web browser asking you to sign-in to your Google account if you are not already. Once signed-in you will be asked to grant read-only access to your Google Analytics account for the Google API project you created in step 1.
* Make sure you are signed into the Google account you wish to use, then grant access by selecting **"Allow access"**. You can then close the page and return back to R.

If you have successfully executed all of the above R commands you should see the output of the default ganalytics query; sessions by day for the past 7 days, for example:

```
        date sessions
1 2015-03-27     2988
2 2015-03-28     1594
3 2015-03-29     1912
4 2015-03-30     3061
5 2015-03-31     2609
6 2015-04-01     2762
7 2015-04-02     2179
8 2015-04-03     1552
```

_Note: A small file will be saved to your home directory ('My Documents' in Windows) containing your new reusable authentication token._

Examples
--------

As demonstrated in the installation steps above, before executing any of the following examples:

1. load the ganalytics package
2. generate a `gaQuery` object with a Google Analytics view ID and API app credentials assigned to it.

### Asumptions

**The following examples assume you have successfully completed the above steps.**

### Example 1 - Setting the date range

```r
# Set the date range from 1 January 2013 to 31 May 2013: (Dates are specified in the format "YYYY-MM-DD".)
GaDateRange(myQuery) <- c("2013-01-01", "2013-05-31")

myData <- GetGaData(myQuery)
summary(myData)

# Adjust the start date to 1 March 2013:
GaStartDate(myQuery) <- "2013-03-01"
# Adjust the end date to 31 March 2013:
GaEndDate(myQuery) <- "2013-03-31"

myData <- GetGaData(myQuery)
summary(myData)
# End
```

### Example 2 - Choosing what metrics to report

```r
# Report number of page views instead
GaMetrics(myQuery) <- "pageviews"

myData <- GetGaData(myQuery)
summary(myData)

# Report both pageviews and sessions
GaMetrics(myQuery) <- c("pageviews", "sessions")
# These variations are also acceptable
GaMetrics(myQuery) <- c("ga:pageviews", "ga.sessions")

myData <- GetGaData(myQuery)
summary(myData)
# End
```

### Example 3 - Selecting what dimensions to split your metrics by

```r
# Similar to metrics, but for dimensions
GaDimensions(myQuery) <- c("year", "week", "dayOfWeek", "hour")

# Lets set a wider date range
GaDateRange(myQuery) <- c("2012-10-01", "2013-03-31")

myData <- GetGaData(myQuery)
head(myData)
tail(myData)
# End
```

### Example 4 - Sort by

```r
# Sort by descending number of pageviews
GaSortBy(myQuery) <- "-pageviews"

myData <- GetGaData(myQuery)
head(myData)
tail(myData)
# End
```

### Example 5 - Row filters

```r
# Filter for Sunday sessions only
sundayExpr <- GaExpr("dayofweek", "=", "0")
GaFilter(myQuery) <- sundayExpr

myData <- GetGaData(myQuery)
head(myData)

# Remove the filter
GaFilter(myQuery) <- NULL

myData <- GetGaData(myQuery)
head(myData)
# End
```

### Example 6 - Combining filters with AND

```r
# Expression to define Sunday sessions
sundayExpr <- GaExpr("dayofweek", "=", "0")
# Expression to define organic search sessions
organicExpr <- GaExpr("medium", "=", "organic")
# Expression to define organic search sessions made on a Sunday
sundayOrganic <- GaAnd(sundayExpr, organicExpr)
GaFilter(myQuery) <- sundayOrganic

myData <- GetGaData(myQuery)
head(myData)

# Let's concatenate medium to the dimensions for our query
GaDimensions(myQuery) <- c(GaDimensions(myQuery), "medium")

myData <- GetGaData(myQuery)
head(myData)
# End
```

### Example 7 - Combining filters with OR

```r
# In a similar way to AND
loyalExpr <- GaExpr("sessionCount", "!~", "^[0-3]$") # Made more than 3 sessions
recentExpr <- GaExpr("daysSinceLastSession", "~", "^[0-6]$") # Visited sometime within the past 7 days.
loyalOrRecent <- GaOr(loyalExpr, recentExpr)
GaFilter(myQuery) <- loyalOrRecent

myData <- GetGaData(myQuery)
summary(myData)
# End
```

### Example 8 - Filters that combine ORs with ANDs

```r
loyalExpr <- GaExpr("sessionCount", "!~", "^[0-3]$") # Made more than 3 sessions
recentExpr <- GaExpr("daysSinceLastSession", "~", "^[0-6]$") # Visited sometime within the past 7 days.
loyalOrRecent <- GaOr(loyalExpr, recentExpr)
sundayExpr <- GaExpr("dayOfWeek", "=", "0")
loyalOrRecent_Sunday <- GaAnd(loyalOrRecent, sundayExpr)
GaFilter(myQuery) <- loyalOrRecent_Sunday

myData <- GetGaData(myQuery)
summary(myData)

# Perform the same query but change which dimensions to view
# - Notice "daysSince" is conveniently accepted as a partial match for "daysSinceLastSession".
GaDimensions(myQuery) <- c("sessionCount", "daysSince", "dayOfWeek")

myData <- GetGaData(myQuery)
summary(myData)
# End
```

### Example 9 - Sorting 'numeric' dimensions (continuing from example 8)

```r
# Continuing from example 8...

# Change filter to loyal session AND recent sessions AND visited on Sunday
loyalAndRecent_Sunday <- GaAnd(loyalExpr, recentExpr, sundayExpr)
GaFilter(myQuery) <- loyalAndRecent_Sunday

# Sort by decending visit count and ascending days since last visit.
GaSortBy(myQuery) <- c("-sessionCount", "+daysSince")
myData <- GetGaData(myQuery)
head(myData)

# Notice that Google Analytics' Core Reporting API doesn't recognise 'numerical' dimensions as
# ordered factors when sorting. We can use R to sort instead, using a plyr::arrange function.
library(plyr)
myData <- arrange(myData, desc(sessionCount), daysSinceLastSession)
head(myData)
tail(myData)
# End
```

### Example 10 - Session segmentation

```r
# Visit segmentation is expressed similarly to row filters and supports AND and OR combinations.
# Define a segment for sessions where a "thank-you", "thankyou" or "success" page was viewed.
thankyouExpr <- GaExpr("pagePath", "~", "thank\\-?you|success")
GaSegment(myQuery) <- thankyouExpr

# Reset the filter
GaFilter(myQuery) <- NULL

# Split by traffic source and medium
GaDimensions(myQuery) <- c("source", "medium")

# Sort by decending number of sessions
GaSortBy(myQuery) <- "-sessions"

myData <- GetGaData(myQuery)
head(myData)
# End
```

### Example 11 - Using automatic pagination to get more than 10,000 rows of data per query

```r
# Sessions by date and hour for the years 2011 (leap year) and 2012: 2 * 365.5 * 24 = 17544 rows
# First let's clear any filters or segments defined previously
GaFilter(myQuery) <- NULL
GaSegment(myQuery) <- NULL
# Define our date range
GaDateRange(myQuery) <- c("2011-01-01", "2012-12-31")
# Define our metrics and dimensions
GaMetrics(myQuery) <- "sessions"
GaDimensions(myQuery) <- c("date", "dayOfWeek", "hour")
# Let's allow a maximum of 17544 rows (default is 10000)
GaMaxResults(myQuery) <- 17544

myData <- GetGaData(myQuery)
nrow(myData)

## Let's use plyr::ddply to analyse the data
library(plyr)

# Sessions by day of week
sessions_by_dayOfWeek <- ddply(myData, ~dayOfWeek, summarise, sessions = sum(sessions))
with(sessions_by_dayOfWeek, barplot(sessions, names.arg=dayOfWeek))

# Sessions by hour of day
sessions_by_hour <- ddply(myData, ~hour, summarise, sessions = sum(sessions))
with(sessions_by_hour, barplot(sessions, names.arg=hour))
# End
```

### Example 12 - Using ggplot2
To run this example first install ggplot2 if you haven't already.

```r
install.packages("ggplot2")
```

Once installed, then run the following example.

```r
library(ggplot2)
library(plyr)

# Sessions by date and hour for the years 2011 (leap year) and 2012: 2 * 365.5 * 24 = 17544 rows
# First let's clear any filters or segments defined previously
GaFilter(myQuery) <- NULL
GaSegment(myQuery) <- NULL
# Define our date range
GaDateRange(myQuery) <- c("2011-01-01", "2012-12-31")
# Define our metrics and dimensions
GaMetrics(myQuery) <- "sessions"
GaDimensions(myQuery) <- c("date", "dayOfWeek", "hour", "isMobile")
# Let's allow a maximum of 40000 rows (default is 10000)
GaMaxResults(myQuery) <- 40000

myData <- GetGaData(myQuery)

# Sessions by hour of day and day of week
avg_sessions_by_hour_wday_mobile <- ddply(
  myData,
  ~hour + dayOfWeek + isMobile,
  summarise,
  sessions = mean(sessions)
)

# Relabel the days of week
levels(avg_sessions_by_hour_wday_mobile$dayOfWeek) <- c(
  "Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"
)

# Plot the summary data
qplot(
  x = hour,
  y = sessions,
  data = avg_sessions_by_hour_wday_mobile,
  facets = ~dayOfWeek,
  fill = isMobile,
  geom = "bar",
  position = "stack",
  stat = "identity"
)

# End
```

Useful references
-----------------

1. [Google Analytics Core Reporting API reference guide](https://developers.google.com/analytics/devguides/reporting/core/v3/reference)
2. [Google Analytics Dimensions and Metrics reference](https://developers.google.com/analytics/devguides/reporting/core/dimsmets)
3. [Creating a Google API project](https://developers.google.com/console/help/#creatingdeletingprojects)
4. [Generating an OAuth 2.0 client ID for Google APIs](https://developers.google.com/console/help/#generatingoauth2)
5. [Using OAuth 2.0 for Installed Applications](https://developers.google.com/accounts/docs/OAuth2InstalledApp)
6. [Setting environment variables in Windows XP](http://support.microsoft.com/kb/310519)
7. [Setting environment variables in earlier versions of OSX](https://developer.apple.com/library/mac/#documentation/MacOSX/Conceptual/BPRuntimeConfig/Articles/EnvironmentVars.html)
8. [EnvPane utility for setting environment variables in OSX](https://github.com/hschmidt/EnvPane)
9. [Setting environment variables in Ubuntu Linux](https://help.ubuntu.com/community/EnvironmentVariables)
