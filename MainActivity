import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.google.android.gms.auth.api.Auth;
import com.google.android.gms.auth.api.signin.GoogleSignInAccount;
import com.google.android.gms.auth.api.signin.GoogleSignInOptions;
import com.google.android.gms.auth.api.signin.GoogleSignInResult;
import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.Scopes;
import com.google.android.gms.common.SignInButton;
import com.google.android.gms.common.api.GoogleApiClient;
import com.google.android.gms.common.api.OptionalPendingResult;
import com.google.android.gms.common.api.ResultCallback;
import com.google.android.gms.common.api.Scope;
import com.google.android.gms.common.api.Status;
import com.google.android.gms.drive.Drive;
import com.google.android.gms.drive.DriveApi;
import com.google.android.gms.drive.DriveContents;
import com.google.android.gms.drive.DriveFile;
import com.google.android.gms.drive.DriveFolder;
import com.google.android.gms.drive.DriveId;
import com.google.android.gms.drive.MetadataChangeSet;
import com.google.android.gms.drive.query.Filters;
import com.google.android.gms.drive.query.Query;
import com.google.android.gms.drive.query.SearchableField;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.Writer;
import java.util.Random;

public class MainActivity
        extends AppCompatActivity
        implements GoogleApiClient.OnConnectionFailedListener, View.OnClickListener {

    private static final String TAG = "MainActivity";
    private static final String FILE_NAME = "testing_gdrive";
    private static final int RC_SIGN_IN = 9001;

    private GoogleSignInOptions gso;
    private GoogleApiClient mGoogleApiClient;
    private TextView mStatusTextView;
    private EditText mEditText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        mStatusTextView = (TextView) findViewById(R.id.status);
        mEditText = (EditText) findViewById(R.id.writing_text);

        gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestScopes(new Scope(Scopes.DRIVE_APPFOLDER)) // if you want to save files in hidden app folder
//                .requestScopes(Drive.SCOPE_FILE,new Scope(Scopes.DRIVE_FILE)) //// if you want to save files in root folder
                .requestEmail()
                .build();

        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .enableAutoManage(this,this)
                .addApi(Auth.GOOGLE_SIGN_IN_API,gso)
                .addApi(Drive.API)
                .build();

        SignInButton signInButton = (SignInButton) findViewById(R.id.sign_in_button);
        signInButton.setSize(SignInButton.SIZE_STANDARD);

        findViewById(R.id.sign_in_button).setOnClickListener(this);
        findViewById(R.id.sign_out_and_disconnect).setOnClickListener(this);
        findViewById(R.id.sign_out_button).setOnClickListener(this);
        findViewById(R.id.button_send_data).setOnClickListener(this);
        findViewById(R.id.button_get_data).setOnClickListener(this);
    }

    @Override
    protected void onStart() {
        super.onStart();

        OptionalPendingResult<GoogleSignInResult> opr =
                Auth.GoogleSignInApi.silentSignIn(mGoogleApiClient);
        if (opr.isDone()) {
            Log.d(TAG, "Got cached sign-in");
            GoogleSignInResult result = opr.get();
            handleSignInResult(result);
        } else {
            opr.setResultCallback(new ResultCallback<GoogleSignInResult>() {
                @Override
                public void onResult(GoogleSignInResult googleSignInResult) {
                    handleSignInResult(googleSignInResult);
                }
            });
        }
    }

    @Override
    protected void onPause() {
        if (mGoogleApiClient != null) {
            mGoogleApiClient.disconnect();
        }
        super.onPause();
    }

    @Override
    public void onConnectionFailed(ConnectionResult connectionResult) {
        Log.d(TAG, "connection failed: " + connectionResult.toString());
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()){
            case R.id.sign_in_button:
                signIn();
                break;
            case R.id.sign_out_button:
                signOut();
                break;
            case R.id.disconnect_button:
                revokeAccess();
                break;
            case R.id.button_send_data:
                sendDataToGDrive();
                break;
            case R.id.button_get_data:
                getDataFromGDrive();
                break;
        }
    }

    private void getDataFromGDrive() {
        Query query = getQuery();
        DriveFolder driveFolder = getFolder();
        lookFiles(query,driveFolder);
    }
    private Query getQuery(){
        Query query = new Query.Builder()
                .addFilter(Filters.eq(SearchableField.TITLE,FILE_NAME))
                .build();
        return query;
    }
    private DriveFolder getFolder(){
        DriveFolder folder = Drive.DriveApi.getAppFolder(mGoogleApiClient);
        return folder;
    }

    private void lookFiles(Query query, DriveFolder folder){
        folder.queryChildren(mGoogleApiClient, query)
                .setResultCallback(new ResultCallback<DriveApi.MetadataBufferResult>() {
                    @Override
                    public void onResult(DriveApi.MetadataBufferResult metadataBufferResult) {
                        if(metadataBufferResult.getMetadataBuffer().getCount() == 0){
                            Log.d(TAG, "No file in GDrive");
                        }else{
                            DriveId idLookingFor = metadataBufferResult.getMetadataBuffer().get(0).getDriveId();
                            Log.d(TAG,"Found ID: " + idLookingFor.encodeToString());
                            getDataFromFileFromGDrive(idLookingFor);
                        }
                    }
                });
    }
    private void getDataFromFileFromGDrive(DriveId file_id){
        DriveFile file = Drive.DriveApi.getFile(mGoogleApiClient, file_id);
        file.open(mGoogleApiClient,DriveFile.MODE_READ_ONLY,null)
                .setResultCallback(new ResultCallback<DriveApi.DriveContentsResult>() {
                    @Override
                    public void onResult(DriveApi.DriveContentsResult driveContentsResult) {
                        if (!driveContentsResult.getStatus().isSuccess()) {
                            Log.d(TAG, "Error while trying to open file");
                            return;
                        }

                        DriveContents contents = driveContentsResult.getDriveContents();
                        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(contents.getInputStream()));
                        StringBuilder builder = new StringBuilder();
                        String line;
                        try {
                            while ((line = bufferedReader.readLine()) != null) {
                                builder.append(line);
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                        }

                        String contentAsString = builder.toString();
                        mEditText.setText("from file: " + contentAsString);

                        //                        contents.commit(mGoogleApiClient,null);
                    }
                });
    }

    private void sendDataToGDrive() {
        Drive.DriveApi.newDriveContents(mGoogleApiClient)
                .setResultCallback(new ResultCallback<DriveApi.DriveContentsResult>() {
                    @Override
                    public void onResult(DriveApi.DriveContentsResult driveContentsResult) {
                        if (!driveContentsResult.getStatus().isSuccess()) {
                            Log.d(TAG,"error while trying to create new file contents");
                            return;
                        }
                        final ResultCallback<DriveFolder.DriveFileResult>fileCallback = new
                                ResultCallback<DriveFolder.DriveFileResult>() {
                                    @Override
                                    public void onResult(DriveFolder.DriveFileResult result) {
                                        if (result.getStatus().isSuccess()) {
                                            Toast.makeText(getApplicationContext(),
                                                    "File created: " + result.getDriveFile().getDriveId(),
                                                    Toast.LENGTH_SHORT).show();
                                        }
                                    }
                                };

                        final DriveContents driveContents = driveContentsResult.getDriveContents();
                        new Thread(){
                            @Override
                            public void run(){
                                OutputStream os = driveContents.getOutputStream();
                                Writer wr = new OutputStreamWriter(os);
                                String s = mEditText.getText().toString() + "\t" + new Random(100);
                                try {
                                    wr.write(s);
                                    wr.close();
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }

                                MetadataChangeSet metadata =
                                        new MetadataChangeSet.Builder()
                                                .setTitle(FILE_NAME)
                                                .setMimeType("text/plain")
                                                .setStarred(true)
                                                .build();
                                // getAppFolder means hidden app folder
                                // getRootFolder means root gdrive's folder
                                Drive.DriveApi.getAppFolder(mGoogleApiClient)
                                        .createFile(mGoogleApiClient,metadata,driveContents)
                                        .setResultCallback(fileCallback);

                            }
                        }.start();
                    }
                });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == RC_SIGN_IN){
            GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
            handleSignInResult(result);
        }
    }

    private void handleSignInResult(GoogleSignInResult result) {
        Log.d(TAG, "handleSignInResult:" + result.isSuccess());
        if (result.isSuccess()) {
            // Signed in successfully, show authenticated UI.
            GoogleSignInAccount acct = result.getSignInAccount();
            mStatusTextView.setText(getString(R.string.signed_in_fmt, acct.getDisplayName() + "\n" + acct.getEmail()));
            updateUI(true);
        } else {
            // Signed out, show unauthenticated UI.
            updateUI(false);
        }
    }

    private void signIn() {
        Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
        startActivityForResult(signInIntent,RC_SIGN_IN);
    }

    private void signOut() {
        Auth.GoogleSignInApi.signOut(mGoogleApiClient).setResultCallback(
                new ResultCallback<Status>() {
                    @Override
                    public void onResult(Status status) {
                        // [START_EXCLUDE]
                        Log.d(TAG,"signOut:onResult: " + status.toString());
                        updateUI(false);
                        // [END_EXCLUDE]
                    }
                });
    }

    private void revokeAccess() {
        Auth.GoogleSignInApi.revokeAccess(mGoogleApiClient).setResultCallback(
                new ResultCallback<Status>() {
                    @Override
                    public void onResult(Status status) {
                        // [START_EXCLUDE]
                        Log.d(TAG,"revokeAccess:onResult: " + status.toString());
                        updateUI(false);
                        // [END_EXCLUDE]
                    }
                });
    }

    private void updateUI(boolean signedIn) {
        if (signedIn) {
            findViewById(R.id.sign_in_button).setVisibility(View.GONE);
            findViewById(R.id.sign_out_and_disconnect).setVisibility(View.VISIBLE);
        } else {
            mStatusTextView.setText(R.string.signed_out);

            findViewById(R.id.sign_in_button).setVisibility(View.VISIBLE);
            findViewById(R.id.sign_out_and_disconnect).setVisibility(View.GONE);
        }
    }
}
