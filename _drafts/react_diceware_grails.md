---
layout: post
title:  "React with Grails on Heroku"
date:   2019-08-19 00:00:00 +1000
categories: react Grails Heroku
---

# React with Grails on Heroku


![Jams](https://mir-s3-cdn-cf.behance.net/project_modules/1400/d728a138739917.57725e810ba47.jpg "Jams")

<p style="font-size: 0.9rem;font-style: italic;"><a href="https://www.behance.net/gallery/38739917/O-Jam">"O' Jam"</a><span> by <span>Mie Le</span></span> is licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/?ref=ccsearch&atype=html" style="margin-right: 5px;">CC BY-NC-ND 4.0</a><a href="https://creativecommons.org/licenses/by-nc-nd/4.0/?ref=ccsearch&atype=html" target="_blank" rel="noopener noreferrer" style="display: inline-block;white-space: none;opacity: .7;margin-top: 2px;margin-left: 3px;height: 22px !important;"><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc_icon.svg" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-by_icon.svg" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-nc_icon.svg" /><img style="height: inherit;margin-right: 3px;display: inline-block;" src="https://search.creativecommons.org/static/img/cc-nd_icon.svg" /></a></p>

## Intro

Last year [I wrote about how I'd deployed a demo app using one of my favourite stacks, Angular and Grails](https://medium.com/@wellsst/angular-with-grails-on-heroku-211e35177804), and had it deployed to [Heroku](https://www.heroku.com/).  I still like using [Grails](https://grails.org/) as a basis for apps, it just has such a logic feel and way to work; backed by [Spring Boot](https://spring.io/projects/spring-boot) and [Groovy](https://groovy-lang.org/) it provides a fast, scalable and reliable path to create an application of any complexity.  Now with the increasing popularity of [Facebook's React](https://reactjs.org/) frontend framework I just had to rewrite this project with React in the frontend.


## Getting started

Now Grails is up to its 4.x release line I also decided to use this version.  A very easy way to dip your toe into the waters of Grails is using the [Grails Application Forge](http://start.grails.org/).

![alt text]({{ site.url }}/grails_forge.png "Title")

Here you'll see other options such as using sdkman and the Grails CLI, but for many this will be the simplest option. The best option though is for IntelliJ IDEA users, a simple description is given on the App Forge (just scroll down a little)

Locally right now you will be able to execute: `./gradlew bootRun --parallel`

In a moment your default browser should open and you'll see the skeleton app:

![alt text]({{ site.url }}/react_grails_init_start.png "Title")

To start the Grails application:              `./gradlew server:bootRun`
To start the client application:              `./gradlew client:bootRun`
To start both client and Grails applications: `./gradlew bootRun --parallel`

## Server-side

Since Facebook also gave us [GraphQL](https://graphql.org/), I am going to use that here as well.  Why? From a client to server, GraphQL gives us a way to get only the data we require, the API can easily evolve without versioning and there is a type system and we can get many resources with a single request kind of like JOINS in SQL.

For this part we are working under the `server` folder of our project.

I've chosen to use the [Grails GORM GraphQL plugin](https://grails.github.io/gorm-graphql/latest/guide/index.html).  It really is overkill for this small demo and perhaps I should have considered a new problem to solve but what the hell I'll stick with it to demonstrate some of the potential differences from the Angular to the React worlds.  Alternatively you could consider [Groovy's library for GQL](https://grooviter.github.io/gql/docs/html5/index.html)

Add the plugin to your `build.gradle` dependencies:

`compile "org.grails.plugins:gorm-graphql:1.0.2"`

For non-code changes you'll likely need to restart the Gradle process.

A very nice feature is the plugin includes the very nifty GraphQL browser enabled by default in development. The browser is accessible at /graphql/browser

I am still using the "official" wordlist from diceware, copy it to server\src\main\webapp\diceware.wordlist.asc.txt so we can access the list of words at runtime.

In the `BootStrap.groovy` lets load those words but this time into a GORM domain class instead of the application wide `servletContext`.

```groovy
def init = { servletContext ->
        log.error "Init diceware app in environment: ${Environment.current.name}"

        // A line looks like: 11214	abyss
        servletContext.getResourceAsStream('diceware.wordlist.asc.txt').text.eachLine { line ->
            List fields = line.tokenize()
            new Word(rollKey: fields[0], theWord: fields[1]).save()
        }
    }
```

We need a super simple domain class as the Grails GraphQL work on domain objects.  We have a couple of options here but I went with adding a custom property

Word.groovy

Add:
```groovy
    static graphql = true

    String theWord

```

If we go to `http://localhost:8080/graphql/browser`, enter and run something like:

```graphql
query {
  wordList(max: 5) {
    id, theWord
  }
}
```

You should get a listing:

```graphql
{
  "data": {
    "wordList": [
      {
        "id": 1,
        "theWord": "a"
      },
      {
        "id": 2,
        "theWord": "a&p"
      },
      {
        "id": 3,
        "theWord": "a's"
      },
      {
        "id": 4,
        "theWord": "aa"
      },
      {
        "id": 5,
        "theWord": "aaa"
      }
    ]
  }
}
```

Not bad for only little work.  If you take through the code that the plugin has provided it gives us a simple, clear and useful basis from which you can build a variety of web applications.

### Getting a random list of phrases

The Grails GraphQL plugin works on domain classes and it does offer both a lot of power out-of-the-box and by customizing.  

Our example requires the server-side to accept a query with parameters for the number of phrases and the number of worlds to generate for each phrase.  GraphQL does allow for such a thing although it doesn't fit that neatly with a Grails domain object but for the sake of demonstration lets go ahead with this.

Change the Word domain class to have a custom operation:

```groovy
static graphql = GraphQLMapping.build {
        query('randomPhrases', Word) {
            argument('nrPhrases', Integer) {
                defaultValue 10
                nullable true
                description 'The number of phrases to generate'
            }
            argument('nrWords', Integer) {
                defaultValue 4
                nullable true
                description 'The number of words in each phrase to generate'
            }
            dataFetcher(new DataFetcher() {
                @Override
                Object get(DataFetchingEnvironment environment) {
                    List<String> phraseList = []
                    (1..environment.getArgument('nrPhrases')).each {
                        String phrase = ""
                        (1..environment.getArgument('nrWords')).each {
                            String rollKey = ""
                            (1..5).each { roll ->
                                rollKey += new Random().nextInt(6) + 1
                            }
                            Word word = Word.findByRollKey rollKey
                            phrase += WordUtils.capitalize(word.theWord)
                        }
                        phraseList << phrase
                    }
                    new Word(phraseList: phraseList)
                }
            })
        }
    }
```

It may look scary at first but if you break it down it is doing essentially 2 things:
    1. Providing a 2 arguments
    2. Some logic that fetches info based on the arguments

Another small addition is to add the phraseList as a domain property, this doesn't fit well with what a domain class should do, but I'll roll with it for now...just for demo purposes or until I find a better way.

```groovy
    List<String> phraseList = []

    static constraints = {
        phraseList nullable: true
    }
```

Now in the GraphiQL browser or CURL we can:

```graphql
query {
  randomPhrases(nrPhrases:6, nrWords:5) {
     phraseList
  }
}
```

and get the result:
```graphql
{
  "data": {
    "randomPhrases": {
      "phraseList": [
        "SledBhoyXyzAnRu",
        "RushAphidLatheEarthGrime",
        "WetGhettoTableAntonBo",
        "CrushBernetCoalDavyZeta",
        "CoaxCwSnoopSitusBook",
        "TestyOboeGustyPaperBid"
      ]
    }
  }
}
```

## Front-end

Now we have a backend that will accept a GraphQL query and provide a result we are ready to create a React Component to use it.  I decided to use Apollo the community client.  For this part we are working under the `client` folder of our project.

Add it as a dependency: `yarn add apollo-boost react-apollo graphql`

Open src/index.js and replace the contents with the following:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import 'bootstrap/dist/css/bootstrap.css';
import './css/App.css';
import './css/grails.css';
import './css/main.css';

import {ApolloProvider} from 'react-apollo'
import {ApolloClient} from 'apollo-client'
import {createHttpLink} from 'apollo-link-http'
import {InMemoryCache} from 'apollo-cache-inmemory'

const httpLink = createHttpLink({
    uri: 'http://localhost:8080/graphql'
})

const client = new ApolloClient({
    link: httpLink,
    cache: new InMemoryCache()
})

ReactDOM.render(
    <ApolloProvider client={client}>
        <App/>
    </ApolloProvider>,
    document.getElementById('root')
)
```

Refresh `http://localhost:3000/` to make sure nothing went wrong.

### Create the PhraseList component

Create a new JS file called PhraseList and give it some placeholder text to render

```javascript
import React, {Component} from 'react';

class PhraseList extends Component {
    render() {
        return (
            <p>PhraseList will go here...</p>
        )
    }
}

export default PhraseList
```

Edit the App.js an include the PhraseList, eg...

```html
            <PhraseList/>
            <p>
              Congratulations, you have successfully started your Grails & React application! While in
              development mode, changes will be loaded automatically when you edit your React app,
...
```

### Call the server and display

Create a Component to display each phrase

```javascript
import React, {Component} from 'react'

class Phrase extends Component {

    constructor() {
        super();
    }

    render() {
        const {text} = this.props;

        return <li className="list-group-item">{text}</li>
    }
}

export default Phrase;
```

Update the PhraseList component to query with GraphQL live data

```javascript
import React, {Component} from 'react';
import { Query } from 'react-apollo'
import gql from 'graphql-tag'
import Phrase from "./Phrase";

const RANDOM_LIST_QUERY = gql`
query {
  randomPhrases(nrPhrases:6, nrWords:5) {
     phraseList     
  }
}
`
class PhraseList extends Component {
    render() {
        return (
            <Query query={RANDOM_LIST_QUERY}>
                {({ loading, error, data }) => {
                    if (loading) return <div>Fetching</div>
                    if (error) return <div>Error</div>

                    const phraseList = data.randomPhrases.phraseList

                    return (
                       [phraseList.map(phrase => <Phrase key={phrase} text={phrase}/>)]
                    )
                }}
            </Query>
        )
    }
}

export default PhraseList
```

### Add a form to control the output

For a larger requirement we could use react forms are even better use [Formik](https://github.com/jaredpalmer/formik) which is a lightweight simpler way to do forms, but for this we can keep things super simple.

```javascript
<div class="form-group">
                    <label htmlFor="nrPhrases">Nr Phrases: {this.state.nrPhrases}</label>
                    <input type="range" name="nrPhrases" placeholder="Nr Phrases" min="1" max="100"
                           class="form-control-range"
                           value={this.state.nrPhrases}
                           width='5'
                           onChange={e => this.setState({nrPhrases: e.target.value})}/>

                    <small id="nrPhrasesHelpInline" className="form-text text-muted">
                        Min 1 / Max 100.
                    </small>

                    <label htmlFor="nrWords">Nr Words/Phrase: {this.state.nrWords}</label>
                    <input type="range" name="nrWords" placeholder="Words/Phrase" min="1" max="20"
                           class="form-control-range"
                           value={this.state.nrWords}
                           width='5'
                           onChange={e => this.setState({nrWords: e.target.value})}/>
                    <small id="nrWordsHelpInline" className="form-text text-muted">
                        Min 1 / Max 20.
                    </small>

                    <button type="submit" class="btn btn-primary" onClick={() => this.updatePhraseList()}>Generate
                    </button>
```

The code looks large but broken down its simple.  Create 2 range slider HTML5 inputs for the number of phrases and the number of words per phrase.  Create a button that calls code to update from the server, based on the values the user selects.

The code to call the server is:

```javascript
    updatePhraseList = async () => {
        const {nrPhrases, nrWords} = this.state
        const result = await this.props.client.query({
            query: RANDOM_LIST_QUERY,
            variables: {nrPhrases, nrWords},
        })
        const phraseList = result.data.randomPhrases.phraseList
        this.setState({phraseList})
    }
```

This will `async` call to the backend GraphQL service on Grails based on the user entered values.

The GraphQL query is updated to use parameters.

```javascript
const RANDOM_LIST_QUERY = gql`
query randomPhrasesQuery($nrPhrases: Int!, $nrWords: Int!) {
  randomPhrases(nrPhrases: $nrPhrases, nrWords: $nrWords) {
     phraseList
  }
}
`
```

One final change is to export our Class Component `withApollo`, this injects the ApolloClient instance that you created in index.js into the Search component as a new prop called client.

```javascript
export default withApollo(PhraseList)

```

One thing you'll notice is that subsequent clicks on the button will not call the server again, this is because caching is turned on in the Apollo client when we created it in the `index.js`
```javascript
const client = new ApolloClient({
    link: httpLink,
    cache: new InMemoryCache()
})
```

## Additional Points

You can see that the app morphed a little from when it was using Angular: https://medium.com/@wellsst/angular-with-grails-on-heroku-211e35177804 .  OVerall I really like using React, it seems simpler, cleaner and more logical (than Angular) which *should* lead to a faster product release and more reliable product.  Angular has lost its way, its just become plain confusing and with each release even though the features are good it becomes difficult to stay in tune with, especially when you don't use it on a daily basis.


## Bundling this up for deployment to the cloud

This is where a couple of tricks came in handy. The Heroku Gradle documentation is a great place to start reading to get an overview, but since this is a Grails multi-project build there are a couple of other things to do.

### Setup environment variables

So far we've been developing locally in 'development' but when we deploy to the cloud it will likely be into the live or production environment.  Each will have separate locations for the GraphQL and DB server.

In React create 2 files...

`.env`

```
REACT_APP_GRAPHQL_SERVER_URL=http://localhost:8080/graphql
```

`.env.production`

```
REACT_APP_GRAPHQL_SERVER_URL=/graphql
```

Update in `index.js`

```javascript
const httpLink = createHttpLink({
    uri: process.env.REACT_APP_GRAPHQL_SERVER_URL
})
```

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
        }
    }
}
```

DB setup on the Heroku side:

[Heroku supports PostgreSQL and is well documented](https://devcenter.heroku.com/articles/heroku-postgresql)

This is so easy to turn on:
heroku addons:create heroku-postgresql:hobby-dev

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

* 


## Deeper reading - references

* http://guides.grails.org/gorm-graphql-with-react-and-apollo/guide/index.html
* 
* https://www.howtographql.com/react-apollo/7-filtering-searching-the-list-of-links/
