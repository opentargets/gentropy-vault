# Gentropy Vault
Some external data is either not guaranteed to be there forever, or is for various reasons difficult to get a hold of. This can be due to unstable/slow infrastructure, non-standard access methods, requiring authentication to access, or any combination of those.

In order to be able to easily access this data and to not lose it in the event the external storage is discontinued, a Gentropy Vault bucket has been set up at `gs://gentropy-vault`.

## Bucket details
* The Vault is a **regional** bucket located in europe-west1.
* Its project is **open-targets-genetics-dev**.
* The bucket has the [Autoclass](https://cloud.google.com/storage/docs/autoclass) feature enabled.
  - This helps reduce cost of storing the infrequently-accessed data while allowing us to retain flexibility on when and how we want to use it.
  - The data starts in Standard storage, then, if it's not accessed, progressively sinks into more colder storage: to Nearline (after 1 months), Coldline (after a fu
  - With Autoclass, there are no retrieval costs and no early deletion costs.
  - See more on Autoclass cost considerations in this ticket: https://github.com/opentargets/issues/issues/3232.
* The bucket has [object versioning](https://cloud.google.com/storage/docs/object-versioning) enabled for the purposes of data recovery.
  - In addition to a live version of the object, at most **one** archive version is kept. This version will survive if the file is overwritten or deleted.
  - Archive versions will be deleted after **14 days,** so any data which is overwritten or deleted will need to be recovered in this time frame.
* The bucket has [public access prevention](https://cloud.google.com/storage/docs/public-access-prevention) enabled, as it is intended only for internal use.

## Datasets and ingestion instructions
See [**common instructions**](ingest_vm.md) to set up a VM for ingesting any dataset. For each dataset, the specific instructions on how exactly it was ingested must also be kept. The table below lists the datasets currently mirrored. Link on the dataset name leads to detailed instructions and notes.

| Name | Vault link | Source platform | Date mirrored | Size |
| ---- | ---- | ---- | ---- | ---- |
| [UKB PPP](datasets/ukb-ppp.md) | `gs://gentropy-vault/ukb-ppp` | Synapse | 2024-03-01 | 8.53 TB |
| [deCODE proteomics](datasets/decode-proteomics.md) | `gs://gentropy-vault/decode-proteomics` | deCODE | 2024-03-05 | 4.26 TB |
