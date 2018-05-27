Zuul Base Jobs
==============

This repo contains a generic Zuul base job recommended for use by simple
Zuul deployments, and a copy named base-test for use in testing changes to
the same.

Container build resources
-------------------------

Asuuming this nodepool configuration:

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


To use a container as an unprivileged instance, adds in a project:

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

    # playbooks/container-machine.yaml
    - hosts: pod
      tasks:
        - command: python3 demo.py
          args:
            chdir: "{{ zuul.project.src_dir }}"


To use a container native job, adds in a project:

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
            - openshift-test

    # playbooks/container-native.yaml
    - hosts: localhost
      tasks:
        - name: Fetch pods list from pre run
          include_vars:
            file: "{{ zuul.executor.work_root }}/pods.yaml"

        - add_host:
            name: "{{ item.pod }}"
            group: "{{ item.name }}"
            ansible_connection: kubectl
          with_items: "{{ pods }}"

    - hosts: demo-project
      tasks:
        - command: python demo.py
          register: demo_output
        - fail:
          when: "'Hello OpenShift' not in demo_output.stdout"
