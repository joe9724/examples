# Google Cloud Endpoints

This example deploys a goa service to 
[Appengine Flex](https://cloud.google.com/appengine/docs/flexible/) and
[Google Cloud Endpoints](https://cloud.google.com/endpoints/).
Google Cloud Endpoints consumes the OAI (Swagger) spec generated by `goagen` to configure the API
endpoints. This makes deploying goa services behind Endpoints effortless. 

With Endpoints goa services automatically get authentication support, centralized logging via
Google Stackdriver, world class monitoring and more.

Check out the [blog entry](https://goa.design/blog/002-endpoints/) for more details.

The instructions below assume that goagen has been installed:

```
go get github.com/goadesign/goa/goagen
```

## Setup and Code Generation

The file [main.go](main.go) contains a `go:generate` comment which invokes `goagen` to
generate the source of the example from the `design` package. 

Run `go generate` to produce the entire source:

```
go generate
```

## Running the Example

The example can then be built and run locally, nothing special:

```
go build
./endpoints
```

## Deploy to Appengine Flex

The steps below assumes that the example is compiled and works locally. Also note that you need to
setup a proper Google project with billing enabled before you can deploy to Appengine and configure
Google Cloud Enpoints. The configuration included in this example limits the number of instances
launched by Appengine to only one. The default instance type is `g1 small` which costs about 3c per
hour (see [Google Compute Engine Pricing](https://cloud.google.com/compute/pricing)).

*1*. Follow the *Before you begin* section of the [Quickstart for Endpoints on App Engine](https://cloud.google.com/endpoints/docs/quickstart-app-engine) guide to setup the Google project that will host the goa service. Note the ID of the project, referred to as `YOUR_PROJECT_ID` in the rest of this document.

*2*. Replace the `Host` value in the design with the string `YOUR_PROJECT_ID.appspot.com`. This value is the hostname of the deployed service used by Google Cloud Endpoints to forward traffic.

*3*. Replace the `Metadata` values in the `jwt` action to match your project settings.

*4*. Re-generate the OAI spec:

```
goagen swagger -d github.com/goadesign/examples/endpoints/design
```

*5*. Install [aedeploy](https://github.com/golang/appengine/tree/master/cmd/aedeploy):

```
go get -u google.golang.org/appengine/cmd/aedeploy
```

*6*. Deploy the service:

```
aedeploy gcloud beta app deploy
```

If everything goes well you now have the service running in Appengine Flex and fronted by Endpoints!

## Using the Client

Compile and run the client in a different terminal:

```
cd client/adder-cli
go build
./adder-cli --host=YOUR_PROJECT_ID.appspot.com basic /auth/info/basic?key=YOUR_API_KEY
2016/03/21 00:33:10 [INFO] started id=nclom9xa GET=http://localhost:8080/add/1/2
2016/03/21 00:33:10 [INFO] completed id=nclom9xa status=200 time=20.102916ms
3
```

Where:

* `YOUR_PROJECT_ID` is the ID of the GCP project containing the Appengine Flex app.
* `YOUR_API_KEY` is the API key retrieved from the [Google
Console](https://console.cloud.google.com/apis/credentials/key?type=SERVER_SIDE).

To use JWT auth first edit the JWT claim created in the file `tool/adder-cli/jwt.go` so that:

- the `Iss` field matches the `issuer` value set in design Metadata.
- the `Aud` field matches the `audiences` value set in the design Metadata.

then recompile and run the client using the `--jwt` flag to enable JWT signing:

```
go build
./adder-cli jwt auth --jwt
2016/09/21 01:09:12 [INFO] started id=dqbZVS23 GET=http://goa-endpoints.appspot.com/auth/jwt
2016/09/21 01:09:12 [INFO] completed id=dqbZVS23 status=200 time=129.817565ms
{"id":"client@goa.design","issuer":"client.goa-endpoints.appspot.com"}
```