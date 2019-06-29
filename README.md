# cep-firebase sample

## Why should I care about a .env file?

![](./src/assets/goodguy.png)

If you don't take precautions to hide your API keys and accidentally expose them somewhere like GitHub, you risk giving others access to your data -- like this bot who extracted my API keys 37 seconds after I published this repo.

Vue CLI comes with [native .env support](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables) for environment variables. This means that we can have a `.env` file [at the root of our repo (like this one)](https://github.com/Inventsable/cep-firebase/blob/master/.env) which contains any shared variables to be used within any other file of our repository, with contents like this:

## ./.env
```
VUE_APP_FIREBASE_KEY=AIzaSyCxuLj0c9pFVikYgppl7qfCKPn4iAf0URc
VUE_APP_FIREBASE_AUTHDOMAIN=cep-sample.firebaseapp.com
VUE_APP_FIREBASE_DATABASEURL=https://cep-sample.firebaseio.com
VUE_APP_PROJECTID=cep-sample
VUE_APP_FIREBASE_BUCKET=
VUE_APP_MESSAGINGSENDERID=850065936779
VUE_APP_MESSAGINGSENDERID=1:850065936779:web:6d5a884eb252c63d
```

> **CAUTION:** Always include the .env file in your gitignore when exposing your code (like putting it on GitHub). It is not gitignored in this example for demonstration purposes only.

Above are the contents from our config variable from firebase:

![](./src/assets/anno.png)

We can now copy/paste the same config file for any and all of our firebase projects or use certain key/value pairs across the scope of our entire project (no need for import, require, or other methods):

## [./src/firebase/init.js](https://github.com/Inventsable/cep-firebase/blob/master/src/firebase/init.js)

```js
import firebase from 'firebase/app'
import firestore from 'firebase/firestore'

var config = {
  apiKey: process.env.VUE_APP_FIREBASE_KEY,
  authDomain: process.env.VUE_APP_FIREBASE_AUTHDOMAIN,
  databaseURL: process.env.VUE_APP_FIREBASE_DATABASEURL,
  projectId: process.env.VUE_APP_PROJECTID,
  storageBucket: process.env.VUE_APP_FIREBASE_BUCKET,
  messagingSenderId: process.env.VUE_APP_MESSAGINGSENDERID,
  appId: process.env.VUE_APP_MESSAGINGSENDERID
};

const firebaseApp = firebase.initializeApp(config);

export default firebaseApp.firestore();
```

Anything defined in `.env` will be accessible in the `process.env` object, with an added layer of security by not including sensitive keys or other info while being able to expose the bulk of any source code.

---

## [./src/components/firebasetest.vue](https://github.com/Inventsable/cep-firebase/blob/master/src/firebase/init.js)

### Three methods to retrieve data from Firestore database:

```html
<script>
// @ is a symbol for ./src, 'db' is abbreviation for 'database'
import db from '@/firebase/init'

export default {
  name: 'firebasetest',
  data: () => ({
    users: []
  }),
  // Best done in created() since it's asynchronous and non-blocking. There will be a slight delay before any database info is available.
  created() {
    // Only one of these is needed, having all produces duplicates.

    // Get firestore database collection once
    db.collection("users")
      .get()
      .then(snapshot => {
        if (snapshot.docs.length) {
          snapshot.docs.forEach(snap => {
            const doc = snap.data()
            console.log('STANDARD:')
            console.log(doc)
          })
        }
      })

    // Query for a single document read to find a particular item
    db.collection("users").where('name', '==', 'Tom')
      .get()
      .then(snapshot => {
        if (snapshot.docs.length) {
          snapshot.docs.forEach(snap => {
            const doc = snap.data()
            console.log('QUERY:')
            console.log(doc)
          })
        }
      })

    // Display entire collection contents and update in real time
    const ref = db.collection("users");
    ref.onSnapshot(snapshot => {
      snapshot.docChanges().forEach(change => {
        if (/added/i.test(change.type)) {
          const doc = change.doc.data();
          console.log('REALTIME:')
          console.log(doc) 
        }
      });
    });
  }
}
</script>
```

### Results and firestore database in question:

![](./src/assets/firestore.png)