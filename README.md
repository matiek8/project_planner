This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

To activate firebase you have to create a firebase profile then copy a firebase script to a fbConfig.js in projec_planner/src/config. Don't forget to add rules in Firebase database:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /projects/{project} {
      allow read, write: if request.auth.uid != null;
    }
    match /users/{userId} {
    	allow create;
      allow read: if request.auth.uid != null;
      allow write: if request.auth.uid == userId;
    }
    match /notifications/{notification} {
      allow read: if request.auth.uid != null;
    }
  }
}
```

To activate functions you need to install   
`npm install -g firebase-tools`  
`firebase login`  
`firebase init`  

Then add functions to project_planner/functions/index.js

```
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp(functions.config().firebase);

exports.helloWorld = functions.https.onRequest((request, response) => {
  response.send("Hello!");
});

const createNotification = (notification => {
  return admin.firestore()
    .collection('notifications')
    .add(notification)
    .then(doc => console.log('notification added', doc))
});

exports.projectCreated = functions.firestore.document('projects/{projectId}').onCreate(doc => {
  const project = doc.data();
  const notification = {
    content: 'Added a new project',
    user: `${project.authorFirstName} ${project.authorLastName}`,
    time: admin.firestore.FieldValue.serverTimestamp()
  };
  return createNotification(notification);
});

exports.userJoined = functions.auth.user()
  .onCreate(user => {

    return admin.firestore().collection('users')
      .doc(user.uid).get().then(doc => {

        const newUser = doc.data();
        const notification = {
          content: 'Joined the party',
          user: `${newUser.firstName} ${newUser.lastName}`,
          time: admin.firestore.FieldValue.serverTimestamp()
        };

        return createNotification(notification);

      });
});

```

deploy functions:  
`firebase deploy --only functions`

deploy full project:  
`npm build`  
`firebase deploy`
