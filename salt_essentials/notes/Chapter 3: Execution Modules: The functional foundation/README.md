# Chapter 3: Execution Modules: The Functional Foundation

## sys: Information and Documentation about modules
List modules and functions loaded.

### sys.doc Basic Documentation
sys.doc will list the docstrings in each function.

Python docstrings are similar to javadoc.

Use either the salt CLI on the master with a list of minions or the salt-call CLI on the minion:
`sudo salt master.example sys.doc test.ping` <br>
`test.ping:` <br>
`  Used to make sure the minion is up and responding. Not an ICMP ping.` <br>
`  Returns 'True'.` <br>
`  CLI Example:` <br>
`    salt '*' test.ping` <br>

sys.doc function takes name of a function or module as arguments.

To see all functions in a module give sys.doc an argument of test:
`sudo salt-call sys.doc test` <br>
`local:` <br>
`  ------` <br>
`  test.arg` <br>
`    Printout the data passed into the function '*args' ...` <br>

To see documentation for all the modules on the system run sys.doc without any argument.

Documentation from python code itself not a set of files.

### sys.list_modules, sys.list_functions: Simple Listings
Functions aimed at providing higher level view of the modules and functions available.

Two modules: <br>
`sys.list_modules` <br>
`sys.list_functions` <br>

Take a modules name as arguments.

No argument lists all modules or functions on minion: <br>
`sudo salt-call sys.list_modules` <br>
`local:` <br>
`  - acl` <br>
`  - aliases` <br>
`sudo salt-call sys.list_functions sys ` <br>
`local: ` <br>
`  - sys.argspec ` <br>
`  - sys.doc ` <br>
`  - sys.list_functions ` <br>
`  - sys.list_modules` <br>

sys.argspec will list arguments and default vaules for each function.

## cmd: Execute via a Shell
Powerful but insecure module to run a command as root on a minion.

### cmd.run: Run Any Command
Most straightforward cmd command.

Run a command on every minion as if typing it in a normal shell.

Example: <br>
`sudo salt \* cmd.run 'grep root /etc/passwd'` <br>
`minion3.example:` <br>
`  root:x:0:0:root:/root:/bin/bash` <br>
`minion2.example:` <br>
`  root:x:0:0:root:/root:/bin/bash` <br>
`  operator:x:11:0:operator:/root:/sbin/nologin` <br>

Keyword arguments

cmd.run can change current working directory with cwd keyword: <br>
`sudo salt-call cmd.run 'pwd'` <br>
`local:` <br>
`  /root` <br>
`sudo salt-call cmd.run cwd=/usr 'pwd'` <br>
`local:` <br>
`  /usr` <br>

Salt daemons usually run as root. Use runas argument to run as another user: <br>
`sudo salt master\* cmd.run whoami runas=vagrant` <br>
`master.example:` <br>
`  vagrant` <br>

env keyword sets and environmental variable.

Needs a key equal to a value.

Set env=key=value.

Use YAML and pass to env keyword: <br>
`sudo salt-call cme.run env='{foo: bar}' 'echo $foo'` <br>
`[INFO ] Executing command 'echo $foo' in directory '/root'` <br>
`local:` <br>
`  bar` <br>

Salt uses YAML at its core, when it doubt try arguments as YAML.

Use a YAML checker,
https://yaml-online-parser.appspot.com/

## pkg: Manage Packages
Four servers, two production, two development

Web and database servers.

Want to install nginx on the web servers and MySQL on the database server

Staging/dev servers have web/database on same servers:
* minion1: production web
* minion2: production database
* minion3: staging server
* minion4: development server

### Virtual Modules
Abstracts package managers into one module, salt refers to as a "virtual module".

Module in name only.

Code used is in other modules.

sys.list_functions, run on a CentOS and Ubuntu host: <br>
`sudo salt -L master.example,minion4.example sys.list_functions pkg` <br>
`master.example:` <br>
`  - pkg.available_version` <br>
`  - pkg.check_db` <br>
`  - pkg.clean_metadata` <br>
`  - pkg.del_repo` <br>
`<snip>` <br>
`  - pkg.upgrade_available` <br>
`  - pkg.verify` <br>
`  - pkg.version` <br>
`minion4.example:` <br>
`  - pkg.available_version` <br>
`  - pkg.del_repo` <br>
`<snip>` <br>
`  - pkg.upgrade_available` <br>
`  - pkg.version` <br>
`  - pkg.version_cmp` <br>

Differences due to package manager

CentOS specific: `pkg.check_db`

Ubuntu specific: `pkg.version_cmp`

Different salt modules that wrap functionality of each package manager.

Salt doesn't hide differences within python code.

Two modules: `yumpkg.py` and `aptpkg.py`.

Both listed as "pkg" for simplicity.

### pkg.list_pkgs: List All Installed Packages
List all install packages on systems: <br>
`sudo salt \* pkg.list_pkgs`

Underlying code is different, command to run is the same.

### pkg.available_version: See What Version Will Be Installed
Check what version of package nginx would be installed on hosts: <br>
`sudo salt \* pkg.available_version nginx`

### pkg.install: Install Packages
Install nginx on a single host `minion1`: <br>
`sudo salt minion1.example pkg.install nginx`

Validate: <br>
`sudo salt minion1.example pkg.version nginx`

## user: Manage Users
Four users, different roles and scopes:
* wilma, DBA
* fred, Developer
* barney, Developer and QA
* bettey, Developer and QA

### user.add: Add Users
Add a single user using `user.add`: <br>
`sudo salt minion2.example user.add wilma`

Verify: <br>
`sudo salt minion2.example cmd.run 'grep wilma /etc/passwd'`

### user.list_users, user.info: Get User Info
Rather than using cmd.run, verify users with `user.list_users`: <br>
`sudo salt minion2.example user.list_users`

Not as informative as grepping passwd file.

Use `user.info` to get additional info: <br>
`sudo salt minion2.example user.info wilma`

Uses basic system utilities wrapped in salt, similar to pkg.

## saltutil: Access Various Salt Utilities
Salt specific utilities.

Useful when something specific needs updated: <br>
`sudo salt-call sys.list_functions saltutil | grep sync` <br>
`  - saltutil.sync_all` <br>
`  - saltutil.sync_grains` <br>
`  - saltutil.sync_modules` <br>
`  - saltutil.sync_outputters` <br>
`  - saltutil.sync_renderers` <br>
`  - saltutil.sync_returners` <br>
`  - saltutil.sync_states` <br>
`  - saltutil.sync_utils` <br>

Next to cover is `saltutil.sync_modules`.

Syncs modules. Pulls in changes.

View long running jobs.

Start long job: <br>
`sudo salt master.example cmd.run 'sleep 100'`

Look for job with saltutil: <br>
`sudo salt master.example saltutil.running`

Will return a JID among other info

Can kill by job ID (JID) if desired: <br>
`sudo salt master.example saltutil.kill_job JID`
