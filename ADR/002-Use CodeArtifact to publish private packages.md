# Use CodeArtifact to publish private packages for generated code

## Status

Accepted

## Context

Generated code, like API clients, must be versioned and archived, and may be consumed by different packages.
Using npm private packages is a preffered solution, because they allow all of the above, while publishing can be automated.

We explored the following solutions:

* _NPM private packages_ - requires to set up an organization subscription, where each developer on the team onboards the npm organization and logs into NPM to install private packages locally, as well as distributing secrets to both Github Actions and the AWS build environment.
* _Github Packages_ - Works out of box on both github actions and developer machines, but requires distributing secrets to AWS build environment manually. The secret is a legacy INDIVIDUAL_TOKEN and thus tied to a Github account. At minimum, this requires us to set up a "robot" member for our team. 
* _AWS CodeArtifact_ - Requires setting up permissions, so that developers may log in to CodeArtifact using AWS SSO, an AWS OIDC provider for Github Actions, and configuring permissions for build environemnts.

## Decision

We use AWS CodeArtifact, because 
* The INDIVIDUAL_TOKEN approach with Github requires storing a github secret in AWS manually
* NPM private packages requires every developer to onboard with npm and our npm organisation.
* While setting up permissions is work, the setup can be replicated trough CDK and only requires a one-time setup of the OIDC connection. 
* Developer ergonomics are acceptable, because logging in to CodeArtifact can automated on a dev enviroment.
* CodeArtifact pricing is a lot cheaper then Github or NPM...

## Consequences

We use CodeArtifact for private package repositories:

* Github actions to use CodeArtifact with OIDC
* Developers use CodeArtifact with AWS SSO
* All build tasks have READONLY access to CodeArtifact
