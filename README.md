
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

## Adding PWA Elements

You now have a Ionic web app but let's turn into a PWA with one magic angular command:
```bash
    ng add @angular/pwa
``` 

It adds different png files for different splash images for various resolutions icon-128x128.png, icon-144x144.png, icon-152x152.png, icon-192x192.png, icon-384x384.png, icon-512x512.png. Additionally, it adds ngsw-config.json and manifest.webmanifest for configuration purposes. Also, it modifies angular.json, package.json, index.html and app.module.ts. So now when your app is served users will be prompted to install the PWA upon vising your app.
![Output](https://firebasestorage.googleapis.com/v0/b/fb-pwa-kit.appspot.com/o/add.PNG?alt=media&token=ebf0de99-f0e5-4461-99b7-0f5bd223d4fb)

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

Before we can move forward we will need a User Model. Let's create a file for it in our auth folder:

user.model.ts
```bash
    export interface User {
        uid: string;
        email: string;
        photoURL?: string;
        displayName?: string;
        returnedAt: any;
        roles?: Roles;
    }

    export interface Roles {
        visitor?: boolean;
        subscriber?: boolean;
        admin?: boolean;
    }
```

On to the auth service:

auth.service.ts
```bash
    import { Injectable } from '@angular/core';
    import { Router } from '@angular/router';
    import { User } from './user.model'; 

    import auth from 'firebase/compat/app';
    import { AngularFireAuth } from '@angular/fire/compat/auth';
    import { AngularFirestore, AngularFirestoreDocument } from '@angular/fire/compat/firestore';

    import { Observable, of } from 'rxjs';
    import { first, switchMap, take } from 'rxjs/operators';

    @Injectable({ providedIn: 'root' })
    export class AuthService {

        user$: Observable<User>;

        constructor(
        private afAuth: AngularFireAuth,
        private afs: AngularFirestore,
        private router: Router,
        ) { 
            // Get the auth state, then fetch the Firestore user document or return null
            this.user$ = this.afAuth.authState.pipe(
                switchMap(user => {
                    // Logged in
                if (user) {
                    return this.afs.doc<User>(`users/${user.uid}`).valueChanges();
                } else {
                    // Logged out
                    return of(null);
                }
                })
            )
        }

        getUser() {
            return this.user$.pipe(first()).toPromise();
        }

        async googleSignin() {
            const provider = new auth.auth.GoogleAuthProvider();
            const credential = await this.afAuth.signInWithPopup(provider);
            return this.updateUserData(credential.user);
        }
    
        private updateUserData(user) {

            // Sets user data to firestore on login
            const userRef: AngularFirestoreDocument<User> = this.afs.doc(`users/${user.uid}`);

            const data = { 
                uid: user.uid, 
                email: user.email, 
                displayName: user.displayName, 
                photoURL: user.photoURL,
                returnedAt: new Date(),
                roles: {
                    visitor: true,
                    subscriber: false,
                    admin: false
                }
            };

            return userRef.set(data, { merge: true });
        }
    
        async signOut() {
        this.afAuth.signOut().then(() => {
            this.router.navigate(['/auth']).then(() => {
            console.log('Redirect To Auth Page');
            }).catch(err => console.log(err));
        });
        }

    }

```

Let's add a Google Auth button to our html for testing:

auth.page.html
```bash
    <ion-button color="dark" expland="block" fill="clear" (click)="auth.googleSignin()">
        <ion-icon name="logo-google" slot="start"></ion-icon> Continue with Google
    </ion-button>
```

In order to use this auth.googleSignin method we will need to inject it into the page and make it public.

auth.page.ts
```bash
    constructor(
        public auth: AuthService
    ) { }
```

Last but not least let's configure an auth based route Guard. This route guard has just one job which is to only allow authenticated users to certain page routes.

auth.guard.ts
```bash
    import { Injectable } from '@angular/core';
    import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, Router } from '@angular/router';

    import { AuthService} from './auth.service'
    import { Observable } from 'rxjs';
    import { tap, map, take } from 'rxjs/operators';
    import { AlertController, ToastController } from '@ionic/angular';

    @Injectable({
        providedIn: 'root'
    })
    export class AuthGuard implements CanActivate {
        constructor(
            private auth: AuthService, 
            private router: Router, 
            private alertCtrl: AlertController,
            private toastCtrl: ToastController
        ) {}

        canActivate(next: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {

            return this.auth.user$.pipe(
                take(1),
                map(user => !!user), // <-- map to boolean
                tap(loggedIn => {
                    if (!loggedIn) {
                    this.toastCtrl.create({
                        message: 'Please login first.',
                        duration: 4000
                    }).then(toastEl => toastEl.present()).catch(err => console.log(err));
                    console.log('access denied');
                    this.router.navigate(['/auth']);
                    }
                })
            )
        }
    }
```

So now we have our guard. At this point you can add it to your routing page module on the routed that you want to protect:

app-routing.module.ts
```bash
    import { AuthGuard } from './pages/auth/auth.guard';

    const routes: Routes = [
        {
            path: '',
            loadChildren: () => import('./pages/auth/auth.module').then(m => m.AuthPageModule)
        },
        {
            path: 'protected-route',
            loadChildren: () => import('./pages/protected-page/protected-page.module').then( m => m.ProtectedPageModule),
            canActivate: [AuthGuard]
        }
    ]
```

Congrats, you have implemented Firebase auth and managed to get your app working with it. But the work is not yet done. We only need to secure our Firebase database by modifying our Firestore rules.

I have gone ahead and shared some helper functions with you to secure your Firestore collection paths using auth or user roles.

Firestore Rules:
```bash
    // Reusable function to determine document ownership
    function isOwner(userId) {
        return request.auth.uid == userId
    }
    
    // Reusable function to get users role
    function getRole(role) {
      return get(/databases/$(database)/documents/users/$(request.auth.uid)).data.roles[role]
    }

    // Allow create, write, read and update on our users collection
    match /users/{userId} {
        allow create, write, read, update;
    }
```

## Deployment to Firebase Hosting

Congrats on making it this far. The last step for us is to build our PWA and deploy it to Firebase Hosting. For this please ensure that you have hosting enables on the Firebase console.

Let's start by installing the firebase-tools with NPM:
```bash
    npm i firebase-tools
```

Next we want to initialize the Firebase CLI in our project directory:
```bash
    firebase init
``` 

In the step above you will need to confirm that you are trying to init firebase, then you will need to select hosting in the options menu. When you promted to setup as a SPA say yes. Lastly, make sure that you link it to your www folder where we will place the build app.

Now lets build the production app shall we:
```bash
    ionic build --prod
```

Once this has completed you should now see the www folder in your project directory.

Now that we have out built production app lets deploy it then:
```bash
    firebase deploy --only hosting
```

The above commant will deploy your application to firebase hosting! You should see the testing link in the response, alternatively have a look at the hosting tab in the Firebase console for more info.


