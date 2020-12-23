User REST Endpoints
==========================
User specific end points are similar to infrastructure end points, but differ in a way role based access control(RBAC) is enforced. The end points assume the user context and allow
operations that an user can perform based on his role and responsibilities. This helps to enforce data security and integrity while exposing the business logic via public REST end points.

## Supported mechanism
Bearer authentication is the only mechanism supported. It requires the user to acquire a token by supplying user credentials for login. Post successful login, a token is issued, which is used for further
API calls.

## Defining API's in Modeler
BizAPP Modeler allows you to define API's using the industry standard best practices and strategies. API's are defined at an application level and thus helps to logically group function specific 
API's and enforce access control on top of it.

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

#### Adding a new endpoint version
To add a new endpoint version in BizAPP Modeler, navigate to an application and perform the below steps.
* Right click on APIs node and click on **Add EndpointVersion**.
* Enter the name in the popup window. If you want to name the versions using different naming strategy, input the same, Otherwise keep the version names same as real versions. For e.g. v1.

![new endpoint version](../images/new-endpoint-version.png)

* Click OK to create a new version.
* A new window appears on the right side with a new version information.
* Enter **API Route Template** value in the text box. This is mandatory to indicate the version in URI Path.

![endpoint version route](../images/endpoint-version-route.png)

### API Endpoints

New API endpoints can be created under a specific version. To create an endpoint, right click on the API version and add New APIEndPoint.
Below image shows a sample endpoint created.

![new-endpoint](../images/endpoint-http-methods.png)

* **API Route Template** is mandatory to set the route for this API.Route needs to be unique across all API's defined under a version.
* Supported **HttpMethod** verbs are GET,POST,PUT and DELETE.
* Supported **Handler Types** are QueryResult, Method and Method Snippet.

Every endpoint defined in the modeler is transformed into equivalent executable code on the fly, thus eliminating the need for compilation and deployment complexities. It applies to 
all **HandlerType** modes supported.

#### API using query result

In most of the cases, an API is created to expose information from the queries executed internally. The results from the query is concise and can be exposed as is to the end users without going through any data obfuscation. In such cases, BizAPP Modeler makes it easy for developers to expose the queries that are already defined in the solution.

To create a new Endpoint using a query,
* Create a new API Endpoint.
* Select **handlerType** as Query.
* Select a query by clicking on ... button.

![endpoint-using-query](../images/endpoint-using-query.png)

#### API using code (Method and MethodSnippet)

For advanced developers that want to make use of ASP.NET Core features to define methods and decorate them, both **Method** and **MethodSnippet** provide an opportunity to control 
request and response parameters. 

As the name suggests **Method** includes a complete signature of the method that would be exposed as an Endpoint. **MethodSnippet** only has the body of the code excluding the method definition.

**Anatomy of an API Method**

![new-endpoint-method](../images/new-endpoint.png)







