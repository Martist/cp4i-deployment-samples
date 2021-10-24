# Attribution

 All credits for the demononstration itself and the scripts and components used for the installation goes to the original creators of the [CP4I Deployment Samples](https://github.com/IBM/cp4i-deployment-samples).

This repository contains several changes to implementsonly the "dev" part of Driveway Dent Deletion demo. Scripts available on the original repository were changed to run minimalistic version of the demonstration without the need to deploy the API Connect component, which takes a lot of resources and is not necessary to achieve the required results - to confirm that the CP4I was deployed sucessfully and that the deployed components are working. Also the scripts were edited to run on CP4I version 2021.3.

# Demonstration Set-Up
1. Clone the [repository](https://github.com/Martist/cp4i-deployment-samples.git) with changed installation scripts to a machine with access to your OpenShift cluster

2. Login to your OpenShift cluster and switch to a project for the demonstration (default project for the demo installation is 'cp4i')

3. navigate to `cp4i-demos-operator-custom/DrivewayDentDeletion/Operators` directory

4. Run the installation:

    `../../products/bash/install-ddd-dev-custom.sh -i aro-ddd-demo.yaml -o output-install-aro.yaml`

    and watch the screen for the installation status. 

    **Note**: when the installation reaches the step to release postgres (`[INFO] Releasing postgres in the 'cp4i' namespace...`) and it takes suspiciously long, check your openshift cluster to see if there are some issues with the posgresql pod. If the pod is not coming up for a long time, check the Deployment Config named postgresql, look into it's YAML file and find `spec.triggerrs.imageGhangeParams.from.name` property - it should have `postgresql:10` value. 
    
    Now go to Builds > ImageStreams in your OCP cluster and switch to `openshift` project. Search for `postgresql` and open the corresponding ImageStream. On the bottom of the page, you should see a list of available tags - see if the `postgresql:10` tag is available - if not, take whatever tag you found for postgresql 10 (like 'postgresql:10-e18') and put it to the `spec.triggerrs.imageGhangeParams.from.name` property of the Deployment Config mentioned above. This should resolve the postgresql pod not coming up. 

    [ImageStreams tags available for PostgreSQL in OCP 4.8](./img/postgresql-image-streams.png)

    [Change the property in postgresql Deployment Config's YAML file](./img/postgresql-dc-yaml.png)

    After a while, all components should be installed and you should receive a SUCCESS note. 

5. Open the Operations Dashboard instance that was installed as part of the script. On the left menu, navigate to Manage > Registration requests - there should be a request waiting for processing. Process it and complete the required task described on the screen. 

6. Execute the pipeline creation:

    `./cicd-apply-dev-pipeline.sh -n cp4i -r https://github.com/Martist/cp4i-deployment-samples.git -b main -f ocs-storagecluster-cephfs -g occluster-ceph-rbd`

        PARAMETERS:
        -n : <NAMESPACE> (string), Defaults to 'cp4i'
        -r : <REPO> (string), Defaults to 'https://github.com/IBM/cp4i-deployment-samples.git' - should point to a publicly available repository with the demo installation files
        -b : <BRANCH> (string), Defaults to 'main'
        -f : <DEFAULT_FILE_STORAGE> (string), Default to 'ibmc-file-gold-gid'
        -g : <DEFAULT_BLOCK_STORAGE> (string), Default to 'cp4i-block-performance'

    After the pipeline is deployed, you can run it directly from the Pipeline detail, by hitting Actions > Start. 

        PARAMETERS:
            imageTag - choose any tag for the images created during the pipeline run
        WORKSPACES:
            git-source - choose PVC - git-source-workspace PVC 

    When the pipeline finishes, the demo is installed. 

## Validation
To validate the demo installation, run the test script:

`./test-api-e2e.sh -n cp4i -s ddd`

    PARAMETERS:
        -n <NAMESPACE> namespace where the demo was installed 
        -s <SUFFIX> suffix for installed demo components. Use 'ddd' as it's a hardcoded to the scripts default. 

After running the testing script, you can visit the Operations Dashboard and you should see new tracing details.