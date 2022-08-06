## AutoIdle cuts your Heroku bill by auto-putting your staging and review apps to sleep

From simple to complex applications, Heroku stands out as a deployment choice for many developers. This is because with Heroku, getting an application up and running is a very simple procedure that abstracts the underlying infrastructure and its scaling needs. However, with more applications running on Heroku is a growing bill even when no traffic is being served. [AutoIdle](https://autoidle.com/) is a Heroku add-on  that helps cut your Heroku bill by automatically putting your staging and review apps to sleep when you don't need them. In this article, we shall see how we can install AutoIdle on a Heroku app and review apps in a pipeline and observe how much we save.

## How does AutoIdle work?
[AutoIdle](https://autoidle.com/) typically puts your apps to sleep after 30mins of inactivity and gets them up and running in under 10seconds of a new request. This way, you are not charged for the time the your apps are up but idle. With that stated…

### …let's get our hands dirty.
Let's create an app called *autoidle-saving* in a  Heroku pipeline. Starting from the *start* branch of [this repository](https://github.com/123MwanjeMike/autoidle-saving), we shall incrementally build our application to the state in the *main* branch; so fork the repository and go [here](https://medium.com/r/?url=https%3A%2F%2Fdashboard.heroku.com%2Fpipelines%2Fnew) to create a Heroku pipeline. You will be presented with the screen below.

![The newly created Heroku pipeline. It's name is saving0with-autoidle](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654638319/_RwlnL1ch.png)

Now select the *Settings* tab and under the *Connect to GitHub* option, search for the forked repository and connect it to the pipeline.

![Connecting the pipeline to a GitHub repository](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654640704/5k6hDtPF5.png)

After connecting the repository, the *Review apps* option should now be available. Click *Enable* to have review apps for Pull Requests.
**Note:** Check the *Wait for CI to pass* option if you want HerokuCI to run your tests before deploying to the review app. There is a detailed article on Heroku CI and how to use it [here](https://medium.com/r/?url=https%3A%2F%2Fdev.to%2Fmwanjemike%2Fbuild-a-ci-cd-pipeline-with-heroku-ci-3de9) that you can check out if you want to learn more about HerokuCI specifically.

![Enabling review apps for the pipeline](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654642316/8tjxfqHO7.png)

Return to the pipeline tab now and for each of staging and production sections, click ***Add app***, and click ***Create new app…*** to create a staging and production app respectively.

![Creating the staging app for autoidle-saving](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654644320/zjqgAC7Yd.png)

Your pipeline now looks like so

![The pipeline after adding a staging and production app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654645971/78jOBvxQZ.png)

Let us make one additional tweak on our two apps. We want the staging app to automatically deploy code in the develop branch and the production app that in main branch on every push to the respective branches of the connected GitHub repository.  So, starting with the staging app, click the *Configure automatic deploys…* option.

![autoidle-saving-staging app options](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654647664/UwtNUdNcBj.png)

You will be presented with a screen such as below. Select the develop branch and click ***Enable Automatic Deploys***

![Enabling automatic deploys on the staging app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654649124/GNLhSkvKh.png)

The same goes for the production application, only that for it we shall deploy the main branch.

![Enabling automatic deploys on the production app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654651326/rWhKZUxnEP.png)

The pipeline is now ready and we can make an addition to our app so as to set everything in motion.
Run the commands below to clone the forked repository and install the dependencies of the application.

```
$ git clone https://github.com/123MwanjeMike/autoidle-saving.git
$ cd autoidle-saving
$ npm install
```

Now to confirm that all is okay, run **npm start** in the terminal and you should have the output below

![Terminal output after running npm start](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654656933/KtnkB-FV_.png)

![Accessing our application through the browser at 127.0.0.1:3000](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654658369/uC4U_wzqZ.png)

Now let us deploy to our staging app. So back to the staging section of the pipeline, and  under the app options, click ***Deploy a branch…***

![staging app options](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654660407/3JOh1J0QK.png)

…and deploy the develop branch.

![Manually deploying the develop branch to staging app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654662268/NfmN-0v7Q.png)

Let us try to access our staging app in the browser.

![Develop branch successfully deployed](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654663827/o4K4x0N80.png)

We are now going to install AutoIdle on the staging app so that it is automatically put to sleep. So if you already [downloaded and installed](https://medium.com/r/?url=https%3A%2F%2Fdevcenter.heroku.com%2Farticles%2Fheroku-cli%23download-and-install) Heroku CLI on your computer, go to your terminal and login.

```
$ heroku login
```

Next, we shall add our staging app's git repository to the local repository. To get a Heroku app's git URL, select the app and go to its Settings tab. You should see the ***Heroku git URL*** listed under the app's information.

![Getting the git url of a  Heroku app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654665531/Nw7Jb-P6s.png)

Copy the link and add it as a remote repository

```
$ git remote add staging-app https://git.heroku.com/autoidle-saving-staging.git
$ git remote -v
```

You should see a list of your remote repositories in the terminal.

![Output after running git remote -v](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654667229/CriGvgXjr.png)

Now we shall add AutoIdle to our application.

```
$ heroku addons:create autoidle
```

![AutoIdle successfully added to the staging app](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654668885/gLgqrjs5O.png)

We shall also add AutoIdle to our review apps. So, create a new branch *heroku-config* and add  an ***app.json*** file, a manifest format for describing Heroku  web apps.

```
$ git checkout -b heroku-config
```

the *app.json* has the content below

```
{
   "environments": {
      "review": {
         "addons": ["autoidle:hobby"]
      }
   }
}
```

Now commit and push your changes to the GitHub repository.

```
$ git add app.json
$ git commit -m "AutoIdle configs for review apps"
$ git push -u origin heroku-config
```

Open a PR for the *heroku-config* branch against develop and merge it.

![Our pipeline after merging our PR](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654670491/zRD0MoUpu.png)

Now lets make a final addition to our app to have a review app and we compare the saving with AutoIdle. The endpoint is for the 'Hello World, happy saving!' message.

```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
    res.status(200).json({ message: `Welcome!, let's save with AutoIdle 🎊`});
});

app.get('/salutation/:name', (req, res) => {
    res.status(200).json({ message: `Happy saving ${req.params.name}.`});
});

app.get('/world', (req, res) => {
    res.status(200).json({ message: 'Hello World, happy saving!'});
});

app.get('*', (req, res) => {
  res.status(404).json({ message: 'Sorry, resource not found.' });
});

const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`Listening on port ${port}`));
```

Commit and push your changes.

```
$ git commit -am "hello-world"
$ git push -u origin hello-world
```

Open a PR for hello-world against develop and we should have a new review app for that.

![Hello-world review app running](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654676528/DYdbITSHU.png)

Now let's observe the saving on the [AutoIdle dashboard](https://app.autoidle.com/). In your terminal, run

```
$ heroku addons:open autoidle
```

You will be taken to the browser and presented with the screen below.

![AutoIdle dashboard with the staging and review apps running](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654680675/cMtwSG4mg.png)

We can see that all our apps are running and we  don't have any available saving data, so shall check again after some time has passed.

![AutoIdle dashboard with available apps stopped](https://cdn.hashnode.com/res/hashnode/image/upload/v1657654683186/YOa3iyujj.png)

We can see that both our staging and review applications were automatically stopped without us having to click any button.

## Conclusion
With cloud computing, every extra second your application is running counts and you want to be as lean as possible. In this tutorial, we only used AutoIdle on two applications, but imagine how much you could save with a large application with multiple contributors and new PRs created by the minute, each with a new review app. The cost of having apps running while not actively in use can be unnecessarily large and overwhelming. I hope [AutoIdle](https://medium.com/r/?url=https%3A%2F%2Fautoidle.com%2F) is a tool you and your organization can leverage to reduce costs.

Till next time,
*Happy saving!*