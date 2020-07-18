---
layout: post
title: Read `exposure-notifications-server` (1/2)(unfinished)
date: 2020-05-17 09:42
author: m0ssc0de
tags: [golang, google, apple, project, code, management]
summary: 
---
Google and Apple release a project to fight the virus, named [exposure-notifications-server](https://github.com/google/exposure-notifications-server). Maybe this is a chance to learn how to make `privacy protection`, manage projects, and write code. It should be better and my honor, if I can make a contribution to this open project for the fighting.

By fast reading the [functional requirements](https://github.com/google/exposure-notifications-server/blob/master/docs/server_functional_requirements.md) of the project, it looks like designed for `positively diagnosed users`. The first model I recognized is the user generates a key and read an incremental file. But whats it for, and how does it work in detail? Just put these questions in far later. Let's get in project repo first.

Get a glance of the project root. The ✅ means will be described later.

```tree
# 4d51daff9aaab4e37782f6eadae5e8c0dddb5ae5
.
✅├── .github
✅│   └── ISSUE_TEMPLATE
✅│       ├── bug_report.md
✅│       └── feature_request.md
  ├── .gitignore
  ├── .prow
  │   └── images
  │       └── exposure-notifications-test
  │           ├── Dockerfile
  │           └── Makefile
  ├── CODE_OF_CONDUCT.md
  ├── CONTRIBUTING.md
  ├── LICENSE
  ├── OWNERS
  ├── README.md
  ├── _config.yml
  ├── builders
  │   ├── build.yaml
  │   ├── deploy.yaml
  │   └── schema.yaml
  ├── cmd
  │   ├── OWNERS
  │   ├── cleanup-export
  │   │   └── main.go
  │   ├── cleanup-exposure
  │   │   └── main.go
  │   ├── export
  │   │   └── main.go
  │   ├── exposure
  │   │   └── main.go
  │   ├── federationin
  │   │   └── main.go
  │   ├── federationout
  │   │   └── main.go
  │   └── monolith
  │       └── main.go
  ├── config
  │   └── 100-exposure-server.yaml
  ├── docker
  │   ├── Makefile
  │   ├── migrate.dockerfile
  │   └── protoc.dockerfile
  ├── docs
  │   ├── OWNERS
  │   ├── deploying.md
  │   ├── diagrams
  │   │   ├── compute_data_in.diagram
  │   │   ├── compute_data_in.svg
  │   │   ├── compute_data_out.diagram
  │   │   ├── compute_data_out.svg
  │   │   ├── google_cloud_run.diagram
  │   │   ├── google_cloud_run.svg
  │   │   ├── hybrid-in.diagram
  │   │   ├── hybrid-in.svg
  │   │   ├── hybrid-out.diagram
  │   │   ├── hybrid-out.svg
  │   │   ├── on_prem.diagram
  │   │   └── on_prem.svg
  │   ├── images
  │   │   ├── Server-Requirements-Diagram.png
  │   │   ├── data-ingres.png
  │   │   ├── data-ingress.svg
  │   │   ├── data-retrieval.svg
  │   │   ├── functional_diagram.svg
  │   │   ├── google_cloud_run.png
  │   │   ├── hybrid_in.png
  │   │   ├── hybrid_out.png
  │   │   └── on_prem.png
  │   ├── index.md
  │   ├── ios_device_check.md
  │   ├── server_deployment_options.md
  │   └── server_functional_requirements.md
  ├── examples
  │   └── export
  │       ├── README.md
  │       ├── public.pem
  │       └── sample-export.zip
  ├── go.mod
  ├── go.sum
  ├── internal
  │   ├── android
  │   │   ├── safetynet.go
  │   │   └── safetynet_test.go
  │   ├── apiconfig
  │   │   ├── config.go
  │   │   ├── database_provider.go
  │   │   ├── memory_provider.go
  │   │   └── provider.go
  │   ├── base64util
  │   │   ├── decode.go
  │   │   └── decode_test.go
  │   ├── cleanup
  │   │   ├── cleanup.go
  │   │   ├── cleanup_test.go
  │   │   └── config.go
  │   ├── database
  │   │   ├── apiconfig.go
  │   │   ├── apiconfig_test.go
  │   │   ├── config.go
  │   │   ├── connection.go
  │   │   ├── connection_test.go
  │   │   ├── database.go
  │   │   ├── database_test.go
  │   │   ├── export.go
  │   │   ├── export_test.go
  │   │   ├── exposure.go
  │   │   ├── exposure_test.go
  │   │   ├── federationin.go
  │   │   ├── federationin_test.go
  │   │   ├── federationout.go
  │   │   ├── federationout_test.go
  │   │   ├── iterator.go
  │   │   ├── lock.go
  │   │   └── lock_test.go
  │   ├── envconfig
  │   │   ├── envconfig.go
  │   │   └── envconfig_test.go
  │   ├── export
  │   │   ├── batcher.go
  │   │   ├── batcher_test.go
  │   │   ├── config.go
  │   │   ├── exportfile.go
  │   │   ├── server.go
  │   │   ├── server_test.go
  │   │   ├── worker.go
  │   │   └── worker_test.go
  │   ├── federationin
  │   │   ├── config.go
  │   │   ├── federationin.go
  │   │   └── federationin_test.go
  │   ├── federationout
  │   │   ├── config.go
  │   │   ├── federationout.go
  │   │   └── federationout_test.go
  │   ├── flag
  │   │   └── flag.go
  │   ├── handlers
  │   │   ├── delay.go
  │   │   └── delay_test.go
  │   ├── ios
  │   │   ├── devicecheck.go
  │   │   └── devicecheck_test.go
  │   ├── jsonutil
  │   │   ├── unmarshal.go
  │   │   └── unmarshal_test.go
  │   ├── logging
  │   │   └── logger.go
  │   ├── metrics
  │   │   ├── exporter.go
  │   │   └── exporter_test.go
  │   ├── model
  │   │   ├── apiconfig.go
  │   │   ├── apiconfig_test.go
  │   │   ├── export.go
  │   │   ├── exposure.go
  │   │   ├── exposure_test.go
  │   │   └── federation.go
  │   ├── pb
  │   │   ├── export
  │   │   │   ├── OWNERS
  │   │   │   ├── export.pb.go
  │   │   │   └── export.proto
  │   │   ├── federation.pb.go
  │   │   └── federation.proto
  │   ├── publish
  │   │   ├── config.go
  │   │   └── publish.go
  │   ├── secrets
  │   │   ├── gcp_secret_manager.go
  │   │   └── secrets.go
  │   ├── serverenv
  │   │   └── env.go
  │   ├── setup
  │   │   └── setup.go
  │   ├── signing
  │   │   ├── gcpkms.go
  │   │   └── signing.go
  │   ├── storage
  │   │   ├── google_cloud_storage.go
  │   │   └── storage.go
  │   └── verification
  │       ├── verify.go
  │       └── verify_test.go
✅├── migrations
✅│   ├── 000001_initial.down.sql
✅│   ├── 000001_initial.up.sql
✅│   ├── 000002_infection-exposure.down.sql
✅│   ├── 000002_infection-exposure.up.sql
✅│   ├── 000003_locking_procedures.down.sql
✅│   ├── 000003_locking_procedures.up.sql
✅│   ├── 000004_add_time_zone.down.sql
✅│   ├── 000004_add_time_zone.up.sql
✅│   ├── 000005_export_file_regions.down.sql
✅│   ├── 000005_export_file_regions.up.sql
✅│   ├── 000006_export_signing.down.sql
✅│   ├── 000006_export_signing.up.sql
✅│   ├── 000007_drop_bypass_safetynet.down.sql
✅│   ├── 000007_drop_bypass_safetynet.up.sql
✅│   ├── 000008_remove_apk_validation_options.down.sql
✅│   ├── 000008_remove_apk_validation_options.up.sql
✅│   ├── 000009_export_bucket_name.down.sql
✅│   ├── 000009_export_bucket_name.up.sql
✅│   ├── 000010_federation_client.down.sql
✅│   ├── 000010_federation_client.up.sql
✅│   ├── 000011_multiple_apk_hashes.down.sql
✅│   ├── 000011_multiple_apk_hashes.up.sql
✅│   ├── 000012_federation_audience.down.sql
✅│   ├── 000012_federation_audience.up.sql
✅│   ├── 000013_adjust_federation_naming.down.sql
✅│   ├── 000013_adjust_federation_naming.up.sql
✅│   ├── 000014_ios_per_app_config.down.sql
✅│   ├── 000014_ios_per_app_config.up.sql
✅│   ├── 000015_cts_integ_default.down.sql
✅│   ├── 000015_cts_integ_default.up.sql
✅│   ├── 000016_idx_exposure_created_at.down.sql
✅│   ├── 000016_idx_exposure_created_at.up.sql
✅│   ├── 000017_vacuum.down.sql
✅│   ├── 000017_vacuum.up.sql
✅│   ├── 000018_idx_app_package_name.down.sql
✅│   ├── 000018_idx_app_package_name.up.sql
✅│   ├── 000019_app_to_exportconfig.down.sql
✅│   └── 000019_app_to_exportconfig.up.sql
✅├── scripts
✅│   ├── create_db_migration.sh
✅│   ├── dev
✅│   └── presubmit.sh
  ├── setup_ko.sh
  ├── terraform
  │   ├── OWNERS
  │   ├── README.md
  │   ├── ko.Dockerfile
  │   ├── main.tf
  │   └── vars.tf
  ├── testdata
  │   └── config.sql
  └── tools
      ├── export-config
      │   └── main.go
      ├── exposure-client
      │   └── main.go
      ├── federationin-query
      │   └── main.go
      ├── federationout-authorization
      │   └── main.go
      ├── federationout-test
      │   └── main.go
      ├── hexconvert
      │   └── main.go
      └── unwrap-signature
          └── main.go
```

First, look at the small and separate folder.

```
├── .github
│   └── ISSUE_TEMPLATE
│       ├── bug_report.md
│       └── feature_request.md
```

`.github` folder save some templates for creating bug issues and feature issues. Make the communication between developers smoothly.

```
├── scripts
│   ├── create_db_migration.sh
│   ├── dev
│   └── presubmit.sh
```

Next is `./scripts`. It has 3 scripts.

- `./scripts/create_db_migration.sh` wraps the usage of [golang-migrate](https://github.com/golang-migrate/migrate). It's used to make database schema versional. We can easily update data schema and rolling back it. The versional database schema lay in `./migrations`

```
├── migrations
│   ├── 000001_initial.down.sql
│   ├── 000001_initial.up.sql
│   ├── 000002_infection-exposure.down.sql
│   ├── 000002_infection-exposure.up.sql
│   ├── 000003_locking_procedures.down.sql
│   ├── 000003_locking_procedures.up.sql
│   ├── 000004_add_time_zone.down.sql
│   ├── 000004_add_time_zone.up.sql
...

...
│   ├── 000019_app_to_exportconfig.down.sql
│   └── 000019_app_to_exportconfig.up.sql
```

- `./scripts/dev` wraps the usage of preparing database instance for dev environment. 

It runs a Postgres instace in local docker. And manage the schema, fake data, tls and protoc. This is all command of dev script.

```
  init         initialization for sourcing
  dbstart      start a dev server
  dbstop       stop the dev server
  dbmigrate    run migrations
  dbseed       seed dev data
  dburl        print url
  dbshell      attach a psql session
  protoc       re-compile protos
```

- `./scripts/presubmit.sh` make verification for us before submit. It also is used before code review by others. Save our time from a low level mistakes.

The verification includes of
- import clean up
- code fmt
- go mod check and clean
- build for test
- prepare db for test
- run test and output test coverage

```
├── terraform
│   ├── OWNERS
│   ├── README.md
│   ├── ko.Dockerfile
│   ├── main.tf
│   └── vars.tf
```

`terraform` is a cloud resource manager. It uses the declarative way to define and mange cloud resources. I have used it for several year. It's a great recommendation.

`vars.tf` includes some variables need to put in before run `terraform`.

`main.tf` includes all cloud object define.

In this project `terraform` manage several import source:

- `cloud build`. a google ci product.
- `cloud run`, a google container product to run the service.
- `database` in a `vpc`.
- `secret manager` is to manage `secret`.
- `google_cloud_scheduler_job` is to run a job periodically

`ko.Dockerfile`. `ko` is new for me. By reading its doc. It looks like to manage the process from compile golang code to apply all stuff to kubernetes cluster. It sounds great. I will recommend it to my colleagues.

To be continued.