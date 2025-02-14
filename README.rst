==================
aws-okta-processor
==================

.. image:: https://img.shields.io/pypi/v/aws-okta-processor.svg
   :target: https://pypi.python.org/pypi/aws-okta-processor
   :alt: Latest Version

.. image:: https://travis-ci.com/godaddy/aws-okta-processor.svg?branch=master
   :target: https://travis-ci.com/godaddy/aws-okta-processor
   :alt: Build Status

.. image:: https://codecov.io/gh/godaddy/aws-okta-processor/branch/master/graph/badge.svg
   :target: https://codecov.io/gh/godaddy/aws-okta-processor
   :alt: Coverage


This package provides a command for fetching AWS credentials through Okta.

------------
Installation
------------

The easiest way to install aws-okta-processor is to use `pip`_ in a ``virtualenv``::

    $ pip install aws-okta-processor

or, if you are not installing in a ``virtualenv``, to install globally::

    $ sudo pip install aws-okta-processor

or for your user::

    $ pip install --user aws-okta-processor

If you have the aws-cli installed and want to upgrade to the latest version
you can run::

    $ pip install --upgrade aws-okta-processor

.. note::

    On OS X, if you see an error regarding the version of six that came with
    distutils in El Capitan, use the ``--ignore-installed`` option::

        $ sudo pip install aws-okta-processor --ignore-installed six

This will install the aws-okta-processor package as well as all dependencies.  You can
also just `download the tarball`_.  Once you have the
aws-okta-processor directory structure on your workstation, you can just run::

    $ cd <path_to_aws-okta-processor>
    $ python setup.py install

---------------
Getting Started
---------------

This package is best used in `AWS Named Profiles`_ 
with tools and libraries that recognize `credential_process`_.

To setup aws-okta-processor in a profile create an INI formatted file like this::

    [default]
    credential_process=aws-okta-processor authenticate --user <user_name> --organization <organization>.okta.com

and place it in ``~/.aws/credentials`` (or in
``%UserProfile%\.aws/credentials`` on Windows). Then run::

    $ pip install awscli
    $ aws sts get-caller-identity

Supply a password then select your AWS Okta application and account role if prompted.
The AWS CLI command will return a result showing the assumed account role. If you run the
AWS CLI command again you will get the same role back without any prompts due to caching.

For tools and libraries that do not recognize ``credential_process`` aws-okta-processor
can be ran to export the following as environment variables::

    AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY
    AWS_SESSION_TOKEN

For Linux or OSX run::

    $ eval $(aws-okta-processor authenticate --environment --user <user_name> --organization <organization>.okta.com)

For Windows run::

    $ Invoke-Expression (aws-okta-processor authenticate --environment --user <user_name> --organization <organization>.okta.com)

----------------------------
Other Configurable Variables
----------------------------

Additional variables can also be passed to aws-okta-processors ``authenticate`` command 
as options or environment variables as outlined in the table below.

============ ============== ===================== ========================================
Variable     Option         Environment Variable  Description
============ ============== ===================== ========================================
user         --user         AWS_OKTA_USER         Okta user name
------------ -------------- --------------------- ----------------------------------------
password     --pass         AWS_OKTA_PASS         Okta user password
------------ -------------- --------------------- ----------------------------------------
organization --organization AWS_OKTA_ORGANIZATION Okta FQDN for Organization
------------ -------------- --------------------- ----------------------------------------
application  --application  AWS_OKTA_APPLICATION  Okta AWS application URL
------------ -------------- --------------------- ----------------------------------------
role         --role         AWS_OKTA_ROLE         AWS Role ARN
------------ -------------- --------------------- ----------------------------------------
duration     --duration     AWS_OKTA_DURATION     Duration in seconds for AWS session
------------ -------------- --------------------- ----------------------------------------
key          --key          AWS_OKTA_KEY          Key used in generating AWS session cache
------------ -------------- --------------------- ----------------------------------------
environment  --environment                        Output command to set ENV variables
------------ -------------- --------------------- ----------------------------------------
silent       --silent                             Silence Info output
============ ============== ===================== ========================================

^^^^^^^^
Examples
^^^^^^^^

If you do not want aws-okta-processor to prompt for any selection input you can export the following::

    $ export AWS_OKTA_APPLICATION=<application_url> AWS_OKTA_ROLE=<role_arn>

Or pass additional options to the command::

    $ aws-okta-processor authenticate --user <user_name> --organization <organization>.okta.com --application <application_url> --role <role_arn>

-------
Caching
-------

This package leverages caching of both the Okta session and AWS sessions. It's helpful to 
understand how this caching works to avoid confusion when attempting to switch between AWS roles.

^^^^
Okta
^^^^

When aws-okta-processor attempts authentication it will check the system's temporary directory 
for a file named ``<user>-<organization>-session.json`` based on the ``user`` and ``organization`` 
option values passed. If the file is not found or the session contents are stale then 
aws-okta-processor will create a new session and write it to the system's temporary directory. 
If the file exists and the session is not stale then the existing session gets refreshed.

^^^
AWS
^^^

After aws-okta-processor has a session with Okta and an AWS role has been selected it will fetch 
the role's keys and session token. This session information from the AWS role gets cached as a 
json file under ``~/.aws/boto/cache``. The file name is a SHA1 hash based on a combination the
``user``, ``organization`` and ``key`` option values passed to the command.

If you want to store a seperate AWS role session cache for each role assumed using the same 
``user`` and ``organization`` option values then pass a unique value to ``key``.
Named profiles for different roles can then be defined in ``~/.aws/credentials`` with content like this::

    [role_one]
    credential_process=aws-okta-processor authenticate --user <user_name> --organization <organization>.okta.com --application <application_url> --role <role_one_arn> --key role_one

    [role_two]
    credential_process=aws-okta-processor authenticate --user <user_name> --organization <organization>.okta.com --application <application_url> --role <role_two_arn> --key role_two

To clear all AWS session caches run::

    $ rm ~/.aws/boto/cache/*


------------
Getting Help
------------

* Ask a question on `slack <https://godaddy-oss-slack.herokuapp.com>`__
* If it turns out that you may have found a bug, please `open an issue <https://github.com/godaddy/aws-okta-processor/issues/new>`__

---------------
Acknowledgments
---------------

This package was influenced by `AlainODea <https://github.com/AlainODea>`__'s
work on `okta-aws-cli-assume-role <https://github.com/oktadeveloper/okta-aws-cli-assume-role>`__.



.. _`pip`: http://www.pip-installer.org/en/latest/
.. _`download the tarball`: https://pypi.org/project/aws-okta-processor/
.. _`AWS Named Profiles`: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html
.. _`credential_process`: https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#sourcing-credentials-from-external-processes