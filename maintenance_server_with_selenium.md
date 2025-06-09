**Procédure maintenance serveur with selenium**
----


## Tutorial

During the installation, check the google chrome version installed
```
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

Once restarted, reconnect to the server

Check the google-chrome installation
```
google-chrome --version
```

Go to this page [https://chromedriver.chromium.org/downloads](https://chromedriver.chromium.org/downloads) and copy paste
the chromedriver link for your google-chrome version (for example
https://chromedriver.storage.googleapis.com/95.0.4638.54/chromedriver_linux64.zip)

Go to root folter and run
```
wget LINK
unzip chromedriver-linux64.zip
sudo mv chromedriver-linux64/chromedriver /usr/bin/chromedriver
sudo chown root:root /usr/bin/chromedriver
sudo chmod +x /usr/bin/chromedriver
rm -r chromedriver-linux64
rm chromedriver-linux64.zip
```


### For firefox
Go to this page https://github.com/mozilla/geckodriver/releases and copy paste the last release for linux 64
geckodriver-v0.36.0-linux64.tar.gz

Go to root folter and run
```
wget LINK
tar -xvzf geckodriver-
sudo mv geckodriver /usr/bin/geckodriver
sudo chown root:root /usr/bin/geckodriver
sudo chmod +x /usr/bin/geckodriver
rm -r geckodriver*
```


Test your selenium --> It should work !

Send a mail to the client 
