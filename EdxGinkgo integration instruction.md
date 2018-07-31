# Интеграция Examus и Edx Ginkgo

## Установка зависимостей
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/edx-proctoring.git@ginkgo.2```
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/open-edx-api-extension.git@ginkgo.2```
 * ```sudo /edx/bin/pip.edxapp install git+https://github.com/examus/npoed_multiproctoring.git#egg=npoed-multiproctoring```
 * ```sudo /edx/bin/pip.edxapp install django-dirtyfields```

 * ```sudo /edx/bin/python.edxapp manage.py lms makemigrations npoed_multiproctoring --settings=aws```
 * ```sudo /edx/bin/python.edxapp manage.py lms migrate npoed_multiproctoring --settings=aws```
 * ```sudo /edx/bin/python.edxapp manage.py lms migrate edx_proctoring --settings=aws```

## Конфигурация
Добавить в файл "lms/envs/aws.py" следующие строки.

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
                "exam_register_endpoint": "https://kfc.examus.net/api/v1/integration/npoed/exams/",
                "exam_sponsor": "Examus",
                "organization": "EXAMUS",
                "secret_key": "0VdlELK1nkZ7isXi8myAXpYxeuf0ObyP",
                "secret_key_id": "1",
                "software_download_url": "https://chrome.google.com/webstore/detail/examus/apippgiggejegjpimfjnaigmanampcjg"
            },
            "settings": {
                "LINK_URLS": {
                    "contact_us": "https://fish.npoed.ru/feedback/",
                    "faq": "https://fish.npoed.ru/proctoring/",
                    "online_proctoring_rules": "{add link here}",
                    "tech_requirements": "https://fish.npoed.ru/systemcheck/"
                }
            }
        }
    }
```

## Добавить в файл "lms/urls.py" следующие строки.

```
urlpatterns += (
    url(r'^api/extended/', include('open_edx_api_extension.urls', namespace='api_extension')),
)



В фалйах "lms.env.json" и "ms.env.json" в словаре FEATURES установить параметр "ENABLE_SPECIAL_EXAMS" = true:

"FEATURES": {
    :
    "ENABLE_SPECIAL_EXAMS": true,
    :
}
```

## Заменить файлы
 - cms/static/js/views/modals/course_outline_modals.js 
 - cms/templates/js/timed-examination-preference-editor.underscore
на файлы из репозитория:
`https://github.com/examus/npoed_multiproctoring/tree/4779cca51bf6b2e9a71f2c58ab34e37f009f0585/npoed_multiproctoring/static`


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
 - common/lib/xmodule/xmodule/seq_module.py : ProctoringFields
 - cms/djangoapps/contentstore/views/item.py : create_xblock_info
 - cms/djangoapps/models/settings/course_metadata.py : CourseMetadata
