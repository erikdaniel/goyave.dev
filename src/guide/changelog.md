---
meta:
  - name: "og:title"
    content: "Changelog - Goyave"
  - name: "twitter:title"
    content: "Changelog - Goyave"
  - name: "title"
    content: "Changelog - Goyave"
---

# Changelog

[[toc]]

## v4.4.0

- Added [`validation.ValidateWithExtra`](./basics/validation.html#validation-validatewithextra).
- The validation middleware now provides the `*goyave.Request` to validation rules as an extra.

## v4.3.0

- Added [post validation hooks](./basics/validation.html#post-validation-hooks).

## v4.2.x

### v4.2.1

- Fixed the test `TestClearDatabaseView` that was failing if run multiple times.
- Fixed a validation crash when a nullable field doesn't pass validation.

### v4.2.0

- Added [`database.IView`](./basics/database.html#views).
- Don't force the `GOYAVE_ENV` to be `test` in `TestSuite`. Only sets it to `test` if the `GOYAVE_ENV` environment variable is not already set.

## v4.1.x

### v4.1.1

- Fixed `database.Paginator` error on count query when using `Preload` and the latest version of gorm.

### v4.1.0

- Added `database.SetConnection()`.
- Added support for raw queries in `database.Paginator`.

## v4.0.0

- Only the last two major versions of Go will be supported, starting with this release.

**Motivation**: *Supporting only the last two minor versions of the language follows Google's policy and allows for development of new modern features. It lets the framework be updated without it being considered a breaking change, and encourages to not use outdated versions of the language while newer ones contain security fixes.*

- Validation changes
    - Added validation rules `before_now` and `after_now`.
    - Added validation `RuleSet` and `Rules` composition for re-usability of redundant validated objects.
    - Changed signature of `validation.RuleFunc` to `func(*validation.Context) bool`. `validation.Context` provides more information and a cleaner and more flexible API.
        - Note: Converting rules now modify `context.Value` instead of directly modifying the input data.
    - Improved n-dimensional array validation with a new syntax.
        - The chevron-based syntax disappears in favor of a new more idiomatic syntax based on the name of the field, for example `array[]` will validate all the elements of the `array` array.
        - This new syntax allows for the validation of objects inside arrays.
    - Improved tree-like validation error messages structure, identifying precisely each element based on the expected structure. This structure also identifies each array element precisely with its index.
    - Removed the `confirmed` validation rule. Use `same:path.to.field` instead.
    - If a field is missing and not required, child fields are now skipped. This prevents `required` rules to not pass if the parent is not required and missing.
    - The `required` rules now allows empty strings.
    - Use `*validation.Context` in validation message placeholders. The `validation.Placeholder` signatures becomes `func(fieldName string, language string, ctx *validation.Context) string`.
    - Added `Extra` (`map[string]interface{}`) to `*validation.Context`. This allows placeholders to use additional information given by the validator function.
    - Added `validation.StructList`. This acts as a better alternative syntax for validation. The previous alternative syntax (`*validation.Rules`) should not be used for validation rules definition anymore but still works. 

**Motivation**: *Although the validation package has seen some major improvements in v3, it was still far from perfect and was really lacking some important capabilities, such as the validation of objects in arrays. The array validation syntax was confusing, hard to read, and incompatible with the new syntax that was needed for object validation in arrays. We now have a more readable syntax, offering better capabilities. Because of how differently the validator works internally, we also had the opportunity to rework the error messages output, which now offers an impressive degree of precision. Precision in error messages gives a real edge over competing validation libraries.*

- Refactored  the `helper` package and split it into several focused packages.
    - `helper/filesystem` moved to `util/fsutil`
    - `helper/walk` moved to `util/walk`
    - New package `util/httputil`
        - Contains `ParseMultiValuesHeader()` and `HeaderValue`
    - New package `util/reflectutil`
        - Contains `Only()`
    - New package `util/sliceutil`
        - Contains `IndexOf()`, `IndexOfStr()`, `Contains()`, `ContainsStr()` and `Equal()` (previously named `SliceEqual()`)
    - New package `util/sqlutil`
        - Contains `EscapeLike()`
    - New package `util/typeutil`
        - Contains `Map`, `ToFloat64()`, `ToString()`

**Motivation**: *The `helper` packaged started to become more and more bloated with functions for things unrelated to each other. To make this part of the code more idiomatic and expressive, the decision to split the package in several parts and re-think the naming was taken.*

- Removed `model:"hide"` because it was redundant with `json:"-"`.
- Added `goyave.ProxyBaseURL()` and `server.proxy` configuration entries.
- Added global middleware. Global middleware are executed on all requests, including requests that don't match any route or that result in "Method Not Allowed". This allows for better logging, rate limiting, and more, while keeping the already existing tools.
- Added `helper/walk` package to navigate complex data structure of unknown type (`map[string]interface{}`). This is used by the validator.
- Migrated from `dgrijalva/jwt-go` library to the maintained [golang-jwt/jwt](https://github.com/golang-jwt/jwt).
- Added ability to set default language strings from code using `lang.SetDefaultLine()`, `lang.SetDefaultValidationRule()` and `lang.SetDefaultFieldName()`.
- JWT: now uses standard `sub` instead of `userid` by default.
- Reduced the amount of reflection in `auth.FindColums`.
- Added support for promoted pointer of struct in `auth.FindColumns` and `helper.Only`.
- `database.Paginator` improvements
    - Now exports its `DB` field so it can be re-assigned with new clauses.
    - Now exports its `UpdatePageInfo()` method so it can be called manually before the real query.
    - Now uses a Gorm session for page info query to prevent weird queries in some specific scenarios.
    - Fields names are now in camel case when json marshalling a `Paginator`. 
- Removed the `DisallowNonValidatedFields` middleware as it would be too expensive and complex to properly implement it with the new validation system.
- Removed the unused `name` parameter in `request.Cookie()`. *Thanks to [@agbaraka](https://github.com/agbaraka) for the contribution!*
- Fixed duplicate error message when a request accessor panicked.

## v3.11.0

- Exported `validation.Rules.Check()` and `validation.Field.Check()`.
- Added validation rule `exists`.
- Changed default invalid credentials "en-US" language entry to "Invalid credentials.".
- Added [`auth.Unauthorizer`](./advanced/authentication.html#auth-unauthorizer).

## v3.10.x

### v3.10.1

- Fixed native middleware don't replace http.Request and always used the initial one.
- Fixed CORS options not working on subrouters. The main router's ones were always used.
- Fixed route groups at root level don't match routes with `/` prefix.

### v3.10.0

- Added Gorm settings to configuration file. *Thanks to [Zao SOULA](https://github.com/zaosoula) for the contribution!*

## v3.9.x

### v3.9.1

- Fixed a crash when using the `nullable` rule.

### v3.9.0

- Fixed random inconsistencies in validation results when using rules comparing value to other fields.
- Added `RuleDefinition.ComparesFields` to specify a rule can compare the field under validation with another field. Custom rules that do so should update their definition to avoid previously mentioned inconsistencies.
- Fixed a bug causing the shutdown hook listening to `SIGINT` and `SIGTERM` to clear itself at server startup.
- Fixed a bug causing fields inside objects to not be converted by type rules and leaving junk into the input data.
- Handle dot-separated path for comparison validation rules.

## v3.8.0

- Added [`helper.Map`](./advanced/helpers.html#helper-map). *Thanks to [@agbaraka](https://github.com/agbaraka) for the contribution!*
- Added `router.Group`, which is an alias for `router.Subrouter("")`, aimed at improving route definition readability.
- Address generation outputs `127.0.0.1` if `0.0.0.0` is set as server host.
- Added validation `Rule.IsType()` and `Rule.IsTypeDependent()`.
- Added [`route.BuildURI()`](./basics/routing.html#route-builduri) to be able to generate a full URI without the host and protocol prefix.
- Added various accessors for routes and routers.
- Fixed timing attack on config basic authenticator.
- Exported `goyave.NewRouter()`.
- Minor memory improvements by re-aligning some structure fields.
- Added support for custom JWT claims. (`jwt.GenerateTokenWithClaims()`). *Thanks to [@Morishiri](https://github.com/Morishiri) for the contribution!*
- Added `JWTController.TokenFunc` to let developers define their own token generation logic. *Thanks to [@Morishiri](https://github.com/Morishiri) for the contribution!*
- Added support for RSA and ECDSA JWT signatures.
- Added `JWTAuthenticator.ClaimName`. This field defines what is the name if the claim used as an ID.
- `JWTAuthenticator` now adds token claims to `request.Extra` if the provided token is valid. (key: `"jwt_claims"`)
- `JWTController.Login` now returns `401 Unauthorized` instead of `422 Unprocessable Entity` if provided credentials are invalid.

## v3.7.x

### v3.7.1

- Don't parse body if `Content-Type` is not set. Query params are still parsed. This change should drastically improve performance on requests with no body.
- Set `request.Data` to `nil` (meaning parse failed) if query params couldn't be parsed.

### v3.7.0

- Added `Optional` flag to `BasicAuthenticator` and `JWTAuthenticator`.
- Added support for `database.options` for the SQLite driver.
- Added [`response.Hijack()`](./basics/responses.html#hijack). Therefore, `*goyave.Response` now implements `http.Hijacker`. Note that status handlers and middleware (requests finalization step in their lifecycle) will still work for hijacked connections.
- Added [websocket](./advanced/websocket.html) support.
- Added [shutdown hooks](./advanced/multi-services.html#shutdown-hooks).
- Added `goyave.BaseURL()`.
- Added [`config.LoadJSON()`](./configuration.html#config-loadjson). This can be used to load configuration from an embedded configuration file using Go's 1.16 embed directive.
- Static file serving will no longer print "no such file or directory" to the error logger.
- Static file serving optimization: check file existence once instead of twice.

## v3.6.0

- Set content type to `application/json; charset=utf-8` instead of `application/json` when using `response.JSON()`.
- Added default behavior for `HEAD` method: all `GET` routes can now match the HTTP `HEAD` method. This fixes `405 Method Not Allowed` when requesting an URL with the `HEAD` method when no route explicitly matches the `HEAD` method. See the [HEAD routing advice](./basics/routing.html#handling-head) for more details.
- Added [`request.ToStruct()`](./basics/requests.html#request-tostruct), which puts the request's data into the given structure.

## v3.5.0

- Added [rate limiter middleware](./advanced/rate-limiting.html)

## v3.4.0

- Type-dependent rules validating integers (via the "integer" type rule) now share their validation message with the "numeric" type.
- Added [paginators](./basics/database.html#pagination).
- Added [`helper.EscapeLike()`](./advanced/helpers.html#helper-escapelike).
- Performance improvement by caching critical config entries (`protocol`, `maxUploadSize` and `defaultLanguage`). This change leads to about 18% more requests per second. However, these entries cannot be dynamically changed anymore: a server restart will be needed.

## v3.3.x

### v3.3.1

- Fixed a bug in the validatior: the original value of type-converted fields was always used, leading to wrong validation of subsequent type-dependent rules.

### v3.3.0

- Added `request.Extra`. Thank you [Guillermo Galvan](https://github.com/gmgalvan) for your contribution!
- `TestSuite` now runs auto migrations if they're enabled before running the tests.
- `TestSuite` don't load config anymore if it's already loaded. This allows you to load a test configuration file using `LoadFrom()` before calling `goyave.RunTest()`.
- `response.JSON()` doesn't remove hidden fields anymore. The use of `json:"-"` makes more sense and saves some execution time. Removing hidden fields manually is still possible. See the [hidden fields documentation](./basics/database.html#hidden-fields) for more details.

## v3.2.0

- Added a way to customize the request's body field used by `JWTController` for the authentication process. (By default, "username" and "password" are used)
- Added a new validation rule: `unique`.
- Added `helper.Only()`.
- `filesystem.File.Save()` now creates directories if needed.

## v3.1.0

- Added support for validating objects (`map[string]interface{}`).
- Added `request.Object()` accessor.

## v3.0.x

### v3.0.1

- Fixed a bug that prevented root-level routes with an empty path (`/`) to be matched.

### v3.0.0

- Changed conventions:
    - `validation.go` and `placeholders.go` moved to a new `http/validation` package.
    - Validation rule sets are now located in a `request.go` file in the same package as the controller.
    
**Motivation**: *Separating the requests in another package added unnecessary complexity to the directory structure and was not convenient to use. Package naming was far from ideal with the "request" suffix. Moving requests to the same package as the controller is more intuitive and requires less imports and makes route definition cleaner and easier.*

- Validation system overhaul, allowing rule sets to be parsed only once instead of every time a request is received, giving better overall performance. This new system also allows a more verbose syntax for validation, solving the comma rule parameter value and a much easier use in your handlers.
    - Rule functions don't check required parameters anymore. This is now done when the rules are parsed at startup time. The amount of required parameters is given when registering a new rule.
    - Optimized regex-based validation rules by compiling expressions once.
    - A significant amount of untested cases are now tested.
    - The following rules now pass if the validated data type is not supported: `greater_than`, `greater_than_equal`, `lower_than`, `lower_than_equal`, `size`.
    - Type-dependent rules now try to determine what is the expected type by looking up in the rule set for a type rule. If no type rule is present, falls back to the inputted type. This change makes it so the validation message is correct even if the client didn't input the expected type.
    - Fixed a bug triggering a panic if the client inputted a non-array value in an array-validated field.

**Motivation**: *The validation system had a lot of room for improvement when it comes to performance, as `RuleSet` were parsed every time a request was received. Moving this process out of the request life-cycle to execute it only once saves a good amount of execution time. Moreover, any handler who would want to read the rules applied to the current request needed to parse them too, which was inconvenient and not effective. With a structure containing everything you need, making middleware interacting with the request's rules is much easier.*

- Routing has been improved by changing how validation and route-specific middleware are registered. The signature of the router functions have been simplified by removing the validation and middleware parameters from `Route()`, `Get()`, `Post()`, etc. This is now done through two new chainable methods on the `Route`: `route.Validate()` and  `route.Middleware()`.

**Motivation**: *In the original design, the validation parameter was included in the main route definition function because most routes were expected to be validated, which turned out not to be the case. In a typical CRUD, only the create and update actions are validated, which made the route definition dirty and filled with `nil` parameters. Separating the rules and middleware definition is more in line with their optional nature and makes routes definition cleaner and more readable, although sometimes slightly longer.*

- Log `Formatter` now receive the length of the response (in bytes) instead of the full body.

**Motivation:** *Keeping in memory the full response has an important impact on memory when sending files or large responses. Using the response content in a log formatter is also a marginal use-case which doesn't justify the performance loss described previously. It is still possible to retrieve the content of the response by writing your own chained writer.*

- Configuration system has been revamped.
    - Added support for tree-like configurations, allowing for better categorization. Nested values can be accessed using dot-separated path.
    - Improved validation: nested entries can now be validated too and all entries can have authorized values. Optional entries can now be validated too.
    - Improved support for slices. The validation system is also able to check slices.
    - Entries that are validated with the `int` type are now automatically converted from `float64` if they don't have decimal places. It is no longer necessary to manually cast `float64` that are supposed to be integers.
    - More openness: entries can be registered with a default value, their type and authorized values from any package. This allows config entries required by a specific package to be loaded only if the latter is imported.
    - Core configuration has been sorted in categories. This is a breaking change that will require you to update your configuration files.
    - Entries having a `nil` value are now considered unset.
    - Added accessors `GetInt()` and `GetFloat()`.
    - Added slice accessors: `GetStringSlice()`, `GetBoolSlice()`, `GetIntSlice()`, `GetFloatSlice()`
    - Added `LoadFrom()`, letting you load a configuration file from a custom path.
    - Added the ability to use environment variables in configuration files.
    - Bug fix: `config.IsLoaded()` returned `true` even if config failed to load.
    - `maxUploadSize` config entry now supports decimal places.

**Motivation:** *Configuration was without a doubt one of the weakest and inflexible feature of the framework. It was possible to use objects in custom entries, but not for core config, but it was inconvenient because it required a lot of type assertions. Moreover, core config entries were not handled the same as custom ones, which was a lack of openness. Hopefully, this revamped system will cover more potential use-cases, ease plugin development and allow you to produce cleaner code and configuration files.*

- Database improvements
    - Goyave has moved to [GORM v2](https://gorm.io/). Read the [release note](https://gorm.io/docs/v2_release_note.html) to learn more about what changed.
    - Protect the database instance with mutex.
    - `database.Close()` can now return errors.
    - Added [database connection initializers](./basics/database.html#connection-initializers).
    - Added the ability to regsiter new SQL dialects to use with GORM.
    - Use `utf8mb4` by default in database options.
    - Added a short alias for `database.GetConnection()`: `database.Conn()`.
    - Factories now use batch insert.
    - Factories now return `interface{}` instead of `[]interface{}`. The actual type of the returned value is a slice of the the type of what is returned by your generator, so you can type-assert safely.
- Status handlers improvements
    - Export panic and error status handlers so they can be expanded easily.
    - Added `goyave.ValidationStatusHandler()`, a status handler for validation errors. Therefore, the format in which validation errors are sent to the client can be customized by using your own status handler for the HTTP status 400 and 422.
- `goyave.Response` improvements
    - `response.Render` and `response.RenderHTML` now execute and write the template to a `bytes.Buffer` instead of directly to the `goyave.Response`. This allows to catch and handle errors before the response header has been written, in order to return an error 500 if the template doesn't execute properly for example.
    - Added `response.GetStacktrace()`, `response.IsEmpty()` and `response.IsHeaderWritten()`.
    - Re-organised the `goyave.Response` structure fields to save some memory.
    - Removed deprecated method `goyave.CreateTestResponse()`. Use `goyave.TestSuite.CreateTestResponse()` instead.
- Recovery middleware now correctly handles panics with a `nil` value.
- Test can now be run without the `-p 1` flag thanks to a lock added to the `goyave.RunTest` method. Therefore, `goyave.TestSuite` still **don't run in parallel** but are safe to use with the typical test command.
- Cache the regex used by `helper.ParseMultiValuesHeader()` to improve performance. This also improves the performance of the language middleware.
- Bug fix: data under validation wasn't considered from JSON payload if the content type included the charset.
- The Gzip middleware will now skip requests that have the `Upgrade` HTTP header set to any value.
- `response.String()` and `response.JSON()` don't write header before calling `Write` anymore. This behavior prevented middleware and chained writers to alter the response headers.
- Added `goyave.PreWriter` interface for chained writers needing to alter headers or status before they are written.
    - Even if this change is not breaking, it is recommended to update all your chained writers to call `PreWrite()` on their child writer if they implement the interface.
    - Thanks to this change, a bug with the gzip middleware has been fixed: header `Content-Length` wasn't removed, resulting in false information sent to the clients, which in turn failed to decompress the response.


## v2.x.x

### v2.10.x

#### v2.10.2

- Fixed a bug in body parsing middleware preventing json body to be parsed if a charset was provided.

#### v2.10.1

- Changed the behavior of `response.File()` and `response.Download()` to respond with a status 404 if the given file doesn't exist instead of panicking.
- Improved error handling:
    - `log.Panicf` is not used anymore to print panics, removing possible duplicate logs.
    - Added error checks during automatic migrations.
    - `goyave.Start()` now exits the program with the following error codes:
        - `2`: Panic (server already running, error when loading language files, etc)
        - `3`: Configuration is invalid
        - `4`: An error occurred when opening network listener
        - `5`: An error occurred in the HTTP server

This change will require a slightly longer `main` function but offers better flexibility for error handling and multi-services.

``` go
if err := goyave.Start(route.Register); err != nil {
	os.Exit(err.(*goyave.Error).ExitCode)
}
```

- Fixed a bug in `TestSuite`: HTTP client was re-created everytime `getHTTPClient()` was called.
- Fixed testing documentation examples that didn't close http response body.
- Documentation meta improvements.
- Protect JSON requests with `maxUploadSize`. 
- The server will now automatically return `413 Payload Too Large` if the request's size exceeds the `maxUploadSize` defined in configuration.
- The request parsing middleware doesn't drain the body anymore, improving native handler compatibility.
- Set a default status handler for all 400 errors.
- Fixed a bug preventing query parameters to be parsed when the request had the `Content-Type: application/json` header.
- Added a dark theme for the documentation. It can be toggled by clicking the moon icon next to the search bar.

#### v2.10.0

- Added router `Get`, `Post`, `Put`, `Patch`, `Delete` and `Options` methods to register routes directly without having to specify a method string.
- Added [placeholder](./advanced/localization.html#placeholders) support in regular language lines.

### v2.9.0

- Added [hidden fields](./basics/database.html#hidden-fields).
- Entirely removed Gorilla mux. This change is not breaking: native middleware still work the same.

### v2.8.0

- Added a built-in logging system.
    - Added a middleware for access logs using the Common Log Format or Combined Log Format. This allows custom formatters too.
    - Added three standard loggers: `goyave.Logger`, `goyave.AccessLogger` and `goyave.ErrLogger`
- Fixed bug: the gzip middleware now closes underlying writer on close.

### v2.7.x

#### v2.7.1

- Changed MIME type of `js` and `mjs` files to `text/javascript`. This is in accordance with an [IETF draft](https://datatracker.ietf.org/doc/draft-ietf-dispatch-javascript-mjs/) that treats application/javascript as obsolete.
- Improved error handling: stacktrace wasn't relevant on unexpected panic since it was retrieved from the route's request handler therefore not including the real source of the panic. Stacktrace retrieval has been moved to the recovery middleware to fix this.

#### v2.7.0

- Added `Request.Request()` accessor to get the raw `*http.Request`.
- Fixed a bug allowing non-core middleware applied to the root router to be executed when the "Not Found" or "Method Not Allowed" routes were matched.
- Fixed a bug making route groups (sub-routers with empty prefix) conflict with their parent router when two routes having the same path but different methods are registered in both routers.
- Added [chained writers](./basics/responses.html#chained-writers).
- Added [gzip compression middleware](./basics/middleware.html#gzip).
- Added the ability to register route-specific middleware in `Router.Static()`.

### v2.6.0

- Custom router implementation. Goyave is not using gorilla/mux anymore. The new router is twice as fast and uses about 3 times less memory.
- Now redirects to configured protocol if request scheme doesn't match.
- Added [named routes](./basics/routing.html#named-routes).
- Added `Route.GetFullURI()` and `Route.BuildURL()` for dynamic URL generation.
- Added `helper.IndexOfStr()` and `helper.ContainsStr()` for better performance when using string slices.
- Moved from GoDoc to [pkg.go.dev](https://pkg.go.dev/github.com/System-Glitch/goyave/v2).
- Print errors to stderr.

### v2.5.0

- Added an [authentication system](./advanced/authentication.html).
- Various optimizations.
- Various documentation improvements.
- Added `dbMaxLifetime` configuration entry.
- Moved from Travis CI to Github Actions.
- Fixed a bug making duplicate log entries on error.
- Fixed a bug preventing language lines containing a dot to be retrieved.
- Fixed `TestSuite.GetJSONBody()` not working with structs and slices.
- Added `TestSuite.ClearDatabaseTables()`.
- Added `Config.Has()` and `Config.Register()` to check for the existence of a config entry and to allow custom config entries valdiation.
- Added `Request.BearerToken()`.
- Added `Response.HandleDatabaseError()` for easier database error handling and shorter controller handlers. 

### v2.4.x

#### v2.4.3

- Improved string validation by taking grapheme clusters into consideration when calculating length.
- `lang.LoadDefault` now correctly creates a fresh language map and clones the default `en-US` language. This avoids the default language entries to be overridden permanently.  

#### v2.4.2

- Don't override `Content-Type` header when sending a file if already set.
- Fixed a bug with validation message placeholder `:values`, which was mistakenly using the `:value` placeholder.

#### v2.4.1

- Bundle default config and language in executable to avoid needing to deploy `$GOROOT/pkg/mod/github.com/!system-!glitch/goyave/` with the application.

#### v2.4.0

- Added [template rendring](./basics/responses.html#response-render).
- Fixed PostgreSQL options not working.
- `TestSuite.Middleware()` now has a more realistic behavior: the finalization step of the request life-cycle is now also executed. This may require your tests to be updated if those check the status code in the response.
- Added [status handlers](./advanced/status-handlers.html).

### v2.3.0

- Added [CORS options](./advanced/cors.html).

### v2.2.x

#### v2.2.1

- Added `domain` config entry. This entry is used for url generation, especially for the TLS redirect.
- Don't show port in TLS redirect response if ports are standard (80 for HTTP, 443 for HTTPS).

#### v2.2.0

- Added [testing API](./advanced/testing.html).
- Fixed links in documentation.
- Fixed `models` package in template project. (Changed to `model`)
- Added [`database.ClearRegisteredModels`](./basics/database.html#database-clearregisteredmodels)

### v2.1.0

- `filesystem.GetMIMEType` now detects `css`, `js`, `json` and `jsonld` files based on their extension.
- Added maintenance mode.
    - Can be [toggled at runtime](./advanced/multi-services.html#maintenance-mode).
    - The server can be started in maintenance mode using the `maintenance` config option. (Defaults to `false`)
- Added [advanced array validation](./basics/validation.html#validating-arrays), with support for n-dimensional arrays.
- Malformed request messages can now be localized. (`malformed-request` and `malformed-json` entries in `locale.json`)
- Modified the validator to allow [manual validation](./basics/validation.html#manual-validation).

### v2.0.0

- Documentation and README improvements.
- In the configuration:
    - The default value of `dbConnection` has been changed to `none`.
    - The default value of `dbAutoMigrate` has been changed to `false`.
- Added [request data accessors](./basics/requests.html#accessors).
- Some refactoring and package renaming have been done to better respect the Go conventions.
    - The `helpers` package have been renamed to `helper`
- The server now shuts down when it encounters an error during startup.
- New [`validation.GetFieldType`](./basics/validation.html#validation-getfieldtype) function.
- Config and Lang are now protected with a `sync.RWMutex` to avoid data races in multi-threaded environments.
- Greatly improve concurrency.
- Config can now be reloaded manually.
- Added the [`Trim`](./basics/middleware.html#trim) middleware.
- `goyave.Response` now implements `http.ResponseWriter`.
    - All writing functions can now return an error.
- Added the [`NativeHandler`](./basics/routing.html#native-handlers) compatibility layer.
- Fixed a bug preventing the static resources handler to find `index.html` if a directory with a depth of one was requested without a trailing slash.
- Now panics when calling `Start()` while the server is already running.
