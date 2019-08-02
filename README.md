# Ansible Role: Borg Client

[![Ansible Galaxy Badge](https://img.shields.io/ansible/role/41596.svg)](https://galaxy.ansible.com/yurihs/borg_client)


- Installs BorgBackup.
- Installs a script to create and prune archives, and manages configurations for the script (prune frequency, connection parameters).
- Configures cron jobs for the executing the script regularly.

Made to work with my other role, [yurihs.borg_server](https://github.com/yurihs/ansible-role-borg-server).

## Role variables (default values)

~~~yaml
borg_client_config_dir: /etc/borg_client
~~~

Where the configurations will be stored.

~~~yaml
borg_client_log_path: /var/log/borg_client.log
~~~

Where the output of the script will be written to (from the cron jobs).

~~~yaml
borg_client_backup_list: []
~~~

A list of backups to manage. Each item should have:

* `name`: Just to identify it. Will be used in the configuration file name and the cron job.
* `repo`: Where to create the archive. Usually a remote server.
* `paths`: A list of locations to include in the backup.

Each item may also have a cron configuration, just like in the [cron module](https://docs.ansible.com/ansible/2.8/modules/cron_module.html).

* `cron`
  * `minute`
  * `hour`
  * `day`
  * `month`
  * `weekday`

Each item may also define an alternative SSH configuration.

* `ssh`
  * `key_path`: Specify which key to use when connecting.
  * `port`: If your remote server uses a nonstandard port. (optional. default: 22)

Each item may also have pruning enabled, by including the following parameters. If so, `borg prune` will be executed after every archive creation.

* `prune`
  * `keep_hourly`
  * `keep_daily`
  * `keep_weekly`
  * `keep_monthly`


## Example

First, create a repo on the client, like so:

~~~sh
borg init --encryption=none borg@backup.example.com:app_uploads
~~~

~~~yaml
- hosts: web
  vars:
    borg_client_backup_list:
      - name: app_uploads
        repo: borg@backup.example.com:app_uploads
        paths:
          - /var/www/app/uploads
        cron:
          minute: '0'
          hour: '*'
          day: '*'
          month: '*'
          weekday: '*'
        ssh:
          key_path: /root/.ssh/id_app
        prune:
          keep_hourly: 5
          keep_daily: 5
          keep_weekly: 2
          keep_monthly: 2
  roles:
    - role: yurihs.borg_client
      become: true
~~~

## Missing features (TODO)

- Handle encryption (keyfiles, passphrases)


