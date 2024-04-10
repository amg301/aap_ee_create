# Here is a great overview for Execution Environments
[Execution Environments Crash Course](https://docs.autodotes.com/EE%20Crash%20Course/01_overview/)

## Requirements for this workflow:
1) A RHEL 8 or 9 server (VM or physical) with internet access
2) The ansible-builder application. This can be installed from an rpm in the ansible-automation-platform-2.4-for-rhel9-x86_64-rpms, from the AAP bundled installer or from pip
    * if installing from the repo you will need to enable the repo, install the rpm, and disable the repo as it should be disabled by default to avoid patching issues.
      * ```subscription-manager repos --enable ansible-automation-platform-2.4-for-rhel9-x86_64-rpms```
      * ```dnf install ansible-builder```
      * ```ansible-builder --version``` should return ```3.0.1``` (correct as of March 2024)
      * ```subscription-manager repos --disable ansible-automation-platform-2.4-for-rhel9-x86_64-rpms```
3) A container repository for hosting the built EEs
    * One of the functions of Private Automation Hub which is part of the platform is a container registry.
      * This example uses [Red Hat Quay](https://quay.io/) instead
      * Other common container repos include GitHub Container Registry, GitLab Container Registry, JFrog Container Registry, Docker Hub, Harbor and more.
5) Your login details for Red Hats Certified Content Collections
    * Get this from [Red Hat Automation Hub on Console](https://console.redhat.com/ansible/automation-hub/token)
    * Take note of the server URL and token and define this in your ansible.cfg

## Key points
1) Each EE you want to build will be a unique directory containing a single execution-environment.yml file
2) You create the execution-environment.yml file
3) Versioning your EE is done with tags once the container image is built
4) The container image is built locally but needs to be pushed into the container repo for storage and calling from AAP Automation Controller
5) The EE needs to be defined within AAP Automation Controller

## The commands you will need. These are run from the root of the directory you created for this EE build. The one with the execution-environment.yml file in it.
1) ```ansible-builder build -t quay.io/agarrett/aap-ees/example-ee:0.0.1```
    * the -t is critical. This tags with EE with the container repository and version. Don't use latest, assign that later.
    * if you are using a different container repo change quay.io/agarrett/aap-ees to the right info. The name of the EE here is example-ee and the version is 0.0.1
2) ```podman login quay.io```
    * you need to authenticate to your container repo. Podman will ask you for your username and password. If correct it will return login succeeded. This is only needed once per terminal session.
3) podman push quay.io/agarrett/aap-ees/example-ee:0.0.1
    * [My EE in Quay](https://quay.io/repository/agarrett/aap-ees/example-ee?tab=tags)
4) in AAP Automation Controller > Resources > Credentials add a secret for your container repo. If using Private Automation Hub this is automatically created by the AAP setup.
    * use Credential Type > Container Registry
4) in AAP Automation Controller > Administration > Execution Environments add a definition for the new EE you have created.
    * give it a name
    * define the image - quay.io/agarrett/aap-ees/example-ee:0.0.1 in this example
    * define the pull style. ------ means missing, only pull if EE does not exist
    * Associate the Registry Credential you created in the previous step
5) Define the EE in a job template and run it

## In this Repo
  * You will find an example execution-environment.yml file
  * You will find a directory showing additional files - an ansible.cfg and an additional repo
  * You will find the build context that is created by the ansible-builder tool
