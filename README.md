# android-sample02

## データベース接続

AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

+    <uses-permission android:name="android.permission.INTERNET"/>

    <application

    </application>

</manifest>
```

build.gradle

```gradle
dependencies {

    implementation libs.appcompat
    implementation libs.material
    implementation libs.activity
    implementation libs.constraintlayout
    testImplementation libs.junit
    androidTestImplementation libs.ext.junit
    androidTestImplementation libs.espresso.core

+    implementation 'org.mariadb.jdbc:mariadb-java-client:2.2.0'
+    implementation 'mysql:mysql-connector-java:5.1.13'

}
```

DatabaseTask.java

```java
package com.example.myapplication;

import android.os.AsyncTask;
import android.util.Log;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DatabaseTask extends AsyncTask<Void, Void, Void> {

    @Override
    protected Void doInBackground(Void... voids) {
        // String URL = "jdbc:mariadb://10.0.2.2:3306/dokoQL";
        String URL = "jdbc:mysql://10.0.2.2:3306/dokoQL";
        String user = "root";
        String password = "trustno1";

        try {
            // Class.forName("org.mariadb.jdbc.Driver");
            Class.forName("com.mysql.jdbc.Driver");
            Connection conn = DriverManager.getConnection(URL, user, password);

            // 接続に成功したら、クエリを実行する
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT * FROM 家計簿");

            // 結果の処理
            while (rs.next()) {
                String column1 = rs.getString("メモ");
                // 取得したデータを処理する
                Log.i("MainActivity", column1);
            }

            rs.close();
            stmt.close();
            conn.close();

        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    protected void onPostExecute(Void aVoid) {
        super.onPostExecute(aVoid);
        // バックグラウンド処理が完了した後に実行される
        // 必要であればUIの更新などを行う
    }
}
```

MainActivity.java

```java
package com.example.myapplication;

import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        Button btn = (Button)findViewById(R.id.button);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // DatabaseTaskを実行
+                new DatabaseTask().execute();
            }
        });
    }
}
```
