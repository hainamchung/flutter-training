# Code Walk — Flutter UI Basics & Navigation Flow

> 📌 **Recap từ Module 0:**
> - Dart syntax: `var`, `final`, `const`, nullable types, async/await
> - OOP: classes, constructors, inheritance
> - Toolchain: `pubspec.yaml`, codegen workflow
>
> Nếu chưa nắm vững → quay lại [Module 0](../module-00-dart-primer/) trước.

---

## Walk Order

```
main.dart — app entry point, runApp bootstrap
    ↓
my_app.dart — MaterialApp setup, router delegate
    ↓
common_scaffold.dart — shared layout wrapper
    ↓
splash_page.dart — minimal page skeleton
    ↓
login_page.dart — form UI example
    ↓
main_page.dart — tab navigation
```

Entry point → App shell → Shared components → Page examples → Navigation pattern.

---

## 1. main.dart — Điểm vào của ứng dụng

<!-- AI_VERIFY: base_flutter/lib/main.dart#L1-L9 -->
```dart
import 'dart:async';

import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:flutter/material.dart';
import 'package:flutter_native_splash/flutter_native_splash.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import 'index.dart';
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/main.dart](../../base_flutter/lib/main.dart)

> 🔎 **Quan sát**
> - Imports chia 3 nhóm: `dart:` (SDK) → `package:` (thư viện bên ngoài) → project files
> - `flutter/material.dart` — Material Design widgets. Không import = không có Material widgets
> - `hooks_riverpod/hooks_riverpod.dart` — State management + hooks (sẽ deep dive ở M8)
> - `import 'index.dart'` — barrel file export toàn bộ project (xem [Module 2](../module-02-architecture-barrel/00-overview.md))
> - **Hỏi:** Tại sao `firebase_core` import ở đây nhưng chưa thấy dùng?

> 💡 **FE Perspective**
> **Flutter:** Entry point import Material widgets từ `package:flutter/material.dart`.
> **React/Vue tương đương:** `import React from 'react'` hoặc `import { createApp } from 'vue'`.
> **Khác biệt quan trọng:** Flutter chỉ có 1 entry point (`main.dart`), không có multiple entry points như Next.js pages.

---

### main() — Entry point

<!-- AI_VERIFY: base_flutter/lib/main.dart#L11-L15 -->
```dart
// ignore: avoid_unnecessary_async_function
Future<void> main() async => runZonedGuarded(
      _runMyApp,
      (error, stackTrace) => _reportError(error: error, stackTrace: stackTrace),
    );
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/main.dart](../../base_flutter/lib/main.dart)

> 🔎 **Quan sát**
> - `main()` là **entry point bắt buộc** — Dart runtime tìm function `main` đầu tiên
> - `runZonedGuarded()` wrap toàn bộ app trong error zone — bắt uncaught exceptions
> - `async` cho phép `await` Firebase và các async operations khác
> - `_runMyApp` là function private (prefix `_`) — chỉ gọi từ `main()`
> - **Hỏi:** `_reportError` được gọi khi nào? Điều gì xảy ra nếu bỏ `runZonedGuarded`?

---

### _runMyApp() — App bootstrap

<!-- AI_VERIFY: base_flutter/lib/main.dart#L17-L27 -->
```dart
Future<void> _runMyApp() async {
  final widgetsBinding = WidgetsFlutterBinding.ensureInitialized();
  FlutterNativeSplash.preserve(widgetsBinding: widgetsBinding);
  await Firebase.initializeApp();
  await AppInitializer.init();
  final initialResource = _loadInitialResource();
  runApp(ProviderScope(
    observers: [AppProviderObserver()],
    child: MyApp(initialResource: initialResource),
  ));
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/main.dart](../../base_flutter/lib/main.dart)

> 🔎 **Quan sát**
> - **Thứ tự init quan trọng!** Mỗi bước phụ thuộc bước trước:
>   1. `WidgetsFlutterBinding.ensureInitialized()` — native binding trước mọi async
>   2. `FlutterNativeSplash.preserve(...)` — giữ splash screen
>   3. `Firebase.initializeApp()` — Firebase setup
>   4. `AppInitializer.init()` — DI, env, system config
>   5. `_loadInitialResource()` — initial data
>   6. `runApp(...)` — **mount widget tree vào screen**
> - `runApp()` nhận 1 widget argument — **root của widget tree**
> - `ProviderScope` wrap `MyApp` — đây là **Riverpod root** (state management)
> - **Hỏi:** Tại sao `runApp` phải gọi SAU `ensureInitialized()`?

> 💡 **FE Perspective**
> **Flutter:** `runApp(Widget)` = render root component vào native screen. Tương đương React `ReactDOM.render(<App />, rootElement)`.
> **React/Vue tương đương:** `createRoot(rootElement).render(<App />)` (React 18) hoặc `new Vue({ render: h => h(App) }).$mount('#app')` (Vue 2).
> **Khác biệt quan trọng:** Flutter chỉ nhận 1 widget làm root, không có multiple providers như React `<Provider store={store}><App /></Provider>` trong 1 call.

---

## 2. my_app.dart — MaterialApp & Router Setup

<!-- AI_VERIFY: base_flutter/lib/ui/my_app.dart#L1-L73 -->
```dart
import 'package:auto_route/auto_route.dart';
import 'package:device_preview_plus/device_preview_plus.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import '../index.dart';

class MyApp extends HookConsumerWidget {
  const MyApp({required this.initialResource, super.key});

  final InitialResource initialResource;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final appRouter = ref.watch(appRouterProvider);

    LocaleSettings.setLocaleRawSync('ja');

    return Consumer(
      builder: (BuildContext context, WidgetRef ref, Widget? child) {
        return TranslationProvider(
          child: DevicePreview(
            enabled: Config.enableDevicePreview,
            builder: (_) => MaterialApp.router(
              builder: (context, child) {
                final widget = MediaQuery.withClampedTextScaling(
                  maxScaleFactor: Constant.appMaxTextScaleFactor,
                  minScaleFactor: Constant.appMinTextScaleFactor,
                  child: child ?? const SizedBox.shrink(),
                );

                return Config.enableDevicePreview
                    ? DevicePreview.appBuilder(context, widget)
                    : widget;
              },
              routerDelegate: appRouter.delegate(
                deepLinkBuilder: (deepLink) {
                  return DeepLink(_mapRouteToPageRouteInfo());
                },
                navigatorObservers: () => [AppNavigatorObserver()],
              ),
              routeInformationParser: appRouter.defaultRouteParser(),
              title: Constant.materialAppTitle,
              color: Constant.taskMenuMaterialAppColor,
              themeMode: ThemeMode.light,
              theme: lightTheme,
              darkTheme: darkTheme,
              debugShowCheckedModeBanner: kDebugMode,
              localeResolutionCallback: (Locale? locale, Iterable<Locale> supportedLocales) =>
                  supportedLocales.map((e) => e.languageCode).contains(locale?.languageCode)
                      ? locale
                      : const Locale('ja'),
              locale:
                  Config.enableDevicePreview ? DevicePreview.locale(context) : const Locale('ja'),
              supportedLocales: AppLocaleUtils.supportedLocales,
              localizationsDelegates: const [
                GlobalMaterialLocalizations.delegate,
                GlobalWidgetsLocalizations.delegate,
                GlobalCupertinoLocalizations.delegate,
              ],
            ),
          ),
        );
      },
    );
  }

  List<PageRouteInfo> _mapRouteToPageRouteInfo() {
    return [const SplashRoute()];
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/my_app.dart](../../base_flutter/lib/ui/my_app.dart)

> 🔎 **Quan sát**
> - `MyApp extends HookConsumerWidget` — dùng hooks + Riverpod trong app
> - `MaterialApp.router()` — router-based navigation (auto_route)
> - Key params:
>   - `routerDelegate` — xử lý navigation logic
>   - `routeInformationParser` — parse URL/deep link
>   - `theme` / `darkTheme` — Material theming
>   - `localizationsDelegates` — i18n setup
> - `DevicePreview` wrapper — preview app trên nhiều device sizes
> - **Hỏi:** `HookConsumerWidget` khác gì `StatelessWidget`? Tại sao dùng nó thay vì `StatelessWidget`?

> 💡 **FE Perspective**
> **Flutter:** `MaterialApp` = root widget chứa theme, routing, localization config. Tương đương `<AppProvider><RouterProvider><ThemeProvider><IntlProvider><App /></AppProvider></RouterProvider></ThemeProvider></IntlProvider>` trong React.
> **React/Vue tương đương:** React Router v6 `<BrowserRouter>` + MUI `<ThemeProvider>` + i18next `<I18nProvider>` combined.
> **Khác biệt quan trọng:** Flutter có 1 `MaterialApp` duy nhất, không nested như React providers.

---

## 3. common_scaffold.dart — Shared Layout Wrapper

<!-- AI_VERIFY: base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart#L1-L93 -->
```dart
// ignore_for_file: missing_golden_test
import 'package:flutter/material.dart';

import '../../../index.dart';

class CommonScaffold extends StatelessWidget {
  const CommonScaffold({
    required this.body,
    this.appBar,
    this.floatingActionButton,
    this.floatingActionButtonLocation,
    this.drawer,
    this.endDrawer,
    this.backgroundColor,
    this.hideKeyboardWhenTouchOutside = true,
    this.shimmerEnabled = false,
    this.showBanner = true,
    this.useSafeArea = true,
    this.enabledEdgeToEdge = false,
    this.scaffoldKey,
    this.isLoading,
    this.preventActionWhenLoading = true,
  });

  final Widget? body;
  final PreferredSizeWidget? appBar;
  final Widget? drawer;
  final Widget? endDrawer;
  final Widget? floatingActionButton;
  final FloatingActionButtonLocation? floatingActionButtonLocation;
  final Color? backgroundColor;
  final bool hideKeyboardWhenTouchOutside;
  final bool shimmerEnabled;
  final bool showBanner;
  final Key? scaffoldKey;
  final bool useSafeArea;
  final bool enabledEdgeToEdge;
  final bool? isLoading;
  final bool preventActionWhenLoading;

  @override
  Widget build(BuildContext context) {
    final effectiveIsLoading = isLoading ?? LoadingStateProvider.isLoadingOf(context);
// ignore: prefer_common_widgets
    final scaffold = Scaffold(
      key: scaffoldKey,
      endDrawer: endDrawer,
      backgroundColor: backgroundColor ?? Colors.white,
      body: IgnorePointer(
        ignoring: preventActionWhenLoading && effectiveIsLoading,
        child: SafeArea(
          top: enabledEdgeToEdge ? false : useSafeArea,
          bottom: enabledEdgeToEdge ? false : useSafeArea,
          left: enabledEdgeToEdge ? false : useSafeArea,
          right: enabledEdgeToEdge ? false : useSafeArea,
          child: shimmerEnabled ? Shimmer(child: body) : body ?? const SizedBox.shrink(),
        ),
      ),
      appBar: appBar,
      drawer: drawer,
      floatingActionButton: floatingActionButton,
      floatingActionButtonLocation: floatingActionButtonLocation,
    );
    // ... banner logic
    return hideKeyboardWhenTouchOutside
        ? GestureDetector(
            onTap: () => ViewUtil.hideKeyboard(context),
            child: scaffoldWithBanner,
          )
        : scaffoldWithBanner;
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/component/common_scaffold/common_scaffold.dart](../../base_flutter/lib/ui/component/common_scaffold/common_scaffold.dart)

> 🔎 **Quan sát**
> - `CommonScaffold` là **wrapper** quanh `Scaffold` — thêm common behavior
> - Key features:
>   - `SafeArea` — tránh content bị che bởi notch/system bars
>   - `IgnorePointer` — prevent interaction khi loading
>   - `GestureDetector` — dismiss keyboard on tap outside
>   - `Shimmer` — loading placeholder effect
> - Đây là **composition pattern** — wrap widget thay vì extend
> - **Hỏi:** Tại sao dùng composition (wrap) thay vì extend `Scaffold`?

> 💡 **FE Perspective**
> **Flutter:** `CommonScaffold` = shared layout component dùng ở mọi page. Tương đương React `<Layout>` component với header/footer/nav slots.
> **React/Vue tương đương:** `<div class="app-layout"><header /><main>{children}</main><footer /></div>` với CSS layout.
> **Khác biệt quan trọng:** Flutter dùng widget composition, React dùng component + CSS. Flutter có built-in `Scaffold` widget với standard app layout structure.

---

## 4. splash_page.dart — Minimal Page Skeleton

<!-- AI_VERIFY: base_flutter/lib/ui/page/splash/splash_page.dart#L1-L37 -->
```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';
import 'package:flutter_hooks/flutter_hooks.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

import '../../../index.dart';

@RoutePage()
class SplashPage extends BasePage<SplashState,
    AutoDisposeStateNotifierProvider<SplashViewModel, CommonState<SplashState>>> {
  const SplashPage({super.key});

  @override
  ScreenViewEvent get screenViewEvent => ScreenViewEvent(screenName: ScreenName.splashPage);

  @override
  AutoDisposeStateNotifierProvider<SplashViewModel, CommonState<SplashState>> get provider =>
      splashViewModelProvider;

  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    useEffect(
      () {
        Future.microtask(() {
          ref.read(provider.notifier).init();
        });

        return null;
      },
      [],
    );

    return const CommonScaffold(
      body: SizedBox(),
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/splash/splash_page.dart](../../base_flutter/lib/ui/page/splash/splash_page.dart)

> 🔎 **Quan sát**
> - `@RoutePage()` — auto_route annotation để generate navigation routes
> - `extends BasePage<S, P>` — base class cho mọi page (lifecycle, error handling)
> - `provider` getter — Riverpod provider cho ViewModel
> - `screenViewEvent` getter — analytics tracking
> - `useEffect()` — hooks cho side effects (init logic)
> - `CommonScaffold` — shared layout wrapper
> - **Hỏi:** Tại sao `init()` được gọi trong `Future.microtask()` thay vì trực tiếp?

> 💡 **FE Perspective**
> **Flutter:** Page structure = `@RoutePage()` + `BasePage` + `useEffect()` + `CommonScaffold`. Tương đương React page component với `useEffect` cho init.
> **React/Vue tương đương:** `const Page = () => { useEffect(() => { init(); }, []); return <Layout><Content /></Layout> }`.
> **Khác biệt quan trọng:** Flutter có `@RoutePage()` annotation cho code-gen routes, React dùng React Router config file.

---

## 5. login_page.dart — Form UI Example

<!-- AI_VERIFY: base_flutter/lib/ui/page/login/login_page.dart#L1-L157 -->
```dart
@RoutePage()
class LoginPage extends BasePage<LoginState,
    AutoDisposeStateNotifierProvider<LoginViewModel, CommonState<LoginState>>> {
  const LoginPage({super.key});

  @override
  ScreenViewEvent get screenViewEvent => ScreenViewEvent(screenName: ScreenName.loginPage);

  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    final scrollController = useScrollController();

    return CommonScaffold(
      body: Stack(
        children: [
          CommonImage.asset(
            path: image.imageBackground,
            width: double.infinity,
            height: double.infinity,
            fit: BoxFit.cover,
          ),
          CommonScrollbarWithIosStatusBarTapDetector(
            routeName: LoginRoute.name,
            controller: scrollController,
            child: SingleChildScrollView(
              controller: scrollController,
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const SizedBox(height: 100),
                  CommonText(
                    l10n.login,
                    style: style(
                      fontSize: 30,
                      fontWeight: FontWeight.w700,
                      color: color.black,
                    ),
                  ),
                  const SizedBox(height: 50),
                  PrimaryTextField(
                    title: l10n.email,
                    hintText: l10n.email,
                    onChanged: (email) => ref.read(provider.notifier).setEmail(email),
                    keyboardType: TextInputType.text,
                    suffixIcon: const Icon(Icons.email),
                  ),
                  const SizedBox(height: 24),
                  PrimaryTextField(
                    title: l10n.password,
                    hintText: l10n.password,
                    onChanged: (password) => ref.read(provider.notifier).setPassword(password),
                    keyboardType: TextInputType.visiblePassword,
                    onEyeIconPressed: (obscureText) {
                      ref
                          .read(analyticsHelperProvider)
                          ._logEyeIconClickEvent(obscureText: obscureText);
                    },
                  ),
                  // error display
                  Consumer(
                    builder: (context, ref, child) {
                      final onPageError = ref.watch(
                        provider.select((value) => value.data.onPageError),
                      );
                      return Visibility(
                        visible: onPageError.isNotEmpty,
                        child: Padding(
                          padding: const EdgeInsets.only(top: 16),
                          child: CommonText(
                            onPageError,
                            style: style(fontSize: 14, color: color.red1),
                          ),
                        ),
                      );
                    },
                  ),
                  // login button
                  Consumer(
                    builder: (context, ref, child) {
                      final isLoginButtonEnabled = ref.watch(
                        provider.select((value) => value.data.isLoginButtonEnabled),
                      );
                      return ElevatedButton(
                        onPressed: isLoginButtonEnabled
                            ? () {
                                ref.read(analyticsHelperProvider)._logLoginButtonClickEvent();
                                ref.read(provider.notifier).login();
                              }
                            : null,
                        // ... button styling
                      );
                    },
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/login/login_page.dart](../../base_flutter/lib/ui/page/login/login_page.dart)

> 🔎 **Quan sát**
> - Widget tree structure:
>   - `CommonScaffold` → `body: Stack` → children
>   - `Stack` chứa: background image + scrollable content
>   - `Column` chứa: text + text fields + button
> - Layout widgets: `Stack`, `Column`, `Row`, `SizedBox`, `Padding`
> - `Consumer` widget — Riverpod để watch state và rebuild UI
> - `ref.watch(provider.select(...))` — selective rebuild, chỉ rebuild khi specific field thay đổi
> - **Hỏi:** Tại sao phải wrap `Consumer` quanh `ElevatedButton`? Không watch thì sao?

> 💡 **FE Perspective**
> **Flutter:** Login page widget tree = `Scaffold > Stack > Column > TextField/Button`. Tương đương React: `<form><input /><button>`. Flutter layout widgets ≈ CSS flexbox.
> **React/Vue tương đương:** `<div class="login-form"><input v-model="email" /><button @click="login">Login</button></div>`.
> **Khác biệt quan trọng:** Flutter layout dùng widget composition (`Column`, `Row`), React dùng CSS (`display: flex`). Flutter declarative tree vs React JSX syntax.

---

## 6. main_page.dart — Tab Navigation

<!-- AI_VERIFY: base_flutter/lib/ui/page/main/main_page.dart#L1-L94 -->
```dart
@RoutePage()
class MainPage extends BasePage<MainState,
    AutoDisposeStateNotifierProvider<MainViewModel, CommonState<MainState>>> {
  MainPage({super.key});

  @override
  ScreenViewEvent get screenViewEvent => ScreenViewEvent(screenName: ScreenName.mainPage);

  @override
  Widget buildPage(BuildContext context, WidgetRef ref) {
    useEffect(
      () {
        Future.microtask(() {
          ref.read(provider.notifier).init();
        });

        return () {};
      },
      [],
    );

    return AutoTabsScaffold(
      routes: ref.read(appNavigatorProvider).tabRoutes,
      bottomNavigationBuilder: (_, tabsRouter) {
        ref.read(appNavigatorProvider).tabsRouter = tabsRouter;

        return BottomNavigationBar(
          currentIndex: tabsRouter.activeIndex,
          onTap: (index) {
            if (index == tabsRouter.activeIndex) {
              ref.read(appNavigatorProvider).popUntilRootOfCurrentBottomTab();
            }
            tabsRouter.setActiveIndex(index);
          },
          showSelectedLabels: true,
          showUnselectedLabels: true,
          unselectedItemColor: Colors.grey,
          selectedItemColor: color.black,
          type: BottomNavigationBarType.fixed,
          backgroundColor: Colors.white,
          items: BottomTab.values
              .map(
                (tab) => BottomNavigationBarItem(
                  label: tab.title,
                  icon: tab.icon,
                  activeIcon: tab.activeIcon,
                ),
              )
              .toList(),
        );
      },
    );
  }
}
```
<!-- END_VERIFY -->

→ [Mở file gốc: lib/ui/page/main/main_page.dart](../../base_flutter/lib/ui/page/main/main_page.dart)

> 🔎 **Quan sát**
> - `AutoTabsScaffold` — auto_route's tab scaffold
> - `routes` — list of tab routes (HomeTab, MyProfileTab)
> - `bottomNavigationBuilder` — render `BottomNavigationBar`
> - `tabsRouter` — manages active tab, nested navigation stacks
> - `onTap` handler: switch tab + pop to root nếu tap active tab
> - **Hỏi:** `maintainState: true` trên tab routes có nghĩa gì? Tab state được giữ khi switch không?

> 💡 **FE Perspective**
> **Flutter:** Tab navigation với `AutoTabsScaffold` + `BottomNavigationBar`. Tương đương React `<Tab><TabPanel>` với state management.
> **React/Vue tương đương:** React Router v6 `<Tabs><Tab value="home"><TabPanel>...</TabPanel></Tab>`.
> **Khác biệt quan trọng:** Flutter tab có **nested navigation stacks** — mỗi tab giữ navigation history riêng. React tabs thường reset khi switch.

---

## Tổng kết Walk

| Layer | File | Vai trò |
|-------|------|---------|
| Entry point | `main.dart` | `runApp()` bootstrap, error handling |
| App shell | `my_app.dart` | `MaterialApp.router`, theme, localization |
| Layout wrapper | `common_scaffold.dart` | SafeArea, keyboard dismiss, shimmer |
| Pages | `splash_page.dart`, `login_page.dart`, `main_page.dart` | Concrete page examples |
| Navigation | `app_navigator.dart` | Navigation patterns (M7-M10) |

### Widget Tree Anatomy

```
runApp(MyApp)
    ↓
MaterialApp.router
    ↓
TranslationProvider > DevicePreview > MyApp child
    ↓
AutoTabsRouter / Page widgets
    ↓
CommonScaffold
    ↓
Scaffold
    ↓
AppBar + body + floatingActionButton + ...
```

> ⏭️ **Forward:** Layout widgets (Column, Row, Container) và built-in widgets sẽ được deep dive trong [Module 5 — Built-in Widgets](../module-05-built-in-widgets/).

<!-- AI_VERIFY: generation-complete -->
