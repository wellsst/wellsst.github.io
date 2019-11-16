---
layout: post
title:  "React with Grails on CloudFoundry - Deploying it"
date:   2019-08-19 00:00:00 +1000
categories: software react Grails cloud Heroku
---

# React with Grails on ~~Heroku~~ CloudFoundry

![the cloud](https://farm1.staticflickr.com/466/18060895474_b7759242c2_b.jpg "cloud foundry")

<p style="font-size: 0.9rem;font-style: italic;"><a href="https://www.flickr.com/photos/111692634@N04/18060895474">"The Cloud - Cloud Data"</a><span>by <a href="https://www.flickr.com/photos/111692634@N04">perspec_photo88</a></span> is licensed under <a href="https://creativecommons.org/licenses/by-sa/2.0/?ref=ccsearch&atype=html" style="margin-right: 5px;">CC BY-SA 2.0</a><a href="https://creativecommons.org/licenses/by-sa/2.0/?ref=ccsearch&atype=html" target="_blank" rel="noopener noreferrer" style="display: inline-block;white-space: none;opacity: .7;margin-top: 2px;margin-left: 3px;height: 22px !important;"><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc_icon.svg" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-by_icon.svg" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-sa_icon.svg" /></a></p>

## Background

Last year [I wrote about how I'd deployed a demo app using one of my favourite stacks, Angular and Grails](https://medium.com/@wellsst/angular-with-grails-on-heroku-211e35177804), and had it deployed to [Heroku](https://www.heroku.com/).  I still like using [Grails](https://grails.org/) as a basis for apps, it just has such a logic feel and way to work; backed by [Spring Boot](https://spring.io/projects/spring-boot) and [Groovy](https://groovy-lang.org/) it provides a fast, scalable and reliable path to create an application of any complexity.  

Now with the increasing popularity of [Facebook's React](https://reactjs.org/) frontend framework I just had to rewrite this project with React in the frontend: [Diceware built with React and Grails]({% link _posts/2019-09-06-react_diceware_grails.md %})

Now the simple app is written it's time to deploy it to a cloud provider.  I had started down the path of going again with Heroku.  Heroku has a free plan and a simple build pipeline.  Initially I was able to follow my old guide as for Angular and with only a couple of minor changes able to get it going...[except that for a reason I still cannot solve the GORM GraphQL was not getting loaded](https://stackoverflow.com/questions/57687205/grails-graphql-plugin-controller-not-running-when-deployed-to-heroku), I scoured briefly the plugin source code and even the grails plugin loader but without a full debug and short on time I deferred to looking at alternatives.  This led me down the path of other cloud providers:

1. [AWS](https://aws.amazon.com/?nc2=h_lg) - Certainly the turn to owner of this space but for this application it was a steeper learning curve and a bit of overkill.  Also it's where many other players host their stuff anyway
2. [Bintray](https://bintray.com/)
2. [Digital Ocean](https://www.digitalocean.com/pricing/)
2. [Google Cloud](https://cloud.google.com/)
2. [Azure (MS)](https://azure.microsoft.com/en-au/)
3. [Cloudsmith](https://cloudsmith.io/), like many of the others possibly pricy for this and seemingly need to use docker for my app (not a bad thing)
4. [The CloudFoundry based solutions](https://www.cloudfoundry.org/certified-platforms/) ![cloud foundry](https://www.cloudfoundry.org/wp-content/uploads/2017/01/CFF_Logo_rgb.png "cloud foundry") I did look initially at the offering from my old company, [SAP](https://cloudplatform.sap.com/try.html) which did look slick but in the end I opted for the PCF or Pivotal CloudFoundry, they own Grails afterall.  It may not yet be the most well polished solution but they do offer USD$87 in credits which is totally enough to get a free developer license going and unlike Heroku's free dyno this one won't shutdown after 30 minutes. 

Many of these support Docker/Kubernates which was a tempting choice, [there is this Grails guide that explains how to use Docker with a Grails app](http://guides.grails.org/grails-as-docker-container/guide/index.html) 

## Common deployment tasks

The 2 options I used there are common tasks and I'd be quite certain this will apply across any deployment scenario.

### Setup environment variables

So far we've been developing locally in 'development' but when we deploy to the cloud it will likely be into the live or production environment.  Each will have separate locations for the GraphQL and DB server.

In React create 2 files...

`.env` with contents:

```
REACT_APP_GRAPHQL_SERVER_URL=http://localhost:8080/graphql
```

`.env.production` with contents:

```
REACT_APP_GRAPHQL_SERVER_URL=/graphql
```

Update in `index.js`

```javascript
const httpLink = createHttpLink({
    uri: process.env.REACT_APP_GRAPHQL_SERVER_URL
})
```

Note that custom variables declared this way must start with `REACT_APP_`

Restart your client process and you should see no difference, this is good, you didn't break anything!  

For a more in depth read on this see: [Adding Custom Environment Variables](https://create-react-app.dev/docs/adding-custom-environment-variables)

For the database in Grails:

Add a dependency to server/build.gradle for the SQL driver:
`compile group: 'org.postgresql', name: 'postgresql', version: '42.2.2'`

point your Grails production environment setup to Postgres, just add a file in: `server/grails-app/conf/application.groovy`

Set its contents to:

```groovy
environments {
    production {
        dataSource {
            dbCreate = "update"
            driverClassName = "org.postgresql.Driver"
            dialect = org.hibernate.dialect.PostgreSQL94Dialect
            uri = new URI(System.env.DATABASE_URL?:"postgres://localhost:5432/test")
            url = "jdbc:postgresql://" + uri.host + ":" + uri.port + uri.path + "?sslmode=require"
            username = uri.userInfo.split(":")[0]
            password = uri.userInfo.split(":")[1]
            initialSize = 2
            maxActive = 5
        }
    }
}
```

Change the initialSize and maxActive depending on how you are deploying, eg the free PCF ElephantSQL gives max connections.

I also had to edit the application.yml file so the driverClassName was not set for all environments, ie move it to just under the development environment (I didn’t think should be the case but be aware)

In the `client/build.gradle`:
Update the version of moowork (Gradle plugin for executing node scripts):
```
plugins {
    id "com.moowork.node" version "1.3.1"
}
```

Add:

```
task buildClient(type: YarnTask, dependsOn: 'yarn') {
    group = 'build'
    description = 'Compile client side assets for production'
    args = ['run', 'build']
}
```

`server/build.gradle`:
Add dependencies:

```
compile 'com.github.jsimone:webapp-runner:8.5.43.1'
provided "org.springframework.boot:spring-boot-starter-tomcat"
```

Add extra tasks for bundling:
```
task stage() {
    dependsOn clean, war            
}
war.mustRunAfter clean

task copyToLib(type: Copy) {
    into "$buildDir/server"
    from(configurations.compile) {
        include "webapp-runner*"
    }
}

war.dependsOn(':client:buildClient');
war {
    baseName = 'diceware'    // otherwise this is 'server'
    version = project.version // + '.' + System.currentTimeMillis();   // I simply prefer to append a ms timestamp in case I've run multiple builds

    // Finally, include the files from the client:
    from('../client/dist') {
        include '**/*'
        //into('WEB-INF')
    }
}

stage.dependsOn(copyToLib)

```

You can run `gradle assemble` and all going well you'll have a runnable war file in the `server/build/libs` folder.

Test this locally with the [Webapp runner](https://github.com/jsimone/webapp-runner) with something like: `java -Dgrails.env=prod -jar server/build/server/webapp-runner-*.jar --port 8099 server/build/libs/*.war`, you will need to setup a DB as well though.  There's also a Jetty runner available.

## The PCF Way

![PCF](https://www.cloudfoundry.org/wp-content/uploads/2017/01/provider-tile-pivotal.jpg "PCF")

[Create a Pivotal Account](https://account.run.pivotal.io/z/uaa/sign-up), quick, simple and no obligations

Hint: Subsequent logins select the "Web Services" option to find your applications etc.

Another good place to start is the short [getting started tutorial](https://pivotal.io/platform/pcf-tutorials/getting-started-with-pivotal-cloud-foundry/introduction), always a good idea to get the basics and ensure you have everything in order before we get to the harder stuff AND this includes installing the CF CLI - essential so do it now.  

An extension to the basic tutorial is the [Getting Started Deploying Grails Apps](https://docs.pivotal.io/pivotalcf/2-4/buildpacks/java/getting-started-deploying-apps/gsg-grails.html)

Create a `manifest.yml` file in the project root.  Give it contents like:

```yaml
---
applications:
- name: diceware
  memory: 768M
  instances: 1
  random-route: true
  path: server/build/libs/diceware-1.war
  buildpacks:
  - https://github.com/cloudfoundry/java-buildpack.git
  services:
  - postgresql
  health-check-type: process
```

* Memory and instances will be dependant on how much you are paying and how much you need. 
* random-route will give your app a random URL like: https://diceware-brash-duiker.cfapps.io/
* services are self explanatory but I'll add to this below.
* health-check-type by default will try and connect to a service running on 8080, but its not and your deploy will fail to start-up.  [More info - App Manifest Attribute Reference](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest-attributes.html#health-check-http-endpoint)

### Create and Bind a Service Instance for your DB

Run the cf CLI `cf marketplace` command to view managed and user-provided services and plans available to you.  You'll see (among other services):
> elephantsql        turtle, panda, hippo, elephant     PostgreSQL as a Service 

Run `cf create-service elephantsql turtle postgresql`. This creates a service instance named postgresql that uses the elephantsql service and the turtle (free) plan

Similar options to this are also available via https://console.run.pivotal.io, select marketplace and find the services you require, this will also list capacities and prices for the other plans for each service.

The service also needs to be bound but this is already done in the `manifest.yml`, otherwise us `cf bind-service` or the WebUI.

### Deploy it

Its a simple 3 steps:

1. `gradle assemble` - Build fresh
2. `cf push diceware` - Push the build up to the remote, and diceware is the name of the app being created
3. `cf logs diceware --recent` - View the logs

Put it all together into one command line:
`gradle assemble && cf push && cf logs diceware --recent`

### Test it

Go to the [PCF Console - WebUI](https://console.run.pivotal.io)

If you can see your app under "Recently Accessed Apps" then click on it.  Otherwise drill down through the Org you created then the space and then the app, there will be a "route" setup for you that links to the deployed app.

There is [a section on troubleshooting on the PCF site](https://docs.pivotal.io/pivotalcf/2-4/devguide/deploy-apps/troubleshoot-app-health.html)

## The Heroku way

![Heroku](https://www0.assets.heroku.com/assets/home/hero/data-a4eeceb4fc7926c678eb97c570037dc83f75a052f523f1c3014b1c0b1d505bf6.png "Heroku")

As I said earlier that this isn't working at the moment due to the GraphQL plugin not loading but other plugins did load so this is still a viable option for many use-cases and who knows by the time you read this it might be all sorted out.  The build-deploy pipeline and other features of Heroku really do make it a slick product.

### Bundling this up for deployment to the cloud

This is where a couple of tricks came in handy. The Heroku Gradle documentation is a great place to start reading to get an overview, but since this is a Grails multi-project build there are a couple of other things to do.

DB setup on the Heroku side:

[Heroku supports PostgreSQL and is well documented](https://devcenter.heroku.com/articles/heroku-postgresql)

This is so easy to turn on:
heroku addons:create heroku-postgresql:hobby-dev



Define a “Procfile” in the root of the project and name it exactly as “Procfile”, give it the contents:
`web: cd server/build ; java -Dgrails.env=prod -jar ../build/server/webapp-runner-*.jar --expand-war --port $PORT libs/*.war`

To have a deployable WAR file that includes all Angular assets put in the right place you can run:
> gradle assemble

To push to your Heroku app the first time (from your base project folder):
> heroku login
> heroku create
> git commit -am "Initial commit..."
> git push heroku master

Then wait for it to do its thing. It will report success or fail and give the URL to check the running app.

For subsequent builds, change your code, commit it and then:
> gradle assemble && git push -f heroku master && heroku logs -t
> 
Login to the Heroku dashboard and check the logs for any errors.

https://limitless-waters-44337.herokuapp.com/ | https://git.heroku.com/limitless-waters-44337.git


## For further investigation

* [Heroku vs AWS](https://www.clickittech.com/aws/heroku-vs-aws/)
* [Heroku vs Digital Ocean](https://medium.com/@standingdreams/heroku-vs-digital-ocean-an-experiment-in-the-making-ce375e7976d)


## Deeper reading - references

* [PCF - Getting Started Deploying Grails Apps](https://docs.pivotal.io/pivotalcf/2-4/buildpacks/java/getting-started-deploying-apps/gsg-grails.html)
* [PCF - App Manifest Attribute Reference](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest-attributes.html#health-check-http-endpoint)
* [Deploy a Grails 3 App to Pivotal Web Services (PWS)](http://guides.grails.org/grails-deploy-pws/guide/index.html)
* http://guides.grails.org/gorm-graphql-with-react-and-apollo/guide/index.html
* [How to deploy a Grails 3 app to AWS Beanstalk and CloudFront CDN](https://medium.com/agorapulse-stories/how-to-deploy-grails-3-app-to-aws-elastic-beanstalk-with-gradle-and-travis-318d084c0f7d)

