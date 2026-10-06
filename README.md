# openstation-docs

<img src="./dbinfrago.png" alt="DB InfraGO's company logo" height="50">

**OpenStation** is [DB InfraGO](https://www.dbinfrago.com/)'s single source of truth for open data on passenger station infrastructure in Germany. It provides data in accordance with European standards ([NeTEx](https://netex-cen.eu), [SIRI](https://siri-cen.eu)), and represents our so-called Inventory of Assets as required by the EU's [TSI-PRM regulation](https://www.era.europa.eu/domains/technical-specifications-interoperability/persons-disabilities-and-reduced-mobility-tsi_en) on the accessibility of passenger rail infrastructure.

This documentation provides an overview of the data model used in the API as well as the technical way(s) to obtain OpenStation data. Furthermore, it outlines our approach to API endpoint stability and breaking changes. Please read these notes carefully if you plan to use our API in a production system.

## Contents

- [Who is this for?](#who-is-this-for)
- [Data model](#data-model)
- [Obtaining OpenStation data](#obtaining-openstation-data)
- [Stability and breaking changes](#stability-and-breaking-changes)
- [License](#license)
- [Contributing](#contributing)

## Who is this for?

The primary audience for OpenStation are developers of **multimodal travel information systems (MMTIS)**, journey planners, and similar data-aggregation backends — for example open-source projects like [OpenTripPlanner](https://www.opentripplanner.org) or [MOTIS](https://motis-project.de) (the latter also powering data-aggregation projects like [Transitous](https://transitous.org)), as well as in-house equivalents at other operators and authorities.

This focus is deliberate. OpenStation is, by design, scoped strictly to the infrastructure operated by DB InfraGO. Any system that produces an end-user experience — a routing app, a station-information frontend, an accessibility-routing tool — typically needs to combine our data with data from other operators (stops, schedules, realtime, accessibility) to be genuinely useful. That combination is the job of an MMTIS, not of an infrastructure operator's open-data API.

Concretely:

- **If you are building or operating an MMTIS / journey planner**, you are our primary target audience. OpenStation delivers data in the shape you'll want to ingest it: standards-compliant NeTEx and SIRI, bulk delivery via Mobilithek with stable URLs and long change-management lead times, and increasingly detailed accessibility and indoor-graph data over time.
- **If you are building an end-user application**, OpenStation on its own will likely not be enough. You will want a multimodal data product that aggregates infrastructure, schedule, and realtime data across operators. Several such products exist — both open-source (the projects mentioned above) and commercial (including offerings from DB itself, such as the **RIS:: API family**). You can still consume OpenStation directly, but expect significant integration work to reach a passenger-facing experience.

This is also why OpenStation does not currently offer conveniences typical of consumer-facing APIs (per-resource JSON lookups, routing primitives, and similar). They would obscure the boundary between an infrastructure publisher and a journey-planning service, and would not actually serve the audience we are building for.

## Data model

### Introduction

OpenStation's data model is based on the European [NeTEx](https://netex-cen.eu) and [SIRI](https://siri-cen.eu) standards. These two norms are part of the so-called [Transmodel](https://transmodel-cen.eu) standards family. The [Transmodel](https://transmodel-cen.eu) norm defines a common vocabulary for the public transportation sector (for example: it is specified that a public transport station should be called a `Stop Place` or that an elevator is called a `Lift Equipment`). Based on this common language, several standards are derived for specific use cases. For our purposes, these include:

- [NeTEx](https://netex-cen.eu), for the exchange of static/scheduled data
- [SIRI](https://siri-cen.eu), for the exchange of dynamic/realtime data

In the scope of OpenStation, which contains passenger station infrastructure data, this translates into the following:

- **OpenStation NeTEx** contains data on our physical station infrastructure (platforms, access spaces, equipments, etc.) as well as DB InfraGO's organizational model, all of which change quite rarely and with a long lead time
- **OpenStation SIRI** contains realtime status data (e.g. indicating that a lift is currently out of service), which can change very frequently

Our NeTEx and SIRI datasets are intended to be combined by the consuming application into a full model of our infrastructure with realtime status updates, which would allow implementation of use cases such as realtime barrier-free routing.

### Data formats

Both NeTEx and SIRI are XML-based standards. For SIRI, an additional JSON variant ("SIRI Lite") has been specified. For NeTEx however, XML is still the only serialization available.

For this reason, as of its initial production release, OpenStation only delivers XML data. This also applies for SIRI, even though a JSON variant would be feasible, because most SIRI users will also need to use NeTEx and therefore be required to handle XML data anyway.

However, we are aware of the ergonomic upsides which a JSON variant would bring, especially for users switching from our old JSON-based StaDa and FaSta APIs, for which the usability of our new OpenStation API might feel like a downgrade at first.

For this reason, we are working on a draft for a JSON variant of NeTEx, which we hope to discuss and implement in late 2026 or early 2027, alongside a JSON-based SIRI delivery. We will update you via our [communication channels](#communication-channels) once such an implementation is available.

### Our data model in detail

**[→ Our NeTEx data model](./NeTEx.md)**

_**→ Our SIRI data model** – coming soon_

## Obtaining OpenStation data

OpenStation data can be obtained in two ways:

1. Via Germany's [Mobilithek](https://mobilithek.info), the country's [national access point](https://transport.ec.europa.eu/transport-themes/smart-mobility/road/its-directive-and-action-plan/national-access-points_en) for open transportation data as mandated by EU legislation (see details [below](#mobilithek))
2. Via [DB's API Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/) (see details [below](#db-api-marketplace))

Both of these sources offer advantages and disadvantages, which are documented below. **However, TL;DR: For production deployments, we strongly recommend obtaining OpenStation data via Mobilithek.**

|                 | Mobilithek | DB API Marketplace |
|-----------------|------------|--------------------|
|**Advantages**   | No API credentials required, no request limits, high performance, bulk output, supports compression, longer stability guarantees (see section on breaking changes below) | Filter parameters (e.g. filtering the output by station id), pagination |
|**Disadvantages**| No filter parameters (just one plain XML dataset containing everything) | API credentials required, request limits, no bulk output (pagination required), no compression |

We intend DB API Marketplace with its additional filter parameters to be used for smaller, exploratory projects, while all other systems, especially large production deployments, should retrieve their data via the Mobilithek.

### Mobilithek

Obtaining data through Mobilithek is our recommended approach. You won't need any credentials (even though creating a dedicated subscription with credentials is possible and can be used to receive updates in case of breaking changes, see also: [Stability and breaking changes](#stability-and-breaking-changes)).

#### NeTEx

##### Manual access

**You can download the dataset manually from your web browser [here](https://mobilithek.info/offers/879076212433727488).**

##### Programmatic access

You can obtain the OpenStation NeTEx dataset as follows:

```sh
curl --compressed 'https://mobilithek.info/mdp-api/mdp-conn-server/v1/publication/879076212433727488/file/noauth' > netex.xml
```

As you might have noticed, this request contains a dataset ID assigned by Mobilithek. That ID is stable for now, but if we, for some unforeseen reason, have to re-register our dataset at any point in the future, the ID might change.

We therefore offer the following additional endpoint, which will always redirect to our latest Mobilithek dataset. We strongly recommend using it:

```sh
curl -L --compressed 'https://bahnhof.de/daten/netex' > netex.xml
```

Please note that the library you use to send the request must support HTTP redirects (in case of `curl`, this is achieved by supplying the `-L` option).

#### SIRI

##### Manual access

**You can download the dataset manually from your web browser [here](https://mobilithek.info/offers/930558234532405248).**

##### Programmatic access

You can obtain the OpenStation SIRI FM dataset as follows:

```sh
curl --compressed 'https://mobilithek.info/mdp-api/mdp-conn-server/v1/publication/930558234532405248/file/noauth' > siri-fm.xml
```

As you might have noticed, this request contains a dataset ID assigned by Mobilithek. That ID is stable for now, but if we, for some unforeseen reason, have to re-register our dataset at any point in the future, the ID might change.

We therefore offer the following additional endpoint, which will always redirect to our latest Mobilithek dataset. We strongly recommend using it:

```sh
curl -L --compressed 'https://bahnhof.de/daten/siri-lite/fm.xml' > siri-fm.xml
```

Please note that the library you use to send the request must support HTTP redirects (in case of `curl`, this is achieved by supplying the `-L` option).

### DB API Marketplace

#### Prerequisites

To obtain OpenStation data through DB API Marketplace, you first need to complete the following steps:

1. Create a new user account [here](https://developers.deutschebahn.com/db-api-marketplace/apis/user/login)
2. Create a new application [here](https://developers.deutschebahn.com/db-api-marketplace/apis/application)
3. Store the credentials (`Client ID`, `Client Secret`) you received when creating the application. You will need them to send requests. You can always (re-)generate additional credentials by navigating to _Anwendungen → YOUR_APPLICATION_.
4. In the [catalog](https://developers.deutschebahn.com/db-api-marketplace/apis/product), search for OpenStation and follow the steps to add a subscription for the application you created before.

#### NeTEx

Given your `Client ID` and `Client Secret`, you can retrieve our OpenStation NeTEx dataset as follows:

```sh
curl 'https://apis.deutschebahn.com/db-api-marketplace/apis/open-station/v1/netex' \
-H 'DB-Client-ID: <YOUR_CLIENT_ID>' \
-H 'DB-Api-Key: <YOUR_CLIENT_SECRET>' > netex.xml
```

#### SIRI

Given your `Client ID` and `Client Secret`, you can retrieve our OpenStation SIRI FM ("facility monitoring") dataset as follows:

```sh
curl 'https://apis.deutschebahn.com/db-api-marketplace/apis/open-station/v1/siri-lite/fm.xml' \
-H 'DB-Client-ID: <YOUR_CLIENT_ID>' \
-H 'DB-Api-Key: <YOUR_CLIENT_SECRET>' > siri-fm.xml
```

## Stability and breaking changes

We make every effort to avoid breaking changes in the API that would require adaptation of code on the consumer side. Nevertheless, situations may arise that necessitate the implementation of such a change. We would therefore like to determine from the outset how and in what time frame we will announce and implement breaking changes should they become necessary.

### Communication channels

We will announce breaking changes (as well as relevant new features) via the following channels:

1. **Email to all subscribers to our dataset in the Mobilithek.** Even though you can consume our dataset without subscribing to it, or consume it via the DB API Marketplace, we strongly recommend you create a subscription in the Mobilithek nonetheless, as that allows us to notify you via email in case of a change.
2. **Issue in this GitHub repository.** For everyone who can not or does not want to create subscriptions in the Mobilithek, we will also create issues to track breaking changes in this GitHub repository. You can receive notifications for these by [watching this repo](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/about-notifications#subscription-options). Please note, however, that you will receive notifications for all issues in this repository, not only those related to breaking changes.

Furthermore, all significant changes to the API will be documented in the [changelog](./CHANGELOG.md).

### Lead times for breaking changes

Depending on the type of breaking change and the data portal you choose, these are the lead times from announcement to implementation:

|                    | Changes to the data model\* | Changes to the API / Transport Layer\*\* |
|--------------------|--|--|
| Mobilithek         | 9 months | 6 months |
| DB API Marketplace | 9 months | 3 months |

\*e.g.: removing entity types \
\*\*e.g.: renaming endpoints

## License

Data in the API as well as the API documentation is released to the public domain (under the [CC0 “license”](https://creativecommons.org/publicdomain/zero/1.0/)). Source code for any supporting programmes in this repository is released under the [APACHE 2.0 license](https://www.apache.org/licenses/LICENSE-2.0).

## Contributing

If you have problems using the API, would like to point out errors or inconsistencies, or have any other feedback to share, please feel free to [open an issue](https://github.com/dbinfrago/openstation-docs/issues) or contact us [by email](mailto:opendata.infrago@deutschebahn.com).

You're also invited to contribute directly by creating a [pull request](https://github.com/dbinfrago/openstation-docs/pulls).

_Note that, by participating in this project, you commit to the [code of conduct](CODE-OF-CONDUCT.md), and agree to release all of your contributions to the public domain (under the [CC0 “license”](https://creativecommons.org/publicdomain/zero/1.0/), for changes to the API documentation), and under the [APACHE 2.0 license](https://www.apache.org/licenses/LICENSE-2.0) (for code of any supporting programs in this repository), respectively._

## AI notice

Parts of this documentation were translated and/or linguistically improved using large language models. All outputs were manually reviewed for correctness by humans.
