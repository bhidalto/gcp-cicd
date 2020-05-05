# gcp-cicd
Simple demo of Cloud Build usage to demonstrate how to implement CI/CD using GCP with git. This repository will cover several topics, including the usage of CI/CD with github repositories as well as GitHub versioning, everything based on Google Cloud Platform products.

Main used products will be Google App Engine that will host our dummy webpage and Cloud Build to automatically deploy changes whenever a change is commited to the git repository. This will allow us to fully-automate the deployment of newer versions as soon as any changes arise and abstract the process of deploying to the developer, whose main focus will be on code-related tasks.

We will also cover the topic of controling different environments (production and development) having them separated in different branches and Cloud Build will be listening on each of them with the usage of Cloud Build triggers in order to deploy different Google App Engine services, each for one of our environments.

# Setting up the GitHub repository

We will start by setting up the GitHub repository. Our master branch will act as Production environment and we will create a separate branch to host our Development environment.

First, let's head to GitHub and create the repository:

- Go to github.
- Log in to your account.
- Click the new repository button in the top-right. You’ll have an option there to initialize the repository with a README file, check it to have the repository initialized.
- Click the “Create repository” button.

Now that we've successfully created our repository, let's create a branch named "dev" from the Cloud Console Shell:

- `git clone https://github.com/$GIT_USERNAME/$REPO_NAME.git`
- `cd $REPO_NAME`
- `git checkout -b dev`
- `git push origin dev`

# Setting up the Google Cloud Platform environment
## Google App Engine
First of all let's set the region in which we will be working, in this case europe-west2 located in [London](https://cloud.google.com/compute/docs/regions-zones#locations).

- `gcloud app create --region=europe-west2`

Now we have initialized the App Engine app, but we need to deploy before it really works. We will use one of the code samples provided by Google.

- `git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git`
- `cd python-docs-samples/appengine/standard/hello_world/`
- `gcloud app deploy --version first --quiet` This will deploy a service named "default" and its version will be named "first. This version will be hosting our production environment.
- `gcloud app browse` will show the URL in which our webpage is serving. URL should look like `$PROJECT_ID.nw.r.appspot.com`

To end up we will need to enable the App Engine API, from the Cloud Shell as well run the following:

`gcloud services enable appengine.googleapis.com`

Pushing the code to our repository, but first let's create a folder to copy the data in.

- `mkdir production`
- `cp -r python-docs-samples/appengine/standard/hello_world/ production/`
- `git add production/`
- `git commit -m "Pushing to production"`
- `git push`

Now we have our webpage serving in `$PROJECT_ID.nw.r.appspot.com` and the code in the Github repo, at our master branch.

## Connecting GitHub to GCP and creating the Cloud Build trigger
### Connecting GitHub to GCP

This operation needs to be done using the Cloud Console UI, as we will find in the [documentation](https://cloud.google.com/cloud-build/docs/running-builds/create-manage-triggers#connecting_to_source_repositories), following the next steps:

- Navigate to `Cloud Build > Triggers` in the Cloud Console menu 
- Click `Connect Repository`
- Select `GitHub (Cloud Build GitHub App)` as source
- Authenticate with your GitHub account
- Select the repository to be linked.
- Skip last part that will create the trigger from the UI as we will create it from the Cloud Shell in the following step, making use of the Gcloud SDK.

### Creating the Cloud Build Trigger
At this point, we already have our repository linked to GCP, but we don't have a way to communicate GCP what should be done whenever a push action is done to our repo. Let's proceed to create a Push Trigger in our Cloud Build trigger in order to automatically deploy to Google App Engine whenever we commit changes. From the Cloud Shell, run the following command:

```    
    gcloud beta builds triggers create github \
    --repo-name=[REPO_NAME] \
    --repo-owner=[REPO_OWNER] \
    --branch-pattern="master" \  #We only want to listen to changes made to master branch
    --build-config=[BUILD_CONFIG_FILE]" #In this case our file will be named cloudbuild.yaml, located at "production/hello_world/cloudbuild.yaml"
```

The trigger gets automatically created, however we don't have the `cloudbuild.yaml` created in our repository and therefore our pipeline will break. Moving on, let's proceed to create the `cloudbuild.yaml` and push it to our repository.

- `touch cloudbuild.yaml`
- Modify the yaml and add the following:
  ```
    steps:
    - name: "gcr.io/cloud-builders/gcloud"
      args: ["app", "deploy", "--version", "first", "production/hello_world/app.yaml"]
  ```
Pushing the changes to GitHub (this time we want to [skip](https://cloud.google.com/cloud-build/docs/running-builds/create-manage-triggers#skipping_a_build_trigger) triggering the Build, as for now we're not interested in re-deploying our code):

- `git add cloudbuild.yaml`
- `git commit -m "[skip ci]"`
- `git push`

Before anything else, let's make sure that our Cloud Build service account has the needed permissions to deploy to App Engine. To do that, run the following commands from the Cloud Shell:

```
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com --role=roles/appengine.deployer
```
App Engine deployer permission is needed to create new [versions](https://cloud.google.com/appengine/docs/admin-api/access-control#app-engine-deployer).
```
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com --role=roles/appengine.serviceAdmin
```
#Service Admin permission will alow our pipeline to [migrate traffic](https://cloud.google.com/appengine/docs/admin-api/access-control#app-engine-service-admin) between versions.

Following up, we will modify our App Engine code and push it to the repo. This time we will make the trigger build and have our application be re-deployed. If everything has been set up correctly, our application will be automatically updated and the GAE will be serving the latest changes commited to the repository. Let's go ahead an do a simple modification to the main.py:

`
import webapp2


class MainPage(webapp2.RequestHandler):
    def get(self):
        self.response.headers['Content-Type'] = 'text/plain'
        self.response.write('Hello, World! This is our production environment!')
`
- `git add main.py`
- `git commit -m "First production trigger!" `
- `git push`
