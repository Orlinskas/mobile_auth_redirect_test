# Тестовый провайдер авторизации для мобильных приложений

https://orlinskas.github.io/mobile_auth_redirect_test/

Статический веб-сайт, размещенный на GitHub Pages, предназначенный для тестирования **Android App Links** и **iOS Universal Links**. Этот тестовый провайдер авторизации имитирует процесс входа и перенаправляет пользователей обратно в ваше мобильное приложение с использованием глубоких ссылок.

## Демонстрация

После развертывания, доступ к вашему тестовому провайдеру авторизации по адресу:
```
https://<your-username>.github.io/mobile_auth_redirect_test/
```

## Структура проекта

```
mobile_auth_redirect_test/
├── index.html                           # Главная страница входа с кнопкой авторизации
├── callback.html                        # Страница обратного вызова (имитирует редирект)
├── .well-known/
│   ├── assetlinks.json                 # Конфигурация Android Digital Asset Links
│   └── apple-app-site-association      # Конфигурация iOS Universal Links
├── .nojekyll                           # Обеспечивает корректную отдачу папки .well-known
└── README.md                           # Этот файл
```

## Конфигурация

### Настройка Android (Digital Asset Links)

1. **Сгенерируйте SHA-256 отпечаток:**
   ```bash
   # Для debug keystore
   keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
   
   # Для release keystore
   keytool -list -v -keystore /path/to/your/release.keystore -alias your-key-alias
   ```

2. **Обновите `.well-known/assetlinks.json`:**
   ```json
   [
     {
       "relation": ["delegate_permission/common.handle_all_urls"],
       "target": {
         "namespace": "android_app",
         "package_name": "com.yourcompany.yourapp",
         "sha256_cert_fingerprints": [
           "YOUR_DEBUG_SHA256_FINGERPRINT",
           "YOUR_RELEASE_SHA256_FINGERPRINT"
         ]
       }
     }
   ]
   ```

3. **В `AndroidManifest.xml` вашего Android приложения:**
   ```xml
   <activity android:name=".MainActivity">
       <intent-filter android:autoVerify="true">
           <action android:name="android.intent.action.VIEW" />
           <category android:name="android.intent.category.DEFAULT" />
           <category android:name="android.intent.category.BROWSABLE" />
           <data
               android:scheme="https"
               android:host="<your-username>.github.io"
               android:pathPrefix="/mobile_auth_redirect_test/callback" />
       </intent-filter>
   </activity>
   ```

### Настройка iOS (Universal Links)

1. **Получите ваш Team ID:**
   - Войдите в [Apple Developer Portal](https://developer.apple.com/account/)
   - Ваш Team ID отображается в правом верхнем углу страницы Membership

2. **Обновите `.well-known/apple-app-site-association`:**
   ```json
   {
     "applinks": {
       "apps": [],
       "details": [
         {
           "appID": "YOUR_TEAM_ID.com.yourcompany.yourapp",
           "paths": [
             "/mobile_auth_redirect_test/*",
             "/callback"
           ]
         }
       ]
     }
   }
   ```

3. **В Xcode проекте вашего iOS приложения:**
   - Перейдите в **Signing & Capabilities**
   - Добавьте возможность **Associated Domains**
   - Добавьте домен: `applinks:<your-username>.github.io`

4. **Обработайте Universal Links в вашем приложении:**
   ```swift
   // В вашем AppDelegate или SceneDelegate
   func application(_ application: UIApplication,
                    continue userActivity: NSUserActivity,
                    restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
       guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
             let url = userActivity.webpageURL else {
           return false
       }
       
       // Обработайте URL
       print("Received Universal Link: \(url)")
       // Извлеките параметры token и status
       
       return true
   }
   ```

## Развертывание на GitHub Pages

### Шаг 1: Создание репозитория

1. Перейдите на [GitHub](https://github.com) и создайте новый репозиторий с именем `mobile_auth_redirect_test`
2. Сделайте его публичным (требуется для GitHub Pages на бесплатных аккаунтах)

### Шаг 2: Отправка кода

```bash
# Инициализируйте git репозиторий (если еще не сделано)
git init

# Добавьте все файлы
git add .

# Сделайте коммит
git commit -m "Initial commit: Mock auth provider for mobile app testing"

# Добавьте remote (замените <your-username> на ваше имя пользователя GitHub)
git remote add origin https://github.com/<your-username>/mobile_auth_redirect_test.git

# Отправьте на GitHub
git branch -M main
git push -u origin main
```

### Шаг 3: Включение GitHub Pages

1. Перейдите в ваш репозиторий на GitHub
2. Нажмите **Settings** → **Pages**
3. В разделе **Source** выберите ветку `main` и папку `/ (root)`
4. Нажмите **Save**
5. Ваш сайт будет опубликован по адресу: `https://<your-username>.github.io/mobile_auth_redirect_test/`

### Шаг 4: Проверка конфигурационных файлов

После развертывания проверьте доступность следующих URL:

**Android:**
```
https://<your-username>.github.io/mobile_auth_redirect_test/.well-known/assetlinks.json
```

**iOS:**
```
https://<your-username>.github.io/mobile_auth_redirect_test/.well-known/apple-app-site-association
```

## Тестирование

### Тестирование на Android

1. **Установите ваше приложение** на Android устройство (рекомендуется физическое устройство)
2. **Проверьте верификацию App Link:**
   ```bash
   # Проверьте, верифицированы ли App Links
   adb shell pm get-app-links com.yourcompany.yourapp
   ```
3. **Откройте URL в Chrome или отправьте через SMS/Email:**
   ```
   https://<your-username>.github.io/mobile_auth_redirect_test/
   ```
4. Нажмите "Authorize and Return to App"
5. Ваше приложение должно открыться автоматически с callback URL

### Тестирование на iOS

1. **Установите ваше приложение** на iOS устройство (требуется физическое устройство - симулятор не полностью поддерживает Universal Links)
2. **Протестируйте Universal Link:**
   - Отправьте себе URL через Сообщения, Email или Заметки
   - **Важно:** Не тестируйте вставкой в Safari - это не активирует Universal Links
3. Нажмите на ссылку в Сообщениях/Email/Заметках
4. Ваше приложение должно открыться автоматически

### Отладка

**Android:**
```bash
# Просмотр логов для App Links
adb logcat | grep -i "AppLinks"

# Ручная верификация домена
adb shell pm verify-app-links --re-verify com.yourcompany.yourapp
```

**iOS:**
```bash
# Тестирование AASA файла локально (macOS)
curl -i https://<your-username>.github.io/.well-known/apple-app-site-association

# Проверка валидности JSON
curl https://<your-username>.github.io/.well-known/apple-app-site-association | python -m json.tool
```

## Как это работает

1. **Пользователь открывает тестовый провайдер авторизации** в мобильном браузере
2. **Пользователь нажимает кнопку "Authorize and Return to App"**
3. **JavaScript перенаправляет** используя `window.location.href` на callback URL:
   ```
   https://<your-username>.github.io/mobile_auth_redirect_test/callback?token=mock_token_2026&status=success
   ```
4. **Мобильная ОС перехватывает** URL, если ваше приложение установлено и правильно настроено
5. **Приложение открывается** и получает URL с параметрами авторизации

## Важные замечания

### Файл `.nojekyll`
GitHub Pages по умолчанию использует Jekyll, который игнорирует папки, начинающиеся с точки (например, `.well-known`). Файл `.nojekyll` в корне отключает обработку Jekyll, обеспечивая корректную отдачу вашей папки `.well-known`.

### Действие, инициированное пользователем
Перенаправление ДОЛЖНО быть запущено пользовательским действием (нажатием кнопки) с использованием `window.location.href`. Мобильные браузеры требуют этого для активации глубоких ссылок по соображениям безопасности.

### Только HTTPS
И Android App Links, и iOS Universal Links требуют HTTPS. GitHub Pages предоставляет это автоматически.

### Заголовок Content-Type
- Android требует, чтобы `assetlinks.json` отдавался как `application/json`
- iOS требует, чтобы `apple-app-site-association` отдавался как `application/json` (без расширения `.json`)
- GitHub Pages обрабатывает это автоматически

## Решение проблем

| Проблема | Решение |
|-------|----------|
| Файлы `.well-known` возвращают 404 | Убедитесь, что файл `.nojekyll` существует в корне |
| Приложение не открывается на Android | Проверьте, что SHA-256 отпечаток совпадает с вашим keystore |
| Приложение не открывается на iOS | Тестируйте нажатием на ссылки в Сообщениях, а не в адресной строке Safari |
| Universal Links не работают | Проверьте, что Associated Domains в Xcode включает префикс `applinks:` |
| Digital Asset Links не работают | Проверьте, что `package_name` точно совпадает с пакетом вашего приложения |

## Полезные ресурсы

- [Документация Android App Links](https://developer.android.com/training/app-links)
- [Документация iOS Universal Links](https://developer.apple.com/ios/universal-links/)
- [Документация GitHub Pages](https://docs.github.com/en/pages)
- [Тестер Digital Asset Links](https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://<your-domain>&relation=delegate_permission/common.handle_all_urls)


