## Refund Request use case
**an IBM Cloud Pak for Business Automation use case demo**

***

**Use Case:** Straight-through processing

**Use Case Overview:** Imagine that a customer purchases your product, but is unfortunately unsatisfied and requests a refund. Today, your refund process has several manual steps. Managing high volumes of refunds that require manual investigation leads to slow average completion times, inconsistent status updates to customers, and higher costs for any errors made.
This use case demonstrates the use of automation in the form of straight-through-processing to save time, money, and customer frustration.

**Choose an option:**

  * **Cloud Pak for Business Automation as a Service demo environment (likely an IBMer):** your environment is predeployed, continue to the [Getting Started Lab](https://ibm-cloud-architecture.github.io/refarch-dba/use-cases/refund-req/#getting-started-lab).
  * **Install Yourself:** To deploy Refund Request on your own environment, and technical architecture information, continue reading.

### Architecture Diagram

The following diagram illustrates the products involved and the solution components when deployed in one possible hybrid cloud architecture:

 ![0](./images/comp-view-hybrid.png)

The following diagram illustrates the products involved and the solution components when deployed to an OpenShift based architecture:

 ![0](./images/comp-view-ocp.png)

### Environment

We assume the following products are installed, up and running:

* IBM Cloud Pak® for Business Automation version 21.0.2
    * Automation Foundation on OpenShift
    * Business Automation Applications (including Studio and App Engine) on OpenShift
    * Business Automation Navigator on OpenShift
    * Business Automation Workflow (BAW) on VMs or OpenShift
    * Operational Decision Manager (ODM) on OpenShift
    * Business Automation Insights (BAI) on OpenShift

### Deploy the artifacts

1. Determine your credentials
    1. If using Cloud Pak for Business Automation as a Service (CP4BAaaS):
        * You will use a single login to access BAS, BAW, ODM and BPC
        * For BAW and ODM, you will need to create a service credential/account under Access Management to connect in the external automation service and invoke the API to Rule Execution Server, make sure to give the service credential an ODM role the allows execution in the environment that will run the ODM rules
    1. If deploying Refund Request on your own OpenShift environment:
        * Make sure you have a login to all required components above
    1. If deploying Refund Request on your own OpenShift environment based on the demo pattern and running on IBM Red Hat OpenShift on IBM Cloud (ROKS):
        * Install the `oc` CLI from the **Client-side requirements** here: [V21.0.x](https://www.ibm.com/docs/en/cloud-paks/cp-biz-automation/21.0.x?topic=deployments-preparing-demo-deployment).  Note: all other **Client-side requirements** are otional for this install but recommended to manage the ROKS cluster.
        * Follow the steps on [Setup ROKS Ingress](setupROKSIngress.md) to ensure ODM can be invoked from workflows
1. Deploy ODM artifacts
    1. Login to Decision Center Business console
    1. On Library, click the import icon (Note: if you are upgrading an environment that has an earlier version of Refund Request installed, you can go into the existing decision service and import the below ZIP files within each branch, selecting to replace existing when prompted)
    1. Choose and upload `Refund Processing [main] YYYY.MM.DD_XX_PLATFORM.zip` (where platform is `saas-<version>` for CP4BAaaS or `ocp-demo-<version>` for deploying Refund Request on your own OpenShift demo pattern environment)
    1. Open the main branch, click Tests -> Test Suites, edit and adjust the server within each test suite if required and run the test suite to confirm
    1. In the main branch, click Deployments -> Configurations, edit and adjust the server target within each deployment configuration if required
    1. Deploy to your preferred Rule Execution Server such as Production in CP4BAaaS (Note: if you are upgrading an environment that has an earlier version of Refund Request installed, you should first delete the ruleapp deployed to RES, as the application is set to use V1.0 and V1.1)
    1. Return to the branch list and click the plus sign to create a new branch based on main named Reduce Manual Processing
    1. Import `Refund Processing [Reduce Manual Processing] YYYY.MM.DD_XX_PLATFORM.zip` to the new branch and choose to replace the existing elements
    1. Repeat the steps above to edit and fix servers in Test Suites and Deployment Configurations
    1. Deploy the new branch to Rule Execution Server as a second version
1. Deploy BAW artifacts
    1. If using CP4BAaaS: login to Workflow Center and navigate to Process Apps
    1. If deploying Refund Request on your own OpenShift environment: login to Workflow Authoring in Business Automation Studio and navigate to Business automations -> Workflows
    1. Import `Refund_Request - YYYY.MM.DD_XX.twx`
    1. Open the Refund Request process app / project and navigate to Process App Settings -> Servers
    1. Edit the settings for hostname, port, authentication and so forth for your ODM server
        1. If using CP4BAaaS: the hostname follows the pattern `odm-dev-<tenant_hostname>` with no `https://` at the beginning and the port is left blank
        1. If deploying Refund Request on your own OpenShift environment based on the demo pattern and running on IBM Red Hat OpenShift on IBM Cloud (ROKS): the hostname should be the ODM Decision Server Console route hostname with no `https://` at the beginning and the port is left blank
    1. Open the BAI Generators team and add an appropriate user to the team, generally your BAW user
    1. Create a new snapshot of the process application / project
    1. If more than one Workflow environment is present (such as CP4BAaaS with Development and Production or a custom OpenShift deployment with Workflow Authoring and Workflow Server), install the new snapshot to the Workflow Server connected to the required BAI emitters
        1. If using CP4BAaaS: this is Production by default
        1. If deploying Refund Request on your own OpenShift environment based on the demo pattern and running on IBM Red Hat OpenShift on IBM Cloud (ROKS): there is only one Workflow environment and no need to deploy
        1. If you are upgrading a Workflow environment that has an earlier version of Refund Request installed, go to the Process Admin Console -> Installed Apps and make the new snapshot just deployed the default version, optionally deactivating any old snapshots
1. Setup BAI data
    1. Login to Process Portal (Workplace is not supported for this step) with the user specified in the above BAI Generators team
    1. Click to start Generate Week 1 BAI Data RR v2 and wait for the spinner to complete in about 20 seconds
    1. Do the same for Generate Week 2 BAI Data RR v2
1. Deploy BPC artifacts
    1. Login to Business Performance Center
    1. If you are upgrading a Workflow environment that has an earlier version of Refund Request installed, you should delete the existing dashboards or ask the original owner to delete them
    1. Import two dashboards using `RefundRequest (RR) – Week 1 - YYYY.MM.DD_XX.json` and `RefundRequest (RR) – Week 2 - YYYY.MM.DD_XX.json`
    1. Adjust the monitoring sources to the deployed BAW snapshot for the dashboard and any charts that require it
    1. Fix any charts that do not have data; the most likely error requires a selection of `decimal (data > TG2 > pTime) – (float)` in the Data item field after the monitoring source is changed
    1. If using CP4BAaaS or another environment shared with multiple users: use the three dot menu next to each Refund Request dashboard and choose to **Share with everyone** so all other users can see the dashboard with read only access
1. Deploy Business Automation Studio artifacts
    1. If using CP4BAaaS
        1. Create an external automation service at Development -> Build -> Business Automation Studio -> Business automations -> Create -> External
        1. Create a connection to the Workflow Server in Production (URL pattern `https://<tenant_hostname>/dba/run`) and use a service credential
        1. Select the `Refund Request` process and `Invoke Refund Processing ODM` service flow
        1. Name the automation service `Refund_Request_Application_Services` (**Important**, this must be exact) and publish
        1. Export the external automation service as a ZIP from the Business automations -> Extenal section
    1. If deploying Refund Request on your own OpenShift environment:
        1. Publish the workflow project's snapshot in Business Automation Studio -> Business automations -> Workflow -> Refund Request -> Versions to make the automation services available to applications
    1. Import the Refund Request application in Business applications using `RefundRequest (RR) – App - YYYY.MM.DD_XX_PLATFORM.twx` (where platform is `saas-<version>` for CP4BAaaS or `ocp-demo-<version>` for deploying Refund Request on your own OpenShift demo pattern environment)
    1. No edit of the application should be required but if an edit is done, create a new snapshot
    1. Export the application from Business Automation Studio -> Business applications -> Refund Request -> Versions as a ZIP
1. Deploy Business Automation Navigator artifacts
    1. Login to Business Automation Navigator's admin desktop
        1. If using CP4BAaaS: Production -> Manage solutions -> Publish
        1. If deploying Refund Request on your own OpenShift environment: use your Navigator URL with `?desktop=appDesktop1` added to the end and use the menu to go to Administration
    1. Select Connections on the left, edit the Application Engine Connection (generally called `APPENGO`) and connect
        1. If using CP4BAaaS: import both the automation service and application ZIP files
        1. If deploying Refund Request on your own OpenShift environment: import only the application ZIP file
    1. Edit Details from the application's menu and add appropriate teams to the Permissions table, such as `#AUTHENTICATED-USERS` to make the app available to everyone
    1. Edit the desktop of your choice (generally `appDesktop1`) and on the Layout tab, select the application

### Refund Request application modes

The Refund Request application supports the following modes and is configured using the hidden control panel as mentioned in the next section:

* **Guided:** recommended for those first learning Cloud Pak for Business Automation
    * the application utilizes a single page with no additional windows/tabs
    * the page includes a welcome step followed by steps 1 through 5
    * each step includes getting started instructions, step completion check boxes, and a summary of what you learned
    * implementation: demoLevel=false, deepDive=false, showDecisionLabels=true, showControlPanel=false, showBPCDashboards=false (showBPCDashboards=true is only supported at this time within public trial tenants)
* **Live Walkthrough:** recommended for those lightly experienced with Cloud Pak for Business Automation and wishing to show it to someone else
    * the application utilizes a single page with no additional browser windows/tabs
    * the page includes multiple tabs for the storyboard, discovery map, diagram and steps 1 through 5 to run the demo
    * the steps do **not** have getting started instructions, step completion check boxes, or a summary of what you learned (but this can be activated as desired, see the Hidden Control Panel section below)
    * implementation: demoLevel=true, deepDive=false, showDecisionLabels=true, showControlPanel=false, showBPCDashboards=false
* **Technical Deep Dive:** recommended for those deeply experienced with Cloud Pak for Business Automation and wishing to show the various tools and interfaces to someone else
    * the page includes multiple tabs for the storyboard, discovery map and diagram
    * the application utilizes a launchpad which provides a simple method to open multiple browser windows/tabs to the various interfaces for each persona
    * the page does **not** include any steps, getting started instructions, step completion check boxes, or a summary of what you learned (but this can be activated as desired, see the Hidden Control Panel section below)
    * implementation: demoLevel=true, deepDive=true, showDecisionLabels=false, showControlPanel=true, showBPCDashboards=false

### Hidden Control Panel

The Refund Request application includes a hidden Control Panel that allows configuration of the application mode as described above.  Show the hidden Control Panel by scrolling to the bottom of any page and clicking the background just above the grey footer bar.  The Control Panel may be hidden again by clicking in the same location.


## Contributors
  * Lead content developer [Jeff Goodhue](https://www.linkedin.com/in/jeffreygoodhue/)
