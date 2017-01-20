## Correlation/Request IDs
Read: [Building Microservices](https://www.amazon.com/Building-Microservices-Sam-Newman/dp/1491950358) Chapters 8

### Rationale

Logging is hard.

Ensuring that you have useful data being logged at the correct severity level *before* systems start going down at 3am is hard enough.  Logging too much data can often increase the size of the haystack without adding any proverbial needles.  A microservices architecture can exacerbate the issue by breaking up events between systems.

If your Orders service throws when getting inventory, it can be non-trivial to find the corresponding message logged by the Inventory service.  And good luck discovering that the problem didn't even originate with the Inventory serivce, but was from a temporary outage of the Warehouse service.

In chapter 8, Sam Newman describes the concept of the **correlation ID**, a value than can be used to identify all messages that originated from the same event. By passing these IDs between systems and including them in log messages, discovering the circumstances surrounding errors gets a lot more sane.

There are cases where calling systems may provide the same correlation ID on subsequent requests, whether by intent or by accident.  Because of this, it may be difficult to distinguish log messages from different request.  To combat this, it is recommended that in addition to a **correlation ID**, a unique **request ID** be generated at the start of each new request.

GUIDs are reasonable for use as correlation and request IDs, however systems should not require them to be.  Remember, the purpose of a correlation ID is to correlate log messages across multiple systems.  The purpose of a request ID is to correlate log messages within a single system.  Any values capable of doing so should be considered sufficient.

### HTTP Implementation

The minimal implementation for a web service should look like:

1. Get the IDs
    * Get the correlation ID from a `X-Correlation-ID` HTTP header or create a new GUID if the header is not provided
    * Create a new GUID for the request ID
1. Store both IDs a request-scoped context accessible from all code
1. Pass the correlation ID to all downstream services with a `X-Correlation-ID` HTTP header
1. Pass the correlation ID and request ID back upstream in a `X-Correlation-ID` and `X-Request-ID` HTTP header
1. Include the current requests correlation AND request IDs in ALL log messages