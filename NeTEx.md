# OpenStation NeTEx

This document contains documentation on OpenStation's NeTEx data model. For a general introduction to NeTEx and SIRI, our SIRI data model, information on API access, licensing etc., please refer to the [main readme](README.md).

## Contents

- [Introduction](#introduction)
- [Entities](#entities)
  - [Metadata elements](#metadata-elements)
  - [Sites and site components ("locations")](#sites-and-site-components-locations)
  - [Accessibility assessment](#accessibility-assessment)
  - [Equipments and local services](#equipments-and-local-services)
  - [Indoor graph elements](#indoor-graph-elements)
  - [Organization](#organization)
- [Example dataset](#example-dataset)
- [Schema](#schema)

## Introduction

The NeTEx standard defines a large set of object types and attributes. For most use cases, only a small subset of types and attributes is relevant. For this reason, the core NeTEx standard does not specify which types and attributes must be included in a data delivery. Instead, sets of required types and attributes are defined on a per use-case basis in what are called *profiles*. To standardize the exchange of mobility data as required by the EU's [MMTIS regulation](https://transport.ec.europa.eu/transport-themes/smart-mobility/road/its-directive-and-action-plan/multimodal-travel-information_en), two European profiles were developed and made part of the extended NeTEx standard:

- **European Passenger Information Profile (EPIP, CEN/TS 16614-4:2017)**. This profile includes some basic rules for NeTEx delivery (e.g. namespaces for identifiers) as well as detailed specifications for the exchange of schedule data.
- **European Passenger Information Accessibility Profile (EPIAP, CEN/TS 16614-6:2024)**. This profile builds on the basic rules for NeTEx delivery defined in EPIP and adds detailed specifications for the exchange of accessibility data (both stations and vehicles).

OpenStation NeTEx is DB InfraGO's implementation of the **EPIAP** profile. We highly recommend reading the corresponding standards document (CEN/TS 16614-6:2024) to get a better understanding of NeTEx's data model for accessibility. (It should be noted, unfortunately, that NeTEx standards documents are currently not openly available and must be purchased through a local standards authority, e.g. DIN in Germany.)

**EPIAP** specifies several levels of detail for expressing infrastructure accessibility. Most simply, a valid EPIAP dataset just needs to include a list of platforms (*quays*) for each station (*stop place*), along some flags to indicate specific aspects of accessibility, the so-called *accessibility assessment*. Additionally, an EPIAP implementation can choose to provide a more detailed model of infrastructure at the *stop place*, including individual *equipments* such as elevators (*lift equipments*), and other relevant areas of the *stop place*, e.g. *access spaces* or *entrances*, as well as a graph model specifying how these *equipments* and *sites* are connected. For our OpenStation NeTEx implementation, we decided to follow this optional, extended approach, because we are convinced that it significantly increases the number of potential use cases for our data.

However, with the decision to use the extended stop place model, we were also faced with the challenge that some of the required objects and relationships were not modelled in our internal systems yet. For the initial release of our API in December 2025, we have expanded our systems so that they could structurally store these new data types, but in many cases the actual object/relationship instances are still to be recorded. The entity descriptions below therefore indicate which maintenance status to expect of each object type.

## Entities

These are the entities we deliver as part of the OpenStation NeTEx dataset.

### Metadata elements

NeTEx Entity | Description | Path
------ | ----------- | ----
`PublicationDelivery` | Root element of every NeTEx dataset, wraps everything else | `/PublicationDelivery`
`CompositeFrame` | Another required wrapper element. In NeTEx, data is delivered in so-called *frames*. Which *frame* to use for which element is in part specified by the NeTEx core, and in part by the profile (EPIAP in our case). If multiple types of *frames* are delivered, these need to be wrapped in a `CompositeFrame`. | `PublicationDelivery/` `dataObjects/` `CompositeFrame`
`SiteFrame` | "Main" element containing our station infrastructure. | `CompositeFrame/` `frames/` `SiteFrame`
`ResourceFrame` | Wrapper element containing auxiliary data, mainly DB InfraGO's organizational model and role assignments. | `CompositeFrame/` `frames/` `ResourceFrame`
`Codespace` | Codespaces are NeTEx's mechanism to ensure global uniqueness of identifiers even when several identifier schemes are used. To do this, every identifier scheme is assigned a unique Codespace, as defined by a prefix and a URL. The URL should be standardized for the given identifier scheme, meaning that all NeTEx datasets using this identifier scheme should specify the same codespace URL. The codespace prefix is local to the individual NeTEx document and represents a placeholder for the URL, to be used in the actual `id` or `ref` attributes of entities. For example: An entity with a UIC id `8000001` would have an `id` attribute `uic:8000001`, assuming that `uic` was chosen as the prefix for that scheme. When comparing identifiers of two NeTEx objects, it is therefore essential to resolve the prefix into the Codespace URL first. To reduce complexity for implementers, all relevant entities in the OpenStation NeTEx dataset include a key `NORMALIZED_ID_URI`, which contains the fully resolved id, including the codespace URL. <br><br> OpenStation uses several codespaces, each also containing a short description on its usage. | `PublicationDelivery/` `codespaces/` `Codespace`

### Sites and site components ("locations")

Household Term | NeTEx Entity | Description | Data Quality | Path
-------------- | ------------ | ----------- | ------------ | ----
Station | `StopPlace` | Each of our railway stations is represented by exactly one *stop place*. Please note: Because we are legally required to deliver accessibility assessments on the *stop place* level, we need to ensure that the *stop place* only contains our own infrastructure, and none by other operators, because we can only assess our own buildings. Since we use the DHID identifier scheme for *stop places*, we run into a problem: DHID stops are not guaranteed to be single-operator. Therefore, we cannot use them directly for our DB-InfraGO-only *stop places*. Our solution - at least for now - was to introduce a separate 'sub-level' DHID for our *stop place* and refer to the original DHID in the *parent stop place* attribute. | Mature data, please report any issues.\* | `SiteFrame/` `stopPlaces/` `StopPlace`
Platform | `Quay` (`QuayType` = `railIslandPlatform`) | Full platform (perron), usually with one or two platform edges. In NeTEx, both platforms and platform edges are called a *quay*. They can be differentiated using the `QuayType` attribute. Note that the term `railIslandPlatform` might be misleading. It only indicates that this *quay* is a platform, and not a platform edge. Therefore, a side platform would also have the type `railIslandPlatform`. | Mature data, please report any issues.\* | `StopPlace/` `quays/` `Quay`
Platform edge, track | `Quay` (`QuayType` = `railPlatform`) | Platform edge, usually identified by a number. In NeTEx, both platforms and platform edges are called a *quay*. They can be differentiated using the `QuayType` attribute. | Mature data, please report any issues.\* | `StopPlace/` `quays/` `Quay`
Platform section (height level) | `Quay` (`QuayType` = `railPlatformSector`) | Section of a platform edge. Note that we currently only use sections to indicate when different parts of a platform edge have varying heights. We do not publish named platform sections (A, B, C, …) yet, due to a lack of data. Note: Like platforms and platform edges, platform sectors are also called *quays*. They can be differentiated using the `QuayType` attribute. | Mature data, please report any issues.\* | `StopPlace/` `quays/` `Quay`
Access space, underpass, overpass | `AccessSpace` | All public areas of the *stop place* which are not platforms. Can be further differentiated using the `AccessSpaceType` attribute into concourses (`concourse`), underpasses (`underpass`), overpasses (`overpass`), generic access areas (`passage`) and forecourts (`forecourt`). | Newly published data. We are working on increasing data quality, but issues are to be expected for the time being. Common problems: Access spaces are internally not marked as public and therefore not included in the dataset; Access spaces are identified with a wrong type. | `StopPlace/` `accessSpaces/` `AccessSpace`

\*Please do not report missing attributes, we are working on filling in empty fields already. However, if you notice that an object is missing entirely, or that an existing value is incorrect, please reach out!

### Accessibility assessment

`AccessibilityAssessment` is the central structure for modeling accessibility of a `StopPlace` as a whole, as well as individual `Quay`s and `AccessSpace`s. Each `AccessibilityAssessment` carries a top-level summary flag (`MobilityImpairedAccess`) as well as an `AccessibilityLimitation` block with more detailed flags (e.g. `WheelchairAccess`, `StepFreeAccess`, …). These flags use the values `true`, `false`, or `unknown`.

Paths:
- `StopPlace/` `AccessibilityAssessment`
- `Quay/` `AccessibilityAssessment`
- `AccessSpace/` `AccessibilityAssessment`

### Equipments and local services

Household Term | NeTEx Entity | Description | Data Quality | Path
-------------- | ------------ | ----------- | ------------ | ----
Assistance Service | `AssistanceService` | Service which usually has to be booked for boarding which requires a boarding aid/hoist. | Mature data, please report any issues.\* | `StopPlace/` `localServices/` `AssistanceService`
Boarding aid, platform hoist | `AccessVehicleEquipment` | Equipment on the platform used for boarding, usually has to be booked in advance. | Mature data, please report any issues.\* | `StopPlace/` `placeEquipments/` `AccessVehicleEquipment` (full),<br> `Quay/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `AccessVehicleEquipmentRef` (reference)
Door | `EntranceEquipment` | Simple door, double door, or revolving door. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `EntranceEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `EntranceEquipmentRef` (reference)
Escalator | `EscalatorEquipment` | Escalator, not to be confused with a travelator. The latter does not have stairs. Realtime status monitoring is usually available via SIRI. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `EscalatorEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `EscalatorEquipmentRef` (reference)
Travelator | `TravelatorEquipment` | Travelator, known from airports and supermarkets. Not to be confused with an escalator. The latter does have stairs. Realtime status monitoring is usually available via SIRI. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `TravelatorEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `TravelatorEquipmentRef` (reference)
Lift, Elevator | `LiftEquipment` | Elevator. Realtime status monitoring is usually available via SIRI. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `LiftEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `LiftEquipmentRef` (reference)
Ramp | `RampEquipment` | Ramp. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `RampEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `RampEquipmentRef` (reference)
Staircase | `StaircaseEquipment` | Staircase. | Mature data, please report any issues.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `StaircaseEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `StaircaseEquipmentRef` (reference)
Information desk, DB Information | `PassengerInformationEquipment` (type = `epd:eu:TypeOfPassengerInformationEquipment:HelpDesk`) | DB Information desk. In NeTEx, information desks, audio announcement devices as well as visual departure monitors are all called *passenger information equipments*. They can be differentiated using the `TypeOfPassengerInformationEquipmentRef` attribute. | Newly published data, please report issues nonetheless.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. | `StopPlace/` `placeEquipments/` `PassengerInformationEquipment` (full),<br> `Quay/` or `AccessSpace/` `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `PassengerInformationEquipmentRef` (reference)
Visual departure monitor | `PassengerInformationEquipment` (type = `epd:eu:TypeOfPassengerInformationEquipment:DepMon`) | Visual departure monitor. In NeTEx, information desks, audio announcement devices as well as visual departure monitors are all called *passenger information equipments*. They can be differentiated using the `TypeOfPassengerInformationEquipmentRef` attribute. | Newly published data, please report issues nonetheless.\* | `StopPlace/` `placeEquipments/` `PassengerInformationEquipment` (full),<br> `Quay/`  `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `PassengerInformationEquipmentRef` (reference)
Speaker, audio announcements | `PassengerInformationEquipment` (type = `dbinfrago:eu:TypeOfPassengerInformationEquipment:DepAnnouncements`) | Speaker for audio announcements. In NeTEx, information desks, audio announcement devices as well as visual departure monitors are all called *passenger information equipments*. They can be differentiated using the `TypeOfPassengerInformationEquipmentRef` attribute. | Newly published data, please report issues nonetheless.\* | `StopPlace/` `placeEquipments/` `PassengerInformationEquipment` (full),<br> `Quay/`  `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `PassengerInformationEquipmentRef` (reference)
Toilet, WC, Restroom | `SanitaryEquipment` | Restroom. | Newly published data, please report issues nonetheless.\* Common problems: Sometimes, the access space where this equipment is located is erroneously not marked as public and therefore omitted from the dataset. In this case, the equipment is also not included. Also, sometimes the equipment is not yet localised to a specific access space at all. In this case, it is returned under the *stop place's* `unlocalisedEquipments`. | `StopPlace/` `placeEquipments/` or `unlocalisedEquipments/` `SanitaryEquipment` (full),<br> `Quay/` or `AccessSpace/`  `equipmentPlaces/` `EquipmentPlace/` `placeEquipments/` `SanitaryEquipmentRef` (reference)

\*Please do not report missing attributes, we are working on filling in empty fields already. However, if you notice that an object is missing entirely, or that an existing value is incorrect, please reach out!

### Indoor graph elements

The following entities allow construction of an indoor graph for calculation of accessible interchange routes. As mentioned in the introduction, we do not store these objects and/or relationships in our internal systems yet. For the initial release of OpenStation NeTEx in December 2025, we enabled our internal systems to store this information, but actual recording and maintenance of new data is still to be done. We hope to provide some fully modeled example stations by autumn 2026.

Household Term | NeTEx Entity | Description | Data Quality | Path
-------------- | ------------ | ----------- | ------------ | ----
Installation location | `EquipmentPlace` | Place at which an equipment is located. In NeTEx, the relationship between a place and an *equipment* is always achieved indirectly, through an *equipment place*. An equipment can have multiple equipment places, e.g. a staircase that connects two levels (would have two *equipment places*), or an elevator, which might connect to even more levels, and therefore have even more *equipment places*. Note that geocoordinates for equipments are also supplied through their equipment places rather than in the equipments directly. | Newly published and to-be-recorded data. See remarks above the table. | `Quay/` or `AccessSpace/`  `equipmentPlaces/` `EquipmentPlace`
Connection, graph edge | `SitePathLink` | Object indicating a connection between two places, to be used for constructing a routing graph of the station. | Newly published and to-be-recorded data. See remarks above the table. | `StopPlace/` `pathLinks/` `SitePathLink`
Level, story | `Level` | Object describing a vertical level/story of the station, used to group places (`Quay`s, `AccessSpace`s) into reference planes for the indoor routing graph. A level is a *topological* reference plane, **not** a measure of physical height or of the effort needed to traverse between places: height differences, gradients and step counts are carried by `SitePathLink`s (and the connecting equipments), and must not be inferred from level values. Consequently, the presence of a `RampEquipment` or `StaircaseEquipment` does **not** by itself imply a level change — at an at-grade station, platforms, access spaces and the forecourt typically share a single ground level even where a ramp or a few steps bridge the platform height. We assign distinct levels where circulation surfaces are genuinely stacked, i.e. where the 2D position alone cannot tell you which surface you are on (e.g. an underpass below the platforms is a different level from the platforms above it). Note that a level does not need to be contiguous (e.g. in stations with multiple platforms, the platforms are often on the same level, but not contiguous). | Newly published and to-be-recorded data. See remarks above the table. | `StopPlace/` `levels/` `Level` (full),<br> `Quay/` or `AccessSpace/`  `LevelRef` (reference)

### Organization

Organisational level | NeTEx Entity | Path
-------------------- | ------------ | ----
DB InfraGO AG | `Operator` | `ResourceFrame/` `organisations/` `Operator`
Regional subdivisions and subsidiaries (2nd level) | `Department` | `Organisation/` `parts/` `Department`
Station managements (3rd level) | `OrganisationalUnit` | `Organisation/` `parts/` `OrganisationalUnit`

The relationship between *organisational units* and the *stop places* they are responsible for is achieved via `ResponsibilitySet` and `ResponsibilityRoleAssignment` objects, which are also located in the *resource frame*.

## Example dataset

A snapshot of a single-station NeTEx delivery for Saarbrücken Hbf (June 2026) is available in [`examples/saarbruecken-hbf.netex.xml`](./examples/saarbruecken-hbf.netex.xml). It is not guaranteed to be in sync with the live API.

## Schema

Our dataset is published in **NeTEx v2.0**. The smallest available schema covering our delivery (NeTEx part 1) can be found [here](https://github.com/NeTEx-CEN/NeTEx/blob/master/xsd/netex_part_1/netex_all_frames_part1.xsd).

