# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2024-08-31

### Added
- Initial release of the App of Apps Helm chart.

## [1.0.1] - 2024-09-08

### Added
- Support for multiple sources in Argo CD applications, allowing configurations from different repositories and Helm charts.
- Enhanced Helm value files handling, including support for referencing multiple `valueFiles` and using `ignoreMissingValueFiles` to prevent errors.
- Flexibility improvements: Added the ability to include additional application variables in the `values.yaml` and `Application` manifests.
- Expanded the example folder with a complete example for multi-source usage and configuration.
