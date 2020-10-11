# Creating Your First QEWD REST API

This document assumes that you've got a working QEWD system up and running.

The instructions below assume that you've installed your QEWD system in
a folder named ~/qewd.  If you have installed QEWD in a different folder path, 
adjust the paths in the instructions appropriately.

## The *routes.json* File

In order for QEWD to accept an incoming REST request, it must be defined in a
special file: *~/qewd/configuration/routes.json*

This file **MUST** specify a valid JSON array, ie all property names and string
values must be double quoted, and it cannot include comments using *//*.

The *routes.json* array contains one or more API route objects, each 
specified with 3 properties:

- uri: the URI path for the REST API, eg */api/test*
- method: the HTTP method for the REST API, eg *GET*
- handler: the name of the JavaScript module that you will have written to
handle the incoming request

Here's an example route object:

        {
          "uri": "/api/test",
          "method": "GET",
          "handler": "test"
        }

**Note:** QEWD REST API uris **MUST** have at least 2 parts to their path.  In other words,
a uri such as */test* is invalid and will cause an error in QEWD.

## Create Your First *routes.json* File

Create the file *~/qewd/configuration/routes.json* and paste the following into it:

        [
          {
            "uri": "/api/test",
            "method": "GET",
            "handler": "test"
          }
        ]


## REST API Handler Modules

QEWD looks for your specified handler modules in a folder named *apis*, eg *~/qewd/apis*.
The handler name specified in each route object must have a correspondingly named
module in the */apis* folder.

So, in our example above, the handler for the *GET /api/test* API was named *test*.  So
QEWD will look for a module in the folder *~/qewd/apis/test*. Within that folder, it
will expect to find your handler's logic in a file named *index.js*

So, in summary, the */api/test* example will require the following files in your QEWD
directory:

        ~/qewd
            |
            |— configuration
            |            |
            |            |— routes.json
            |
            |— apis
            |    |
            |    |— test
            |         |
            |         |— index.js


All QEWD REST API Handler Modules have the same signature:

      module.exports = function(args, finished) {

        // ... perform your logic here using
        // the REST request contents in args,
        // create a response object, then
        // finish by invoking:

        finished(responseObject);
      };


In summary, QEWD REST Handler modules export a function with 2 arguments:

- **args**: an object that contains the content of your REST request, including its path, headers, HTTP method, any querystring values and any body payload

- **finished**: the QEWD function that you use to end your handler.  This function releases the QEWD worker process that handled your module back to QEWD's available worker pool, and tells QEWD to return the object you provide as its argument as a JSON response to the REST client that sent the original request.

So let's create a simple handler for our test API, specifically let's make it return the *args* object
as the response, so you can get an idea of how an incoming REST request is packaged up for you into that
*args* object.

Create the file *~/qewd/apis/test/index.js* and paste the following into it:

      module.exports = function(args, finished) {
        finished(args);
      };


You're now ready to try out your first QEWD REST API.


## Try out the Test API

If you already had QEWD running, you'll find that you can't yet try out the test API.

QEWD only reads the contents of the *routes.json* file when it first starts up.  So every time
you add a new route object to the *routes.json* file, you'll have to restart QEWD in the usual way.

Once restarted, in a browser enter the URL:

        http://x.x.x.x:3000/api/test

and you should get back the response:

        {
          req: {
            type: "ewd-qoper8-express",
            path: "/api/test",
            method: "GET",
            headers: {
              host: "192.168.1.207:8080",
              connection: "keep-alive",
              upgrade - insecure - requests: "1",
              user - agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36",
              accept: "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
              accept - encoding: "gzip, deflate",
              accept - language: "en-GB,en-US;q=0.9,en;q=0.8",
              cookie: "io=gUatnFSBn9dzHGN2AAAA"
            },
            params: {
                type: "test"
            },
            query: {},
            body: {},
            ip: "::ffff:192.168.1.77",
            ips: [],
            application: "api",
            expressType: "test"
          },
          session: {}
        }

You'll see that the *args.req* object contains all the information you'll need when processing an
incoming REST request.

Try changing the URL in the browser to:


        http://x.x.x.x:3000/api/test?foo=bar&hello=world

You'll see that those queryString name/value pairs you added now appear in
*args.req.query*:

        query: {
          foo: "bar",
          hello: "world"
        }

Congratulations! You have your first QEWD REST API working!


## Recommended Next Steps

Read this [detailed tutorial](https://github.com/robtweed/qewd-baseline/blob/master/REST.md)
 on developing REST APIs.  

One thing to note in the above tutorial: it assumes you're using the
Dockerised version of QEWD, which makes no difference to the files you create, but when QEWD
needs to be stopped and restarted, it describes how you stop and restart the QEWD Docker
container.  Ignore these instructions and:

- use *CTRL&C* or the *qewd-monitor* application to stop QEWD
- use *npm start* to start QEWD

You'll also see that the paths referred to in the tutorial assume QEWD is installed
at *~/qewd-baseline*.  Substitute these references with *~/qewd*.








