# Ansible playbooks

Arnold is composed of a suite of playbooks and OpenShift templates to create
various objects representing services. In this section, we will list and
describe all playbooks bundled with Arnold. We choose to describe them in the
order they are supposed to be used.

> **Remark**: when running an Ansible playbook, you can pass arguments to
> override Ansible variables definition (_e.g._ `ansible-playbook my_playbook.yml -e "my_var=my_value"` to define `my_var` with the `my_value`
> value in executed playbook). Hence, every time you see the `-e` option for a
> playbook invocation, it means that we will override default values. You must
> know that default values for `customer` and `env_type` are `patient0` and
> `development` respectively. **In all cases, the `-e "customer=foo env_type=bar"` option is not required if you want to work with `patient0` in
> `development`**.

## `init_project.yml`

This playbook is a "meta" playbook that creates a new project with all required
OpenShift objects for a customer (default: `patient0`) in a particular
environment (default: `development`). It executes sequentially the following
playbooks:

* `create_project.yml`
* `create_secrets.yml`
* `create_config.yml`
* `create_acme.yml`
* `create_objects.yml`

### Usage

```bash
# development
$ bin/ansible-playbook init_project.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook init_project.yml -e "customer=patient0 env_type=staging"
```

## `deploy.yml`

The `deploy.yml` playbook defines a new `deployment_stamp` to create a whole new
stack by defining OpenShift objects with unique labels and names. This allows us
to initiate a blue/green deployment strategy.

### Usage

```bash
# development
$ bin/ansible-playbook deploy.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook deploy.yml -e "customer=patient0 env_type=staging"
```

## `create_project.yml`

This playbook only creates a new OpenShift project name defined with the
following pattern: `{{ env_type }}-{{ customer }}`

### Usage

```bash
# development
$ bin/ansible-playbook create_project.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_project.yml -e "customer=patient0 env_type=staging"
```

## `create_secrets.yml`

The `create_secrets.yml` playbook creates OpenShift secrets that may store
sensitive credentials. It uses vaulted Yaml definition files stored in
`group_vars/secrets/{{ customer }}/{{ env_type }}/*/credentials.vault.yml` to
create those secrets. It must be invoked with the `--ask-vault-pass` option so
that Ansible can decrypt vaulted credentials and push secrets to OpenShift.

### Usage

```bash
# development
$ bin/ansible-playbook create_secrets.yml --ask-vault-pass -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_secrets.yml --ask-vault-pass -e "customer=patient0 env_type=staging"
```

## `create_config.yml`

This playbook creates required `ConfigMap` objects for each service. Default
`ConfigMap`s that will be created are stored in
`templates/configmap/<< service >>`. Know that these `ConfigMap`s can be
overridden by adding them to
`files/configmap/{{ customer }}/{{ env_type }}/<< service >>`.

### Usage

```bash
# development
$ bin/ansible-playbook create_config.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_config.yml -e "customer=patient0 env_type=staging"
```

## `create_objects.yml`

As you might expect, this playbook creates all OpenShift objects required for a
project, _i.e._:

* volumes (_aka_ PVC for Persistant Volume Claim)
* end points (_aka_ EP)
* deployment configurations (_aka_ DC)
* services (_aka_ SVC)
* jobs
* routes

All templates for these objects are listed in the `group_vars/all/main.yml` file.

### Usage

```bash
# development
$ bin/ansible-playbook create_objects.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_objects.yml -e "customer=patient0 env_type=staging"
```

## `create_acme.yml`

We use this playbook to automatically generate [Let's
Encrypt](https://letsencrypt.org) SSL certificates for every deployment (if the
current `env_type` is not `development`). The certificates issued are
automatically renewed by the service running to OpenShift.

### Usage

```bash
# development
$ bin/ansible-playbook create_acme.yml -e "customer=patient0 env_type=staging"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_acme.yml -e "customer=patient0 env_type=staging"
```

## `create_object.yml`

This playbook is mostly used in development to create or update an OpenShift
object we are working on.

### Usage

```bash
# development
$ bin/ansible-playbook create_object.yml -e "customer=patient0 env_type=staging" \
        -e "object_template=templates/openshift/edxapp/job/import_demo_course.yml.j2"

# native command for production
$ docker run --rm -it \
    --env-file env.d/production \
    arnold \
    ansible-playbook create_object.yml -e "customer=patient0 env_type=staging" \
        -e "object_template=templates/openshift/edxapp/job/import_demo_course.yml.j2"
```
