---
title: Fathom
description: Fathom Analytics provides very simple website stats without tracking or storing personal data.
author: esune
resourceType: Components
personas: 
  - Developer
  - Product Owner
  - Designer
labels:
  - fathom
  - analytics
  - web
---
# Fathom

Fathom Analytics provides very simple website stats without tracking or storing personal data.  If you require more comprehensive analytics, a Google Analytics alternative, where data ownership and privacy compliance are still a concern check out [Matomo Openshift](https://github.com/BCDevOps/matomo-openshift).

[fathom-openshift](https://github.com/BCDevOps/fathom-openshift) is a set of OpenShift configurations to set up an instance of the Fathom web analytics server. See: [Fathom](usefathom.com)

## Architecture
The service is composed by the following components:
- *fathom*: the main analytics service.
- *fathom-db*: a [postgresql](https://www.postgresql.org) instance that will be used to store the analytics data.
- *fathom-proxy*: the [nginx](https://www.nginx.com) service used as reverse-proxy

## Deployment / Configuration
The templates provided in the `openshift` folder include everything that is necessary to create the required builds and deployments.

In order to run Fathom on the openshift cluster, you MUST install [openshift-developer-tools](https://github.com/BCDevOps/openshift-developer-tools) and [Artifactory](https://github.com/BCDevOps/OpenShift4-Migration/issues/51)  

There should already be a "artifacts-default-******" secret in the tools environment of your openshift cluster. Copy the username and password of this 
secret and run the following command in each environment you wish to build/deploy to. (example: tools and dev / tools and prod)
~~~
oc create secret docker-registry artifactory-creds \
    --docker-server=docker-remote.artifacts.developer.gov.bc.ca \
    --docker-username=<our username from secret> \
    --docker-password=<our password from secret> \
    --docker-email=unused
oc secrets link default artifactory-creds --for=pull
oc secrets link builder artifactory-creds
~~~

Once the secret is created, use the manage script in the openshift folder to deploy your project  
```./manage -n 4a9599 init```  
this will generate local param files, make sure to go through each of the param files, uncomment NAMESPACE_NAME, and set it to your project namespace. In this case, 4a9599.  
next we can build and deploy  
```./manage build```
```./manage -e dev deploy```


## First Run
Once everything is up and running in OpenShift, follow the [instructions](https://github.com/usefathom/fathom/blob/master/docs/Installation%20instructions.md#register-your-admin-user) and create your admin user to secure the analytics dashboard.

To start tracking, create a site in your Fathom dashboard and copy the tracking snippet to the website(s) you want to track.

## Filtering traffic from localhost and dev/test instances
Fathom does not yet support defining filters to exclude traffic based on URL/referrer. To separate traffic coming from different environments it is necessary to use a different tracking ID for each instance.

it is possible to achieve this by:

1) importing a "generic" script to the app/pages you will want to track, like `<script type="text/javascript" src="/fathom.js"></script>`
2) configuring your webserver to serve, if available, a static file containing the tracking code when receiving requests to `/fathom.js`.
For nginx the configuration to add looks like this:
```
# serve the fathom analytics tracking code, if available
location =/fathom.js {
    root /tmp;
    gzip            on;
    gzip_min_length 1000;
    gzip_types      *;
}
```