# vendor-proxy-service

This microservice aims to be a proxy for any vendor endpoint developed according to https://lblod.github.io/pages-vendors/#/docs/vendor-sparql.

## Setup

In order to configure it, you will need to provide the following environment variables:

- `QUERY_BASE_URL`: The base url of your vendor endpoint, so if you query endpoint is `https://mandatenbeheer.lblod.info/vendor/query` your query base url will be `https://mandatenbeheer.lblod.info`.
- `VENDOR_KEY`: The key provided by the vendor for authentication.
- `VENDOR_URI`: The uri provided by the vendor for authentication.
- `AUTH_GROUP`: The auth group to check for the administrative unit the user belongs to.

You will also need to add the service to your dispatcher as the service needs to get your authGroups in order to determine which administrative unit do you belong, something like this
```
 options "/vendor-proxy/*path", _ do
    conn
    |> Plug.Conn.put_resp_header( "access-control-allow-headers", "content-type,accept" )
    |> Plug.Conn.put_resp_header( "access-control-allow-methods", "*" )
    |> send_resp( 200, "{ \"message\": \"ok\" }" )
  end

  match "/vendor-proxy/*path" do
    forward conn, path, "http://vendor-proxy/"
  end
```

The options part is only needed if you will do some cors requests

## Usage
Once that is done you will be able to access your service with a GET request at `/` that will respond with a hello world message
To query your endpoint you can send a POST request to `/query` with a json body that has a query parameter with the query you want to perform, like:
```
{
    "query": "SELECT * WHERE { ?a ?b ?c } LIMIT 100"
}
```

The service will then redirect the response of the vendor endpoint to your client


## Errors

Possible errors that can arise when using the service but they should be pretty self explanatory:

- Status code `500` and body `{error: 'Missing X environment variable}`: the environment variable specified is missing.
- Status code `401` and body `{error: 'You should me logged to access this service'}`: the service couldn't determine which administrative unit you belong to
- Status code `400` and body `{error: 'Please specify a query to perform'}`: you didn't include a query in your POST body.
- Other errors: The service also redirects the errors from the vendor endpoints, so you may be receiving other errors, hopefully they are self explanatory, if they are not contact the administrator of your endpoint
