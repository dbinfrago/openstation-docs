# openstation-docs

**OpenStation** is [DB InfraGO](https://www.dbinfrago.com/)'s single source of truth for open data on passenger station infrastructure in Germany. It provides data in accordance with European standards ([NeTEx](https://netex-cen.eu), [SIRI](https://siri-cen.eu)), and represents our so-called Inventory of Assets as required by the EU's [TSI-PRM regulation](https://www.era.europa.eu/domains/technical-specifications-interoperability/persons-disabilities-and-reduced-mobility-tsi_en) on the accessibility on passenger rail infrastructure.

This documentation provides an introduction to the two ways of obtaining _OpenStation_ data and presents a brief overview of the data model used in the API. Furthermore, it outlines our approach to API endpoint stability and breaking changes. Please read that section carefully if you plan to use our API in a production system.

## Contents

- [Beta status](#beta-status)
- [Obtaining _OpenStation_ data](#obtaining-openstation-data)
- [Data model](#data-model)
- [Stability and Breaking Changes](#stability-and-breaking-changes)
- [License](#license)
- [Contributing](#contributing)

## Beta status

A note before we dive into the details: _OpenStation_ is currently in an open beta phase, which is – as of October 2025 – planned to last until December 2025. This means that some elements of the “target data model” are not yet available at all stations. We are working on collecting that data in a timely manner.

## Obtaining _OpenStation_ data

_OpenStation_ data can be obtained in two ways:

1. Via Germany's [Mobilithek](https://mobilithek.info) (the country's [national access point](https://transport.ec.europa.eu/transport-themes/smart-mobility/road/its-directive-and-action-plan/national-access-points_en) for open transportation data as mandated by EU legislation) (see details [below](#mobilithek))
2. Via [DB's API Marketplace](https://developers.deutschebahn.com/db-api-marketplace/apis/) (see details [below](#db-api-marketplace))

Both of these sources offer advantages and disadvantages, which are documented below. **However, TL;DR: For a permanent productive connection, we strongly recommend obtaining _OpenStation_ data via the Mobilithek.**

|                 | Mobilithek | DB API Marketplace |
|-----------------|------------|--------------------|
|**Advantages**   | No API credentials required, no request limits, high performance, bulk output, supports compression, longer stability guarantees (see section on breaking changes below) | Filter parameters (e.g. filtering the output by station id), pagination, already includes SIRI data |
|**Disadvantages**| No SIRI data yet, no filter parameters (just one plain XML dataset containing everything) | API credentials required, request limits, no bulk output (pagination required), no compression |

We intend DB API Marketplace with its additional filter parameters to be used for smaller, exploratory projects, while all other systems, especially large production deployments, should retrieve their data via the Mobilithek.

### Mobilithek

Obtaining data through Mobilithek is our recommended approach. You won't need any credentials (even though creating a dedicated subscription with credentials is possible and can be used to receive updates in case of breaking changes, see also: [Stability and Breaking Changes](#stability-and-breaking-changes)).

#### NeTEx

##### Manual access

**You can download the dataset manually from your web browser [here](https://mobilithek.info/offers/879076212433727488).**

##### Programmatic access

You can obtain the _OpenStation_ NeTEx dataset as follows:

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

As of 2025-11-26, we are still working on bringing our SIRI FM ("facility monitoring") data to Mobilithek. We estimate that data will be available there starting early December 2025. Documentation for accessing the data will be added here once the data is available.

### DB API Marketplace

#### Prerequisites

To obtain _OpenStation_ data through the DB API Marketplace, you first need to complete the following steps:

1. Create a new user account [here](https://developers.deutschebahn.com/db-api-marketplace/apis/user/login)
2. Create a new application [here](https://developers.deutschebahn.com/db-api-marketplace/apis/application)
3. Store the credentials (`Client ID`, `Client Secret`) you received when creating the application. You will need them to send requests. You can always (re-)generate additional credentials by navigating to _Anwendungen → YOUR_APPLICATION_.
4. In the [catalog](https://developers.deutschebahn.com/db-api-marketplace/apis/product), search for _OpenStation_ and follow the steps to add a subscription for the application you created before.

#### NeTEx

Given your `Client ID` and `Client Secret`, you can retrieve our _OpenStation_ NeTEx dataset as follows:

```sh
curl 'https://apis.deutschebahn.com/db-api-marketplace/apis/open-station/v1/netex' \
-H 'DB-Client-ID: <YOUR_CLIENT_ID>' \
-H 'DB-Api-Key: <YOUR_CLIENT_SECRET>' > netex.xml
```

#### SIRI

Given your `Client ID` and `Client Secret`, you can retrieve our _OpenStation_ SIRI FM ("facility monitoring") dataset as follows:

```sh
curl 'https://apis.deutschebahn.com/db-api-marketplace/apis/open-station/v1/siri-lite/fm.xml' \
-H 'DB-Client-ID: <YOUR_CLIENT_ID>' \
-H 'DB-Api-Key: <YOUR_CLIENT_SECRET>' > siri-fm.xml
```

## Data model

_Coming soon._

## Stability and Breaking Changes

We make every effort to avoid breaking changes in the API that would require adaptation of code on the consumer side. Nevertheless, situations may arise that necessitate the implementation of such a change. We would therefore like to determine from the outset how and in what time frame we will announce and implement breaking changes should they become necessary.

### Communication channels

We will announce breaking changes (as well as relevant new features) via the following channels:

1. **Email to all subscribers to our dataset in the Mobilithek.** Even though you can consume our dataset without subscribing to it, or consume it via the DB API Marketplace, we strongly recommend you create a subscription in the Mobilithek nonetheless, as that allows us to notify you via email in case of a change.
2. **Issue in this GitHub repository.** For everyone who can not or does not want to create subscriptions in the Mobilithek, we will also create issues to track breaking changes in this GitHub repository. You can receive notifications for these by [watching this repo](https://docs.github.com/en/account-and-profile/managing-subscriptions-and-notifications-on-github/setting-up-notifications/about-notifications#subscription-options). Please note however, that you will receive notifications for all issues in this repository, not only those related to breaking changes.

Furthermore, all significant changes to the API will be documented in the [changelog](./CHANGELOG.md).

### Lead times for Breaking Changes

Depending on the type of breaking change and the data portal you choose, these are the lead times from announcement to implementation:

**Important: These lead times only apply after the API's beta phase has ended. Until then, we might implement breaking changes on shorter notice.**

|                    | Changes to the data model | Changes to the API / Transport Layer |
|--------------------|--|--|
| Mobilithek         | 9 months | 6 months |
| DB API Marketplace | 9 months | 3 months |

## License

Data in the API as well as the API documentation is released to the public domain (under the [CC0 “license”](https://creativecommons.org/publicdomain/zero/1.0/)). Source code for any supporting programmes in this repository is released under the [APACHE 2.0 license](https://www.apache.org/licenses/LICENSE-2.0).

## Contributing

If you have problems using the API, would like to point out errors or inconsistencies, or have any other feedback to share, please feel free to [open an issue](https://github.com/dbinfrago/openstation-docs/issues) or contact us [by email](mailto:opendata.infrago@deutschebahn.com).

You're also invited to contribute directly by creating a [pull request](https://github.com/dbinfrago/openstation-docs/pulls).

_Note that, by participating in this project, you commit to the [code of conduct](CODE-OF-CONDUCT.md), and agree to the release all of your contributions to the public domain (under the [CC0 “license”](https://creativecommons.org/publicdomain/zero/1.0/), for changes to the API documentation), and under the [APACHE 2.0 license](https://www.apache.org/licenses/LICENSE-2.0) (for code of any supporting programs in this repository), respectively._
