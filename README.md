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

`uploadString` works, more or less:

```js
import { initializeApp } from "firebase/app";
import * as functions from "firebase-functions";
import { getStorage, ref, uploadBytes, uploadString, connectStorageEmulator, getMetadata, updateMetadata, getDownloadURL } from "firebase/storage";

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
const storageRef = ref(storage, 'Pictures/message.txt'); // target location for to upload to
// connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud

export const sillyString = functions.firestore.document('Strings/{docId}').onUpdate((change, context) => {
  // const metadata = {
  //   contentType: 'text/plain',
  // };

  async function stringUploader() {
    try {
      const message = 'This is my message.'; // string to upload
      await uploadString(storageRef, message);
      console.log('Uploaded a raw string!');
    } catch (error) {
      console.error(error);
    }
  }

  return stringUploader()
});
```

To execute the function, open your emulator console to Firestore. Make a new collection "Strings", let Firestore generate a document ID, and make a property. Put whatever you want in the key-value pair. I like `season: 'winter'`.

Change the value, e.g., to `spring`. We have to change the value because we're triggering the function with `onUpdate`, not `onCreate`.

Open your Firebase Console and you should see `message.txt` in your Cloud Storage. 

If you get an error message 

```
Firebase Storage: User does not have permission to access 'Pictures/message.txt'. (storage/unauthorized)"
```

then you need to change your Storage Rules to:

```js
match /Pictures/{file=**} {
  allow read, write
}
```

Every Cloud Function I run in the emulator logs a warning: `Your function timed out after ~60s.` I ignore these warnings.

#### Storage Emulator

Let's try the emulator. Remove the comments from:

```js
connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud
```

My terminal logs that the function begins executing, then nothing but the timed out warning. The Storage Emulator doesn't appear to be working.

#### Metadata

Remove the comments from 

```js
const metadata = {
  contentType: 'text/plain',
};
```

and add `metadata` to this line:

```js
await uploadString(storageRef, message, metadata)
```

Comment out this line:

```js
// connectStorageEmulator(storage, "localhost", 9199); // comment out to write to the cloud
```

Execute the function. I get this error message:

```
'Firebase Storage: An unknown error occurred, please check the error payload for server response. (storage/unknown)'
```

### Upload a Uint8 Array

`uploadBytes` can upload a `file`, a `blob`, or a `UINT8 array`. 

A `UINT8` is an [Unsigned Integer with 8 bits](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array), i.e., 0 through 255 decimal. Uploading a `UINT8 array` is similar to uploading a string, but we change `uploadString` to `uploadBytes`:

```js
export const uploadUint8 = functions.firestore.document('Uint8/{docId}').onUpdate((change, context) => {
  // const metadata = {
  //   contentType: 'text/plain',
  // };

  async function uint8Uploader() {
    try {
      const bytes = new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x77, 0x6f, 0x72, 0x6c, 0x64, 0x21]);
      await uploadBytes(storageRef, bytes)
      console.log('Uploaded a UINT8 array!');
    } catch (error) {
      console.error(error);
    }
  }

  return uint8Uploader()
});
```

You should see `message.txt` in your Firebase Console. The number of bytes in the stored array should be the number of bytes in the array in your function.

#### Metadata

Remove the comments from 

```js
const metadata = {
  contentType: 'text/plain',
};
```

and add `metadata` to this line:

```js
await uploadBytes(storageRef, bytes, metadata);
```

That should execute. If you open the file data in your Firebase Console you'll see the metadata.

### Upload a File or Blob

Now we get to the part you've been waiting for. There are two "gotchas" here. 

First, you can't just upload any file. You can only upload a [JavaScript File](https://developer.mozilla.org/en-US/docs/Web/API/File). JavaScript Files are made with the `new File()` constructor, which isn't available in Node. In Node you use

```js
fs.writeFile('<fileName>',<contenet>, callbackFunction)
```

There are several tutorials teaching how to create a file in Node.js. It's not simple.

But wait! You can upload a [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) instead of a file. A blob is a "file-like object of immutable, raw data...Blobs can represent data that isn't necessarily in a JavaScript-native format." That sounds like we can get around the JavaScript File requirement but I haven't found this to be a solution.

Second, you must `import` your file into `index.js`, which means your file must be an ES module. This is simple enough for a text file but gets complex with pictures or other large binary files. This is done with [Webpack](https://webpack.js.org/). I spent a day learning Webpack and gave up.

#### Make and Import an ES Module Text File

Let's make a simple text file. Make a new file and call it `myModule.js`:

```js
// myModule.js
export const txtFile = "Hello world";
```

In `index.js`, import your new ES module:

```js
import { txtFile } from "./myModule.js";
```

So far, so good. We have a file available but it's not a JavaScript File. Let's try uploading our file to Storage.

Let's change our target location:

```js
const storageRef = ref(storage, 'Pictures/hello'); // target location for to upload to
```

##### Write the Cloud Function

Here's the Cloud Function.

```js
import { txtFile } from "./myModule.js";

export const uploadFile = functions.firestore.document('File/{docId}').onUpdate((change, context) => {
  // const metadata = {
  //   contentType: 'text/plain',
  // };

  async function fileUploader() {
    try {
      // const file = new File(txtFile);
      const file = txtFile;
      console.log(file); // Hello world
      await uploadBytes(storageRef, file);
      console.log('Uploaded a file!');
    } catch (error) {
      console.error(error);
    }
  }

  return fileUploader()
});
```

That throws this error:

```
TypeError: Cannot read properties of undefined (reading 'byteLength')
```

In this error message, `undefined` is the file to upload. The error message is saying that it can't find a file to upload. We gave it a file but not a JavaScript File. The log shows "Hello world", indicating that the contents of the file have been imported into `index.js` but it wasn't imported as a file. The contents have to be converted into a file.

The syntax for blobs and files is the same. Firebase should have uploaded our file as a blob but it didn't.

Switch the comments:

```js
const file = new File(txtFile);
// const file = txtFile;
```

That threws this error:

```
ReferenceError: File is not defined
```

As I noted, `new File()` isn't available in Node.

#### Upload a Blob

But `new Blob()` is available in Node:

```js
import { Blob } from 'buffer';

...

const blob = new Blob(txtFile);
```

That threw this error:

```
TypeError [ERR_INVALID_ARG_TYPE]: The "sources" argument must be a sequence. Received type string ('Hello world')
```

##### Back to UINT8

Let's try converting our text file into a UINT8 array.

export const uploadUint8 = functions.firestore.document('Uint8/{docId}').onUpdate((change, context) => {
  const metadata = {
    contentType: 'application/octet-stream',
  };

  var enc = new TextEncoder(); // always utf-8
  const bytes = enc.encode(txtFile);
  console.log(bytes);

  async function uint8Uploader() {
    try {
      // const bytes = new Uint8Array([0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x77, 0x6f, 0x72, 0x6c, 0x64, 0x21]);
      await uploadBytes(storageRef, bytes, metadata);
      console.log('Uploaded a UINT8 array!');
    } catch (error) {
      console.error(error);
    }
  }

  return uint8Uploader()
});

All we did was add

```js
  var enc = new TextEncoder(); // always utf-8
  const bytes = enc.encode(txtFile);
  console.log(bytes);
```

and comment out the old UINT8 array.

This works, was easy, and you can compare the log of 11 integers to the uploaded file size (11 bytes).

#### Download a File From an API

Let's try downloading a file from an API then uploading the file to Storage. We'll download this audiofile from the Oxford English Dictionary:

```
https://audio.oxforddictionaries.com/en/mp3/winter__us_2.mp3
```

Put that in a browser and listen to the audiofile.

We'll use the npm package [got](https://www.npmjs.com/package/got) for our http requests. In your terminal get `got`:

```
npm install got
```

Make a new function in `index.js`:

```js
import got from 'got';

export const downloadFromAPI = functions.firestore.document('API/{docId}').onUpdate((change) => {
  let oedAudioDownloadURL = 'https://audio.oxforddictionaries.com/en/mp3/winter__us_2.mp3';
  let word = change.after.data().word;
  let audioType = 'mp3';
  
  const storage = getStorage();
  const storageRef = ref(storage, 'Pictures/' + word + '.' + audioType);
  
  // const metadata = {
  //   contentType: 'audio/mpeg',
  //   contentLanguage: 'en', // must be ISO 639-1
  //   customMetadata: {
  //     'audioType': audioType,
  //     'longAccent': 'United_States',
  //     'shortAccent': 'US',
  //     'longLanguage': 'English',
  //     'shortLanguage': 'en',
  //     'source': 'Oxford Dictionaries',
  //     'word': word,
  //   },
  // };
  
  async function getAudiofileWrite2Storage() {
    try {
      let file = await got(oedAudioDownloadURL);
      await uploadBytes(storageRef, file['rawBody']);
      const downloadURL = await getDownloadURL(ref(storage, 'Pictures/' + word + '.' + audioType));
      console.log(downloadURL);
      const gotMetadata = await getMetadata(storageRef);
      console.table(gotMetadata);

      // Update metadata properties
      // updateMetadata(storageRef, metadata)
      // console.log("Metadata updated: ");
      // console.table(metadata);

    } catch (error) {
      console.error(error);
    }
  };
  
  return getAudiofileWrite2Storage()
});
```

We import `got` to handle the API download.

We trigger the function from the collection `API`.

We set the URL for the API download.

We get the word that you entered into Firestore to trigger the function. This becomes the filename.

We set the file type as `mp3`.

We set `storage` and `storageRef`.

We make an async function `getAudiofileWrite2Storage`.

First, we use `got()` to download the audiofile from the OED.

Next, we run `uploadBytes()` to upload the file to Cloud Storage. Note the undocumented keyword `['rawBody']`. This enables `uploadBtyes()` to handle a file that isn't a JavaScript File. (But not the "Hello world" file from the previous section.)

That's it, two lines to download a file from an API and upload it to Cloud Storage. In years past I had to write twenty complex lines of Node and Firebase to do this.

Then we get the download URL and log it. You should see this in your terminal. Put the URL in a browser window and you'll hear the same file that you downloaded directly from the OED.

Lastly, we get the metadata with `getMetadata()` and log this.

##### Update the metadata

Look at the logged metadata. Several values are undefined, and `contentType` is set to the default `application/octet-stream`, which is for UINT8 arrays. Let's fix these. Remove the comments:

```js
const metadata = {
    contentType: 'audio/mpeg',
    contentLanguage: 'en', // must be ISO 639-1
    customMetadata: {
      'audioType': audioType,
      'longAccent': 'United_States',
      'shortAccent': 'US',
      'longLanguage': 'English',
      'shortLanguage': 'en',
      'source': 'Oxford Dictionaries',
      'word': word,
   },
};

...

  updateMetadata(storageRef, metadata)
  console.log("Metadata updated: ");
  console.table(metadata);
```

When you execute this you should see the new metadata logged in the console. Go to your Firebase Console, select the file, click "Other metadata" and you'll see the updated metadata.

Note that there are two types of metadata. Using `getMetadata()` you can see the 15 metadata fields provided by Firebase Storage. Then there's a 16th field `customMetadata`. This is an object within the metadata object.

## Download From Storage

## Conclusion

We've succeeded in uploading a local small text file and a remote API large binary file. We haven't been able to upload a local large binary file but with more effort with Webpack we should we able to do that.

We were able to upload to Cloud Storage but not to the Storage Emulator.
