## Setup ROKS Ingress for Refund Request use case

By default, OpenShift environments based on the demo pattern and running on IBM Red Hat OpenShift on IBM Cloud (ROKS) will not have any trusted ingress or certificates for Cloud Pak for Business Automation (CP4BA) routes and services.  This means that workflows (such as those running in Workflow Authoring in Business Automation Studio) will not be able to successfully connect to ODM services.

The following is one option to ensure the ODM certificates (and some others of CP4BA) are trusted by browsers and by Workflow Authoring.  Additional networking configurations may also be supported but have not been tested.

### OpenShift web console

1. Open the OpenShift web console using the button in the upper right of your ROKS cluster page in IBM Cloud console.
1. Navigate to Home -> Projects -> openshift-ingress
1. Navigate to Workloads -> Secrets and locate the secret with the name equal to your cluster hostname (such as `dteroks-0600004t7y-ycnqs-4b4a324f027aea19c5cbc0c3275c4656-0000.us-south.containers.appdomain.cloud` or `ibmcloud-roks-83eow-4b4a324f027aea19c5cbc0c3275c4656-0000`)
1. Extract/copy tls.key and save to a new file (be careful not to type any other character in the file)
1. Repeat for tls.crt in a second new file
1. Navigate to Home -> Projects and click the name of your CP4BA project (such as `dtecp4ba`)
1. Create a new secret named `cp4a-wildcard` of type `tls` and set `cert` to `tls.crt` and `key` to `tls.key` (using the CLI: `oc create secret tls cp4a-wildcard --cert tls.crt --key tls.key`)
1. Navigate to Operators -> Installed Operators -> IBM Cloud Pak for Business Automation -> CP4BA Deployment tab -> `icp4adeploy` -> YAML tab (or if not using OLM, Administration -> Custom Resource Definitions -> ICP4ACluster -> Instances tab -> icp4adeploy -> YAML tab)
1. Carefully search for the `shared_configuration` section
1. Carefully add a new line below `shared_configuration` and indent two spaces
1. Carefully enter the following: `external_tls_certificate_secret: cp4a-wildcard`
1. Click Save
1. Wait for the operator to reconcile the CR which normally requires 30-120 mins
1. Verify the configuration is effective by navigating to an ODM route in a browser (such as `https://odm-decisionserverconsole-<namespace>.<cluster>.<region>.containers.appdomain.cloud`) and ensure it shows as secure with a trusted certificate (most browsers show a lock icon to the left of the URL)
