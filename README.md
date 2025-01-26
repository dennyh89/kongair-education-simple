# Konnect Setup
Create a control plane named `training-kongair-simplified`.
Create a dataplane as advised by Konnect on your platform (e.g. docker on mac).
Wait for the Dataplane to connect.

Create a personal access token and copy it.

# Repo setup 

* fork this repo
* Settings --> Secrets and Variables --> Actions 
  * New repository secret --> create KONNECT_PAT=<your token>
  * New repository variable --> 
    * KONNECT_INTERNAL_CP=training-kongair-simplified
    * KONNECT_ADDR=https://<region>.api.konghq.com
* Settings --> Actions --> General --> Workflow permissions --> select `Read and write permissions` AND `Allow GitHub Actions to create and approve pull requests`

# Setup Services locally
* `podman compose -f docker-compose-arm-64.yaml up -d` (maybe need to change docker config.json credStore to osxkeychain)
* `docker-compose -f docker-compose-arm-64.yaml up -d` (maybe need to change docker config.json credStore to osxkeychain)

# Test services directly

User service is not protected, so we can query any user by adding the custom header.
```
curl -s -i localhost:5051/health
curl -s -i -H "x-consumer-username: jdoe" localhost:5051/customer | jq
curl -s -i -H "x-consumer-username: jsmith" localhost:5051/customer | jq

curl -s -i localhost:5052/health
curl -s -i localhost:5052/flights | jq
curl -s -i localhost:5052/flights/KA0292 | jq
```

# Trigger build and deployments

Go to the repository --> Actions and select e.g. "Build Customer API" and run the workflow via the "run workflow" button.
Go to the repository --> Actions and select e.g. "Deploy Customer API" and run the workflow via the "run workflow" button.

Repeat this for all other workflows.

# Test via Gateway

```
curl -s -i -H "apikey: dfreese-key" localhost:8000/customers/health
curl -s -H "apikey: dfreese-key" localhost:8000/customers/customer | jq

curl -s -i localhost:8000/flights/health
curl -s -i localhost:8000/flights/flights | jq
curl -s  localhost:8000/flights/flights/KA0292 | jq
curl -s  localhost:8000/flights/flights/KA0292/details | jq
```

# Commit a change to the repository and follow it through the workflows
* create a branch e.g. "feat/dev-customer-api"
* Change a plugin configuration of the Customer API
* commit it
* file a PR and approve/merge it
* watch the Build workflow run and how the PR to deployment is created with the Diff
* approve and merge the PR to main and watch the deployment to be done by the next workflow




######### additional info - may not apply for simplification #########
# KongAir

An example Kong application based on a fictitious airline, KongAir.

This project will aim to provide working examples of various Kong, Inc.
technolgies including [Kong Konnect](https://docs.konghq.com/konnect/)
and [APIOps](https://github.com/Kong/go-apiops).

The repository is modeled as a shared monorepo with various teams having
their code and configurations stored in folders at the top level.
For example, the fictitious "Flight Data" team stores their projects in the [flight-data](flight-data/) subfolder.

Also modeled here is a simulated central governance team, named [platform](platform/).
This team mimics the typical responsibility of a central team that manages shared infastructure, like API Gateways.

Automated APIOps processes are exemplified in [GitHub Actions](.github/workflows) using the Kong APIOps capabilities.

## Teams

* The [flight-data](flight-data/) team owns public facing APIs that serve KongAir's flight data information services
including the [routes](flight-data/routes/) and [flights](flight-data/flights/) services.
* The [sales](sales/) engineering team owns services that service customer needs. The [customer](/sales/customer/)
service hosts customer information including payment methods, frequent flyer information, etc...
The [bookings](/sales/bookings/) service manages customer flight bookings and depends on the public flight-data team
services.
* The [experience](experience/) team uses GraphQL and builds "experience" APIs to drive applications. The experience
APIs aggregate the other KongAir REST APIs to make a dynamic unified API for applications.


