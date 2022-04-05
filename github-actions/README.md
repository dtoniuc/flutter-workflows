# Flutter Workflows - Github Actions

## Sanity checks

Run code format, flutter analyze and tests for branches or PRs.

Setting up this workflow is streight forward. Copy the following files into your projects `.github/workflows` folder.

- [Verify](verify.yml) - Triggers the sanity_check workflow when PRs/commits are created for various branches.
- [Sanity Check](sanity_check.yml) - Format, analyze and run tests in a Flutter project. 

## Alpha Deployment - Firebase App Distribution

The [alpha deployment workflow](alpha_deployment.yml) makes extenisve use of reusable workflows. The workflow's diagram can be seen below.

<img width="1128" alt="Screenshot 2022-04-05 at 17 40 59" src="https://user-images.githubusercontent.com/31992665/161787390-8fc03350-11eb-4027-8a68-93f4084d6fc6.png">

The workflow runs a sanity_check as a prerequisite step, builds the apps and then releases them using firebase app distribution. 

For this workflow to run, several Github Secrets need to be defined. The way to define each of these is presented in the steps below. The complete list of secretes is: 

##### -- android build
- `FIREBASE_SERVICES_ANDROID_PASSPHRASE` - encryption password for `google-services.json` file

##### -- ios build
- `APPSTORE_CERT_BASE64` - base64 encoded app code signing certificate
- `APPSTORE_CERT_PASSPHRASE` - app code signing certificate password
- `IOS_MOBILEPROVISION_BASE64` - provisioning file
- `RUNNER_KEYCHAIN_PASSWORD` - randomg string to be used as runner's keychain password

##### - firebase app distribution
- `FIREBASE_TOKEN` - Firebase access token used by the actions to create Firebase app releases
- `FIREBASE_ANDROID_APP_ID` 
- `FIREBASE_IOS_APP_ID` 

### Build Android App

In order to deploy android app to Firebase App Distribution we need the `google-services.json`. However, to mitigate potential security issues, it is a common practice to not commit this file directly to the public repository. The approach taken here to avoid this problem is to archive the file into a `.tar` archive and encrypt it using [gnupg](https://formulae.brew.sh/formula/gnupg). The resulted, encrypted file, is commited into the repository. 

The password used to encrypt the file needs to be added to `FIREBASE_SERVICES_ANDROID_PASSPHRASE` Github secret. 

The [build android workflow](build_android.yml) will decrypt and restore the `google-services.json` file, build the APK and upload it as an artefact to Github. 

### Build IOS App

In order to build an ios app we need to have access to the signing certificate and profiles. [Github Docs](https://docs.github.com/en/actions/deployment/deploying-xcode-applications/installing-an-apple-certificate-on-macos-runners-for-xcode-development) provide details on how to add this data to a github runner. 

Set the following secrets as per the guide above. 

- `APPSTORE_CERT_BASE64`
- `APPSTORE_CERT_PASSPHRASE` 
- `IOS_MOBILEPROVISION_BASE64`
- `RUNNER_KEYCHAIN_PASSWORD` 

##### Firebase app distribution

In order to allow Github Actions to add releases to Firebase App Distribution we need to [create a firebase token](https://firebase.google.com/docs/cli#cli-ci-systems) and store it in the `FIREBASE_TOKEN` secret.

To deploy an IOS and an Android you need to create the associated apps in the Firebase App Distribution console and store the App Ids in the following secrets:

- `FIREBASE_ANDROID_APP_ID` 
- `FIREBASE_IOS_APP_ID` 

When you setup the Android app you will be give the `google-services.json` file we've mentioned above. Follow the instructions and save it in the `android/app` folder
