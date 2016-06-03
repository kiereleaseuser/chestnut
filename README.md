RELEASING
=========
Table of content
----------------

* **[Introduction](#introduction)**
* **[Pre release actions](#pre-release-actions)**
    * **[Zanata](#zanata)**
        * **[Pull Zanata translation changes](#zanata-pulls)**
        * **[Push Zanata translation changes](#zanata-pulls)**
    * **[Remove old Uberfire & Ubefire-extensions release branches](#remove-old-branches)**
    * **[Remove old Dashbuilder release branches](#remove-old-branches)**
    * **[Remove old KIE release branches](#remove-old-branches)**
    * **[Close release on JIRA](#close-release-on-jira)**
    * **[Mail to the bsig team](#mail-to-bsig-team)**
    
* **[Releasing third party repositories](#releasing-third-party-repositories)**
    * **[Releasing Uberfire & Uberfire-extensions](#releasing-uberfire-uberfire-extensions)**
    * **[Releasing Dashbuilder](#releasing-dashbuilder)**
                    
* **[KIE releases](#kie-releases)**
    * **[Push KIE release branches](#push-kie-release-branch)**
    * **[Deploy locally KIE release artifacts](#deploy-locally-kie)**
    * **[Sanity Checks](#sanity-checks)**
    * **[Cherry-picking](#cherry-picking)**
    * **[Copy deployed KIE binaries to Nexus](#copy-deployed-kie-to-nexus)**
        * *[jbpm-test-coverage](#jbpm-test-coverage)*
        * *[kie-api-backwards-compat-check](#kie-api-backwards-compatible-check)*
        * *[kie-server-matrix](#kie-server-martix)*
        * *[kie-wb-smoke-tests-matrix](#kie-workbench-smoke-tests-matrix)*
        
* **[Post release actions](#post-release-actions)**
    * **[Push tags to droolsjbpm/jboss-integration](#push-tags)**
        * *[Push Uberfire & Uberfire-extensions tag](#push-uberfire-tag)*
        * *[Push Dashbuilder tag](#push-dashbuilder-tag)*
        * *[Push KIE tags](#push-kie-tags)*
    * **[Release staging repositories](#release-staging-repositories)**    
    * **[Community](#community)**    
        *  *[Updating to next development version](#update-to-next-development-version)*
            * *[Update Uberfire & Uberfire-extensions](#update-uberfire)*
            * *[Update Dashbuilder](#update-dashbuilder)*
            * *[Update KIE ](#update-kie)*
        *  *[Copy artifacts to filemgmt.jboss.org](#copy-community-artifacts)*
        *  *[Run scripts for latest and latestFinal](#run-scripts-for-latest)*
        *  *[Enter release date on JIRA](#enter-release-date-on-jira)*
    * **[Product](#product)**
        *  *[Integration Package](#integration-pack)*      
        *  *[Fill out hand off report](#hand-off-report)*
        
        
Introduction
============
Historically the releases for droolsjbpm (kie) were done manually following this document [README](https://github.com/droolsjbpm/droolsjbpm-build-bootstrap/blob/master/RELEASE-README.md).<br>
Nowerdays kie team use the [Jenkins CI](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/) with all its views and scripts to release.<br>
When kie-team has to do a release it has two main reasons:

* a realease + tag for community (i.e. 6.4.0.Final) based on an community branch or a community tag + some cherry-picks
* a tag for productization (i.e. sync-6.4.x-2016.04.21) based on a community branch, a community tag or previous product tag + some cherry-picks

All this possibilities are covered with the different scripts.

The main views for releasing in Jenkins CI are:

* **[Zanata](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/Zanata)**
* **[uf-releases-0.7.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.7.x)**
* **[uf-releases-0.8.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.8.x)**
* **[dashbuilder-releases 0.3.x and 0.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/dashbuilder-releases%200.3.x%20and%200.4.x/)**
* **[kie-releases-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/)**
* **[kie-releases-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/)**

In future time there will be created new views and old views, that no longer are supported will dissapear.<br>
Right now (April 2016) the master branch of droolsjbpm is 7.0.0-SNAPSHOT. There will be created a new 7.0.x branch soon and so created a new view and new scripts on Jenkins, also 
it will be removed some day the 6.2.x view.<br>
The server where the releases are build a slave for Jenkins CI is **kie-releases.lab.eng.brq.redhat.com**.


Pre release actions
===================
Before releasing kie we have first to pull latest Zanata translation changes from Zanata server and remove temporary branches used for previous releases for Uberfire, Uberfire-extensions, dashbuilder and
and kie.
Zanata
------
There are two main scripts for pushing or pulling Zanata modules. *pull-Zanata-translation-changes-\<branch\>* and *push-Zanata-translation-changes-\<branch\>* available on the view 
[Zanata](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/Zanata/) of Jenkins CI.<br>
**push-Zanata-translation-changes-\<branch\>** pushes the new words in i18n files for beeing translated from droolsjbpm to Zanata-server.<br>
**pull-Zanata-translation-changes-\<branch\>** is executed before a release (+/-two days).<br>
What this script does is basically:

    mvn -B zanata:pull-module (pulls all Zanata trabslations from Zanata server)
    mvn replacer:replace -N (replaces basically apostrophe "'" by double apostrophe "''" - needed in french translations)
    mvn native2ascii=native2ascii (only in repositories where this is needed)
    git adds and commits (if there are changes)
    raises a PR (pull request) to the repositories on github

The main repositories affected by Zanata changes are:

    uberfire
    uberire-extensions
    dashbuilder
    kie-uberfire-extensions
    guvnor
    kie-wb-common
    jbpm_form_modeler
    drools-wb
    jbpm-designer
    cd jbpm-designer-api
    jbpm-console-ng
    dashboard-builder
    optaplanner-wb
    jbpm-dashboard
    kie-wb-distributions

The jobs for Zanata push is executed every Monday (triggered).<br>
The jobs for Zanata pull is executed one day before the release, because the raised PRs should be merged before releasing. In some cases the jobs are triggered manually.<br>
It is to define if it makes more sense to run the scripts every day.
    
Remove old branches
-------------------
Before pushing new "release" branches to github/jboss-integration is is important to remove old branches from previous builds to avoid collecting these temporary branches.<br>
Therefore there are three scripts to remove this branches.<br>

**012.Releasing_Uberfire_Ubefire-extensions_remove-release-branches-\<branch\>** \[removes uberfire and uberfire-extensions branches\]<br>
**022.Releasing_Dashbuilder_remove-release-branches** \[removes dashbuilder branches\]<br>
**034.Releasing_KIE_remove-release-branches-\<branch\>** \[removes kie branches\]<br>
All scripts have this parameters when building (Build with parameters):

    the user has to select if it is a community (droolsjbpm) or a product (jboss-integration) branch
    the exact branch name has to be edited here 
        (r\<tag name\> for community i.e r6.4.0.Final)
        (bsync-\<branch\>-yyyy.mm.dd for productization  i.e. bsync-6.4.x-2016.04.21)

Close release on JIRA
---------------------
When a community release is upcoming there is a time period where people don't push anymore to the branch that serves as base for the release. In this time period developers
can continue i.e working on JIRAs. When they select the version they can assign it to the recent version whereas this version is already building. So these JIRAs will be assigned to the recent
version but will not be in this release. Therefore it is good to close the release on JIRA **without** a date, so all not yet fixed JIRAs will be migrated to the next version.<br>
Once the release is done the date will be edited.<br>
**<font color="red">IMPORTANT</font>:** this only is required for community releases!

Mail to bsig team
-----------------
Before starting the release procedure (two days before) it is important to altert the team about this.
When releasing for community a mail to <font color="blue">bsig@redhat.com</font> or <font color="blue">bsig-all@redhat.com</font> should be send announcing the new release.
 
    EXAMPLE MAIL:
    Next week on Tuesday, 5th of April, we are going to branch the 6.4.x
    for releasing the community 6.4.0.CR3.
    It is supposed that we have the tag for it on Tuesday EOD so we can
    do the prod tag 6.3.0.ER3 on Wednesday.
    I hope you will have everything prepared for Tuesday.
    Have in mind that we still can cherry-pick something that has to be
    in the CR3. The same procedure as every time, a mail to me and in CC Edson and Kris with
    the commits to cherry pick.

It is important that everyone who could not finish his task until the announced day should send a mail to the release engineer (<font color="blue">mbiarnes@redhat.com</font>) and to Edson (<font color="blue">etirelli@redhat.com</font>) 
and Kris (<font color="blue">kverlaen@redhat.com</font>).<br>
There can also be changed the subject on the XChat to inform the team about the status of the release.

Releasing third party repositories
==================================
Once all steps specified in this doc until here are closed and before beginning to release the KIE repositories,<br>
Uberfire<br>
Uberfire-extensions (only versions 0.7.x and 0.8.x)<br>
and Dashbuilder<br>
have to be released.
 
Releasing Uberfire-Uberfire-extensions
--------------------------------------
For Uberfire and Uberfire-extensions there exist different views on [Jenkins CI](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com) depending on the branch we want to release:

0.7.x:&nbsp;[uf-releases-0.7.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.7.x)<br>
0.8.x:&nbsp;[uf-releases-0.8.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.8.x)<br>
master:&nbsp;[uf-releases-master](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-master)<br>

Each view has different jobs, for releasing this job should be run:<br><br>
**010.Releasing_Uberfire_Uberfire-extensions_deploy_\<branch\>**<br>

Basically this job<br>
- clones the repository<br>
- creates a release branch<br>
- changes version on this release branch<br>
- builds & deploys the repository locally<br>
- uploads the binaries to Nexus<br><br>
- pushes this release branches to droolsjbpm (community) or jboss-integration (product)<br>

This job is named *`010.Releasing_Uberfire_Uberfire-extensions_deploy_<branch>`* for 0.7.x and 0.8.x, for master it is named *`010.Releasing_Uberfire_deploy-master`*.<br>

This job runs **parametrized**.<br><br>
**TARGET**: community or productized<br>
**RELEASE_BRANCH**: name of the release branch that should be created<br>
**oldVersion**: the version the module has before changing the version (only available in 0.7.x and 0.8.x)<br>
**newVersion**: the version the poms will be upgraded to after the version change<br>
In job *`010.Releasing_Uberfire_deploy-master`* the parameter **`oldVersion`** was removed.

When the job finished a kie-staging-repository of Uberfire (and Ubefire-extensions for 0.7.x and 0.8.x) will be created and closed on [Nexus](https://repository.jboss.org/nexus/index.html#stagingRepositories)

Releasing Dashbuilder
---------------------
For Dashbuilder there exist two diffrerent views on [Jenkins CI](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com) depending on the branch ew want to release:

0.3.x and 0.4.x branch:&nbsp;[dashbuilder-releases 0.3.x and 0.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/dashbuilder-releases%200.3.x%20and%200.4.x/)<br>
master:&nbsp;[dashbuilder-releases-master](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/dashbuilder-releases%20master/)

Basically this job<br>
- clones the repository<br>
- creates a release branch<br>
- changes version on this release branch<br>
- builds & deploys the repository locally<br>
- uploads the binaries to Nexus<br><br>
- pushes this release branches to droolsjbpm (community) or jboss-integration (product)<br>

This job is named *`020.Releasing_Dashbuilder_deploy`* for 0.3.x and 0.4.x branch and *`020.Releasing_Dashbuilder_deploy-master`* for master. <br>

This job runs **parametrized**.<br><br>
**TARGET**: community or productized<br>
**BASE_BRANCH**: could be 0.3.x or 0.4.x (for *`020.Releasing_Dashbuilder_deploy-master`* the BASE_BRANCH is master and this parameter was removed)<br>
**RELEASE_BRANCH**: name of the release branch that should be created<br>
**oldVersion**: the version the module has before changing the version<br> (or *`020.Releasing_Dashbuilder_deploy-master`* this parameter was removed)
**newVersion**: the version the poms will be upgraded to after the version change<br>
**UBERFIRE_VERSION**: the version of Uberfire used by Dashbuilder
**ERRAI_VERSION**: the version of Errai used by Dashbuilder


When the job finished a kie-staging-repository of Dashbuilder will be created and closed on [Nexus](https://repository.jboss.org/nexus/index.html#stagingRepositories)

KIE releases
=============
For releasing the KIE repositories the exist two different views on [Jenkins CI](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com) depending on the branch ew want to release:

6.3.x:&nbsp;[kie-releases-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/)<br>
6.4.x:&nbsp;[kie-releases-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/)

For coming up 7.0.x branch (now master) there will be generated a new view. This is not yet existing.<br>
Both views have different jobs to:<br>
- clone the repository<br>
- create a release branch<br>
- change version on this release branch<br>
- build & deploy the repository locally<br>
- upload the binaries to Nexus<br><br>
- push the release branches to droolsjbpm (community) or jboss-integration (product)<br>
- update kie repositories to next development version

Push KIE release branches
-------------------------
The jobs [030.Releasing_KIE_push-release-branches-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/030.Releasing_KIE_push-release-branches-6.3.x/) or [030.Releasing_KIE_push-release-branches-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/030.Releasing_KIE_push-release-branches-6.4.x/)
or create the release branch based on a branch or previous tag,<br> upgrade its version and pushes the branch to community or productization.<br>


This job runs **parametrized**:<br><br>
**TARGET**:community or productized<br>
**SOURCE**:could be the community-branch (6.3.x. or 6.4.x, depending on selected job as it is hard coded), a previous tag of community or productiaztion<br>
**TAG**:only to edit if **SOURCE** is a previous tag<br>
**RELEASE_VERSION**:the version the poms will be upgraded to after the version change<br>
**RELEASE_BRANCH**:name of the release branch that should be created<br>
**UBERFIRE_VERSION**:the version of Uberfire used by KIE<br>
**DASHBUILDER_VERSION**:the version of Dashbuilder used by KIE<br>

Right now the ERRAI version on mastrer is a -SNAPSHOT version whereas the ERRAI version on 6.3.x. and 6.4.x is \*.Final version.<br>
When creating a new branch based on master (7.0.0) this will be changed since in community builds -SNAPSHOTs can't be used.<br>

Deploy locally kie
------------------
The jobs [031.Releasing_KIE_deployLocally-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/031.Releasing_KIE_deployLocally-6.3.x/), [031.Releasing_KIE_deployLocally-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/031.Releasing_KIE_deployLocally-6.4.x/) and [031.Releasing_KIE_deployLocally-master](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/031.Releasing_KIE_deployLocally-master/)<br>
fetch the just created and pushed *release branches* from community or jboss-integration and build and deploy them locally. This means the artifacts are not deployed directly to Nexus but rather
to a local directory on the server. (This saves a lot of time).<br>


This job runs **parametrized**.<br>
**TARGET**:community or productized<br>
**RELEASE_BRANCH**:name of the release branch that should be fetched for this release<br>

These jobs run the build and deploy. Looking at the Jenkins UI it can be controlled if the build failed and the if the unit tests run or failed.<br>

Sanity checks
-------------
Once the build has concluded these three artifacts have to be downloaded<br>
[drools-distribution](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org/drools/drools-distribution) /\<version\>/drools-distribution-\<version\>.zip<br>
[droolsjbpm-integration-distribution](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org/drools/droolsjbpm-integration-distribution) /\<version\>/droolsjbpm-integration-distribution-\<version\>.zip<br>
[optaplanner-distribution](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org/optaplanner/optaplanner-distribution) /\<version\>/optaplanner-distribution-\<version\>.zip<br>
These zips should be unzipped. This will create a dirctory (drools-distribution, droolsjbpm-integration-distribution and optaplanner-distribution). In these directories are the folders /examples. In each emaple folder there is
a runExamples.sh that should be executed.<br>
When running the JVM will pop up small menu windows. All menu options should be tested.<br>
These tests normally should only be run when a community release it done.

Another sanity check is simply to verify if the `kie-wb-<version>-<container>.war` or `kie-droolswb-<version>-<container>.war` is deployable in different containers.<br>
The wars can be downloaded at<br>
**PRODUCT:**<br>
[kie-wb](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org/kie/kie-wb-distribution-wars) /\<version\>/
kie-wb-distribution-wars-\<version\>-\<container\>.war<br>and<br>
[kie-drools-wb](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org/kie/kie-drools-wb-distribution-wars) /\<version\>/kie-drools-wb-distribution-wars-\<version\>-\<container\>.war
<br>or<br>
**COMMUNITY:**<br>
[kie-wb](https://origin-repository.jboss.org/nexus/content/groups/kie-group/org/kie/kie-wb-distribution-wars) /\<version\>/
kie-wb-distribution-wars-\<version\>-\<container\>.war<br>and<br>
[kie-drools-wb](https://origin-repository.jboss.org/nexus/content/groups/kie-group/org/kie/kie-drools-wb-distribution-wars) /\<version\>/kie-drools-wb-distribution-wars-\<version\>-\<container\>.war

Cherry-picking
--------------
Once the first check (sanity checks, smoke tests) was done it should be considered that some bugs will be discovered. These bugs will be fixed and then commited to the correspondent upstream branches. 
Thus these commits have to be fetched and rebased to the respective release branches.<br>
When a cherry-pick **X** on the release-branch **RB** of a commit **ID** from branch **B** in repository **R** has to be done::<br>
When a commit **ID** of repository **R** on a branch **B** has to be cherry-picked to the release-branch 

    > cd directory R
    > git checkout branch B
    > git fetch origin (or however is called the remote)
    > git rebase origin/B B
    > git log -5 --pretty=oneline (to get the commitId to cherry-pick)
    > git checkout RB
    > git cherry-pick -x ID
    > git push "remote" RB (where remote is droolsjbpm or jboss-integration)

Often it is better and more easy to override the whole release-branch instead if cherry-picking. Since the release-branches are on droolsjbpm or jboss-integration it is easy to override them.<br>
When the whole branch should override the existing branch:<br>

    > cd directory R
    > git fetch "remote" (however is called the remote)
    > git rebase "remote"/B B
    > git checkout RB
    > git rebase B (as the last commit on RB is the commit that upgrades the version, with a rebase this will be the last commit in history)
    > git push "remote" RB
    
Depending on the selected procedure to cherry-pick or override the realease branches on droolsjbpm (community) or jboss-integration (prod) will be updated.<br>
If sanity testing, smoke testing (people of KIE team) and cherry-picks were done the step of [deploying locally](#deploy-locally-kie) has to be repeated.
          
Copy deployed KIE binaries to Nexus
-----------------------------------
If finally the build was successful, the artifacts were deployed locally and the unit tests executed during the build run without failures the artifacts could be copied to [Nexus](https://repository.jboss.org/nexus/index.html#stagingRepositories)
running this script:<br>

6.3.x:&nbsp;[032.Releasing_KIE_copy-deployed-directory-to-Nexus-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/032.Releasing_KIE_copy-deployed-directory-to-Nexus-6.3.x/)<br>
6.4.x:&nbsp;[032.Releasing_KIE_copy-deployed-directory-to-Nexus-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/032.Releasing_KIE_copy-deployed-directory-to-Nexus-6.4.x/)<br>
<br>
These jobs copy the locally deployed artifacts **(Deploy_dir)** to Nexus.<br>
These jobs run **parametrized**:<br><br>
**TARGET**:community or productized<br>

The artifacts could be downloaded here:<br>
community:&nbsp;[KIE-group](https://origin-repository.jboss.org/nexus/content/groups/kie-group/org/)<br>
product:&nbsp;[KIE-internal-group](https://origin-repository.jboss.org/nexus/content/groups/kie-internal-group/org)<br>
The artifacts could be downloaded here for people doing some smoke tests. In addition, when the artifacts were uploaded correctly, there will be run the following tests. 

Additional test coverage
------------------------
The previous job triggers the following listed additional tests
 
### jbpm test coverage
6.3.x:&nbsp;[032a.Releasing_KIE_kie-all-jbpm-test-coverage-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/032a.Releasing_KIE_kie-all-jbpm-test-coverage-6.3.x/)<br>
6.4.x:&nbsp;[032a.Releasing_KIE_kie-all-jbpm-test-coverage-matrix-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/032a.Releasing_KIE_kie-all-jbpm-test-coverage-matrix-6.4.x/)<br>

These tests execute tests on jbpm source code (integration tests).<br>
Whereas on 6.3.x the test are only executed with JAVA 1.6, in 6.4.x they are executed with JAVA 1.6, 1.7 and 1.8.

### kie api backwards compatible check
6.3.x:&nbsp;[032c.Releasing_KIE_kie-all-kie-server-matrix-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/032c.Releasing_KIE_kie-all-kie-server-matrix-6.3.x)<br>
6.4.x:&nbsp;[032c.Releasing_KIE_kie-all-kie-server-matrix-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/032c.Releasing_KIE_kie-all-kie-server-matrix-6.4.x)<br>

Checks a backward compatibility of the KIE API. By default, the current build is checked against the last released (non-SNAPSHOT) version.<br>
The check can't be added directly to the default builds as it requires Java 8, to run.<br>
Once KIE moves to JDK 8 for builds, the check should be moved directly to the default build.<br>

###KIE server martix
1<br>
2<br>
3<br>

###KIE workbench smoke tests matrix
1<br>
2<br>
3<br>

Post release actions
====================
When all previous listed steps have been fulfilled and there are no concerns about the upcoming release/tag the following steps have to be undertaken:<br>

Push tags
---------
Once the release is finished, the artifacts are copied to staging repositories on Nexus and the tags are created, they have to be pushed to `community:droolsjbpm` or `product:jboss-integration`.

###Push Uberfire tag
There are two scripts for pushing the Uberfire/Uberfire-extensions tag:<br>
0.7.x:&nbsp;[011.Releasing_Uberfire_Uberfire-extensions_push-tag_0.7.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.7.x/job/011.Releasing_Uberfire_Uberfire-extensions_push-tag_0.7.x/)<br>
0.8.x:&nbsp;[011.Releasing_Uberfire_Uberfire-extensions_push-tag-0.8.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/uf-releases-0.8.x/job/011.Releasing_Uberfire_Uberfire-extensions_push-tag-0.8.x/)<br>

This job runs **parametrized**.<br>

**TARGET**:community or productized<br>
**RELEASE_BRANCH**:name of the release branch that should be fetched for this tag<br>
**TAG**:name of the tag<br>

###Push Dashbuilder tag
There is one script for pushing the Dashbuilder tag:<br>
0.3.x and 0.4.x:&nbsp;[021.Releasing_Dashbuilder_push-tag](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/dashbuilder-releases%200.3.x%20and%200.4.x/job/021.Releasing_Dashbuilder_push-tag)<br>

This job runs **parametrized**.<br>

**TARGET**:community or productized<br>
**RELEASE_BRANCH**:name of the release branch that should be fetched for this tag<br>
**TAG**:name of the tag<br>

###Push KIE tags
There are two scripts for pushing the KIE tags:<br>
6.3.x:&nbsp;[033.Releasing_KIE_push-tags-6.3.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.3.x/job/033.Releasing_KIE_push-tags-6.3.x/)<br>
6.4.x:&nbsp;[033.Releasing_KIE_push-tags-6.4.x](https://kie-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/view/kie-releases-6.4.x/job/033.Releasing_KIE_push-tags-6.4.x/)<br>

This job runs **parametrized**.<br>

**TARGET**:community or productized<br>
**RELEASE_BRANCH**:name of the release branch that should be fetched for this tag<br>
**TAG**:name of the tag<br>


Release staging repositories
----------------------------
Once the tags are pushed 

Community
---------
1<br>
2<br>
3<br>

###Update to next development version
1<br>
2<br>
3<br>

####Update Uberfire
1<br>
2<br>
3<br>

####Update Dashbuilder
1<br>
2<br>
3<br>

####Update KIE
1<br>
2<br>
3<br>
  
###Copy community Artifacts
1<br>
2<br>
3<br>

###Run scripts for latest
1<br>
2<br>
3<br>
  
###Enter release date on JIRA
1<br>
2<br>
3<br>
  
Product
-------
In most cases if there is a new community release it has to be build also a tag for productization. Since the release procedure is the same and script driven the same scripts have to run. 
The tag for productization can be based on a community branch, a community tag or a previous productization tag. All these possibilities are taken into account. <br>
Also, when reached community **\*.Final** productization builds a product version. After this productization could ask for Update patches. 

###Integration Pack
1<br>
2<br>
3<br>

###Hand off report
1<br>
2<br>
3<br>  
  


 


