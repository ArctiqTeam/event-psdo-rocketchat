# RocketChat Deployment

- Description

	This instruction set should enable a repetitive deployment of rocket-chat in an OpenShift environment. This application uses a persistent mongodb to ensure, and requires a persistent volume. 

- High-level deployment steps
	1. Download / modify template as necessary
	2. Create new project and add template
	3. Allow pod to run with anyuid
	4. Build new app with the template
	5. Modify rocket-chat database 
	6. Deploy rocket-chat app
	7. Login and test the app
- Deployment Output
	1. Download / modify template as necessary

			arctiq-smsair:.ssh sheastewart$ git clone [https://github.com/ArctiqTeam/event-psdo-rocketchat](https://github.com/ArctiqTeam/event-psdo-rocketchat) 
			Cloning into 'event-psdo-rocketchat'...
			remote: Counting objects: 4, done.
			remote: Compressing objects: 100% (4/4), done.
			remote: Total 4 (delta 0), reused 4 (delta 0), pack-reused 0
			Unpacking objects: 100% (4/4), done.
			Checking connectivity... done.
			arctiq-smsair:.ssh sheastewart$ ls
			event-psdo-rocketchat	github_rsa		github_rsa.pub		known_hosts
			arctiq-smsair:.ssh sheastewart$ cd event-psdo-rocketchat/

	2. Create new project and add template

			arctiq-smsair:.ssh sheastewart$ oc new-project rocketchat
			Now using project "rocketchat" on server "https://osm101.dev.arctiq.ca:8443".
			
			You can add applications to this project with the 'new-app' command. For example, try:
			
			 oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git
			
			to build a new example application in Ruby.
			arctiq-smsair:event-psdo-rocketchat sheastewart$ oc create -f rocket-chat-persistent.yaml 
			template "rocket-chat" created

	3. Allow the pod to run with any uid

			[root@osm101 ~]# oadm policy add-scc-to-user anyuid -z default

	4. Build new app with the template

			arctiq-smsair:event-psdo-rocketchat sheastewart$ oc new-app rocket-chat -p MONGODB_DATABASE=rocketchat,MONGODB_USER=rocketchat-admin,MONGODB_PASSWORD=rocketchat
			--> Deploying template rocket-chat
			
			 rocket-chat
			 ---------
			 Rocket.Chat with a MongoDB database running with a Persistent storage. Use this template if you want to run Rocket.Chat in production.
			
			 * With parameters:
			 * Memory Limit=512Mi
			 * Namespace=openshift
			 * Database Service Name=mongodb
			 * MongoDB User=rocketchat-admin
			 * MongoDB Password=rocketchat
			 * MongoDB Database Name=rocketchat
			 * MongoDB Admin Password=hfYJLgaMbNSDObes # generated
			 * Volume Capacity=1Gi
			
			--> Creating resources with label app=rocket-chat ...
			 persistentvolumeclaim "mongodb" created
			 deploymentconfig "mongodb" created
			 deploymentconfig "rocket-chat" created
			 route "rocket-chat" created
			 service "mongodb" created
			 service "rocket-chat" created
			--> Success
			 Run 'oc status' to view your app.

	5. Modify rocket-chat database

			arctiq-smsair:event-psdo-rocketchat sheastewart$ oc rsh mongodb-1-5nrtu
			sh-4.2$ mongo mongo localhost:27017
			MongoDB shell version: 3.2.6
			connecting to: mongo
			2016-11-11T16:41:35.988-0500 E - [main] file [localhost:27017] doesn't exist
			failed to load: localhost:27017
			sh-4.2$ mongo localhost:27017
			MongoDB shell version: 3.2.6
			connecting to: [localhost:27017/test](http://localhost:27017/test) 
			Welcome to the MongoDB shell.
			For interactive help, type "help".
			For more comprehensive documentation, see
				 [http://docs.mongodb.org/](http://docs.mongodb.org/) 
			Questions? Try the support group
				 [http://groups.google.com/group/mongodb-user](http://groups.google.com/group/mongodb-user) 
			> use rocketchat
			switched to db rocketchat
			> db.auth('rocketchat-admin','rocketchat')
			1
			> db.rocketchat_settings.update({_id:'Accounts_UseDNSDomainCheck'},{$set:{value:false}})
			WriteResult({ "nMatched" : 0, "nUpserted" : 0, "nModified" : 0 })
			> exit
			bye
			sh-4.2$ exit
			exit

	6. Deploy rocket-chat app

			arctiq-smsair:event-psdo-rocketchat sheastewart$ oc deploy rocket-chat —latest
			Started deployment #1
			Use 'oc logs -f dc/rocket-chat' to track its progress.
			arctiq-smsair:event-psdo-rocketchat sheastewart$ oc get pods 
			NAME READY STATUS RESTARTS AGE
			mongodb-1-5nrtu 1/1 Running 0 32m
			rocket-chat-1-epyyl 1/1 Running 0 3m

	7. Login and test the app

		![](https://s3-us-west-2.amazonaws.com/notion-static/f821707252944e8ba371ab0732d5cd56/Untitled)

---

- References / links 

	 [https://rocket.chat/docs/installation/automation-tools/openshift/](https://rocket.chat/docs/installation/automation-tools/openshift/) 

	 [https://github.com/ArctiqTeam/event-psdo-rocketchat](https://github.com/ArctiqTeam/event-psdo-rocketchat) 

	 [https://github.com/RocketChat/Rocket.Chat/tree/develop/.openshift](https://github.com/RocketChat/Rocket.Chat/tree/develop/.openshift) 

- Todo
	- [ ]  Validate SCC permissions

- Comments/Discussion