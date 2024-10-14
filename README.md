# THIS IS A TEST. FORGET WHAT IS BELOW.

# sci-devops-action

Custom GitHub action for doing all the core logic of interacting with SCI.

The action uses a temporary folder for storing all relevant files. At no point any file outside the internal temporary folder is created or modified.

From this temporary folder the step(s) coming after the action can use both the actions output and/or generated files to do the proper GIT related logic.

# Notes

- Remember to set this repository config: `Accessible from repositories in the kksap organization`
- Remember to add a secret for `GH_TOKEN` to this repository (ideally this should be an org secret)
- Remember to create labels:
  - _actions_: color example: `#f9b405`, description: `GitHub Actions issues and PRs`
  - _dependabot_: color example: `#c6eaf7`, description: `Dependabot issues and PRs`
  - _npm_: color example: `#19154d`, description `Node.js issues and PRs`
- The following module must be installed _globally_ when using download operation: `@kksap/sci-devops-documenter`

# Operations for SCI

1. [Artifact, download ](#artifact-download)
1. [Artifact, upload](#artifact-upload)
1. [Artifact, deploy ](#artifact-deploy)
1. [Artifact, undeploy ](#artifact-undeploy)
1. [Artifact, configure ](#artifact-configure)
1. [Packages, backup ](#packages-backup)
1. [Packages, restore ](#packages-restore)
1. [Internal, merge iflow parameters ](#internal-merge-ifow-params)

<br>
<a name="artifact-download"></a>

## Artifact, download

Downloads a single SCI artifact of the specified type. It also downloads and stores package info.

The artifact is stored unzipped.

This operation also generates documentation. All downloaded artifacts are documented in the common file (`README.md`). This included basic design- and runtime
information.

For iFlows the following actions are also performed:

- Generation of technical documentation. This is done in the file `documentation.md`
- Linting
- Generation of an adapted version of the existing, local GitHub workflow for configuration. All input parameters are replaced with current external properties
  and values.
- IFF the optional input parameter `apiServiceKeyApim` is supplied, then the action will also look for related SAP API Management API Proxies and list results in the generated common documentation.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`download`)
- folder_name
  - description: name of local folder artifact is downloaded to
- package_id
  - description: SCI package ID
- artifact_id
  - description: SCI artifact ID
- artifact_type
  - description: SCI artifact type. One of: `valuemapping`, `scriptcollection`, `integrationflow`
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`
- apiServiceKeyApim
  - description: APIM service key (OAuth credentials) for `API Management, API portal (apimanagement-apiportal)` using plan `apiportal-apiaccess`
- skipProcessingSteps
  - description: Optional parameter containing a comma separated string specifying the steps to skip.

### Output

- ARTIFACT_ZIP_FILE_PATH:
  - description: Path to temporary artifact ZIP file
- ARTIFACT_FOLDER_PATH:
  - description: Path to temporary artifact folder containing downloaded artifact
- PACKAGE_INFO_FILE_PATH:
  - description: Path to temporary package info file
- DOCU_COMMON_FILE_PATH:
  - description: Path to temporary documentation file, common
- DOCU_IFLOW_IMG_FILE_PATH:
  - description: Path to temporary documentation file, iflow image
- DOCU_IFLOW_DOC_FILE_PATH:
  - description: Path to temporary documentation file, iflow documentation
- CONFIGURE_PARAMS_FILE_PATH:
  - description: Path to temporary file containing the externalized parameters used during configure job execution
- ARTIFACT_VERSION_OLD:
  - description: Artifact version, old
- ARTIFACT_VERSION_NEW:
  - description: Artifact version, new
- IS_ARTIFACT_NEW:
  - description: 'Boolean indicator: is artifact new?'
- IS_ARTIFACT_IFLOW:
  - description: 'Boolean indicator: is artifact an iflow?'
- SKIP_LINTING:
  - description: 'Boolean indicator: skip linting?'

<br>
<a name="artifact-upload"></a>

## Artifact, upload

Uploads local, unzipped artifact to SCI.

If the package does not exist in advance, this is created.

> [!IMPORTANT]
> When uploading an artifact and creating a package we intentionally do not allow the package version to be carried through. This is because when we transport single artifacts only we have absolutely no way of knowing what the proper package version should be. So the package version may quickly become confusing when transporting single artifacts.

> [!WARNING]
> During an upload, Value Mappings are treated a little special, since the current API does not allow updating an entire value mapping but only independent values. For
this reason value mappings are (if existing in advance) deleted and then uploaded. _This means history is lost in SCI_.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`upload`)
- folder_name
  - description: name of local folder cantaining artifact
- package_id
  - description: SCI package ID
- artifact_id
  - description: SCI artifact ID
- artifact_type
  - description: SCI artifact type. One of: `valuemapping`, `scriptcollection`, `integrationflow`
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- ARTIFACT_VERSION_OLD:
  - description: Artifact version, old
- ARTIFACT_VERSION_NEW:
  - description: Artifact version, new

<br>
<a name="artifact-deploy"></a>

## Artifact, deploy

Deploys SCI design time artifact to runtime.

Checks for succesful deployment and terminates with error in case of failure.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`deploy`)
- folder_name
  - description: name of local folder cantaining artifact
- package_id
  - description: SCI package ID
- artifact_id
  - description: SCI artifact ID
- artifact_type
  - description: SCI artifact type. One of: `valuemapping`, `scriptcollection`, `integrationflow`
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- ARTIFACT_VERSION_OLD:
  - description: Artifact version, old
- ARTIFACT_VERSION_NEW:
  - description: Artifact version, new
- DEPLOYMENT_STATUS:
  - description: Status of deployment of artifact to SCI runtime
- DEPLOYMENT_TASK_ID:
  - description: Deployment task ID

<br>
<a name="artifact-undeploy"></a>

## Artifact, undeploy

Undeploys SCI runtime artifact from runtime.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`undeploy`)
- folder_name
  - description: name of local folder cantaining artifact
- artifact_id
  - description: SCI artifact ID
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- ARTIFACT_VERSION_OLD:
  - description: Artifact version, old
- ARTIFACT_VERSION_NEW:
  - description: Artifact version, new
- UNDEPLOYMENT_STATUS:
  - description: Status of undeployment of artifact from SCI runtime

<br>
<a name="artifact-configure"></a>

## Artifact, configure iFlow

Configures an iFlow based on the content of the provided GitHub input parameters.

Recall input parameters are auto-updated during execution of an artifact download.

GitHub workflow input parameters are fetched directly from the GitHub context and not from direct input parameters (we do not know number and names of
parameters).

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`configure`)
- folder_name
  - description: name of local folder cantaining artifact
- artifact_id
  - description: SCI artifact ID
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- ARTIFACT_VERSION_OLD:
  - description: Artifact version, old
- ARTIFACT_VERSION_NEW:
  - description: Artifact version, new
- PARAMETERS_TOTAL:
  - description: Total number of externalized parameters
- PARAMETERS_CHANGED:
  - description: Total number of externalized parameters requiring change

<br>
<a name="packages-backup"></a>

## Packages, backup

Backup _all_ packages from SCI to GitHub.

Stores the raw packages unchanged in the specified folder. Also stores a file `packagesInfo.json` containing info of all packages. Based on this file
documentation is generated:

The file `README.md` contains a (dynamic) list of packages downloaded along with some basic header and status information.

Notes:

- Backup of packages can be seen as _additive_. A package is never deleted from GitHub no matter if it is deleted in SCI. Instead we mark these packages with
  status `DELETED` in the documentation file.
- If a package cannot be downloaded this is logged and processing continues to next package. This will also be reflected in the documentation.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`backup-packages`)
- folder_name
  - description: name of local folder cantaining packages
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- PACKAGE_COUNT_SCI:
  - description: 'Number of packages to be downloaded from SCI'
- PACKAGE_COUNT_NEW:
  - description: 'Number of downloaded packages that are new (did not exist in GIT repository)'
- PACKAGE_COUNT_UPDATED:
  - description: 'Number of downloaded packages that are updated (did exist in GIT repository)'
- PACKAGE_COUNT_DELETED:
  - description: 'Number of downloaded packages that are deleted (exist in GIT repository, but no longer in SCI)'
- PACKAGE_COUNT_ERROR:
  - description: 'Number of packages that could not be processed due to errors'
- PACKAGE_COUNT_TOTAL:
  - description: 'Total number of packages (local + SCI)'
- PACKAGE_DOC_FILE:
  - description: 'Path to temporary package documentation file'
- PACKAGE_INFO_FILE:
  - description: 'Path to temporary packages info file'

<br>
<a name="packages-restore"></a>

## Packages, restore

Restores (uploads) one or more local packages to SCI.

Selection of which packages to restore in SCI is based on the following GitHub workflow input parameters:

- _selection_: valid values can be either `all` or `<file name>`
- _overwrite_: determines if any existing package(s) in SCI can be overwritten or not

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`restore-packages`)
- folder_name
  - description: name of local folder cantaining packages
- apiServiceKeySci
  - description: SCI service key (OAuth credentials) for `Process Integration Runtime (it-rt)` using plan `API`

### Output

- PACKAGES_UPLOAD_FILES_COUNT:
  - description: Number of packages uploaded to SCI

<br>
<a name="internal-merge-ifow-params"></a>

## Merge iflow parameters

This is an interal operation used during move of iFlows from source to target branch in GitHub.

It generates a temporary file containing merged externalized parameters so that all property names from source branch are move to target branch. If a property
already existed in target branch then the value of the property is preserved. If the property did not exist in advance then value from source is carried over to
target branch.

### Input

- scope
  - description: name of scope (`sci`)
- operation
  - description: name of operation (`merge-iflow-params`)
- artifact_type
  - description: SCI artifact type. Used to ensure `integrationflow` is being processed
- folder_name
  - description: name of local folder cantaining artifact
- oldPropertiesFile
  - description: file containing externalized properties from source branch
- newPropertiesFile
  - description: file containing externalized properties from target branch
- targetPropertiesFile
  - description: file containing merged externalized properties to be used in target branch

### Output

- MERGED_PARAMETERS_FILE:
  - description: Path to file containing merged externalized parameters
