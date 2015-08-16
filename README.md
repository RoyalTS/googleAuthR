# googleAuthR - Google API Client Library for R
[![Travis-CI Build Status](https://travis-ci.org/MarkEdmondson1234/googleAuthR.svg?branch=master)](https://travis-ci.org/MarkEdmondson1234/googleAuthR)

Build libraries for Google APIs with OAuth2 for both local and Shiny app use.

Here is a list of [available Google APIs.](https://developers.google.com/apis-explorer/#p/)

## Install

```
## load the library or download it if necessary
if(!require(googleAuthR)){
  if(!require(devtools)){
    install.packages("devtools")
  } else {
    devtools::install_github("MarkEdmondson1234/googleAuthR")
  }
}
library(googleAuthR)
```

## Overview

The main two functions are `gar_auth()` and `gar_api_generator()`.

### `gar_auth`
This takes care of getting the authentication token, storing it and refreshing. 
Use it before any call to a Google library.

### `gar_api_generator`
This creates functions for you to use to interact with Google APIs.
Use it within your own function definitions, to query the Google API you want.

## Google API Setup

`googleAuthR` has a default project setup with APIs activated for several APIs, but it is recommended you use your own Client IDs as the login screen will be big and scary for users with so many APIs to approve.  

It is preferred to configure your functions to only use the scopes they need.

Set scopes via the option `googleAuthR.scopes.selected`.



The below example sets scopes for Search Console, Google Analytics and Tag Manager:
```
options("googleAuthR.scopes.selected" = c("https://www.googleapis.com/auth/webmasters",
                                          "https://www.googleapis.com/auth/analytics",
                                          "https://www.googleapis.com/auth/tagmanager.readonly"))
```

A non-definitive scope list to choose from will be attempted to be maintained via `getOption("googleAuthR.scopes")`

### Set up steps
1. Set up your project in the [Google API Console](https://code.google.com/apis/console) to use the search console v3 API.

#### For local use
2. Click 'Create a new Client ID', and choose "Installed Application".
3. Note your Client ID and secret.
4. Modify these options after `googleAuthR` has been loaded:
  + `options("googleAuthR.client_id" = "YOUR_CLIENT_ID")`
  + `options("googleAuthR.client_secret" = "YOUR_CLIENT_SECRET")`

#### For Shiny use
2. Click 'Create a new Client ID', and choose "Web Application".
3. Note your Client ID and secret.
4. Add the URL of where your Shiny app will run, as well as your local host for testing including a port number.  e.g. https://mark.shinyapps.io/searchConsoleRDemo/ and http://127.0.0.1:4624
5. In your Shiny script modify these options:
  + `options("googleAuthR.webapp.client_id" = "YOUR_CLIENT_ID")`
  + `options("googleAuthR.webapp.client_secret" = "YOUR_CLIENT_SECRET")`
6. Run the app locally specifying the port number you used e.g. `shiny::runApp(port=4624)`
7. Or deploy to your Shiny Server that deploys to web port (80 or 443).

#### Activate API

1. Click on "APIs"
2. Select and activate the API you want to use.
3. Go to the documentation and find the API scope URL
4. Set option in your R script for the scope e.g. 

```
options("googleAuthR.scopes.selected" = 
      c("https://www.googleapis.com/auth/urlshortener"))
```

## Building your own functions

If the above is successful, then you should go through the Google login flow in your browser when you run this command:

```
googleAuthR::gar_auth()
```

If you ever need to authenticate with a new user, use:

```
googleAuthR::gar_auth(new_user=TRUE)
```

Authentication token is cached in a hidden file called `.httr-oauth` in the working directory.

### Authentication with no browser

If for some reason you need authentication without access to a browser (for example when using Shiny Server), then you can authenticate locally and upload the `.httr-oauth` file to the folder of your script.

### Authentication with Shiny

If you want to create a Shiny app just using your data, upload the app with your own `.httr-oauth`.

If you want to make a multi-user Shiny app, where users login to their own Google account and the app works with their data, read on:

googleAuthR provides these functions to help make the Google login process as easy as possible:

* `loginOutput()` - creates the client side login button for users to authenticate with.
* `renderLogin()` - creates the server side login button for users to authenticate with.
* `reactiveAccessToken()` - creates the user's authentication token.
* `with_shiny()` - wraps your API functions so they can be passed the user's authentication token.

#### Shiny authentication example

```
## in server.R
shinyServer(function(input, output, session)){

  ## Get auth code from return URL
  access_token  <- reactiveAccessToken(session)

  ## Make a loginButton to display using loginOutput
  output$loginButton <- renderLogin(session, access_token())

  output$websites <- renderTable({

    ## using with_shiny adds a shiny_access_token parameter to pass access_token()
    list_websites <- with_shiny(list_websites,
                                access_token())

    })

}

## in ui.R
shinyUI(fluidPage(
  loginOutput("loginButton"),
  renderTable("websites)
  ))

```

## Generating your function

Creating your own API should then be a matter of consulting the Google API documentation, and filling in the required details.

`gar_api_generator` has these components:

* `baseURI` - all APIs have a base for every API call
* `http_header` - what type of request, most common are GET and POST
* `path_args` - some APIs need you to alter the URL folder structure when calling, e.g. `/account/{accountId}/` where `accountId` is variable.
* `pars_args` - other APIS require you to send URL parameters e.g. `?account={accountId}` where `accountId` is variable.
* `data_parse_function` - [optional] If the API call returns data, it will be available in `$content`. You can create a parsing function that transforms it in to something you can work with (for instance, a dataframe)

Example below for generating a function:
```
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                             "POST",
                             data_parse_function = function(x) x$id)

```

## Using your generated function

The function generated uses `path_args` and `pars_args` to create a template, but when the function is called you will want to pass dynamic data to them.  This is done via the `path_arguments` and `pars_arguments` parameters.

`path_args` and `pars_args` and `path_arguments` and `pars_arguments` all accept named lists.

If a name in `path_args` is present in `path_arguments`, then it is substituted in.  This way you can pass dynamic parameters to the constructed function.  Likewise for `pars_args` and `pars_arguments`.

```
## Create a function that requires a path argument /accounts/{accountId}
  f <- gar_api_generator("https://www.googleapis.com/example",
                             "POST",
                             path_args = list(accounts = "defaultAccountId")
                             data_parse_function = function(x) x$id)
                             
## When using f(), pass the path_arguments function to it 
## with the same name to modify "defaultAccountId":

  result <- f(path_arguments = list(accounts = "myAccountId"))


```

### Body data

A lot of Google APIs look for you to send data in the Body of the request.  This is done after you construct the function.

 `googleAuthR` uses `httr`'s JSON parsing via `jsonlite` to construct JSON from R lists.
 
 Construct your list, then use `jsonlite::toJSON` to check if its in the correct format as specified by the Google documentation.  This is often the hardest part using the API.
 
### Parsing data

Not all API calls return data, but if they do:

If you have no `data_parse_function` then the function returns the whole request object.  The content is available in `$content`.  You can then parse this yourself, or pass a function in to do it for you.

If you parse in a function into `data_parse_function`, it works on the response's `$content`.

Example below of the differences between having a data parsing function and not:

```
  ## the body object that will be passed in
  body = list(
    longUrl = "http://www.google.com"
  )
  
  ## no data parsing function
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                             "POST")
  no_parse <- f(the_body = body)
  
  ## parsed data, only taking request$content$id
  f2 <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                             "POST",
                             data_parse_function = function(x) x$id)
  
  parsed <- f2(the_body = body)
  
  ## str(no_parse) has full details of API response.
  ## just looking at no_parse$content as this is what API returns
  > str(no_parse$content)
  List of 3
   $ kind   : chr "urlshortener#url"
   $ id     : chr "http://goo.gl/ZwT9pG"
   $ longUrl: chr "http://www.google.com/"
 
  ## compare to the above - equivalent to no_parse$content$id 
  > str(parsed)
   chr "http://goo.gl/mCYw2i"
                             
```
The response is turned from JSON to a dataframe if possible, via `jsonlite::fromJSON`

### Putting it together

Below is an example for a link shortner API call to goo.gl:

```
#' Shortens a url using goo.gl
#'
#' @param url URl to shorten with goo.gl
#' 
#' @return a string of the short URL
#'
#' Documentation: https://developers.google.com/url-shortener/v1/getting_started

## a wrapper for the function that users pass in the URL to shorten
shorten_url <- function(url){
  
  ## turns into {'longUrl' : '<<example.com>>'} when using jsonlite::toJSON(body)
  body = list(
    longUrl = url
  )
  
  ## generate the API call function
  ## POST https://www.googleapis.com/urlshortener/v1/url
  ## response has 4 objects $kind, $id, $longUrl, and $status, but we only want $id
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                             "POST",
                             data_parse_function = function(x) x$id)
                             
                             
  ## now the function has been generated, pass in the body.
  ## this function has no need for path_arguments or pars_arguments, but that will differ for other APIs.
  f(the_body = body)
  
}

## to use:

gar_auth()
shorten_url("http://www.google.com")

```

### Using with Shiny

If you want to create a Shiny app just using your data, upload the app with your own `.httr-oauth`.

If you want to make a multi-user Shiny app, where users login to their own Google account and the app works with their data, googleAuthR provides these functions to help make the Google login process as easy as possible:

* `loginOutput()` - creates the client side login button for users to authenticate with.
* `renderLogin()` - creates the server side login button for users to authenticate with.
* `reactiveAccessToken()` - creates the user's authentication token.
* `with_shiny()` - wraps your API functions so they can be passed the user's authentication token.

#### Shiny authentication example

```
## in server.R
shinyServer(function(input, output, session)){

  ## Get auth code from return URL
  access_token  <- reactiveAccessToken(session)

  ## Make a loginButton to display using loginOutput
  output$loginButton <- renderLogin(session, access_token())

  output$websites <- renderTable({

    ## using with_shiny adds a shiny_access_token parameter to pass access_token()
    list_websites <- with_shiny(list_websites,
                                access_token())

    })

}

## in ui.R
shinyUI(fluidPage(
  loginOutput("loginButton"),
  renderTable("websites)
  ))

```
 
### More info

See more at `?gar_api_generator` once the documentation has caught up.

## Example with goo.gl

Below is an example building a link shortner R package using `googleAuthR`.

It was done referring to the [documentation for Google URL shortener](https://developers.google.com/url-shortener/v1/getting_started).

Note the help docs specifies the steps outlined above. These are in general the steps for every Google API.

1. Creating a project
2. Activate API
3. Provide scope
4. Specify the base URL (in this case `https://www.googleapis.com/urlshortener/v1/url`)
5. Specify the httr request type e.g. `POST`
6. Constructing a body request
7. Giving the response format

### Example goo.gl R library
```
library(googleAuthR)

## change the native googleAuthR scopes to the one needed.
options("googleAuthR.scopes.selected" = 
        c("https://www.googleapis.com/auth/urlshortener"))

#' Shortens a url using goo.gl
#'
#' @param url URl to shorten with goo.gl
#' 
#' @return a string of the short URL
shorten_url <- function(url){
  
  body = list(
    longUrl = url
  )
  
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                             "POST",
                             data_parse_function = function(x) x$id)
  
  f(the_body = body)
  
}

#' Expands a url that has used goo.gl
#'
#' @param shortUrl Url that was shortened with goo.gl
#' 
#' @return a string of the expanded URL
expand_url <- function(shortUrl){
  
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                                  "GET",
                                  pars_args = list(shortUrl = "shortUrl"),
                                  data_parse_function = function(x) x)
  f(pars_arguments = list(shortUrl = shortUrl))
  
}

#' Get analyitcs of a url that has used goo.gl
#'
#' @param shortUrl Url that was shortened with goo.gl
#' @param timespan The time period for the analytics data
#' 
#' @return a dataframe of the goo.gl Url analytics
analytics_url <- function(shortUrl, 
                          timespan = c("allTime", "month", "week","day","twoHours")){
  
  timespan <- match.arg(timespan)
    
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url",
                                  "GET",
                                  pars_args = list(shortUrl = "shortUrl",
                                                   projection = "FULL"),
                                  data_parse_function = function(x) { 
                                    a <- x$analytics 
                                    return(a[timespan][[1]])
                                    })
  
  f(pars_arguments = list(shortUrl = shortUrl))
}

#' Get the history of the authenticated user
#' 
#' @return a dataframe of the goo.gl user's history
user_history <- function(){
  f <- gar_api_generator("https://www.googleapis.com/urlshortener/v1/url/history",
                                  "GET",
                                  data_parse_function = function(x) x$items)
  
  f()
}
```

To use the above functions:

```
library(googleAuthR)

# got through authentication flow
gar_auth()

s <- shorten_url("http://markedmondson.me")
s

expand_url(s)

analytics_url(s, timespan = "month")

user_history()
```
