## Build a CI/CD pipeline with Heroku CI.

> Heroku CI automatically runs your app’s test suite with every push to your app’s GitHub repository, enabling you to easily review test results before merging or deploying changes to your codebase.

Besides strong Dev/prod parity, using Heroku CI comes with many benefits key of which include;
- Parallel Test Runs.
- Browser tests and User Acceptance Tests
- Seamless integration with Heroku pipelines
- Easy customization of test dependencies in the test environment
- A Simple Integrated Solution (integration, hosting and deployment)
- Simple, prescriptive developer experience in modern CI/CD solutions
- Management support
- Many supported languages

In this article, we shall build a Heroku Flow CI/CD pipeline that utilizes Heroku CI.
![Heroku Flow illustration](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655673965/46cjevUqD.gif)

### Part 1: App init.
We are going to build a simple node.js application that calculates the factorial of a number. We shall however start with its api and then add the factorial functionality later. This is so we can see Heroku CI in action.
To start, fork the repository [here](https://github.com/123MwanjeMike/cicd-with-herokuci.git) and then clone your forked repository to your computer.
```
$ git clone https://github.com/123MwanjeMike/cicd-with-herokuci.git
```
Next, change to the created directory and switch to the start branch.
```
$ cd cicd-with-herokuci/
$ git checkout start
```
Initialize the project with _npm init_ and then install express
```
$ npm init -y
$ npm install express
```
You should have a new file _package.json_ in your directory. Open it and add the start script _node index.js_.
```
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "node index.js"
},
```
Now create a simple api for the app in _index.js_ file with the code below.
```
$ touch index.js
```
```javascript
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.status(200).json({ message: 'Welcome to the Factorial calculator 🎊' });
});

app.get('*', (req, res) => {
  res.status(404).json({ message: 'Resource not found.' });
});

const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`Listening on port ${port}`));

```
Running _npm start_ should give a similar output as below
![Output after running npm start](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655675963/SbUhXI6dO.png)
![When base url is accessed in browser](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655678082/-o63WiNDP.png)

We now have a basic setup for our application.

### Part 2: The pipeline.

We shall be adding our application to a Heroku pipeline.
First we need to tell Heroku which services to run and how to run these. To do this, we shall add a _Procfile_ file at the root of our directory by running;
```
$ echo "web: node index.js" >> Procfile
```
Commit and push your changes to the remote repository.
```
$ git add .
$ git commit -m "<enter your message here>"
$ git push origin start
```

Now move [here](https://dashboard.heroku.com/pipelines/new) if you have a Heroku account and create a new Heroku pipeline. Remember to connect your forked repository as you create the pipeline. You should end up with a screen like below.
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655679634/syDiquSbl.png)
Click **Enable Review Apps** to have deploy previews for the pull requests(PRs) before they are merged.
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655681289/sUu_-nYWG.png)
Next, under the staging section of the pipeline, click **Add app** and create a new app. Its common practice to have the staging app deployed from the develop branch and the production app from the main branch.
In this case, my factorial-app is being deployed from my default branch(main).
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655683309/EtePD76_d.png)
Click settings and scroll to the Heroku CI section. Click Enable Heroku CI.
**Note:** Heroku gives 1000 free dyno hours per month to its verified users and these can be used to access the Heroku CI service for free until they are depleted — in which case you start paying per extra dyno minutes used.
![Before enabling Heroku CI](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655685003/-C4G6elKu.png)
Our pipeline is now set up and we are ready for action.
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655686837/oQpHz2gZr.png)

### Part 3: The application’s factorials service

We are now going to add a service to our application. We want to enable it calculate the factorial of a number. This will be through a get request with path parameters. We shall use the test driven development approach while at it, so install the test runner mocha and the assertion library chai.
```
$ npm install -D mocha chai
```

Open the _package.json_ file and find the test key under scripts and change its value to have _mocha *.test.js || true_. Your scripts now look like so.
```
"scripts": {
  "test": "mocha *.test.js || true",
  "start": "node index.js"
},
```
We are basically telling mocha to execute _.test.js_ files without displaying errors when we run _npm test_.
Let us now create our test file _factorial.test.js_ that will have our tests which will be automated later on.
```
$ touch factorial.test.js
```
```javascript
const { assert } = require('chai');
const factorial = require('./factorial');

describe('Factorial test', () => {
  it('Factorial(0) = 1', () => {
    assert.equal(factorial(0), 1);
  });

  it('Factorial(1) = 1', () => {
    assert.equal(factorial(1), 1);
  });

  it('Factorial(5) = 120', () => {
    assert.equal(factorial(5), 120);
  });

  it('Factorial(171) = Infinity', () => {
    assert.equal(factorial(171), Infinity);
  });
});
```

Next, create the file _factorial.js_ to which we shall write our function that calculates the factorial of a number.

```
$ touch factorial.js
```
```javascript
const factorial = (number) => {
  let result = 1;
  if (number === 0 || number === 1) {
    return result;
  }
  for (let i = number; i >= 1; i -= 1) {
    result *= i;
  }
  return result;
};

module.exports = factorial;
```
We can run our test suite with _npm test_ and get the output as below
![npm test output](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655688470/UZJr9KHkD.png)
Now let us update our _index.js_ too with an end point that can serve requests for the factorials of numbers.
```javascript
const express = require('express');
const factorial = require('./factorial');

const app = express();

app.get('/', (req, res) => {
  const { host } = req.headers;
  res.status(200).json({ message: 'Welcome to the Factorial calculator 🎊', docs: `http://${host}/docs` });
});

app.get('/docs', (req, res) => {
  const { host } = req.headers;
  res.status(200).json({
    message: 'Documentation',
    request: `http://${host}/factorial/<number>`,
    response: 'The factorial of <number> is <result>`',
    example: {
      request: `http://${host}/factorial/5`,
      response: 'The factorial of 5 is 120`',
    },
  });
});

app.get('/factorial/:number', (req, res) => {
  const { number } = req.params;
  if (isNaN(number)) return res.status(400).json({ message: `'${req.params.number}' is not a number.` });
  if (number > 200) return res.status(200).json({ message: `The factorial of ${number} is Infinity` });
  return res.status(200).json({ message: `The factorial of ${number} is ${factorial(number)}` });
});

app.get('*', (req, res) => {
  const { host } = req.headers;
  res.status(404).json({ message: 'Resource not found.', docs: `http://${host}/docs` });
});

const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`Listening on port ${port}`));

```
I added a docs endpoint to provide documentation for our service request.

### Part 4: The magic

The moment of truth has come. Now we shall push our changes again to GitHub, open a pull request then observe how it all works.
```
$ git add .
$ git commit -m "Factorial service"
$ git push origin start
```
After opening the pull request, we see that Heroku CI tests passed and that the branch was successfully deployed. We can also go ahead to checkout the deployment.
![Open pull request on GitHub](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655690062/4EdCSUyNo.png)
![Deploy preview for our pull request](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655691773/ceTyHJXBn.png)
![All our tests passed on Heroku CI](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655693387/1D2LTYGLU.png)
![Our Heroku pipeline](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655695119/WG8Cl9kKh.png)
Finally, lets enable automatic deploys for our factorial-app still under the staging section of our pipeline.
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655696931/RchURlZzS.png)
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655698686/sAk7GVPhM.png)

From now on, any code push to our main branch will automatically be deployed to the staging environment of our application.
![Our changes on main branch deployed to staging](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655700531/TM791R9cy.png)
![Factorial app running](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655702188/r5IjZvL8B.png)
Since everything is working fine, we can then move/promote the app to production.
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1657655703872/k2_56wOHv.png)

## Conclusion

As you have seen, setting up a CI/CD pipeline with Heroku CI is quite easy and Heroku Flow is a very intuitive workflow for visualizing your code delivery process. Heroku is ideal for small and medium business applications even though large business application with a microservices architecture can still leverage it. With Heroku CI, you can move fast without breaking things.

For reference, you can find the entire project code in this [repository](https://github.com/123MwanjeMike/cicd-with-herokuci) with branches start, part-1, part-2, and part-3 that have the resultant code version of each respective part above and the final application code in the main branch.


This post was sponsored by [AutoIdle](https://autoidle.com/). AutoIdle is an add-on that cuts your Heroku bill by automatically putting your staging and review apps to sleep when you don't need them.

*Happy hacking.*