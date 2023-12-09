Ansible
=======

.. _Ansible: https://www.ansible.com/
.. _Ansible documentation: https://docs.ansible.com/ansible/latest/index.html

`Ansible`_ is a fantastic piece of software for configuration
and deployment automation. It allows for the creation of ``playbooks``
using the YAML format to specify the setup steps.

Each step can be checked for the success, allowing for the configuration
to continue in case it has been successful, or interrupting it as early
as a problem is hit.

Note: it is not always clear why the configuration may
fail, so you may need to double-check manually where it stopped.

Since Ansible can do so much stuff via built-in modules, community-modules
and custom-modules, we won't document it and will just defer readers
to the `Ansible documentation`_. Their documentation isn't perfect, but
a quick search there, Stackoverflow and some quick tests are enough
to get things working.

We highlight two of the necessary steps to get Ansible working on our
testbed.

Ansible installation
--------------------

Depending on the software you are running, specific versions of Ubuntu,
or other server OS, may be restricted. In order to always get the newest
release for ansible, do the following:

.. sourcecode:: console

    $ sudo apt-add-repository ppa:ansible/ansible
     Ansible is a radically simple IT automation platform that makes your applications and systems easier to deploy. Avoid writing scripts or custom code to deploy and update your applications— automate in a language that approaches plain English, using SSH, with no agents to install on remote systems.

    http://ansible.com/

    If you face any issues while installing Ansible PPA, file an issue here:
    https://github.com/ansible-community/ppa/issues
     More info: https://launchpad.net/~ansible/+archive/ubuntu/ansible
    Press [ENTER] to continue or Ctrl-c to cancel adding it.

    Get:1 http://ppa.launchpad.net/ansible/ansible/ubuntu focal InRelease [18.0 kB]
    ...
    Get:38 http://ports.ubuntu.com/ubuntu-ports focal-security/multiverse arm64 c-n-f Metadata [116 B]
    Fetched 22.4 MB in 11s (2131 kB/s)
    Reading package lists... Done
    $ sudo apt install -y ansible
    Reading package lists... Done
    Building dependency tree... 50%
    Building dependency tree
    Reading state information... Done
    The following additional packages will be installed:
      ansible-core python3-bcrypt python3-jmespath python3-kerberos
      python3-ntlm-auth python3-packaging python3-paramiko python3-pyparsing
      python3-requests-kerberos python3-requests-ntlm python3-resolvelib
      python3-winrm python3-xmltodict sshpass
    Suggested packages:
      python3-gssapi python-pyparsing-doc
    The following NEW packages will be installed:
      ansible ansible-core python3-bcrypt python3-jmespath python3-kerberos
      python3-ntlm-auth python3-packaging python3-paramiko python3-pyparsing
      python3-requests-kerberos python3-requests-ntlm python3-resolvelib
      python3-winrm python3-xmltodict sshpass
    0 upgraded, 15 newly installed, 0 to remove and 15 not upgraded.
    Need to get 22.3 MB of archives.
    After this operation, 323 MB of additional disk space will be used.
    Get:1 http://ports.ubuntu.com/ubuntu-ports focal/main arm64 python3-pyparsing all 2.4.6-1 [61.3 kB]
    ...
    Setting up ansible (5.10.0-1ppa~focal) ...
    Processing triggers for man-db (2.9.1-1) ...

Now we got the latest ansible release for our platform.
We can happily run our playbook now.

Ansible playbook execution
--------------------------

Since our playbooks are pretty big and convoluted to set everything up,
let's see how to execute a small playbook called ``helloworld.yaml``.

.. sourcecode:: yaml

    - name: My first play
      hosts: localhost
      tasks:
       - name: Print message
         ansible.builtin.debug:
          msg: Hello world

Now we can call ansible to execute it:

.. sourcecode:: console

    $ ansible-playbook helloworld.yaml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that
    the implicit localhost does not match 'all'

    PLAY [My first play] ***********************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [localhost]

    TASK [Print message] ***********************************************************
    ok: [localhost] => {
        "msg": "Hello world"
    }

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Elevating privileges
--------------------

As a security-conscious person, I am the first to tell people not to use ``sudo``,
however it isn't always possible to do that. And since we are in a VM, it can't be
that bad to let people run amok... Am I right? (No I am not...).

So, Ansible allows for privilege elevation using the following construct:

.. sourcecode:: yaml

    - name: My first play
      hosts: localhost
      become: true
      tasks:
       - name: Test sudo
         command: id -u
         register: id_output
       - name: Print message
         ansible.builtin.assert:
          that: id_output.stdout == '0'
          success_msg: Hello world sudo user '{{ lookup('env', 'USER') }}'
          fail_msg: Hello world non-sudo user '{{ lookup('env', 'USER') }}'

When we execute it, ``become: true`` will elevate the current user with sudo.

.. sourcecode:: console

    $ ansible-playbook helloworld.yaml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that
    the implicit localhost does not match 'all'

    PLAY [My first play] ***********************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [localhost]

    TASK [Test sudo] ***************************************************************
    changed: [localhost]

    TASK [Print message] ***********************************************************
    ok: [localhost] => {
        "changed": false,
        "msg": "Hello world sudo user 'ubuntu'"
    }

    PLAY RECAP *********************************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

After setting ``become: false``, we can see it does it fact work as expected.

.. sourcecode:: console

    ansible-playbook helloworld.yaml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that
    the implicit localhost does not match 'all'

    PLAY [My first play] ***********************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [localhost]

    TASK [Test sudo] ***************************************************************
    changed: [localhost]

    TASK [Print message] ***********************************************************
    fatal: [localhost]: FAILED! => {
        "assertion": "id_output.stdout == '0'",
        "changed": false,
        "evaluated_to": false,
        "msg": "Hello world non-sudo user 'ubuntu'"
    }

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0


UnB's testbed playbooks
-----------------------

.. _Open AI Cellular: https://www.openaicellular.org/
.. _OAIC documentation: https://openaicellular.github.io/oaic/

The UnB testbed setup playbook is based on the great guide from `Open AI Cellular`_
available in `OAIC documentation`_. It can be found along in the directory
``ansible-playbooks``, which is along with the sources for this documentation.

.. _ORAN_testbed_docs: https://github.com/Gabrielcarvfer/ORAN_testbed_docs

Currently, it is hosted in `ORAN_testbed_docs`_ repository.

TODO: finalize playbooks and upload them.