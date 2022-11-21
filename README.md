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

`uploadString` works fairly well:

```js
import { initializeApp } from "firebase/app";
import * as functions from "firebase-functions";
import { getStorage, ref, uploadString, connectStorageEmulator, getMetadata } from "firebase/storage";

const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  databaseURL: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};

initializeApp(firebaseConfig);

const storage = getStorage();
const storageRef = ref(storage, 'Pictures/message.txt');
// connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud

export const sillyString = functions.firestore.document('Strings/{docId}').onUpdate((change, context) => {
  const message = 'This is my message.';
  uploadString(storageRef, message).then((snapshot) => {
    console.log('Uploaded a raw string!');
  });
});
```

You specify a string and where you want it to go. It uploads to your Cloud Storage.

Let's try the emulator. Remove the comments from:

```js
connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud
```

My terminal logs that the function executed without a problem but nothing appears in the Storage emulator. Comment out `connectStorageEmulator`.

Let's get the metadata. 



### Upload a Uint8 Array

`uploadBytes` can upload a `file`, a `blob`, or a `UINT8 array`. 

A `UINT8` is an [Unsigned Integer with 8 bits](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array), i.e., 0 through 255 decimal. Uploading a `UINT8 array` is similar to uploading a string:

```js
import { getStorage, ref, uploadBytes, connectStorageEmulator } from "firebase/storage";

const storage = getStorage();
const storageRef = ref(storage, 'UINT8_Arrays/my-first-uint8-array'); // where you want the string to go
connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud  

const metadata = {
  contentType: 'application/octet-stream',
};

const bytes = new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x77, 0x6f, 0x72, 0x6c, 0x64, 0x21]);
uploadBytes(storageRef, bytes).then((snapshot) => {
  console.log('Uploaded an array!');
});
```


