<p align="center">
  <img src="images/giganodes-logo.png" alt="Giganodes Logo">
</p>

**Part 2**

**Instructions for the Launchnodes Validator Service**

**(This Document 1 Mainnet Configuration for Validator Nodes)**

**[<ins>Register</ins>](https://www.launchnodes.com/register-for-enhanced-support/) to get free enhanced support that includes**

* Early notification of planned software client updates
* Email/online conferencing support for setting up Beacon and Validator nodes
* Email and online conferencing support for existing Launchnodes products

Before you run the Validator Service, you need to successfully have set up and run the Validator_gateway. If you have not set up and run the Validator_gateway, please follow this [<ins>documentation</ins>](https://docs.google.com/document/d/108jGhP3rv3-iOSj5VhpCVYIg2wqPXdtLnFEp0QBElTg/edit?usp=sharing) to do so and come back to the Validator Service once that is done.

The validator_gateway service opens a gateway for your validator node to connect with a beacon node. To do this we are going to create and run a task definition of the validator. If you are connecting to your own beacon node then make sure your beacon node security group has prior IP permissions for connecting a validator.

**Note: There is also a video guide available for instructions which is linked into below steps. Avoid the usage of static data from video.**

Before creating the Validator task we need to create a wallet and import keys to your own instance. To create a wallet and import your created validator key into it follow the 10 steps below.

**Wallet Creation And Importing Keys**

1. Login in to your Instance.
2. Run below commands to create a wallet. As we don’t have a wallet in a fresh instance we need to create one and import keys into it. (cut and paste the code below).

```
docker run -it -v $HOME/Eth2Validators/prysm-wallet-v2:/wallet --network="host" \
gcr.io/prysmaticlabs/prysm/validator:v1.3.11 \
wallet create \
--wallet-dir=/wallet
```

3. Now, Accept the terms and conditions of usage. To do that write, accept and click the enter button.
4. After that select the Imported wallet option.
5. Now, set your wallet Password.
6. Your wallet will now be created.
7. After the creation of your wallet we need to import the validator key which we created before and also copied from your local device to your AWS Instance.
8. Run the Below command to import your keys into your wallet.

```
docker run -it -v $HOME/eth2.0-deposit-cli/validator_keys:/keys \
  -v $HOME/Eth2Validators/prysm-wallet-v2:/wallet \
  -v $HOME/Eth2:/validatorDB \
  --name validator \
  gcr.io/prysmaticlabs/prysm/validator:v1.3.11 \
  --datadir=/validatorDB accounts import --keys-dir=/keys --wallet-dir=/wallet
```

It's important to note here that “$HOME/eth2.0-deposit-cli/validator_keys” is your folder path where the validator keys are stored.

9. It will now ask for your wallet password, enter your password.
10. Now, It’ll ask the password for keys that you have to set at the time of key creation and It will import your keys into the wallet.

**Instructions for creating the Launchnodes Validator Task**

Create Validator Task definition from the launchnodes’s JSON and to create the task definition follow below sets of instructions:

1. Go to task definitions from within the ECS dashboard.
2. Select create a new task definition and then select EC2 then click on next.
3. Select Configure via JSON and clear the given text and paste the [<ins>validator related JSON file</ins>](https://validatortask.s3.us-east-2.amazonaws.com/validator.json) (provided by Launchnodes) data and then click on save.
4. Now, keep the screen as it is don’t click on create.
5. Now, you need to login to the server instance via SSH to do that follow these instructions.
    <ol type="a">
        <li>Go to EC2 Dashboard then Click on instances.</li>
        <li>Select your desired instance and click on connect.</li>
        <li>You can see many options to connect with your instance choose SSH Client.</li>
        <li>After that follow the instructions given on your screen to connect with instance via SSH.</li>
    </ol>
6. After successfully logging into the instance create a folder at your desired path and give any name to it. (command to create a folder is mkdir FolderName)
7. Now in that folder create a text file with the following command.
    <ol type="a">
        <li>cd FolderName</li>
        <li>nano pass.txt</li>
        <li>If nano command is not installed on your linux instance then install it using “sudo yum install nano”.</li>
        <li>Now writedown your wallet password that you used at the wallet creation time.</li>
        <li>Press ctrl+x</li>
        <li>Press y.</li>
        <li>Press enter.</li>
        <li>It saves your password into the txt file.</li>
    </ol>
8. Now, mount your password volume to the container and to do that follow below instructions.
    <ol type="a">
        <li>Get back to the ECS task definition screen and now find the field Volumes.</li>
        <li>Click on add volume.</li>
        <li>Give the name as “walletps”.</li>
        <li>Select Bind Mount as volume type.</li>
        <li>Now you have to paste the source path of your folder which contains the wallet password text file that we created in step 7.(For example path will start as /home/ec2-user/YourFolder)</li>
        <li>After that Go to Container Definition and click on validator_container.</li>
        <li>Go to the field Storage and Logging.</li>
        <li>Click on Add mount point.</li>
        <li>Click on the dropbox of source volume and you need to select walletps (which is the volume that we just binded).</li>
        <li>Write “/walletpth” on the container path field.</li>
        <li>Now click on update.</li>
    </ol>
9. If you are willing to connect with launchnodes’s beacon node then keep the settings as it is. But if you want to connect with your own beacon node then follow below steps.
    <ol type="a">
        <li>Go to container definition and click on validator_container.</li>
        <li>Go to the Environment field and change the IP of flag value so its like this --beacon-rpc-provider=YOUR_BEACON_INSTANCE_IP_HERE:4000</li>
        <li>Note: If you are connecting your beacon node then make sure you whitelist your validator instance ip intp your beacon node’s security group.</li>
        <li>After that click on update.</li>
    </ol>
10. Go to container definitions and select validator_container and go to Storage and Logging.
11. Select Auto-configure CloudWatch Logs which is recommended to check for checking logging details and then click on update.
12. Click on create.

To see the reference video guide to create validator task click [<ins>here</ins>](https://drive.google.com/file/d/1562iHQkMHMxTw4aXE5X8Pd0Zg3duaS-L/view?usp=sharing).

**Running the Validator Service**

1. Click on your created cluster.
2. Select the service tab then click on create.
3. Select launch type ec2.
4. Choose validator related task definition(which we have configured via our JSON file earlier) from Task definition Family.
5. Choose a name for the service. For example, validator_service.
6. Go to the number of task field and type 1.
7. Go to Minimum healthy percent and make it 0.
8. Go to Maximum percent and make it 100.
9. Click next then again next then create.

To see our video guide for running a validator service click [<ins>here</ins>](https://drive.google.com/file/d/1jic3NQRPiyafG3bGXUxFfCjI-fEkKKBm/view?usp=sharing).

You now have a Validator node connected to a Beacon Node (Launchnodes or Your own) that is staking Ethereum on Ethereum 2.0. Congratulations and well done

**Updation of Validator Services**

**Prerequisites:**

1. **You are already running an old version of validator services.**

Follow the below steps to update your validator services.

1. Click on marketplace [<ins>validator product link</ins>](https://aws.amazon.com/marketplace/pp/B08LDRGQ8C).
2. Click on “Continue to subscribe”.
3. Now, click on “continue to configure”.
4. Select “Validator Container” from the Delivery method drop down menu.
5. Click on “Continue to Launch”.
6. Click on “View Container Image Details” from Container Images division.
7. Now, you will be able to see the docker image url on your screen at “**Step 2: Pull all docker images listed below.**” Copy that URL.
8. Now, open <ins>https://aws.amazon.com/console/</ins> and login into it.
9. Search and click on “Elastic Container Service” from find services bar.
10. Click on “Task Definitions” from side menu.
11. Click on “aws_validator_task”
12. Click on the latest validator task definition that you created and select “Create new revision”
13. Select “Configure via JSON” button at the bottom of the page.
14. Now, find out “image” and after that you can see that the value of that “image” will be like “117940112483.dkr.ecr.us-east-1.amazonaws.com/xxxxxxxxx/xxxx/xxxx”. You need to change that value with the URL you got on step 7.
15. Click on “save” button.
16. Click on “create” button at the bottom of the page.

**Updating the Services of Validator task**

1. Click on “Clusters” from the left side menu at your ECS page.
2. Click on the Cluster you have created for the validator node.
3. Select the Service Which was running the old task definition for aws_validator_task.
4. Click on “Update” button.
5. Select the drop-down menu of revision and select the latest revision that we just created you can see the number(Latest) written on the menu. For example, If I have created the 4th revision of task then I will be seeing the 4(latest) written on drop-down menu.
6. Now, Tick the “Force new deployment” box.
7. Click on the next step at the bottom of the page.
8. Again Click on “Next Step”
9. Again Click on “Next Step”
10. Click on the “Update Service” button.

<p align="right">
  <img height="50px" src="images/launchnodes-logo.png" alt="Launchnodes Logo">
</p>
