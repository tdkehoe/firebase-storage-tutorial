# Firebase Storage: Stuff Left Out of the Official Documentation

Firebase Storage is a database for storing large digital files such as photos or audio. The Firebase team has improved the syntax to make Storage easy to use. However, the documentation leaves out a few things.

This reference will use Firebase Cloud Functions, ES modules, and the Firebase Emulator.

## Initialize and Test the Firebase Emulator

Follow my tutorial about the Firebase Emulator to start the Firestore, Functions, and Storage emulators.

## Getting Started and Creating a Reference

This tutorial supplements the official documentation for Cloud Storage on Web. Follow the [Get Started](https://firebase.google.com/docs/storage/web/start) to set up Storage in your Firebase Console.

The next documentation page, [Create a Cloud Storage reference on Web](https://firebase.google.com/docs/storage/web/create-reference), is clear about making references to the paths or locations where your files will go.

## Uploading Stuff

The third documentation page, [Upload files with Cloud Storage on Web](https://firebase.google.com/docs/storage/web/upload-files), looks straighforward enough. Two commands are provided for uploading to Storage: `uploadString` and `uploadBytes`.

### Upload a String

`uploadString` couldn't be easier to use:

```js
import { getStorage, ref, uploadString } from "firebase/storage";

const storage = getStorage();
const storageRef = ref(storage, 'Strings/message.txt');

const metadata = {
  contentType: 'text/plain',
};

const message = 'This is my message.';
uploadString(storageRef, message).then((snapshot) => {
  console.log('Uploaded a raw string!');
});
```

You just specify a string and where you want it to go




