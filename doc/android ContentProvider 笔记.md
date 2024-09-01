学习android的contentprovider。笔记记录于此。

contentprovider作用是将数据共享给其他的应用。

#### 参考链接

https://www.tutorialspoint.com/android/android_content_providers.htm

http://www.cnblogs.com/linjiqin/archive/2011/05/28/2061396.html

http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/0821/367.html

http://blog.csdn.net/luoshengyang/article/details/6950440


#### 实现contentProvider

创建一个Android studio project。app名称MyApplication。

* 继承ContentProvider

* 其中contentprovider的onCreate()函数是创建的时候调用。

insert(),query(),delete(),update()分别是其中的函数。

* UriMatcher用于设置匹配规则。

Uri的结构是content://authority/path

content是固定的。

authority可以自己设置。

path用来表示要操作的数据。

* 内部实现了一个SQLiteOpenHelper对象。用于存储数据。

文件名StudentsProvider.java

```
package com.example.myapplication;

import java.util.HashMap;

import android.content.ContentProvider;
import android.content.ContentUris;
import android.content.ContentValues;
import android.content.Context;
import android.content.UriMatcher;

import android.database.Cursor;
import android.database.SQLException;

import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import android.database.sqlite.SQLiteQueryBuilder;

import android.net.Uri;
import android.text.TextUtils;
import android.util.Log;

/*
 * 系统实例化所有的ContentProvider对象；你从来不需要自己做。
 * 事实上，你从来不会直接处理ContentProvider对象。
 * 通常，对于每个类型的ContentProvider只有一个简单的实例。
 */
public class StudentsProvider extends ContentProvider {
    static final String PROVIDER_NAME = "com.example.myapplication.StudentsProvider";
    static final String URL = "content://" + PROVIDER_NAME + "/students";
    static final Uri CONTENT_URI = Uri.parse(URL);

    static final String _ID = "_id";
    static final String NAME = "name";
    static final String GRADE = "grade";

    private static HashMap<String, String> STUDENTS_PROJECTION_MAP;

    static final int STUDENTS = 1;
    static final int STUDENT_ID = 2;

    static final UriMatcher uriMatcher;
    // 设置匹配规则
    static{
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        // 第一个参数：authority,第二个参数：path
        // 第三个参数:code, 匹配成功之后的返回值
        /* 结合authority和path。下面设置的匹配规则
            com.example.myapplication.StudentsProvider/students/
            表示匹配students这个项，而students包含了很多的id。
            匹配的结果是一个集合
         */
        uriMatcher.addURI(PROVIDER_NAME, "students", STUDENTS);
        // com.example.myapplication.StudentsProvider/students/1
        // student/后面的数字表示一个id.也可以是其他数字
        // 匹配的结果是特定的某一个实例
        uriMatcher.addURI(PROVIDER_NAME, "students/#", STUDENT_ID);
    }

    /**
     * Database specific constant declarations
     */

    private SQLiteDatabase db;
    static final String DATABASE_NAME = "College";
    static final String STUDENTS_TABLE_NAME = "students";
    static final int DATABASE_VERSION = 1;
    static final String CREATE_DB_TABLE =
            " CREATE TABLE " + STUDENTS_TABLE_NAME +
                    " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    " name TEXT NOT NULL, " +
                    " grade TEXT NOT NULL);";

    /**
     * Helper class that actually creates and manages
     * the provider's underlying data repository.
     */
    // 创建数据库类,用于保存数据
    private static class DatabaseHelper extends SQLiteOpenHelper {
        // 构造方法, 一般传入需要创建的数据库名称。
        DatabaseHelper(Context context){
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }
        // 创建数据库时调用
        @Override
        public void onCreate(SQLiteDatabase db) {
            // 初始化数据表结构，执行一条建表语句
            db.execSQL(CREATE_DB_TABLE);
        }
        // 数据库版本更新时调用
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            db.execSQL("DROP TABLE IF EXISTS " +  STUDENTS_TABLE_NAME);
            onCreate(db);
        }
    }

    @Override
    public boolean onCreate() {
        Context context = getContext();
        // 创建数据库对象
        DatabaseHelper dbHelper = new DatabaseHelper(context);

        /**
         * Create a write able database which will trigger its
         * creation if it doesn't already exist.
         */
        // 获取一个可读可写的数据库
        db = dbHelper.getWritableDatabase();
        return (db == null)? false:true;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        /**
         * Add a new student record
         */
        // 数据库插入
        long rowID = db.insert(	STUDENTS_TABLE_NAME, "", values);

        /**
         * If record is added successfully
         */
        if (rowID > 0) {
            Uri _uri = ContentUris.withAppendedId(CONTENT_URI, rowID);
            // 通知注册在此URI上的访问者，监听contentprovider的变化
            getContext().getContentResolver().notifyChange(_uri, null);
            return _uri;
        }

        throw new SQLException("Failed to add a record into " + uri);
    }

    @Override
    public Cursor query(Uri uri, String[] projection,
                        String selection,String[] selectionArgs, String sortOrder) {
        SQLiteQueryBuilder qb = new SQLiteQueryBuilder();
        qb.setTables(STUDENTS_TABLE_NAME);
        // 根据不同的匹配规则进行处理
        Log.e("TEST", "query");
        switch (uriMatcher.match(uri)) {
            // 匹配某一类，返回的是一个集合
            case STUDENTS:
                qb.setProjectionMap(STUDENTS_PROJECTION_MAP);
                Log.e("TEST", "STUDENTS");
                break;
            // 匹配某一个，返回的是一个特定的实例
            case STUDENT_ID:
                qb.appendWhere( _ID + "=" + uri.getPathSegments().get(1));
                Log.e("TEST", "STUDENTS_ID");
                break;

            default:
        }

        if (sortOrder == null || sortOrder == ""){
            /**
             * By default sort on student names
             */
            sortOrder = NAME;
        }

        Cursor c = qb.query(db,	projection,	selection,
                selectionArgs,null, null, sortOrder);
        /**
         * register to watch a content URI for changes
         */
        /*
         * Cursor类的setNotificationUri函数来把当前上下文的ContentResolver对象保存到Curosr里面去，
         * 以便当通过这个Cursor来改变数据库中的数据时，可以通知相应的ContentObserver来处理。
         */
        c.setNotificationUri(getContext().getContentResolver(), uri);
        return c;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        int count = 0;
        switch (uriMatcher.match(uri)){
            case STUDENTS:
                count = db.delete(STUDENTS_TABLE_NAME, selection, selectionArgs);
                break;

            case STUDENT_ID:
                String id = uri.getPathSegments().get(1);
                count = db.delete( STUDENTS_TABLE_NAME, _ID +  " = " + id +
                                (!TextUtils.isEmpty(selection) ?
                                " AND (" + selection + ')' : ""), selectionArgs);
                break;
            default:
                throw new IllegalArgumentException("Unknown URI " + uri);
        }
        // 通知那些注册了监控特定URI的ContentObserver对象，使得它们可以相应地执行一些处理，例如更新数据在界面上的显示。
        getContext().getContentResolver().notifyChange(uri, null);
        return count;
    }

    @Override
    public int update(Uri uri, ContentValues values,
                      String selection, String[] selectionArgs) {
        int count = 0;
        switch (uriMatcher.match(uri)) {
            case STUDENTS:
                count = db.update(STUDENTS_TABLE_NAME, values, selection, selectionArgs);
                break;

            case STUDENT_ID:
                count = db.update(STUDENTS_TABLE_NAME, values,
                        _ID + " = " + uri.getPathSegments().get(1) +
                                (!TextUtils.isEmpty(selection) ?
                                 " AND (" +selection + ')' : ""), selectionArgs);
                break;
            default:
                throw new IllegalArgumentException("Unknown URI " + uri );
        }

        getContext().getContentResolver().notifyChange(uri, null);
        return count;
    }
    //  获取MIME类型
    @Override
    public String getType(Uri uri) {
        switch (uriMatcher.match(uri)){
            /**
             * Get all student records
             */
            //访问一个集合
            case STUDENTS:
                return "vnd.android.cursor.dir/vnd.example.students";
            /**
             * Get a particular student
             */
            //访问单个资源
            case STUDENT_ID:
                return "vnd.android.cursor.item/vnd.example.students";
            default:
                throw new IllegalArgumentException("Unsupported URI: " + uri);
        }
    }
}
```

#### 注册contentprovider

使用<provider .../>标签注册在AndroidManifest.xml中.

* name指定ContentProvider的类名。

* authorities 用于唯一标示特定的ContentProvider。一般使用ContentProvider的package来命名。

AndroidManifest.xml 文件如下：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapplication">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <provider android:name="StudentsProvider"
            android:authorities="com.example.myapplication.StudentsProvider"/>
    </application>

</manifest>
```

#### 布局文件

添加2个EditText用于输入信息。添加2个按键用于添加和获取信息。

activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical" >

<EditText
    android:id="@+id/editTextName"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="name"/>
<EditText
    android:id="@+id/editTextGrade"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="grade"/>
<Button
    android:id="@+id/buttonAddName"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Add Name"
    android:onClick="onClickAddName"/>
<Button
    android:id="@+id/buttonRetrieveStudents"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Retrive student"
    android:onClick="onClickRetrieveStudents"/>
</LinearLayout>
```

效果如下

![](http://images2015.cnblogs.com/blog/745188/201703/745188-20170308103101703-216321649.png)

#### 实现activity

* 外部的程序通过ContentResolver提供的接口,访问ContentProvider的数据。

* 可以通过getContentResolver()获取一个ContentResolver对象。

* 在进行匹配时:

如果Uri="content://com.example.myapplication.StudentsProvider/students",将会匹配STUDENT。

这里使用了ContentUris.withAppendedId(students, 1)，在Uri后面添加ID。

如果Uri="content://com.example.myapplication.StudentsProvider/students/1",将会匹配STUDENT_ID。

MainActivity.java

```
package com.example.myapplication;

import android.content.ContentUris;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.net.Uri;
import android.os.Bundle;
import android.app.Activity;

import android.content.ContentValues;
import android.content.CursorLoader;

import android.database.Cursor;

import android.view.Menu;
import android.view.View;

import android.widget.EditText;
import android.widget.Toast;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    public void onClickAddName(View view) {
        // Add a new student record
        ContentValues values = new ContentValues();
        values.put(StudentsProvider.NAME,
                ((EditText)findViewById(R.id.editTextName)).getText().toString());

        values.put(StudentsProvider.GRADE,
                ((EditText)findViewById(R.id.editTextGrade)).getText().toString());
        /*
         *  通过getContentResolver()获取一个ContentResolver的对象。
         *  再通过contentResolver类进行数据插入
         */
        Uri uri = getContentResolver().insert(
                StudentsProvider.CONTENT_URI, values);

        Toast.makeText(getBaseContext(),
                uri.toString(), Toast.LENGTH_LONG).show();
    }
    public void onClickRetrieveStudents(View view) {
        // Retrieve student records
        String URL = "content://com.example.myapplication.StudentsProvider/students";
        // 将String转为URi对象
        // Uri:"content://com.example.myapplication.StudentsProvider/students";
        Uri students = Uri.parse(URL);
        // 在URI后面添加一个ID
        // 如果这行注释了，那么匹配students表里面的所有项
        // Uri:"content://com.example.myapplication.StudentsProvider/students/1";
        //students = ContentUris.withAppendedId(students, 1);
        /*
         *  数据查询
         *  返回数据库游标接口Cursor。
         */
		 // 下面2个函数都可以使用，二选一。
//        Cursor c = managedQuery(students, null, null, null, null);
        Cursor c = getContentResolver().query(students, null, null, null, null);

        if (c.moveToFirst()) {
            do{
                Toast.makeText(this,
                        c.getString(c.getColumnIndex(StudentsProvider._ID)) +
                                ", " +  c.getString(c.getColumnIndex( StudentsProvider.NAME)) +
                                ", " + c.getString(c.getColumnIndex( StudentsProvider.GRADE)),
                        Toast.LENGTH_SHORT).show();
            } while (c.moveToNext());
        }
    }
}
```

Tony Liu

2017-3-8, Shenzhen