import 'dart:async';
import 'dart:convert';
import 'dart:developer';
import 'dart:io';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';

Function(Map)? onPayload;

Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  await FirebaseMessaging.instance
      .setForegroundNotificationPresentationOptions(
    alert: true,
    badge: true,
    sound: true,
  );
  await listenNotification(message);
}


const AndroidNotificationChannel _defaultChannel = AndroidNotificationChannel(
  'default',
  'System Notifications',
  description: 'System Notifications',
  importance: Importance.max,
  enableVibration: true,
  playSound: true,
);

hasSupportPushNotification(){
  return (kIsWeb || Platform.isAndroid || Platform.isIOS || Platform.isMacOS);
}

final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();

class PushNotificationLib{

  final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();
  PushNotificationLib._();
  static final instance = PushNotificationLib._();

  final StreamController<ReceivedNotification> didReceiveLocalNotificationStream = StreamController<ReceivedNotification>.broadcast();


  static void notificationTapBackground(NotificationResponse notificationResponse) {
    // ignore: avoid_print
    print('notification(${notificationResponse.id}) action tapped: '
        '${notificationResponse.actionId} with'
        ' payload: ${notificationResponse.payload}');
    if (notificationResponse.input?.isNotEmpty ?? false) {
      // ignore: avoid_print
      print(
          'notification action tapped with input: ${notificationResponse.input}');
    }
  }

  Future<void> initialization() async{
    const AndroidInitializationSettings initializationSettingsAndroid =
    AndroidInitializationSettings('app_icon');
    FirebaseMessaging.onBackgroundMessage(firebaseMessagingBackgroundHandler);
    final DarwinInitializationSettings initializationSettingsDarwin =
    DarwinInitializationSettings(
      requestAlertPermission: false,
      requestBadgePermission: false,
      requestSoundPermission: false,
      onDidReceiveLocalNotification:
          (int id, String? title, String? body, String? payload) async {
        didReceiveLocalNotificationStream.add(
          ReceivedNotification(
            id: id,
            title: title,
            body: body,
            payload: payload,
          ),
        );
      },
      notificationCategories: darwinNotificationCategories,
    );
    final LinuxInitializationSettings initializationSettingsLinux =
    LinuxInitializationSettings(
      defaultActionName: 'Open notification',
      defaultIcon: AssetsLinuxIcon('icons/app_icon.png'),
    );
    final InitializationSettings initializationSettings = InitializationSettings(
      android: initializationSettingsAndroid,
      iOS: initializationSettingsDarwin,
      macOS: initializationSettingsDarwin,
      linux: initializationSettingsLinux,
    );

    // await flutterLocalNotificationsPlugin.initialize(initializationSettings,onDidReceiveBackgroundNotificationResponse: ,
    //     onSelectNotification: (String? payload) async {
    //       if (payload != null) {
    //         final _payload = (payload.startsWith('{') &&
    //             payload.endsWith('}')) ? jsonDecode(payload) : payload;
    //         _handlerMessage(_payload);
    //       }
    //     }
    // );
    await flutterLocalNotificationsPlugin.initialize(
      initializationSettings,
      onDidReceiveNotificationResponse: (NotificationResponse notificationResponse) {
        switch (notificationResponse.notificationResponseType) {
          case NotificationResponseType.selectedNotification:
            final _payload = (notificationResponse.payload!.startsWith('{') &&
                notificationResponse.payload!.endsWith('}')) ? jsonDecode(notificationResponse.payload!) : notificationResponse.payload;
            _handlerMessage(_payload);
            // _handlerMessage(json.decode(notificationResponse.payload!));
            break;
          case NotificationResponseType.selectedNotificationAction:
            if (notificationResponse.actionId == 'navigationActionId') {
              // selectNotificationStream.add(notificationResponse.payload);
            }
            break;
        }
      },
      onDidReceiveBackgroundNotificationResponse: notificationTapBackground,
    );

  }

  initListen() async{
    if (Platform.isIOS || Platform.isMacOS) {
      await flutterLocalNotificationsPlugin.resolvePlatformSpecificImplementation<IOSFlutterLocalNotificationsPlugin>()?.requestPermissions(
        alert: true,
        badge: true,
        sound: true,
      );
      await flutterLocalNotificationsPlugin.resolvePlatformSpecificImplementation<MacOSFlutterLocalNotificationsPlugin>()?.requestPermissions(
        alert: true,
        badge: true,
        sound: true,
      );
    } else if (Platform.isAndroid) {
      final AndroidFlutterLocalNotificationsPlugin? androidImplementation =
      flutterLocalNotificationsPlugin.resolvePlatformSpecificImplementation<
          AndroidFlutterLocalNotificationsPlugin>();

      // final bool? granted = await androidImplementation?.requestPermission();
    }
    didReceiveLocalNotificationStream.stream.listen((ReceivedNotification receivedNotification) async {
      log('-----------------${receivedNotification.title}----');
    });

    if (hasSupportPushNotification()) {
      await FirebaseMessaging.instance.requestPermission(
          alert: true,
          badge: true,
          provisional: true,
          sound: true
      ).then((value) {
      });

      FirebaseMessaging.instance.getAPNSToken();
      FirebaseMessaging.instance.getInitialMessage().then((RemoteMessage? message) {
        if (message != null) {
          if (kDebugMode) {
            _handlerMessage(message.data);
          }
        }
      });
      FirebaseMessaging.instance.getToken().then((String? token) {
        assert(token != null);
        if (token != null) {
          log('token: $token');
          // Setting('Config').put('tokenPushNotification', token);
        }
      });
      // FirebaseMessaging.instance.subscribeToTopic('All');
      FirebaseMessaging.onMessage.listen((RemoteMessage message) async{
        await listenNotification(message);
      });

      FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
        _handlerMessage(message.data);
      });
      FirebaseMessaging.instance.onTokenRefresh.listen((fcmToken) {

      }).onError((err) {
      });

    }


    // selectNotificationStream.stream.listen((String? payload) async {
    //   print('--------------------$payload');
    // });

  }

  showNotificationLocal(ReceivedNotification message){
    flutterLocalNotificationsPlugin.show(
      message.hashCode,
      message.title,
      message.body,
      NotificationDetails(
        android: AndroidNotificationDetails(
          _defaultChannel.id,
          _defaultChannel.name,
          channelDescription: _defaultChannel.description,
        ),
      ),
      payload: jsonEncode(message.title),
    );
  }

  setPayloadFunction(Function(Map) a){
    onPayload = a;
  }

  final List<DarwinNotificationCategory> darwinNotificationCategories = <DarwinNotificationCategory>[
    DarwinNotificationCategory(
      'darwinNotificationCategoryText',
      actions: <DarwinNotificationAction>[
        DarwinNotificationAction.text(
          'text_1',
          'Action 1',
          buttonTitle: 'Send',
          placeholder: 'Placeholder',
        ),
      ],
    ),
    DarwinNotificationCategory(
      'darwinNotificationCategoryPlain',
      actions: <DarwinNotificationAction>[
        DarwinNotificationAction.plain('id_1', 'Action 1'),
        DarwinNotificationAction.plain(
          'id_2',
          'Action 2 (destructive)',
          options: <DarwinNotificationActionOption>{
            DarwinNotificationActionOption.destructive,
          },
        ),
        DarwinNotificationAction.plain(
          'navigationActionId',
          'Action 3 (foreground)',
          options: <DarwinNotificationActionOption>{
            DarwinNotificationActionOption.foreground,
          },
        ),
        DarwinNotificationAction.plain(
          'id_4',
          'Action 4 (auth required)',
          options: <DarwinNotificationActionOption>{
            DarwinNotificationActionOption.authenticationRequired,
          },
        ),
      ],
      options: <DarwinNotificationCategoryOption>{
        DarwinNotificationCategoryOption.hiddenPreviewShowTitle,
      },
    )
  ];
}

_handlerMessage(Map payload)async{
  if(onPayload != null){
    return await onPayload!(payload);
  }
}

listenNotification(RemoteMessage message)async{
  RemoteNotification? notification = message.notification;
  AndroidNotification? android = message.notification?.android;
  if (notification != null && android != null) {
    showNotification(message);
  }
}

showNotification(RemoteMessage message){
  RemoteNotification? notification = message.notification;
  flutterLocalNotificationsPlugin.show(
      notification.hashCode,
      notification!.title,
      notification.body,
      NotificationDetails(
        android: AndroidNotificationDetails(
          _defaultChannel.id,
          _defaultChannel.name,
          channelDescription: _defaultChannel.description,
        ),
      ),
      payload: jsonEncode(message.data)
  );
}


class ReceivedNotification {
  ReceivedNotification({
    required this.id,
    required this.title,
    required this.body,
    required this.payload,
  });

  final int id;
  final String? title;
  final String? body;
  final String? payload;
}
