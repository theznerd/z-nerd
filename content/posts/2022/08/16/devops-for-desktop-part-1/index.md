---
title: "\"DevOps\" for Desktop Engineers - Part 1 - Introduction"
description: "DevOps probably isn't the right word. Whatever. We're adopting all the corporate buzzwords to make it sound worthwhile."
summary: "DevOps probably isn't the right word. Whatever. We're adopting all the corporate buzzwords to make it sound worthwhile." # For the post in lists.
date: '2022-08-16'
aliases:
keywords: ["github","Azure DevOps","ado","PowerShell","Actions","devops"]
author: 'Nathan Ziehnert'
usePageBundles: true
toc: true

featureImage: 'https://images.unsplash.com/photo-1584949091598-c31daaaa4aa9?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=350&q=80' # Top image on post.
featureImageAlt: 'Code on a Screen' # Alternative text for featured image.
featureImageCap: 'Image by Mitchell Luo on Unsplash' # Caption (optional).
thumbnail: 'https://images.unsplash.com/photo-1584949091598-c31daaaa4aa9?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # Image in lists of posts.
shareImage: 'https://images.unsplash.com/photo-1584949091598-c31daaaa4aa9?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&crop=edges&w=1169&h=800&q=80' # For SEO and social media snippets.

categories:
  - PowerShell
  - DevOps
tags:
  - PowerShell
  - DevOps
---
## Pepperidge Farm Remembers
I remember simpler times. When I was just getting started in corporate desktop
support it was "read tickets, fix computers, and be nice." Then I eventually
moved into packaging applications, building images, and more desktop 
"engineering" work. None of it was exceptionally complicated, it was just more
responsibility. We looked for ways to automate our work because we were ~~lazy~~ 
attempting to maximize the amount of work that we could accomplish with the
dwindling resources given to us. 

And over time we grew and evolved.

Now we have tools like [OSDBuilder](https://osdbuilder.osdeploy.com/). Gone are
the days of building a thick gold image or even a "thin" gold image.

Or [Driver Automation Tool](https://msendpointmgr.com/driver-automation-tool/).
Gone are the days of importing a bunch of INFs and hoping for the best during
imaging. 

Or Autopilot. Gone are the days of imaging altogether... when it works.

We've grown so much, and I'm so proud of us.

![Lil' Miss Proud Of You](lilmiss.jpg " ")

## Our Little Secret
However, we have a dirty secret.
We have a lot of permissions in our environments. Depending on where you work
you might have unfettered administrative access to devices, the ability to push
software to 100,000 devices with a couple of clicks, or god-tier powers to
bring a company to its knees with a mistake. We never make mistakes though,
right?

Of course we don't. Because we have a peer review every deployment we create,
right?

Well surely we at least double-check our work, right?

![Padme and Anakin](startrek.png " ")

The truth is, we might have processes or procedures in place to mitigate the
potential for disaster but those processes are about as exciting and fun as
a proctologist visit. And when it comes to actually doing the work (creating 
a deployment, changing a setting) they're still as manual as a prostate exam. 
*\*snaps gloves\**

There has to be a better way.

## Enter "DevOps"
DevOps isn't exactly the correct term to use here. But it's corporate bingo 
time and I'm going for blackout... also it's why I put "DevOps" in quotes.
Regardless, it's time to start thinking about ways we can adopt principles that:

 - Eliminate some possibility of human error
 - Force us to have a second set of eyes
 - Allow us to actually test things in "development"

In my current role, I can't afford to have a bad actor in my environment. I
can't afford to have mistakes either. Everything must be triple-checked, 
reviewed by multiple teams, signed-off, and then we can move forward. 
Additionally, I need to make sure that the processes we create are followed 
without the possibility of someone putting a decimal in the wrong place or 
something.

![I always do that... I always mess up some mundane detail](kingofthehill.gif " ")

One of the tools that I am building to do this is a CI/CD pipeline and protected
repository. The principal is simple but the terminology may be new. The simple
explanation is:

 - We upload our new artifact for review (commit a change to a branch git
 repository and open a pull request on the main branch)
 - When the artifact is uploaded, a set of test scripts executes to make sure
 certain rules are satisfied (run an Azure DevOps Pipeline or GitHub Action
 automatically when the pull request is opened)
 - At the same time a development environment is updated via script (removing 
 the human interaction necessary to do the work) and a peer can review the
 artifact
 - If the rules are satisfied, the peer may then approve the change (merge the
 pull request into the production branch)
 - Now the production environment is updated by the same script used for
 development

One important thing to note here is that both the update to the development
environment and production environment are done by a script. This means that
we can be reasonably certain that if it worked in dev it will work in prod. I
won't accidentally select the wrong collection when creating the deployment in
my ConfigMgr environment.

Additionally notice how there is a record of the peer "reviewing" my work. With
tools like GitHub, we have the opportunity to have communication about the
change kept in the record of the pull request. It's about accountability and
auditing - which I know are about as sexy as the thought of your least favorite
politician in a revealing swimsuit, but I promise you upper management enjoys
~~that mental picture~~ these concepts.

We also have the opportunity here to automate some tests against our work. Maybe
you need to ensure any new application you create in ConfigMgr has certain
metadata set - automate a test to ensure that happened. Maybe you need to
validate that the new version of your WinPE boot image has the PowerShell
components loaded - automate a test to check. The possibilities are as endless
as the mind that creates the tests.

## This sounds like rocket surgery
This is all easier said than done. It's going to take time to figure out where 
you could create scripts to do this type of work. It's going to take even longer
to migrate your existing artifacts into a CI/CD pipeline. I personally have the 
benefit of starting from zero... I know that you likely don't. However, let us 
journey together my friend. We're going to build some fun things.

We're also going to take things slow. Over the coming posts I'm going to 
introduce some basics to building and executing a pipeline. Then we'll talk 
about and do a proof of concept or two for some simple workflows. Later we might 
get into more complicated workflows. The point of this series however, won't be 
for me to just create everything for you to 
["steal with pride"](https://www.amazon.com/Stealing-Pride-Vol-Customizations-ConfigMgr/dp/9187445034) 
because frankly I think many of you will be able to take this concept much 
further than I ever will.

I'm here to introduce you to a neat idea. 

It's your job to take it and make something cool out of it (and share with the
community).

I'm excited to see what you create.