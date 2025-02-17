============================
Publishing the Documentation
============================

In this section you will publish your documentation on a website for others to
view. This is an important step to make your package more accessible and
usable. If you are using the internal UKAEA GitLab, the documentation will be
hosted using GitLab Pages, a built-in service of the GitLab server. Keep in
mind that like GitLab itself, this will only be visible to users with access to
GitLab and the VPN. If your repo is on GitHub, then the docs will be hosted
by GitHub Pages.

This section assume you have already set up your CI pipeline, as we covered in
a previous section, :doc:`ci`.

Documentation on Git*Lab* Pages
-------------------------------

GitLab makes deploying and hosting your documentation simple. Deployment of
your project's documentation to GitLab Pages is already set up in the
``.gitlab-ci.yml`` file provided in the cookie-cutter (see the ``pages`` job),
and it is done for every commit to the main branch. GitLab CI recognises ``pages``
as a special name for the job and automatically handles the deployment and
hosting of any artifacts stored in the special folder ``public/``. Of course,
these artifacts must be valid, static HTML pages, and the HTML built by Sphinx
for your project is perfectly suited. You can find further details
in `GitLab's documentation <https://git.ccfe.ac.uk/help/user/project/pages/index.md>`_.

Documentation on Git*Hub* Pages
-------------------------------

Unfortunately, deploying your documentation with GitHub is not as easy.
We recommend `doctr <https://drdoctr.github.io>`_ a tool that configures
Travis-CI to automatically publish your documentation every time the master
branch is updated.

.. warning::
    The repo you want to build the docs for has to be a root repo.
    You cannot build docs for a forked repo by doctr yet.
    The doctr team is working on enabling a forked repo build under
    `PR #343 <https://github.com/drdoctr/doctr/pull/343>`_.

Install doctr.

   .. code-block:: bash

      python3 -m pip install --upgrade doctr

Type ``doctr configure`` and follow the prompts. Example:

.. code-block:: bash

    $ doctr configure
    Welcome to Doctr.

    We need to ask you a few questions to get you on your way to automatically
    deploying from Travis CI to GitHub pages.

    What is your GitHub username? YOUR_GITHUB_USERNAME
    Enter the GitHub password for YOUR_GITHUB_USERNAME:
    A two-factor authentication code is required: app
    Authentication code: XXXXXX
    What repo do you want to build the docs for (org/reponame, like 'drdoctr/doctr')? YOUR_GITHUB_USERNAME/YOUR_REPO_NAME
    What repo do you want to deploy the docs to? [YOUR_GITHUB_USERNAME/YOUR_REPO_NAME]

    The deploy key has been added for YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.

    You can go to https://github.com/NSLS-II/NSLS-II.github.io/settings/keys to revoke the deploy key.

Then doctr shows some instructions, which we will take one at a time,
elaborating here.

.. code-block:: bash

    ================== You should now do the following ==================

    1. Add the file  github_deploy_key_your_github_username_your_repo_name.enc to be staged for commit:

        git add github_deploy_key_your_github_username_your_repo_name.enc

Remember that you can always use ``git status`` to list uncommitted files like
this one.

.. code-block:: bash

    2. Add these lines to your `.travis.yml` file:

        env:
          global:
            # Doctr deploy key for YOUR_GITHUB_USERNAME/YOUR_REPO_NAME
            - secure: "<REDACTED>"

        script:
        - set -e
        - <Command to build your docs>
        - pip install doctr
        - doctr deploy --built-docs <path/to/built/html/> <target-directory>

    Replace the text in <angle brackets> with the relevant
    things for your repository.

    Note: the `set -e` prevents doctr from running when the docs build fails.
    We put this code under `script:` so that if doctr fails it causes the
    build to fail.

This output includes an encrypted token --- the string next to ``secure:``
which I have redacted in the example above --- that gives Travis-CI permission
to upload to GitHub Pages.

Putting this together with the default ``.travis.yml`` file generated by the
cookiecutter template, you'll have something like:

.. literalinclude:: example_travis_with_doctr.yml
   :emphasize-lines: 11-14,27-30

where ``<REDACTED>`` is replaced by the output you got from ``doctr configure``
above.

.. code-block:: bash

    3. Commit and push these changes to your GitHub repository.
    The docs should now build automatically on Travis.

    See the documentation at https://drdoctr.github.io/ for more information.

Once the changes are pushed to the ``master`` branch on GitHub, Travis-CI will
publish the documentation to
``https://<YOUR_GITHUB_USERNAME>.github.io/<YOUR_REPO_NAME>``. It may take a
minute for the updates to process.

Alternatives
------------

Another popular option for publishing documentation is
`https://readthedocs.org/ <https://readthedocs.org/>`_ . We slightly prefer
using Travis-CI + GitHub Pages because it is easier to debug any installation
issues. It is also more efficient: we have to build the documentation on
Travis-CI anyway to verify that any changes haven't broken them, so we might as
well upload the result and be done, rather than having readthedocs build them
again.
