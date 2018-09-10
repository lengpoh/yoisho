# Yoisho Banking Corporation

Webservices and REST APIs that expose bank related data services with dynamic content, you can use them for testing and demos. Run them in containers on your local environment or in any cloud that gives you Docker.

<img src="https://raw.githubusercontent.com/u1i/yoisho/master/resources/yoisho-logo.png" width="250"/>

Available APIs and Webservices:

* Bank Assets - SOAP/XML, 2 methods
* Currency Exchange Rates - REST/JSON, 1 parameter
* ATM Locator - Full CRUDL REST interface (Create, Read, Update, Delete, List)
* Stock Quote - REST/JSON, 0 parameters, cached output.
* Fixed Deposit Calculator - REST/JSON, 2 parameters, semantic error handling, complex output
* Account Balance - OAuth (3-legged)

Looking for a quick way to deploy? [Docker Compose](docker-compose.yml) | [Chef Cookbook](chef-cookbook)

# Bank Assets - SOAP/XML

A webservice that gives you total assets and debt of the bank. Each request will produce a slightly different result - it's a busy bank so cash is flowing in & out constantly!

### Run the container (choosing port 8080, feel free to modify):

`docker run -d -p 8080:80 u1ih/yoisho-assets`

### Get the WSDL:

`curl http://localhost:8080/?WSDL`

<img src="https://raw.githubusercontent.com/u1i/yoisho/master/resources/wsdl.png" width="650"/>

### Import & run the request in SoapUI

<img src="https://raw.githubusercontent.com/u1i/yoisho/master/resources/soap.png" width="650"/>


# Currency Exchange Rates - REST/JSON

This API gives you exchange rates for currencies (USD, GBP and SGD) that the bank buys and sells. Each time you ask for a quote the amounts might be slighly different - they're really busy adjusting the rates constantly!

### Run the container (choosing port 8080, feel free to modify):

`docker run -d -p 8080:8080 u1ih/yoisho-currency`

### Get the Swagger:

`curl http://localhost:8080/swagger`

> {"info": {"version": "1.0", "description": "", "title": "Yoisho Currency Exchange"}, "paths": {"/get_currency": {"get": {"responses": {"default": {"description": "successful operation"}}, "produces": ["application/json"], "description": "", "parameters": [{"required": true, "type": "string", "description": "The desired currency", "name": "currency", "in": "query"}], "operationId": "get_currency"}}}, "schemes": ["http"], "basePath": "/currency", "host": "", "x-axway": {"deprecated": false, "serviceType": "rest", "basePaths": [""], "corsEnabled": true, "tags": {}}, "swagger": "2.0"}

### Get exchange rates - USD (also supported: GBP, SGD)

`curl http://localhost:8080/get_currency?currency=USD`

> {"sell": "489.185", "timestamp": "2017-09-17 02:58:40.194337", "buy": "389.105"}

# ATM Locator - REST/JSON

This API gives you a full CRUDL interface (Create, Read, Update, Delete, List) for ATM locations. In memory database.

### Run the container (choosing port 8080, feel free to modify):

`docker run -d -p 8080:8080 u1ih/yoisho-atm`

### Get the Swagger:

`curl http://localhost:8080/swagger`

> { "swagger": "2.0", "info": { "version": "", "title": "ATM Locations", "description": "List of ATM locations for Yoisho Banking Corporation" }, "basePath": "/api", "consumes": [ "application/json" ], "produces": [ "application/json" ], "paths": { "/atm/{id}": { "parameters": [ { "name": "id", "in": "path", "required": true, "type": "string" } ], "get": { "operationId": "GET-atm-location", "summary": "Get ATM Location", "tags": [ "Atm locations" ], "responses": { "200": { "description": "", "schema": { "$ref": "#/definitions/api-location-input" } } } }, "put": { "operationId": "PUT-atm-location", "summary": "Update ATM Location", "tags": [ "Atm locations" ], "parameters": [ { "name": "body", "in": "body", "schema": { "$ref": "#/definitions/api-location-input" } } ], "responses": { "200": { "description": "", "schema": { "$ref": "#/definitions/api-location-input" } } } }, "delete": { "operationId": "DELETE-atm-location", "summary": "Delete ATM Location", "tags": [ "Atm locations" ], "responses": { "204": { "description": "" } } } }, "/atm": { "get": { "operationId": "LIST-atm-locations", "summary": "List Atm locations", "tags": [ "Atm locations" ], "responses": { "200": { "description": "", "schema": { "type": "object", "properties": { "result": { "type": "array", "items": { "type": "object", "properties": { "lat": { "type": "string" }, "lon": { "type": "string" }, "location": { "type": "string" }, "id": { "type": "string" } } } } } }, "examples": { "application/json": { "data": [ { "lat": "35.6684231", "lon": "139.6833085", "location": "Ebisu Station" }, { "lat": "35.6284713", "lon": "139.736571", "location": "Shinagawa Station" } ] } } } } }, "post": { "operationId": "POST-atm-location", "summary": "Create ATM Location", "tags": [ "Atm locations" ], "parameters": [ { "name": "body", "in": "body", "schema": { "$ref": "#/definitions/api-location-input" } } ], "responses": { "201": { "description": "", "schema": { "$ref": "#/definitions/api-location-input" } } } } } }, "definitions": { "atm-location-input": { "title": "ATM Location Input", "type": "object", "properties": { "location": { "type": "string" }, "lat": { "type": "string" }, "lon": { "type": "string" } }, "required": [ "location" ] } }

### Get an ATM Location

The entries with id 1 and 2 are prepopulated when the container starts:

`curl http://localhost:8080/api/atm/1`

> {"lat": "35.6284713", "lon": "139.736571", "location": "Shinagawa Station"}

`curl http://localhost:8080/api/atm/2`

> {"lat": "35.6684231", "lon": "139.6833085", "location": "Ebisu Station"}

### Create an ATM

`curl -X POST http://localhost:8192/api/atm -d '{ "lat": "35.4657858", "lon": "139.6201245,17", "location": "Yokohama Station" }'"`

> {"message": "created", "id": "565"}

### Delete an ATM

`curl -X DELETE http://localhost:8080/api/atm/565`

> {"message": " 565 deleted"}

### Update an ATM

`curl -X PUT http://localhost:8080/api/atm/105 -d '{"lat": "123", "lon": "982", "location": "some place"}'`

> {"message": "updated", "id": 105}

### Get all ATM's

`curl http://localhost:8080/api/atm`

> {"result": [{"lat": "35.6684231", "lon": "139.6833085", "location": "Ebisu Station", "id": "2"}, {"lat": "35.6284713", "lon": "139.736571", "location": "Shinagawa Station", "id": "1"}]}






# Stock Quote - REST/JSON

This API returns the stock price of Yoisho Banking Corp. The data updates every minute, it's also sending additional Cache-Control headers.

### Run the container

`docker run --rm -d -p 8080:8080 u1ih/yoisho-stockquote:latest`

### Get the Swagger

`curl http://localhost:8080/swagger`

> { "swagger" : "2.0", "host" : "", "basePath" : "", "schemes" : [ "http" ], "paths" : { "/get_quote" : { "get" : { "description" : "", "operationId" : "get_quote", "produces" : [ "application/json" ], "responses" : { "default" : { "description" : "successful operation" } } } } }, "info" : { "title" : "Yoisho Stock Quote", "description" : "", "version" : "1.0" }, "x-axway" : { "corsEnabled" : true, "basePaths" : [ "" ], "serviceType" : "rest", "deprecated" : false, "tags" : { } } }

### Get current quote

`curl http://localhost:8080/get_quote`

> {"message": "Stock Price refreshes every minute", "stockprice": "371.44"}

# Fixed Deposit Calculator - REST/JSON

This API calculates the total interest earned from a fixed deposit, along with a breakdown for the number of years the deposit is running. Parameters: amount, number of years the fixed deposit will run

### Run the container

`docker run --rm -d -p 8080:8080 u1ih/yoisho-deposit:latest`

### Get the Swagger

`curl http://localhost:8080/swagger`

> { "swagger" : "2.0", "host" : "", "basePath" : "/fixeddeposit", "schemes" : [ "http" ], "paths" : { "/calculate" : { "get" : { "description" : "Calculates the total interest earned from a fixed deposit, along with a breakdown for the number of years the deposit is running.", "operationId" : "calculate", "produces" : [ "application/json" ], "parameters" : [ { "description" : "The number of years the fixed deposit is running.", "required" : true, "in" : "query", "name" : "years", "type" : "string" }, { "description" : "The amount for the fixed deposit", "required" : true, "in" : "query", "name" : "amount", "type" : "string" } ], "responses" : { "200" : { "description" : "OK" }, "400" : { "description" : "Invalid request" } } } } }, "info" : { "title" : "Fixed Deposit Calculator", "description" : "", "version" : "1.0" } }

### Perform Calculation

`curl "http://localhost:8080/calculate?amount=50000&years=12"`

> {"info": "Current interest rate is 3.15% pa.", "yield_breakdown_by_year": [{"amount": "50,000.00", "year": "1"}, {"amount": "51,575.00", "year": "2"}, {"amount": "53,199.61", "year": "3"}, {"amount": "54,875.40", "year": "4"}, {"amount": "56,603.98", "year": "5"}, {"amount": "58,387.00", "year": "6"}, {"amount": "60,226.19", "year": "7"}, {"amount": "62,123.32", "year": "8"}, {"amount": "64,080.20", "year": "9"}, {"amount": "66,098.73", "year": "10"}, {"amount": "68,180.84", "year": "11"}, {"amount": "70,328.53", "year": "12"}], "deposit_amount": "50,000.00", "yield_amount": "70,328.53", "years": "12"}

# Account Balance - REST/JSON OAuth

Gives you an account balance API with a 3-legged OAuth flow including consumer banking login.

##### 3-legged OAuth components

1. Consumer: the app, identified by client\_id and client\_secret
2. Resource Owner: the banking customer, here it's Dave and Jane
3. API: the account balance service

![](https://raw.githubusercontent.com/u1i/yoisho/master/resources/account-2.png)

### Credentials:

* client_id: 7b6fc8ed5127b0b2f076d
* client_secret: 724e6890757b0ae624684b70e111b705fe6b050c
* user accounts: jane, dave (use any random password)

### Run the container

`docker run --rm -d -p 8080:8080 u1ih/yoisho-account:latest`

### Start the flow

Access this URL in the browser (replace the redirect_uri with your own):

`http://localhost:8080/authorize?redirect_uri=http://www3.sotong.io&client_id=7b6fc8ed5127b0b2f076d`

Login with either 'dave' or 'jane' (any random password).

> ?code=4990047

### Get the access_token - valid for 60 seconds

Use the code from the previous step to exchange it for an access token. Of course, you'd typically do that server side.

`curl http://localhost:8080/access_token?code=4990047&client_id=7b6fc8ed5127b0b2f076d&client_secret=724e6890757b0ae624684b70e111b705fe6b050c`

> {"token_type": "bearer", "scope": "read", "access_token": "eSlcNwnLxTuzsYXyzFrhGGU3mrCKPxQ5fy51Jx93.MTUzMjM1NDM0OA=="}
>

### Get account balance for user

Now with this access token we can call the actual API and retrieve the account balance from the resource owner, which is the banking client.

`curl -X GET http://localhost:8080/balance -H 'Authorization: Bearer eSlcNwnLxTuzsYXyzFrhGGU3mrCKPxQ5fy51Jx93.MTUzMjM1NDM0OA=='`

> {"account_owner": "dave", "account_balance": "10,187.91"}
> 

### OAuth client - Python implementation

Here's a simple web based client implementation that shows the entire flow in the browser - the banking customer logs in and authorizes the app to access the account balance on their behalf: [Yoisho OAuth Client (Python)](https://github.com/u1i/yoisho/tree/master/yoisho-account-oauth-client)


[What does yoisho mean?](https://www.japantimes.co.jp/life/2013/03/25/language/yoisho-a-word-to-move-mountains-and-smaller-things/#.W2a6mv4zbMI)
