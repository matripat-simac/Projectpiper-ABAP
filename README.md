# Build add-ons on a permanent ABAP Environment system

[back to main](https://github.com/SAP-samples/abap-platform-ci-cd-samples/tree/main)

## Prerequisites

* A permanent add-on assembly System is created in advance with provisioning parameter `is_development_allowed = false` and the service instance name as provided in `cfServiceInstance` [parameter in the pipeline configuration](.pipeline/config.yml#L15). The add-on assembly system is created using the `abap/standard` service. How to create such a system is described in [Creating an ABAP System](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html).
* The initial [clone of software components](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/18564c54f529496ba420d4c83545a2ce.html) included in the [add-on descriptor file](addon.yml) must be triggered in advance of the pipeline execution via the [Manage Software Components](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/3dcf76a072c9450eb46b99db947dab46.html) app.

## Description

Please refer to the [documentation](https://sap.github.io/jenkins-library/scenarios/abapEnvironmentAddons/) for more details about the scenario.

In this example a **permanent** add-on assembly system is being used and the system **is not deleted** in the [Post stage](https://www.project-piper.io/pipelines/abapEnvironment/stages/post/).
Software components are imported into the system by using the [`CheckoutPull` strategy](https://www.project-piper.io/pipelines/abapEnvironment/stages/cloneRepositories/#stage-parameters).

**Note**: The add-on assembly system should be only kept for the duration of a maintenance branch, and only be used to build new patch versions on a single support package level.

### Pipeline Stages

![ABAP Environment Build Pipeline](https://www.project-piper.io/images/abapEnvironmentBuildPipeline.png "ABAP Environment Build Pipeline")

#### [Initial Checks](https://www.project-piper.io/pipelines/abapEnvironment/stages/initialChecks/)
- Check the validity of defined addon product version
- Check the validity of defined software component versions

#### [Prepare System](https://www.project-piper.io/pipelines/abapEnvironment/stages/prepareSystem/)
- Preparing a system for add-on assembly
- Creating a Communication Arrangement for the Scenario SAP_COM_0510 via a service key in the assembly system

#### [Clone Repositories](https://www.project-piper.io/pipelines/abapEnvironment/stages/cloneRepositories/)
- Switch between branches of a git repository on a SAP Cloud Platform ABAP Environment system
- Clone software components relevant for the add-on build

#### [ATC](https://www.project-piper.io/pipelines/abapEnvironment/stages/ATC/)
- Check software components to be assembled as part of the add-on build via ABAP Test Cockpit (Check Variant: SAP_CLOUD_PLATFORM_ATC_DEFAULT)

**Note**: The ATC results are displayed using the [Warnings Next Generation Plugin](https://www.jenkins.io/doc/pipeline/steps/warnings-ng/#warnings-next-generation-plugin). If you don't want to use this plugin - or if it's not available on your Jenkins server - leave out the extension for the ATC stage (.pipeline/extensions/ATC.groovy).

#### [Build](https://www.project-piper.io/pipelines/abapEnvironment/stages/build/)
  * Creating a Communication Arrangement for the Scenario SAP_COM_0582 via a service key in the assembly system
  * Determine the ABAP delivery packages (name and type) to be created as part of the add-on build
  * Assembly of installation, support package or patch in in the assembly system
  * Upload resulting SAR archives and creating physical delivery packages in in the File Content Management System of SAP
  * Create a target vector for software lifecycle operations
  * Release the physical delivery packages
  * Trigger publication of the target vector with test scope

#### [Integration Tests](https://www.project-piper.io/pipelines/abapEnvironment/stages/integrationTest/)
  * Preparing a system for the add-on test installation, system is deprovisioned after successful testing has been confirmed

#### [Confirm](https://www.project-piper.io/pipelines/abapEnvironment/stages/confirm/)
Release Decision

#### [Publish](https://www.project-piper.io/pipelines/abapEnvironment/stages/publish/)
  * After confirmation: Trigger publication of the target vector with production scope

## Configuration
### [.pipeline/config.yml](.pipeline/config.yml)
Based on the [pipeline configuration default values](https://github.com/SAP/jenkins-library/blob/master/resources/com.sap.piper/pipeline/abapEnvironmentPipelineStageDefaults.yml) a configuration file .pipeline/config.yml is used to provide all required values to run the pipeline.

| **Pipeline Stage/Parameter**                                                                                    | **Description**                                                                                                                                                                                                                              | **Remarks**                                                                                                                                                                                 |                          
|-----------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| general › abapAddonAssemblyKitCredentialsId                                                                     | Credentials stored in Jenkins for [technical communication user](https://launchpad.support.sap.com/#/notes/2174416) to access AAKaaS                                                                                                         | ID of username/password credentials                                                                                                                                                         |
| general › addonDescriptorFileName                                                                               | Path to YAML config file for [addon descriptor](https://www.project-piper.io/scenarios/abapEnvironmentAddons/#add-on-descriptor-file)                                                                                                        | ID of username/password credentials                                                                                                                                                         |
| general › cfApiEndpoint                                                                                         | [Cloud Foundry API Endpoint](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/350356d1dc314d3199dca15bd2ab9b0e.html#loio879f37370d9b45e99a16538e0f37ff2c) specific to region of Cloud Foundry Environment to be used | Subaccount Overview → Cloud Foundry Environment → API Endpoint                                                                                                                              |
| general › cfOrg                                                                                                 | [Cloud Foundry Org](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/fe1ebf3cd6fe46798efcaf45c73a54ce.html) used to create assembly system                                                                           | Subaccount Overview → Cloud Foundry Environment → Org Name                                                                                                                                  |
| general › cfSpace                                                                                               | [Cloud Foundry Space](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/5209d55d8dd84228897112b0655d999b.html) used to create assembly system                                                                         |                                                                                                                                                                                             |
| general › cfCredentialsId                                                                                       | Credentials stored in Jenkins for [Space Developer](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/967fc4e2b1314cf7afc7d7043b53e566.html) user to authenticate to the Cloud Foundry API                            | User needs org member/subaccount administrator and assigned to space withs space developer role                                                                                             |
| general › cfServiceInstance                                                                                     | Name of [ABAP service instance](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html) for add-on assembly                                                                          |                                                                                                                                                                                             |
| general › cfServiceKeyName                                                                                      | Name of [Cloud Foundry Service Key](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/4514a14ab6424d9f84f1b8650df609ce.html)                                                                                          | Creation of communication arrangement [SAP_COM_0510](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/b04a9ae412894725a2fc539bfb1ca055.html)                        |
| stages › [Prepare System](https://www.project-piper.io/pipelines/abapEnvironment/stages/prepareSystem/)         | In this stage, the SAP BTP, ABAP environment system for add-on assembly is created                                                                                                                                                           |                                                                                                                                                                                             |
| stages › Prepare System › abapSystemAdminEmail                                                                  | [E-Mail address for the initial administrator](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html) of the assembly system                                                        |                                                                                                                                                                                             |
| stages › Prepare System › abapSystemID                                                                          | Three character name of the assembly system - maps to [sapSystemName](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html)                                                        | Shown in Landscape Portal application                                                                                                                                                       |
| stages › Prepare System › cfServiceKeyConfig                                                                    | Path to [JSON config file for Cloud Foundry Service Key](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/1cc5a1da02594b93a70f6c0fe2bfdfe8.html) creation                                                            | Creation of communication arrangement [SAP_COM_0510](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/b04a9ae412894725a2fc539bfb1ca055.html)                        |
| stages › [Clone Repositories](https://www.project-piper.io/pipelines/abapEnvironment/stages/cloneRepositories/) | This stage creates pulls/clones the specified software components (repositories) to the SAP BTP, ABAP environment system                                                                                                                     | addon.yml file can be used as format is the same as repositories.yml                                                                                                                        |
| stages › Clone Repositories › repositories                                                                      | Specifies a YAML file containing [configuration of repositories](https://www.project-piper.io/pipelines/abapEnvironment/stages/cloneRepositories/#repositoriesyml) to be imported                                                            | addon.yml file can be used as format is the same as repositories.yml                                                                                                                        |
| stages › Clone Repositories › strategy                                                                          | Influences, [which steps will be executed to import the repositories](https://www.project-piper.io/pipelines/abapEnvironment/stages/cloneRepositories/#stage-parameters)                                                                     | Clone' is recommended if a new system is created in the Prepare System stage, 'CheckoutPull' is recommended if a static system is used. The software component should be cloned beforehand. |
| stages › [ATC](https://www.project-piper.io/pipelines/abapEnvironment/stages/ATC/)                              | In this stage, ATC checks can be executed using abapEnvironmentRunATCCheck                                                                                                                                                                   |                                                                                                                                                                                             |
| stages › ATC › atcConfig                                                                                        | Path to a YAML [file including check configuration](https://www.project-piper.io/steps/abapEnvironmentRunATCCheck/#atc-config-file-example) for packages and/or software components                                                          |                                                                                                                                                                                             |
| stages › [Build](https://www.project-piper.io/pipelines/abapEnvironment/stages/build/)                          | This stage is responsible for building an ABAP add-on for the SAP BTP, ABAP environment                                                                                                                                                      |                                                                                                                                                                                             |
| stages › Build › cfServiceKeyName                                                                               | Name of [Cloud Foundry Service Key](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/4514a14ab6424d9f84f1b8650df609ce.html)                                                                                          | Creation of communication arrangement [SAP_COM_0582](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/26b8df5435c649aa8ea7b3688ad5bb0a.html)                        |
| stages › Build › cfServiceKeyConfig                                                                             | Path to [JSON config file for Cloud Foundry Service Key](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/1cc5a1da02594b93a70f6c0fe2bfdfe8.html) creation                                                            | Creation of communication arrangement [SAP_COM_0582](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/26b8df5435c649aa8ea7b3688ad5bb0a.html)                        |
| stages › [Integration Tests](https://www.project-piper.io/pipelines/abapEnvironment/stages/integrationTest/)    | Creates an SAP BTP, ABAP environment (Steampunk) system and installs the add-on product, that was built in the Build stage                                                                                                                   |                                                                                                                                                                                             |
| stages › Integration Tests › cfOrg                                                                              | [Cloud Foundry Org](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/fe1ebf3cd6fe46798efcaf45c73a54ce.html) used to create installation test system                                                                  | Subaccount Overview → Cloud Foundry Environment → Org Name                                                                                                                                  |
| stages › Integration Tests › cfSpace                                                                            | [Cloud Foundry Space](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/5209d55d8dd84228897112b0655d999b.html) used to create installation test system                                                                |                                                                                                                                                                                             |
| stages › Integration Tests › cfServiceInstance                                                                  | Name of [ABAP service instance](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html) for installation test                                                                        |                                                                                                                                                                                             |
| stages › Integration Tests › abapSystemAdminEmail                                                               | [E-Mail address for the initial administrator](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html) of the installation test system                                               |                                                                                                                                                                                             |
| stages › Integration Tests › abapSystemID                                                                       | Three character name of the installation test system - maps to [sapSystemName](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/50b32f144e184154987a06e4b55ce447.html)                                               | Shown in Landscape Portal application                                                                                                                                                       |
| stages › [Publish](https://www.project-piper.io/pipelines/abapEnvironment/stages/publish/)                      | This stage publishes an add-on for the SAP BTP, ABAP environment                                                                                                                                                                             |                                                                                                                                                                                             |

### Users
Create [Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) for following users.
#### cfCredentialsId
Plaform User in Cloud Foundry environment to create assembly system, add-on installation test system, create communication arrangements SAP_COM_0510 and SAP_COM_0582 via service keys. The user needs to be a member of the global account and has to have the [Space Developer role](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/967fc4e2b1314cf7afc7d7043b53e566.html) in space of ABAP Environment systems.

#### abapAddonAssemblyKitCredentialsId
Communication with the AAKaaS needs a technical communication user. The creation and activation of such a user in SAP ONE Support Launchpad is described in [SAP note 2174416](https://launchpad.support.sap.com/#/notes/2174416). Make sure that this technical communication user is assigned to the customer number under which the “SAP CP ABAP ENVIRONMENT” tenants are licensed and for which the development namespace was reserved.

### [addon.yml](addon.yml)
Definition of addon product version/software component versions bundle to be assembled.
| **Parameter**           | **Description**                                                                                           | **Remarks**                                                                                                                                                                                                        |
|-------------------------|-----------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| addonProduct            | Technical name of the add-on product                                                                      | Add-on product namne registered with SAP, see [Register Add-on Product for a Global Account](https://www.project-piper.io/scenarios/abapEnvironmentAddons/#register-add-on-product-for-a-global-account)           |
| addonVersion            | Technical version of the add-on product: `<product version>.<support package stack level>.<patch level>    |                                                                                                                                                                                                                    |
| repositories            | Contains one or multiple software component versions                                                      | Order of software components defines import order                                                                                                                                                                  |
| repositories > name     | Technical name of the software component                                                                  | Name of software component in [Manage Software Components](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/3dcf76a072c9450eb46b99db947dab46.html) application                             |
| repositories > branch   | This is the maintenance branch from the git repository                                                    | Name of branch in [Manage Software Components](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/3dcf76a072c9450eb46b99db947dab46.html) application to be checked out for add-on assembly   |
| respoitories > version  | This is the technical software component version                                                          |                                                                                                                                                                                                                    |
| repositories > commitID | commit id specified for a repository                                                                      | Short Commit ID as provided in [Manage software Components](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/3dcf76a072c9450eb46b99db947dab46.html)) application                           |

### [atcConfig.yml](atcConfig.yml)
Configuration of software components checked via ATC, check variant `SAP_CLOUD_PLATFORM_ATC_DEFAULT` is used for check runs.
Please refer to the documentation on [ATC config file](https://www.project-piper.io/steps/abapEnvironmentRunATCCheck/#atc-config-file-example) for more information.

### [sap_com_0510.json](sap_com_0510.json)
Service key parameters for creation of communication arrangement SAP_COM_0510

### [sap_com_0582.json](sap_com_0582.json)
Service key parameters for creation of communication arrangement SAP_COM_0582

## Feedback
Please create a [GitHub issue](https://github.com/SAP-samples/abap-platform-ci-cd-samples/issues) in this repository if you have suggestions on how to improve this example.
