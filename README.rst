Zuul Base Jobs
==============

This repo contains a generic Zuul base job recommended for use by simple
Zuul deployments, and a copy named base-test for use in testing changes to
the same. This repo also contains building blocks to use Openshift resources
provider.


Container build resources
-------------------------

To use containers in an OpenShift cluster, you can use one of the base
openshift jobs in this repo.
You need to configure nodepool to handle container and then configure jobs.

As an example, the following is a working nodepool configuration:

.. code-block:: yaml

    labels:
      - name: openshift-project
      - name: openshift-pod

    providers:
      - name: openshift-rdocloud
        driver: openshift
        context: "myproject/openshift.rdocloud:8443/developer"
        pools:
          - name: main
            labels:
              - name: openshift-project
                type: project
              - name: openshift-pod
                type: pod
                image: docker.io/fedora:28


To use a container as an unprivileged instance, add to the project's zuul.yaml
configuration file:

.. code-block:: yaml

    # .zuul.yaml
    - job:
        name: demo-project-linters
        parent: base-openshift-container-as-unprivileged-machine
        run: playbooks/container-machine.yaml

    - project:
        check:
          jobs:
            - demo-project-linters

And create this playbook file:

.. code-block:: yaml

    # playbooks/container-machine.yaml
    - hosts: pod
      tasks:
        - command: python3 demo.py
          args:
            chdir: "{{ zuul.project.src_dir }}"


To use a container native job, add to the project's zuul.yaml
configuration file:

.. code-block:: yaml

    # .zuul.yaml
    - job:
        name: demo-project-test
        parent: base-openshift-container-native
        run: playbooks/container-native.yaml
        vars:
          base_image: "python:3.6"

    - project:
        check:
          jobs:
            - demo-project-test

And create this playbook file:

.. code-block:: yaml

    # playbooks/container-native.yaml
    - hosts: demo-project
      tasks:
        - command: python demo.py
          register: demo_output
        - fail:
          when: "'Hello OpenShift' not in demo_output.stdout"
