# Stability
A brain dump of RESTful stability patterns

## Preface
This work provides a specification for stability features that assume nothing but the HTTP protocol.  The result is that these features can be implemented in any language using any framework and deployed on any platform.  Language-specific code can (and likely should) be written and reused in order to reduce the cost of stability, but the purpose of this document is to standardize on a set of specifications rather than on any particular technology stack.

Not all applications mandate the same SLA.  An outage of a shopping cart service might prevent an OLTP system from processing any transactions, while a recommendation service could degrade or go down altogether but still allow the OLTP system as a whole to continue operating.  The decision of which stability features to implement should take into consideration the cost of both implementation and maintenance as compared to the cost and frequency of system failure.

Consider also that some features are more easily implemented in various languages or with various frameworks.  For example, middleware to manage HTTP headers is trivial to implement in javascript using Express.  Feature switching using interface implementations is trivial in a strongly typed language such as C# using a DI framework such as StructureMap.  Therefore, the selection of technology stacks should take into consideration which features are needed and thus their complexity and cost.

Lastly, stability is an investment.  It costs time and resources to implement, but the return on investment comes in the form of time and resources not spent on recovering from system failures.  While not all failures can be prevented, their impact can be mitigated which reduces their cost.  Redesigning stability features in each system is expensive.  Similar conversations happen repeatedly and the solutions that are reached have a bias toward solving a particular problem in a particular technology stack.  This repeated effort is costly but does not by default generate a higher ROI.  This results in a hesitation to add common stability features because they are not “worth it.”  An engineered approach to stability reduces its cost, and when stability is cheap it is far more likely to happen.

The following are specific guidelines which define various stability features in terms of the HTTP protocol.  Each feature is accompanied with recommended reading which provides source material and rationale.  Discussion concerning updates to these guidelines are most productive when all participants are on the same page (pun intended).

## [Circuit Breakers](./docs/CircuitBreakers.md)

## [Correlation IDs](./docs/CorrelationIds.md)

## License & Copyright

The materials herein are all (c) 2016-2017 Steve Konves.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivs 4.0 Unported License</a>.