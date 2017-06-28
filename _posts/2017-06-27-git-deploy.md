Использование утилиты git-deploy для автоматического развертывания
==================================================================

Утилита git-deploy помогает в развертывании rails-приложений в production-средах. Нам она
понадобится для развертывания плагинов к redmine. 

Подготовка
----------

Подразумевается, что у вас уже установлен redmine на своей машине.

Для начала нужно установить утилиту.

   sudo gem install git-deploy

После этого нужно настроить плагин для работы `git-deploy`. Предположим, что:

 * redmine установлен в каталоге `/home/user/redmine` 
 * мы хотим развернуть плагин для редмайна, который лежит в каталоге `/home/user/redmine/plugins/myplugin`

тогда для настройки нам нужно перейти в каталог плагина

     cd /home/user/redmine/plugins/myplugin

инициализировать там git-репозиторий (этот шаг можно пропустить если репозиторий уже есть)

    git init 
    git add .
    git commit -am "1"

Добавить в `git remotes` путь до production-среды

    git remote add production username@product.server:/path/to/redmine/plugins/myplugin

(в случае нашей лаборатории будет что-то вроде katya@redmine.ksa:/usr/share/redmine/plugins/myplugin)

после чего инициализировать git-deploy
    
    git deploy setup -r production

Далее следует создать необходимые для работы `git-deploy` каталоги

    git deploy init
    mkdir log
    touch log/.placeholder

после чего отредактировать файл `deploy/restart` следующим образом

    #!/bin/sh
    echo "restarting Passenger app"
    sudo service apache2 reload

после чего, в свою очередь, закомиттить изменения

    git add deploy
    git add log/.placeholder
    git commit -am "Git deploy hooks"

и отправить их в production-среду

    git push production master

При этом в выводе git должно содержаться что-то вроде

    remote: HEAD is now at 955d11b Log dir
    remote: files changed: 1
    remote: restarting Passenger app

Дальнейшая работа
-----------------

После того как плагин настроен должным образом, дальнейшая работа сводится к обычной работе с git 
с тем отличием, что при необходимости обновления production-среды нужно просто туда отправить измеенения 

    git push production master

перезагрузка Redmine произойдет при этом автоматически.

В случае если `git-deploy` в плагине уже настроен другим разработчиком, нужно загрузить его с сервера

    git pull

и настроить путь до produciton-среды 

    git remote add production username@product.server:/path/to/redmine/plugins/myplugin

после чего все готово к работе.