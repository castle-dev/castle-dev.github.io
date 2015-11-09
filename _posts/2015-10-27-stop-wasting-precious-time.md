---
layout: post
title:  "Stop Wasting Precious Time"
author: Scott Lowe
date:   2015-10-27 14:03:00
categories: tutorial
featured_image: /images/robot-with-heart.jpg
---

Google defines Continuous Integration (CI) as:

> a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.

We're big fans of CI because it allows us to move faster with more confidence. Good CI practices bring with them a peace of mind for your dev ops people–have you ever fucked up a manual deploy before? I have, and it wasn't fun. Luckily, most of these stressful, yet critical processes can be delegated to robots without much work, which frees your developers to focus on what they do best: building things.

In this post, we'll walk through Castle's current CI setup in hopes that this will help others join the club. Be warned, non-nerds, it's gonna get technical.

## our current setup
Castle runs [Jenkins][robot] on a $10 / month [Digital Ocean][servers] droplet. I like Ubuntu, so if you pick another flavor of linux then I can't guarantee that the commands in this post will work (also, be sure to install `git` and any other dependencies your project has). Once you've got Jenkins up and running, you'll need to create a new account on GitHub specifically for Jenkins to have access to your repos. After creating the GitHub account, ssh into your new droplet and run `sudo su jenkins` to switch to the jenkins user, then [generate new ssh keys and add them to your new GitHub account][add-ssh-keys]. Give that user access to your repo (read & write). Finally, use the browser to login to Jenkins, click **Manage Jenkins** in the left sidebar, click **Manage Plugins**, then install both the **Git plugin** and the **GitHub plugin**.

Now you're ready to configure your Jenkins jobs. We maintain 3 jobs for each repo:

- Lint & run unit tests anytime a new commit is pushed to GitHub
- Bump the version anytime a PR is merged into `develop`
- Deploy the release anytime a PR is merged into `master`

Let's go through them in reverse order, as deploying is easily the riskiest of these operations–so you really don't want it to be done manually by humans! If you're going to just set up one of these jobs to get your feet wet, automating your deploy is where I recommend you start.

## Deploy the release anytime a PR is merged into `master`
In order for this to work, you'll need your project to be using the ssh remote (`git remote set-url origin [your ssh remote, ex: git@github.com:castle-dev/castle-dev.github.io.git]` if not) on your prod server, the jenkins user on your droplet must have ssh access to your prod server (copy the contents from jenkin's pulic key `~/.ssh/id_rsa.pub` on the CI server into `~/.ssh/authorized_keys` on the prod server if not), and the user account you're sshing into on prod needs read access to your repo (generate a new ssh key pair with `ssh-keygen` for that user and add it as a deploy key on the repo if not). Once all this is done, you're ready to configure the job in Jenkins.

To do this, create a new item from the Jenkins sidebar. Select **Freestyle project** and call it whatever you'd like (we use `[repo name] deploy`). Now, fill in the config:

- GitHub project: [your repo's url]
- Source Code Management: **Git**
- Repository URL: [your repo's ssh remote]
- Credentials: **Add** > **SSH Username with private key** > **From the Jenkins master ~/.ssh**
- Branches to build: `refs/heads/master`
- Build Triggers: **Build when a change is pushed to GitHub**
- Build: **Add build step** > **Execute shell** > paste in your deploy process

Here's an example deploy process from our web app:
{% highlight bash %}
ssh [user]@[host] <<EOF
  cd [project folder] && git checkout master && git pull && gulp build:dist && cp -r build/dist/* /var/www/public/ && exit
EOF
{% endhighlight %}

Notice the `<<EOF/EOF` tags & how all the commands after sshing are on one line? That's important, otherwise Jenkins will report fails as passing when using ssh. Also, note that `gulp build:dist` and `cp -r build/dist/* /var/www/public/` are commands that are specific to our web app project and your results will likely vary.

And that's it! Now, everytime someone adds new commits to the `master` branch (which if you're trying at all will only be when someone approves a PR from `develop` into `master`), Jenkins will ssh into your production server and update your app. There's also an easy-to-setup Slack plugin that you can install to have Jenkins notify a Slack channel about build results, but that exercise is left for the reader.

## Bump the version anytime a PR is merged into `develop`
If you've made it to here then the hard work is done, as going from 1 to 2 is much easier than going from 0 to 1 when it comes to computers.

Before we discuss the details, let's reflect on the inspiration for this one. Software moves very quickly, so developers have conventions for describing their changes over time. [Semantic Versioning][semver] or semver for short has become the standard for tagging software updates by dividing all changes into 3 distinct buckets: **major**, **minor**, and **patch**. **Major** updates are the ones that introduce breaking changes into the application, **minor** updates are the ones that add new features without breaking any existing functionality, and **patch** updates are the ones that fix bugs but don't add any new functionality.

At Castle, we bump the version everytime a feature PR is merged into `develop`, and the type of bump (**major**, **minor**, or **patch**) is determined by the changes included in the pull request. To communicate this information from the individual who built the feature to the individual who's reviewing it, we append **major**, **minor**, or **patch** on the end of the title of all PRs from feature branches to `develop`. We started using semver by just manually updating the version number and creating a corresponding git tag when merging PRs, but that quickly became tedius so we used some **gulp** magic to automate it [like so][gulpfile-example]. That gulpfile contains 3 tasks for updating the version number: `gulp bump:major`, `gulp bump:minor`, and `gulp bump:patch`. This was much better than changing the numbers by hand every time, but after a while checking out develop, pulling, running the appropriate gulp command, pushing the new commit, and finally pushing the new tag grew old as well.

To that end, we setup Jenkins to automatically run the appropriate bump command when a PR is merged into `develop` for us, and now we don't ever have to think about it again. Now let's dive in!

Create a new job (or copy the deploy job) and configure it like so:

- GitHub project: [your repo's url]
- Source Code Management: **Git**
- Repository URL: [your repo's ssh remote]
- Credentials: **Add** > **SSH Username with private key** > **From the Jenkins master ~/.ssh** or use the identity you created for the deploy job
- Refspec(found under the **Advanced** button between **Credentials** and **Branches to build**): `+refs/pull/*/head:refs/remotes/origin/pr/merge/* +refs/heads/develop:refs/remotes/origin/develop` 
- Branches to build: `**`
- Build Triggers: **Build when a change is pushed to GitHub**
- Build: **Add build step** > **Execute shell** > paste in your bump process

Here's the example bump process from our web app:
{% highlight bash %}
git checkout develop
git pull origin develop
type=$(git show | grep -oE '[^ ]+$' | tail -1)
if [ $type = "major" ] || [ $type = "minor" ] || [ $type = "patch" ]
then
gulp bump:$type
git push origin develop
git push origin develop --tags
fi
{% endhighlight %}

The only complicated stuff here is the type assignment, `$(git show | grep -oE '[^ ]+$' | tail -1)` which says replace me with the last word of the last line of the last commit message. If that matches **major**, **minor** or **patch** then Jenkins will run the appropriate bump command and push the commit and tag up to GitHub. (Note that I'm not a whiz with refspec yet so this job will run more often than just when PRs are merged into `develop` but that's okay because the shell script only takes action when the previous commit message ends in **major**, **minor** or **patch** which won't be an issue if the only way new commits get to `develop` is through a merged feature PR.)

## Lint & run unit tests anytime a new commit is pushed to GitHub
So you've done the heavy lifting, now let's take it to the next level. At Castle, we lint and unit test **every commit** that is pushed to GitHub. With all the hard work out of the way, this job is suprisingly easy to set up:

- GitHub project: [your repo's url]
- Source Code Management: **Git**
- Repository URL: [your repo's ssh remote]
- Credentials: **Add** > **SSH Username with private key** > **From the Jenkins master ~/.ssh** or use the identity you created for the deploy job
- Branches to build: `**`
- Build Triggers: **Build when a change is pushed to GitHub**
- Build: **Add build step** > **Execute shell** > paste in your lint & test process

And finally, here's an example lint & test process from our web app:
{% highlight bash %}
bower install
npm install
gulp build
gulp jshint
gulp test:unit
{% endhighlight %}

Thanks for reading, hope it helps! :]

[servers]: https://digitalocean.com
[robot]: https://jenkins-ci.org
[add-ssh-keys]: https://help.github.com/articles/generating-ssh-keys/
[semver]: http://semver.org/
[gulpfile-example]: https://github.com/castle-dev/le-ascii-art/blob/develop/gulpfile.js
