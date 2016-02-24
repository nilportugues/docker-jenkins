#Jenkins Docker 

Contains:

- Jenkins
- Java 8
- PHP7
- NodeJS
- NPM
- SQLite
- Ansible
- Ansistrano

## Jenkins Plugins

The following plugins come installed by default: 

```
postbuild-task:1.8
postbuildscript:0.17
jobgenerator:1.22
disable-failed-job:1.15
greenballs:1.15
s3:0.8
docker-plugin:0.16.0
diskcheck:0.26
timestamper:1.7.4
build-pipeline-plugin:1.4.9
naginator:1.16.1
git:2.4.2
ansicolor:0.4.2
build-monitor-plugin:1.8+build.201601112328
build-timeout:1.16
rebuild:1.25
heavy-job:1.1
cucumber-testresult-plugin:0.8.2
ssh-agent:1.9
role-strategy:2.2.0
git-changelog:1.7
```


## Jenkins SSH Key `id_rsa.pub`


Output the contents by issuing: 

```
docker run -it dockerjenkins_jenkins bash -c "cat /home/jenkins/.ssh/id_rsa.pub" 
```
This has to be done from your machine. 

## Password-less SSH for remote machines

For each machine, you'll have to pass your public id_rsa.pub. This should be done with a command that should be along the lines of:

```
docker run -it dockerjenkins_jenkins \ 
bash -c  "cat ./home/jenkins/.ssh/id_rsa.pub | ssh -L user@host -p22 'cat >> ./.ssh/authorized_keys'"
```
