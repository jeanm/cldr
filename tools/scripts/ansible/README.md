# Ansible for Survey Tool

These are ansible scripts for setup and maintenance of the Survey Tool.

## Scope

Right now, the test setup mostly controls Tomcat, but not the nginx proxy
due to public port issues (https).

## Setup

### Setup: Control system

This is your local system, where you control the others from.

- Install Ansible <https://ansible.com>
- Install some prereqs:

```shell
ansible-galaxy install -r roles.yml
```

- Make sure you can `ssh` into all of the needed systems. For example,
`ssh cldr-ref.unicode.org` should succeed without needing a password.

- You should be able to run `ansible all -m ping` and get something back
like the following:

```shell
cldr-ref.unicode.org | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

### Setup: Managed systems

- Install python3. Make sure `python --version` returns "Python 3…"

- TODO: these shouldn't be needed, but they are. Here's the entire
install command:

```shell
sudo apt-get update && sudo apt-get install python3 python-apt python3-pymysql
```

### Setup: surveytool keypair

Create a RSA keypair with no password for the buildbot:

```shell
mkdir -p ./local-vars
ssh-keygen -t rsa -b 4096 -f ./local-vars/surveytool -P '' -C 'surveytool deploy'
```

The contents of the `local-vars/surveytool.pub` file is used for the
`key:` parameter below in `local.yml`. The `local-vars/surveytool`
private key is used in the secret `RSA_KEY_SURVEYTOOL`.

Then setup github secrets as shown:

- `SMOKETEST_HOST` -
  hostname of smoketest
- `SMOKETEST_PORT` -
  port of smoketest
- `RSA_KEY_SURVEYTOOL` -
  contents of `local-vars/surveytool` (the secret key)
- `SMOKETEST_KNOWNHOSTS` -
  run `ssh-keyscan smoketest.example.com` where _smoketest.example.com_
  is the name of the smoketest server.  Put the results into this
  secret. One of these lines should match `~/.ssh/known_hosts` on your
  own system when you ssh into smoketest.
  Try `grep -i smoke ~/.ssh/known_hosts`

### Setup: Config file

- Create a file `local-vars/local.yml` with the following
(but with a secure password instead of `hunter42`.)

```yaml
mysql_users:
  - name: surveytool
    host: localhost
    password: hunter42
    priv: 'cldrdb.*:ALL'
surveytooldeploy:
  password: hunter43
  vap: hunter44
  testpw: hunter45
  oldversion: 36
  newversion: 37
  key: ssh-rsa …  ( SSH key goes here)
```

## Configure

Run the setup playbook.

```shell
ansible-playbook --check setup-playbook.yml
```

This is in dry run mode. When it looks good to you, take the
`--check` out and run it again.

## Local Test

- install vagrant and some provider such as virtualbox or libvirt

```shell
vagrant up
```

- To log into the new host, run `vagrant ssh`

- To iterate, trying to reapply ansible, run `vagrant provision --provision-with=ansible`

- to deploy ST to this, use:

```shell
curl -v -u surveytooldeploy:${SURVEYTOOLDEPLOY} -T ../cldr-apps/cldr-apps.war  'http://127.0.0.1:8080/manager/text/deploy?path=/cldr-apps&tag=cldr-apps&update=true'
```

where `SURVEYTOOLDEPLOY` is the password from surveytooldeploy.password above.
