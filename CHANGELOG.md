# JFrog Log Analytics Changelog

All changes to the New Relic log analytics integration will be documented in this file.

## [0.0.7] - July 16, 2024
* Fluentd sidecar version bumped to 4.5, to upgrade base image to bitnami/fluentd 1.17.0
* Fixing fluent-plugin-jfrog-metrics issue (upgrading to 0.2.7) - resolving PTRENG-6234
* Fixing logs and metrics default New Relic uri in helm value files

## [0.0.6] - June 6, 2024
* [BREAKING] Adding deprecation notice for partnership-pts-observability.jfrog.io docker registry
* FluentD sidecar version bumped to 4.3, to upgrade base image to bitnami/fluentd 1.16.5
* Update FluentD sidecar helm charts to match recent changes in JFrog's official charts

## [0.5.0] - April 19, 2024

* Fix fluentd regex to fetch correctly docker image and repo names
* Make New Relic's SaaS logs and metrics endpoints configurable
* Fix order of request and response content length to match spec

## [0.4.0] - April 12, 2024

* Bump fluentd sidecar image to 4.2
* Revamp Readme's Artifactory and Xray installation instructions

## [0.3.0] - April 11, 2024

* Fix Artifactory access's regex to match log input changes

## [0.2.0] - Jan 09, 2024

* Supporting only OS/VM, Docker and k8s installation types
* Adding .env files instead of setting/filling variables in fluentd config
* Adding jfrog and heap callhome in fluentd config
* Supporting only Artifactory and Xray Fluentd config
* Removing support for JFrog APIKeys

## [0.1.0] - Dec 07, 2022

* Initial release of New Relic Jfrog Logs Analytic integration
