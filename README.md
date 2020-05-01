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
### Google App Engine
First of all let's set the region in which we will be working, in this case europe-west2 located in [London](https://cloud.google.com/compute/docs/regions-zones#locations).

- `gcloud app create --region=europe-west2`

Now we have initialized the App Engine app, but we need to deploy before it really works. We will use one of the code samples provided by Google.

- `git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git`
- `cd python-docs-samples/appengine/standard/hello_world/`
- `gcloud app deploy --version first --quiet` This will deploy a service named "default" and its version will be named "first. This version will be hosting our production environment.
- `gcloud app browse` will show the URL in which our webpage is serving. URL should look like `$PROJECT_ID.nw.r.appspot.com`

Pushing the code to our repository, but first let's create a folder to copy the data in.

- `mkdir production`
- `cp -r python-docs-samples/appengine/standard/hello_world/ production/`
- `git add production/`
- `git commit -m "Pushing to production"`
- `git push`

Now we have our webpage serving in `$PROJECT_ID.nw.r.appspot.com` and the code in the Github repo, at our master branch.
