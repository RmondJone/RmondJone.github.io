---
title: "FlutterBoost原理解析"
date: 2023-04-10T15:24:55+08:00
draft: false
categories: ["移动端"]
tags: ["Flutter"]
---

## 一、push原理
### 流程图
![](/images/flutterboost_1.webp)

### 核心代码
```dart
Future<T> push<T extends Object?>(String name,
                                  {Map<String, dynamic>? arguments,
                                   bool withContainer = false,
                                   bool opaque = true}) async {
  var pushOption = BoostInterceptorOption(name,
                                          arguments: arguments ?? <String, dynamic>{});
  var future = Future<dynamic>(
    () => InterceptorState<BoostInterceptorOption>(pushOption));
  for (var interceptor in appState!.interceptors) {
    future = future.then<dynamic>((dynamic _state) {
      final state = _state as InterceptorState<dynamic>;
      if (state.type == InterceptorResultType.next) {
        final pushHandler = PushInterceptorHandler();
        interceptor.onPush(state.data, pushHandler);
        return pushHandler.future;
      } else {
        return state;
      }
    });
  }
  
  return future.then((dynamic _state) {
    final state = _state as InterceptorState<dynamic>;
    if (state.data is BoostInterceptorOption) {
      assert(state.type == InterceptorResultType.next);
      pushOption = state.data;
      if (isFlutterPage(pushOption.name)) {
        return appState!.pushWithResult(pushOption.name,
                                        uniqueId: pushOption.uniqueId,
                                        arguments: pushOption.arguments,
                                        withContainer: withContainer,
                                        opaque: opaque) as FutureOr<T>;
      } else {
        Map<String, dynamic> data = {};
        data = Map<String, dynamic>.from(pushOption.arguments);
        
        final params = CommonParams()
          ..pageName = pushOption.name
          ..arguments = data;
        appState!.nativeRouterApi!.pushNativeRoute(params);
        return appState!.pendNativeResult(pushOption.name);
      }
    } else {
      assert(state.type == InterceptorResultType.resolve);
      return Future<T>.value(state.data as T);
    }
  });
  }
```

```dart
  Future<void> pushNativeRoute(CommonParams arg) async {
    final Object encoded = arg.encode();
    final BasicMessageChannel<Object?> channel = BasicMessageChannel<Object?>(
        'dev.flutter.pigeon.NativeRouterApi.pushNativeRoute', const StandardMessageCodec(), binaryMessenger: _binaryMessenger);
    final Map<Object?, Object?>? replyMap =
    await channel.send(encoded) as Map<Object?, Object?>?;
    if (replyMap == null) {
      throw PlatformException(
        code: 'channel-error',
        message: 'Unable to establish connection on channel.',
        details: null,
      );
    } else if (replyMap['error'] != null) {
      final Map<Object?, Object?> error = (replyMap['error'] as Map<Object?, Object?>?)!;
      throw PlatformException(
        code: (error['code'] as String?)!,
        message: error['message'] as String?,
        details: error['details'],
      );
    } else {
      // noop
    }
  }
```

```dart
  Future<void> pushFlutterRoute(CommonParams arg) async {
    final Object encoded = arg.encode();
    final BasicMessageChannel<Object?> channel = BasicMessageChannel<Object?>(
        'dev.flutter.pigeon.NativeRouterApi.pushFlutterRoute', const StandardMessageCodec(), binaryMessenger: _binaryMessenger);
    final Map<Object?, Object?>? replyMap =
    await channel.send(encoded) as Map<Object?, Object?>?;
    if (replyMap == null) {
      throw PlatformException(
        code: 'channel-error',
        message: 'Unable to establish connection on channel.',
        details: null,
      );
    } else if (replyMap['error'] != null) {
      final Map<Object?, Object?> error = (replyMap['error'] as Map<Object?, Object?>?)!;
      throw PlatformException(
        code: (error['code'] as String?)!,
        message: error['message'] as String?,
        details: error['details'],
      );
    } else {
      // noop
    }
  }
```

```dart
    @Override
    public void pushNativeRoute(CommonParams params) {
        if (delegate != null) {
            requestCode++;
            if (pageNames != null) {
                pageNames.put(requestCode, params.getPageName());
            }
            FlutterBoostRouteOptions options = new FlutterBoostRouteOptions.Builder()
                    .pageName(params.getPageName())
                    .arguments((Map<String, Object>) (Object) params.getArguments())
                    .requestCode(requestCode)
                    .build();
            delegate.pushNativeRoute(options);
        } else {
            throw new RuntimeException("FlutterBoostPlugin might *NOT* set delegate!");
        }
    }
```

```dart
    @Override
    public void pushFlutterRoute(CommonParams params) {
        if (delegate != null) {
            FlutterBoostRouteOptions options = new FlutterBoostRouteOptions.Builder()
                    .pageName(params.getPageName())
                    .uniqueId(params.getUniqueId())
                    .opaque(params.getOpaque())
                    .arguments((Map<String, Object>) (Object) params.getArguments())
                    .build();
            delegate.pushFlutterRoute(options);
        } else {
            throw new RuntimeException("FlutterBoostPlugin might *NOT* set delegate!");
        }
    }
```
## 二、pop原理
### 流程图
![](/images/flutterboost_2.webp)

### 核心代码
```dart
  Future<bool> pop(
      {String? uniqueId, Object? result, bool onBackPressed = false}) async {
    BoostContainer? container;
    if (uniqueId != null) {
      container = _findContainerByUniqueId(uniqueId);
      if (container == null) {
        Logger.error('uniqueId=$uniqueId not found');
        return false;
      }
      if (container != topContainer) {
        await _removeContainer(container);
        return true;
      }
    } else {
      container = topContainer;
    }

    final currentPage = topContainer?.topPage.pageInfo.uniqueId!;
    assert(currentPage != null);
    _completePendingResultIfNeeded(currentPage);

    // 1.If uniqueId == null,indicate we simply call BoostNavigaotor.pop(),
    // so we call navigator?.maybePop();
    // 2.If uniqueId is topPage's uniqueId, so we navigator?.maybePop();
    // 3.If uniqueId is not topPage's uniqueId, so we will remove an existing
    // page in container.
    if (uniqueId == null ||
        uniqueId == container!.pages.last.pageInfo.uniqueId) {
      final handled = onBackPressed
          ? await container?.navigator?.maybePop(result)
          : container?.navigator?.canPop();
      if (handled != null) {
        if (!handled) {
          assert(container!.pageInfo!.withContainer!);
          Map data = {};
          if (result is Map) {
            data = Map<Object, Object?>.from(result);
          }
          final params = CommonParams()
            ..pageName = container!.pageInfo!.pageName!
            ..uniqueId = container.pageInfo!.uniqueId!
            ..arguments = data;
          await nativeRouterApi!.popRoute(params);
        } else {
          if (!onBackPressed) {
            container!.navigator!.pop(result);
          }
        }
      }
    } else {
      final page = container.pages.singleWhereOrNull(
              (entry) => entry.pageInfo.uniqueId == uniqueId);
      container.removePage(page);
    }

    Logger.log('pop container, uniqueId=$uniqueId, result:$result, $container');
    return true;
  }
```

```dart
  Future<void> _removeContainer(BoostContainer container) async {
    if (container.pageInfo!.withContainer!) {
      Logger.log('_removeContainer ,  uniqueId=${container.pageInfo!.uniqueId}');
      final params = CommonParams()
        ..pageName = container.pageInfo!.pageName!
        ..uniqueId = container.pageInfo!.uniqueId!
        ..arguments = container.pageInfo!.arguments as Map<Object, Object>;
      return await _nativeRouterApi!.popRoute(params);
    }
  }
```

```dart
    @Override
    public void popRoute(CommonParams params, Messages.Result<Void> result) {
        if (delegate != null) {
            FlutterBoostRouteOptions options = new FlutterBoostRouteOptions.Builder()
                    .pageName(params.getPageName())
                    .uniqueId(params.getUniqueId())
                    .arguments((Map<String, Object>) (Object) params.getArguments())
                    .build();
            boolean isHandle = delegate.popRoute(options);
            //isHandle代表是否已经自定义处理，如果未自定义处理走默认逻辑
            if (!isHandle) {
                String uniqueId = params.getUniqueId();
                if (uniqueId != null) {
                    FlutterViewContainer container = FlutterContainerManager.instance().findContainerById(uniqueId);
                    if (container != null) {
                        container.finishContainer((Map<String, Object>) (Object) params.getArguments());
                    }
                    result.success(null);
                } else {
                    throw new RuntimeException("Oops!! The unique id is null!");
                }
            }
        } else {
            throw new RuntimeException("FlutterBoostPlugin might *NOT* set delegate!");
        }
    }
```

```dart
    @Override
    public void finishContainer(Map<String, Object> result) {
        if (result != null) {
            Intent intent = new Intent();
            intent.putExtra(ACTIVITY_RESULT_KEY, new HashMap<String, Object>(result));
            setResult(Activity.RESULT_OK, intent);
        }
        finish();
        if (DEBUG) Log.d(TAG, "#finishContainer: " + this);
    }
```
## 三、popUtil原理
### 流程图
![](/images/flutterboost_3.webp)

### 核心代码

```dart
  void popUntil({String? route, String? uniqueId}) async {
    BoostContainer? targetContainer;
    BoostPage? targetPage;
    int popUntilIndex = containers.length;
    if (uniqueId != null) {
      for (int index = containers.length - 1; index >= 0; index--) {
        for (BoostPage page in containers[index].pages) {
          if (uniqueId == page.pageInfo.uniqueId ||
              uniqueId == containers[index].pageInfo!.uniqueId) {
            //uniqueId优先级更高，优先匹配
            targetContainer = containers[index];
            targetPage = page;
            break;
          }
        }
        if (targetContainer != null) {
          popUntilIndex = index;
          break;
        }
      }
    }

    if (targetContainer == null && route != null) {
      for (int index = containers.length - 1; index >= 0; index--) {
        for (BoostPage page in containers[index].pages) {
          if (route == page.name) {
            targetContainer = containers[index];
            targetPage = page;
            break;
          }
        }
        if (targetContainer != null) {
          popUntilIndex = index;
          break;
        }
      }
    }

    if (targetContainer != null && targetContainer != topContainer) {
      /// containers item index would change when call 'nativeRouterApi.popRoute' method with sync.
      /// clone containers keep original item index.
      List<BoostContainer> _containersTemp = [...containers];
      for (int index = _containersTemp.length - 1; index > popUntilIndex; index--) {
        BoostContainer container = _containersTemp[index];
        final params = CommonParams()
          ..pageName = container.pageInfo!.pageName!
          ..uniqueId = container.pageInfo!.uniqueId!
          ..arguments = {"animated": false};
        await nativeRouterApi!.popRoute(params);
      }

      if (targetContainer.topPage != targetPage) {
        Future<void>.delayed(
            const Duration(milliseconds: 50),
                () => targetContainer?.navigator
                ?.popUntil(ModalRoute.withName(targetPage!.name!)));
      }
    } else {
      topContainer?.navigator?.popUntil(ModalRoute.withName(targetPage!.name!));
    }
  }
```

