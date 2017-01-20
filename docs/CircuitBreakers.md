## Circuit Breakers
Read: [Release it!](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213) Chapters 4, 5.2

“Preventing cascading failures is the very key to resilience.  The most effective patterns to combat cascading failures are Circuit Breaker and Timeouts.” - [Release it!](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213) Ch. 4

“Circuit breakers are a way to automatically degrade functionality when the system is under stress” -[ibid](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213) Ch. 5.2

“Changes in a circuit breaker’s state should always be logged, and the current state should be exposed for querying and monitoring.” -[ibid](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213)

“Popping a Circuit Breaker always indicates there is a serious problem.  It should be visible to operations.  It should be reported, recorded, trended, and correlated.” -[ibid](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213)

### Response Categories
Based on the state of its dependencies, a service can respond in one of three ways: 

* Nominal - All underlying functionality is available
* Degraded - One or more dependencies are unavailable, but a meaningful yet partial response can still be returned.
* Unavailable - One or more dependencies are unavailable which prevent a meaningful response from being returned.

### Responding to Requests
The service should communicate the state of its internal circuits by using HTTP status codes and headers:

* In an unavailable state
  * `HTTP 503` is returned
  * A `Retry-After` header is returned to indicate at which point in time all dependencies will be available
  * A `X-Tripped-Circuits` header is returned containing the IDs of all tripped circuits
* In a degraded state
  * `HTTP 2xx` is returned
  * A `Retry-After` header is returned to indicate at which point in time all dependencies will be available
  * A `X-Tripped-Circuits` (rename?) header is returned containing the IDs of all tripped circuits
* In a nominal state
  * `HTTP 2xx` is returned
  * No additional headers are needed

### Calling Circuit-Broken Dependencies
* Service making the call should be circuit-broken
* On an `HTTP 503`
  * Trip this circuit
  * Set the circuit to reopen at the time specified by the response’s `X-Retry-After` header
* On an `HTTP 2xx` with a `X-Tripped-Circuits` and/or `X-Retry-After` header
  * Log the fact that a degraded response was returned
  * Determine if the degraded response is sufficient for further operation
  * If not
    * Trip this circuit
    * Set the circuit to reopen at the time specified by the response’s `X-Retry-After` header

### Exposing Circuit States
A service’s collection of circuit breakers, collectively referred to as a Breaker Panel, should be exposed via HTTP in order for other systems to determine the health of this system.  The response from this route should include a collection of Circuit Breaker summaries that at a minimum shows their ID and state.

The following is a recommended collection of circuit-related routes:

* `GET /health/circuits` - gets a list of circuit breakers
* `GET /health/circuits/:id` - gets info about a particular circuit breaker
* `GET /health/circuits/:id/history` - gets a collection of circuit breaker state changes
* `GET /health/circuits/:id/state` - gets the current state of a circuit breaker
* `PUT /health/circuits/:id/state` - changes the state of a circuit breaker

### Representation of Circuit State
The representation of a circuit should, at a minimum, include:

#### ID/Name
This value should be unique across all circuits defined for an application.  Also note that there is a distinction between the *circuit* being publicly exposed and the implementation details being concealed.  For example, if you application serves documents from a Google Docs API, your circuit would be named 'documents' or 'documents-service' rather than 'google-docs-api'.

#### Availability
This should be a simple boolean value indicating whether or not a request via this circuit should be expected to succeed.  Even if your circuit implementation includes a "half-open", or "half-on" state, an `isAvailable` propery exposes a simple `true`/`false` value.  This prevents consuming systems from having to hard-code knowledge about the names of this systems circuit states.  The availability, reason, and timestamp values can be included as a single sub-object (see history).

#### Reason for Current State
This exposes why the circuit is in the state that it is in.  This helps to determine if the circuit was manually tripped or if it was due to an error with the underlying dependency.  If a unique error ID is generated, consider including it in the reason text to aid debugging.  The availability, reason, and timestamp values can be included as a single sub-object (see history).

#### DateTime of last state change
Simply the point in time that the state changed to the current value.  Time should be in UTC and formated via an established standard.  The availability, reason, and timestamp values can be included as a single sub-object (see history).

#### History
This exposes a collection of the last several state changes.  If the availability, reason, and timestamp values are grouped as a sub-object, then the history can simply be an array of the same type of object.  The collection should have a fixed upper limit of items to prevent massive amounts of data from being transferred, but doesn't necessarily need to support paging.  If a long history of circuit state is needed, then the application logs will likely be a more appropriate place to go looking for data.

#### JSON Example
A JSON representation of a circuit breaker might look like:

``` JSON
{
  "id": "document-service",
  "current": {
    "isAvailable": true,
    "asOf": 1484435696458,
    "reason": "manually reset by {username}"
  },
  "history": [
    {
      "isAvailable": false,
      "asOf": 1484435680265,
      "reason": "tripped at error f39ba71c-bbf6-4b3e-965b-2b9612bbe1ff"
    },
    {
      "isAvailable": true,
      "asOf": 1484435400655,
      "reason": "application start"
    }
  ]
}
```
