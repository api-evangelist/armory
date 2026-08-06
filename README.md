# Armory

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Armory, Inc. is a San Mateo, California software company founded in 2016 that built and sold an
enterprise distribution of the open source Spinnaker continuous delivery platform. Its product line
covered Armory Continuous Deployment (self-hosted and Armory-managed Spinnaker), Armory Continuous
Deployment-as-a-Service, and a set of proprietary Spinnaker plugins — the Armory Scale Agent for
Spinnaker and Kubernetes, Pipelines-as-Code (Dinghy), an OPA-backed Policy Engine, Terraform
Integration, GitHub Integration and AWS Event Cache.

Harness acquired Armory's continuous delivery assets in January 2024. `www.armory.io` now 301s to
`harness.io/products/continuous-delivery` and the `armory.io` apex no longer completes a TLS
handshake, but the product did not stop: **docs.armory.io remains live**, Armory CD releases were
still shipping as recently as **v2.40.2 on 2026-07-27**, and Armory still publishes a real Swagger
2.0 API reference for the Scale Agent.

## API surface

- **Armory Scale Agent API** — the REST surface Clouddriver exposes when the Armory Scale Agent
  plugin is installed. **51 paths, 56 operations**, published as Swagger 2.0 at
  [`/reference/scale-agent/swagger.json`](https://docs.armory.io/reference/scale-agent/swagger.json).
  Self-hosted only — served by the operator's own Clouddriver, so there is no vendor base URL.
- **Armory Continuous Deployment API** — the Spinnaker Gate API, which Armory documents exposing for
  automation clients on a second port behind x509 client certificates.

## What is gone

`developer.armory.io` and `api.cloud.armory.io` — the Armory CD-as-a-Service developer portal and API
host — no longer resolve. There is no status page, no `/.well-known/` surface, no security.txt, no
trust center, no MCP server and no agent card on any Armory host.

- Docs: https://docs.armory.io/
- Source: https://github.com/armory · https://github.com/armory-io
- Containers: https://hub.docker.com/u/armory (219 repositories)
- Secondary-market listing: https://forgeglobal.com/armory_stock/
