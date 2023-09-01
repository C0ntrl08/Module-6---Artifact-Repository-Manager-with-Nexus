#  6 - Artifact Repository Manager with Nexus
Exercises for Module "Artifact Repository Manager with Nexus"

You and other teams in 2 different projects in your company see that they have many different small projects, including the NodeJS application you built in the previous step, the java-gradle helper application and so on. You discuss and decide it would be a good idea to be able to keep all these app artifacts in 1 place, where each team can keep their app artifacts and can access them when they need.

So they ask you to setup Nexus in the company and create repositories for 2 different projects.


## EXERCISE 1: Install Nexus on a server
If you already followed the demo in the Nexus module for installing Nexus, then you can use that one.

If not, you can watch the module demo video to install Nexus.

***
```
Set up a Droplet on Digital Ocean with 8 GB RAM, 160GB Disk in Fra1 Server for 48$/mo. After finishing the installation made an inbound firewall rule, to be able to use SSH to operate with the server. Nexus uses Java 8 JDK, so I had to install it. Made a "nexus" user to run nexus as a service and moved on the the second exercise.
```

## EXERCISE 2: Create npm hosted repository
For a Node application you:

create a new npm hosted repository with a new blob store
***
```
Created an npm repo with a new npm blob store and added the required roles and users (project, project1, project2 for users and npm-team1, nx-maven and project roles with the necessary permission)
```

## EXERCISE 3: Create user for team 1
You create Nexus user for the project 1 team to have access to this npm repository

***
```
After adding the user with adduser username command (project1) made the needed permissions for the user and added the user to the newly created role (npm-team1)
```

## EXERCISE 4: Build and publish npm tar
You want to test that the project 1 user has correct access configured. So you:
build and publish a nodejs tar package to the npm repo

Use: Node application from Cloud & IaaS Basics exercises

Hint:

# for publishing project tar file 
npm publish --registry={npm-repo-url-in-nexus} {package-name}

***
```
Opened the project in IDEA to be able to pack the npm files into a tar (C:\Cache\cloud-basics-exercises\app\bootcamp-node-project-1.0.0.tgz)
Used this to login:
npm login --registry=http://your-nexus-repo-url/repository/npm-repo-name/
and used this after:
npm publish --registry=http://your-nexus-repo-url/repository/npm-repo-name/
Had to configure Security - Realms and add to Active Realms: npm Bearer Token Realm
After I managed it, the file was in the related npm repo.
```
## EXERCISE 5: Create maven hosted repository
For a Java application you:

create a new maven hosted repository

***
```
Added a new maven2 repo, called: http://209.38.192.169:8081/repository/maven2-hosted/
```

## EXERCISE 6: Create user for team 2
You create a Nexus user for project 2 team to have access to this maven repository
***
```
Created a second user, called project2 and added the needed permissions with security roles to it. (project2, nx-maven, maven2-hosted-*)
```

## EXERCISE 7: Build and publish jar file
You want to test that the project 2 user has the correct access configured and also upload the first version. So:

build and publish the jar file to the new repository using the team 2 user.

Use the java-app application from the Build Tools module

***
```
Opened the project in IDEA, rewrote the build.gradle and settings.gradle file, created a gradle.properties file.
Added:
---build.gradle:
apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version"+".jar"){
                extension 'jar'
            }
        }
    }

    repositories {
        maven {
            name 'nexus'
            url "http://209.38.192.169:8081/repository/maven2-hosted/"
            allowInsecureProtocol = true
            credentials {
                username project.repoUser
                password project.repoPassword
            }
        }
    }
}

---gradle.properties:
repoUser = project2
repoPassword = project2

---settings.gradle:
pluginManagement {
    repositories {
        maven { url 'https://repo.spring.io/milestone' }
        maven { url 'https://repo.spring.io/snapshot' }
        gradlePluginPortal()
    }
}
rootProject.name = 'my-app'

gradle build
Notes: care for the permission and for the authentication, otherwise it will fail.
gradle publish
```

## EXERCISE 8: Download from Nexus and start application
Create new user for droplet server that has access to both repositories
On a digital ocean droplet, using Nexus Rest API, fetch the download URL info for the latest NodeJS app artifact
Execute a command to fetch the latest artifact itself with the download URL
Untar it and run on the server!
Hint:

# fetch download URL with curl
curl -u {user}:{password} -X GET 'http://{nexus-ip}:8081/service/rest/v1/components?repository={node-repo}&sort=version'
***
```
Created a third user for Droplet Server and for the Nexus Server - called project. Added a new role, with permissions to view all the repo on nexus. (nx-repository-view-npm-*-* and nx-repository-view-maven2-maven2-hosted)
Executed the command to fetch, download and untar the artifacts (but first wanted to find out):
--- curl -u project:project -X GET 'http://209.38.192.169:8081/service/rest/v1/repositories'

--- curl -u project:project -X GET 'http://209.38.192.169:8081/service/rest/v1/repositories/npm-hosted'

--- curl -u project:project -X GET 'http://209.38.192.169:8081/service/rest/v1/components'

--- curl -u project:project -X GET 'http://209.38.192.169:8081/service/rest/v1/components?repository=npm-hosted&sort=version'

--- wget http://209.38.192.169:8081/repository/npm-hosted/bootcamp-node-project/-/bootcamp-node-project-1.0.0.tgz

--- tar -zxvf bootcamp-node-project-1.0.0.tgz

Started it and tested in the browser.

``` 


## EXERCISE 9: Automate
You decide to automate the fetching from Nexus and starting the application So you:

Write a script that fetches the latest version from npm repository. Untar it and run on the server!
Execute the script on the droplet
Hints:

# save the artifact details in a json file
curl -u {user}:{password} -X GET 'http://{nexus-ip}:8081/service/rest/v1/components?repository={node-repo}&sort=version' | jq "." > artifact.json

# grab the download url from the saved artifact details using 'jq' json processor tool
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)

# fetch the artifact with the extracted download url using 'wget' tool
wget --http-user={user} --http-password={password} $artifactDownloadUrl

***
```
With the help of the documentation and the Slack forum, made a shell script and automated all the steps:

# Replace placeholders with actual values
user="project"
password="project"
nexus_ip="209.38.192.169"
node_repo="npm-hosted"

# save the artifact details in a json file
curl -u "$user":"$password" -X GET "http://$nexus_ip:8081/service/rest/v1/components?repository=$node_repo&sort=version" | jq "." > artifact.json

# grab the download url from the saved artifact details using 'jq' json processor tool
artifactDownloadUrl=$(jq '.items[].assets[].downloadUrl' artifact.json --raw-output)

# fetch the artifact with the extracted download url using 'wget' tool
wget --http-user="$user" --http-password="$password" "$artifactDownloadUrl"

Made a couple screenshots to proof, that the module was done.
```