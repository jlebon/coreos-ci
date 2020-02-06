# Setting up CoreOS CI

First, create the Jenkins base infra:

```
oc new-app --file=manifests/jenkins.yaml
```

Create the JCasC configmap:

```
oc create configmap jenkins-casc-cfg --from-file=jenkins/config
```

Create the GitHub OAuth secret (these creds currently live
in jlebon's "CoreOS CI" GitHub OAuth App settings, which he
will soon transfer to the coreos/ org).

```
oc secrets new github-oauth client-id=github-oauth-client-id client-secret=github-oauth-client-secret
```

Create the GitHub webhook shared secret:

```
oc secret new github-webhook-shared-secret secret=github-webhook-shared-secret
```

Now we can set up the Jenkins S2I builds. We use the same
settings as the FCOS pipeline to ensure that the environment
is the same (notably, Jenkins and plugin versions):

```
# XXX oc process -l app=coreos-ci -f https://raw.githubusercontent.com/coreos/fedora-coreos-pipeline/master/manifests/jenkins-s2i.yaml | oc create -f -
oc process -l app=coreos-ci -f https://raw.githubusercontent.com/jlebon/fedora-coreos-pipeline/split-out-s2i/manifests/jenkins-s2i.yaml | oc create -f -
```

If working on your own fork/branch, you can use the
`JENKINS_JOBS_URL` and `JENKINS_JOBS_REF` parameters to
override the repo in which to look for jobs, and/or
`JENKINS_S2I_URL` and `JENKINS_S2I_REF` to override the repo
in which to look for the Jenkins S2I configuration.

Then start a build:

```
oc start-build jenkins
```

And that's it! It's already set up so that jobs will be
created on first boot, etc...
