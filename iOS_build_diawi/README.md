# Continuous Delivery with Jenkins + Diawi

![N|Solid](https://www.geckoboard.com/assets/jenkins.png) ![N|Solid](https://images.prismic.io/ionicframeworkcom/5e0c47b5b609bf05937ff94df5cde2b942332727_diawi.png?auto=compress,format)

### Summary
The purpose of this tutorial is to show how to setup continuous delivery using Jenkins and Diawi to accelerate iOS development and also as a record keeping for myself in case I need to refer back. Once your setup is successful, you will be able to:
  - Automate or schedule iOS build
  - Automate upload process to diawi for tester
  - Receive email notification on build with logs attached
  
### Requirements
1. Xcode
2. Homebrew
3. Jenkins
4. Diawi account

### Installation
Before we can install Jenkins, we need to install homebrew. Skip this step if you already have homebrew installed.

##### 1. Homebrew
- Skip this step if you already have homebrew installed.
- If you don't have yet, open your terminal and enter the following. 
```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

##### 2. Install Jenkins
- Now you have your homebrew installed, it's time to use it to install Jenkins.
- Enter the folowing into your terminal. This will install the latest Long-term Support(LTS) version of Jenkins into your mac.
```sh
brew install jenkins-lts
```
- Once installation is done, you may now start Jenkins server by entering below to your terminal
```sh
brew services start jenkins-lts
```
- Other than that, you can also restart the server or upgrade jenkins to the latest version as well.
```sh
#Restart jenkins server
brew services restart jenkins-lts
```
```sh
#Upgrade jenkins
brew upgrade jenkins-lts
```

- Once your jenkins server is started, open browser and proceed to http://localhost:8080 to perform initial setup to complete the Jenkins installation. By right you should see a page as below
![](https://www.macminivault.com/wp-content/uploads/unlock_jenkins.png)

- Enter below to your terminal to get the initial password and paste it there
```sh
cat /Users/administrator/.jenkins/secrets/initialAdminPassword
```

- After that, you should be prompted with two option as below, Choose (Install suggesrted plugins)
![](https://www.macminivault.com/wp-content/uploads/customize_jenkins.png)

- jenkins will now download and install plugins
![](https://www.macminivault.com/wp-content/uploads/jenkins_getting_started.png)

- Once initial plugins are installed, you will be prompted to create your first admin user.
![](https://www.macminivault.com/wp-content/uploads/jenkins_create_admin.png)

- After creating your first user, it will prompt you to enter your jenkins URL which will be used to access jenkins from browser. By default it will be http://localhost:8080. If you're accessing jenkins remotely, enter the URL this way (http://jenkins.[YOUR-DOMAIN].com:8080).
![](https://www.macminivault.com/wp-content/uploads/jenkins_url.png)

## Finally, Jenkins is installed and you should see this homepage
![](https://www.macminivault.com/wp-content/uploads/jenkins_screenshot.png)

#### 3. Install Additional Plugins
- Go to Manage Jenkins > Manage Plugins
- Inthe plugin manager, switch to Available tab and search for the plugin to install.
- The following is the plugins needed for this tutorial
1. Xcode integration (For setting project workspace, schemes, etc when build)
2. Environment Injector Plugin (To set which Xcode version to use when building)
3. Diawi Upload Plugin (To automate upload ipa to diawi)
- Once the above plugins are installed, restart Jenkins and head back to the homepage.

#### 4. Creating a Jenkins job
##### 4.1 Add new item
- To start create a new job. Click on (New Item).
![](https://www.macminivault.com/wp-content/uploads/jenkins_screenshot.png)
- Enter your project or job name in the item name field and select job as "Freestyle Project" and click "Ok".
![](https://i.ibb.co/RSCQxhh/Screenshot-2020-01-22-at-4-00-10-PM.png)
##### 4.2 Configure source code management
- Go to Source Code Management section, check subversion and enter the URL to your iOS project repository which you want this job to build.
- Select the credentials to login to your repository, you may add one if you don't have yet.
![](https://i.ibb.co/VqMSJDN/Screenshot-2020-01-23-at-10-53-21-AM.png)
##### 4.3 Configure build environment
- Go to Build Environment section and check the "Add timestamps to the Console Output"
- Check the "Inject environment variables to the build process" (Environment Injector Plugin) and add below path
```sh
DEVELOPER_DIR=/Applications/Xcode_10.2.1.app/Contents/Developer
```
- This is to preset Jenkins to build your Xcode project with a specific Xcode version, you can set any version you want. But if you just want Jenkins to build your project with the latest Xcode installed in your Mac, you may skip this step.
![](https://i.ibb.co/Rc4bBcr/Screenshot-2020-01-23-at-11-23-46-AM.png)
##### 4.4 Add build step
- Go to Build section and click on the "Add build step", a list of option will popup, select "Xcode".
- Then in Xcode build step, under General build settings, set any build settings for your project but must supply the following
1. Configuration set to "Release"
2. Xcode schema file (Put the scheme you want to build)
3. Export Method (Put either ad-hoc or enterprise)
4. .ipa filename pattern
![](https://i.ibb.co/b3y71SB/Screenshot-2020-01-23-at-11-46-22-AM.png)
- Under Code signing & OS X keychain options section, select Automatic Signing and select Unlock keychain and supply keychain password.
![](https://i.ibb.co/HnjtCQF/Screenshot-2020-01-23-at-2-26-28-PM.png)
- Then under Advanced Xcode build options section, enter your workspace file name into Xcode Workspace File if you are using workspace to build project. Leave empty if you're not using workspace.
![](https://i.ibb.co/V3TPb9V/Screenshot-2020-01-23-at-2-28-59-PM.png)
- Now we're done with Xcode build step, now add another build step and select "Diawi Upload Step".
![](https://i.ibb.co/vcJgsDg/Screenshot-2020-01-23-at-2-32-12-PM.png)
- Token and ipa file name must be supplied. You can obtain a token from your diawi account, simply generate a API token and paste here. For the ipa file name, you must put the same name which you set on Xcode build step with below format
```sh
build/Release-iphoneos/your_app_name.ipa
```
- That's all needed for diawi to automate upload. Next we need to configure this job to send notification email when the build is complete and uploaded to diawi with the link.
#### 5. Email Notification
##### 5.1 Setup email account in Jenkins
- Before you can send emails from jenkins, you need to add a email account with it's SMTP configuration.
- Go to Homepage > Manage jenkins > Configure system and head to Extended E-mail Notification section
- Click on Advanced and more options will show up. Then enter your email settings with username and password so jenkins can use this account to send out email.
![](https://i.ibb.co/DwdD6gG/Screenshot-2020-01-23-at-2-52-06-PM.png)
- Once done, click save and head back to the job configuration page.
##### 5.2 Add Post Build Action
- Click on add post build action and select "Editable Email Notification".
- Then add in emails (If more than one, separate by comma) into the Project Recipient List
- Then add below code into "Default Content"
```sh
Hi,
<br/>
<br/>
$PROJECT_NAME package- Build # $BUILD_NUMBER - $BUILD_STATUS.
<br/>
<br/>
Proceed to this ${FILE,path="build/Release-iphoneos/your_app_name.ipa.diawilink"} to download .ipa
<br/>
<br/>
${CHANGES_SINCE_LAST_SUCCESS, reverse=true, format="<b>Changes for Build #%n</b><br/>%c<br/>", changesFormat="<br/>Date- %d<br/>Author - %a<br/>SVN Revision - %r<br/>Path Changes - %p<br/>Commit Message - %m <br/>"}

<br/>
<br/>
<br/>
<br/>
Thanks
```
- Select "Compress and attach build log" for the option Attach Build Log
- Now your settings should look as below
![](https://i.ibb.co/tK8fsy4/Screenshot-2020-01-23-at-2-57-27-PM.png)
- That's all to create and configure your job item. Click save.
![](https://i.ibb.co/kGypcH3/Screenshot-2020-01-24-at-10-30-37-AM.png)
##### Click on Build Now and enjoy your coffee while waiting for Jenkins to send you email after build with the diawi link.


### Todos
 - Add in steps and screenshots for Git repo

License
----

MIT

