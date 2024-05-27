<!--yml
category: 未分类
date: 2024-05-27 12:55:13
-->

# How to take down production with a single Helm command :: ./techtipsy

> 来源：[https://ounapuu.ee/posts/2024/04/04/helm-rollbljat/](https://ounapuu.ee/posts/2024/04/04/helm-rollbljat/)

You’re Cletus Kubernetus: a software developer, and a proud Fedora Linux user.

You know Kubernetes, especially after the time you migrated some services to it.

[Everything is calm.](https://www.youtube.com/watch?v=ia8Q51ouA_s&pp=ygUGa3JhemFt)

Your pods are running. Your service is up. Business as usual.

You release some minor changes to production. Everything is still working. Great!

But then you receive a message from a colleague. Oh no, something has gone wrong with a particular piece of functionality!

No worries. You’re using Helm. You can roll this change back safely. You ask your colleague. “Oh yeah, `helm rollback` should work.”

`helm rollback` it is.

Cool, cool, new pod is starting up. Seems like it is indeed working.

**Wait, where did all the pods go?**

After a hectic troubleshooting session with the team, you redeploy the service and start investigating. A colleague uses the staging environment to do a `helm rollback` and it works as expected, the previous version of the service is successfully deployed.

You investigate logs. The `helm rollback` call worked as expected, and then it began deleting every entity related to the deployment. Pods, secrets, ingresses, *everything* related to the service was gone, and your name was present on each deletion.

The troubleshooting was on standby for a few days since you had no further leads and had to get other work done. But you couldn’t really move on from this issue mentally, could you?

One day you continue the investigation by opening the Helm GitHub repository, looking at the open issues and throwing in some keywords that might be relevant, such as “rollback”.

[What the fuck.](https://github.com/helm/helm/issues/12681)

It wasn’t an issue with Helm, or the way you ran it. Apparently the version of Helm packaged in Fedora Linux included a patch that introduced this issue. You then use the staging environment to reproduce the issue. Everything was gone, again, but this time in a safer environment.

You promptly run `dnf remove -y helm`.

After this and the [xz backdoor](https://openwall.com/lists/oss-security/2024/03/29/4), the idea of living in the countryside and learning beekeeping doesn’t sound *that* bad, does it?