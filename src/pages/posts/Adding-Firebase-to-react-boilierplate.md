---
title: Adding Firebase Auth, Data Binding to a react-boilerplate Application
date: '2019-11-04T08:34:59.186Z'
thumb_img_path: images/Adding-Firebase-Auth,-Data-Binding-to-a-react-boilerplate-Application/header.png
excerpt: >-
  6 easy steps to add authentication and include react-redux-firebase in your react-boilerplate application.
template: post
---

Are you starting off a react project? You must probably be researching which boilerplate to use?

While there are some articles which talk about if or not to use a boilerplate, I dont think this is something worth spending any time on. A boilerplate is a must, but there still is some discussion possible on which one to use.

For example this article lists the 11 React Boilerplates and Starter Kits for 2019 that you could use. Feel free to go over this or any of the many others and take time to decide which one would fit your needs.

Or just agree that this is not the biggest decision to be made and move on - I chose to go with react-boilerplate for my project.

Honestly, when I downloaded the boilerplate, while it was trivial to get it working it was not easy to get an understanding of why or how something has been included.

Things improved over time and I think I have gained a reasonable level of understanding of why an how things have been done in the boilerplate. Three simple commands to get the boilerplate installed and working.

```
git clone https://github.com/react-boilerplate/react-boilerplate.git
npm run setup
npm run start
```

While there a number of things already in place with the boilerplate, am assuming you would want to incorporate two elements in your application as well.

- Login / Authentication
- Data Binding with your backend database and front end data store.

There are a number of options to carry out each of them, but in my experience I felt that using Firebase / Google ecosystem to achieve both were helpful. I used the following packages to build this functionalities into my application.

- Data binding : react-redux-firebase
- Firebase Auth UI : react-firebaseui/StyledFirebaseAuth

Clearly getting users onboarded on our application is a detailed workflow. It can be as simple as allowing a user to login or have a more complete workflow including mandatory onboarding forms, email verification etc. I will aim to cover that in a subsequent post.

Suffice to know that we need a Auth component which will allow user to login and results of that login are stored in a local cookie for use across the application.

### Step 1 - Package to link Firebase/Firestore to your React App

While there are probably again a number of packages to support this, I used [react-redux-firebase](https://www.reactreduxfirebase.com) in my project and am will detail how we can get this inclulded in the `react-boilerplate` application that you just created.

```
cd react-boilerplate
npm install react-redux-boilerplate --save
```

### Step 2 - Get Firebase Project Config Details

Am assuming you already have a Firebase project. If not it is again quite simple to set it up and I will leave it to you to visit www.firebase.com and discover it for yourself.

Once you have created a project or logged into your existing Firebase project, **_Go to Settings -> General -> scroll down and look for config under Firebase SDK Snippet._**

```
const firebaseConfig = {
  apiKey: "xxxxxxxxxxxxxxxx",
  authDomain: "xxxxx-dev.firebaseapp.com",
  databaseURL: "https://xxxxxx-dev.firebaseio.com",
  projectId: "xxxx-dev",
  storageBucket: "xxxxxxx-dev.appspot.com",
  messagingSenderId: "xxxxx",
  appId: "1:xxxxx:web:xxxxx",
  measurementId: "G-xxxxx"
};
```

### Step 3 - Update reducers.js in react-boilerplate code

The default code of the boilerplate in reducer.js would need a few changes to build in the reducers from react-redux-firebase. This would set the locations in your redux-store where the package will bind data from firebase/firestore.

I have assumed here that you would want to use both firebase and firestore. You can remove the firestore lines if you are only using firebase to store your data.

```
import { combineReducers } from 'redux';
import { firebaseReducer } from 'react-redux-firebase';
import { firestoreReducer } from 'redux-firestore';

/**
 * Merges the main reducer with the router state and dynamically injected reducers
 */
export default function createReducer(injectedReducers = {}) {
  const rootReducer = combineReducers({
	 ....
    firestore: firestoreReducer,
    firebase: firebaseReducer,
    .....
  });

  return rootReducer;
}

```

### Step 4 - Changes to app.js

Changes needed to app.js are detailed below.

End Objective of the changes in the file is to create the `rrfProps` that would be passed via the `ReactReduxFirebaseProvider` into the `<App/>`.

This would ensure that all necessary functionality is available through out the app to ensure a two way binding between local application data (stored in redux) and the copy on Firebase/Firestore.

```
import { createFirestoreInstance } from 'redux-firestore';
import { ReactReduxFirebaseProvider } from 'react-redux-firebase';

import firebase from 'firebase/app';

// if you are not using any particular element of firebase, please feel free to remove that line
import 'firebase/auth';
import 'firebase/database';
import 'firebase/storage';
import 'firebase/firestore';
import 'firebase/functions';

// Firebase Config options
let fbConfig = {
	// paste the configuration that we copied in the previous step.
};

// Initialize firebase instance
firebase.initializeApp(fbConfig);
firebase.firestore();
firebase.functions();

// React Redux Firebase Config
const rrfConfig = {
  userProfile: null, // saves user profiles to '/users' on Firebase
  logErrors: false, // avoid the RRF errors / warnings
  useFirestoreForStorageMeta: true, // needed if you are uploading some files to storage
};

const rrfProps = {
  firebase,
  config: rrfConfig,
  // needed assuming that you will be dispatching changes to the redux firestore copy of your data
  dispatch: store.dispatch,
  createFirestoreInstance, // <- needed if using firestore
};


const render = messages => {
  ReactDOM.render(
    <Provider store={store}>
      <ReactReduxFirebaseProvider {...rrfProps}>
        <LanguageProvider messages={messages}>
          <Router>
            <App />
          </Router>
        </LanguageProvider>
      </ReactReduxFirebaseProvider>
    </Provider>,
    MOUNT_NODE,
  );
};
```

### Step 5 : Access Data in your components

Once the changes in above steps are completed you are ready to access the data.

`react-redux-firebase` provides a set of hooks to connect specific collections wtih application `redux-store`

In any file that you want to access the data. `useFirestoreConnect` hook will pull the collection passed to it into the redux firestore.

From there on you can use the `useSelector` hook to pull the data you want into a local variable.

```
import { useFirestoreConnect } from 'react-redux-firebase';

  useFirestoreConnect('users');
  const user = useSelector(
    ({ firestore: { data } }) => data.users && data.users[uid],
  );

```

If you want to sync multiple collections you could do something like this,

```
import { useFirestoreConnect } from 'react-redux-firebase';

  useFirestoreConnect('users');
  useFirestoreConnect('uploadedFiles');
  const user = useSelector(
    ({ firestore: { data } }) => data.users && data.users[uid],
  );
  const uploadedFiles = useSelector(
      ({ firestore: { data } }) => data.users && data.users[uid].uploadedFiles,
  );
```

### Step 6 - Adding Auth to your project

Note in the above snippets that `uid` is the identifier of the logged in user. In my application I am also using `firebase` authentiation as well.

To achieve this, you can add a component in the `react-boilerplate` called `Auth` with the following code,

```
import React from 'react';
import { getFirebase } from 'react-redux-firebase';
import StyledFirebaseAuth from 'react-firebaseui/StyledFirebaseAuth';
import { setGlobal } from 'reactn';
import cookie from 'react-cookies';

// style is passed here to style the login auth popup.

function Auth({ style }) {
  const firebase = getFirebase();

  const AuthUiConfig = {
    signInFlow: 'popup',
    signInSuccessUrl: '/user/authsuccess',
    signInOptions: [firebase.auth.GoogleAuthProvider.PROVIDER_ID],
    callbacks: {
      signInSuccessWithAuthResult(authResult) {
        const { user } = authResult;

        // update the local storage so that app can be hydrated with
        // these values if the cookies exist next time app is brought up.
        cookie.save(
          'au',
          {
            uid: user.uid,
            lastLogin: Date.now(),
          },
          { maxAge: 60 * 60 * 24 * 30 },
          { path: '/' },
        );
        cookie.save('isLoggedIn', { status: true }, { path: '/' });
        setGlobal({
          uid: user.uid,
          isLoggedIn: true,
        });

        window.location.assign('/user/loggedInPage');
      },
    },
  };

  return (
    <React.Fragment>
      <StyledFirebaseAuth
        uiConfig={AuthUiConfig}
        className={style}
        firebaseAuth={firebase.auth()}
      />
    </React.Fragment>
  );
}

Auth.propTypes = {};

export default Auth;
```

Somewhere on your landing page where ever you want to show the login button / firebase auth ui include,

```
 <Auth style={classes.firebaseUi} />
```

`react-firebaseui/StyledFirebaseAuth` takes care of authentication, creation of a firebase account for the user the first time they login etc. seamlessly.

You could additionally define a Google Cloud Function (Serverless backend) to trigger on a new account creation to create a new document in the users collection with the logged in users UID. I have taken most of the data from what is received by login, but you can also add additional data from a onboarding flow as well.

The application would need to have some checks to see if user is logged in or not, and render routes conditionally.

Hope this has helped you include authentication and bind your firebase/firestore into your redux-store.
