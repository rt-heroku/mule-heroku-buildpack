# Deploy a Mule App to Heroku Dynos

This projects deploys a Mule 4.4.0 (default) Runtime to a Heroku Dyno and handles the registration and de-registration of the Mule Runtime with Anypoint Runtime Manager. The registration with ARM is handled through the platform REST APIs and scripts.

The buildpack includes a pre-built Domain project that has an HTTP Listener that leverages the `PORT` that is assigned when the Dyno is deployed. 
It will also generate the necessary Profile to run, but you can create your own and deploy it, just add it to the folder and commit it to git.

## Requirements

1. [Heroku Account](https://dashboard.heroku.com/)
1. [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install)
1. [Mulesoft Anypoint Platform Account](https://anypoint.mulesoft.com/login/)
1. [Anypoint Studio](https://www.mulesoft.com/lp/dl/studio)

## Setup

```
export $MULE_PROJECT=$HOME/projects/workspace/mule
export $HEROKU_APP_FOLDER=$HOME/projects/heroku_app

# Change to mulesoft project
cd $MULE_PROJECT

# Compile your mulesoft project
mvn clean package

# Copy the mulesoft jar file to your heroku app folder
cd $HEROKU_APP_FOLDER
cp $MULE_PROJECT/target/*.jar $HEROKU_APP_FOLDER/.

# Create Heroku app
heroku create "app_name"

# Add buildpack to app
heroku buildpacks:set https://github.com/rt-heroku/mule-heroku-buildpack
```

### Config

Set the following config vars. These are needed for the scripts to understand which Organization and Environment to register the Mule Runtime against. 

```bash
    heroku config:set MULE_ENV="<Anypoint Environment Name>"
    heroku config:set MULE_ORG="<Anypoint Organization Name>"
    heroku config:set MULE_PASSWORD="<Anypoint Password>"
    heroku config:set MULE_USERNAME="<Anypoint Username>"
```

Additionally you can also go into the `Settings` tab of your app in Heroku and enter those values in the `Config Vars` section.

## Deploy Runtime to Heroku
```
    git add .
    git commit -m "Heroku deployment"
    git push heroku master
```

The Java Heap Size for the Mule Runtime is set to 512MB by default in the `conf\wrapper.conf` file. The first deployment may fail sporadically, but you can scale up to an instance with more than 1GB of RAM. You can do so with the following command:

```
heroku ps:scale web=1:performance-m
```

To see the output on how the deployment is running; run the following in a seperate terminal in the same directory:

```
heroku logs -t
```

## Change Mule runtime version

* The default version of the runtime is 4.4.0, but you can change it by adding the property `mule.version` in `system.properties` located in the root directory of your project 


# Deployment with Maven

## Global Configuration

### Heroku API Key
This plugin uses Heroku's Platform API and thus requires an API key to function. If you have the 
[Heroku CLI](https://cli.heroku.com/) installed and logged in with `heroku login`, the plugin will automatically
pick up your API key. Alternatively, you can use the `HEROKU_API_KEY` environment variable to set your API key:

```sh-session
$ HEROKU_API_KEY="xxx-xxx-xxxx" mvn clean package heroku:deploy
```

## Cookbook

### Deploying a Standalone Application

Add the following to your `pom.xml`, but replace the `<web>` element with the command used to run your application.

```xml
<build>
	<plugin>
		<groupId>com.heroku.sdk</groupId>
		<artifactId>heroku-maven-plugin</artifactId>
		<version>3.0.4</version>
		<configuration>
			<logProgress>true</logProgress>
			<buildpacks>
				<buildpack>https://github.com/rt-heroku/mule-heroku-buildpack</buildpack>
			</buildpacks>
			<appName>${heroku.appName}</appName>
			<includeTarget>false</includeTarget>
			<includes>
			  <include>target/*.jar</include>
			</includes>
	        <processTypes>
	          <web>$MULE_HOME/bin/mule -M-DPORT=$PORT -M-Dmule.agent.enabled=true $JAVA_OPTS</web>
	        </processTypes>
		</configuration>
	</plugin>
</build>
```

You can then run the following command in your mulesoft project to deploy your application:

```sh-session
$ mvn clean package heroku:deploy
```
#### Using System Properties

You can provide the application name as a system property like this:

```sh-session
$ mvn heroku:deploy -Dheroku.appName=myapp
```

#### Using a Heroku Properties File

This solution is best when multiple developers each need their own apps.
Create a `heroku.properties` file in the root directory of your application and put the following code in it
(but replace "myapp" with the name of your Heroku application):

```
heroku.appName=myapp
```

Then add the file to your `.gitignore` so that each developer can have their own local versions of the file.
The value in `heroku.properties` will take precedence over anything configured in your  `pom.xml`.
