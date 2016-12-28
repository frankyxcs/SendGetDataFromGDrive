# SendGetDataFromGDrive
Short app showing how to send and download files from Google Drive


In onCreate method we create GoogleSignInOptions and GoogleApiClient objects
which are responsible for connecting to Google Account and handling sending/downloading
files from GDrive.

In onStart we are trying to silentSignIn which is basically automatically singing without
showing that information to user. If data is cached, we succesfully achieved that.

Sending data to GDrive is easy, we need just previously created GoogleApiClient.
Creating and sending data is covered in "run" method, creating OutputStream of DriveContents,
writing data to it, creating metadata for that file and create it in the appfolder/root folder.

Downloading file from GDrive is a little bit complicated. 
Firstly, I create Query object with filter for file's title, and DriveFolder object that is basically
a file we are going to look in for specific file. 
Later, I queryChildren of that folder, which result in list of found files. 
Get Id of that file (we know, that there is one file), and I pass this Id to method which opens file
with given Id and read its content.
