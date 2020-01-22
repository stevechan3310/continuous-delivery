# Continuous Delivery with Jenkins + Diawi

![N|Solid](https://www.geckoboard.com/assets/jenkins.png) ![N|Solid](https://images.prismic.io/ionicframeworkcom/5e0c47b5b609bf05937ff94df5cde2b942332727_diawi.png?auto=compress,format)

### Summary
The purpose of this tutorial is to show how to setup continuous delivery using Jenkins and Diawi to accelerate iOS development. Once your setup is successful, you will be able to:
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

## Finally, Jenkins is installed and ready
![](https://www.macminivault.com/wp-content/uploads/jenkins_setup_complete.png)



### Todos
 - Add in steps and screenshots for Git repo

License
----

MIT

