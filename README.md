name: Build ALFARUQ DATA SERVICE

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Create Android project
        shell: bash
        run: |
          mkdir -p app/src/main/java/com/alfaruq/dataservice
          mkdir -p app/src/main/res/values
          mkdir -p app/src/main/res/drawable
          mkdir -p app/src/main/res/mipmap-hdpi
          mkdir -p app/src/main/res/mipmap-mdpi
          mkdir -p app/src/main/res/mipmap-xhdpi
          mkdir -p app/src/main/res/mipmap-xxhdpi
          mkdir -p app/src/main/res/mipmap-xxxhdpi

          cat > settings.gradle <<'EOF'
          pluginManagement {
              repositories {
                  google()
                  mavenCentral()
                  gradlePluginPortal()
              }
          }

          dependencyResolutionManagement {
              repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
              repositories {
                  google()
                  mavenCentral()
              }
          }

          rootProject.name = "ALFARUQ-DATA-SERVICE"
          include(":app")
          EOF

          cat > build.gradle <<'EOF'
          plugins {
              id 'com.android.application' version '8.7.3' apply false
          }
          EOF

          cat > app/build.gradle <<'EOF'
          plugins {
              id 'com.android.application'
          }

          android {
              namespace 'com.alfaruq.dataservice'
              compileSdk 35

              defaultConfig {
                  applicationId "com.alfaruq.dataservice"
                  minSdk 23
                  targetSdk 35
                  versionCode 2
                  versionName "2.0"
              }
          }
          EOF

          cat > app/src/main/res/values/strings.xml <<'EOF'
          <resources>
              <string name="app_name">ALFARUQ DATA SERVICE</string>
          </resources>
          EOF

          cat > app/src/main/res/values/colors.xml <<'EOF'
          <resources>
              <color name="green">#087D5B</color>
              <color name="dark_green">#064D3B</color>
              <color name="gold">#D9A441</color>
              <color name="white">#FFFFFF</color>
              <color name="background">#F2F6F4</color>
              <color name="text">#14231D</color>
              <color name="muted">#71807A</color>
          </resources>
          EOF

          cat > app/src/main/res/values/themes.xml <<'EOF'
          <resources>
              <style name="AppTheme" parent="android:style/Theme.Material.Light.NoActionBar">
                  <item name="android:fontFamily">sans</item>
                  <item name="android:colorAccent">@color/green</item>
                  <item name="android:statusBarColor">@color/dark_green</item>
                  <item name="android:navigationBarColor">@color/white</item>
                  <item name="android:windowLightStatusBar">false</item>
              </style>
          </resources>
          EOF

          cat > app/src/main/res/drawable/logo.xml <<'EOF'
          <vector xmlns:android="http://schemas.android.com/apk/res/android"
              android:width="100dp"
              android:height="100dp"
              android:viewportWidth="100"
              android:viewportHeight="100">

              <path
                  android:fillColor="#087D5B"
                  android:pathData="M50,4 A46,46 0,1 0,50 96 A46,46 0,1 0,50 4" />

              <path
                  android:fillColor="#D9A441"
                  android:pathData="M29,30 L29,70 L38,70 L38,54 L62,70 L71,70 L71,30 L62,30 L62,55 L38,39 L38,30 Z" />

              <path
                  android:fillColor="#FFFFFF"
                  android:pathData="M43,24 L57,24 L57,33 L43,33 Z" />

          </vector>
          EOF

          cat > app/src/main/AndroidManifest.xml <<'EOF'
          <manifest xmlns:android="http://schemas.android.com/apk/res/android">

              <uses-permission android:name="android.permission.INTERNET" />

              <application
                  android:allowBackup="false"
                  android:icon="@drawable/logo"
                  android:label="@string/app_name"
                  android:supportsRtl="true"
                  android:theme="@style/AppTheme">

                  <activity
                      android:name=".MainActivity"
                      android:screenOrientation="portrait"
                      android:exported="true">

                      <intent-filter>
                          <action android:name="android.intent.action.MAIN" />
                          <category android:name="android.intent.category.LAUNCHER" />
                      </intent-filter>

                  </activity>

              </application>

          </manifest>
          EOF

          cat > app/src/main/java/com/alfaruq/dataservice/MainActivity.java <<'EOF'
          package com.alfaruq.dataservice;

          import android.app.Activity;
          import android.os.Bundle;
          import android.graphics.Color;
          import android.graphics.Typeface;
          import android.view.Gravity;
          import android.view.View;
          import android.widget.Button;
          import android.widget.EditText;
          import android.widget.ImageView;
          import android.widget.LinearLayout;
          import android.widget.ScrollView;
          import android.widget.TextView;
          import android.widget.Toast;

          import org.json.JSONObject;

          import java.io.BufferedReader;
          import java.io.InputStream;
          import java.io.InputStreamReader;
          import java.io.OutputStream;
          import java.net.HttpURLConnection;
          import java.net.URL;

          public class MainActivity extends Activity {

              private static final String SUPABASE_URL =
                      "https://vcexczhmvtawldjgldob.supabase.co";

              private static final String SUPABASE_KEY =
                      "sb_publishable_Pa0ZBzzEP0VdKLx8sDGafQ_BVKGoXK8";

              private LinearLayout root;
              private EditText emailInput;
              private EditText passwordInput;
              private EditText nameInput;
              private EditText phoneInput;
              private boolean registerMode = false;
              private String accessToken = "";

              @Override
              protected void onCreate(Bundle savedInstanceState) {
                  super.onCreate(savedInstanceState);
                  showLogin();
              }

              private TextView text(String value, int size) {
                  TextView t = new TextView(this);
                  t.setText(value);
                  t.setTextSize(size);
                  t.setTextColor(Color.rgb(20,35,29));
                  t.setPadding(0, 8, 0, 8);
                  return t;
              }

              private Button button(String value) {
                  Button b = new Button(this);
                  b.setText(value);
                  b.setTextColor(Color.WHITE);
                  b.setTextSize(15);
                  b.setBackgroundColor(Color.rgb(8,125,91));
                  return b;
              }

              private void base() {
                  ScrollView scroll = new ScrollView(this);
                  root = new LinearLayout(this);
                  root.setOrientation(LinearLayout.VERTICAL);
                  root.setPadding(28, 28, 28, 28);
                  root.setBackgroundColor(Color.rgb(242,246,244));
                  scroll.addView(root);
                  setContentView(scroll);
              }

              private void logo() {
                  ImageView image = new ImageView(this);
                  image.setImageResource(com.alfaruq.dataservice.R.drawable.logo);
                  image.setLayoutParams(new LinearLayout.LayoutParams(
                          120, 120
                  ));
                  image.setPadding(10,10,10,10);

                  LinearLayout box = new LinearLayout(this);
                  box.setGravity(Gravity.CENTER);
                  box.addView(image);

                  root.addView(box);
              }

              private void showLogin() {
                  registerMode = false;
                  base();
                  logo();

                  TextView title = text("ALFARUQ DATA SERVICE", 25);
                  title.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
                  title.setGravity(Gravity.CENTER);
                  root.addView(title);

                  TextView sub = text("Login to your account", 15);
                  sub.setGravity(Gravity.CENTER);
                  root.addView(sub);

                  emailInput = new EditText(this);
                  emailInput.setHint("Email");
                  emailInput.setInputType(33);
                  root.addView(emailInput);

                  passwordInput = new EditText(this);
                  passwordInput.setHint("Password");
                  passwordInput.setInputType(129);
                  root.addView(passwordInput);

                  Button login = button("LOGIN");
                  root.addView(login);

                  Button register = new Button(this);
                  register.setText("Create a new account");
                  root.addView(register);

                  login.setOnClickListener(v -> login());
                  register.setOnClickListener(v -> showRegister());
              }

              private void showRegister() {
                  registerMode = true;
                  base();
                  logo();

                  TextView title = text("Create Account", 25);
                  title.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
                  title.setGravity(Gravity.CENTER);
                  root.addView(title);

                  nameInput = new EditText(this);
                  nameInput.setHint("Full name");
                  root.addView(nameInput);

                  phoneInput = new EditText(this);
                  phoneInput.setHint("Phone number");
                  phoneInput.setInputType(3);
                  root.addView(phoneInput);

                  emailInput = new EditText(this);
                  emailInput.setHint("Email");
                  emailInput.setInputType(33);
                  root.addView(emailInput);

                  passwordInput = new EditText(this);
                  passwordInput.setHint("Password");
                  passwordInput.setInputType(129);
                  root.addView(passwordInput);

                  Button create = button("CREATE ACCOUNT");
                  root.addView(create);

                  Button back = new Button(this);
                  back.setText("Back to Login");
                  root.addView(back);

                  create.setOnClickListener(v -> register());
                  back.setOnClickListener(v -> showLogin());
              }

              private void login() {
                  String email = emailInput.getText().toString().trim();
                  String password = passwordInput.getText().toString();

                  if (email.isEmpty() || password.isEmpty()) {
                      toast("Enter email and password");
                      return;
                  }

                  new Thread(() -> {
                      try {
                          JSONObject body = new JSONObject();
                          body.put("email", email);
                          body.put("password", password);

                          String result = request(
                                  "/auth/v1/token?grant_type=password",
                                  "POST",
                                  body.toString(),
                                  null
                          );

                          JSONObject json = new JSONObject(result);
                          accessToken = json.getString("access_token");

                          runOnUiThread(this::showDashboard);

                      } catch (Exception e) {
                          runOnUiThread(() ->
                                  toast("Login failed: " + e.getMessage()));
                      }
                  }).start();
              }

              private void register() {
                  String name = nameInput.getText().toString().trim();
                  String phone = phoneInput.getText().toString().trim();
                  String email = emailInput.getText().toString().trim();
                  String password = passwordInput.getText().toString();

                  if (name.isEmpty() || email.isEmpty() || password.isEmpty()) {
                      toast("Fill name, email and password");
                      return;
                  }

                  new Thread(() -> {
                      try {
                          JSONObject body = new JSONObject();
                          body.put("email", email);
                          body.put("password", password);

                          JSONObject metadata = new JSONObject();
                          metadata.put("full_name", name);
                          metadata.put("phone", phone);
                          body.put("data", metadata);

                          String result = request(
                                  "/auth/v1/signup",
                                  "POST",
                                  body.toString(),
                                  null
                          );

                          JSONObject json = new JSONObject(result);

                          if (json.has("access_token")) {
                              accessToken = json.getString("access_token");
                              runOnUiThread(this::showDashboard);
                          } else {
                              runOnUiThread(() ->
                                      toast("Account created. Check your email, then login."));
                          }

                      } catch (Exception e) {
                          runOnUiThread(() ->
                                  toast("Registration failed: " + e.getMessage()));
                      }
                  }).start();
              }

              private void showDashboard() {
                  base();

                  logo();

                  TextView title = text("ALFARUQ DATA SERVICE", 24);
                  title.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
                  title.setGravity(Gravity.CENTER);
                  root.addView(title);

                  TextView welcome = text("Welcome to your dashboard 👋", 17);
                  welcome.setGravity(Gravity.CENTER);
                  root.addView(welcome);

                  TextView wallet = text("Wallet Balance\n₦0.00", 24);
                  wallet.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
                  wallet.setPadding(0,30,0,30);
                  root.addView(wallet);

                  Button data = button("🌐  BUY DATA");
                  root.addView(data);

                  Button airtime = button("📱  AIRTIME");
                  root.addView(airtime);

                  Button tv = button("📺  TV SUBSCRIPTION");
                  root.addView(tv);

                  Button electricity = button("⚡  ELECTRICITY");
                  root.addView(electricity);

                  Button transactions = button("🧾  TRANSACTIONS");
                  root.addView(transactions);

                  Button profile = button("👤  PROFILE");
                  root.addView(profile);

                  Button logout = button("LOGOUT");
                  root.addView(logout);

                  data.setOnClickListener(v -> serviceMessage("Data"));
                  airtime.setOnClickListener(v -> serviceMessage("Airtime"));
                  tv.setOnClickListener(v -> serviceMessage("TV Subscription"));
                  electricity.setOnClickListener(v -> serviceMessage("Electricity"));
                  transactions.setOnClickListener(v -> serviceMessage("Transactions"));
                  profile.setOnClickListener(v -> serviceMessage("Profile"));

                  logout.setOnClickListener(v -> {
                      accessToken = "";
                      showLogin();
                  });
              }

              private void serviceMessage(String service) {
                  toast(service + " service will be connected to the VTU provider.");
              }

              private String request(
                      String endpoint,
                      String method,
                      String body,
                      String token
              ) throws Exception {

                  URL url = new URL(SUPABASE_URL + endpoint);
                  HttpURLConnection connection =
                          (HttpURLConnection) url.openConnection();

                  connection.setRequestMethod(method);
                  connection.setRequestProperty(
                          "apikey", SUPABASE_KEY
                  );
                  connection.setRequestProperty(
                          "Content-Type", "application/json"
                  );

                  if (token != null && !token.isEmpty()) {
                      connection.setRequestProperty(
                              "Authorization", "Bearer " + token
                      );
                  }

                  if (body != null) {
                      connection.setDoOutput(true);
                      OutputStream os = connection.getOutputStream();
                      os.write(body.getBytes("UTF-8"));
                      os.close();
                  }

                  int code = connection.getResponseCode();

                  InputStream stream =
                          code >= 200 && code < 400
                                  ? connection.getInputStream()
                                  : connection.getErrorStream();

                  BufferedReader reader =
                          new BufferedReader(new InputStreamReader(stream));

                  StringBuilder response = new StringBuilder();
                  String line;

                  while ((line = reader.readLine()) != null) {
                      response.append(line);
                  }

                  reader.close();

                  if (code < 200 || code >= 400) {
                      throw new Exception(response.toString());
                  }

                  return response.toString();
              }

              private void toast(String message) {
                  Toast.makeText(
                          this,
                          message,
                          Toast.LENGTH_LONG
                  ).show();
              }
          }
          EOF

          gradle wrapper --gradle-version 8.9

      - name: Build APK
        run: |
          chmod +x gradlew
          ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: ALFARUQ-DATA-SERVICE-APK
          path: app/build/outputs/apk/debug/app-debug.apk
