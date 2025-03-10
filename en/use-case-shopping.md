---
# Copyright Yahoo. Licensed under the terms of the Apache 2.0 license. See LICENSE in the project root.
title: "Use Case - shopping"
redirect_from:
- /documentation/use-case-shopping.html
---

The [e-commerce, or shopping, use case](https://github.com/vespa-engine/sample-apps/tree/master/use-case-shopping)
is an example of an e-commerce site complete with sample data and a web front
end to browse product data and reviews. To quick start the application, follow
the instructions in the
[README](https://github.com/vespa-engine/sample-apps/blob/master/use-case-shopping/README.md)
in the sample app.

To browse the application, navigate to
<a href="http://localhost:8080/site" data-proofer-ignore>localhost:8080/site</a>.
This site is  implemented through a custom [request handler](jdisc/developing-request-handlers.html)
and is meant to be a simple example of creating a front end / middleware that
sits in front of the Vespa back end. As such it is fairly independent of Vespa
features, and the code is designed to be fairly easy to follow and as
non-magical as possible. All the queries against Vespa are sent as HTTP
requests, and the JSON results from Vespa are parsed and rendered.

This sample application is built around the Amazon product data set found at
[http://jmcauley.ucsd.edu/data/amazon/links.html](http://jmcauley.ucsd.edu/data/amazon/links.html).
A small sample of this data is included in the sample application, and full
data sets are available from the above site. This sample application contains
scripts to convert from the data set format to Vespa format. These are the
`convert_meta.py` and `convert_reviews.py`. See the README file for example of
use.

When feeding reviews, there is a custom [document processor](document-processing.html)
that intercepts document writes and updates the parent item with the review
rating, so the aggregated review rating is kept stored with the item. This is
more an example of a custom document processor than a recommended way to do
this, as feeding the reviews more than once will result in inflated values. To
do this correctly, one should probably calculate this offline so a re-feed does
not cause unexpected results.

### Highlighted features

* [Multiple document types](schemas.html)

    Vespa models data as documents, which are configured in schemas
    that defines how documents should be stored, indexed, ranked, and searched.
    In Vespa, you can have multiple documents types, which can be defined in
    `services.xml` how these should be distributed around the content clusters.
    This application uses two document types that both are stored in the same
    content cluster: item and review. Search is done on items, but reviews
    refer to a single parent item and are rendered on the item page.

* [Custom document processor](document-processing.html)

    In Vespa, you can set up custom document processors to perform any type of
    extra processing during document feeding. One example is to enrich the
    document with extra information, and another is to precalculate values of
    fields to avoid unnecessary computation during ranking. This application
    uses a document processor to intercept reviews and update the parent item's
    review rating.

* [Custom handlers](jdisc/developing-request-handlers.html)

    With Vespa, you can set up general request handlers to handle any type of
    request. This example site is implemented with a single such request
    handler, `SiteHandler` which is set up in `services.xml` to be bound to
    `/site`. Note that this handler is for example purposes and is designed to
    be independent of Vespa. Most applications would serve this through a dedicated
    setup.

* [Custom configuration](configuring-components.html)

    When creating custom components in Vespa, for instance document processors,
    searchers or handlers, one can use custom configuration to inject config
    parameters into the components. This involves defining a config definition
    (a `.def` file), which creates a config class. You can instantiate this
    class with data in `services.xml` and the resulting object is dependency
    injected to the component during construction. This application uses custom
    config to set up the Vespa host details for the handler.

* [Partial update](reference/document-json-format.html#update)

    With Vespa, you can make changes to an existing document without submitting
    the full document. Examples are setting the value of a single field, adding
    elements to an array, or incrementing the value of a field without knowing
    the field value beforehand. This application contains an example of a
    partial update, in the voting of whether a review is helpful or not.  The
    `SiteHandler` receives the request and the `ReviewVote` class sends a
    partial update to increment the `up`- or `downvotes` field.

* [Search using YQL](query-language.html)

    In Vespa, you search for documents using YQL. In this application, the
    classes responsible for retrieving data from Vespa (in the `data` package
    beneath the `SiteHandler`) set up the YQL queries which are used to query
    Vespa over HTTP.

* [Grouping](grouping.html)

    Grouping is used to group various fields of query results together.  For
    this application, many of the queries to Vespa include grouping requests.
    The home page uses grouping to dynamically extract the first 3 levels of
    categories from the stored items. The search page groups results matching
    the query into categories, brands, item rating and price ranges. The item
    page groups review ratings to construct the rating graph.

* [Rank profiles](ranking.html)

    Rank profiles are profiles containing instructions on how to score
    documents for a given query. The most important part of rank profiles are
    the ranking expressions. The schemas for the item and review
    document types contain different ranking profiles to sort or score the
    data.

* [Ranking functions](reference/schema-reference.html#function-rank)

    Ranking functions are contained in ranking profiles and can be referenced
    as part of any ranking expression from either first phase, second phase or
    other functions.


Going forward, there are quite a few things one could add to this application
to get a more functional site. One important thing is user handling, with
features such as user profiles, recently viewed items, marking of favorites
etc. Then one could use a custom searcher that retrieves the user profile, for
instance based on a cookie, and uses the profile during search to personalize
results. However, this is currently left as exercises for the reader.

