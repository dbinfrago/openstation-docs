# Changelog

## 2025, August

- Initial go-live with NeTEx dataset

## 2025, November

- Initial release of OpenStation's SIRI FM ("facility monitoring") endpoint, containing status information for all monitorable lift equipments (elevators), updated once per minute
- We removed incorrect DHID entries from several stations, platforms and quays. Temporary IDs (as described in the NeTEx `Codespace` object) are used for the time being, while we work with local transit authorities to issue missing DHIDs.

## 2026, January

- Added station category and price category to `StopPlace`

## 2026, June

- Added organizational structure (`Department`, `OrganisationalUnit`) for [DB Regio-Netze Infrastruktur](https://www.db-regionetz.de/infrastruktur), a subsidiary of DB InfraGO
- Each `StopPlace` now carries a `TopographicPlaceRef` identifying its municipality via the German *Amtlicher Gemeindeschlüssel* (AGS, e.g. `ags:09771146` for Mering). The references are unresolved (no `TopographicPlace` entities included), as DB InfraGO does not own this data; the `ags` codespace declared at the top of the dataset documents the identifier scheme.
- Upgraded to NeTEx v2.0 (non-breaking). Along with this:
  - Station names now support multiple languages via `Text` sub-elements inside `Name` (initial rollout: regional minority languages where applicable, e.g. Lower Sorbian / `dsb` for Cottbus Hbf).
  - For backwards compatibility, `Name`'s inline text content is preserved alongside the new `Text` sub-elements during a transition period, but it will be removed on 2027-04-01 (see breaking change announcement below). Consumers should start reading names from the `Text` sub-elements now.
  - Announced upcoming breaking changes — consolidation of identifiers into the new `privateCodes` element, and removal of inline text from `Name` — which will take effect on 2027-04-01. See the [breaking change announcement](https://github.com/dbinfrago/openstation-docs/issues/2) for details.
