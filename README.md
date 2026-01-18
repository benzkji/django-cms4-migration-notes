# django-cms4-migration-notes

### some links first :)

- https://www.django-cms.org/en/blog/2026/01/10/why-now-is-the-right-time-to-migrate-from-django-cms-3-11/

### prepare in old cms3 ecosystem

- ./manage.py cms page fix-tree
- ./manage.py plugin page fix-tree
- switch to djangocms-text if you use djangocms-text-ckeditor

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

# if you stored page ids somewhere, but not as FK or m2m (as django-ckeditor-link does, in html)
# you'll need to update page ids with
CMS_MIGRATION_PROCESS_PAGE_REFERENCES = "mymodule.myfunction2" 

# example to follow

./manage.py cms4_migration
```

### update outside cms placeholders

-   https://docs.django-cms.org/en/latest/how_to/01-placeholders.html#
-   makemigrations, migrate, after new field + property are there
-   be careful with AutoSlugMixin, when editing objects! (unwanted 301s)
-   page_title not shown on detail pages: https://github.com/django-cms/django-cms/issues/8427
-   no modal editing/edit mode in overviews/apphooks: https://github.com/django-cms/django-cms/issues/7712#issuecomment-1844845602

### replace Page.public()

for example, when trying to get a page's url:
```
# before
Page.objects.public().get(reverse_id="downloads").get_absolute_url()

# after
from djangocms_versioning.constants import PUBLISHED

Page.objects.filter(
                reverse_id=my_rev_id
                pagecontent_set__versions__state=PUBLISHED,
            ).get_absolute_url()
```

### current problems

-   remove unnecessary"Vorschau/Preview" button ?!!
