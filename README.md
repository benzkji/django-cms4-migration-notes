# django-cms4-migration-notes

### migrate core

work on branch cms4  
use https://github.com/django-cms/djangocms-4-migration - works!

```
install
django-cms>=4.1,<5
djangocms-versioning
djangocms-alias
# djangocms-text-ckeditor
git+https://github.com/django-cms/djangocms-4-migration

INSTALLED_APPS = [
    ...,
    "djangocms_4_migration",
    "djangocms_versioning",
    "djangocms_alias",
    ...,
]
CMS_CONFIRM_VERSION4 = True
CMS_TOOLBAR_URL_ENABLE = "edit"  # keep .com/?edit to login

./manage.py cms4_migration
```

### update outside cms placeholders

-   https://docs.django-cms.org/en/latest/how_to/01-placeholders.html#
-   makemigrations, migrate, after new field + property are there
-   be careful with AutoSlugMixin, when editing objects! (unwanted 301s)
-   page_title not shown on detail pages: https://github.com/django-cms/django-cms/issues/8427
-   no modal editing/edit mode in overviews/apphooks: https://github.com/django-cms/django-cms/issues/7712#issuecomment-1844845602

### current problems

-   remove unnecessary"Vorschau/Preview" button ?!!
