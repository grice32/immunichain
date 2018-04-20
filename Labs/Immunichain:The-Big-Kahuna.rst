Part 1: Setting Up Your LinuxONE Community Cloud Account
======================================================================

**1.**  From your browser, go to https://developer.ibm.com/linuxone/

**2.** Click on Start your trial now

.. image:: Images/1.1.png

**3.** Complete the required fields, once filled in click on Request your trial

.. image:: Images/1.2.png

**4.** You will receive an email containing your User ID and Password. Follow the link to the LinuxONE CC

**5.** You’ll be at the website asking for your User ID and Password. Enter your credentials and click on Sign in

.. image:: Images/1.3.png

**6.** You’ll now be on the LinuxONE CC Home Page. Click on the Manage Instances under the Infrastructure section

.. image:: Images/1.4.png

**7.** Click on Create

**8.** Select the correct information::

  Type: General Purpose VM
  Instance Name: Immunichain
  Instance Description: (whatever you like)
  Image: SLES12 SP3
  Flavor: LinuxONE-Medium

.. image:: Images/1.5.png

**9.** Under the SSH Key Pair section, click on Create

.. image:: Images/1.6.png

**10.** In the pop-up box, enter a name you would like to give your keys. Then click on Create a new key pair

.. image:: Images/1.7.png

**11.** You will then be prompted to save the file (keys). Click on Save File. If you didn’t get prompted, it might of automatically download to your folders

**12.** Back on the LinuxONE CC, select your new keys in the SSH Key Pair section

**13.** Review all the selections you selected and click on Create

.. image:: Images/1.8.png

**14.** You will see the Instance starting up. Once it says ACTIVE in the status column you are good to go. Note your IP Address. It might be best to write down your IP Address. You can always toggle between your terminal and the LinuxONE CC website. 

.. image:: Images/1.9.png

 **15.** You can just ssh linux1@xxx.xxx.x.xx to connect to your Linux guest. If that doesn’t work jump to the next step. If that does work, jump to step 19

**16.** From your terminal, navigate to where you downloaded your keys

**17.** Modify the permissions of your private key by entering chmod 600 keyname.pem::

  keyname refers to what you named your keys

.. image:: Images/1.10.png

**18.** From the path where your keys are located enter ssh –i keyname.pem linux1@xxx.xxx.xxx.xxx:: 

  xxx.xxx.xxx.xxx refers to your LinuxONE Community Cloud IP Address

**19.** Type yes when you are prompted to continue connecting to your instance

.. image:: Images/1.11.png

**20.** You are now connected to your LinuxONE CC instance! 

Why stop now, we are just now having fun! Continue to the next part. 


Part 2: Getting Your Linux Guest Ready for Composer Playground
==============================================================

The previous part got you ready for Hyperledger Playground from the perspective of creating your LinuxONE Community Cloud Instance. This part will now get you ready from the perspective of your active Linux guest. Don’t worry, this part is extremely short!

**1.** Make a file named Linux1BlockchainScript.sh by enter the command below::

	touch Linux1BlockchainScript.sh

**2** Go into your vi editor by entering the command below::

	vi Linux1BlockchainScript.sh
	
**3** Once you are in the file, type the letter i (as in **ice**)to enter insert mode. Paste in the following lines to create your script::

	#!/bin/bash

	# Sanity checks
	relog=false
	# Check for docker group
	if ! $( id -Gn | grep -wq docker ); then
  	sudo usermod -aG docker linux1
  	echo "ID linux1 was not a member of the docker group. This has been corrected."
  	relog=true
	fi
	# Check PATH for /data/npm/bin
	if ! $( echo $PATH | grep -q /data/npm/bin ); then
  	echo "export PATH=/data/npm/bin:$PATH" >> $HOME/.profile
  	echo "PATH was missing '/data/npm/bin'. This has been corrected."
  	relog=true
	fi
	# Relog needed?
	if [[ "$relog" = true ]]; then
  	echo "Some changes have been made that require you to log out and log back in."
  	echo "Please do this now and then re-run this script."
  	exit 1
	fi
	# Ensure /data exists
	if [[ ! -d "/data" ]]; then
  	echo "/data disk is missing. It could take up to 10 minutes to format and mount the /data disk. Issue 'df -h' to 	 verify the /data disk is available before running this script again. When /data is available, please run this script 	      again."
       	exit 2
       	fi
       	# END Sanity checks

       	printf "
	IBM Master the Mainframe
	::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
	:::::::::::''  ''::'      '::::::  ::::::::::::::'.:::::::::::::::
	:::::::::' :. :  :         :::: :  :::::::::::.:::':::::::::::::::
	::::::::::  :   :::.       ::: M :::::::..::::'     :::: : :::::::
	::::::::    :':  '::'     '' M   M :::::: :'           '' ':::::::
	:'        : '   :  ::    . M       M   '                        .:
	:               :  .:: . M           M                         :::
	:. .,.        :::  ':: M M M       M M M                 .:...::::
	:::::::.      '      M   M   M   M   M   M               :: :::::.
	::::::::           M     M     M     M     M   '    '   .:::::::::
	::::::::.        ::: M   M           M   M :         ''' :::::::::
	::::::::::      :::::: M M           M M             :::::::::::::
	: .::::::::.   .:''::::: M           M   ::   :   '::.::::::::::::
	:::::::::::::::. '  '::::: M       M   :::::.:.:.:.:.:::::::::::::
	:::::::::::::::: :     ':::: M   M  ' ,:::::::::: : :.:'::::::::::
	::::::::::::::::: '     :::::: M    . :'::::::::::::::' ':::::::::
	::::::::::::::::::''   :::::::: : :' : ,:::vem:::::'      ':::::::
	:::::::::::::::::'   .::::::::::::  ::::::::::::::::       :::::::
	:::::::::::::::::. .::::::::::::::::::::::::::::::::::::.'::::::::
	IBM Master the Mainframe
	"


	#Install NodeJS
	echo -e “*** install_nodejs ***”
	cd /tmp
	wget -q https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-s390x.tar.gz
	cd /usr/local && sudo tar --strip-components=1 -xzf /tmp/node-v8.9.4-linux-s390x.tar.gz
	echo -e “*** Done withe NodeJS ***\n”

	#Setup and install docker-compose
	echo -e “*** Installing docker-compose. ***\n”
	sudo zypper install -y python-pyOpenSSL python-setuptools
	sudo easy_install pip
	sudo pip install docker-compose==1.13.0
	echo -e “*** Done with docker-compose. ***\n”

	#Install Hyperledger Composer Components
	echo -e “*** Installing Hyperledger Composer command line tools. ***\n”
	mkdir /data/linux1/ 
	npm config set prefix '/data/npm'
	npm config set cache /data/linux1/.npm
	export PATH=/data/npm/bin:$PATH
	cd /data/linux1/
	npm install -g composer-cli@0.19.0

	echo -e “*** Installing Hyperledger Composer rest server. ***\n”
	npm install -g composer-rest-server@0.19.0

	echo -e “*** Installing Hyperledger Composer playground. ***\n”
	npm install -g composer-playground@0.19.0

	echo -e "*** Clone and install the Coposer Tools repository.***\n"
	mkdir ~/fabric-tools && cd ~/fabric-tools
	curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-		dev-servers.tar.gz
	tar -xvf fabric-dev-servers.tar.gz
	export FABRIC_VERSION=hlfv11
	echo "export FABRIC_VERSION=hlfv11" >> $HOME/.profile
	./downloadFabric.sh
	./startFabric.sh
	./createPeerAdminCard.sh
	mkdir /data/playground/
	nohup composer-playground >/data/playground/playground.stdout 2>/data/playground/playground.stderr & disown
	sudo iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
	sudo iptables -I INPUT 1 -p tcp --dport 3000 -j ACCEPT
	sudo iptables -I INPUT 1 -p tcp --dport 1880 -j ACCEPT
	sudo bash -c "iptables-save > /etc/linuxone/iptables.save"

	#Install NodeRed
	echo -e "*** Installing NodeRed. ***\n"
	npm install -g node-red
	nohup node-red >/data/playground/nodered.stdout 2>/data/playground/nodered.stderr & disown

	# Persist PATH setting
	# Check PATH for /data/npm/bin
	if ! $( echo $PATH | grep -q /data/npm/bin ); then
  	echo "export PATH=/data/npm/bin:$PATH" >> $HOME/.profile
  	echo "PATH was missing '/data/npm/bin'. This has been corrected."
	fi

	# Persist docker group addition
	sudo usermod -aG docker linux1

	echo "Please log out of this system and log back in to pick up the group and PATH changes."
	
**4.** Hit ESC and then type in :wq to write and quit your vi editor tool. Once you have done this, you have created a Blockchain script::

	ESC + :wq

**5.** Make the file executable by entering chmod u+x Linux1BlockchainScript.sh::

	linux1@blockchain:~> chmod u+x Linux1BlockchainScript.sh

**6.** Enter ls -l again to see the file again

**7.** Enter df –h and if you do not see “/data” in the mounted column, wait a few moments before going onto the next step::

  linux1@blockchain:~> df -h
  Filesystem      Size  Used Avail Use% Mounted on
  devtmpfs        2.0G  8.0K  2.0G   1% /dev
  tmpfs           2.0G     0  2.0G   0% /dev/shm
  tmpfs           2.0G  219M  1.7G  12% /run
  tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
  /dev/dasda2     5.8G  3.2G  2.4G  58% /
  /dev/dasda1     388M   47M  322M  13% /boot/zipl
  /dev/dasdb1      45G  5.0G   37G  12% /data
  tmpfs           391M     0  391M   0% /run/user/1001

**8.** Now, run the file by entering ./Linux1BlockchainScript.sh – Be patient, this script will take 7 to 10 minutes to run. If it doesn’t want to run, you might need to exit out of your Linux guest and sign back in::

	linux1@blockchain:~> ./Linux1BlockchainScript.sh

**9.** The first time you run the script you will need to exit in order for permissions and environment variables to take effect. You can do this by entering exit once you get your command line back

**10.** Now you can log back into your Linux guest

**11.** Now, verify that you have running Hyperledger Fabric Docker containers network by entering docker ps –a

.. image:: Images/2.1.png

Congratulations if you just did all of this successfully. You just did the hard part. In next part we will start Immunichain. 


Part 3: Web Immunichain
=======================

**1.** Go to your http://composer-playground.mybluemix.net within a browser. Google Chrome is preferred, but Firefox works just as well::

	http://composer-playground.mybluemix.net

**2.** You will get a Welcome pop-up box with a graphic and a few words. Click on Let’s Blockchain

.. image:: Images/3.1.png

**3.** Then you will be in the Composer Playground homepage. Click on Deploy a New Business Network. Make sure it says Web Browser in the top right.

.. image:: Images/3.2.png

**4.** Select empty-business-network. Then give your business network a name and a description:: 

	Select: empty-business-network
	Business Network Name: immunichain
	Description: tracking of immunization records

**5.** Then click on Deploy 

Afterwards, you can come back to the Composer Playground play with some of the other sample business network applications, like animal tracking or vehicle lifecycles.

**6.** You will then be taken to Your Wallet. Your wallet is basically a quick, seamless connection to multiple connections that you can jump around with. You will see later how easy it is. Click on Connect now in order to get connected to our immunichain network

.. image:: Images/3.5.png

**7.** Fill in your model file with the below text::

  /* Immunization definitions */

  namespace ibm.wsc.immunichain

  participant Guardian identified by gid {
	o String gid
	o String name
  }

  participant MedProvider identified by medid {
	o String medid
	o String name
  }

  abstract concept immunization {
	o String name
	o String provider
	o String imdate
  }

  concept immunirecord extends immunization {}

  asset Childform identified by cid {
	o String cid
	o String name
	o String address
	--> Guardian guardian
	o String dob
	--> MedProvider [] medproviders optional
	o immunirecord [] immunizations
  }
 
  transaction assignMedProvider {
	--> Guardian guardian
	--> MedProvider medprovider
	--> Childform childform
  }

  transaction authMember {
	--> Guardian guardian
	--> Childform childform
  }

  transaction removeMemberAuth {
	--> Guardian guardian
	--> Childform childform
  }

  transaction addImmunizations {
	o immunirecord [] vaccines
	--> Childform childform
  }

  transaction updateChildForm {
	o String name optional
	o String address optional
	--> Childform childform
  }

  transaction reassignGuardian {
  --> Guardian oldguardian
  --> Guardian newguardian
  --> Childform childform
  }

**8.** Then click on Add a File in the bottom left. Then select Script File (.js) and click on Add. 

.. image:: Images/3.6.png

**9.** Replace the content of the Script file with the following below::

  'use strict';

  /**
 	* Add medical provider to child record
  * @param {ibm.wsc.immunichain.assignMedProvider} assignMedProvider - the assignMedProvider transaction
 	* @transaction
 	*/
  function assignMedProvider(assignMedProvider) {
  	var guardian = assignMedProvider.guardian;
  	var child = assignMedProvider.childform;
  	var medprovider = assignMedProvider.medprovider;
  	child.medproviders.push(medprovider);
  
  	return getAssetRegistry('ibm.wsc.immunichain.Childform')
   	.then(function(result) {
    	return result.update(child);
  	});
  }

  /**
  * Add immunization(s) to child record
  * @param {ibm.wsc.immunichain.addImmunizations} addImmunizations - the addImmunizations transaction
 	* @transaction
 	*/
  function addImmunizations(addImmunizations){
	var vaccines = addImmunizations.vaccines;
	var child = addImmunizations.childform;
 	var immunizations = child.immunizations;
  /*  	if (immunizations[0].name == 'default'){
    	immunizations.splice(0,1) 
    	} */
  	immunizations.push.apply(immunizations,vaccines);
  
	return getAssetRegistry('ibm.wsc.immunichain.Childform')
		.then(function(ChildRegistry){
			//save the childform
			return ChildRegistry.update(child);
		});
  }

  /**
 	* Update information on child record, can only be done by guardian
  * @param {ibm.wsc.immunichain.updateChildForm} updateChildForm - the updateChildForm transaction
 	* @transaction
 	*/
  function updateChildForm(updateChildForm){
  	var newaddress = null;
  	var newname = null;
    	var child = updateChildForm.childform;
  	newaddress = updateChildForm.address;
  	newname = updateChildForm.name;
  
  	if (newaddress != null && newname != null){
    	child.name = newname;
      	child.address = newaddress;
    	}
  	else if (newaddress != null){
    	child.address = newaddress;
    	}
 	else if (newname != null){
    	child.name = newname;
    	}
	return getAssetRegistry('ibm.wsc.immunichain.Childform')
		.then(function(ChildRegistry){
			//save the childform
			return ChildRegistry.update(child);
		});
  }

  /**
 	* Assign child to his/herself when he/she is of legal age
  * @param {ibm.wsc.immunichain.reassignGuardian} reassignGuardian - the reassignGuardian transaction
  * @transaction
 	*/
  function reassignGuardian(reassignGuardian) {
  	var oldguardian = reassignGuardian.oldguardian;
  	var newguardian = reassignGuardian.newguardian;
  	var child = reassignGuardian.childform;
  	child.guardian = newguardian;
  
  	return getAssetRegistry('ibm.wsc.immunichain.Childform')
    	.then(function(result) {
    	return result.update(child);
  	});
  }

  /**
 	* Get the immunizations for a child
 	* @query
 	* @param {String} cid - the unique id assigned to the childform
 	* @returns {immunirecord[]} - the immunizations that the child has gotten
  */
  function listImmunizations(cid) {
  	return query('select x.immunizations from Childform where x.cid ==: cid');
  }

**10.** Then click on Deploy Changes in the bottom left. 

.. image:: Images/3.7.png

In 0.19.0, they changed Update to Deploy Changes. Even in the Bluemix version of Composer, you are deploying this network to Fabric. When you hit the Deploy Changes button, you have to give that chaincode a version, but it has to go in sequential order. For example, 0.0.1 would move to 0.0.2 – thus cannot go from 0.0.1 to 0.0.3. If you were not to rename your chaincode version, the deploy-0 to deploy-1 would also work due to the sequential requirement. 

**11.** After you have done that, your screen should look like this. If it does, then we are in business (get it? In business, business network – great!)

.. image:: Images/3.8.png


Part 4: Creating Assets and Participants
========================================

**1.** Now that you have an Immunichain Business Network, jump over to the Test section of the Composer Playground. The test area allows you to create assets, participants and submit transactions against your assets and participants. Your screen should look like this: 

.. image:: Images/4.1.png

Before we create assets and participants, we need to know what each asset and participants represent. 
	- Guardian is the parent
	- MedProvider is simply a medical provider, like a doctor
	- Childform is simply the child or the asset in this business network

**2.** Now create a Guardian by clicking on +Create New Participant. Give the Guardian a number. I stick to 1, 2, 3 or low numbers that you can remember, but you can create any ID number you want. I suggest writing your ID numbers down as we move along. Once you have filled in the information click on Create

.. image:: Images/4.2.png

.. image:: Images/4.3.png

**3.** Once you have created a Guardian, your screen should look like this: 

.. image:: Images/4.4.png

**4.** Go ahead and make a Medical Provider. Remember the Medical Provider number you create

.. image:: Images/4.5.png

**5.** Now, let’s make a child. Click on optional properties at the bottom first. Assign him to the guardian you just created a step ago

.. image:: Images/4.6.png

**6.** Your screen should look like this when you are done:

.. image:: Images/4.7.png

**7.** Go ahead and create more medical providers, guardians and children. Just to remember to write down the ID numbers. This will make more sense when we submit transactions. 


Part 5: Submitting Transactions
===============================

**1.** Now, click on Submit Transaction in the bottom left and let’s authorize a member to view the health record of our child. You can change the type of transaction you want by click on the middle grey box. I have it in a square below

.. image:: Images/5.1.png

**2.** Now, let’s make an authorized medical provider transaction. Here is my transaction. You can make any type of transaction you want here

.. image:: Images/5.2.png

My transaction says let medical provider #1 (HealthQuest) have Child #1’s (Emily) healthcare record. This also means that HealthQuest can administer immunization shots to Emily.

**3.** You can view this transaction by clicking on childform on the left and then Show All on Emily. Notice that member 1 is now in Emily’s description

.. image:: Images/5.3.png

**4.** Click on Submit Transaction and then change the transaction type to addImmunizations. The format to add an immunization is a little different. In the Vaccine section put { "name" : "immunization", "provider" : "medical provider", "imdate" : "date" } inbetween the brackets. Replace the immunization, medical provider and date with whatever you would like. Here is what my transaction looks like::

  { "name" : "immunization", "provider" : "medical provider", "imdate" : "date" }

.. image:: Images/5.4.png

**5.** To view your immunization, go your child in the Childform section

.. image:: Images/5.5.png

**6.** Once you submit the transaction and it is good, click on All Transactions in the bottom left. This is what Composer likes to call the Historian. Now is a good time to tell you about the Historian. The Historian is the sequence of transactions or addition or removal of participants or assets. I didn’t tell you to look at the Historian when you were creating the Participants and Assets, but the Historian kept track of when and what type of participant or asset you created. You can scroll to the bottom to view the first transaction you created, which should be the Medical Provider - HealthQuest - or whatever you called it. You can see by clicking on view record. 

.. image:: Images/5.6.png

.. image:: Images/5.7.png

**7.** Continue to make various transactions that you want

**8.** When you are done, click on Export from the Define section. This will export your business network as an .bna file. You can take this .bna file and deploy that network on other Composer-Playground interfaces. 

.. image:: Images/5.8.png


Part 6: Deploying Your Business Network to Hyperledger Fabric
=============================================================

**1.** Go to your IP address with the port of 8080 with the instance you created with the LinuxONE Community Cloud::

	148.100.xxx.xxx:8080

**2.** You are welcomed to the homepage of Composer Playground. Click on Deploy a New Business Network, to the right of the PeerAdmin card. It should say hlfv1 in the top left.

**3.** Drop in your immunichain.bna file that you just exported. Drop it in the “Drop Here to Upload or Browse” box

.. image:: Images/6.1.png

**4.** Then scroll down and select ID and Secret. For Enrollment ID enter admin and for Enrollment Secret enter adminpw. Scroll back up and click on Deploy::

	Enrollment ID: admin
	Enrollment Secret: adminpw

**5.** Then you will be in your Wallet. You will see a second card right next to the PeerAdmin. Click on Connect Now on the new card your just created. Mine says admin, but it might be called something else in your Playground. 

.. image:: Images/6.2.png

**6.** Now you are connected to a running Fabric. To verify that you actually are, go to your command line and enter docker ps –a and notice a docker container that starts out as dev-peer0 

.. image:: Images/6.3.png


Part 7: Creating Assets and Participants
========================================

This section is very similar to Part 4. You are going to create assets and participants in our Immunichain network. This time connected to the Hyperledger Fabric. Whenever you connect Composer to a running Fabric, you deploy your running business network (BNA file) as a chaincode as a Docker container. The different participants and assets you create in this network are going to be stored in the chaincode. This means whenever you update the network, the chaincode will be updated. Let’s say you want to add another participant type to our network, the chaincode will update to represent the additional participant. 

**1.** Now that you have an Immunichain Business Network connected to the Hyperledger Fabric, jump over to the Test section of the Composer Playground. The test area allows you to create assets, participants and submit transactions against your assets and participants. Your screen should look like this: 

.. image:: Images/7.1.png

Before we create assets and participants, we need to know what each asset and participants represent. 
	- Guardian is the parent
	- MedProvider is simply a medical provider, like a doctor
	- Childform is simply the child or the asset in this business network

**2.** Now create a Guardian by clicking on +Create New Participant. Give the Guardian a number. I stick to 1, 2, 3 or low numbers that you can remember, but you can create any ID number you want. I suggest writing your ID numbers down as we move along. Once you have filled in the information click on Create

.. image:: Images/7.2.png

.. image:: Images/7.3.png

**3.** Once you have created a Guardian, your screen should look like this: 

.. image:: Images/7.4.png

**4.** Go ahead and make a Medical Provider. Remember the Medical Provider number you create

.. image:: Images/7.5.png

**5.** Now, let’s make a child. Click on optional properties at the bottom first. Assign him to the guardian you just created a step ago

.. image:: Images/7.6.png

**6.** Your screen should look like this when you are done:

.. image:: Images/7.7.png

**7.** Go ahead and create more medical providers, members, guardians and children. Just to remember to write down the ID numbers. This will make more sense when we submit transactions. 


Part 8: Adding a Participant Type and Transactions
==================================================

So far, everything has been a bit easy. Now, we are going to add a participant type and some transaction code for that new participant. It is important to follow the instructions as to where to add the code.

**1.** Head into your model file by going to the Define section and clicking on the Model File

.. image:: Images/8.1.png

**2.** On line 15, add in this participant::

  participant Member identified by memid {
	o String memid
	o String name
  }

.. image:: Images/8.2.png

**3.** On line 35, add in this line in the asset childform::

  --> Member [] members optional

.. image:: Images/8.3.png

**4.** On line 47, add in this line in the transaction authMember::

  --> Member member

.. image:: Images/8.4.png

**5.** On line 53, add in this line in the transaction removeMemberAuth::

  --> Member member

.. image:: Images/8.5.png

**Note** What other participants or assets could you see being added the Immunichain Blockchain network? Collaborate with a few people around you to gather ideas. Later you can add these participants and assets to your network. 

Now, let’s add some transactions.

**6.** Switch to the Script File (.js) in the Define Section

.. image:: Images/8.6.png

**7.** On line 20, add in this transaction::

  /**
 	* Authorize member to child record
  * @param {ibm.wsc.immunichain.authMember} authMember - the authMember transaction
 	* @transaction
 	*/
  function authMember(authMember) {
  	var guardian = authMember.guardian;
  	var child = authMember.childform;
  	var member = authMember.member;
  	child.members.push(member);
  	return getAssetRegistry('ibm.wsc.immunichain.Childform')
    	.then(function(ChildRegistry) {
    	return ChildRegistry.update(child);
  	});
  }

.. image:: Images/8.7.png

**8.** On line 36, add in this transaction as well::

  /**
 	* Deauthorize member to child record, so remove from members list
  * @param {ibm.wsc.immunichain.removeMemberAuth} removeMemberAuth - the removeMemberAuth transaction
 	* @transaction
 	*/
  function removeMemberAuth(removeMemberAuth) {
	var guardian = removeMemberAuth.guardian;
	var child = removeMemberAuth.childform;
	var member = removeMemberAuth.member;
	var mem = child.members;
	var idx = mem.indexOf(member);

	//if the member is in the array of Members, we can remove it
	if (idx !== -1){
	mem.splice(idx,1);
	}

	return getAssetRegistry('ibm.wsc.immunichain.Childform')
	.then(function(result) {
	return result.update(child);
            });
  }

Look at the picture below to get a sense of what to do.

.. image:: Images/8.8.png

**9.** Click on Deploy Changes to update your business network. Due to 0.19.0 in Hyperledger Composer, you will get a pop up asking for an installation card and upgrade card. Choose the PeerAdmin@hlfv1 card and click upgrade. You will see this pop up every time you upgrade your chaincode version.


Part 9: Submitting Transactions
===============================

**1.** Now that we have a new participant type, let’s create one. Jump to the test section and click on Member on the left. 

.. image:: Images/9.1.png

**2.** Click on Create New Participant and follow the steps below to add a Member.

.. image:: Images/9.2.png

**3.** Now that we have created a Member, let’s make some transactions. Click on Submit Transaction in the bottom left.

**4.** A pop-up will appear with the transaction of adding Immunizations in the grey box. Switch to assignMedProvider to assign a Medical Provider to one of the children you’ve created

**5.** Now, replace the ID Numbers to replicate the guardian, medical provider and child. Look at the below picture to get a sense of what to do

.. image:: Images/9.3.png

That basically says, assign medical provider #1 (Healthquest) to Child #1 (Emily).

**6.** Click Submit once you have the ID Numbers you want

**7.** Once you submit the transaction and it is good, click on All Transactions in the bottom left. This is what Composer likes to call the Historian. Now is a good time to tell you about the Historian. The Historian is the sequence of transactions or addition or removal of participants or assets. I didn’t tell you to look at the Historian when you were creating the Participants and assets, but the Historian kept track of when and what type of participant or asset you created. You can scroll to the bottom to view the first transaction you created, which should be the Medical Provider, HealthQuest or whatever you called it. You can see by clicking on view record. 

.. image:: Images/9.4.png

**8.** Back to our transaction, click on the Childform on the left. Find the child you assigned a Medical Provider to. Click on Show All to view the entire asset of your child. Notice the medical provider you assigned it to? 

.. image:: Images/9.5.png

**9.** Should we do another transaction? Of course! This time we will add a member to our child. To do this, we need to go back to our Child. 

**10.** Then click on the pencil in the top right of our child’s box.

.. image:: Images/9.6.png

**11.** Click on Optional Properties. You will notice the member section appearing now. Then click on Update.

.. image:: Images/9.7.png

**12.** Now, click on Submit Transaction and let’s authorize a member to view the health record of our child. You can change the type of transaction you want by clicking on the middle grey box. 

**13.** Now, let’s make an authorized member transaction. Here is my transaction. You can make any type of transaction you want here

.. image:: Images/9.8.png

My transaction says let member #1 (High School) have Child #1’s (Emily) health record. This would be extremely useful when every year millions of kids get physicals in order to play a sport. Imagine having your medical provider authorize your child’s health record to approve them playing a sport. I know my mom would’ve enjoyed not going up to the High School an additional time. 

**14.** You can view this transaction by clicking on childform on the left and then Show All on Emily. Notice that member 1 is now in Emily’s description

.. image:: Images/9.9.png

**15.** We have submitted some transactions, but now let’s actually add some immunizations to a child

**16.** Click on Submit Transaction and then change the transaction type to addImmunizations. The format to add an immunization is a little different. In the Vaccine section put { "name" : "immunization", "provider" : "medical provider", "imdate" : "date" } in-between the brackets. Replace the immunization, medical provider and date with whatever you would like. Here is what my transaction looks like::

  { "name" : "immunization", "provider" : "medical provider", "imdate" : "date" }

.. image:: Images/9.10.png

**17.** To view your immunization, go your child in the Childform section

.. image:: Images/9.11.png

**18.** Continue to make various transactions that you want


Part 10: Modifying Permissions and Creating Identities
======================================================

If you were to go to the permissions.acl file in the Define section, you would notice that there aren’t many rules in our network. In fact, the rules there mean anyone in the network can create, update, delete and submit transactions in the network. This doesn’t actually replicate what would happen in a real Immunichain business network. In this section we are going to change the permissions to the business network. You will notice these permissions by submitting transactions with the various participant identities you are about to create. 

**1.** Go to the Define section of Composer Playground. Then click on admin in the top right. Then click on ID Registry

.. image:: Images/10.1.png

**2.** We are doing great if this is what your page looks like

.. image:: Images/10.2.png

**3.** Click on Issue New ID

**4.** A pop-up will appear. Give your identity a name (disclaimer: the identity will be tied to a participant you created earlier in the lab; ie: Guardian Austin, Medical Provider HealthQuest). Then type in the number 1. You should now see the various participants that have an ID number of 1. If you gave your participants a different ID number, you won’t see anything by typing in 1. Instead, type in the number you gave to your participants. Here is what I did below:

.. image:: Images/10.3.png

**5.** If your screen looks like this, then we are in good shape

.. image:: Images/10.4.png

**6.** Go ahead and create other identities for your participants

**7.** I have a total of 4 identities in my business network. Here is what my screen looks like. You could have more identities if you created more, depending on how many participants your created in Part 2

.. image:: Images/10.5.png

**8.** Since we are in the admin identity (make sure you see admin in the top right), lets change our permissions file. Click on Define and then Access Control in the bottom left.

.. image:: Images/10.6.png

**9.** In the permissions.acl file, copy all that is below::

  rule UpdatePersonal {
      description: "Allow the guardian update the child's personal info"
      participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource(c): "ibm.wsc.immunichain.Childform"
    	transaction(tx): "ibm.wsc.immunichain.updateChildForm"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule txUpdatePersonal {
    	description: "Allow the guardian to update the child assets"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource: "ibm.wsc.immunichain.updateChildForm"
    	action: ALLOW
  }

  rule AssignProvider {
    	description: "Allow the guardian to assign and update medical providers"
    	participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: UPDATE
    	resource(c): "ibm.wsc.immunichain.Childform"
    	transaction(tx): "ibm.wsc.immunichain.assignMedProvider"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule txAssignProvider {
    	description: "Allow the guardian to assign and update medical providers"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource: "ibm.wsc.immunichain.assignMedProvider"
    	action: ALLOW
  }

  rule AuthMembers {
    	description: "Allow the guardian to authorize member organizations"
    	participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: UPDATE
    	resource(c): "ibm.wsc.immunichain.Childform"
    	transaction(tx): "ibm.wsc.immunichain.authMember"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule txUAuthMembers {
    	description: "Allow the guardian to authorize member organizations"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource: "ibm.wsc.immunichain.authMember"
    	action: ALLOW
  }

  rule DeauthMembers {
    	description: "Allow the guardian to deauthorize member organizations"
    	participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: UPDATE
    	resource(c): "ibm.wsc.immunichain.Childform"
    	transaction(tx): "ibm.wsc.immunichain.removeMemberAuth"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule txDeauthMembers {
    	description: "Allow the guardian to deauthorize member organizations"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource: "ibm.wsc.immunichain.removeMemberAuth"
    	action: ALLOW
  }

  rule Reassign {
    	description: "Allow the guardian to reassign their children (if of age)"
   	  participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: UPDATE
    	resource(c): "ibm.wsc.immunichain.Childform"
    	transaction(tx): "ibm.wsc.immunichain.reassignGuardian"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule txReassign {
    	description: "Allow the guardian to reassign their children (if of age)"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: ALL
    	resource: "ibm.wsc.immunichain.reassignGuardian"
    	action: ALLOW
  }

  rule GuardianRead {
    	description: "Allow guardians to view their own child's health record"
    	participant(g): "ibm.wsc.immunichain.Guardian"
    	operation: UPDATE, READ
    	resource(c): "ibm.wsc.immunichain.Childform"
    	condition: (c.guardian.getIdentifier() == g.getIdentifier())
    	action: ALLOW
  }

  rule readMembers {
    	description: "Allow Guardian to view the Member"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: READ
    	resource: "ibm.wsc.immunichain.Member"
    	action: ALLOW
  }

  rule readMedicalProviders {
      description: "Allow the Guardian to read the Medical Providers in the network"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: READ
    	resource: "ibm.wsc.immunichain.MedProvider"
    	action: ALLOW
  }

  rule addChild {
    	description: "Allow the Medical Provider to add a child in the network"
    	participant: "ibm.wsc.immunichain.MedProvider"
    	operation: CREATE
    	resource: "ibm.wsc.immunichain.Childform"
    	action: ALLOW
  }

  rule CreateChild {
    	description: "Allow the Guardian to add a child in the network"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: CREATE
    	resource: "ibm.wsc.immunichain.Childform"
    	action: ALLOW
  }

  rule MedicalProviderRead {
      description: "Allow Medical Providers to view children that have them as a medical provider"
    	participant(g): "ibm.wsc.immunichain.MedProvider"
    	operation: UPDATE, READ
    	resource(c): "ibm.wsc.immunichain.Childform"
    	condition: (c.medproviders.some(function(MedProvider) {
    	return MedProvider.getIdentifier() == g.getIdentifier();
    	}))
    	action: ALLOW
  }

  rule medRead1 {
      description: "Allow the Medical Providers to read all the members available in the network"
    	participant: "ibm.wsc.immunichain.MedProvider"
    	operation: READ
    	resource: "ibm.wsc.immunichain.Member"
    	action: ALLOW 
  }

  rule medRead2 {
      description: "Allow the Medical provider to view all the guardian's in the network"
    	participant: "ibm.wsc.immunichain.MedProvider"
    	operation: READ
    	resource: "ibm.wsc.immunichain.Guardian"
    	action: ALLOW 
  }

  rule MemRead {
    	description: "Allow the Members to view all the Children in the network"
    	participant: "ibm.wsc.immunichain.Member"
    	operation: READ
    	resource: "ibm.wsc.immunichain.Childform"
    	action: ALLOW  
  }

  rule medUser {
      description: "Allow the Medical provider to access the network"
    	participant: "ibm.wsc.immunichain.MedProvider"
    	operation: READ
    	resource: "org.hyperledger.composer.system.*"
    	action: ALLOW 
  }

    rule memberUser {
      description: "Allow the Member to access the network"
    	participant: "ibm.wsc.immunichain.Member"
    	operation: READ
    	resource: "org.hyperledger.composer.system.*"
    	action: ALLOW 
  }

  rule GuardanUser {
      description: "Allow the Guardian to access the network"
    	participant: "ibm.wsc.immunichain.Guardian"
    	operation: READ
    	resource: "org.hyperledger.composer.system.*"
    	action: ALLOW 
  }

**10.** Now, paste all that you have copied above the two rules you currently have. Here is what I my screen looks like now:

.. image:: Images/10.8.png

**11.** Once you are good to go, click on Deploy Changes in the bottom left and that will make changes across the entire business network. Read through some of the rules that we just implemented. What do you think will change as we go through the various identities?

.. image:: Images/10.9.png

**12.** Click on admin in the top right again. This time, click on My Business Networks. This will take us to the Composer Playground homepage

**13.** Now your screen should look like this:

.. image:: Images/10.10.png

When you created the identities, Composer was creating ID Cards for those identities. That is why I have 4 ID Cards. They are all tied to the Immunichain business network and to the participants you created earlier. You could think of this as a 4 peer Blockchain network, with 1 of the peers being an admin who oversees the entire network. 

**14.** Go ahead and click on Connect Now with your Guardian ID.

.. image:: Images/10.11.png

**15.** You are now in the Guardian’s perspective in the Immunichain business network. Go ahead and click on the other participants in the Test section

Medical Providers:

.. image:: Images/10.12.png

Members: 

.. image:: Images/10.13.png

Child: 

.. image:: Images/10.14.png

What did you notice about the permissions here? From the Guardian perspective, you can view all the Medical Providers, Members and Children that the Guardian has ownership of. 

**16.** Go ahead and update your Child by clicking on the pencil in the top right. Delete the Medical Providers and Members

.. image:: Images/10.15.png

.. image:: Images/10.16.png

**17.** Submit transaction from the Guardian perspective. Start with assigning a Medical Provider. 

.. image:: Images/10.17.png

**18.** Submit another transaction by assigning a Member

.. image:: Images/10.18.png

From the Guardian perspective, you are able to do a lot of different things. First, you can view the Children in the network that the Guardian has ownership of. Also, the guardian can create additional children with the way the permissions are set up. Do you think this is a viable option in a production environment? I would say no, but rather you should have the Medical Provider, who administered the birth of the Child, create the Child asset. In a production environment, this would be negotiated between all the participants in the business network. Also, as the Guardian you can also view all the Members and Medical Providers. Why do you think that is so? When you have a child as a guardian you want to be able to view all the options you have as possible Medical Providers and Members. In a real-world scenario, maybe the Guardian would only view all the Medical Providers that are tied to their Health Insurance, but that would require an Insurer in this Immunichain business network. Maybe in the future :) 

**19.** I think you’re getting the sense from the Guardian perspective. Before we jump to another perspective, delete all Members. You previously did this from step 16 in this part. Once you have successfully done that, go ahead and switch to the Medical Provider perspective. Click on My Business Networks in the top right. Then click on Connect Now on the Medical Provider

.. image:: Images/10.19.png

**20.** Click around on the other participants in the Immunichain Business Network

Guardian: 

.. image:: Images/10.20.png

Members:

.. image:: Images/10.21.png

Child: 

.. image:: Images/10.22.png

**21.** Click on Submit Transaction. Start with assigning a Member

.. image:: Images/10.23.png

**22.** Now, create another Child asset. Have the Child’s guardian be the first Guardian. In my business network, this would be Guardian Austin. 

.. image:: Images/10.24.png

.. image:: Images/10.25.png

If you noticed, I now have TWINS! My life suddenly got crazy for a 23-year-old. I guess I need to continue work in order to support them. Or just become a crypto-currency millionaire (I don’t know if that’s possible these days). 

On a slightly more serious note, maybe having the Medical Provider create additional children isn’t the best idea. It really depends on who the Medical Provider is. Is it the hospital? Or more specifically, is the Medical Provider the doctor who works in the baby delivery department of the hospital? Should the Medical Provider be able to create the child, or should we leave it up to the Guardians to create the children? These types of conversations have to occur between the peers in the business network if this was to be a production environment. 

**23.** Great, we just created another Child. Jump back over to the Guardian perspective. Did the new Child show up? 

.. image:: Images/10.26.png

**24.** Go ahead and only assign a Medical Provider to the new Child by submitting a transaction 

**25.** Should we jump to the Member perspective? Absolutely! 

.. image:: Images/10.27.png

**26.** Look around at the various participants in the Immunichain business network

Child: 

.. image:: Images/10.28.png

**27.** If you noticed, all the children showed up. Click on Show All on the Bobbie, you notice that this member isn’t listed as one her authorized Members.

.. image:: Images/10.29.png

Is this a good thing – that Bobbie appeared to this member? Absolutely not. This would be a non-negotiable in the business network. You wouldn’t want a Member to be able to see a Child, unless it has authorization. Could you imagine a Member being able to read all the Immunization records of every Child? We have to modify the permissions in our Access Control file. 

See if you can modify the rule in the Access Control file in the Define section. 


Part 11: Admin Immunichain Rest Server
======================================

In this section you are going to open your business network to a REST server. You are going to act as the admin of Immunichain. This means you can submit any type of transaction you want due to the REST server being from the admin’s perspective. 

**1.** Enter composer card list from your command line and you should see your cards with the Business Network of Immunichain::

  linux1@blockchain:~> composer card list

**2.** Now enter composer-rest-server and enter the same information as I have shown in the picture below. Make sure your card name matches what your composer card list output represented. We only want the admin card::

  linux1@blockchain:~> composer-rest-server
  ? Enter the name of the business network card to use: admin@immunichain  ## or whatever you called your admin card
  ? Specify if you want namespaces in the generated REST API: always use namespaces
  ? Specify if you want to enable authentication for the REST API using Passport: No
  ? Specify if you want to enable event publication over WebSockets: Yes
  ? Specify if you want to enable TLS security for the REST API: No

**3.** Go to xxx.xxx.xx.xxx:3000 on your web browser to be taken to the REST Server::

  xxx.xxx.xxx.xxx:3000

.. image:: Images/11.1.png

**4.** Then click on MedProvider

.. image:: Labs/Images/11.2.png

**5.** Select POST and click on the light brown box in the bottom right. That will place that code in the white box in the bottom left

.. image:: Images/11.3.png

**6.** Make appropriate changes that you see in the picture below

.. image:: Images/11.4.png

**7.** Click on Try it out! 

**8.** Scroll down and look at the response code. If you get Response Code: 200 that is very good. That means it was added as a Medical Provider

.. image:: Images/11.5.png

**9.** Let’s try adding a Member. Click on Member and then POST

**10.** Change the syntax to replicate what is in the picture below and then click on Try it out! Again, response code 200 is what we want

.. image:: Images/11.6.png

**11.** Scroll up to GET within the Member and click on Try it out!

**12.** Now, you receive all your members that have been created

.. image:: Images/11.7.png

**13.** Now go back to your Composer Playground and click on All Transactions

.. image:: Images/11.8.png

**14.** There you will see the addition of participants that you created from the REST Server. Click on View Record to see that transaction that is timestamped. It should be the same transaction that you did from the REST Server

**15.** Go ahead and add a few other participants and assets through the REST server. Stick to Participants and Assets. If you are confused on what the expected syntax is, go back into the Composer Playground and add a participant. Then go back into the REST server with the correct expected syntax.

**Bonus:** If you go to xxx.xxx.xxx.xxx:3000/explorer/swagger.json – You will be taken to the Swagger document for the REST Server. Remember, this is from the admin’s perspective. In the following sections, we will look at the other participants’ REST Server perspective. With the Swagger document, you are able to incorporate these APIs to a working presentation logic (app or web application)::

  xxx.xxx.xxx.xxx:3000/explorer/swagger.json


Part 12: Guardian Immunichain REST Server
=========================================

In this section you are going to open your business network to a REST server. You are going to act as the Guardian you created earlier in this lab.

**1.** To end your current REST server, hit Control and c at the same time. You can also close your current one and open another command line terminal::

  Control + c

**2.** Enter composer card list from your command line and you should see your cards with the Business Network of Immunichain::

  linux1@blockchain:~> composer card list

**3.** Now enter composer-rest-server and enter the same information as I have below. Make sure your card name matches what your composer card list output represented. We only want the guardian card::

  linux1@blockchain:~> composer-rest-server
  ? Enter the name of the business network card to use: austin@immunichain  ## or whatever you called your guardian card
  ? Specify if you want namespaces in the generated REST API: always use namespaces
  ? Specify if you want to enable authentication for the REST API using Passport: No
  ? Specify if you want to enable event publication over WebSockets: Yes
  ? Specify if you want to enable TLS security for the REST API: No

**4.** Now go to your REST Server::

  xxx.xxx.xxx.xxx:3000

**5.** Your screen should look like this

.. image:: Images/12.1.png

**6.** Run a GET on the Child (Childform). Click on GET, then Try it Out!

.. image:: Images/12.2.png

**7.** Scroll down to view the two children we have associated with our Guardian - Austin. 

.. image:: Images/12.3.png

**8.** Run a POST on a Guardian as well. Here is a sample of what I did

.. image:: Labs/Images/12.4.png

**9.** When you have the information you want, click on Try it Out

.. image:: Labs/Images/12.5.png

You would’ve received an error with a 500 response code. If you scroll to the right in the Response Body you will see that the guardian doesn’t have create access on guardians, meaning that a guardian can’t create a guardian

**10.** Continue to make various participants and assets and discover what you can or cannot do based on our ACL

**Bonus:** If you go to xxx.xxx.xxx.xxx:3000/explorer/swagger.json – You will be taken to the Swagger document for the REST Server. Remember, this is from the guardian’s perspective. In the following sections, we will look at the other participants’ REST Server perspective. With the Swagger document, you are able to incorporate these APIs to a working presentation logic (app or web application)::

  xxx.xxx.xxx.xxx:3000/explorer/swagger.json


Part 13: Member Immunichain REST Server 
=======================================

In this section you are going to open your business network to a REST server. You are going to act as the Member you created earlier in this lab.

**1.** To end your current REST server, hit Control and c at the same time. You can also close your current one and open another command line terminal::

  Control + c

**2.** Enter composer card list from your command line and you should see your cards with the Business Network of Immunichain::

  linux1@blockchain:~> composer card list

**3.** Now enter composer-rest-server and enter the same information as I have below. Make sure your card name matches what your composer card list output represented. We only want the Member card::

  linux1@blockchain:~> composer-rest-server
  ? Enter the name of the business network card to use: fairmont@immunichain  ## or whatever you called your member card
  ? Specify if you want namespaces in the generated REST API: always use namespaces
  ? Specify if you want to enable authentication for the REST API using Passport: No
  ? Specify if you want to enable event publication over WebSockets: Yes
  ? Specify if you want to enable TLS security for the REST API: No

**4.** Run a GET on the Child. Click on Try it Out

.. image:: Images/13.1.png

**5.** Below is what should appear in the response

.. image:: Images/13.2.png

If you notice, the member still receives all the children whether they are authorized or not. That would not work in a production environment. 

**6.** Jump back over to the Composer Playground from the admin perspective. Jump to the Define section and click on the Access Control on the left

.. image:: Images/13.3.png

**7.** Delete rule memberRead starting on line 159. Here is a before and after picture of what your screen should look like now

.. image:: Images/13.4.png

.. image:: Images/13.5.png

**8.** Now add in this rule on line 159. Click Deploy Changes in the bottom left whenever you’ve added this rule::

  rule MemberRead {
    	description: "Allow members to view children that have them as a member"
   	  participant(g): "ibm.wsc.immunichain.Member"
    	operation: UPDATE, READ
    	resource(c): "ibm.wsc.immunichain.Childform"
   	  condition: (c.members.some(function (member) {
      	return member.getIdentifier() == g.getIdentifier();
    	}))
    	action: ALLOW
  }

This rule states that the member can only read the child’s immunization record if they authorization to do so.

**9.** Now, submit a transaction to add our member #1 to one of our children

.. image:: Images/13.6.png

**10.** Now, go back to the REST Server. Run a GET on the Child and click on Try it Out

.. image:: Images/13.7.png

You now notice that we don’t receive the other child – Bobbie – in our response. That is because our Member #1 – High School – can only see the children’s immunization record in which they have authorization to.

**Bonus:** If you go to xxx.xxx.xxx.xxx:3000/explorer/swagger.json – You will be taken to the Swagger document for the REST Server. Remember, this is from the member’s perspective. In the following sections, we will look at the other participants’ REST Server perspective. With the Swagger document, you are able to incorporate these APIs to a working presentation logic (app or web application)::

  xxx.xxx.xxx.xxx:3000/explorer/swagger.json


Part 14: Medical Provider Immunichain REST Server
=================================================

In this section you are going to open your business network to a REST server. You are going to act as the Medical Provider you created earlier in this lab.

**1.** To end your current REST server, hit Control and c at the same time. You can also close your current one and open another command line terminal::

  Control + c

**2.** Enter composer card list from your command line and you should see your cards with the Business Network of Immunichain::

  linux1@blockchain:~> composer card list

**3.** Now enter composer-rest-server and enter the same information as I have below. Make sure your card name matches what your composer card list output represented. We only want the Medical Provider card::

  linux1@blockchain:~> composer-rest-server
  ? Enter the name of the business network card to use: healthquest@immunichain ## or whatever you called your Medical 	Provider card
  ? Specify if you want namespaces in the generated REST API: always use namespaces
  ? Specify if you want to enable authentication for the REST API using Passport: No
  ? Specify if you want to enable event publication over WebSockets: Yes
  ? Specify if you want to enable TLS security for the REST API: No

**4.** Run a GET on the Guardian. Click on Try it Out

.. image:: Images/14.1.png

**5.** Run a GET on the Member. Click on Try it Out

.. image:: Images/14.2.png

**6.** Run a GET on the Child (Childform). Click on Try it Out

.. image:: Images/14.3.png

**Bonus:** If you go to xxx.xxx.xxx.xxx:3000/explorer/swagger.json – You will be taken to the Swagger document for the REST Server. Remember, this is from the medical provider’s perspective. With the Swagger document, you are able to incorporate these APIs to a working presentation logic (app or web application)::

  xxx.xxx.xxx.xxx:3000/explorer/swagger.json


Part 15: Multi-User REST Server
===============================

**Pre-Requisites for this section:**

Github Account (go to www.github.com to create an account)

In this section, we will go over how to create and use a multi-user REST Server. What are the benefits of having a multi-user REST Server? For starters, in order to use the server in multi-user mode, it uses authentication. We will go over that very shortly. Another benefit of the multi-user REST server is the potential for an organization to use 2 or more of their peers. Well, why would an organization have 2 or more peers in the same network, shouldn’t 1 be enough? In a production environment, it is recommended for an organization to have at least 3 peers. One peer or more for participation in the network, another peer for development purposes and then an additional peer for backup. 

**1.** To end your current REST server, hit Control and c at the same time. You can also close your current one and open another command line terminal::

  Control + c

**2.** Enter this command, as we will use Github for our authentication::

  npm install -g passport-github

In order to configure the passport-github strategy, we will need to register an OAuth application on GitHub and retrieve the client ID and client secret

**3.** Go to your Github account

**4.** Click on your profile picture on the top right, and click on Settings from the drop down menu

**5.** Click on OAuth applications under Developer settings on the left hand bar

**6.** Click on Register a new application

**7.** Specify the following settings::
  
  Application name: Composer
  Homepage: http://xxx.xxx.xxx.xxx:3000/
  Application description: OAuth application for Composer
  Authorization callback URL: http://xxx.xxx.x.xxx:3000/auth/github/callback
  
**8.** Click on Register application

**9.** Leave open your Github account in order to view your Client ID and Secret 

**10.** Enter composer card list to list out all your cards in the Immunichain network::

  composer card list

**11.** Enter this command to enter the multi-user composer rest server with the admin card (again, replace admin with whatever you called your admin card). You can find out your admin card name by doing composer card list command::

  composer-rest-server -c admin@immunichain -m true

**12.** Navigate to your REST server on your browser

.. image:: Images/15.1.png

It looks exactly like your traditional REST server that we have seen before, but just like book – don’t judge this REST server by its cover

**13.** Run a GET on the Child. Click on Try it Out

.. image:: Images/15.2.png

**14** What kind of message did you receive? It should be a response code of 401 with the response body saying authorization is required

.. image:: Images/15.3.png

**15.** End your current REST server by hitting Control and c at the same time. You can also close your current one and open another command line terminal::

  Control + c

**16.** Open up a notepad on your computer and paste this into the note::

  export COMPOSER_PROVIDERS='{
 	 "github": {
    	"provider": "github",
    	"module": "passport-github",
    	"clientID": "REPLACE_WITH_CLIENT_ID",
    	"clientSecret": "REPLACE_WITH_CLIENT_SECRET",
    	"authPath": "/auth/github",
    	"callbackURL": "/auth/github/callback",
    	"successRedirect": "/",
    	"failureRedirect": "/"
  	}
  }'

Now, replace clientID with the clientID produced by the Github. Do the same for the clientSecret. Here is what my export command looks like

.. image:: Images/15.4.png

**17.** Copy that export command and paste it back into your command line. Press enter to execute the command.

**18.** Before going to the REST Server, restart the REST server, but repeating step 11. Once you enter the command below, navigate to xxx.xxx.xxx.xxx:3000/auth/github::

  composer-rest-server -c admin@immunichain -m true

**19.** Click on Sign In and then Authorize yourself

.. image:: Images/15.5.png

**20.** You will then be taken back to the REST server

**21.** Scroll down till you get to the Wallet section of the REST server. Do a GET on the /wallet

.. image:: Images/15.6.png

**22.** Click on Try it Out

.. image:: Images/15.7.png

That basically means the REST server is looking for a card for the server to act as

**22.** Jump back to Composer Playground and navigate to the home screen where all your ID cards are located. Go ahead and click on the download button on all your ID cards.

.. image:: Images/15.8.png

**23.** Go back to your REST server and scroll down to POST /wallet/import in the Wallet section

.. image:: Images/15.9.png

**24.** Click on Choose File and drop in your Admin Card

.. image:: Images/15.10.png

**25.** Once your Admin Card is in the server, click on Try it Out

.. image:: Images/15.11.png

Receiving a 200-level response code is totally fine

**26.** Scroll back up and do a GET /wallet within the Wallet section. Click on Try it Out

.. image:: Images/15.12.png

That means you are acting as the admin of the Immunichain Blockchain network. 

Go ahead and submit various transactions, GETs, or POSTs from within the REST server. While you’re in multi-user mode, act as various participants a well.

Once you act as the guardian, medical provider or member you can do various REST calls (GET, POST, PUT, DELETE) based on our ACLs back in the Composer Playground. 

**End of lab!**
