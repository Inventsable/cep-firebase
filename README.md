# cep-firebase sample

Vue CLI comes with [native .env support](https://cli.vuejs.org/guide/mode-and-env.html#environment-variables) for environment variables. This means that we can have a `.env` file at the root of our repo which contains any shared variables to be used within any other file of our repository, with contents like this:

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

> **CAUTION:** Always include the .env file in your gitignore when exposing your code (like putting it on GitHub). It is not gitignored in this example for demonstration purposes only, but publishing your API keys potentially gives any one the ability to use or modify your database with read/write privileges.

Above are the contents from our config variable from firebase:

![](./src/assets/anno.png)

We can now copy/paste the same config file for any and all of our firebase projects or use certain key/value pairs across the scope of our entire project (no need for import, require, or other methods):

## ./src/firebase/init.js

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

## Firestore database (./src/components/firebasetest.vue)

### Three methods to retrieve data:

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

    // Display firestore database collection once
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

    // Query for a single document read to find particular item
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

    // Display any added items in real time
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

### Results and firestore database:

![](./src/assets/firestore.png)