+++
title = "SourceHut Code Hosting"

[taxonomies]
tags = ["git", "code_hosting", "sourcehut"]
+++

I really love traditional thenchniques. No mater about what. Well, lets face it; Git is a decentralized VCS. Many new
programmers even don't know it. Many of people even think the git is a tool that relates to Github!

<!-- more -->

In recent years, using online services jumped up and people don't need to install many of traditional applications on
their systems. Alltoghether this facts are removing the tradition (that is the reallity) of systems from peoples' mind.
So more and more people are forgetting this fact that Git is not a web service provided by github. Many of my friends
glare on my eyes and tell me that they found a new service *like* Github that is so classy... and I'm antiquated
because I don't use them: Gitlab!!!

Nevermind. I'm antiquated :-)

But I have some strong reasons that why I don't like them or at least prefer to use that bare-metal git.

## Corporation poison
Nearly all of open source projects that get touched by corporations, became zombie. Corporations are fundations that
live with money. Not just live, but also exists because of money in any kind and origin. We can't blame them, it is
their cause of existance.

The shining star of open source community, the Linux kernel, leads the hate about a corporation that is the symbol of
all *proprietary* corporation (something like Evil Corp): Microsoft.

Hate about macrosoft was not something hide. But when corp bought the Github... Just no one was happy. Personally, I
was confused because I didn't think that beeing under the shadow of a corporation can force a fundation to obay all of
the orders and rules of corporation. As times go on, Github gots more and more microsoftic.

This embarrassing fact is not a faith war between community and microsoft, it is just something uncomfortable.
Like the difference between "Small is pretty" and "Fully integrated" terms. By the way it is the **Last** reason of
open source community to don't use github and similars.

## Tradition
Well, Git (and all other VCSs) where born when there was nothing named "Code Hosting". Contribution was carrying out by
email and discussion was on going via mailling list. For driving the development just a simple git http server is
sufficient and all other work was done through email. There is no **Pull Request** with green button to do the work
automatically; and so... maintainer can do anything to have a clean history and easy integration.

# Sourcehut to the rescue
The original workflow was on go in private hostings. any one that want to adhere to the traditional mechanism, host
her/his git repo on own server. Some example projects are:

 - **Linux kernel**: Linux kernel itself has a private repo ([repository](https://git.kernel.org))
 - **CGit**: Git http front-end written in C ([repository](https://git.zx2c4.com/cgit/)).
 - **NGINX**: Famouse HTTP server and load balancer. This one uses mercurial
   ([repository](http://hg.nginx.org/nginx)).

To have a working flow, beside the git hosting you need a mailing list for discussion, development and bug tracking.
Keeping all of these operational is a time consuming task.

Sourcehut was founded by **Drew DeVault** to address many of this issues. In fact, sourcehut is a collection of open
source web applications that are coupled toghether, and can be deployed by any one. The service is a composition of
following:

 - A repository hosting for Git and Mercurial at [git.sr.ht](https://git.sr.ht) and [hg.sr.ht](https://hg.sr.ht).
   This services just keep the source code; like in CGit.
 - Mailing list for discussion and developmnet at [lists.sr.ht](https://lists.sr.ht). All addresses are also at this
   domain.
 - Bug tracking system at [todo.sr.ht](https://todo.sr.ht). Bugs are reported like in bugzilla.

In additoin to this *basic* facilities, SourceHut also provides CI pipelines but at the moment is not free and not
available in free accounts.

# Conclusion
SourceHut keeps the tradition of Git and Mercurial development principles.

