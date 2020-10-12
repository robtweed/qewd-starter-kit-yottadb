# Installing QEWD with YottaDB on Ubuntu Linux
 
Rob Tweed <rtweed@mgateway.com>  
10 October 2020, M/Gateway Developments Ltd [http://www.mgateway.com](http://www.mgateway.com)  

Twitter: @rtweed

Google Group for discussions, support, advice etc: [http://groups.google.co.uk/group/enterprise-web-developer-community](http://groups.google.co.uk/group/enterprise-web-developer-community)


# About this Repository

This repository provides instructions on how to install, configure and run QEWD on a Linux system,
using YottaDB (release 1.30) as the database.  It specifically applies to Ubuntu Linux, but with
appropriate changes where necessary, it should apply to most versions of Linux.


# Initial Steps / Pre-requisites

Clone this repository to your Linux system.  For example, to clone it
to the folder ~/qewd on your system:

        cd ~
        git clone https://github.com/robtweed/qewd-yottadb qewd

The instructions in this document will assume you've cloned it
to the ~/qewd folder.  Adjust the paths in the examples appropriately
if you cloned to a different folder on your Linux system.

QEWD has three key dependencies:

- Node.js must be installed.  The latest version 14 is recommended, but QEWD will also run
satisfactorily with versions 10 and 12.

- a Global Storage Database.  Here we're going to assume you'll be using YottaDB.  The latest
version (release 1.30) is recommended, but QEWD will also run satisfactorily with any earlier
version of YottaDB.

- The [mg-dbx](https://github.com/chrisemunt/mg-dbx) interface module used by QEWD 
to connect to YottaDB has to be built from
source during installation.  This assumes that a C++ compiler in available in your Linux
system.

To satisfy these dependencies:


## Install Node.js

If you don't have Node.js installed, the simplest approach is to use the 
[installation script](./install_node.sh)
included in this repository.  It will install the latest version 14.x build:

        cd ~/qewd
        source install_node.sh

You can test if it installed correctly by typing:

        node -v

If everything worked correctly, it should return the installed version, eg:

        v14.13.1


## Install YottaDB

If you don't have YottaDB installed, you can use the
[installation script](./yottadb-130/install.sh)
included in this repository.  It will install the latest Release 1.30:

        cd ~/qewd/yottadb-130
        source install.sh

You can test if the installation ran to completion by opening the
YottaDB shell.  Simply type the command:

        ydb

You should see the YottaDB shell prompt:

        YDB>

Try:

        YDB> w $zyrelease

You should see:

        YottaDB r1.30 Linux x86_64

Next, try setting a Global node, eg:

        YDB> set ^test("foo")="bar"

If it returns the *YDB>* prompt then it should be working OK.  Try listing the Global just
to be sure:

        YDB> zwr ^test

and you should see:

        ^test("foo")="bar"

        YDB>

Exit the shell:

        YDB> halt

and you should return to the Linux shell prompt.


## Install a C++ Compiler

Note: you can ignore this step if you've run the YottaDB installation script above.

Run the following commands:

        sudo apt-get update
        sudo apt-get install build-essential


These commands are safe to type even if you're unsure whether or not you've already
installed a C++ compiler on your Linux machine.

----


# Installing QEWD

You're now ready to install QEWD.  When you cloned this repo, you will have created the
file *~/qewd/package.json* which should look like this:

        {
          "name": "qewd-up",
          "version": "1.0.0",
          "description": "Automated QEWD Builder",
          "author": "Rob Tweed <rtweed@mgateway.com>",
          "scripts": {
            "start": "node node_modules/qewd/up/run_native"
          },
          "dependencies": {
            "qewd": "",
            "mg-dbx": ""
          }
        }

Note the two Node.js dependencies: the modules *qewd* and *mg-dbx*.

Installation of QEWD is a one-off step that makes use of this *package.json* file.
Simply type:

        cd ~/qewd
        npm install


All the dependent Node.js modules used by QEWD will be installed into a new folder named *node_modules*.  
On completion, your *~/qewd* folder should now contain:

        ~/qewd
            |
            |_ package.json
            |
            |_ package-lock.json
            |
            |_ configuration
            |
            |- node_modules



The second part of QEWD's installation is performed automatically when you start QEWD up
for the very first time.  Type:

        cd ~/qewd
        npm start

You should see the following:


        > qewd-up@1.0.0 start /home/rtweed/qewd
        > node node_modules/qewd/up/run_native
        
        mg-webComponents installed
        qewd-client installed
        
        *** Installation Completed.  QEWD will halt ***
        *** Please restart QEWD again using "npm start"

and you'll be returned to the Luinux shell prompt.

If you now check your ~/qewd folder, you'll find it contains:


        ~/qewd
            |
            |_ package.json
            |
            |_ package-lock.json
            |
            |_ configuration
            |
            |- node_modules
            |
            |- qewd-apps
            |      |
            |      |- qewd-monitor-adminui
            |
            |- www
            |   |
            |   |- components
            |   |
            |   |- qewd-monitor-adminui
            |   |
            |   |- mg-webComponents.js
            |   |
            |   |- qewd-client.js


Everything QEWD needs to run should now be present.


## Starting QEWD

Every time you want to start QEWD, simply type:

        cd ~/qewd
        npm start

You should see something like the following:

        > qewd-up@1.0.0 start /home/rtweed/qewd
        > node node_modules/qewd/up/run_native

        ** loading /home/rtweed/qewd/configuration/config.json
        Checking for onWorkerStarted path: /home/rtweed/qewd/orchestrator/onWorkerStarted.js
        Checking for onWorkerStarted path: /home/rtweed/qewd/onWorkerStarted.js
        ** results = {
          "routes": [],
          "config": {
            "managementPassword": "secret",
            "serverName": "QEWD-Up Server",
            "port": 3000,
            "poolSize": 2,
            "poolPrefork": false,
            "database": {
              "type": "dbx"
            },
            "webServerRootPath": "/home/rtweed/qewd/www/",
            "cors": true,
            "bodyParser": false,
            "mode": "production",
            "sessionDocumentName": "qs",
            "qewd_up": true,
            "moduleMap": {
              "qewd-monitor-adminui": "/home/rtweed/qewd/qewd-apps/qewd-monitor-adminui"
            }
          },
          "cwd": "/home/rtweed/qewd",
          "startupMode": "normal"
        }
        config: {
          "managementPassword": "secret",
          "serverName": "QEWD-Up Server",
          "port": 3000,
          "poolSize": 2,
          "poolPrefork": false,
          "database": {
            "type": "dbx"
          },
          "webServerRootPath": "/home/rtweed/qewd/www/",
          "cors": true,
          "bodyParser": false,
          "mode": "production",
          "sessionDocumentName": "qs",
          "qewd_up": true,
          "moduleMap": {
            "qewd-monitor-adminui": "/home/rtweed/qewd/qewd-apps/qewd-monitor-adminui"
          }
        }
        routes: []
        Double ended queue max length set to 20000
        webServerRootPath = /home/rtweed/qewd/www/
        Worker Bootstrap Module file written to node_modules/ewd-qoper8-worker.js
        ========================================================
        ewd-qoper8 is up and running.  Max worker pool size: 2
        ========================================================
        ========================================================
        QEWD.js is listening on port 3000
        ========================================================


If so, QEWD is ready and waiting, listening for incoming requests on port 3000.

Note: if you want to use a different listener port, edit the *~/qewd/configuration/config.json* file
and change the *port* property value.  Then stop and restart QEWD.


# Try the QEWD-Monitor Application

You can check if QEWD is working correctly by running the
*qewd-monitor* application that will now have been installed:

Start the QEWD-Monitor application in your browser using the URL:

        http://x.x.x.x:3000/qewd-monitor

or try the latest version:

        http://x.x.x.x:3000/qewd-monitor-adminui

Replace the *x.x.x.x* with the IP address or domain name of your Linux server.

You'll need to enter the QEWD Management password.  Use the value that you
specified in the *managementPassword* property in the *~/qewd/configuration/config.json* file.
This was pre-set to *secret*.

You'll now see the Overview panel, from where you can monitor your QEWD run-time environment, view the master and worker process activity.

If the *qewd-monitor* application works correctly, then you can be sure that QEWD
is working correctly and is ready for use.


# Stopping QEWD

if you're running QEWD as a foreground process in a terminal window, you can simply type *CTRL&C* to stop
QEWD.

Alternatively you can stop QEWD from within the *qewd-monitor* or *qewd-monitor-adminui* applications.
In the Overview or Processes screeens, click the stop button next to the *Master* process.  QEWD will
shut down and the *qewd-monitor* or *qewd-monitor-adminui* applications will no longer work.


# Start Developing

Now that you have QEWD up and running on your Linux system, you can begin developing both
REST APIs and/or interactive/WebSocket applications.

Your QEWD system can support both at once, and you can develop and run as many REST APIs as you
wish and as many simultaneous interactive applications as you wish.

From this point onwards, there's no difference in how you develop QEWD applications,
regardless of the Operating System you use, version of Node.js you use, or type of database
you use (YottaDB, Cach&eacute; or IRIS).  The only difference will be in file paths.

So you can now use the following tutorials:

- [this tutorial](https://github.com/robtweed/qewd-baseline/blob/master/INTERACTIVE.md)
explains how to develop interactive applications using the *qewd-client* browser module.
This is a useful tutorial to take as it will help to explain the basics of how
QEWD supports interactive, WebSocket message-based applications, and how you handle those messages
in your browser's logic.  Note that your version of QEWD includes the *qewd-client* module.

- once familiar with the basics covered in the tutorial above, 
can can find out how to develop a modern interactive WebSocket application whose front-end uses the  
[*mg-webComponents*](https://github.com/robtweed/mg-webComponents) framework
that has also been automatically installed in your QEWD system.
[See this document, starting at the *mg-webCOmponents Framework* section](https://github.com/robtweed/qewd-microservices-examples/blob/master/WINDOWS-IRIS-2.md#the-mg-webcomponents-framework)


- to develop REST APIs, get started with [this document](./REST.md)

- to find out more about how QEWD abstracts the YottaDB database as persistent 
JSON objects, see the [QEWD-JSdb documentation](https://github.com/robtweed/qewd-jsdb).
You can try the
QEWD-JSdb REPL by using the copy of the *jsdb_shell.js* module that is included in the Node.js REPL:

        cd ~/qewd
        node

You'll see the Node.js REPL shell response:

        Welcome to Node.js v14.13.1.
        Type ".help" for more information.
        > 

Now type:

        > var jsdb = require('./jsdb_shell')

Now you can try out any of the QEWD-JSdb APIs.  For example, remember that *test* global we
created when installing and testing YottaDB?  Try this:

        > var doc = jsdb.use('test')
        > doc.getDocument()

You should see:

        { foo: "bar" }

If you have an existing YottaDB system, you can use QEWD-JSdb to abstract any of your existing Global Storage.

To find out more about QEWD-JSdb, you can try out the 
[QEWD-JSdb REPL-based tutorial](https://github.com/robtweed/qewd-jsdb/blob/master/REPL.md#getting-started-with-qewd-jsdb).


## License

 Copyright (c) 2020 M/Gateway Developments Ltd,                           
 Redhill, Surrey UK.                                                      
 All rights reserved.                                                     
                                                                           
  http://www.mgateway.com                                                  
  Email: rtweed@mgateway.com                                               
                                                                           
                                                                           
  Licensed under the Apache License, Version 2.0 (the "License");          
  you may not use this file except in compliance with the License.         
  You may obtain a copy of the License at                                  
                                                                           
      http://www.apache.org/licenses/LICENSE-2.0                           
                                                                           
  Unless required by applicable law or agreed to in writing, software      
  distributed under the License is distributed on an "AS IS" BASIS,        
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
  See the License for the specific language governing permissions and      
   limitations under the License.      
