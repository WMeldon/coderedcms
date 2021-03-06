Contributing
============


Developing CodeRed CMS
---------------------------------

To create a test project locally:

#. Clone the code from https://github.com/coderedcorp/coderedcms.
#. Run ``pip install -e ./[dev]`` from the root coderedcms directory. The -e flag makes the install editable,
   which is relevant when running makemigrations in test project to actually generate the migration
   files in the coderedcms pip package. The ``[dev]`` installs extras such as sphinx for generating docs.
#. Follow the steps in :doc:`/getting_started/install`. Use "testproject" for
   your project name to ensure it is ignored by git.
#. When making model or block changes within coderedcms, run ``makemigrations coderedcms`` in the
   test project to generate the relevant migration files for the pip package. ALWAYS follow steps
   4 and 5 in :doc:`/getting_started/install` with a fresh database before making migrations.
#. When model or block changes affect the local test project (i.e. the "website" app), run
   ``makemigrations website`` in the test project to generate the relevant migration files locally.
   Apply and test the migrations. When satisfied, re-generate the 0001_initial.py migration in
   ``project_template/website/migrations/`` as so:

       #. Create a new test project using ``coderedcms start testproject``.
       #. Before running migrations, DELETE all migrations in ``testproject/website/migrations/``.
       #. Run ``python manage.py makemigrations website``. This should generate an ``0001_initial.py``
          migration.
       #. Replace ``project_template/website/migrations/0001_initial.py`` with your newly generated migration.

When making changes that are potentially destructive or backwards incompatible, increment the minor
version number until coderedcms reaches a stable 1.0 release. Each production project that uses
coderedcms should specify the appropriate version in its requirements.txt to prevent breakage.

.. note:
    When testing existing projects with coderedcms installed from the master or development branches,
    be sure to use a disposable database, as it is likely that the migrations in master will
    not be the same migrations that get released.

Testing CodeRed CMS
-------------------

To run the built in tests for CodeRed CMS, run the following in your test project's directory:

``python manage.py test coderedcms --settings=coderedcms.tests.settings``


Adding New Tests
----------------

Test coverage at the moment is fairly minimal and it is highly recommended that new features and models include proper unit tests.
Any testing infrastructure (ie implementations of abstract models and migrations) needed should be added to the ``tests`` app in your
local copy of CodeRed CMS.  The tests themselves should be in their relevant section in CodeRed CMS (ie tests for 
models in ``coderedcms.models.page_models`` should be located in ``coderedcms.models.tests.test_page_models``).

For example, here is how you would add tests for a new abstract page type, ``CoderedCustomPage`` that would live in ``coderedcms/models/page_models.py``:

1. Navigate to ``coderedcms/tests/testapp/models.py``
2. Add the following import: ``from coderedcms.models.page_models import CoderedCustomPage``
3. Implement a concrete version of ``CoderedCustomPage``, ie ``CustomPage(CoderedCustomPage)``.
4. Run ``python manage.py makemigrations`` to make new testing migrations.
5. Navigate to ``coderedcms/models/tests/test_page_models.py``
6. Add the following import: ``from coderedcms.models import CoderedCustomPage``
7. Add the following import: ``from coderedcms.tests.testapp.models import CustomPage``
8. Add the following to the bottom of the file: 
  ::
      
      class CoderedCustomPageTestCase(AbstractPageTestCase, WagtailPageTests):
          model = CoderedCustomPage
9. Add the following to the bottom of the file: 
  ::
      
      class CustomPageTestCase(ConcreteBasicPageTestCase, WagtailPageTests):
          model = CustomPage
10. Write any specific test cases that ``CoderedCustomPage`` and ``CustomPage`` may require.


Contributor guidelines
----------------------

We are happy to accept pull requests from the community if it aligns with our vision for coderedcms.
When creating a pull request, please make sure you include the following:

* A description in the pull request of what this change does and how it works.
* Reference to an issue if the change is related to one of the issues on our GitHub page.
* Documentation updates in the ``docs/`` directory describing your change.

Following submission of your pull request, a CodeRed member will review and test your change.
**All changes, even by CodeRed members, must go through a pull request process to ensure quality.**


Building pip packages
---------------------

To build a publicly consumable pip package, run::

    python setup.py sdist bdist_wheel


Building documentation
----------------------

For every code or feature change, be sure to update the docs in the repository. To build and publish
the documentation run::

    cd docs/
    make clean
    make html

Output will be in ``docs/_build/html/`` directory.


Publishing a new release
------------------------

First checkout the code/branch for release.

Next build a pip package::

    python setup.py sdist bdist_wheel

Then upload the pip package to pypi::

    twine upload dist/*

Finally build and update docs::

    cd docs/
    make clean
    make html

If updating docs for an existing minor version release:

#. Copy the contents of ``docs/_build/html/`` to the CodeRed docs server under the existing version directory.

If this is a new major or minor version release:

#. Create a new major.minor directory on the CodeRed docs server.
#. Update the ``stable`` symlink to point to the new version directory.
#. Add the new version to the ``versions.txt`` file on the docs server.
#. Copy the contents of ``docs/_build/html/`` to the CodeRed docs server under the new version directory.

Note that we do not release separate documentation versions for maintenance releases. Update the existing minor
version docs with release notes and other changes.
