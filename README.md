# Kontiki
Secure MR glasses


## Apps:
### 1. Verieye authentication [App](https://github.com/Shubhodeepdas83/KontikiVeriEye):
* This app uses Verieye sdk for making authentication system using iris image.
* Extraction: extract the template from image
* Identification:  check whether the extracted template already exist in database or not
* Enrollment:  if template doesn't match the new template will be added to database.
        
### 2. Landscape [App](https://github.com/Shubhodeepdas83/KontikiLandscape)
* this app is similar to Kontiki App in most functionality.
* the app uses 'Verieye authentication app' for it's authntication process
* the app continuously calls http://$ipAddress/pstate.txt and http://$ipAddress/imgcounter.txt to check Device-state and image-counter
* all encrypted messages and files are decrypted automatically by the system itself
        
### 3. Kontiki [App](https://github.com/Shubhodeepdas83/Kontiki)
* this app needs 'Landscape app' to scan its Qr-code for authentication
* once the app(qr-code) is scanned. 'Landscape app' user account will be open in this app too.
* the encrypted messages and files are inaccessible directly
* user must use 'Landscape app' to access encrypted messages and files of 'Kontiki app'
* once the message or file is encrypted, it can not be decrypted fully but only get decrypted entity temporarily.


## Auth Flow:

### 1. Verieye authentication App:
![](https://firebasestorage.googleapis.com/v0/b/kontiki-mr-security.appspot.com/o/readme%20images%2FkontikiBlockDiagram.jpg?alt=media&token=dc372b79-e72f-4898-91b6-7bf6879d8420)

#### Important functions:
* getLicense(): check the license and initialize client.
* downloadImages(): download all the images(3 images) from local server.
* extractTemplate(): extract the template/subject from image we downloaded.
* identity(): check whether the template/subject already exist or not in the database.
* enrollSubject(): insert/save the template/subject in the database(if the template already exist it will throw error).
* onHandlerResult(): this function execute whenever result of extraction, identification and enrollment(success or failure) is return.
* createToken(): used to create token from a string

Source for getting User-id:
* During Enrollment process a unique firebase key/id is created using ``` FirebaseDatabase.getInstance().getReference("genIds").push().key ```.
* genIds is firebase path whose rule is set as: read: true, write: true.
* Now that key/id is assigned as subject id for that particular subject(iris template).

### 2. Landscape App:
* The authentication of this app is fully depend on 'Verieye authentication App'.
* When this app open it continuously check device-state(whether device is wearing or removed) and image-counter, by calling ```http://$ipAddress/pstate.txt``` and ```http://$ipAddress/imgcounter.txt```
* Once it get device-state as 1 and image-counter greater than previously stored value it open 'Verieye authentication App' using Intent and also send some values like ipaddress, image-counter and action.
* How 'Verieye authentication App' handle the request has been already discuss in above diagram.
* After successful enrollment 'Verieye authentication App' send back token-Id which is used by this app(Landscape App) for firebase custom authentication.
* This app will also used for scanning the Kontiki App's encrypted files and messages.

#### Important functions:
* runVrListener(): As soon as 'Landscape App' runs this function is called. This function checks device-state and image-counter.
* onImageCounterChange(): Whenever device state is 1 and image-counter change this function called. This function start 'Landscape App'.
* handleReceivedIntent(): When 'Landscape App' finished it sends back some data it action wither error or success with error message or token-id. 
* customTokenLogin(): If 'Landscape App' send back token-id, this function use token-id for login.
* isUserExist(): check whether user new or old. More specifically it checks whether user has already created account in firebase database. If not ```startActivityEnterName()``` called else ```startActivityWelcome```.


### 3. Kontiki App:
* This app require it's QR-code to be scanned for it's Authentication Process by 'Landscape App'.
* When this app open, it generate unique firebase key/id then push it into firebase database under ``` userQrAuths```
* then it generate qr-code using this key/id.
* When 'Landscape App' scan the qr-code of 'Kontiki App'. the 'Landscape App' push it's user-id under ```userQrAuths``` firebase path.
* once user-id is pushed, "Kontiki App" authenticate the app using this user-id.

#### Important functions:
* anonymousLogin(): for anonymous login. So that this app can communicate with Firebase Database.
* generateQr(): it generate unique firebase key/id and push it to firebase database under ```userQrAuths``` path.
* showQrCode(): show the qr-code so that user can scan it.
* listenForUserId(): after showing qr-code the app listen for user-id under ```userQrAuths``` path, in it's qr-code. When user-id found then it open the app with found user-id account.
