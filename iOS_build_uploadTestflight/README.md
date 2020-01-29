# Continuous Delivery with Jenkins + Fastlane + Testflight

![N|Solid](https://www.geckoboard.com/assets/jenkins.png) ![N|Solid](https://fastlane.tools/assets/images/fastlane-logo-lockup.png)
![N|Solid](https://developer.apple.com/assets/elements/icons/testflight/testflight-64x64_2x.png)

### Summary
The purpose of this tutorial is to show how to setup continuous delivery using Jenkins and Fastlane to automate iOS build with testflight upload. It also serve as a record keeping for myself in case I need to refer back. Once your setup is successful, you will be able to:
  - Automate or schedule iOS build
  - Automate upload process to testflight for tester
  - Receive email notification on build with logs attached
  
### Requirements
1. Xcode
2. Homebrew
3. Jenkins
4. iTunes Connect Account

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
- Now we need to install and setup fastlane before we can create a job to execute fastlane commands.

#### 3. Fastlane
##### 3.1 Installing Fastlane
- Open terminal and enter below command to install fastlane
```
sudo gem install fastlane -NV
```
- Once the installation is done, cd to your iOS project and type below command and follow the instructions prompted
```
fastlane init
```
- This will create a fastfile, a appfile and a gemfile on the root of your project. 
##### 3.2 Setup fastlane
- Now open appfile using textedit and paste below code on it
```
app_identifier "[your_app_identifier]"
apple_id "[your_developer_apple_id]"
itc_team_name "[your_itunes_connect_team_name]"

for_platform :ios do
  for_lane :beta do
    app_identifier '[your_app_identifier]'
  end
end
```
- Then open fastfile using textedit and paste below code on it
```
ENV["FASTLANE_XCODEBUILD_SETTINGS_TIMEOUT"] = "60"
ENV["FASTLANE_XCODE_LIST_TIMEOUT"] = "60"
ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "[your_apple_app_specific_password]"

default_platform(:ios)

platform :ios do
  desc "Uploads build to TestFlight"
  lane :beta do
    xcversion(version: "10.2.1")
    build_app(scheme: "[scheme_to_build]]",
            workspace: "[workspace_of_your_project]",
            include_bitcode: false, 
	    export_method: "app-store",
	    export_xcargs: "-allowProvisioningUpdates",
            export_options: {
            	provisioningProfiles: { 
        		    "[your_app_identifier]" => "[UUID_of_provisioning_profile]"
      		    }
    	    })
    upload_to_testflight
  end
end
```
- For the apple app specific password, you need to login to your apple account and generate the password from there.
- Now open gemfile using textedit and paste below code on it
```
if RUBY_VERSION =~ /2.3/
  Encoding.default_external = Encoding::UTF_8
  Encoding.default_internal = Encoding::UTF_8
end

source "https://rubygems.org"

gem "xcode-install"
gem "fastlane"
```
- Open terminal and run below code
```
sudo bundle update
```
- Once completed, it will generate a Gemfile.lock, commit the following files to your version control
1. Gemfile
2. Gemfile.lock
3. Fastlane folder (Inside includes Fastfile and Appfile)
- That's all to setup fastlane, now head back to your jenkins homepage to create your automation job.

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

#### 5. Add build step
- Go to Build section and click on the "Add build step", a list of option will popup, select "Execute Shell".
- Add below code to it as your first shell execution
```
bundle install
```
- Then add another "Execute Shell" build step and enter below code
```
bundle exec fastlane beta
```

#### 6. Email Notification
##### 6.1 Setup email account in Jenkins
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
Upload testflight process - $BUILD_STATUS. 
<br/>
Please check attached build log for more info.
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
##### Click on Build Now and enjoy your coffee while waiting for Jenkins to send you email after build and uploaded to testflight.


### Todos
 - Add in steps and screenshots for Git repo

License
----

MIT

