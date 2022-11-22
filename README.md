
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

Once you have your firebase config object added to your environments files you almost ready to go. Up next we need to configure the AngularFire pacakage in necessary modules to interact with firebase from our Ionic/Angular app.

Let's jump to our app module at src/app/app.module.ts and add the following imports:
```bash
    // First we bring in our environments file
    import { environment } from '../environments/environment';

    // Then we bring in the AngularFireModule which we'll need shortly
    import { AngularFireModule } from '@angular/fire/compat';

    // We're also going to bring in the Firestore Database module to interact with Firestore
    import { AngularFirestoreModule } from '@angular/fire/compat/firestore';
```

Now let's bring those imports into the imports array in your app module.
```bash
  imports: [
    ...
    // initialize the firebase app
    AngularFireModule.initializeApp(environment.firebaseConfig),

    // initialize the Firestore module with offline persistence
    AngularFirestoreModule.enablePersistence()
  ]
```

Firestore Usage in page component (eg: app.component.ts):
```bash
  // Import the AngularFirestore package
  import { AngularFirestore } from '@angular/fire/compat/firestore';

  // Use the package via dep injection
  constructor(
    private db: AngularFirestore
  ) {}

  // Read a collection
  readCollection() {
    this.db.collection('my-collection')
      // Listen to changes to collection
      .valueChanges()
      // Retrieve data as observable
      .subscribe(data => console.log(data));
  }

  // Add data to collection
  addData() {
    // Document to add to collection
    const myData = {
      name: 'John',
      surname: 'Doe'
    };

    this.db.collection('my-collection')
      .add(myData)
      // Added Successfully
      .then(docRef => console.log(docRef))
      // Uh Oh. 
      .catch(err => console.log(err));
  }
```

Congrats. Now you have your app connected to firebase and you can start using it. To see if your documents have successfully been stored check the Firebase console.

## Firebase Authentication

So now you have Firebase installed and configured. At this point in time I would recomment that you jump right into setting up Authentication. There are many ways to achieve this however we will make use of pages, services and guards to achieve our desired auth flow.
 
Luckily for us we can employ the Ionic CLI. Please run the following commands to generate the relevant pages, services and page guards.

```bash
  // This will create an auth page directory in your src and add the relevant auth page files
  ionic g page auth/auth 

  // This will generate a auth guard inside the auth directory. auth.guard.ts 
  ionic g guard auth/auth

  // Up next we'll generate the service auth.service.ts inside our auth directory
  ionic g service auth/auth
```

With that all done, we have the necessary files required to move onto the next step. Before we do let me explain the purpose of these files. The first is our auth page, this will house our UI and Component code, the second is a page guard that we will make use of to only allow authenticated users to a specific page route and lastly we have our auth service which is where we will house all the important auth code to be injected into other components later. 




