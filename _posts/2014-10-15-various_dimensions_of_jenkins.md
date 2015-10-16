---
layout: post
title: "Various Dimensions of Jenkins"
date: 2015-10-15
author: Thyagaraju
tags: jenkins cron script chef automation
category: Use Cases
excerpt: "Lets discuss the various use cases of Jenkins"
---


## Introduction

`Jenkins`, we all know is an absolute best `Continous Integration Tool`. But hey, lets not stop at that point because I feel there can be a lot more use cases of Jenkins and we will try to cover those in this article.

I would not get into the history and installation/configuration of Jenkins, as it is beyond the scope of this article. However, if you would like to read about it, I would highly recommend [this article][6]

To start with, let me tell you guys as to how I look at `Jenkins`.

For me, `Jenkins` is an extremely powerful, extremely capable but a `dumb tool`. I call this as a `dumb tool` only because, as of now `Jenkins` is an absolute executor but it cannot make any logical decisions. 

Just because `Jenkins` does not make decisions, lets not undermine `Jenkins` in any way, because as I said earlier, its an absolute executor and it can do anything and everything to perfection and its just that, we need to know how we can use `Jenkins` and how best we can utilize it's potential and that is what I would like to touch upon in the below sections.

## Use Cases of `Jenkins`

- Continous Integration and Continous Delivery and Deployment - CI/CD
- Scheduler 
- `SPOT` (Single Point of Trigger) for `BAD` (Build, Automate Infrastructure and Deployment)


## `Jenkins` for Continous Integration


Continuous Integration is a process in which all development work is integrated at a predefined time or event and the resulting work is automatically tested and built. The idea is that development errors are identified very early in the process. The basic functionality of Jenkins is to execute a predefined list of steps based on a certain trigger. The trigger might be a change in a version control system or a time based trigger

`Jenkins` performs the following steps:

- Perform a `Software Build`
- Run `Execute Shell` Sections
- Archive the `Build Artifacts`
- Perform `Integration Tests`

`Jenkins` can be extended by additional plug-ins, e.g. for building and testing Android applications or to support the Git version control system or any other activities can be performed using the [right plugins][6]

`Jenkins` stores all the settings, logs and build artifacts in its home directory, for example, in `/var/lib/jenkins` under the default install location of Ubuntu. 

The jobs directory contains the individual jobs configured in the Jenkins install. We can move a job from one Jenkins installation to another by copying the corresponding job directory. We can also copy a job directory to clone a job or rename the directory.


## `Jenkins` as Scheduler

We all know how a Cron works. A Cron is nothing but a Scheduler, just a Scheduler which schedules the process and and those processes can be absolutely anything. Cron runs things at fixed times, repeat them etc. 

In the same lines, if implemented properly, `Jenkins` can be your best alternate for `Cron`. 

`Jenkins` has something called as `periodic builds` which accepts the time frame in a fashion very much similar to cron and this in combination with `Execute Shell` feature  can beat Cron anytime, provided, we use it in the Right Way.

Even if we want to migrate from `Cron` to `Jenkins`, it can be done effortlessly as the learning curve is as good as nothing.

*Advantages of Jenkins Over Cron:*

- Measuring Build Successs: `Jenkins` verifies a zero return status for every `Execute Shell` build step and if it returns anything else other than Zero, it can notify us.
- Logs are available for every job, which means, we have logs for every `Execute Shell` steps inside the Job.
- `Jenkins` comes with a feature called `Queue`, which comes handly if a job is still running and the `periodic build`, then based on how we define, `Jenkins` can run the job immediately like cron or queue the job to run when the one in progress finishes.
- Jenkins also supports `Remote Jobs`. `Jenkins` can sign onto systems over SSH, copy over its own runtime, and run whatever you’d like on the remote system. No matter servers in a cluster need scheduled jobs, `Jenkins` can schedule, execute and log them from one server.

## `Jenkins` as `SPOT` for `BAD`

`Jenkins` as a single point of trigger for `Build`, `Automate Infrastructure` and `Deployment` is more of a process that can be implemented and is not a ready to use feature of `Jenkins`. When we are using Jenkins as `SPOT` for `BAD`, we are effectively giving instructions to Jenkins as to what needs to be done and in what way and we also need to pass the required parameters. 

*Build:*

How a code gets built, varies per project.

Jenkins can be used to make different types of builds, such as continuous, official, periodic, nightly and others. Builds can be triggered manually by initializing the build through Jobs or through scheduled or periodic jobs or even through email.

Jenkins can accept the code from various `Source Control` systems, such as `Git` or `SVN` or any other `Source Control` and all it needs is the right kind of plugins to support it. Once it gets the Code, it compiles, based on the result, if it passes, it builds an artifact and stores the artifact in a location defined by you and if the build process fails, it triggers the notifications to the distribution list, again pre configured by you. 

I would not like to get into the details of build steps as that is not what I would like cover in this article and would rather prefer to the Use Cases of Jenkins and yeah, `Build` is one Really Important Use case.

*Automate Infrastructure:*

When I am talking about Infrastructure Automation, what I mean is, how do I bring up the entire stack of infrasturture  using `Jenkins` and to achieve this, I would prefer using Infrastructure Automation tools such as `Chef` or `Puppet` or even `Ansible` along with `Jenkins` and if configured properly, infrastructure can be brought up anywhere, be it, Physical Data Center or Public Clouds such as `AWS` or Private Clouds such as `OpenStack`. Infact, the same logic can be applied to say `Azure` or even `VMWare`.

In my case, I would be using `Chef` as my Infrastructure Automation Tool and I would want my Infrastructure to come up in `AWS` and `OpenStack`.

What I would do is, one of my `Jenkins Job` which brings up the Infrastructure would accept the requred answers in the form of Parameters and these Parameters would include but not limited to details such as the `CLOUD TYPE` and the `ACCOUNT or Tenant` and the `Components` that would need to come up and the `Environment Name` which needs to be created and a few others.

Once all the paramters are taken, as part of `Execute Shell` feature, I would be calling a couple of bash scripts in the backend, which are again version controlled. These scripts would take the parameters from previous section and contacts the  Chef Server and would create a `Chef Environment`.

Once the environment is created, then I would do a `Knife Create` on either `AWS` or `OpenStack` and bring up the instances based on the parameters and also `Tagging` would be taken care. Once the instances are up, with the help of `Public IP` and `Tags`, I would attach those instances to the `Chef Environment` and the `Roles` would be assigned through the script itself, based on the `Instance Type` or `Tags` and then would do a bootstarp on those machines and run the `Chef-Client` and my Infrastructure would be ready. Everything is scripted and everything is controlled via Jenkins. The same approach can also be used via another Job to bring down the infrastructure.

The above scenario would suit any kind of Infrastructure and is best suited for `Immutable Infrastructures`.

*Deployment:*

Deployment through Jenkins is a Straight forward process and guess there is no need to go into too much of details.

However, if we follow the above logic, then, depolyments would be even much easier, because, as parameters, all we need to probably pass is the `Build Name` and the `Chef Environment Name` and the `Account` name and again through the scripts in the backend and the paramters passed, Jenkins can be configured to do a auto discovery to connect to the nodes and deploy the artifacts on them. 

This deployment job can also have some `Post Job` actions to take care of any post deployment activities and may also include the required test cases to do `Sanity`.


`Jenkins` if configured and controlled properly, can be a best `SPOT` for `BAD`


**Hope this helps! Keep forking.**

Please leave your comments below if you have any doubts or questions.

#####Need DevOps help? - Get in touch with [The Remote Lab][1] 
[LinkedIn][2] [Facebook][3] [Github][4] [Twitter][5]


  [1]: http://theremotelab.io
  [2]: https://www.linkedin.com/company/the-remote-lab
  [3]: https://www.facebook.com/TheRemoteLab
  [4]: https://github.com/TheRemoteLab
  [5]: https://twitter.com/TheRemoteLab
  [6]: https://slideshare.net/bhalothia/jenkins-a-complete-solution