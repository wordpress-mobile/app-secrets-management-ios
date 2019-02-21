# App Secrets Management for iOS

This repo contains files related to the way application secrets are managed in our iOS apps.

Application secrets are stored in an JSON document outside of an app's source control so they are not accidentially commited. The included files read the secrets from the JSON document and compose application code defining the secrets from templates.


## Usage

Follow these steps to configure application secret management in an iOS app.  It is assumed your project is organized with a top level workspace folder containing a project folder for the app.  If your app has a different structure you'll need to modify file paths accordingly.


### Create a credentials file for your app in the secrets repo

If the app uses our secrets reppository, 
1. Navigate to `~/.mobile-secrets/iOS/`
2. Create a new folder for your app and add a JSON file with your apps secrets.  
3. Commit the changes and push them to master.


### Copy files and folders

1. Copy the `Scripts/build-phases` folder from this repo into the root of your app’s workspace folder.
2. Copy the `Credentials` folder from this repo into the root of your app’s project folder.


### Edit files for your project needs

1. Open your Project in Xcode. 
2. Select `File > Add Files` and add the `Credentials` folder to your project.
3. Edit the following files to have the secrets your app needs:
```
ApiCredentials.tpl
InfoPlist.tpl 
Templates/ApiCredentials-Template.swift
Templates/InfoPlist-Template.h
```
4. Remove the reference to the `Credentials/Templates` folder from your project. Leaving it will cause build errors later.


### Configure project build settings and build phases

1. Open your project’s build settings.  
2. Add a user defined setting named `SECRETS_PATH` whose value is the path to your secrets file. Example: 
``` $HOME/.mobile-secrets/iOS/your-app/your-secrets.json```
3. Add a new *Cross-platform Aggregate* target named `GenerateCredentials`
4. Add a new *Run Script* build phase to the `GenerateCredentials` target.
5. Under `Shell` add `$SRCROOT/../Scripts/build-phases/generate-credentials.sh`
6. Under `Input Files` add the following:
```
$(SRCROOT)/Credentials/replace_secrets.rb
$(SRCROOT)/Credentials/ApiCredentials.tpl
$(SRCROOT)/Credentials/InfoPlist.tpl
~/.mobile-secrets/iOS/yourapp/yourapp_credentials.json (or whatever is the path to your secrets file)
$(SRCROOT)/Credentials/Templates/APICredentials-Template.swift
$(SRCROOT)/Credentials/Templates/InfoPlist-Template.h
```
7. Under `Output Files` add the following:
```
$(BUILT_PRODUCTS_DIR)/DerivedSources/ApiCredentials.swift
$(BUILT_PRODUCTS_DIR)/DerivedSources/InfoPlist.h
```
8. Open your application target’s build phases.
9. Select `Target Dependencies` and add the `GenerateCredentials` target.


### Test

1. Build and Run. Confirm your app works as expected.  

Bonus: Inspect your application's build directory. Find the DerivedSources folder and confirm it contains your credential files.



