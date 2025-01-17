# ishetalveilig.nl

## Installation instructions

1. run `composer install`
2. run `php please make:user`
3. run `npm i` && `npm run watch`

The site is deployed to Netlify using the Statamic Static Site Generator. Due to Netlify limits the site has to runder on PHP 8.0

* Build command: composer build
* Publish directory: storage/app/static

The environment variables needed on Netlify are:
* APP_DEBUG
* APP_ENV
* APP_KEY
* APP_NAME
* APP_URL
* STATAMIC_LICENSE_KEY

## Environment file contents

### Development

```env
APP_NAME='Is het al veilig?'
APP_ENV=local
APP_KEY=base64:sv/WnEgkV2Mvp2ofr6CkvX8MYsFUx4SRDpmAjH0VZHI=
APP_DEBUG=true
APP_URL=http://ishetalveilig.test

DEBUGBAR_ENABLED=true

LOG_CHANNEL=stack

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_DATABASE=
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=0.0.0.0
MAIL_PORT=1025
MAIL_USERNAME=testuser
MAIL_PASSWORD=testpwd
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

IMAGE_MANIPULATION_DRIVER=imagick

STATAMIC_LICENSE_KEY=
STATAMIC_THEME=business

STATAMIC_API_ENABLED=false
STATAMIC_REVISIONS_ENABLED=false

STATAMIC_GIT_ENABLED=false
STATAMIC_GIT_PUSH=false
STATAMIC_GIT_DISPATCH_DELAY=5

SAVE_CACHED_IMAGES=true
STATAMIC_CACHE_TAGS_ENABLED=false
STATAMIC_STACHE_WATCHER=true
STATAMIC_STATIC_CACHING_STRATEGY=null

#STATAMIC_CUSTOM_CMS_NAME=
#STATAMIC_CUSTOM_LOGO_OUTSIDE_URL=
STATAMIC_CUSTOM_LOGO_OUTSIDE_URL='/visuals/nee.svg'
#STATAMIC_CUSTOM_FAVICON_URL=
#STATAMIC_CUSTOM_CSS_URL=
```

### Production

```env
APP_NAME='Is het al veilig?'
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=https://ishetalveilig.nl

DEBUGBAR_ENABLED=false

LOG_CHANNEL=stack

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=redis
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_DATABASE=
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.postmarkapp.com
MAIL_PORT=587
MAIL_ENCRYPTION=tls
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_FROM_ADDRESS=noreply@ishetalveilig.nl
MAIL_FROM_NAME="${APP_NAME}"

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

IMAGE_MANIPULATION_DRIVER=imagick

STATAMIC_LICENSE_KEY=
STATAMIC_THEME=business

STATAMIC_API_ENABLED=false
STATAMIC_REVISIONS_ENABLED=false

STATAMIC_GIT_ENABLED=true
STATAMIC_GIT_PUSH=true
STATAMIC_GIT_DISPATCH_DELAY=5

STATAMIC_STATIC_CACHING_STRATEGY=full
SAVE_CACHED_IMAGES=true
STATAMIC_STACHE_WATCHER=false
STATAMIC_CACHE_TAGS_ENABLED=true

#STATAMIC_CUSTOM_CMS_NAME=
STATAMIC_CUSTOM_LOGO_OUTSIDE_URL='/visuals/nee.svg'
#STATAMIC_CUSTOM_LOGO_NAV_URL=
#STATAMIC_CUSTOM_FAVICON_URL=
#STATAMIC_CUSTOM_CSS_URL=
```

## NGINX config

Add the following to your NGINX config __inside the server block__ enable static resource caching:
```
expires $expires;
```

And this __outside the server block__:
```
map $sent_http_content_type $expires {
    default    off;
    text/css    max;
    ~image/    max;
    application/javascript    max;
    application/octet-stream    max;
}
```

## Deploy script Ploi

```bash
if [[ {COMMIT_MESSAGE} =~ "[BOT]" ]]; then
    echo "Automatically committed on production. Nothing to deploy."
    {DO_NOT_NOTIFY}
    # Uncomment the following line when using zero downtime deployments.
    # {CLEAR_NEW_RELEASE}
    exit 0
fi

cd {SITE_DIRECTORY}
git pull origin main
composer install --no-interaction --prefer-dist --optimize-autoloader --no-dev

npm ci
npm run build
php{SITE_PHP_VERSION} artisan cache:clear
php{SITE_PHP_VERSION} artisan config:cache
php{SITE_PHP_VERSION} artisan route:cache
php{SITE_PHP_VERSION} artisan statamic:stache:warm
php{SITE_PHP_VERSION} artisan queue:restart
php{SITE_PHP_VERSION} artisan statamic:search:update --all
php{SITE_PHP_VERSION} artisan statamic:static:clear
php{SITE_PHP_VERSION} artisan statamic:static:warm --queue
php{SITE_PHP_VERSION} artisan statamic:assets:generate-presets --queue

{RELOAD_PHP_FPM}

echo "🚀 Application deployed!"
```

## Deploy script Forge

```bash
if [[ $FORGE_DEPLOY_MESSAGE =~ "[BOT]" ]]; then
    echo "Automatically committed on production. Nothing to deploy."
    exit 0
fi

cd $FORGE_SITE_PATH
git pull origin main
$FORGE_COMPOSER install --no-interaction --prefer-dist --optimize-autoloader --no-dev

npm ci
npm run build
$FORGE_PHP artisan cache:clear
$FORGE_PHP artisan config:cache
$FORGE_PHP artisan route:cache
$FORGE_PHP artisan statamic:stache:warm
$FORGE_PHP artisan queue:restart
$FORGE_PHP artisan statamic:search:update --all
$FORGE_PHP artisan statamic:static:clear
$FORGE_PHP artisan statamic:static:warm --queue
$FORGE_PHP artisan statamic:assets:generate-presets --queue

( flock -w 10 9 || exit 1
    echo 'Restarting FPM...'; sudo -S service $FORGE_PHP_FPM reload ) 9>/tmp/fpmlock
```

Copyright 2022 - Studio 1902
Peak is open source.
Statamic is closed source.
Project design and content is protected under copyright.
