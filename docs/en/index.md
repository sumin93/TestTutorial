# Screenshot-тесты EN

В этом уроке мы научимся писать screenshot-тесты, узнаем, зачем они нужны, и как правильно разрабатывать приложение, чтобы его можно было покрыть тестами.

## Продвинутый уровень

Для успешного прохождения предыдущих уроков было достаточно базовых навыков программирования на Kotlin, знания Android разработки при этом не требовались, и успешно пройти все уроки могли как разработчики, так и тестировщики. Но для нашей сегодняшней темы, а также всех последующих, нужно понимание того, как разрабатываются приложения, чем отличаются архитектурные шаблоны MVVM и MVP, как применять Dependency Injection и другое.

Поэтому предполагается, что все дальнейшие действия (или бОльшую их часть), которые мы будем проходить в курсе, находятся в зоне ответственности разработчиков, и эти уроки ориентированы на них. Если же с Android разработкой вы не знакомы, то можете все равно проходить эти уроки, чтобы иметь представление о возможностях Kaspresso, но учитывайте тот факт, что часть материала может быть непонятной.

## Тестирование LoginActivity на разных локалях

Чтобы узнать, зачем нужны скриншот-тесты, давайте разберем небольшой пример. Представим, что наше приложение должно быть локализовано на французский язык. Для этого в проекте были добавлены переводы в файл `strings.xml` в папку `values-fr`.

Давайте установим на устройстве французский язык

// screenshots

и запустим LoginActivityTest. Тест пройден успешно, значит теоретически это приложение рабочее, и его можно раскатывать на пользователей. Но давайте откроем `LoginActivity` вручную (французский язык должен быть установлен на устройстве) и посмотрим, как выглядит этот экран.

// screenshots

Видим, что вместо корректных текстов здесь указано «TODO: add french locale». Похоже, что разработчики во время добавления строк оставили комментарий, чтобы добавить переводы в будущем, но забыли это сделать, поэтому приложение выглядит некорректно. Обнаружить эту ошибку тесты не могут, потому что они не знают, какой должен быть текст на французском языке. По этой причине приложение работает неправильно, но тесты проходят успешно.

## Screenshot-тесты, как решение проблемы со строками

Эту проблему могут решить скриншот-тесты. Их суть заключается в том, что для всех экранов, где пользователю отображаются строки, создаются так называемые «скриншотилки» – классы, которые делают скриншоты экрана во всех необходимых состояниях и для всех поддерживаемых языков.

После выполнения таких тестов все скриншоты складываются в определенные папки. Тогда люди, ответственные за переводы и строки, смогут просмотреть снимки и убедиться, что для всех локалей использованы корректные значения.

Screenshot-тесты будут отличаться от тестов, которые мы писали ранее.

Во-первых, нас интересуют только строки на определенном экране, поэтому нет необходимости проходить весь процесс от старта приложения до открытия нужного экрана. Вместо этого, в тесте мы сразу будем открывать [Activity]( https://developer.android.com/reference/android/app/Activity) или [Fragment]( https://developer.android.com/guide/fragments), скриншоты которого мы хотим получить.

Во-вторых, мы хотим только получить снимки всех возможных состояний экрана для каждой локали, поэтому добавлять проверки элементов или выполнять шаги, имитирующие действия пользователя, как мы делали ранее, мы не будем. Наша цель – открыть экран, установить нужное состояние, сделать скриншот. Затем при необходимости изменить состояние и снова сделать скриншот. Дальше нужно поменять локаль и повторить все перечисленные действия.

Подробнее про состояния (или стейты, как их часто называют) мы поговорим позже, а сейчас напишем простой screenshot-тест, который откроет экран LoginActivity, сделает скриншот, затем сменит язык на устройстве на французский и снова сделает скриншот.

## Простой screenshot-тест

Создание screenshot-теста начинается так же, как мы делали ранее – в папке тестов создаем новый класс. Классы для скриншотов обычно называются с окончанием `Screenshots`. Давайте все скриншот-тесты будем хранить в отдельном пакете, назовем его screenshot_tests.

// create package

В этом пакете создаем класс `LoginActivityScreenshots`

// create class

У тестов, которые мы сейчас будем создавать есть особенности: во-первых, они должны запускаться для разных локалей, во-вторых, полученные скриншоты должны быть размещены в удобной структуре папок – для каждого языка своя папка. По этим причинам тестовый класс мы унаследуем от класса `DocLocScreenshotTestCase`, а не от `TestCase`, как мы это делали ранее

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase

class LoginActivityScreenshots : DocLocScreenshotTestCase() {

}

```

В качестве параметра конструктору нужно передать список локалей, для которых будут делаться скриншоты. В данном случае нас интересует английский и французский языки, устанавливаем их. Делается это следующим образом:

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "en, fr") {

}

```
Вторым параметром необходимо передать значение типа Boolean, которое определяет, нужно менять язык на устройстве или нет. Нам необходимо менять язык на устройстве, поэтому мы передаем `changeSystemLocale = true`

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "fr,en", changeSystemLocale = true) {

}
```
Для того чтобы тестовое приложение могло изменять конфигурацию устройства (установленную локаль), необходимо в манифесте дать соответствующее разрешение. Открываем манифест приложения tutorial и добавляем разрешение на смену конфигурации `<uses-permission android:name="android.permission.CHANGE_CONFIGURATION" tools:ignore="ProtectedPermissions"/>`

Теперь наш тест будет запущен для каждой локали, которую мы передали в качестве параметра.

Как мы говорили ранее, здесь мы не будем проходить весь процесс от старта приложения до открытия необходимого экрана. Вместо этого мы сразу создадим `Rule`, в котором укажем, что при старте теста должен быть открыт экран `LoginActivity`

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import androidx.test.ext.junit.rules.activityScenarioRule
import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase
import com.kaspersky.kaspresso.tutorial.login.LoginActivity
import org.junit.Rule

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "fr,en", changeSystemLocale = true) {

    @get:Rule
    val activityRule = activityScenarioRule<LoginActivity>()
}
```

В этом классе мы можем использовать все методы, которые использовали в других тестах. Давайте создадим один step, в котором проверим только исходное состояние экрана. Назовем метод takeScreenshots()

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import androidx.test.ext.junit.rules.activityScenarioRule
import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase
import com.kaspersky.kaspresso.tutorial.login.LoginActivity
import org.junit.Rule
import org.junit.Test

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "fr,en", changeSystemLocale = true) {

    @get:Rule
    val activityRule = activityScenarioRule<LoginActivity>()

    @Test
    fun takeScreenshots() = run {
        step("Take screenshots initial state") {

        }
    }
}

```

Для того чтобы сделать скриншоты, и чтобы эти скриншоты были сохранены в правильные папки на устройстве, необходимо вызвать метод `captureScreenshot` и в качестве параметра передать название файла.

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import androidx.test.ext.junit.rules.activityScenarioRule
import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase
import com.kaspersky.kaspresso.tutorial.login.LoginActivity
import org.junit.Rule
import org.junit.Test

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "fr,en", changeSystemLocale = true) {

    @get:Rule
    val activityRule = activityScenarioRule<LoginActivity>()

    @Test
    fun takeScreenshots() = run {
        step("Take screenshots initial state") {
            captureScreenshot("Initial state")
        }
    }
}

```

Разрешение на доступ к файлам здесь давать не нужно, это реализовано «под капотом». На данном этапе мы сделали все, что нужно, чтобы получить скриншоты экрана и посмотреть, как выглядит приложение на разных локалях, но желательно сделать еще одно изменение.

Сейчас у нас открывается нужный экран, и сразу делается скриншот, поэтому есть вероятность, что какие-то данные на экране не успеют загрузиться, и снимок будет сделан до того, как мы увидим нужные нам элементы.

Чтобы решить эту проблему, давайте в Page Object `Login Screen` мы добавим метод, который дождется загрузки всех необходимых элементов интерфейса. В этом методе мы просто для всех объектов сделаем проверку на `isVisible`. Это проверка в своей реализации использует `flakySafely`, поэтому даже если данные мгновенно загружены не будут, то тест будет ждать, пока условие не выполнится в течение нескольких секунд.

Добавляем метод, назовем его `waitForScreen`:

```kotlin
package com.kaspersky.kaspresso.tutorial.screen

import com.kaspersky.kaspresso.screens.KScreen
import com.kaspersky.kaspresso.tutorial.R
import io.github.kakaocup.kakao.edit.KEditText
import io.github.kakaocup.kakao.text.KButton

object LoginScreen : KScreen<LoginScreen>() {

    override val layoutId: Int? = null
    override val viewClass: Class<*>? = null

    val inputUsername = KEditText { withId(R.id.input_username) }
    val inputPassword = KEditText { withId(R.id.input_password) }
    val loginButton = KButton { withId(R.id.login_btn) }

    fun waitForScreen() {
        inputUsername.isVisible()
        inputPassword.isVisible()
        loginButton.isVisible()
    }
}

```
В тестовом классе можем вызвать этот метод перед тем, как сделать скриншот:

```kotlin
package com.kaspersky.kaspresso.tutorial.screenshot_tests

import androidx.test.ext.junit.rules.activityScenarioRule
import com.kaspersky.kaspresso.testcases.api.testcase.DocLocScreenshotTestCase
import com.kaspersky.kaspresso.tutorial.login.LoginActivity
import com.kaspersky.kaspresso.tutorial.screen.LoginScreen
import org.junit.Rule
import org.junit.Test

class LoginActivityScreenshots : DocLocScreenshotTestCase(locales = "fr,en", changeSystemLocale = true) {

    @get:Rule
    val activityRule = activityScenarioRule<LoginActivity>()

    @Test
    fun takeScreenshots() = run {
        step("Take screenshots initial state") {
            LoginScreen {
                waitForScreen()
                captureScreenshot("Initial state")
            }
        }
    }
}
```

Запускаем тест. Тест пройден успешно в `Device File Explorer` в папке `sdcard/Documents/screenshots` вы сможете найти все скриншоты, при этом для каждой локали была создана своя папка и вы сможете просмотреть, как выглядит ваше приложение на разных языках.

//  screenshots

Теперь, просмотрев скриншоты, можно увидеть проблему в приложении, что не все строки были добавлены корректно, и разработчик может исправить ошибку, добавив необходимые значения в файл `values-fr/strings.xml`.
