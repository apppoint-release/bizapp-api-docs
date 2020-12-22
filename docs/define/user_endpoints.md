User REST Endpoints
==========================
User specific end points are similar to infrastructure end points, but differ in a way role based access control(RBAC) is enforced. The end points assume the user context and allow
operations that an user can perform based on his role and responsibilities. This helps to enforce data security and integrity while exposing the business logic via public REST end points.

## Supported mechanism
Bearer authentication is the only mechanism supported. It requires the user to acquire a token by supplying user credentials for login. Post successful login, a token is issued, which is used for further
API calls.

## Defining API's in Modeler
BizAPP Modeler allows you to define API's using the industry standard best practices and strategies. API's are defined at an application level and thus helps to logically combine function specific 
API's and enforce access control.

It is mandatory to set the route template at the application level. The following snapshot shows one such example of setting the base route template URI for all API's defined under an application.

![application-routing](../images/application-route.png)

### API Versioning
One of the major challenges around expsosing services is handling updates to the API contracts. Clients may not want to update their application when the API changes, hence a versioning
strategy is crucial. Versioning strategy allows clients to continue to use the existing API and migrate their applications to newer API when they are ready.

#### Versioning through URI Path
This is one of the most commonly used mechanism to version API's. BizAPP Modeler implictly enforces this mechanism to ensure that URI includes a versioning strategy.
Internal version of the API can use 1.2.3 format, so it looks like as follows.

![api version](../images/api-versioning.png)

* **Major Version** : The version used in the URI and denotes a breaking changes to the API. Internally a new major version implies creating a new API and version number is used 
to route to the correct version of the API.

* **Minor and Patch versions** : These are transparent to the client and used internally for backward-compatible updates. They are usually communicated in change logs to 
inform clients about a new functionality or a bug fix. Minor and Patch version is optional can be managed with only Major version alone.

This solution often uses URI routing to point to a specific version of the API. Cache keys are controlled by the URI and any change in the URI path with a different version invalidates
the cache keys. This also enables clients to cache URI resources without worrying about the impact of newer version of API's.