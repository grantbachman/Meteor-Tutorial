#Tutorial 1
#### In this tutorial we'll get the bare minimum default Meteor application deployed onto Amazon Elastic Beanstalk.

1. First things first, open up a terminal and install meteor with the following [command](https://www.meteor.com/install):<br />
`curl https://install.meteor.com/ | sh`

2. Navigate to whichever directory you use for your web projects(*/path/to/projects*) and create a new app called *sample_app*.<br />`meteor create sample_app`<br />
This will create Meteor's default application at */path/to/projects/sample_app*. Enter that directory and run meteor.<br />
`cd sample_app`<br />
`meteor`<br />
This will start your server and host the app at http://localhost:3000

3. Meteor only requires 1 external dependency, which is Node.js.
Elastic Beanstalk comes with preconfigured Node.js environments, but at the time of writing this, Beanstalk only supports Node.js up to version 0.10.31.
Since the most recent release of Meteor we just installed (1.0.2.1) requires Node.js 0.10.33, we need to downgrade Meteor to a version that is compatible with the version of Node running in the remote environment.
Inside your app directory, run the following meteor command<br />
`meteor update --release 1.0.1`<br />
This version of Meteor only requires Node 0.10.29.
**Note: Make sure you're in your sample_app directory when you run that command. If you run this command outside of that directory, you'll be downgrading your meteor installation systemwide instead of just for your project.**

##### Now that we have a fully running sample app, it's time to deploy it.
1. We won't muck with deploying via git just yet, so we'll instead bundle/zip it up and use the Elastic Beanstalk site to upload it. First, bundle everything up.<br />
`meteor build .. --directory`<br />
This will create a directory called *bundle* at */path/to/projects/bundle*

2. Next, we need to modify this bundle a bit so that Amazon can work it's magic and actually run it.<br />
`cd ../bundle`<br />
`cp programs/server/package.json .`<br />
Amazon NEEDS a package.json file to work with at the top level of the bundle directory, and luckily one is already made for us, we just needed to copy it.

3. Next, you'll need to create a .zip file. Go into Finder (on OSX), select all the files and click `Compress x Files`. This will create an *Archive.zip* file we can upload at */path/to/projects/bundle/Archive.zip*. **Note: Only copy the files *within bundle* not the directory itself.**

4. If you don't already have an account, sign up for Amazon Web Services at http://aws.amazon.com/free.
Elastic Beanstalk isn't actually listed among the free services, but [you still get it for free](http://aws.amazon.com/elasticbeanstalk/pricing/).

5. Go to Elastic Beanstalk and create a new application called *sample_app*. Click throught the prompts, and answer the following when they come up:
 * Environment Tier: *Web Server*
 * Predefined Configuration: *Node.js*
 * Environment Type: *Load Balancing, autoscaling*

6. When it prompts you for a source, click *Upload your own* and select your *Archive.zip* file we created.

7. When it prompts you for an environment name/url, just pick something like the following
 * Environment Name: *sampleApp-dev*
 * Evironment URL: *gdbSampleApp-dev* (.elasticbeanstalk.com)
 *
8. Click through the rest of the prompts, those aren't important right now. Click *Launch*. It'll take a few minutes.

9. After it Launches, you'll notice that there is an ERROR in the Event Log. What just happened is that Amazon doens't know what it needs to do to actually start your new app, so let's fix that. On the left sidebar, click *Configuration*, then on the next page click the little gear next to the *Software Configuration* card. Enter the following in the field:
 * Node Command: *main.js*<br />
(There is a file inside our *bundle* directory called *main.js*, which is what that is referencing)

10. At this point, if you were to go to http://gdbSampleApp-dev.elasticbeanstalk.com you'll see a big fat nothing. Now's a good time to check out the *Logs* feature of Elastic Beanstalk. On the left sidebar, click *Logs* > *Request Logs* > *Last 100 lines*. Click the link. You'll notice there are many sections relating to the stack, but we're only concerned with a section called */var/log/nodejs/nodejs.log*. In there, you'll see it's missing a ROOT_URL variable. So, let's fix that :) Again, go to *Configuration* > *Software Configuration* gear. Under *Environment Properties*, let's add the folowing environment variable and value:
 * *ROOT_URL* : *http://gdbSampleApp-dev.elasticbeanstalk.com*

11. After the environment variable is set, and Amazon reloads it, go to http://gdbSampleApp-dev.elasticbeanstalk.com. YAY! It works!
