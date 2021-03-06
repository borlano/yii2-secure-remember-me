# Yii2 Secure RememberMe

**Secure RememberMe** - расширение для фреймворка Yii2, реализующее безопасный способ RememberMe-аутентификации по токену в Cookie.

## Установка

Используя Composer:

``` bash
$ composer require "borlano/yii2-secure-remember-me"
```

## Миграции

Выполнить в консоли:

``` bash
$ php yii migrate --migrationPath=@vendor/borlano/yii2-secure-remember-me/source/migrations
```

## Настройка

Прописать в конфигурации config/main.php:

``` php
    'bootstrap' => [

        /* ... */

        'rememberMe', # Добавляем к загрузочным компонентам приложения
    ],
    
    'components' => [
        'rememberMe' => [ # Добавляем компонент в приложение
            'class' => 'iiifx\yii2\SecureRememberMe\components\Manager',
            # Конфигурация компонента
            'cookieKey' => 'remember-me', # Имя Cookie параметра
            'domain' => 'example.com', # Указываем домен, к которому привязывается Cookie
            'lifetime' => 2592000, # Время жизни токена, в секундах
            'userClass' => 'common\models\User', # Класс пользователя, который используется в приложении
        ],
        
        /* ... */
        
    ],
```

## Использование

### Пример подключения расширения к **[Yii2 Advanced Template](https://github.com/yiisoft/yii2-app-advanced)**.

Вносим изменения в логику входа([LoginForm::login()](https://github.com/yiisoft/yii2-app-advanced/blob/master/common/models/LoginForm.php#L56)):

``` php
    /**
     * Logs in a user using the provided username and password.
     *
     * @return boolean whether the user is logged in successfully
     */
    public function login ()
    {
        if ( $this->validate() ) {
            if ( Yii::$app->user->login( $this->getUser() ) ) {

                # Если разрешен RememberMe пользователем
                if ( $this->rememberMe ) {
                    /** @var \iiifx\yii2\SecureRememberMe\components\Manager $rememberMe */
                    $rememberMe = Yii::$app->rememberMe;
                    # Создаем токен для пользователя
                    $rememberMe->create( $this->_user->id );
                }
                return TRUE;
                
            }
        }
        return FALSE;
    }
```

И добавляем удаление токена при выходе([SiteController::actionLogout()](https://github.com/yiisoft/yii2-app-advanced/blob/master/frontend/controllers/SiteController.php#L104)):

``` php
    /**
     * Logs out the current user.
     *
     * @return mixed
     */
    public function actionLogout()
    {
        Yii::$app->user->logout();

        /** @var \iiifx\yii2\SecureRememberMe\components\Manager $rememberMe */
        $rememberMe = Yii::$app->rememberMe;
        $rememberMe->delete();

        return $this->goHome();
    }
```

Проверка, аутентификация по токену и его регенерация происходит автоматически.

Все. Компонент готов к работе.

## Лицензия

[![Software License][ico-license]](LICENSE.md)







[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg
