## Synchronizace dat - SyncAdapter
[SyncAdapter](https://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html) slouží ke zpracování operací na pozadí. Typicky synchronizace dat mezi zařízením a serverem. Registruje se u SyncManageru, který je společný pro celý systém a stará se o spouštění synchronizace - pokud je explicitně vyžádána nebo naplánována.

Proč používat SyncAdapter, když toto obstará i Service?
* **Optimální plánování** - Synchronizace jsou plánovány centrálně, takže jsou slučovány požadavky od více aplikací do dávek a ty jsou prováděny zaráz. Efficient Data Transfers - Understanding the Cell Radio
* **Synchronizace jen při změnách** - SyncAdapter může sledovat změny provedené ContentProviderem a synchronizovat jen v případě, že ke změně dojde
* **Automatické opakování požadavků** - pokud aktualizace selže (není internet, nedostupný server), provede se časem znovu, případně, až bude připojení k internetu
* **Součást systémového nastavení** - uživatel může ovlivnit, zda bude synchronizace probíhat

### Automatická synchronizace
*ContentResolver.setSyncAutomatically()* slouží pro vyvolání automatické synchronizace, když obdrží "network tickle" - typicky push notifikaci GCM.

#### Google Cloud Messaging ([GCM](https://developers.google.com/cloud-messaging/))
Abychom se nemuseli neustále doptávat serveru, jestli se něco změnilo, je lepší použít push notifikace a nechat se informovat ze strany serveru a pak až aktualizovat data.

### Pravidelná synchronizace
Pro pravidelnou synchronizaci je nutné mít zapnutou automatickou synchronizaci, jinak nefunguje. *ContentResolver.addPeriodicSync()* - umožňuje naplánovat pravidelné aktualizace v uvedeném intervalu. Tyto aktualizace jsou ale provedeny jen pokud je dostupný internet. Taktéž nemusí být vyvolány přesně po uplynutí uvedeného intervalu, ale později, až bude synchronizovat více aplikací.

### Manuálnísynchronizace
*ContentResolver.requestSync()* slouží k vyvolání synchronizace ve chvíli, kdy ji potřebujeme. Jeden z parametrů je Bundle. Do Bundlu je potřeba specifikovat parametry synchronizace:
* SYNC_EXTRAS_MANUAL – vynutí synchronizaci, ikdyž je vypnuté automatické synchronizování
* SYNC_EXTRAS_EXPEDITED - vynutí synchronizaci ihned, bez čekání na optimální okamžik

### Synchronizace při změně lokálních dat
Pro synchronizaci při změně lokálních dat potřebujeme vytvořit třídu rozšiřující [ContentObserver](http://developer.android.com/reference/android/database/ContentObserver.html). Ta by v metodě *onChange()* měla volat *requestSync()*. Použití ContentObserveru je směřováno tomto případě na viditelný cyklus aplikace - tam je přímá interakce od uživatele, který mění data. ContentObserver je třeba registrovat *ContentResolver.registerContentObserver()* na URI. Pokud na této URI provedena změna, je ContentObserver notifikován.

### Rušení synchronizace
*ContentResolver.cancelSync()* slouží ke zrušení už naplánované synchronizace, pokud ještě nebyla spuštěna.

## Implementace synchronizace
1. Vytvoření account authenticatoru - bude použit pro autentifikaci v SyncAdapteru
2. Vytvoření SyncAdapteru - rozhraní pro veškeré synchronizace včetně jejich aplikační logiky
3. Vytvoření SyncService - Service, které bude spouštět SyncAdapter a poskytovat mu Context
4. Propojení všech částí

### 1. AccountAuthenticator
[AccountAuthenticator](http://developer.android.com/reference/android/accounts/AbstractAccountAuthenticator.html) je komponenta, která má na starosti autentizaci uživatelů jejich přihlašovacími údaji oproti serveru. Po úspěšném ověření si uloží autorizační token, který se dále používá a uživatel se nemusí neustále znovu logovat. Zalogování je vyžadováno jen po vypršení platnosti tokenu nebo změně hesla

SyncAdapter ve své definici očekává, že budete mít vytvořený AccountAuthenticator, protože synchronizace jsou asociovány vždy s určitým typem účtu (např. Google) a případně ještě uživatelem toho účtu (např. user@google.com). Taktéž očekává uvedenou authority (ContentProvider).

Pokud nechcete používat při synchronizaci ContentProvider - nemusíte. Authority je jen String, takže stačí uvést libovolný. Pokud v aplikaci nemáte uživatelské účty, je potřeba vytvořit fake AccountAuthenticator s prázdným účtem. Ten bude sloužit k asociaci synchronizací s naší aplikací - systémový ContentResolver musí vědět, kdo synchronizaci žádá, aby požadavek vyřídil.

```java
public class UpdaterAuthenticator extends AbstractAccountAuthenticator {
 
    public UpdaterAuthenticator(Context context) {
        super(context);
    }
 
    // No properties to edit.
    @Override
    public Bundle editProperties(
            AccountAuthenticatorResponse r, String s) {
        throw new UnsupportedOperationException();
    }
 
    // Because we're not actually adding an account to the device, just return null.
    @Override
    public Bundle addAccount(
            AccountAuthenticatorResponse r,
            String s,
            String s2,
            String[] strings,
            Bundle bundle) throws NetworkErrorException {
        return null;
    }
 
    // Ignore attempts to confirm credentials
    @Override
    public Bundle confirmCredentials(
            AccountAuthenticatorResponse r,
            Account account,
            Bundle bundle) throws NetworkErrorException {
        return null;
    }
 
    // Getting an authentication token is not supported
    @Override
    public Bundle getAuthToken(
            AccountAuthenticatorResponse r,
            Account account,
            String s,
            Bundle bundle) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
 
    // Getting a label for the auth token is not supported
    @Override
    public String getAuthTokenLabel(String s) {
        throw new UnsupportedOperationException();
    }
 
    // Updating user credentials is not supported
    @Override
    public Bundle updateCredentials(
            AccountAuthenticatorResponse r,
            Account account,
            String s, Bundle bundle) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
 
    // Checking features for the account is not supported
    @Override
    public Bundle hasFeatures(
            AccountAuthenticatorResponse r,
            Account account, String[] strings) throws NetworkErrorException {
        throw new UnsupportedOperationException();
    }
}
```

```java
/**
 * The service which allows the sync adapter framework to access the authenticator.
 */
public class UpdaterAuthenticatorService extends Service {
    // Instance field that stores the authenticator object
    private UpdaterAuthenticator mAuthenticator;
 
    @Override
    public void onCreate() {
        // Create a new authenticator object
        mAuthenticator = new UpdaterAuthenticator(this);
    }
 
    /*
     * When the system binds to this Service to make the RPC call
     * return the authenticator's IBinder.
     */
    @Override
    public IBinder onBind(Intent intent) {
        return mAuthenticator.getIBinder();
    }
}
```

### 2. SyncAdapter
Tvoří se jako rozšíření třídy [AbstractThreadedSyncAdapter](http://developer.android.com/reference/android/content/AbstractThreadedSyncAdapter.html), kde se implementuje metoda *onPerformSync()*. Tělo této metody je vykonáváno v 2. vlákně.

SyncAdapter je vždy asociován s 1 typem účtu. S tímto typem účtu však na 1 zařízení může být přihlášeno několik uživatelů, proto *onPerformSync()* dostává v paramatru o který účet (uživatele) se jedná.

```java
public class UpdaterSyncAdapter extends AbstractThreadedSyncAdapter {
 
    // Interval at which to sync with the weather, in seconds.
    public static final int SYNC_INTERVAL = 60 * 60 * 24; //day
    public static final int SYNC_FLEXTIME = SYNC_INTERVAL / 3;
 
    private static final VersionComparator VERSION_COMPARATOR = new VersionComparator();
 
    public UpdaterSyncAdapter(Context context, boolean autoInitialize) {
        super(context, autoInitialize);
    }
 
    /**
     * Helper method to schedule the sync adapter periodic execution
     */
    public static void configurePeriodicSync(Context context, int syncInterval, int flexTime) {
        Account account = getSyncAccount(context);
        String authority = context.getString(R.string.content_authority);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            // we can enable inexact timers in our periodic sync
            SyncRequest request = new SyncRequest.Builder()
                    .syncPeriodic(syncInterval, flexTime)
                    .setSyncAdapter(account, authority)
                    .setExtras(Bundle.EMPTY) //enter non null Bundle, otherwise on some phones it crashes sync
                    .build();
            ContentResolver.requestSync(request);
        } else {
            ContentResolver.addPeriodicSync(account, authority, Bundle.EMPTY, syncInterval);
        }
    }
 
    /**
     * Helper method to have the sync adapter sync immediately
     *
     * @param context The context used to access the account service
     */
    public static void syncImmediately(Context context) {
        Bundle bundle = new Bundle();
        bundle.putBoolean(ContentResolver.SYNC_EXTRAS_EXPEDITED, true);
        bundle.putBoolean(ContentResolver.SYNC_EXTRAS_MANUAL, true);
        ContentResolver.requestSync(getSyncAccount(context), context.getString(R.string.content_authority), bundle);
    }
 
    public static void initializeSyncAdapter(Context context) {
        getSyncAccount(context);
    }
 
    /**
     * Helper method to get the fake account to be used with SyncAdapter, or make a new one if the
     * fake account doesn't exist yet.  If we make a new account, we call the onAccountCreated
     * method so we can initialize things.
     *
     * @param context The context used to access the account service
     * @return a fake account.
     */
    public static Account getSyncAccount(Context context) {
        // Get an instance of the Android account manager
        AccountManager accountManager = (AccountManager) context.getSystemService(Context.ACCOUNT_SERVICE);
 
        // Create the account type and default account
        Account newAccount = new Account(context.getString(R.string.app_name), context.getString(R.string.sync_account_type));
 
        // If the password doesn't exist, the account doesn't exist
        if (null == accountManager.getPassword(newAccount)) {
 
        /*
         * Add the account and account type, no password or user data
         * If successful, return the Account object, otherwise report an error.
         */
            if (!accountManager.addAccountExplicitly(newAccount, "", null)) {
                return null;
            }
            /*
             * If you don't set android:syncable="true" in
             * in your <provider> element in the manifest,
             * then call ContentResolver.setIsSyncable(account, AUTHORITY, 1)
             * here.
             */
 
            onAccountCreated(newAccount, context);
        }
        return newAccount;
    }
 
    private static void onAccountCreated(Account newAccount, Context context) {
        /*
         * Since we've created an account
         */
        UpdaterSyncAdapter.configurePeriodicSync(context, SYNC_INTERVAL, SYNC_FLEXTIME);
 
        /*
         * Without calling setSyncAutomatically, our periodic sync will not be enabled.
         */
        ContentResolver.setSyncAutomatically(newAccount, context.getString(R.string.content_authority), true);
 
        /*
         * Finally, let's do a sync to get things started
         */
        syncImmediately(context);
    }
 
    @Override
    public void onPerformSync(Account account, Bundle extras, String authority, ContentProviderClient provider, SyncResult syncResult) {
        //TODO update
    }
}
```

### 3. SyncService
SyncAdapter potřebuje ke svému spuštění Service pocházející z naší aplikace. Pokud by nebyl spuštěn z naší aplikace, neměl by práva na přístup ke zdrojům této aplikace, jako je třeba Account, nebo ContentProvider.

To souvisí se sandboxováním aplikací. Každá aplikace je spuštěna ve vlastním procesu s unikátním user id. Zdroje patřící dané aplikaci má práva využívat pouze stejné UID.

* synchronizace je nutná kvůli paralelnímu přístupu
* aplikační Context je třeba kvůli zabránění možného leaknutí instance Service

```java
public class UpdaterSyncService extends Service {
 
    private static final Object LOCK = new Object();
    private static UpdaterSyncAdapter sUpdaterSyncAdapter = null;
 
    @Override
    public void onCreate() {
        synchronized (LOCK) {
            if (sUpdaterSyncAdapter == null) {
                sUpdaterSyncAdapter = new UpdaterSyncAdapter(getApplicationContext(), true);
            }
        }
    }
 
    @Override
    public IBinder onBind(Intent intent) {
        return sUpdaterSyncAdapter.getSyncAdapterBinder();
    }
}
```

### 4. Propojení všech částí
*MainActivity.onCreate:*
```java
UpdaterSyncAdapter.initializeSyncAdapter(this);
```

*Manifest.xml*
```xml
<manifest>
    <!-- Permissions required by the sync adapter -->
    <uses-permission android:name="android.permission.READ_SYNC_SETTINGS" />
    <uses-permission android:name="android.permission.WRITE_SYNC_SETTINGS" />
    <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS" />
 
    <application>
 
        <!-- SyncAdapter's dummy authentication service -->
        <service android:name=".sync.UpdaterAuthenticatorService" >
            <intent-filter>
                <action android:name="android.accounts.AccountAuthenticator" />
            </intent-filter>
 
            <meta-data
                    android:name="android.accounts.AccountAuthenticator"
                    android:resource="@xml/authenticator" />
        </service>
 
        <!-- The SyncAdapter service -->
        <service
                android:name=".sync.UpdaterSyncService"
                android:exported="true" >
            <intent-filter>
                <action android:name="android.content.SyncAdapter" />
            </intent-filter>
 
            <meta-data
                    android:name="android.content.SyncAdapter"
                    android:resource="@xml/syncadapter" />
        </service>
 
    </application>
 
</manifest>
```

*res/xml/authenticator.xml*
```xml
<?xml version="1.0" encoding="utf-8"?>
<account-authenticator xmlns:android="http://schemas.android.com/apk/res/android"
                       android:accountType="@string/sync_account_type"
                       android:icon="@drawable/ic_account"
                       android:label="@string/app_name"
                       android:smallIcon="@drawable/ic_account_small" />
```

*res/xml/syncadapter.xml*
```xml
<?xml version="1.0" encoding="utf-8"?>
<sync-adapter xmlns:android="http://schemas.android.com/apk/res/android"
              android:contentAuthority="@string/content_authority"
              android:accountType="@string/sync_account_type"
              android:userVisible="false"
              android:allowParallelSyncs="false"
              android:isAlwaysSyncable="true"
              android:supportsUploading="false" />
```

* **contentAuthority** - ContentProvider
* **accountType** - AccountAuthenticator
* **userVisible** - je viditelné v systémových settings
* **allowParallelSyncs** - umožňuje synchronizovat zaráz několik uživatelských účtů
* **isAlwaysSyncable** - umožňuje vyvolání synchronizace
* **supportsUploading** - umožňuje reagování na změny v ContentProvideru. Pokud v něm dojde ke změně, a je aktivováno * supportsUploading, pak SyncAdapter obdrží v *onPerformSync()* v Bundlu příznak ContentResolver.SYNC_EXTRAS_UPLOAD, identifikující, že jde o změnu, která by se měla zpropagovat na server. Hlášení těchto změn ovlivňuje ContentProvider voláním *ContentResolver.notifyChange()* a nastavením 3 argumentu na true - pokud chceme dostat zprávu o této synchronizaci pro upload, false, pokud ji chceme ignorovat.

## Kam dál?
* [Write your own Android Sync Adapter](http://blog.udinic.com/2013/07/24/write-your-own-android-sync-adapter/)
* [Transferring Data Using Sync Adapters](https://developer.android.com/training/sync-adapters/index.html)
* [Creating a Sync Adapter](https://developer.android.com/training/sync-adapters/index.html)
* [Creating a Stub Authenticator](https://developer.android.com/training/sync-adapters/creating-authenticator.html)
* [Running a Sync Adapter](https://developer.android.com/training/sync-adapters/running-sync-adapter.html)