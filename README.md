
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```
/*
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: DHINESH R
Registeration Number : 212223220019
*/
```

### activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Employee Management"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_gravity="center"
        android:layout_marginBottom="16dp"/>

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Employee Name"
        android:inputType="textPersonName"/>

    <EditText
        android:id="@+id/etDepartment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Department"
        android:inputType="text"/>

    <EditText
        android:id="@+id/etSalary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Salary"
        android:inputType="numberDecimal"/>

    <EditText
        android:id="@+id/etAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Age"
        android:inputType="number"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginTop="12dp">

        <Button
            android:id="@+id/btnSave"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Save"/>

        <Button
            android:id="@+id/btnUpdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Update"/>

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Delete"/>
    </LinearLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_marginTop="16dp"/>

</LinearLayout>
```

### MainActivity.java
```
package com.example.employeedetails;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etName, etDepartment, etSalary, etAge;
    Button btnSave, btnUpdate, btnDelete;
    RecyclerView recyclerView;

    ArrayList<Employee> employeeList;
    EmployeeAdapter adapter;

    DatabaseReference databaseReference;

    String selectedId = "";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etDepartment = findViewById(R.id.etDepartment);
        etSalary = findViewById(R.id.etSalary);
        etAge = findViewById(R.id.etAge);

        btnSave = findViewById(R.id.btnSave);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);

        recyclerView = findViewById(R.id.recyclerView);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        try {
            databaseReference = FirebaseDatabase.getInstance().getReference("Employees");
        } catch (Exception e) {
            Toast.makeText(this, "Connection Error: " + e.getMessage(), Toast.LENGTH_LONG).show();
        }

        employeeList = new ArrayList<>();

        adapter = new EmployeeAdapter(employeeList, employee -> {
            selectedId = employee.getId();

            etName.setText(employee.getName());
            etDepartment.setText(employee.getDepartment());
            etSalary.setText(employee.getSalary());
            etAge.setText(employee.getAge());
        });

        recyclerView.setAdapter(adapter);

        loadEmployees();

        btnSave.setOnClickListener(v -> saveEmployee());

        btnUpdate.setOnClickListener(v -> updateEmployee());

        btnDelete.setOnClickListener(v -> deleteEmployee());
    }

    private void saveEmployee() {
        if (databaseReference == null) {
            Toast.makeText(this, "Database not available", Toast.LENGTH_SHORT).show();
            return;
        }

        String id = databaseReference.push().getKey();

        if (id == null) {
            Toast.makeText(this, "Error generating ID", Toast.LENGTH_SHORT).show();
            return;
        }

        Employee employee = new Employee(
                id,
                etName.getText().toString().trim(),
                etDepartment.getText().toString().trim(),
                etSalary.getText().toString().trim(),
                etAge.getText().toString().trim()
        );

        databaseReference.child(id).setValue(employee);

        Toast.makeText(this, "Employee Saved", Toast.LENGTH_SHORT).show();

        clearFields();
    }

    private void loadEmployees() {
        if (databaseReference == null) return;

        databaseReference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {

                employeeList.clear();

                for (DataSnapshot ds : snapshot.getChildren()) {

                    Employee employee = ds.getValue(Employee.class);

                    if (employee != null) {
                        employeeList.add(employee);
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {

                Toast.makeText(MainActivity.this,
                        error.getMessage(),
                        Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void updateEmployee() {
        if (databaseReference == null) {
            Toast.makeText(this, "Database not available", Toast.LENGTH_SHORT).show();
            return;
        }

        if (selectedId.isEmpty()) {

            Toast.makeText(this,
                    "Please select an employee",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        Employee employee = new Employee(
                selectedId,
                etName.getText().toString().trim(),
                etDepartment.getText().toString().trim(),
                etSalary.getText().toString().trim(),
                etAge.getText().toString().trim()
        );

        databaseReference.child(selectedId).setValue(employee);

        Toast.makeText(this,
                "Employee Updated",
                Toast.LENGTH_SHORT).show();

        clearFields();
    }

    private void deleteEmployee() {
        if (databaseReference == null) {
            Toast.makeText(this, "Database not available", Toast.LENGTH_SHORT).show();
            return;
        }

        if (selectedId.isEmpty()) {

            Toast.makeText(this,
                    "Please select an employee",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        databaseReference.child(selectedId).removeValue();

        Toast.makeText(this,
                "Employee Deleted",
                Toast.LENGTH_SHORT).show();

        clearFields();
    }

    private void clearFields() {

        etName.setText("");
        etDepartment.setText("");
        etSalary.setText("");
        etAge.setText("");

        selectedId = "";
    }
}
```

### AndroidManifest.xml
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.EmployeeDetails">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### build.gradle.kts(:app)
```
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.google.services)
}

android {
    namespace = "com.example.employeedetails"
    compileSdk {
        version = release(37)
    }

    defaultConfig {
        applicationId = "com.example.employeedetails"
        minSdk = 24
        targetSdk = 37
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            optimization {
                enable = false
            }
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
}

dependencies {
    implementation(platform(libs.firebase.bom))
    implementation(libs.firebase.database)
    implementation(libs.activity.ktx)
    implementation(libs.appcompat)
    implementation(libs.constraintlayout)
    implementation(libs.material)
    implementation(libs.cardview)
    testImplementation(libs.junit)
    androidTestImplementation(libs.espresso.core)
    androidTestImplementation(libs.ext.junit)
}
```

### build.gradle.kts(EmployeeDetails)
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.google.services) apply false
}
```
## OUTPUT
<img width="196" height="217" alt="image" src="https://github.com/user-attachments/assets/c22eb991-b1d2-4e1b-bc74-c3130980f8e8" />
<img width="1917" height="895" alt="image" src="https://github.com/user-attachments/assets/70f583df-fad3-49f0-bffb-c428704a6ab5" />




## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
