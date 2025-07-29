# Using Power Query to Call API via Service Principal

This guide demonstrates how to use Power Query to obtain an access token via a service principal (SP) and call a REST API. In these examples, anonymous authentication is used since the credentials are specified directly in the query.

---

## 1. Retrieve Access Token Using Service Principal

The following M code retrieves an access token for the service principal:

```powerquery
let
    url = "https://login.microsoftonline.com/TENANTID/oauth2/token",
    body = [
        grant_type = "client_credentials",
        client_id = "XXXX",
        client_secret = "XXXX",
        resource = "https://analysis.windows.net/powerbi/api"
    ],
    Source = Json.Document(Web.Contents(url, [
        Content = Text.ToBinary(Uri.BuildQueryString(body)),
        Headers = [#"Content-Type" = "application/x-www-form-urlencoded"]
    ])),
    access_token = Source[access_token]
in
    access_token
```

---

## 2. Combine Token Retrieval and API Call in One Query

You can encapsulate the token retrieval and API call in a single query as shown below:

```powerquery
let
    GetAccessToken = () =>
        let
            url = "https://login.microsoftonline.com/TENANTID/oauth2/token",
            body = [
                grant_type = "client_credentials",
                client_id = "XXXX",
                client_secret = "XXXX",
                resource = "https://analysis.windows.net/powerbi/api"
            ],
            Source = Json.Document(Web.Contents(url, [
                Content = Text.ToBinary(Uri.BuildQueryString(body)),
                Headers = [#"Content-Type" = "application/x-www-form-urlencoded"]
            ])),
            access_token = Source[access_token]
        in
            access_token,

    // Retrieve the access token
    token = GetAccessToken(),

    // Define the API endpoint
    url = "https://api.powerbi.com/v1.0/myorg/admin/activityevents?startDateTime='2024-10-21T00:55:00.000Z'&endDateTime='2024-10-21T23:55:00.000Z'",

    // Set the authorization header
    headers = [Authorization = "Bearer " & token],

    // Execute the API request
    Source = Json.Document(Web.Contents(url, [Headers = headers]))
in
    Source
```

---

## 3. Separate Token Retrieval and API Call into Two Queries

You may also separate the logic into two queries: one for the token function and one for the API call.

**Define the token function (name this query `GetAccessToken`):**

```powerquery
let
    GetAccessToken = () =>
        let
            url = "https://login.microsoftonline.com/TENANTID/oauth2/token",
            body = [
                grant_type = "client_credentials",
                client_id = "XXXX",
                client_secret = "XXXX",
                resource = "https://analysis.windows.net/powerbi/api"
            ],
            Source = Json.Document(Web.Contents(url, [
                Content = Text.ToBinary(Uri.BuildQueryString(body)),
                Headers = [#"Content-Type" = "application/x-www-form-urlencoded"]
            ])),
            access_token = Source[access_token]
        in
            access_token
in
    GetAccessToken
```

**Call the function and run the REST API:**

```powerquery
let
    // Retrieve the access token
    token = GetAccessToken(),

    // Define the API endpoint
    url = "https://api.powerbi.com/v1.0/myorg/admin/activityevents?startDateTime='2024-10-21T00:55:00.000Z'&endDateTime='2024-10-21T23:55:00.000Z'",

    // Set the authorization header
    headers = [Authorization = "Bearer " & token],

    // Execute the API request
    Source = Json.Document(Web.Contents(url, [Headers = headers]))
in
    Source
```
