# java-iod-client

Java Client for communicating with [IDOL OnDemand](http://www.idolondemand.com)

## About
java-iod-client provides a Java Interface to the IDOL OnDemand APIs to simplify calling the IDOL OnDemand APIs from Java,
allowing consumers of java-iod-client to access IDOL OnDemand in a typesafe way.

## Goals
Currently only a small subset of IDOL OnDemand APIs are supported. The eventual aim to support all the IDOL OnDemand
APIs. We also plan to provide support for asynchronous APIs, and automated error handling of IDOL OnDemand error codes.

If an API you need is not supported, submit a pull request!

## Usage

java-iod-client is available from the central Maven repository.

    <dependency>
        <groupId>com.hp.autonomy.iod</groupId>
        <artifactId>java-iod-client</artifactId>
        <version>0.3.0</version>
    </dependency>

java-iod-client uses [Retrofit](http://square.github.io/retrofit/) as the basis of its HTTP implementation. This
requires the use of a RestAdapter to use the services. We have used [Jackson](https://github.com/FasterXML/jackson) for
JSON transformation, so you will need to use the Jackson Converter. To send multipart requests to IDOL OnDemand
correctly, you will need to wrap this in an IodConverter. An error handler is supplied for parsing error responses from
IDOL OnDemand.

    final RestAdapter restAdapter = new RestAdapter.Builder()
        .setEndpoint("https://api.idolondemand.com/1")
        .setConverter(new IodConverter(new JacksonConverter()))
        .setErrorHandler(new IodErrorHandler())
        .build();

    final QueryTextIndexService queryTextIndexService = restAdapter.create(QueryTextIndexService.class);

You can then call the methods on queryTextIndexService to communicate with IDOL OnDemand.

    final Map<String, Object> params = new QueryTextIndexRequestBuilder()
        .setAbsoluteMaxResults(10)
        .setTotalResults(true)
        .setIndexes("wiki_eng")
        .build();

    final Documents documents = queryTextIndexService.queryTextIndexWithText(
        "myApiKey",
        "cats",
        params);

## Asynchronous requests
For asynchronous actions the Retrofit service returns a JobId. We also provide a service which will track the status of
the job IDs.

    final AddToTextIndexJobService addToTextIndexService = new AddToTextIndexJobService(restAdapter.create(AddToTextIndexJobService.class));

The methods on this service take a callback which will be called when the job is completed

    addToTextIndexService.addFileToTextIndex(getApiKey(), file, getIndex(), params, new IodJobCallback<AddToTextIndexResponse>() {
        @Override
        public void success(final AddToTextIndexResponse result) {
            // called if the job succeeds
        }

        @Override
        public void error(final IodErrorCode error) {
            // called if the job fails
        }

        @Override
        public void handleException(final RuntimeException exception) {
            // called if a RuntimeException is thrown during the process
        }
    });


The APIs which are currently asynchronous are

* AddToTextIndex
* DeleteFromTextIndex

## Is it any good?
Yes.

## License
Copyright 2015 Hewlett-Packard Development Company, L.P.

Licensed under the MIT License (the "License"); you may not use this project except in compliance with the License.