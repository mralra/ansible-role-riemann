# Riemann Ansible role

Ansible role for configuring the [Riemann] stream processor
for alerting on Logstash events.
Intended for use with the [freedomofpress.elk] role.

Requirements
------------
* a Logstash server to receive events from (see the [freedomofpress.elk] role)
* web service integration for alerting (currently only [Slack] is supported)

Role Variables
--------------
```yaml
# Version of the deb package to install. You can view the most
# recent release here: https://github.com/riemann/riemann/releases
riemann_version: "0.3.0"

riemann_download_directory: /usr/local/src
riemann_download_url: "https://github.com/riemann/riemann/releases/download/{{riemann_version}}/riemann_{{riemann_version}}_all.deb"
riemann_deb_file_fullpath: "{{ riemann_download_directory }}/{{ riemann_download_url | basename }}"

# The checksum file will be downloaded separately and the MD5sum (best that's offered)
# verified prior to installing the deb package.
riemann_checksum_url: "{{ riemann_download_url }}.md5"
riemann_checksum_fullpath: "{{ riemann_download_directory }}/{{ riemann_checksum_url | basename }}"

# Ignored by default. Override to add Slack alerts based on tags.
# Requires keys: account, token, channel. Example structure:
#
#   riemann_slack_credentials:
#     channel: "#alerts"
#     account: exampleorg
#     token: jf4PAD9s1T1N6m5S6iMngAfG
#
# Note: the value for "token" should be the last element after separating
# the URL by '/'. So paste the webhook URL into this one-liner:
#
#   perl -F/ -lanE 'say $F[-1]'
#
# and store the returned value as the "token".
riemann_slack_credentials: {}

# Name of the Logstash tag to trigger alerts.
riemann_slack_tag: slack_alert

# Disable verbose logging, useful for standing up the server.
riemann_debug_logging: false

# Allow override ability of installed JVM
riemann_jvm_pkg: default-jre-headless

# Remote config directories to be created. Config files, via fileglob
# or otherwise, will be added to these directories.
riemann_create_folders:
  - "conf.d"
  - "utils"
  - "vars"

riemann_default_templates:
  - src: riemann.config.j2
    dest: /etc/riemann/riemann.config
  - src: slack-alerts.clj.j2
    dest: /etc/riemann/conf.d/slack-alerts.clj

# Override the default template with a unified config file.
riemann_optional_baseconfig: ""
# Provide additional config files, via fileglob.
riemann_optional_addconfs: []
riemann_optional_addutils: []

# Key/values that riemann can import and utilize
riemann_alerts_auth_map: {}


# Additionally added Configuration

# SMTP Host for sending emails
riemann_email_host: "smtp.google.com"
# use ssl for email sending
riemann_email_ssl: true
# authentication user
riemann_email_user: "foo@example.com"
# authentication password
riemann_email_pass: "VerySecurePassword"
# from-header for mail
riemann_email_from: "foo@example.com"

# receiver for notifications
riemann_email_receiver: "bar@example.com"


```

Example Playbook
----------------

```
- name: Install Riemann server.
  hosts: logserver
  become: yes
  vars_files:
    - vars/slack_api.yml
  roles:
    - role: freedomofpress.riemann
  tags: riemann
```

Running the tests
-----------------

This role uses [Molecule] and [ServerSpec] for testing. To use it:

```
pip install molecule
gem install serverspec
molecule test
```

You can also run selective commands:

```
molecule idempotence
molecule verify
```

See the [Molecule] docs for more info.

Further Reading
---------------
The following resources were invaluable in creating this role.

* [Riemann official guide](http://riemann.io/howto.html#putting-riemann-into-production)
* [Clojure for the Brave and True](http://www.braveclojure.com/do-things/)
* [An Introduction to Riemann](https://kartar.net/2014/12/an-introduction-to-riemann/)
* [Programmatic Real-time Monitoring and Alerting with Riemann](http://www.stuartgunter.org/programmatic-realtime-monitoring-alerting-riemann/)
* [Riemann Slack documentation](http://riemann.io/api/riemann.slack.html)
* [Ansible role for Riemann by @dhruvbansal](https://github.com/dhruvbansal/riemann-server-ansible-role)

License
-------

MIT

[Molecule]: http://molecule.readthedocs.org/en/master/
[ServerSpec]: http://serverspec.org/
[freedomofpress.elk]: https://github.com/freedomofpress/ansible-role-elk
[Riemann]: http://riemann.io
[Slack]: https://slack.com
