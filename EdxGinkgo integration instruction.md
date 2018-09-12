# Интеграция Examus и Edx Ginkgo

## Установка зависимостей
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/edx-proctoring.git@ginkgo.2```
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/open-edx-api-extension.git@ginkgo.2```
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/npoed_multiproctoring.git#egg=npoed-multiproctoring```
 * ```sudo /edx/bin/pip.edxapp install django-dirtyfields```

 * ```sudo /edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms makemigrations npoed_multiproctoring --settings=aws```
 * ```sudo /edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms migrate npoed_multiproctoring --settings=aws```
 * ```sudo /edx/bin/python.edxapp /edx/app/edxapp/edx-platform/manage.py lms migrate edx_proctoring --settings=aws```

## Параметры конфигурации
 * Экзамус предоставляет url своего сервера для интеграции. Для проверки интеграции можно указывать https://stage.examus.net
 * Экзамус предоставляет `username` и `password` для EXAMUS_PROCTORING_AUTH
 * Экзамус предоставляет `secret_key` и `exam_register_endpoint` для PROCTORING_BACKEND_PROVIDERS
 * На инсталляции OpenEdx нужно активировать oAuth2 client, сообщив Экзамусу YOUR_OAUTH2_KEY и YOUR_OAUTH2_SECRET

## Конфигурация
Добавить в файл "/edx/app/edxapp/edx-platform/lms/envs/aws.py" следующие строки.

```
############## EXAMUS INTEGRATION ################
INSTALLED_APPS += ("open_edx_api_extension", "npoed_multiproctoring")


EXAMUS_PROCTORING_AUTH = {
    "username": "edx_integration@examus.net",
    "password": "1XBMvWtX"
    }

PROCTORING_BACKEND_PROVIDER = {
        "class": "edx_proctoring.backends.null.NullBackendProvider",
        "options": {}
    }
PROCTORING_BACKEND_PROVIDERS = {
        "EXAMUS": {
            "class": "edx_proctoring.backends.examus.ExamusBackendProvider",
            "options": {
                "crypto_key": "123456789012345678901234",
                "exam_register_endpoint": "https://stage.examus.net/api/v1/integration/npoed/exams/",
                "exam_sponsor": "Examus",
                "organization": "EXAMUS",
                "secret_key": "0VdlELK1nkZ7isXi8myAXpYxeuf0ObyP",
                "secret_key_id": "1",
                "software_download_url": "https://chrome.google.com/webstore/detail/examus/apippgiggejegjpimfjnaigmanampcjg"
            },
            "settings": {
                "LINK_URLS": {
                    "contact_us": "https://your-edx-url.com/feedback/",
                    "faq": "https://your-edx-url.com/proctoring/",
                    "online_proctoring_rules": "{add link here}",
                    "tech_requirements": "https://your-edx-url.com/systemcheck/"
                }
            }
        }
    }
```

## Добавить в файл "/edx/app/edxapp/edx-platform/lms/urls.py" следующие строки.

```
urlpatterns += (
    url(r'^api/extended/', include('open_edx_api_extension.urls', namespace='api_extension')),
)



В фалйах "/edx/app/edxapp/lms.env.json" и "/edx/app/edxapp/cms.env.json" в словаре FEATURES установить параметр "ENABLE_SPECIAL_EXAMS" = true:

"FEATURES": {
    :
    "ENABLE_SPECIAL_EXAMS": true,
    :
}
```

## Заменить файлы
 - /edx/app/edxapp/edx-platform/cms/static/js/views/modals/course_outline_modals.js
 - /edx/app/edxapp/edx-platform/cms/templates/js/timed-examination-preference-editor.underscore
на следующие файлы:
`https://github.com/examus/npoed_multiproctoring/blob/539c447fa999727ee28bf630ec44d128b5b943ea/npoed_multiproctoring/static/cms.static.js.views.modals.course_outline_modals.js`
`https://github.com/examus/npoed_multiproctoring/blob/539c447fa999727ee28bf630ec44d128b5b943ea/npoed_multiproctoring/static/cms.templates.js.timed-examination-preference-editor.underscore`


## Добавить Сервис прокторинга
В Панели администрирования Django по урлу
`admin/npoed_multiproctoring/proctoringservice/`
Добавить сервис с именем EXAMUS и любым не пустым JSON в поле settings


## Добавить декоратор
```
from npoed_multiproctoring import enable_npoed_multiproctoring
@enable_npoed_multiproctoring
```

В следующие классы и функции:
 - /edx/app/edxapp/edx-platform/common/lib/xmodule/xmodule/seq_module.py : ProctoringFields
 - /edx/app/edxapp/edx-platform/cms/djangoapps/contentstore/views/item.py : create_xblock_info
 - /edx/app/edxapp/edx-platform/cms/djangoapps/models/settings/course_metadata.py : CourseMetadata
 

## Включение oAuth2
Добавить новый oAuth client:
```sudo /edx/bin/python.edxapp /edx/bin/manage.edxapp lms --setting=aws create_oauth2_client https://stage.examus.net https://stage.examus.net/complete/examusedx/ confidential --client_name examus --client_id YOUR_OAUTH2_KEY --client_secret YOUR_OAUTH2_SECRET --trusted```

В фалйах ```/edx/app/edxapp/lms.env.json``` и ```/edx/app/edxapp/cms.env.json``` в словаре FEATURES установить параметр "ENABLE_OAUTH2_PROVIDER" = true

Перезапустить edx:
``` sudo /edx/bin/supervisorctl restart edxapp: ```


## Минифицировать статику и пересобрать
```
sudo -H -u edxapp bash
source /edx/app/edxapp/edxapp_env
cd /edx/app/edxapp/edx-platform
paver update_assets cms --settings=aws

python manage.py cms --settings=aws collectstatic --noinput
```

## Integration key
integration key находится в файле "/edx/app/edxapp/lms.auth.json" по ключу "EDX_API_KEY"
