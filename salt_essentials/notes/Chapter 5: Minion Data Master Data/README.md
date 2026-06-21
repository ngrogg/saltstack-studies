# Chapter 5: Minion Data/Master Data
Salt runs on top of other systems like operating systems.

Even the operating system runs on top of hardware.

## Grains are Minion Data
Grains are calculated when the minion starts.

Static data.

Data generated on the minion itself, then sent to the master.

Salt minion will have grains set up by defualt.

Can be expanded by including a static list or using Python

### Performing Basic Grain Operations
Default grains already configured.

List all of them with `grains.ls`: <br>
`sudo salt master.example grains.ls`

Use `grains.items` to list all of the keys and their values.

Call `grains.item os` to show the value of the os grain.

Use `--out=txt` to simplify output: <br>
`sudo salt \* grains.item os --out=txt`

### Setting Grains
Can use minion IDs to configure them, can be cumbersome.

Use grains to set own metadata about a host.

Target hosts based on combinations of that data.

Several ways to set grains.

Set grains from the master itself.

Set two grains: `myenv` and `roles`.

Set minion1 and 2 to prod: <br>
`sudo salt -E 'minion(1|2).*' grains.setval myenv prod`

Set minion3 to stage: <br>
`sudo salt 'minion3.*' grains.setval myenv stage`

Set minion4 to dev: <br>
`sudo salt 'minion4.*' grains.setval myenv dev`

Set minion1 role: <br>
`sudo salt 'minion1.*' grains.setval roles '[webserver,appserver]'`

Set minion2 role: <br>
`sudo salt 'minion2.*' grains.setval roles '[database]'`

Set minion 3 and 4 roles: <br>
`sudo salt -E 'minion(3|4).*' grains.setval roles '[webserver,appserver,database]'`

Note the Perl regex using the -E flag

For more PCRE info visit: https://www.pcre.org

List roles with `grains.item`: <br>
`sudo salt \* grains.item myenv roles`

Data written into /etc/salt/grains

Salt will retain config that way.

File is a YAML file

### Targeting with Grains in the Top File
Previously used minions for top file.

Can now use grains instead: <br>
`cat /srv/salt/file/base/top.sls` <br>
`base:` <br>
`  'os:CentOS'` <br>
`  - match: grain` <br>
`  - default.vim-enhanced` <br>
` ` <br>
`  'os:Ubuntu'` <br>
`  - match: grain` <br>
`  - default.vim` <br>
` ` <br>
`  'roles:webserver'` <br>
`  - match: grain` <br>
`  - roles.webserver` <br>
`  - sites` <br>
` ` <br>
`  'roles:database'` <br>
`  - match: grain` <br>
`  - users.dba` <br>
` ` <br>
`  'myenv:stage'` <br>
`  - match: grain` <br>
`  - users.qa` <br>
` ` <br>
`  'myenv:dev'` <br>
`  - match: grain` <br>
`  - users.all` <br>
`  - run_first` <br>

Removed all targets with minion IDs and replaced them with grains.

Verify with state.show_top: <br>
`sudo salt minion4\* state.show_top`

Grains are power way of assigning metadata but are mostly static.

For more dynamic data use pillars.

## Pillars are data from the master
Grains are set on the minion, pillars are set/stored on the master.

Only available for the given minion.

Useful for same key but different values for different minions.

Minion can only see it's own pillar data.

### Querying Pillar Data
Almost like querying grains: <br>
`sudo salt minion4.example pillar.items`

All under a key called `master`.

Actually the master's configuration settings.

Output of `pillar.items` can be large.

Can be disabled with the `pillar_opts` configuration value in the master's configuration.

Set this to `False` and the output should be more manageable.

May lose important data however.

Use pillar data to set the list of userse for each host.

Stored in a YAML file like most of Salt's data.

First set the pillar root in the master's config: <br>
`cat /etc/salt/master.d/pillar.conf` <br>
`pillar_roots:` <br>
`  base:` <br>
`  - /srv/salt/pillar/base` <br>

Need to restart salt-master to pick up change.

Setup is similar to state files.

Pillar file also needs a top file.

Contains targeting information: <br>
`cat /srv/salt/pillar/base/top.sls` <br>
`base:` <br>
`  '*':` <br>
`  - default` <br>

Simple default with test data: <br>
`cat /srv/salt/pillar/base/default.sls` <br>
`my_data: some data for stuff` <br>

Create some user data for each group of users.

Pillar for all users, staging users and DBA users: <br>
`cat /srv/salt/pillar/base/users/all.sls` <br>
`users:` <br>
`  wilma: 2001` <br>
`  fred: 2002` <br>
`  barney: 2003` <br>
`  betty: 2004` <br>
` ` <br>
`cat /srv/salt/pillar/base/users/stage.sls` <br>
`users:` <br>
`  wilma: 2001` <br>
`  barney: 2003` <br>
`  betty: 2004` <br>
` ` <br>
`cat /srv/salt/pillar/base/users/dba.sls` <br>
`users:` <br>
`  wilma: 2001` <br>

Similar to state files, however a pillar file cannot be directly called.

Collection of files compiled to create a data set for minion.

Use a top file to pull it all together: <br>
`cat /srv/salt/pillar/base/top.sls` <br>
`base:` <br>
`  '*':` <br>
`  - default` <br>
` ` <br>
`  'G@myenv and G@roles:database':` <br>
`  - match: compound` <br>
`  - users.dba` <br>
` ` <br>
`  'myenv:stage':` <br>
`  - match: grain` <br>
`  - users.stage` <br>
` ` <br>
`  'myenv:dev':` <br>
`  - match: grain` <br>
`  - users.all` <br>

Query pillar data for every minion: <br>
`sudo salt \* pillar.item users`

Does only duplicate state files at this point.

Only meant to introduce pillar data.

### Querying other sources with external pillars
Pillar system provides a data layer but all the data must be in YAML files within the pillar file tree.

Query external data sources using the external pillars via `ext_pillar`.

System has built in options from MySQL to Git to Amazon's S3. For this example using `cmd_yaml`.

Runs a command, parses the output as YAML and adds it to the pillar data.

External pillars are run on the master not the minion.

Needs to be on the master but doens't need to be in the salt files.

Example: <br>
`cat /srv/salt/scripts/user-pillar.sh` <br>
`#!/bin/bash` <br>
`echo "users:"` <br>
`echo "  app: 9001"` <br>

Data will be added to the pillar data by salt.

Double checking users on host should show new user: <br>
`sudo salt minion4.example pillar.item users`

## Renderes Give Data Options
Defaults are YAML.

Salt is written in Python, will eventually translate into Python data structures.

When salt needs data from an SLS or pillar file is parsed using a templating engine.

Other supported examples are Jinja, Mako, Wempy and in YAML or JSON format.

Default renderer is defined in the renderer config option.

Book mostly uses YAML, JSON example below: <br>
`cat /srv/salt/file/base/json.sls` <br>
`#!json` <br>
`{` <br>
`  "json_test: {` <br>
`    "cmd.run": [` <br>
`      {` <br>
`        "name": "echo \"Json test\""` <br>
`      }` <br>
`    ]` <br>
`  }` <br>
`}` <br>

First line contains shebang type directive.

Rest of the file is JSON.

Will be read same as YAML.

Double check work with: <br>
`sudo salt master\* state.sls json`
