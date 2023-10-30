# PROJECT - communication with external services ( RFC )
**Authors:** I D, K Z, N B etc...
**Approvers:** TBD;
**Informed:**  TBD;

## Impact: No impact on end-users  

## Target users: 
- PROJECT developers
- data-pipeline developers
- metadata management service

### Problem statement:

PROJECT is a platform which is implemented by using Microservices architecture
and it's a common task for a microservice to send a request to various internal and external services. 
In order to establish the communication every service have to deal with:

- APIs / contracts / protocols
- authentication
- storing credentials / encryption ( system accounts )
- retry logic
- response validation
- slowness ( caching strategy ) 
- network access
- communication monitoring / logging
- proxy configuration ( this part is tricky )
- prevent a network or service failure from cascading to other services ( circuit breaker pattern )

List of services of interest: 
A LIST OF INTERNAL SERVICES
the list is not limited by mentioned services and could be extended 

### Suggestion:

To consolidate the logic responsible for communication with external services into AWS Lambda functions.
Thus a service client should talk to an external service via a proxy lambda function.

#### Pros:

- offloads external service communication logic from every service
- secrets and sys_accounts are not spread across all services
- in most cases PROJECT platform failures resulted from downstream services unreliability
- endpoints protected by OAuth2.0 or public
- offloads authn / authz overkill from data-pipeline developers / PROJECT SDK users
- enables PROJECT platform healthcheck/availability dashboard

#### Cons:
- to come up with;
- to ask approvers to create a list of potential disadvantages in case of proposal reject;


## Conceptual diagram
![image info](./communication_with_external_services.conceptual_diagram.png)

- the validation lambda firstly is trying to get required data from Elasticsearch cluster managed by PROJECT

## Data-flow diagram
![image info](./communication_with_external_services.data_flow_diagram.png)

### API specification:

_DRAFT_V.0.1

#### Proxy request

`PROJECT:/api/v3/externalServices/:serviceId/proxyRequest, GET / POST`

request payload: 
```
{
  "path": "string"
  "body_payload": {}
}    
```

serviceId - is it the service Appstore Id?  
path - the target service path  
body_payload - the final request body 

#### Validation request

`PROJECT:/api/v3/externalServices/:serviceId/validationRequest, GET / POST`

request payload: 
```
{
  "action": "string"
  "body_payload": {}
}    
```

response: 
```
depends on selected action for "isUserExists" action, returns
{
  "user": "USER_ID",
  "isUserExists": false
}
```

serviceId - is it the service Appstore Id?  
action - pre-registered particular action to perform upon external request, for instance:  
"isUserExist" - which constructs a specific query to external service and format and validate response. 
body_payload - the final request body 

notes: 
- action name matches to Metadata Service Term attribute "valueValidator"
todo: add reference to the property link once it's available in the Monorepo.

#### Healthcheck request

`PROJECT:/api/v3/externalServices/:serviceId/healthcheckRequest, GET / POST`

request payload: 
```
{
  "empty body"
}    
```
serviceId - is it the service Appstore Id?  
expected 200 status code 

## Implementation notes:

- API Gateway already is part of PROJECT infrastructure
- All proposed endpoints already exist in various implementation forms in both PROJECT.
