# Raspberry Pi Telegram Bot

## Telegram бот на Raspberry Pi в Docker контейнере
```
По запросу показывает температуру с присоедиенного датчика DTH22
```

### Технологии
 - Docker, docker-compose
 - Python

### Установка и запуск
 - Регистрируем своего бота [здесь](https://core.telegram.org/bots#3-how-do-i-create-a-bot), запоминаем/копируем его токен и @имя.
 - Ставим docker и docker-compose (ищем в инете как)
 - Клонируем репозиторий в `/srv/tbot` - все конфиги заточены под эту директорию
 - Редактируем [Dockerfile](https://github.com/vsb2007/raspberry_telegram_bot/blob/master/Dockerfile) - можно все оптимизировать, чтобы было меньше слоев, 
я оставил так, чтобы было понятнее, что происходит.
 - Собираем свой образ `sh docker_build.sh`, преварительно поменяв в `docker-compose.yml` [vsb2007](https://github.com/vsb2007/raspberry_telegram_bot/blob/eb46c118f6f6fa0cabf7323a7100e22bac73e74f/docker-compose.yml#L5) 
и в `docker_build.sh` [vsb2007](https://github.com/vsb2007/raspberry_telegram_bot/blob/497bf655755e04479f1314706a1186c5d64d22d5/docker_build.sh#L3) на что-то свое
 - Открываем фаил [myconfig.py.sample](config/myconfig.py.sample), убираем расширение `.sample` и подставляем свои данные
 - Настраиваем nginx для домена (у меня есть домен, потому рассматриваю именно этот вариант)
    * Обязательно ssl (бесплатно [тут](https://letsencrypt.org/))
    * Секция
```
location / {
	include uwsgi_params;
	uwsgi_pass unix:/srv/tbot/socket/tbot.sock;
}
```
должна соответствовать [этой строчке](https://github.com/vsb2007/raspberry_telegram_bot/blob/f2904be2290ce14fd414bc5954cfbd771170c50a/app/tbot.ini#L7)

 - Далее
    * Если нет возможности создать домен 3 уровня специально для бота, то `location /bla-bla` синхронизируем с [этой строчкой](https://github.com/vsb2007/raspberry_telegram_bot/blob/9e39e42851f5435869b11303d87cabb4effee6f5/app/bot.py#L66),
    и остальными аналогично (а может и нет, я не пробовал :))
 - В теории все!!! 
    * В файле `bot.py` раскомментируем секцию [Set_webhook](https://github.com/vsb2007/raspberry_telegram_bot/blob/497bf655755e04479f1314706a1186c5d64d22d5/bot.py#L114)
для настройки `webhook` для нашего бота. 
    * запускаем бота `docker-compose up -d`
    * Заходим по адресу `https://tbot.example.net/set_webhook` - если не видим `ok` - смотрим логи - ищем ошибку.
    * После настройки обратно комментируем.
 - Добавляем бота в группу или общаемся напрямую с ним. В данном примере команда `/m70 temp` вернет
```
temp:  21.8
humidity: 25.5
```
или что-то похожее :-)


# git-lab-ci
 - нужен ранер на rpi (так как архитектура другая)
   * ставим как пишут [здесь](https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/install/linux-manually.md) и регистрируем в gitlab
   * правим конфиг до примерно [такого](./gitlab-runner/config.toml.example) состояния
   * добавляем узера gitlab-runner (ну или как вы его обозвали) в группу docker. - `$sudo usermod -aG docker gitlab-runner`
   * в настройках репозитория добавляем переменные `HUB_DOCKER_USERNAME` и `HUB_DOCKER_PASSWORD` (думаю смысл их понятен)
   * ставим на ноде `socat`
   * переводим ноду на подключение по ssh ключу
   * в настройках репозитория создаем переменную `SSH_PRIVATE_KEY` с соответствующим содержанием
