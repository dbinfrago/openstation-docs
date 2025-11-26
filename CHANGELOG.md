# Changelog

## 2025-08-04

- Initial go-live with NeTEx dataset

## 2025-11-26

- Initial release of OpenStation's SIRI FM ("facility monitoring") endpoint, containing status information for all monitorable lift equipments (elevators), updated once per minute
- We removed incorrect DHID entries from several stations, platforms and quays. Temporary IDs (as described in the NeTEx `Codespace` object) are used for the time being, while we work with local transit authorities to issue missing DHIDs.
