# django-cms4-migration-notes

I'll try to write down any problems and resulting solutions/tips right in this README.

### some links first

- django cms blog: upgrade encouragement :) https://www.django-cms.org/en/blog/2026/01/10/why-now-is-the-right-time-to-migrate-from-django-cms-3-11/
- django cms blog: upgrade to 4.1 story https://www.django-cms.org/en/blog/2025/03/02/how-i-successfully-upgraded-a-legacy-django-cms-project-to-4-1-4/

### prepare in old cms3 ecosystem

- use latest django-cms 3.X, no out of date plugins or other dependencies
- best to use django 4.2
- ./manage.py cms page fix-tree
- ./manage.py plugin page fix-tree
- switch to djangocms-text if you use djangocms-text-ckeditor

### migrate core

work on branch cms4  
use https://github.com/django-cms/djangocms-4-migration - works!

```
# install new packages
django-cms>=4.1,<5
djangocms-versioning
djangocms-alias
git+https://github.com/django-cms/djangocms-4-migration

# in settings, add them
INSTALLED_APPS = [
    ...,
    "djangocms_4_migration",
    "djangocms_versioning",
    "djangocms_alias",
    ...,
]
# some more settings
CMS_CONFIRM_VERSION4 = True
CMS_TOOLBAR_URL_ENABLE = "edit"  # keep .com/?edit to login

# if you stored page ids somewhere, but not as FK or m2m (as django-ckeditor-link does, in html)
# you'll need to update page ids manually. old way stored draft page id, new way uses "old published".
CMS_MIGRATION_PROCESS_PAGE_REFERENCES = "mymodule.myfunction2" 

# example for page reference function, when using django-ckeditor-link
def update_ckeditor_link_pages_for_cms4(old_page, new_page):
    print("--old page > new page hook -------------------------")
    print(new_page.get_absolute_url())
    for model in apps.get_models():
        for field in model._meta.get_fields():
            # Skip relational reverse fields if you do not want them
            if not hasattr(field, "attname"):
                continue

            if not isinstance(field, RichTextField):
                continue  # filter out RichTextFields

            # Your logic here
            kwargs = {f"{field.name}__contains": f'data-cms_page="{old_page.id}"'}
            to_update = model.objects.filter(**kwargs)
            if to_update.count():
                print(
                    f"updating {model.__name__}.{field.name} -> {field.__class__.__name__}"
                )
                for obj in to_update:
                    text = getattr(obj, field.name)
                    text = text.replace(
                        f'data-cms_page="{old_page.id}"',
                        f'data-cms_page="{new_page.id}"',
                    )
                    setattr(obj, field.name, text)
                    obj.save()

# actually migrate to new db structure
./manage.py cms4_migration

# upgrade to Django 5.2 LTS

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

### oppiniated enhancements I want to establish

- change "create version" to "edit", always (VersioningToolbar)
- remove "preview" button, as it's unneeded and causing confusion (VersioningToolbar)
