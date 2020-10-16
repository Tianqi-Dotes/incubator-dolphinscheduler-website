Preparation

First, fork the dolphinscheduler code from the remote repository to your local repository.

Install MySQL/PostgreSQL, JDK and MAVEN in your own software development environment.

Clone your forked repository to the local file system.

git clone https://github.com/apache/incubator-dolphinscheduler.git

After finished the clone, go into the project folder and execute the following commands:

git branch -a  #check the branch
git checkout dev #shift the dev branch
git pull #sychronize the branch
mvn -U clean package -Prelease -Dmaven.test.skip=true  #need to compile the necessary classes due to the project use gRPC dependencies

Install node

Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh 

Refresh the environment variables
source ~/.bash_profile

Install node
nvm install v12.12.0
note:mac users could install npm through brew:brew install npm

Validate the node installation
node --version

Install zookeeper

Download zookeeper
http://apache.mirrors.hoobly.com/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz

Copy the zookeeper config file
cp conf/zoo_sample.cfg conf/zoo.cfg

Modify zookepper cofig
vi conf/zoo.cfg
dataDir=./tmp/zookeeper

Start/stop zookeeper
./bin/zkServer.sh start ./bin/zkServer.sh stop

Create database

Create user, user name: ds_user, password: dolphinscheduler
mysql> CREATE DATABASE dolphinscheduler DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> GRANT ALL PRIVILEGES ON dolphinscheduler.* TO 'ds_user'@'%' IDENTIFIED BY 'dolphinscheduler';
mysql> GRANT ALL PRIVILEGES ON dolphinscheduler.* TO 'ds_user'@'localhost' IDENTIFIED BY 'dolphinscheduler';
mysql> flush privileges;

Set up the front-end

Enter the dolphinscheduler-ui folder
cd dolphinscheduler-ui

Run command
npm install

Set up the back-end

Import the project to IDEA
file-->open

Modify the database configurations, the 'datasource.properties' under the resource folder which belongs to the dao module

    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/dolphinscheduler
    spring.datasource.username=ds_user
    spring.datasource.password=dolphinscheduler  
Modify the pom.xml under the root directory, change the scope of the mysql-connector-java to complie

Refresh the dao module
try to run the main method inside org.apache.dolphinscheduler.dao.upgrade.shell.CreateDolphinScheduler to insert tables and data needed for the project.

Modify the service module
try to change the zookeeper.quorum part of the zookeeper.properties file
zookeeper.quorum=localhost:2181

Modify the dolphinscheduler-ui module
try to change the .env file

API_BASE = http://localhost:12345
DEV_HOST = localhost

Start the project

Start zookeeper
./bin/zkServer.sh start

Start MasterServer
try to run the main method inside org.apache.dolphinscheduler.server.master.MasterServer and VM Options is needed:

    -Dlogging.config=classpath:logback-master.xml -Ddruid.mysql.usePingMethod=false
Start WorkerServer
try to run the main method inside org.apache.dolphinscheduler.server.worker.WorkerServer and VM Options is needed:

    -Dlogging.config=classpath:logback-worker.xml -Ddruid.mysql.usePingMethod=false
Start ApiApplicationServer
try to run the main method inside org.apache.dolphinscheduler.api.ApiApplicationServer and VM Options is needed:

    -Dlogging.config=classpath:logback-api.xml -Dspring.profiles.active=api
We are not going to start the other modules. if they are required to be started, check script/dolphinscheduler-daemon.sh and set them the same VM Options.

    if [ "$command" = "api-server" ]; then
      LOG_FILE="-Dlogging.config=classpath:logback-api.xml -Dspring.profiles.active=api"
      CLASS=org.apache.dolphinscheduler.api.ApiApplicationServer
    elif [ "$command" = "master-server" ]; then
      LOG_FILE="-Dlogging.config=classpath:logback-master.xml -Ddruid.mysql.usePingMethod=false"
      CLASS=org.apache.dolphinscheduler.server.master.MasterServer
    elif [ "$command" = "worker-server" ]; then
      LOG_FILE="-Dlogging.config=classpath:logback-worker.xml -Ddruid.mysql.usePingMethod=false"
      CLASS=org.apache.dolphinscheduler.server.worker.WorkerServer
    elif [ "$command" = "alert-server" ]; then
      LOG_FILE="-Dlogback.configurationFile=conf/logback-alert.xml"
      CLASS=org.apache.dolphinscheduler.alert.AlertServer
    elif [ "$command" = "logger-server" ]; then
      CLASS=org.apache.dolphinscheduler.server.log.LoggerServer
    else
      echo "Error: No command named \`$command' was found."
      exit 1
    fi

Start the front-end ui modules
cd dolphinscheduler-ui directory

And run 
npm run start

Visit the project
visit http://localhost:8888

Sign in with the administrator role
username: admin
password:dolphinscheduler123