# Repro for isse 11167

# Versions

firebase-tools: v15.32.0

## Steps to reproduce

1. Run `firebase emulators:start --project PROJECT_ID`
2. Open `http://127.0.0.1:5000/?useEmulator=true` in the browser
3. Click "Initialize Firebase"
4. Click "Seed Data"
5. Click "Read Storage File" 
```
Error: Firebase Storage: Object 'users/11167' does not exist. (storage/object-not-found)
Not Found
```

The error should have been a permission error

Comparing behaviour when runnign against prod

1. Run `firebase emulators:start --project PROJECT_ID`
2. Open `http://127.0.0.1:5000/?useEmulator=false` in the browser
   - `/?useEmulator=false` connects to prod
3. Click "Initialize Firebase"
4. Click "Seed Data"
5. Click "Read Storage File" 
   - Permission error since 3 documents are read(expected)
```
Error: Firebase Storage: User does not have permission to access 'users/11167'. (storage/unauthorized)
```

## Notes

Why will 3 documents be read?

With the ff rules 
```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/{allPaths=**} {
      allow read: if 
        firestore.exists(/databases/(default)/documents/users/$(userId)) 
        && (firestore.get(/databases/(default)/documents/user-levels/$(userId)).data.user_level >= 50
        || firestore.get(/databases/(default)/documents/user-status/$(userId)).data.is_active == true);
      allow write: if true;
    }
  }
}
```

and with the ff docs

`user/11167` - exists
`user-levels/11167` - user_level: 49
`user-status/11167` - is_active: true

Since `user/11167` exists(1 read), it will go to check if `user-levels/11167` is >= 50(2nd read), which would fail since it is set to 49, it will then check if `user-status/11167` `is_active` is true(3rd read).