
# The Firebase PWA Kit

The Firebase PWA Kit is an all in one guide on how to build secure, valuable, fast web apps with Ionic, Angular and Firebase. This kit is a combination of steps that you can follow, code snippets and packages that you'll need to get it all done. 

This guide is ideal for web developers who are familiar with Ionic/Angular and want to get up and running with Firebase. My goal is to help you initialise a secure, progressive web app using Firebase. 



## What you will learn

- Initialising an Ionic App with Angular
- Adding PWA Elements
- AngularFire & Firebase
- User Authentication 
- Auth Route Guards 
- Firestore Rules
- Deploy PWA to Hosting

## What you will need

- A Computer & Internet
- Firebase Project
- VSCode or other text editor
- 15 - 30 Minutes of Time

## Sections

- Initialisation
- Firebase Connection
- Deployment
## Ionic Initialisation

Install Ionic CLI Globally

```bash
  npm i -g @ionic/cli
```

Create a new Ionic App with Angular. Select the blank template and select Angular as the Framework

```bash
  ionic start
```

![Output](https://firebasestorage.googleapis.com/v0/b/fb-pwa-kit.appspot.com/o/Ionic%20Start.PNG?alt=media&token=b5aa862b-64c6-4c9f-afd0-1544e18da68c)

Install dependencies

```bash
  npm i @angular/fire firebase --save
```

Navigate into the App Directory

```bash
  cd <app-name>
```

Start the server

```bash
  ionic serve
```

## Firebase Initialisation

Once you have a project setup go ahead and click the web icon to get your config. Alternatively you can find it by clicking the settings icon in the top left and selecting ptoject settings.

![Output](https://firebasestorage.googleapis.com/v0/b/fb-pwa-kit.appspot.com/o/Add%20Web%20Config.PNG?alt=media&token=abfad087-c856-4876-a874-c700224d2763)

![Output](https://firebasestorage.googleapis.com/v0/b/fb-pwa-kit.appspot.com/o/FB%20Config.PNG?alt=media&token=b1d8c586-ccaf-4a13-b3b5-ffb004dfc500)

Let's grab that config object and and bring it to both your environments file in the ionic app.

src/app/environments/environments.ts and environments.prod.ts
```bash
  firebaseConfig = {
    apiKey: <key>,
    authDomain: <authDomain>,
    projectId: <projectId>,
    storageBucket: <bucket>,
    messagingSenderId: <senderId>,
    appId: <appId>
  }
```
