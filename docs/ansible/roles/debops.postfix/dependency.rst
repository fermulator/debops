.. _postfix__ref_dependency:

Usage as a role dependency
==========================

The ``debops.postfix`` role can be used as a dependency by other Ansible roles
to manage contents of the :file:`/etc/postfix/main.cf` and
:file:`/etc/postfix/master.cf` configuration files idempotently.  Configuration
options from multiple roles can be merged together and included in the Postfix
configuration, or removed conditionally.

.. contents::
   :local:


Dependent role variable
-----------------------

The role exposes the :envvar:`postfix__dependent_maincf` and
:envvar:`postfix__dependent_mastercf` variables which can be used to define
Postfix configuration options and services by other Ansible roles through the
role dependent variables.

The variables are an YAML lists with YAML dictionaries as entries. A short
format of the configuration uses the dictionary key as a name of the dependent
role and dictionary value as that role's configuration, in the format defined
by :ref:`postfix__ref_maincf` and :ref:`postfix__ref_mastercf` variables,
respectively (see playbook excerpt below):

.. code-block:: yaml

   roles:

     - role: debops.postfix
       postfix__dependent_maincf:
         - role_name: '{{ role_name__postfix__dependent_maincf }}'
       postfix__dependent_mastercf:
         - role_name: '{{ role_name__postfix__dependent_mastercf }}'

The extended version of the configuration uses YAML dictionaries with specific
parameters:

``role``
  Required. Name of the role, used to save its configuration in a YAML
  dictionary on the Ansible Controller. Shouldn't be changed once selected,
  otherwise the configuration will be desynchronized.

``config``
  Required. YAML list with configuration of the Postfix options and services in the
  same format defined by :ref:`postfix__ref_maincf` and
  :ref:`postfix__ref_mastercf` variables.

``state``
  Optional. If not specified or ``present``, the configuration will be included
  in the generated configuration files. If ``absent``, the configuration will
  be removed from the configuration files. If ``ignore``, a given configuration
  entries will be skipped during data evaluation and won't affect any existing
  entries.

An example extended configuration (playbook excerpt):

.. code-block:: yaml

   roles:

     - role: debops.postfix
       postfix__dependent_maincf:
         - role: 'role_name'
           config: '{{ role_name__postfix__dependent_maincf }}'
       postfix__dependent_mastercf:
         - role: 'role_name'
           config: '{{ role_name__postfix__dependent_mastercf }}'

The above configuration layout allows for use of the multiple role dependencies
in one playbook by providing configuration of each role in a separate
configuration entry.


Dependent configuration storage and retrieval
---------------------------------------------

The dependent configuration from other roles is stored in the :file:`secret/`
directory on the Ansible Controller (see :ref:`debops.secret` for more details) in
a JSON file (one for each variable), with each role configuration in a separate
dictionary. The ``debops.postfix`` role reads these files when Ansible local
facts indicate that the Postfix is installed, otherwise empty files are
created. This ensures that the stale configuration is not present on a new or
re-installed host.

The YAML dictionaries from different roles are merged with the main
configuration in the :envvar:`postfix__combined_maincf` and
:envvar:`postfix__combined_mastercf` variables that are used to generate the
final configuration. The merge order of the different ``postfix__*_maincf`` and
``postfix__*_mastercf`` variables allows to further affect the dependent
configuration through Ansible inventory if necessary, therefore the Ansible
roles that use this method don't need to provide additional variables for this
purpose themselves.


Example role default variables
------------------------------

.. literalinclude:: examples/application-defaults.yml
   :language: yaml


Example role playbook
---------------------

.. literalinclude:: examples/application-playbook.yml
   :language: yaml
