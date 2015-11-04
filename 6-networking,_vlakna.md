# Networking

* [Záznam z přednášky (mp3)](https://drive.google.com/file/d/0B2ZerSqwiAA-Q1VnbC0waE5pQjQ/view?usp=sharing)

### Manifest Permissions
Aby se aplikace mohla připojit k síti, je potřeba v Manifestu deklarovat povolení:
 
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
Povolení slouží k obeznámení uživatele s tím, k čemu nebo jakým prostředkům má aplikace přístup. Bez definování daného povolení ani v aplikaci požadovaný zdroj neobdržíte. Slouží tak nejen k omezení "nekalého" chování aplikací, ale mj. i v Play store definuje množinu zařízení, kde je možné aplikaci spustit. Např. pokud vyžadujete pro chod fotoaparát a dané zařízení jej nemá, aplikace se mu na Play nezpřístupní.

### StrictMode
Od API 11 (3.0 Honeycomb) je defaultně nastaven StrictMode, aby hlídal, že veškerá práce se sítí probíhá mimo hlavní vlákno aplikace. Pokud tak neučiníte, aplikace spadne s NetworkOnMainThreadException. Je to z toho důvodu, že primární úkol hlavního vlákna jsou operace s GUI. Pokud má toto vlákno např. čekat na spojení se vzdáleným serverem, nemůže obsluhovat uživatelskou interakci s UI a dochází k zásekům, případně ke smrti aplikace v podobě ANR (Application Not Responding).

## HTTP klienti
Android do api 23 obsahoval 2 implementace HTTP klienta - standardní java.net.HttpURLConnection a Apache HTTP Client v podobě v podobě AndroidHttpClient. Oba s podporou HTTPS, streamování, uploadování/stahování, nastavitelnými timeouty...

AndroidHttpClient je Googlem [doporučovaný](http://android-developers.blogspot.in/2011/09/androids-http-clients.html) používat pouze pro API 7 a 8. V ostatních verzích ho převyšuje HttpURLConnection s lepší schopností cachovat a i s menší velikostí ze strany kódu.

### OkHttp
```groovy
compile 'com.squareup.okhttp:okhttp:2.4.0'
```
[OkHttp](http://square.github.io/okhttp/) je populární knihovna od Square. Umí zpracovávat jak synchronní, tak asynchronní požadavky. Definuje 3 základní typy: Request, Response, a Call. Request je synchronní požadavek, který je následován odezvou Response. Call slouží pro asynchronní volání.

Interně tato knihovna vychází z obou implementací výše zmíněných klientů a bere s z každého to nejlepší. Výhodou je, že jako knihovna funguje na všech verzích Androidu stejně a je možné ji použít i v čisté Javě. Taktéž s ní pracují všechny networking knihovny od Square.
 
```java
private final OkHttpClient client = new OkHttpClient();
 
public void run() throws Exception {
 Request request = new Request.Builder()
     .url("http://publicobject.com/helloworld.txt")
     .build();
 
 Call call = client.newCall(request);
 Response response = call.execute();
 
 if (!response.isSuccessful()) {
   throw new IOException("Unexpected code " + response);
 }
 System.out.println(response.body().string());
}
```

Při synchronním volání Request tedy provádějící vlákno čeká na Response. Zde je tedy potřeba zajistit, aby toto provádějící vlákno, nebylo hlavním vláknem. Proto může být výhodnější asynchronní varianta:

```java
Request request = new Request.Builder()
   .url("http://publicobject.com/helloworld.txt")
   .build();
 
Call call = client.newCall(request);
call.enqueue(new Callback() {
 @Override
 public void onFailure(Request request, IOException e) {
   logger.log(Level.SEVERE, "Failed to execute " + request, e);
 }
 
 @Override
 public void onResponse(Response response) throws IOException {
   if (!response.isSuccessful()) {
     throw new IOException("Unexpected code " + response);
   }
   System.out.println(response.body().string());
 }
});
```

Asynchronní varianta s callbackem je automaticky prováděna v druhém vlákně. Pozor, metody onFailure a onResponse jsou doručeny a zpracovávány síťovým vláknem. Proto v nich nedělejte dlouhotrvající operace a z cizího vlákna též není dovolen přístup k prvkům GUI.

**Při práci s networkingovými klienty s nimi nakládejte jako se singletony.**

* Každý klient má vlastní cookies, vlastní keepAlive, vlastní spojení

## Další
### Knihovny
* [Retrofit](http://square.github.io/retrofit/) - REST API snadno
* [Picasso](http://square.github.io/picasso/) - Jednoduché stahování obrázků
* [Glide](https://github.com/bumptech/glide) - Stahování obrázků, podobné Picasso, větší velikost, RGB565

### DownloadManager
Existuje od API 9. Jedná se o systémovou službu. Globální správce stahování, který umožňuje jednoduché definování požadavku na stažení souboru, jeho zařazení do stahovací fronty, správu celého průběhu stahování, včetně nadvázání při přerušení a následnou notifikaci o dokončení stahování.

Je potřeba si zaregistrovat BroadcastReceiver, který přijímá zprávu o ACTION_DOWNLOAD_COMPLETE a pokud je obdržené ID shodné s ID stahování požadavku, který byl námi vyvolán, pak můžeme stažený soubor dále zpracovávat.

```java
public class DownloadManagerActivity extends Activity {
   private long mDownloadId;
   private DownloadManager mDownloadManager;
 
   @Override
   public void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.main);
 
       BroadcastReceiver receiver = new BroadcastReceiver() {
           @Override
           public void onReceive(Context context, Intent intent) {
               String action = intent.getAction();
               if (DownloadManager.ACTION_DOWNLOAD_COMPLETE.equals(action)) {
                   long downloadId = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, 0);
                   Query query = new Query();
                   query.setFilterById(mDownloadId);
                   Cursor c = mDownloadManager.query(query);
                   if (c.moveToFirst()) {
                       int columnIndex = c.getColumnIndex(DownloadManager.COLUMN_STATUS);
                       if (DownloadManager.STATUS_SUCCESSFUL == c.getInt(columnIndex)) {
 
                           ImageView view = (ImageView) findViewById(R.id.imageView1);
                           String uriString = c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI));
                           view.setImageURI(Uri.parse(uriString));
                       }
                   }
               }
           }
       };
 
       registerReceiver(receiver, new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE));
   }
 
   public void onClick(View view) {
       mDownloadManager = (DownloadManager) getSystemService(DOWNLOAD_SERVICE);
       Request request = new Request(Uri.parse("http://www.fi.muni.cz/images/logo_fi_cnv.png"));
       mDownloadId = mDownloadManager.enqueue(request);
 
   }
 
   public void showDownload(View view) {
       Intent i = new Intent();
       i.setAction(DownloadManager.ACTION_VIEW_DOWNLOADS);
       startActivity(i);
   }
}
```

# Vlákna
## Thread, Runnable

Stejně jako v Javě. Známe z [PV168 - Vlákna](http://kore.fi.muni.cz/wiki/index.php/PV168/Vl%C3%A1kna).
 
```java
// Thread
public class MyThread extends Thread {
 
    public MyThread() {
        super("MyThread");
    }
 
    @Override
    public void run() {
        // kód pro provedení v novém vlákně
    }
}
// spustíme nové vlákno
new MyThread().start();
```
 
```java
// Runnable
public class MyRunnable implements Runnable {
 
    @Override
    public void run() {
        // kód pro provedení v novém vlákně
    }
}
// spustíme nové vlákno
new Thread(new MyRunnable()).start();
```

## Handler
Každé vlákno má 0, nebo 1 Looper. Klasické Thread nemá Looper. Pokud ho potřebujeme, musíme zavolat Looper.prepare() a pak Looper.loop(). Případně použijeme HandlerThread. Hlavní vlákno má vždy vytvořený Looper. Je přístupný přes Looper.getMainLooper().

Každý looper má právě 1 MessageQueue. Looper.loop() neustále prochází MessageQueue a zpracovává její zprávy (pokud nějaké jsou).

Handler je třída, kde budeme řešit veškerý kód související se zpracováním zpráv (Message). Zpráva se vytváří přes obtainMessage(), odesílání přes sendMessage() a zpracování přes handleMessage(). Každý handler se při instancializaci napojuje na Looper. Buď explicitně přes konstruktor, nebo je napojen na Looper vlákna, ve kterém je instancializován. Při poslání sendMessage() bude zpráva poslána do MessageQueue asociovaného Looperu. Handler umožňuje poslat a zpracovat Message a Runnable objekty, které jsou spojeny s MessageQueue.

Pozor - sendMessage() může být provedeno z libovolného vlákna, ale její zpracování - handleMessage() bude vždy provedeno ve vlákně, ve kterém se nachází asociovaný Looper.

Jeden Looper může zaráz využívat několik různých Handlerů. Jen v hlavním vlákně existuje několik Handlerů - ActivityThread::H() , ViewRootImpl::ViewRootHandler.

Každá Message musí mít definován za cíl Handler, který ji umí zpracovat.

Procedura práce se zprávami uvnitř hlavního vlákna:

### Příklady
* Komunikace pomocí zpráv: [Communicating with the UI Thread](https://developer.android.com/training/multiple-threads/communicate-ui.html)

Chci provést dlouhotrvající operaci v jiném vlákně a na základě toho pak updatovat UI (dám uživateli vizuálně vědět, že se operace dokončila):

```java
public class TestActivity extends Activity {
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
 
        // ...
 
        // jsme v hlavním vlákně, takže Handler s ním bude asociovaný
        Handler mHandler = new Handler();
 
        // Nadefinuju si Runnable, která bude provádět dlouhou operaci v jiném vlákně
        Runnable mRunnableOnSeparateThread = new Runnable() {
            @Override
            public void run () {
 
                longOperation();
 
                // mHandler je asociovaný s hl. vláknem, takže následující kód provede na něm
                mHandler.post(new Runnable(){
                    @Override
                    public void run(){
                        // operace nad UI - změna progress baru, nastavení textu k TextView...
                    }
                });
            }
        };
 
        // spustíme mRunnableOnSeparateThread v jiném vlákně
        new Thread(mRunnableOnSeparateThread).start();
    }
 }
 ```
 
Chci implementovat autocomplete, ale nechci drtit backend hromadou požadavků:

```java
private long mLastChange = 0;
private Handler mHandler = new Handler(); // Vytvořím handler asociovaný s hl. vláknem
private Runnable mAutoCompleteRunnable = new Runnable() {
                @Override
                public void run() {
                    if (noChangeInTextInTheLastFewSeconds()) {
                        searchAndPopulateListView(chars.toString());
                    }
                }
            };
 
@Override
public void onTextChanged(final CharSequence chars,
                          int start, int before, int count) {
        mHandler.removeCallbacks(mAutoCompleteRunnable); // zruším případné volání, které ještě neproběhlo
        mHandler.postDelayed(
 
            // 1. argument - zpráva, kterou vykonám na hl. vlákně
            mAutoCompleteRunnable,
 
            // 2. argument - za kolik ms mám operaci provést
            300);
 
        mLastChange = System.currentTimeMillis();
}
 
 
private boolean noChangeInTextInTheLastFewSeconds() {
    return System.currentTimeMillis() - mLastChange >= 300;
}
```

## AsyncTask
AsyncTask je třída inspirovaná javovým [SwingWorkerem](http://kore.fi.muni.cz/wiki/index.php/PV168/Swing_3). Má řešit následující scénář:
1. Uživatel vyplní vstup - např. text do EditTextu; operace na hl. vlákně
2. Na základě vstupních dat chceme provést operaci v 2. vlákně
3. Při zpracování aktualizovat progres zpracování z 2. vlákna na hl. vlákno
4. Po skončení operace aktualizujeme UI

```java
public class MainActivity extends AppCompatActivity {
 
    private TextView mInfoView;
    private ProgressBar mProgressBar;
    private Button mStartButton;
    private Button mCancelButton;
 
    private CountingTask mCountingTask;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        //todo najdu views v layoutu
 
        mStartButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mCountingTask != null) {
                    throw new IllegalStateException("Operation is already in progress");
                }
                mCountingTask = new CountingTask();
                mCountingTask.execute();
            }
        });
 
        mCancelButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (mCountingTask == null) {
                    throw new IllegalStateException("Operation is not running");
                }
                mCountingTask.cancel(true);
            }
        });
    }
 
    private class CountingTask extends AsyncTask<Void, Integer, Integer> {
 
        @Override
        protected void onPreExecute() { // hl. vlakno
            mStartButton.setEnabled(false);
            mCancelButton.setEnabled(true);
        }
 
        @Override
        protected Integer doInBackground(Void... params) { // 2. vlakno
            int result = 0;
            for (int i = 0; i <= 100; i++) {
                if (isCancelled()) {
                    return null;
                }
                try { // simulujeme časově náročnou operaci
                    Thread.sleep(10);
                } catch (InterruptedException e) {}
                result += i;
                publishProgress(i);
            }
            return result;
        }
 
        @Override
        protected void onProgressUpdate(Integer... values) { // hl. vlakno; progres je zpracovavan v davkach
            mProgressBar.setProgress(values[0]);
        }
 
        @Override
        protected void onPostExecute(Integer integer) {
            mStartButton.setEnabled(true);
            mCancelButton.setEnabled(false);
            mCountingTask = null;
            if (isCancelled()) {
                mInfoView.setText("Task cancelled.");
            } else {
                mInfoView.setText("Result is: " + integer);
            }
        }
 
    }
}
```

### Na co si dát pozor
* Na AsyncTasku jde zavolat *execute()* právě 1x, pro další spuštění je třeba vytvořit novou instanci
* Metoda *get()* je navrácena až, když proběhla metoda *doInBackground()*. Posloupnost akcí *task.execute()* *task.get()* tedy zablokuje UI stejně, jak kdybychom nepoužívali 2. vlákno
* Od API 11 se spouštějí AsyncTasky sériově. Tzn. když spustím 2 tasky zároveň tak 2. čeká na dokončení 1. Paralelní spuštění musím vynutit explicitně. Např. pomocí [AsyncTaskCompat.executeParallel()](https://developer.android.com/reference/android/support/v4/os/AsyncTaskCompat.html).
* Při zničení Aktivity nedojde automaticky k přerušení a zastavení tasku vykonávaného v 2. vlákně

## IntentService

Více v kapitole [Služby, notifikace, BroadcastReceiver](7-sluzby,_notifikace,_broadcastreceiver.md).

## Loader

Více v kapitole [Databáze - ContentProvider, Loader, Testy](8-databaze_-_contentprovider,_loader,_testy.md)

## Kam dál?
* [Looper](http://pierrchen.blogspot.cz/2015/08/android-concurrent-programming-looper.html)
* [Handlers](http://nerds.weddingpartyapp.com/tech/2014/06/20/primer-threading-handlers-android/)
* [Nevýhody AsyncTasku](http://bon-app-etit.blogspot.cz/2013/04/the-dark-side-of-asynctask.html)
* [Úvod do Rx Javy](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)
* [Ukázky kódu v Rx Javě](https://github.com/kaushikgopal/RxJava-Android-Samples)