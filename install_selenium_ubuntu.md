# Selenium installation on Ubuntu 18.04
_Install and use selenium on Ubuntu 18.04

[Link 1](https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/)

## Installation
```
sudo apt-get install -y unzip xvfb libxi6 libgconf-2-4
```

## Download and install chrome
`sudo curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add`
`sudo nano /etc/apt/sources.list.d/google-chrome.list`
Add `/etc/apt/sources.list.d/google-chrome.list` to the file
`sudo apt-get -y update`
`sudo apt-get -y install google-chrome-stable`

## Download and install chromedriver
```
wget https://chromedriver.storage.googleapis.com/89.0.4389.23/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
sudo mv chromedriver /usr/bin/chromedriver
sudo chown root:root /usr/bin/chromedriver
sudo chmod +x /usr/bin/chromedriver
```