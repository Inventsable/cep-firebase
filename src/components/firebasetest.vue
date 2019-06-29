<template>
  <div></div>
</template>

<script>
// @ is a symbol for ./src
import db from '@/firebase/init'

export default {
  name: 'firebasetest',
  data: () => ({
    users: []
  }),
  created() {
    // Only one of these is needed, having both produces duplicates.

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
