* [Záznam z přednášky (mp3)](https://drive.google.com/file/d/0B2ZerSqwiAA-WWM5OTFRTkZNZTQ/view?usp=sharing)

# android.content
Balík [android.content](http://developer.android.com/reference/android/content/package-summary.html) obsahuje třídy přistupující k a zveřejňující data. Je to mechanizmus Androidu, zajištující bezpečnost přístupu k datům aplikací. K datům cizí aplikace nelze přímo přistupovat, pokud to explicitně nepovolí pomocí API definovaným sadou URI. Přístup k datům je umožněn přes [ContentResolver](http://developer.android.com/reference/android/content/ContentResolver.html) a [ContentProvider](http://developer.android.com/reference/android/content/ContentProvider.html).


## ContentResolver
Přijímá požadavek, který zpracuje a předá patřičnému ContentProvideru. Aby to mohl udělat, má ContentProvider uložena mapování autorit na ContentProvidery. Tento návrh slouží k zabezpečení přístupu ke ContentProviderům jiných aplikací (neznám-li mapování na daný provider, nemůžu k němu ani přistupovat).

K datům můžeme přistupovat pouze přes ContentResolver. Ten až předá dál požadavek ContentProvideru. Mějme [URI](http://developer.android.com/guide/topics/manifest/provider-element.html):

```plain
content://com.example.project.healthcareprovider/nurses/rn
```

1. query vytvoříme voláním *getContentResolver().query(Uri, ...)*
2. Nejdříve Uri zpracovává ContentResolver, který podle schématu **content://** identifikuje, že se jedná o požadavek pro content provider
2. Dále authority - ten, kdo bude URI zpracovávat bude **com.example.project.healthcareprovider**
3. Samotnou path už zpracovává ContentProvider, který má zpracovat **nurses/rn**
4. Pokud ContentProvider danou path rozpozná, zpracuje ji, jinak by měl zareagovat vyhozením výjimky

## ContentProvider
ContentProvider definuje [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operace nad daty. Tyto operace (metody) 1:1 odpovídají odpovídají metodám ContentResolveru. Rozhraní pro přístup k datům je unifikované schválně, protože slouží jako abstrakce nad zdrojem dat, který může být cokoli, ale nejčastěji to je SQLite databáze. ContentProvider může sloužit pro přístup k datům nejen z vlastní aplikace. Např. přístup ke kontaktům na zařízení je umožněn tímto způsobem. Pokud chceme mít ContentProvider privátní, stačí u jeho deklarace v manifestu uvést:

```xml
android:exported="false"
```

### Současný multivláknový přístup
Metody měnící data v ContentProvideru **nejsou** implicitně synchronizovány. Při použití se SQLite databází se ale nemusíte o synchronizaci starat. SQLiteDatabase je synchronizována (a od api 16 nejde synchronizace ani vypnout). Proto by bylo použití synchronizace nad ContentProviderem zbytečné.

# Loader, LoaderManager
Loadery byly zavedeny v API 11 a jsou součástí CompatLibrary. Předtím byl Cursor (výsledky query na db) spravovány Aktivitou a operace nad ním běžely v hlavním vlákně. To vedlo k neresponzivním aplikacím. Loadery zajišťují že operace jím prováděné jsou asynchronní. Navíc ještě reagují na změny těchto dat, takže automaticky při změně načítají data nová. LoaderManager zase zajišťuje uchování Loaderu a jeho dat nezávisle na změnách v životním cyklu Aktivity.

## LoaderManager
Spravuje až několik Loaderů. Každá Aktivita a Fragment má přesně 1 LoaderManager, který se stará o Loadery. Inicializuje je, spravuje je a taky je ruší (např. když je Aktivita rušena).

### LoaderCallbacks
LoaderManager komunikuje stav Loaderu/ů pomocí callbacků. Operace v Loaderech jsou prováděny asynchronně, proto není možno použít blokující operace, ale reportovat stav callbacky. LoaderCallbacks implementujeme např. v Aktivitě jen 1x, callbacky dostávají informace o všech Loaderech asociovaných s daným LoaderManagerem. Loadery jsou od sebe odlišeny pomocí ID. Metoda *onCreateLoader* je zavolána, když se Loader inicializuje, *onLoadFinished* se volá vždy, když Loader načte data a je potřeba adekvátně updatovat UI. *onLoaderReset* signalizuje, že dojde ke změně dat, proto je potřeba uklidit reference na stará data. Např. kyž jsou data Cursor, se kterým by např. pracoval nějaký Adapter.

## Loader
[Loader](http://developer.android.com/reference/android/content/Loader.html) sám o sobě asynchronně v 2. vlákně operace nezpracovává, pro zjednodušení takovéto implementace je potřeba použít [AsyncTaskLoader](http://developer.android.com/reference/android/content/AsyncTaskLoader.html). Rovněž aby zvládal automaticky načítat nová data, musíme mít správně implementovánu notifikaci jeho Observeru (viz WorkTimeProvider v ukázce implementace).

Inicializace Loaderu a jeho asociace s LoaderManagerem:
```java
getLoaderManager().initLoader(0, null, this);  //initLoader (int id, Bundle args, LoaderCallbacks<D> callback)
```

# Object-relational mapping (ORM)

Mapování objektů do jejich reprezentace v databázi. Umožňuje pracovat s databází bez definování SQL příkazů. Typicky je potřeba pomocí anotací označit entitu (Java třídu) a její atributy, které chceme perzistovat a následně nám ORM nástroj pomocí Data Access Object umožní základní CRUD operace nad takto připravenou entitou. Např.:
* [OrmLite](http://ormlite.com/)
* [ActiveAndroid](https://github.com/pardom/ActiveAndroid/wiki/Creating-your-database-model)

# Testy databáze

Databáze nejde testovat klasickým jUnit testem, protože potřebujete získat instanci Contextu. Test tedy musí běžet přímo na zařízení.

**Pozor**, pokud nezměníte jméno databáze, budete testovat nad stejnou databází jakou pak bude využívat vaše aplikace.

Pro vytvoření třídy s testy je potřeba rozšířit třídu [AndroidTestCase](http://developer.android.com/reference/android/test/AndroidTestCase.html). Ta umožňuje přístup ke Contextu a Resources. Třídy s testy se umísťují do adresáře:

```plain
src/androidTest/java/<package>/
```

## Stetho
Někdy je potřeba pro ověření stavu záznamů, možnost je přímo procházet v živé databázi. Ideální je pro debug takovýchto situtuací použít knihovnu [Stetho](http://facebook.github.io/stetho/). Ta umožňuje po připojení k ADB procházet stav databáze pomocí debug nástrojů Chromu.

# Kam dál?
* [Loaders](http://developer.android.com/guide/components/loaders.html)
* [Content Providers & Content Resolvers](http://www.androiddesignpatterns.com/2012/06/content-resolvers-and-content-providers.html)
* [Understanding the LoaderManager](http://www.androiddesignpatterns.com/2012/07/understanding-loadermanager.html)
* [Implementing Loaders](http://www.androiddesignpatterns.com/2012/08/implementing-loaders.html)
* [Android SQLite database and content provider - Tutorial](http://www.vogella.com/tutorials/AndroidSQLite/article.html)


# Ukázka implementace

## package data
* datum se do databáze ukládá jako řetězec
* ID sloupec v tabulce má jméno _id
* třída Contract by měla obsahovat veškeré definice tabulek a jejich identifikaci pomocí URI

```java
package cz.droidboy.worktime.data;
 
import android.content.ContentUris;
import android.net.Uri;
import android.provider.BaseColumns;
 
import org.joda.time.DateTime;
import org.joda.time.format.DateTimeFormat;
 
public class WorkTimeContract {
 
    public static final String CONTENT_AUTHORITY = "cz.droidboy.worktime.app";
    public static final Uri BASE_CONTENT_URI = Uri.parse("content://" + CONTENT_AUTHORITY);
    public static final String PATH_WORK_TIME = "worktime";
 
    public static final String DATE_FORMAT = "yyyyMMddHHmm";
 
    /**
     * Converts Date class to a string representation, used for easy comparison and database
     * lookup.
     *
     * @param date The input date
     * @return a DB-friendly representation of the date, using the format defined in DATE_FORMAT.
     */
    public static String getDbDateString(DateTime date) {
        return date.toString(DATE_FORMAT);
    }
 
    /**
     * Converts a dateText to a long Unix time representation
     *
     * @param dateText the input date string
     * @return the Date object
     */
    public static DateTime getDateFromDb(String dateText) {
        return DateTime.parse(dateText, DateTimeFormat.forPattern(DATE_FORMAT).withOffsetParsed());
    }
 
    public static final class WorkTimeEntry implements BaseColumns {
 
        public static final Uri CONTENT_URI = BASE_CONTENT_URI.buildUpon().appendPath(PATH_WORK_TIME).build();
 
        public static final String CONTENT_TYPE = "vnd.android.cursor.dir/" + CONTENT_AUTHORITY + "/" + PATH_WORK_TIME;
        public static final String CONTENT_ITEM_TYPE = "vnd.android.cursor.item/" + CONTENT_AUTHORITY + "/" + PATH_WORK_TIME;
 
 
        public static final String TABLE_NAME = "worktime";
 
        public static final String COLUMN_START_DATE_TEXT = "start_date";
        public static final String COLUMN_END_DATE_TEXT = "end_date";
 
        public static Uri buildWorkTimeUri(long id) {
            return ContentUris.withAppendedId(CONTENT_URI, id);
        }
    }
}
```

* DbHelper se stará o vytvoření/upgrade všech tabulek databáze
* používejte konstant z Contract třídy - nespletete se pak v názvech

```java
package cz.droidboy.worktime.data;
 
import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
 
import cz.droidboy.worktime.data.WorkTimeContract.WorkTimeEntry;
 
public class WorkTimeDbHelper extends SQLiteOpenHelper {
 
    public static final String DATABASE_NAME = "worktime.db";
    private static final int DATABASE_VERSION = 1;
 
    public WorkTimeDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
 
    @Override
    public void onCreate(SQLiteDatabase db) {
        final String SQL_CREATE_LOCATION_TABLE = "CREATE TABLE " + WorkTimeEntry.TABLE_NAME + " (" +
                WorkTimeEntry._ID + " INTEGER PRIMARY KEY," +
                WorkTimeEntry.COLUMN_START_DATE_TEXT + " TEXT NOT NULL, " +
                WorkTimeEntry.COLUMN_END_DATE_TEXT + " TEXT," +
                "UNIQUE (" + WorkTimeEntry.COLUMN_START_DATE_TEXT + ", " + WorkTimeEntry.COLUMN_END_DATE_TEXT + ") ON CONFLICT REPLACE" +
                " );";
        db.execSQL(SQL_CREATE_LOCATION_TABLE);
    }
 
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + WorkTimeEntry.TABLE_NAME);
        onCreate(db);
    }
}
```

* ContentProvider přesně ověřuje Uri, které dostává, při neznámé vyhodí výjimku
* defensivní přístup je dobrý - umožní včasné odhalení chyby - např. pokud při neúspěchu zápisu dat nevyhodíme SQLException, program může pokračovat dál a zkolabovat ve chvíli, kdy skutečná příčina bude těžko odhalitelná
* ContentResolver vždy upozorníme *notifyChange()*, že na dané URI došlo ke změně dat - tak může např. Loader vědět, že má provést aktualizaci dat. 2. argument je ContentObserver, který způsobil povedení změny - v našem případě vždy null. 3. argument syncToNetwork udává, zda se má provést synchronizace dat na server (více v kapitole [Synchronizace dat](9-synchronizace_dat.md)), pokud je neuveden, je implicitně true.


```java
package cz.droidboy.worktime.data;
 
import android.content.ContentProvider;
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.UriMatcher;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.net.Uri;
import android.util.Log;
 
import java.util.Arrays;
 
public class WorkTimeProvider extends ContentProvider {
 
    private static final String TAG = WorkTimeProvider.class.getSimpleName();
 
    private static final int WORK_TIME = 100;
    private static final int WORK_TIME_ID = 101;
    private static final UriMatcher sUriMatcher = buildUriMatcher();
    private WorkTimeDbHelper mOpenHelper;
 
    private static UriMatcher buildUriMatcher() {
        final UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
        final String authority = WorkTimeContract.CONTENT_AUTHORITY;
 
        matcher.addURI(authority, WorkTimeContract.PATH_WORK_TIME, WORK_TIME);
        matcher.addURI(authority, WorkTimeContract.PATH_WORK_TIME + "/#", WORK_TIME_ID);
 
        return matcher;
    }
 
    @Override
    public boolean onCreate() {
        mOpenHelper = new WorkTimeDbHelper(getContext());
        return true;
    }
 
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Log.d(TAG, Arrays.toString(selectionArgs));
        Cursor retCursor;
        switch (sUriMatcher.match(uri)) {
            case WORK_TIME_ID: {
                retCursor = mOpenHelper.getReadableDatabase().query(
                        WorkTimeContract.WorkTimeEntry.TABLE_NAME,
                        projection,
                        WorkTimeContract.WorkTimeEntry._ID + " = '" + ContentUris.parseId(uri) + "'",
                        null,
                        null,
                        null,
                        sortOrder
                );
                break;
            }
            case WORK_TIME: {
                retCursor = mOpenHelper.getReadableDatabase().query(
                        WorkTimeContract.WorkTimeEntry.TABLE_NAME,
                        projection,
                        selection,
                        selectionArgs,
                        null,
                        null,
                        sortOrder
                );
                break;
            }
 
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
        retCursor.setNotificationUri(getContext().getContentResolver(), uri);
        return retCursor;
    }
 
    @Override
    public String getType(Uri uri) {
        final int match = sUriMatcher.match(uri);
 
        switch (match) {
            case WORK_TIME:
                return WorkTimeContract.WorkTimeEntry.CONTENT_TYPE;
            case WORK_TIME_ID:
                return WorkTimeContract.WorkTimeEntry.CONTENT_ITEM_TYPE;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
    }
 
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        Log.d(TAG, values.toString());
 
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        final int match = sUriMatcher.match(uri);
        Uri returnUri;
 
        switch (match) {
            case WORK_TIME: {
                long _id = db.insert(WorkTimeContract.WorkTimeEntry.TABLE_NAME, null, values);
                if (_id > 0)
                    returnUri = WorkTimeContract.WorkTimeEntry.buildWorkTimeUri(_id);
                else
                    throw new android.database.SQLException("Failed to insert row into " + uri);
                break;
            }
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
        getContext().getContentResolver().notifyChange(uri, null);
        return returnUri;
    }
 
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        final int match = sUriMatcher.match(uri);
        int rowsDeleted;
        switch (match) {
            case WORK_TIME:
                rowsDeleted = db.delete(WorkTimeContract.WorkTimeEntry.TABLE_NAME, selection, selectionArgs);
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
        // Because a null deletes all rows
        if (selection == null || rowsDeleted != 0) {
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return rowsDeleted;
    }
 
    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        final SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        final int match = sUriMatcher.match(uri);
        int rowsUpdated;
 
        switch (match) {
            case WORK_TIME:
                rowsUpdated = db.update(WorkTimeContract.WorkTimeEntry.TABLE_NAME, values, selection, selectionArgs);
                break;
            default:
                throw new UnsupportedOperationException("Unknown uri: " + uri);
        }
        if (rowsUpdated != 0) {
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return rowsUpdated;
    }
}
```

* pokud přistupujete k metodám ContentProvideru a nepotřebujete přímo pracovat s Cursorem, který vrací, je dobré si udělat Manager třídu, která bude wrapper nad ContentProviderem a usnadní snazší přístup k jeho metodám a získávání dat v podobě objektů
* Manager obstará jak přípravu dat do ContentValues, tak jejich validaci před vlkládáním
* Cursor je nutné po ukončení práce s ním uzavřít
* pro ochranu před SQL injections nepoužívejte raw query. Jednotlivé příkazy na databázi jsou rozloženy na část selection, kde na místo hodnot, které bychom dosazovali, definujeme pouze znak ?. Konkrétní hodnoty jsou pak do výrazu dosazeny ze selectionArgs, což je pole, které tyto hodnoty obsahuje. Tyto hodnoty nejsou vnímány jako SQL příkaz a tak je zabráněno možnému útoku.
* *Collections.emptyList()* je lepší jak null - pak je potřeba neustále ověřovat před přístupem k datům

```java
package cz.droidboy.worktime.data;
 
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
 
import org.joda.time.Interval;
import org.joda.time.LocalDate;
 
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
 
import cz.droidboy.worktime.data.WorkTimeContract.WorkTimeEntry;
import cz.droidboy.worktime.model.WorkTime;
 
public class WorkTimeManager {
 
    public static final int COL_WORK_TIME_ID = 0;
    public static final int COL_WORK_TIME_START_DATE = 1;
    public static final int COL_WORK_TIME_END_DATE = 2;
    private static final String[] WORK_TIME_COLUMNS = {
            WorkTimeEntry._ID,
            WorkTimeEntry.COLUMN_START_DATE_TEXT,
            WorkTimeEntry.COLUMN_END_DATE_TEXT
    };
 
    private static final String LOCAL_DATE_FORMAT = "yyyyMMdd";
 
    private static final String WHERE_ID = WorkTimeEntry._ID + " = ?";
    private static final String WHERE_DAY = WorkTimeEntry.COLUMN_END_DATE_TEXT + " IS NOT NULL AND substr(" + WorkTimeEntry.COLUMN_START_DATE_TEXT + ",1,8) = ? OR " + "substr(" + WorkTimeEntry.COLUMN_END_DATE_TEXT + ",1,8) = ?";
    private static final String WHERE_DATES = WorkTimeEntry.COLUMN_END_DATE_TEXT + " IS NOT NULL AND substr(" + WorkTimeEntry.COLUMN_END_DATE_TEXT + ",1,8) >= ? AND " + "substr(" + WorkTimeEntry.COLUMN_START_DATE_TEXT + ",1,8) <= ?";
 
    private Context mContext;
 
    public WorkTimeManager(Context context) {
        mContext = context.getApplicationContext();
    }
 
    //todo neukladat casy, co jsou ta stejna minuta
    public void createWorkTime(WorkTime workTime) {
        if (workTime == null) {
            throw new NullPointerException("workTime == null");
        }
        if (workTime.getId() != null) {
            throw new IllegalStateException("workTime id shouldn't be set");
        }
        if (workTime.getStartDate() == null) {
            throw new IllegalStateException("workTime startDate cannot be null");
        }
        if (workTime.getEndDate() != null && workTime.getEndDate().isBefore(workTime.getStartDate())) {
            throw new IllegalStateException("workTime endDate cannot be before startDate");
        }
 
        workTime.setId(ContentUris.parseId(mContext.getContentResolver().insert(WorkTimeEntry.CONTENT_URI, prepareWorkTimeValues(workTime))));
    }
 
    public List<WorkTime> getWorkTimesInDay(LocalDate day) {
        if (day == null) {
            throw new NullPointerException("day == null");
        }
 
        Cursor cursor = mContext.getContentResolver().query(WorkTimeEntry.CONTENT_URI, WORK_TIME_COLUMNS, WHERE_DAY, new String[]{day.toString(LOCAL_DATE_FORMAT)}, null);
        if (cursor != null && cursor.moveToFirst()) {
            List<WorkTime> workTimes = new ArrayList<>(cursor.getCount());
            try {
                while (!cursor.isAfterLast()) {
                    workTimes.add(getWorkTime(cursor));
                    cursor.moveToNext();
                }
            } finally {
                cursor.close();
            }
            return workTimes;
        }
 
        return Collections.emptyList();
    }
 
    public List<WorkTime> getWorkTimesInInterval(Interval interval) {
        if (interval == null) {
            throw new NullPointerException("interval == null");
        }
 
        Cursor cursor = mContext.getContentResolver().query(WorkTimeEntry.CONTENT_URI, WORK_TIME_COLUMNS, WHERE_DATES, new String[]{interval.getStart().toString(LOCAL_DATE_FORMAT), interval.getEnd().toString(LOCAL_DATE_FORMAT)}, null);
 
        if (cursor != null && cursor.moveToFirst()) {
            List<WorkTime> workTimes = new ArrayList<>(cursor.getCount());
            try {
                while (!cursor.isAfterLast()) {
                    workTimes.add(getWorkTime(cursor));
                    cursor.moveToNext();
                }
            } finally {
                cursor.close();
            }
            return workTimes;
        }
 
        return Collections.emptyList();
    }
 
    public void updateWorkTime(WorkTime workTime) {
        if (workTime == null) {
            throw new NullPointerException("workTime == null");
        }
        if (workTime.getId() == null) {
            throw new IllegalStateException("workTime id cannot be null");
        }
        if (workTime.getStartDate() == null) {
            throw new IllegalStateException("workTime startDate cannot be null");
        }
        if (workTime.getEndDate() != null && workTime.getEndDate().isBefore(workTime.getStartDate())) {
            throw new IllegalStateException("workTime endDate cannot be before startDate");
        }
 
        mContext.getContentResolver().update(WorkTimeEntry.CONTENT_URI, prepareWorkTimeValues(workTime), WHERE_ID, new String[]{String.valueOf(workTime.getId())});
    }
 
    public void deleteWorkTime(WorkTime workTime) {
        if (workTime == null) {
            throw new NullPointerException("workTime == null");
        }
        if (workTime.getId() == null) {
            throw new IllegalStateException("workTime id cannot be null");
        }
 
        mContext.getContentResolver().delete(WorkTimeEntry.CONTENT_URI, WHERE_ID, new String[]{String.valueOf(workTime.getId())});
    }
 
    private ContentValues prepareWorkTimeValues(WorkTime workTime) {
        ContentValues values = new ContentValues();
        values.put(WorkTimeEntry.COLUMN_START_DATE_TEXT, WorkTimeContract.getDbDateString(workTime.getStartDate()));
        values.put(WorkTimeEntry.COLUMN_END_DATE_TEXT, workTime.getEndDate() != null ? WorkTimeContract.getDbDateString(workTime.getEndDate()) : null);
        return values;
    }
 
    private WorkTime getWorkTime(Cursor cursor) {
        WorkTime workTime = new WorkTime();
        workTime.setId(cursor.getLong(COL_WORK_TIME_ID));
        workTime.setStartDate(WorkTimeContract.getDateFromDb(cursor.getString(COL_WORK_TIME_START_DATE)));
        String endDate = cursor.getString(COL_WORK_TIME_END_DATE);
        if (endDate != null) {
            workTime.setEndDate(WorkTimeContract.getDateFromDb(endDate));
        }
        return workTime;
    }
}
```

## package model

```java
package cz.droidboy.worktime.model;
 
import org.joda.time.DateTime;
 
public final class WorkTime {
 
    private Long id;
    private DateTime startDate;
    private DateTime endDate;
 
    public Long getId() {
        return id;
    }
 
    public void setId(Long id) {
        this.id = id;
    }
 
    public DateTime getStartDate() {
        return startDate;
    }
 
    public void setStartDate(DateTime startDate) {
        this.startDate = startDate;
    }
 
    public DateTime getEndDate() {
        return endDate;
    }
 
    public void setEndDate(DateTime endDate) {
        this.endDate = endDate;
    }
 
    ...
}
```

## test

```java
package cz.droidboy.worktime;
 
import android.test.AndroidTestCase;
import android.util.Log;
 
import org.joda.time.DateTime;
import org.joda.time.Interval;
import org.joda.time.LocalDate;
 
import java.util.ArrayList;
import java.util.List;
 
import cz.droidboy.worktime.data.WorkTimeContract.WorkTimeEntry;
import cz.droidboy.worktime.data.WorkTimeManager;
import cz.droidboy.worktime.model.WorkTime;
 
public class TestWorkTimeManager extends AndroidTestCase {
 
    private static final String TAG = TestWorkTimeManager.class.getSimpleName();
 
    private WorkTimeManager mManager;
 
    @Override
    protected void setUp() throws Exception {
        mManager = new WorkTimeManager(mContext);
    }
 
    @Override
    public void tearDown() throws Exception {
        mContext.getContentResolver().delete(
                WorkTimeEntry.CONTENT_URI,
                null,
                null
        );
    }
 
    public void testGetWorkTimesInDay() throws Exception {
        List<WorkTime> expectedWorkTimes = new ArrayList<>(2);
        WorkTime workTime1 = createWorkTime(new DateTime(2015, 5, 26, 1, 0), new DateTime(2015, 5, 26, 5, 0));
        WorkTime workTime2 = createWorkTime(new DateTime(2015, 5, 26, 6, 0), new DateTime(2015, 5, 26, 7, 0));
        expectedWorkTimes.add(workTime1);
        expectedWorkTimes.add(workTime2);
 
        mManager.createWorkTime(workTime1);
        mManager.createWorkTime(createWorkTime(new DateTime(2015, 5, 27, 1, 0), new DateTime(2015, 5, 27, 5, 0)));
        mManager.createWorkTime(workTime2);
 
        List<WorkTime> workTimes = mManager.getWorkTimesInDay(new LocalDate(2015, 5, 26));
        Log.d(TAG, workTimes.toString());
        assertTrue(workTimes.size() == 2);
        assertEquals(expectedWorkTimes, workTimes);
    }
 
    public void testGetWorkTimesInInterval() throws Exception {
        List<WorkTime> expectedWorkTimes = new ArrayList<>(2);
        WorkTime workTime1 = createWorkTime(new DateTime(2015, 5, 26, 1, 0), new DateTime(2015, 5, 26, 5, 0));
        WorkTime workTime2 = createWorkTime(new DateTime(2015, 5, 26, 6, 0), new DateTime(2015, 5, 26, 7, 0));
        WorkTime workTime3 = createWorkTime(new DateTime(2015, 5, 27, 1, 0), new DateTime(2015, 5, 27, 5, 0));
        expectedWorkTimes.add(workTime1);
        expectedWorkTimes.add(workTime2);
        expectedWorkTimes.add(workTime3);
 
        mManager.createWorkTime(createWorkTime(new DateTime(2015, 5, 25, 2, 0), new DateTime(2015, 5, 25, 5, 0)));
        mManager.createWorkTime(workTime1);
        mManager.createWorkTime(workTime2);
        mManager.createWorkTime(workTime3);
        mManager.createWorkTime(createWorkTime(new DateTime(2015, 5, 28, 1, 0), new DateTime(2015, 5, 28, 5, 0)));
 
        List<WorkTime> workTimes = mManager.getWorkTimesInInterval(new Interval(new DateTime(2015, 5, 26, 0, 0), new DateTime(2015, 5, 27, 0, 0)));
        Log.d(TAG, workTimes.toString());
        assertTrue(workTimes.size() == 3);
        assertEquals(expectedWorkTimes, workTimes);
    }
 
    private WorkTime createWorkTime(DateTime startDate, DateTime endDate) {
        WorkTime workTime = new WorkTime();
        workTime.setStartDate(startDate);
        workTime.setEndDate(endDate);
        return workTime;
    }
}
```