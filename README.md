# gatsby-source-google-sheets-raw

Why go through the hassle of setting up a complicated headless CMS when Google Sheets already has user permissions, revision history, and a powerful UI? 

This source plugin for [Gatsby JS](https://github.com/gatsbyjs/gatsby) will turn any Google Sheets worksheet into a GraphQL type for build-time consumption. 

Forked from `gatsby-source-google-sheets`, but comes with config options for disabling/enabling automatic type casting.

# How to:

## Step 1: set up sheets/permissions

1. Create a [Google Service Account](https://developers.google.com/identity/protocols/OAuth2ServiceAccount#creatinganaccount) and download the credentials file.
1. Open your google sheet, click "File > Share..." and enter your service account's e-mail address (you can find it in the credentials file).


## Step 2: configure your gatsby project

Standard source plugin installation.

```
yarn add gatsby-source-google-sheets


// gatsby-config.js
// ...
{
    resolve: 'gatsby-source-google-sheets',
    options: {
        spreadsheetId: 'get this from the sheet url',
        worksheetTitle: 'ie the name in the worksheet tab',
        credentials: require('./path-to-credentials-file.json'),
        autoCast: true
    }
},
// ...

```

The plugin makes the following conversions before feeding Gatsby nodes, if `autoCast` is true:
1. Numbers are converted to numbers. Sheets formats numbers as comma-delineated strings, so to determine if something is a number, the plugin tests to see if the string (a) is non-empty and (b) is composed only of commas, decimals, and digits:
```
if (
    "value".replace(/[,\.\d]/g, "").length === 0 
      && "value" !== ""
   ) { 
    ...assume value is a number and handle accordingly
}
```
2. "TRUE"/"FALSE" converted to boolean true/false
3. empty cells ("" in sheets payload) converted to null
4. Column names are converted to camelcase via lodash _.camelCase() (see note 2 in 'A few notes')


A few notes:

1. Not tested with cells of data type dates.
2. Google sheets mangles column names and converts them all to lower case. This plugin will convert them to camelcase, so the best convention here is to name your columns all lowercase with dashes. e.g. instead of "Column Name 1" or "columnName1", prefer "column-name-1"--this last one will be turned into "columnName1" in your GatsbyQL graph. 

# Troubleshooting
3. If you get the error "No key or keyFile set", make sure you are using a Service Account API key and not a simple API key.
4. If you get the error "Cannot read property 'worksheets' of undefined", make sure you have shared your spreadsheet with your service account user.