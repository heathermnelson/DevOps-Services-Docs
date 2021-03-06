#Build and deploy

Last modified: 9 July 2015

The IBM® Bluemix™ DevOps Services Build & Deploy feature, also known as the pipeline, automates the continuous deployment of your projects. In a project's pipeline, sequences of stages retrieve input and run jobs, such as builds, tests, and deployments.

---
* [Stages](#stages)
* [Jobs](#jobs)
* [Manifest files](#manifests)
* [An example pipeline](#example)
* [Adding a stage](#add_stage)
* [Adding a job to a stage](#add_job)
* [Running a stage](#run_stage)
* [Deploying an app](#deploy)
* [Viewing logs](#logs)

---

<a name="stages"></a>
##Stages

Stages organize input and jobs as your code is built, deployed, and tested. Stages accept input from either source control repositories or build jobs in other stages. When you create your first stage, the default settings are set for you on the **INPUT** tab.

A stage's input is passed to the jobs it contains, and each job is given a clean container to run in. The jobs in a stage can't pass artifacts to each other. However, you can define stage environment properties that can be used in all jobs. For example, you could define a `TEST_URL` property that passes a single URL to deploy and test jobs in a single stage. The deploy job would deploy to that URL, and the test job would test the running app at the URL. 

To learn how to add a stage, [see Adding a stage][19].

By default in a stage, builds and deployments are triggered automatically every time changes are delivered to a project's source control repository. Stages and jobs run serially; they enable flow control for your work. For example, you might place a test stage before a deployment stage. If the tests in the test stage fail, the deployment stage won't run. 

You might want tighter control of a specific stage. If you do not want a stage to run every time a change occurs at its input, you can disable the capability. On the **INPUT** tab, in the Stage Trigger section, click **Only execute jobs when a user manually runs this stage**. 



![The INPUT tab][8]

<a name="jobs"></a>
##Jobs
A job is an execution unit within a stage. A stage can contain multiple jobs, and the jobs in a stage run sequentially. By default, if a job fails, subsequent jobs in the stage do not run.

![Build and test jobs within a stage][15]

Jobs run in discrete working directories that are created by the pipeline. Before a job is run, its working directory is populated with input that is defined at the stage level. For example, you might have a stage that contains a test job and a deploy job. If you install dependencies on one job, the dependencies are not available to the other job. However, if you make the dependencies available in the stage's input, they are available to both jobs.

To learn how to add a job to a stage, [see Adding a job to a stage][20].

<a name="builds"></a>
###Build jobs

Build jobs compile your project in preparation for deployment. 

**Note**: If you select the **Simple** builder type for a build job, you skip the build process. In that case, your code is not compiled, but is sent to the deployment stage as is. To both build and deploy, select a builder type other than **Simple**. 

<a name="builds_var"></a>
####Environment variables for build scripts
You can include environment variables within a build job's build shell commands. The variables provide access to information about the job's execution environment.

| Environment variable  | Description  |
|---|---|
| BUILD_NUMBER  | The current build job number.  |
| ARCHIVE_DIR  | The current build job's build artifact directory.   |

<a name="deploys"></a>
###Deploy jobs

Deploy jobs upload your project to Bluemix as an app and are accessible from a URL. After a project is deployed, you can find the deployed app on your Bluemix Dashboard. You can configure the build and deploy jobs as separate stages or add them to the same stage to run automatically.

<a name="deploys_var"></a>
####Environment variables for deployment scripts

You can include environment variables within a deploy job's deployment script. These variables provide access to information about the job's execution environment.

| Environment variable  | Description  |
|---|---|
| BUILD_NUMBER  | The current deploy job number.  |
| CF_APP  | The app name. This is required for deployment and can be specified in the script itself, the deploy job configuration interface, or the project's `manifest.yml` file.  |
| CF_ORG  | The targeted organization (org).  |
| CF_SPACE  | The targeted space within the supplied org.  |

<a name="tests"></a>
###Test jobs
If you want to require that certain conditions are met, include test jobs before or after your build and deploy jobs. Test jobs are highly customizable. For example, you might run tests on your project code and a deployed instance of your app. 

<a name="manifests"></a>
##Manifest files
Manifest files, which are named `manifest.yml` and stored in a project's root directory, control how your project is deployed to Bluemix. For information about creating manifest files for a project, [see the Bluemix documentation about application manifests][1]. To integrate with Bluemix, your project must have a manifest file in its root directory. However, you are not required to deploy based on the information in the file. 

In the pipeline, you can specify everything that a manifest file can by using `cf push` command arguments. The `cf push` command arguments are helpful in projects that have multiple deployment targets. If multiple deploy jobs all try to use the route that is specified in the project manifest file, a conflict occurs. 

To avoid conflicts, you can specify a route by using `cf push` followed by the host name argument, `-n`, and a route name. By modifying the deployment script for individual stages, you can avoid route conflicts when you deploy to multiple targets.

To use the `cf push` command arguments, open the configuration settings for a deploy job and modify the **Deploy Script** field. For more information, [see the Cloud Foundry Push documentation][3]. 

<a name="example"></a>
##An example pipeline

The simplest possible pipeline contains two stages. First, there is a stage that contains a build job and takes input from a Git repository. When that build job runs, the app is built and sent to a build archive directory. If the first stage runs, a second stage that contains a deploy job runs. The second stage takes input from the earlier build job and deploys the app to Bluemix.

![A two-stage pipeline][12]

---

<a name="add_stage"></a>
##Adding a stage

1. On the Build & Deploy Pipeline page, click **ADD STAGE**. The Stage Configuration page opens. 
2. Configure the stage.
  1. On the **INPUT** tab, select an input for the stage.
  2. On the **JOBS** tab, add and configure at least one job. The first stage usually has at least a build job.
3. Click **SAVE**.

![Adding a stage to a pipeline][13]

<a name="add_job"></a>
##Adding a job to a stage

1. On the stage, click the **Stage Configuration** icon, and then click **Configure Stage**. 
2. Click the **JOBS** tab.
3. Click **ADD JOB**. Select the type of job to add. 
4. Configure the job.
5. Click **SAVE**.

![Adding a job to a stage][14]

<a name="run_stage"></a>
##Running a stage

You can manually run a stage by clicking the **Run Stage** icon on the Build & Deploy Pipeline page. 

![Clicking the Run Stage icon on a stage][16]

You can also request on-demand builds and deployments from the build history page in one of two ways:
* Drag a build to the box that is under a configured stage.
* Next to a build, click the **Execute stage with this build** icon and then select a space to deploy to.
  ![The Execute stage with this build icon][9]
  
<a name="deploy"></a>
##Deploying an app

A properly configured deploy job deploys your app to your target whenever the job is run. To manually run a deploy job, click the **Run Stage** icon of the stage that the job is in.

When you run a stage manually, or if it runs because the stage before it is completed, the running stage selects its input revision. Usually, the input revision is a build number. To select the input revision, the stage follows this process:

1. If a specific revision is selected, use it.
2. If a specific revision is not specified, search previous stages until a stage is found that uses the same input. Find and use the last successfully run revision of that input.
3. If a specific revision is not specified and no other stages use the specified source as input, use the latest revision of the input.

<a name="logs"></a>
##Viewing logs

You can view the logs for your jobs on the Stage History page. 

To view a job's log, click the job. Alternatively, on a stage, click **View logs and history**.

To view the runtime log, click **View runtime log**.

![Areas in a stage tile that can be clicked to open relevant logs][10]

In addition to job logs, you can view unit test results, generated artifacts, and code changes for any build job.

 
[1]: https://www.ng.bluemix.net/docs/#manageapps/index-gentopic2.html#appmanifest
[2]: https://www.ng.bluemix.net/docs/#services/DeliveryPipeline/index.html#getstartwithCD
[3]: http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#push
[4]: https://console.ng.bluemix.net/?ace_base=true/#/pricing/cloudOEPaneId=pricing
[5]: ./images/open_logs.png
[6]: #manifests
[7]: ./images/runbar-annotated-dark.png
[8]: ./images/input_tab_only_execute.png
[9]: ./images/deploy_to.png
[10]: ./images/view_logs_and_history.png
[11]: ./images/play_button.png
[12]: ./images/basicAnimate.gif
[13]: ./images/AddStage.png
[14]: ./images/AddJob.png
[15]: ./images/jobs.png
[16]: ./images/RunStage.png
[17]: https://www.ng.bluemix.net/docs/starters/container_pipeline.html#container_pipeline
[18]: ../../../tutorials/basicbuild
[19]: #add_stage
[20]: #add_job
