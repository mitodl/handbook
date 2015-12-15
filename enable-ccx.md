Starting with a fresh install of devstack,

1. Create a private.py for cms (`edx-platform/cms/envs/private.py`) with the following lines:

 ```
from .devstack import * # pylint: disable=wildcard-import, unused-wildcard-import

FEATURES['CUSTOM_COURSES_EDX'] = True
 ```

2. Create a private.py for lms (`edx-platform/lms/envs/private.py`) with the following lines:

   ```
from .devstack import * # pylint: disable=wildcard-import, unused-wildcard-import

FEATURES['CUSTOM_COURSES_EDX'] = True

##### Custom Courses for EdX #####
if FEATURES.get('CUSTOM_COURSES_EDX'):
        INSTALLED_APPS += ('ccx',)
        FIELD_OVERRIDE_PROVIDERS += (
            'ccx.overrides.CustomCoursesForEdxOverrideProvider',
        )
   ```

3. As the edxapp user in vagrant, run `paver update_db`

   The output put should include something like the following:

   ```
Running migrations for ccx:
 - Migrating forwards to 0002_convert_memberships.
 > ccx:0001_initial
 > ccx:0002_convert_memberships
 - Migration 'ccx:0002_convert_memberships' is marked for no-dry-run.
 - Loading initial data for ccx.
   ```

3. Start the servers with `paver run_all_servers`

3. Continue with the user instructions at
http://edx.readthedocs.org/projects/open-edx-building-and-running-a-course/en/latest/building_course/custom_courses.html
