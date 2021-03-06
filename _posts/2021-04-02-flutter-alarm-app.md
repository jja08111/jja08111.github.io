---
title: "[Flutter] Flutter로 알람 앱 기능 구현하기"
date: 2021-04-02 03:00:00 -0400
categories: flutter
tags:
- flutter
- android_alarm_manager
- alarm
---



우리가 흔히 생각하는 알람 앱을 Flutter를 이용하여 만들고 싶었습니다. 알람 시간이 되면 앱이 실행되고 알람 화면이 띄워져 알람이 울리는 것처럼요. 하지만 Flutter로 구현된 알람앱은 흔하지 않았습니다. 다행히 [random-alarm](https://github.com/geisterfurz007/random-alarm)과 같은 좋은 소스를 찾게되어 공유하고자 이 글을 작성했습니다.

**참고1** : _이 코드는 **안드로이드에서만 작동**하며 네이티브로 구현하지 않았습니다. 그저 플러터로 알람앱을 흉내 냈다 정도로 읽어주시면 감사하겠습니다. 네이티브로 작성하면 훨씬 빠르고 기본 알람 앱처럼 구동이 될 것입니다. 혹시 해당 오픈소스가 있다면 공유해주세요!_

**참고2** : *해당 게시글은 **Dart 2.12** 버전을 타겟으로 작성되었습니다.*



# 미리보기

- [random-alarm](https://github.com/geisterfurz007/random-alarm) - by [**geisterfurz007**](https://github.com/geisterfurz007) (**게시글의 코드와는 약간 다를 수 있습니다.**)

  | Main screen                                                  | Edit screen                                                  | Alarm screen                                                 |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | <img src="https://user-images.githubusercontent.com/26303198/78984346-48e28580-7b26-11ea-932f-23d8f3f18ce8.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984373-5c8dec00-7b26-11ea-8120-d07890608036.png" style="zoom:50%;" /> | <img src="https://user-images.githubusercontent.com/26303198/78984586-ea69d700-7b26-11ea-8e27-6edb27bc971f.png" style="zoom:50%;" /> |

  

- [꿀밤(Bedtime)](https://play.google.com/store/apps/details?id=io.github.jja08111.good_night_app)

  <img src="https://github.com/jja08111/jja08111.github.io/blob/master/assets/images/bedtime_app_preview.gif?raw=true" style="zoom:75%;" />





# 작동 순서

먼저 앱의 알람 기능은 다음과 같은 순서로 작동합니다.

1. `android_alarm_manager`로 알람을 설정한다.
2. 알람 시간이 되어 `android_alarm_manager`의 `callback`함수가 호출된다.
3. 알람 플래그 파일을 생성한다.(`shared_preference`등 이용)
4. 직접 구현한 `polling_worker`를 이용하여 플래그 파일이 생성되었는지 확인한다.
5. 플래그 파일이 존재한다면 상태를 변경(`mobx, provider`등 이용)하여 알람 화면을 띄운다.





# 요구사항

## 권한 

안정적으로 알람 앱을 구현하기 위해서는 아래 두 가지의 권한이 필요합니다. 이는 안드로이드 10 이상 버전에는 무조건 허용되어야합니다.

1. 다른 앱 위에 표시(Display over other apps) : 앱이 실행중이 아니거나 백그라운드에서 실행중일때 앱을 최상단에 띄우기 위해 필요합니다.
2. 배터리 최적화 무시(Ignore battery optimization) : 배터리 최적화 기능 때문에 가끔 알람이 제대로 동작하지 않을 때가 있는데 이를 방지합니다.

## 플러그인 

필요한 플러그인은 다음과 같습니다.

1. [android_alarm_manager_plus](https://pub.dev/packages/android_alarm_manager_plus) [![pub package](https://img.shields.io/pub/v/android_alarm_manager_plus.svg)](https://pub.dev/packages/android_alarm_manager)

   알람이 작동할때, 즉 설정한 시간이 되었을 때 앱을 실행하기 위해서는 아래와 같이 플러그인 수정이 필요합니다. 해당 플러그인의 `AlarmBroadcastReceiver.java`파일을 다음과 같이 수정하세요. 

   ```
   파일 위치 예 : C:\Users\Name\AppData\Local\Pub\Cache\hosted\pub.dartlang.org\android_alarm_manager_plus-1.0.2\android\src\main\java\dev\fluttercommunity\plus\androidalarmmanager\AlarmBroadcastReceiver.java
   ```

   ```java
   package dev.fluttercommunity.plus.androidalarmmanager;
   
   import android.content.BroadcastReceiver;
   import android.content.Context;
   import android.content.Intent;
   import android.os.PowerManager;
   
   public class AlarmBroadcastReceiver extends BroadcastReceiver {
       @Override
       public void onReceive(Context context, Intent intent) {
           PowerManager powerManager = (PowerManager)
                   context.getSystemService(Context.POWER_SERVICE);
           PowerManager.WakeLock wakeLock = powerManager.newWakeLock(PowerManager.FULL_WAKE_LOCK |
                   PowerManager.ACQUIRE_CAUSES_WAKEUP |
                   PowerManager.ON_AFTER_RELEASE, "AlarmBroadcastReceiver:My wakelock");
   
           Intent startIntent = context
                   .getPackageManager()
                   .getLaunchIntentForPackage(context.getPackageName());
   
           if (startIntent != null)
               startIntent.setFlags(
                       Intent.FLAG_ACTIVITY_REORDER_TO_FRONT |
                               Intent.FLAG_ACTIVITY_NEW_TASK |
                               Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED
               );
   
           wakeLock.acquire(3 * 60 * 1000L /*3 minutes*/);
           context.startActivity(startIntent);
           AlarmService.enqueueAlarmProcessing(context, intent);
           wakeLock.release();
       }
   }
   ```
   
   이는 [이곳](https://github.com/flutter/flutter/issues/30555#issuecomment-501597824)에서 자세히 볼 수 있습니다.
   
2. [mobx](https://pub.dev/packages/mobx) [![pub package](https://img.shields.io/pub/v/mobx.svg)](https://pub.dev/packages/mobx)

   이 플러그인을 이용하려면 클래스를 생성 후 다음과 같은 빌드 코드를 터미널 창에 입력해야합니다.

   `flutter packages pub run build_runner build` 

   자세한 정보는 [mobx 페이지](https://mobx.netlify.app/getting-started/)에서 볼수 있습니다.

3. [shared_preferences](https://pub.dev/packages/shared_preferences) [![pub package](https://img.shields.io/pub/v/shared_preferences.svg)](https://pub.dev/packages/shared_preferences) 

   알람 플래그를 형성할 때 이용할 것입니다.

4. (필수 X) [permission_handler](https://pub.dev/packages/permission_handler) [![pub package](https://img.shields.io/pub/v/permission_handler.svg)](https://pub.dev/packages/permission_handler) 

   앱에서 사용자에게 권한을 요구할때 유용한 플러그인입니다. 

   

이와 같은 준비가 되면 알람 앱을 만들 준비가 끝났습니다. 





# 기능 구현 

알람 객체와 리스트를 구현하여 관리한다고 가정하고 기능적인 부분만 다루도록 하겠습니다.

## 1. 알람 설정 

먼저 `android_alarm_manager`를 이용하여 목표하는 시간에 알람을 설정합니다. 이때 `alarmClock`을 true로 하여 안드로이드 내부에서 정확한 알람 시계로 작동할 수 있도록 합니다. 그리고 알람 작동시 스마트폰의 화면을 켜기 위해 `wakeup` 또한 true로 설정합니다. 재부팅시 알람이 작동할 수 있도록 `rescheduleOnReboot`도 true로 설정합니다.

```dart
AndroidAlarmManager.oneShotAt(
  dateTime,
  id,
  _callback,
  alarmClock: true,
  wakeup: true,
  rescheduleOnReboot: true,
);
```



## 2. callback 함수 호출

해당 시간이 되면 `android_alarm_manager`의 `callback`함수가 호출될 것 입니다. 이 함수 내부는 아래와 같습니다. 이 함수는 클래스 내부인 경우 아래와 같이 정적으로 선언해야 합니다.

```dart
static void _callback(int id) {
  AlarmFlagManager().set(id);
}
```

함수가 호출되고 플래그를 설정하는 모습입니다. 저는 `AlarmFlagManager`라는 클래스를 만들어 플래그를 설정하고 있습니다. 

해당 클래스를 보겠습니다. 이 클래스는 플래그를 설정하는 함수, 지우는 함수, 현재 울린 알람의 id를 반환하는 함수로 구성되어있습니다.

### alarm_flag_manager.dart

```dart
import 'package:shared_preferences/shared_preferences.dart';

class AlarmFlagManager {
  static final AlarmFlagManager _instance = AlarmFlagManager._();

  factory AlarmFlagManager() => _instance;

  AlarmFlagManager._();

  static const String _alarmFlagKey = "alarmFlagKey";

  Future<void> set(int id) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt(_alarmFlagKey, id);
  }

  Future<int?> getFiredId() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.reload();
    return prefs.getInt(_alarmFlagKey);
  }

  Future<void> clear() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove(_alarmFlagKey);
  }
}
```

## 3. Flag 탐색 후 상태변경

알람 플래그를 형성했으니 `AlarmPollingWorker`로 탐색을 하여 **알람이 울린 상태**로 변경해야합니다. 이때 탐색기는 다음 두 가지 경우에 실행됩니다.

- 앱 실행시 -> 앱의 매인 클래스 진입시
- 앱을 종료하지 않고 다시 진입한 후 -> 앱 메인 루트 화면에서 `WidgetsBindingObserver` 이용

두 번째 경우에 앱이 실행 중일때 알람이 울린 경우도 포함됩니다. 그렇다면 어떻게 위와 같은 사항을 탐색할까요? 바로 앱의 메인 UI에 해당하는 곳에 `WidgetsBindingObserver`를 이용하면 됩니다. 이는 앱이 종료되는지 혹은 다시 진입했는지 등 앱의 라이프 사이클을 관찰하는 클래스라고 생각하시면 됩니다.

탐색이 된 경우 아래의 `AlarmStatus`의 상태를 변경하면 됩니다.

### 앱 실행시

```dart
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  const MyApp() {
    AlarmPollingWorker().createPollingWorker();
  }
  ...생략
}
```

### 앱 재진입시

```dart
class SomeScreen extends StatefulWidget {
  @override
  _SomeScreenState createState() => _SomeScreenState();
}

class _SomeScreenState extends State<ObserveAlarm>
    with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance!.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance!.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      
      ...생략
            
      case AppLifecycleState.resumed:
        AlarmPollingWorker().createPollingWorker();
        break;
    }
  }
  
  ...생략
}
```

### alarm_polling_worker.dart

```dart
class AlarmPollingWorker {
  static AlarmPollingWorker _instance = AlarmPollingWorker._();

  factory AlarmPollingWorker() {
    return _instance;
  }

  AlarmPollingWorker._();

  bool _running = false;

  // 알람 플래그 탐색을 시작한다.
  void createPollingWorker() async {
    if (_running) return;

    _running = true;
    _poller(120).then((callbackAlarmId) async {
      _running = false;
      if (callbackAlarmId != null) {
        final alarmStatus = AlarmStatus();

        if (alarmStatus.alarmId == null) {
          alarmStatus.fire(callbackAlarmId);
        }
        await AlarmFlagManager().clear();
      }
    });
  }
    
  // 알람 플래그를 찾은 경우 해당 알람의 Id를 반환하고, 플래그가 없는 경우 null을 반환한다.
  Future<int?> _poller(int iterations) async {
    int? alarmId;
    int iterator = 0;

    await Future.doWhile(() async {
      alarmId = await AlarmFlagManager().getFiredId();
      if (alarmId != null || iterator++ >= iterations) return false;
      await Future.delayed(const Duration(milliseconds: 25));
      return true;
    });
    return alarmId;
  }
}
```

### alarm_status.dart

```dart
part 'alarm_status.g.dart';

class AlarmStatus extends _AlarmStatus with _$AlarmStatus {
  static final AlarmStatus _instance = AlarmStatus._();

  factory AlarmStatus() {
    return _instance;
  }

  AlarmStatus._();
}

abstract class _AlarmStatus with Store {
  @observable
  bool _isFired = false;

  @computed
  bool get isFired => _isFired;

  int? callbackAlarmId;

  @action
  void fire(int id) {
    callbackAlarmId = id;
    _isFired = true;
  }

  @action
  void clear() {
    _isFired = false;
    callbackAlarmId = null;
  }
}
```

*위의 코드는 `mobx` 플러그인을 이용하고 있습니다. 따라서 part file을 생성해야 하는데요. 이는 [이곳](https://mobx.netlify.app/getting-started/)을 참고하시기 바랍니다.*

## 4. 메인 화면에서 알람 화면으로 변경

이제 상태를 변경하였으니 알람 화면을 보여주면 됩니다. 저는 `ObserveAlarm`이라는 클래스를 만들어 화면을 분기했습니다. `AlarmStatus().isFired`가 true이면 알람 화면을, 아니면 홈 화면을 보여줍니다. 저는 앞서 소개한  `WidgetsBindingObserver`를 이곳에 적용했습니다.

### observe_alarm.dart

```dart
class ObserveAlarm extends StatefulWidget {
  final Widget child;

  ObserveAlarm({
    Key? key,
    required this.child,
  }) : super(key: key);

  @override
  _ObserveAlarmState createState() => _ObserveAlarmState();
}

class _ObserveAlarmState extends State<ObserveAlarm>
    with WidgetsBindingObserver {
  
  ...생략
    
  @override
  Widget build(BuildContext context) {
    return Observer(builder: (context) {
      AlarmStatus status = AlarmStatus();
        
      if (status.isFired) {
        return AlarmScreen(alarm: AlarmList().getAlarmFrom(id));
      }
      return widget.child;
    });
  }
}
```

위에서 보이는 `AlarmScreen`, `AlarmList`클래스는 원하시는대로 구현하시면 됩니다. 알람 화면 경우는 진입시 소리와 진동을 작동하면 좋겠죠? 



# 마치며

추가적으로 알람을 주 단위로 반복하여 설정하는 기능, 알람 다시 울림 기능 등을 구현할 수도 있습니다.

Flutter로 알람 앱의 기본적인 기능을 구현해봤습니다. 알람 기능만 네이티브로 따로 구현하는 것도 좋은 방법이라고 생각합니다. 이상 긴 글 읽어 주셔서 감사합니다!





# 업데이트 내역

- 2021-04-02 초기 업로드
- 2021-07-09 내용 추가, 보완 및 Null-safety 적용

