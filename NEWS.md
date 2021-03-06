# News Updates

# v0.4.0.9000

* tbd

# v0.4.0

* Add client based authentication in JavaScript plus example app
* Add check to `gar_auth_service` to see if you have downloaded right JSON file
* Discovery API functions to get details on Google APIs added: `gar_discovery_apis_list` and `gar_discovery_api`
* Add `gar_create_package` that takes `gar_discovery_api` JSON and creates R package
* Change warnings() in batch to myMessage() level 2
* ensure batch requests only occur per second to help calculation of QPS limits
* Add 404 message if batch requests are not found.
* Fixed halt error if message can't parse body JSON, will now fail gracefully but carry on
* allow overwriting of default httr "encode" again (#28)
* Headers will contain up to date version number of package
* Add `gar_auto_auth` and `gar_attach_auto_auth` for auto-authentication upon a package load
* Fix bug where you couldn't pass in the file location of the ".httr-oauth" location to `gar_auth()`
* `gar_auth` now raises errors not NULL for passing incorrect token file locations of tokens
* `gar_auth` respects renamed `.httr-oauth` tokens now via `getOption("googleAuthR.httr_oauth_cache")`
* Add link to Github repo with auto-generated packages: `https://github.com/MarkEdmondson1234/autoGoogleAPI`

# v0.3.1

* Add link to [example shiny app](https://mark.shinyapps.io/googleAuthRexample/)
* Add `option(googleAuthR.rawResponse)` - skip API checks on response - should now work
* A successfull request is now classed as all response codes matching ^20 e.g. 201, 204 etc.

# v0.3.0 

* Document default options in `?googleAuthR`
* Add `option(googleAuthR.rawResponse)` - skip API checks on response.
* Add an example Shiny app in `/inst/shiny/shiny-example.R`
* Add an [RStudio Addin](https://rstudio.github.io/rstudioaddins/) for easy authentication.  Run via menu or `googleAuthR:::gar_gadget()`
* Move simplifyVector option to be able to be passed in generated function, defaults to `getOption("googleAuthR.jsonlite.simplifyVector")`
* Remove scopes option as not used. 
* Added `googleAuthR.verbose` to control feedback. 0 = everything, 1 = debug, 2=normal, 3=important
* Make the retry kick in more often for every 5** and 429 status error
* Support non-JSON uploads (#28)
* Add option to force user consent screen on Shiny login
* Move specification of scope for `gar_auth_service` to param for more flexibility
* Migrated shiny functions to Shiny Modules (#27)

# v0.2 

* Added ability to add your own custom headers to requests via `customConfig` in `gar_api_generator`
* Add 'localhost' to shiny URL detection. 
* Google Service accounts now supported.  Authenticate via "Service Account Key" JSON.
* Exposed `gar_shiny_getUrl` and the authentication type (online/offline) in `renderLogin`
* `renderLogin` : logout now has option `revoke` to revoke authentication token
* Added option for `googleAuthR.jsonlite.simplifyVector` for content parsing for compatibility for some APIs
* Batch Google API requests now implemented.  See readme or `?gar_batch` and `?gar_batch_walk` for details.
* If data parsing fails, return the raw content so you can test and modify your data parsing function 
* Missed Jenny credit now corrected
* Add tip about using `!is.null(access_token())` to detect login state
* Add HTTP backoff for certain errors (#6) from Johann
* Remove possible NULL entries from path and pars argument lists
* Reduced some unnecessary message feedback
* moved `with_shiny` environment lookup to within generated function
* added gzip to headers

# v0.1 - CRAN

* Shiny compatibility
* Local authentication compatibility
