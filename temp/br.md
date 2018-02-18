---
layout : post
title: 브로드캐스트와 브로드캐스트 리시버
author: jwkang2
date: 2018-02-18 16:00
---

## 정적 리시버와 동적 리시버
### Broadcasting
- 보낼 intent에 action을 정의한다.
- data를 putExtra 함수로 정의 한다.
- sendBroadcast 함수로 intent를 전달한다.
```java
    public void onClick(View v)
    {
        Intent intent = new Intent();

        intent .setAction("com.superdroid.test.Broadcasting.action.FILE_DOWNLOADED");
        intent.putExtra("FILE_NAME", "superdroid.png");

        sendBroadcast(intent);

        Toast.makeText(v.getContext(), "Send Broadcast message", Toast.LENGTH_SHORT).show();
    }
```
### 정적 리시버
- BroadCastReceiver는 안드로이드 application 4대 구성요소 이므로 AndroidMenifest.xml에 application에 정의 한다.
- receiver를 정의할 때 클래스 이름을 android:name tag로 등록한다.
- intent-filter 에 수신할 BR의 action name을 정의한다.
```xml
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".BR_Receiver_MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver android:name=".BroadcastSideReceiver">
            <intent-filter>
                <action android:name=
                    "com.superdroid.test.Broadcasting.action.FILE_DOWNLOADED" />
            </intent-filter>
        </receiver>
    </application>
```
### 동적 리시버
- BroadCastReceiver 인스턴스를 생성해 onReceive를 재정의 한다.
- IntentFilter 인스턴스를 생성하고 필요한 action을 정의한다.
  -  getString(R.string.FILE_DOWNLOAD_MESSAGE)
- registerReceiver 함수의 인자로 IntentFilter 인스턴스를
- onDestroy 함수를 재정의하고 unregisterReceiver 함수로 등록한 BR을 release 한다.
- 정적 receiver는 system이 등록 및 해제하기 때문에 unregisterReceiver가 필요하지 않다.
```java
public class MainActivity extends Activity {

    BroadcastReceiver mBRReceiver = null;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(getString(R.string.FILE_DOWNLOAD_MESSAGE));

        mBRReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                String fileName = intent.getStringExtra("FILE_NAME");

                TextView text_view = (TextView) findViewById(R.id.message);
                text_view.setText(fileName+" has been received");
            }
        };

        registerReceiver(mBRReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        unregisterReceiver(mBRReceiver);;
        super.onDestroy();
    }
}
```
## 리시버의 동작 제한
1. Intent.FLAG_EXCLUDE_STOPPED_PACKAGES
- BR을 포함하는 activity가 한번도 실행되지 않았거나 강제종료 되었다면 BR은 동작하지 않는다.
- adb shell sumpsys package 를 통해 package 정보를 확인할 수 있고 sopped=[false/true] 를 통해 그 값을 확인할 수 있다.
```
adb shell sumpsys package
```
- stopped가 true이면 한번도 실행되지 않았거나 강제 종료 됐음을 의미한다.
2. Intent.FLAG_INCLUDE_STOPPED_PACKAGES
- FLAG_EXCLUDE_STOPPED_PACKAGES의 반대로 한번도 실행되지 않거나 강제 종료된 package 역시 동작하게 만든다. 단 AndroidMenifest에 등록이 돼 있는 package에 한함

## 리시버 호출순서와 우선순위
- 정적 리시버 호출 순서
  - 정적 리시버가 포함된 앱이 설치된 순으로 호출된다.
  - 정적 리시버는 절대 동시에 실행되지 않고 하나의 리시버가 처리되면 다음 리시버가 호출된다.
- 동적 리시버 호출 순서
  - 동적 리시버는 등록된 순서로 처리된다.
  - 정적 리시버와 다르게 동시에 처리된다.
- setPriority 함수를 통해 우선순위를 변경할 수 있다.

## 시스템이 발송하는 방송


## 기타
포그라운드 서비스는 사용자가 능동적으로 인식하고 있으므로 메모리 부족 시에도 시스템이 중단할 후보로 고려되지 않는 서비스를 말합니다. 포그라운드 서비스는 상태 표시줄에 대한 알림을 제공해야 합니다. 이것은 "진행 중" 제목 아래에 배치되며, 이는 곧 해당 알림은 서비스가 중단되었거나 포그라운드에서 제거되지 않은 이상 해제될 수 없다는 뜻입니다.
포어그라운드는 앱의 액티비티가 화면에 보이는 상태이다.

반대로 백그라운드 서비스는 시스템 메모리가 부족해 지면 종료될 수 있다. 백그라운드는 다른 앱 액티비티에 완전히 가려져 보이지 않는 상태를 말한다.
```
adb shell dumpsys activity -p 로 foreground / background를 알 수 있다.
```
