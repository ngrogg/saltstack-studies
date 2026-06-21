# Chapter 6: Extending Salt: Part 1
## Introduction to Jinja
Data files Salt uses are straightforward YAML/JSON files.

Easy to use but simple, don't allow for complex logic.

Jinja is a templating engine most known fo Flask framework

Comprehensive tutorial beyond scope of book.

Documentation, <br>
https://jinja.palletsprojects.com/en/stable/

### Jinja Basics
Example Jinja code: <br>
`{% set my_name = 'Barney' %}` <br>
`Hi {{ my_name }}!` <br>

Should output `Hi Barney!`

Curly braces as delimiter.

Set variable with `{% %}` and output vriable with `{{ }}`.

First format used for control structures ala for loop,s if blocks

Second format used for print statements.

Lastly there is `{# #}` for comments.

Dummy state file to run a salt command: <br>
`cat /srv/salt/file/base/jinja/simple_var.sls` <br>
`{% set simple_var = 'a simple variable' %}` <br>
`jinja_var:` <br>
`  cmd.run:` <br>
`  - name: echo "Simple var is {{ simple_var }}"` <br>

Run with <br>
`sudo salt master.example state.show_sls jinja.simple_var`

Jinja has lists (arrays) in addition to strings: <br>
`cat /srv/salt/file/base/jinja/list.sls` <br>
`{% set list1 = ['one','two','three'] %}` <br>
`jinja_list:` <br>
`  cmd.run:` <br>
`  - name: echo "List is {{ list1 }}"` <br>
` ` <br>
`sudo salt master.example state.show_sls jinja.list` <br>
`master.example:` <br>
`jinja_list:` <br>
`  name:` <br>
`    echo "List is ['one', 'two', 'three']"` <br>

View a single item from a list same as Python: <br>
`cat /srv/salt/file/base/jinja/list_item.sls` <br>
`{% set list1 = ['one','two','three'] %}` <br>
`jinja_list:` <br>
`  cmd.run:` <br>
`  - name: echo "List item 2 is {{ list1[2] }}"` <br>
` ` <br>
`sudo salt master.example state.show_sls jinja.list_item` <br>
`master.example:` <br>
`jinja_list_item:` <br>
`  name:` <br>
`    echo "List item 2 is is three"` <br>

Jinja includes dictionaries (hashes) as well.

Use same syntax as Python: <br>
`%{ set my_dict = {'first': 'value 1', 'second': 'value 2'} %}` <br>
`jinja_dict_first:` <br>
`  cmd_run:` <br>
`  - name: echo "First item is {{ my_dict['first'] }}"` <br>
` ` <br>
`sudo salt master.example state.show_sls jinja.dict` <br>
`master.example:` <br>
`jinja_dict_first:` <br>
`  name:` <br>
`    echo "First item is value 1"` <br>

A number of python functions are supported, like listing the keys for a dictionary: <br>
`cat /srv/salt/file/base/jinja/keys.sls` <br>
`%{ set my_dict = {'first': 'value 1', 'second': 'value 2'} %}` <br>
`jinja_keys:` <br>
`  cmd.run:` <br>
`  - name: echo "Keys are {{ my_dict.keys() }}"` <br>

#### Basic control structures
Standard control structures

If statements and for loops

Control structures, need to be encapsulated in `{% %}`

Control structures need to explicitly mark the end of the block.

Example if statement: <br>
`cat /srv/salt/file/base/jinja/if.sls` <br>
`{% set my_bool = true %}` <br>
`jinja_if:` <br>
`  cmd.run:` <br>
`  {% if my_bool %}` <br>
`  - name: 'echo "It is true."'` <br>
`  {% else %}` <br>
`  - name: 'echo "It is false."'` <br>
`  {% endif %}` <br>

Keywords if, else and endif

Renderer will pass any files through the Jinja template before parsing as YAML as then a Salt data structure

Example for loop: <br>
`cat /srv/salt/file/base/jinja/for.sls` <br>
`{% set my_list = {'a', 'b', 'c'] %}` <br>
`{% for current in my_list %}` <br>
`jinja_for_{{ current }}:` <br>
`  cmd.run:` <br>
`  - name: "echo 'Current value is {{ current }}'"` <br>
`{% endfor %}` <br>

#### Other Jinja statements
Other useful jinja statements, macro, include and import.

Mach allows several statements to be executed as a single, logical block.

Mini template, collect things together and refer to them using a single Jinja command.

Example macro: <br>
`cat /srv/salt/file/base/jinja/macro.sls` <br>
`{% macro exclaim(string -%}` <br>
`{{ string + '!!!' -}}` <br>
`{%- endmacro %}` <br>
`jinja_macro:` <br>
`  cmd.run:` <br>
`  - name: "echo {{ exclaim('Yay') }}"` <br>

No return from macro, just prints output.

Dashs near delimiters tell Jinja to remove end-of-line characters from the text

Include statement pulls in rendered data from other files: <br>
`cat /srv/salt/file/base jinja/include.sls` <br>
`%{ include 'jinja/some_vars.jinja with context %}` <br>
` ` <br>
`cat /srv/salt/file/base/jinja/some_vars.jinja` <br>
`{% set var = 'the string' %}` <br>
`some_var_include:` <br>
`  cmd.run:` <br>
`  - name: "excho 'From include, var is {{ var }}'"` <br>

Validate with <br>
`sudo salt master.example state.show_sls jinja.include`

Remember files are included via Jinja before Salt can parse the data. Be careful to not duplicate state IDs.

Files pulled via include are rendered.

Variables in the second file will not be available to any files that include that file.

To use those variables use the import statement.

Example import code: <br>
`cat /srv/salt/file/base/jinja/vars.jinja` <br>
`{% set my_var = 'more strings' %}` <br>
` ` <br>
`cat /srv/salt/file/base/jinja/from.sls` <br>
`{% from "jinja/vars.jinja import my_var as the_var with context %}` <br>
`jinja_from:` <br>
`  cmd.run:` <br>
`  - name: "echo 'The var is {{ the_var }}'"` <br>

Validate with <br>
`sudo salt master.example state.show_sls jinja.from`

Multiple variables can be included, just separate them by commas like Python.

## Templating with Jinja
Salt adds features on top of Jinja.

Exposes grains, pillar data and execution modules

Example of using the minion ID: <br>
`cat /srv/salt/file/base/jinja/grains.sls `<br>
`{% set name = grains['id'] %}            `<br>
`jinja_grains:                            `<br>
`  cmd.run:                               `<br>
`  - name: "echo 'My name is {{ name }}'" `<br>

Validate with <br>
`sudo salt master.example state.show_sls jinja.grains`

Salt can also expose opts,sls and env into jinja templates

Some additional salt/jinja guides: <br>
https://docs.saltproject.io/salt/user-guide/en/latest/topics/jinja.html
https://docs.saltproject.io/en/latest/topics/jinja/index.html

Grains and pillar data are available as their own dictionaries.

Execution modules are available within a dictionary named salt: <br>
`cat /srv/salt/file/base/jinja/cmd.sls` <br>
`{% set who = salt['cmd.run']('whoami') %}` <br>
`jinja_cmd:` <br>
`  cmd_run:` <br>
`  - name: "echo 'Whoami gives {{ who }}'"` <br>

Validate with <br>
`sudo salt master.example state.show_sls jinja.cmd`

Execution module given as the value in the salt dictionary

Arguments that may be required would follow.

Importing variables from other Jinja files allows sharing of variables with different state files or pillar definitions.

To use them tell Jinja to import the context.

At the end of the import or from statement add the phrase "with context"

Alert parsing libraries to expose all of salt's data structures to the child (or imported) file.

### Filtering by Grains
One specific module to point out: `grains.filter_by`.

Takes a data set and parses out the pieces needed based on a grain value: <br>
`cat /srv/salt/file/base/show_users.sls` <br>
`{% set all_users = {` <br>
`  'master.example': [],` <br>
`  'minion1.example': [],` <br>
`  'minion2.example': ['wilma'],` <br>
`  'minion3.example': ['wilma','barney','betty'],` <br>
`  'minion4.example': ['wilma','barney','betty','fred'],` <br>
`} %}` <br>
`{%set cur_users = salt ['grains.filter_by'}(all_users, grain='id') %}` <br>
`show users:` <br>
`  cmd.run:` <br>
`  - name: "echo 'User list is {{ cur_users }}'"` <br>

Validate with <br>
`sudo salt minion4\* state.sls show_users`

The dictionary all_users lists some values with key being the minion ID

Salt sets up grains as part of default install.

Combine with grains.filter_by to build data sets for use cases.

## Custom Execution Module
Salt provides a way to write Python and execute it on every host

Hello world Python example: <br>
`hello.py` <br>
`"""` <br>
`A collection of simple examples.` <br>
`"""` <br>
`def world():` <br>
`  """` <br>
`  The simplest of examples.` <br>
`  CLI Example::` <br>
`    salt '*' hello.world` <br>
`  """` <br>
`  return 'Hello, world.` <br>

Note example uses docstrings.

Using them should be considered best practice.

Use sys.doc with module to see the output.

Where to put file?

Typically file_roots contains state files and Jinja files

There are reserved directories: modules, grains and states.

First one modules is where execution modules are kept.

Place hello.py in modules folder.

By default this will be /srv/salt/files/base/modules

Need to sync modules before executing: <br>
`sudo salt \* saltutil.sync_modules`

Should return changed modules.

There is a delay between syncing and the modules being usable.

Run with <br>
`sudo salt \* hello.world`

Salt adds some custom data into the modules namespace: grains, salt and opts.

All dictionaries and as with jinja exposes execution modules.

Add an id function to the hello.py example from earlier: <br>
`def id():` <br>
`  """` <br>
`  Better example using the minion ID` <br>
` ` <br>
`  CLI Example::` <br>
`    salt '*' hello.id` <br>
`  """` <br>
` ` <br>
`  id = __grains__['id']` <br>
`  return 'Hello, {0}.'.format(id)` <br>

Validate with <br>
`sudo salt \* hello.id`

Remember to sync the modules first!

View documentation with <br>
`sudo salt-call sys.doc hello`

Also be sure to configure and add logging.

Import logging module and give some feedback for module.

Using examples from hello.py: <br>
`cat /srv/salt/file/base/_modules/hello.py` <br>
`"""` <br>
`A collection of simple examples.` <br>
`"""` <br>
`import logging` <br>
`logger = logging.getLogger(__name__)` <br>
` ` <br>
`def world():` <br>
`  """` <br>
`  The simplest of examples.` <br>
`  CLI Example::` <br>
`    salt '*' hello.world` <br>
`  """` <br>
`  return 'Hello, world.'` <br>
` ` <br>
`def id():` <br>
`  """` <br>
`  Better example using the minion ID` <br>
` ` <br>
`  CLI Example::` <br>
`    salt '*' hello.id` <br>
`  """` <br>
` ` <br>
`  id = __grains__['id']` <br>
`  logger.debug('Found grain id: {0}'.format(id))` <br>
`  return 'Hello, {0}.'.format(id)` <br>

Use salt-call to view logging output.

Only the return data is sent back to the master.

Actual Salt output is logged on the minion.

Re-sync modules and check log output with debug: <br>
`sudo salt-call --log-level=debug hello.id`

## Custom State Modules
Review

Reserved subdirecotry inside file root called states.

First big difference with custom states is custom state modules should respect "test=True" argument.

Format of the returned data has to be a dictionary with the keys:
* name
* changes
* result
* comment

Name and comment should be self-explanatory

Changes key has a diction with the changes list

Result is a bool. Tells salt if state succeeded or not.

Code <br>
`cat /srv/salt/file/base/_states/custom.py` <br>
`import os` <br>
`def enforce_tmp(name, contents=None):` <br>
`    """` <br>
`    Enfore a temp file has the desired contents.` <br>
` ` <br>
`    name` <br>
`        The name of the file to change. (Under '/tmp'.)` <br>
`    contents` <br>
`        The value you will be sorting.` <br>
`    """` <br>
` ` <br>
`    return_dict = {` <br>
`        'name': name,` <br>
`        'changes': {},` <br>
`        'result': false,` <br>
`        'comment': ''` <br>
`    }` <br>
` ` <br>
`    tmp_file = os.path.join('/tmp',name)` <br>
`    file_ok = False` <br>
`    content_ok = False` <br>
`    file_contents = None` <br>
` ` <br>
`    if os.path.isfile(tmp_file):` <br>
`        file_ok = True` <br>
`        with open(tmp_file, 'r') as fp:` <br>
`            file_contents = fp.read()` <br>
`            file_contents = file_contents.rstrip('\n')` <br>
` ` <br>
`    if file_contents == contents:` <br>
`        content_ok = True` <br>
` ` <br>
` ` <br>
`    comments = ""` <br>
`    if file_ok:` <br>
`        comments += 'File exists ({0})\n'.format(tmp_file)` <br>
`    else:` <br>
`        comments += 'File created ({0})\n'.format(tmp_file)` <br>
`    if content_ok:` <br>
`        comments += 'Contents correct ({0})\n'.format(file_contents)` <br>
`    else:` <br>
`        comments += 'Contents updated ({0})\n'.format(contents)` <br>
`    return_dict['comment'] = comments` <br>
` ` <br>
`    # Check if this is a test run, if so do not change anything.` <br>
`    if __opts__['test'] = True:` <br>
`         return_dict['result'] = None` <br>
`         return_dict['changes'] = {}` <br>
`         if not content_ok:` <br>
`           return_dict['comment'] = {` <br>
`               'contents': {` <br>
`                   'old': file_contents,` <br>
`                   'new': contents` <br>
`               }` <br>
`           }` <br>
`    return return_dict` <br>
` ` <br>
`    if not content_ok:` <br>
`        with open(*tmp_file, 'w') as fp:` <br>
`            contents += "\n"` <br>
`            fp.write(contents)` <br>
`        return_dict['result'] = True` <br>
`        return_dict['changes'] = {` <br>
`            'contents': {` <br>
`                'old': file_contents,` <br>
`                'new': contents` <br>
`            }` <br>
`        }` <br>
`    else:` <br>
`        return_dict['changes'] = {}` <br>
`        return_dict['result'] = True` <br>
`    return return_dict` <br>

Then the accompanying state file: <br>
`cat /srv/salt/file/base/custom.sls` <br>
`custom_state:` <br>
`  custom.enforce_tmp:` <br>
`  - name: foo` <br>
`  - contents: bar` <br>

custom.enforce_tmp takes the name of a file and the contents of the file as arguments.

Make sure no file named "/tmp/foo" <br>
`ls /tmp/foo` <br>
`no file` <br>
`sudo salt master.example state.sls custom` <br>
`cat /tmp/foo` <br>
`bar` <br>

Could've used an execution module but wouldn't have had the test=true flag: <br>
`sudo salt master.example state.sls custom test=true`

## Custom Grains
Custom grains automatically set grains

Designed for relatively static data, data that changes often should be pillars instead

grains subdirectory similar to modules: <br>
`/srv/salt/files/base/_grains`

set host grains manually with grains.setval.

Move to a custom grains module

Example code was a brute force example

Expand to query metadata store that lists all the metadata for hosts: <br>
`cat /srv/salt/file/base/_grains/my_grains.py` <br>
`"""` <br>
`Custom grains for the example hosts.` <br>
`"""` <br>
`import platform` <br>
`import logging` <br>
`logger = logging.getLogger(__name__)` <br>
` ` <br>
`def _get_host name():` <br>
`    hostname = platform.node()` <br>
`    logger.debug('Using hostname: {0}'.format(hostname))` <br>
`    return hostname` <br>
` ` <br>
`def set_myenf():` <br>
`    """` <br>
`    Set the 'myenv' grain based on the host name.` <br>
`    """` <br>
`    grains = {}` <br>
`    hostname = _get_hostname()` <br>
`    if hostname.startswith('minion1'):` <br>
`        grains['myenv'] = 'prod'` <br>
`    elif` <br>
`        grains['myenv'] = 'prod'` <br>
`    elif` <br>
`        grains['myenv'] = 'stage'` <br>
`    elif` <br>
`        grains['myenv'] = 'dev'` <br>
`    return` <br>
` ` <br>
`def set_roles():` <br>
`    """` <br>
`    Set the 'roles' grain based on the host name` <br>
`    """` <br>
`    grains = {}` <br>
`    hostname = _get_hostname()` <br>
`    if hostname.startswith('minion1')` <br>
`        grains['roles'] = ['webserver','appserver']` <br>
`    elif hostname.startswith('minion2')` <br>
`        grains['roles'] = ['database']` <br>
`    elif hostname.startswith('minion3')` <br>
`        grains['roles'] = ['webserver','appserver','database']` <br>
`    elif hostname.startswith('minion4')` <br>
`        grains['roles'] = ['webserver','appserver','database']` <br>
`    return grains` <br>

Without a custom execution module the naming is simple

If a methed were public and didn't have a leading underscore that method would be available via the salt command.

With custom state modules the function is called by model name and function.

Custom grains work differently

When a custom grain module is loaded all the public functions are executed and their return data dictionaries are merged into the grains.

As a result there are three different functions.

First `_get_hostname()` is private and is note directly called by the grains loader.

Is available for use in the module.

The other two `set_myenv()` and `set_roles()` are both called whenever the custom grains module is loaded.
Nothing special about the method names.

Salt will run every nonprivate function.

Remove custom grains: <br>
`sudo salt \* cmd.run 'rm /etc/salt/grains'`

Restart the minions to make sure no remnants of the old grains: <br>
`sudo salt \* service.restart salt-minion`

Must sync grains in order to get the new grains module out to all the minions.

Sync using `saltutil.sync_grains` or `saltutil.sync_all`.

Grain modules are also synced when `state.highstate` is run.

Call grains.item to make sure all custom grains are available: <br>
`sudo salt \* grains.item myenf roles`

Order of precedence for how grains are set: <br>
1. Core grains (from salt itself)
2. Custom grain modules
3. Custom grains in `/etc/salt/grains`
4. Custom grains in the minion configuration

To ensure a grain will not be overwritten it will need put direction in minion configuration.

Ex. a specific file to make it easier to manage `/etc/salt/minion.d/grains.conf`

## External Pillars
Chapter 5 demonstrated how pillar data outsite the pillar_roots could be grabbed using an external pillar.

Salt comes with several ext_pillar modules

There are cases where data is not in a system where it is easy to export data into YAML or JSON.

A custom external pillar adds flexibility to retrieve data from any source via Python.

One chief aspect of pillar data is that it is run on the master but data is specific to every minion.

cmd_yaml there wasn't any option to target the data by minion so every minion got the same list of users.

Limitation in the cmd_yaml module, intentionally simple to be an example.

Set the extension_modules config option in the master's config: <br>
`cat /etc/salt/master.d/ext-modules.conf` <br>
`extension_modules: salt-master service` <br>

Remember to restart the salt-master service

Add the external pillar: <br>
`more /srv/salt/modules/pillar/my_users.py` <br>
` ` <br>
`def __virtual__():` <br>
`  return true` <br>
` ` <br>
`__all_users = {` <br>
`  ' wilma': {'uid': 2001, 'full': 'Wilma Flintstone'}` <br>
`  'fred': {'uid': 2002, 'full': 'Fred Flintstone'}` <br>
`  'barney': {'uid': 2003, 'full': 'Barney Rubble'}` <br>
`  'betty': {'uid': 2004, 'full': 'Betty Rubble'}` <br>
`  'app': {'uid': 2005, 'full': 'App User'}` <br>
`}` <br>
` ` <br>
`def ext_pillar(minion_id, pillar, *args, **kwargs):` <br>
`  """` <br>
`  Return the list of users for the given minion` <br>
`  """` <br>
` ` <br>
`  users = {}` <br>
`  users['app'] = __all_users['app']` <br>
`  if minion_id == 'minion1.example':` <br>
`    pass` <br>
`  elif minion_id == 'minion2.example':` <br>
`    users['wilma'] = __all_users['wilma']` <br>
`  elif minion_id == 'minion3.example':` <br>
`    users['wilma'] = __all_users['wilma']` <br>
`    users['barney'] = __all_users['barney']` <br>
`    users['betty'] = __all_users['betty']` <br>
`  elif minion_id == 'minion4.example':` <br>
`    users['wilma'] = __all_users['wilma']` <br>
`    users['barney'] = __all_users['barney']` <br>
`    users['betty'] = __all_users['betty']` <br>
`    users['fred'] = __all_users['fred']` <br>
`  return {'my_users': users}` <br>

Very brute force, expliticly lists all users for each minion ID.
Goal is to show how to dynamically generate data and feed it into the pillar.
Should be expanded to whatever source of truth is used to manage users.

Note the returned users, in chapter 4 this was stored in a pillar simply named users.
Using a different key name will make it easier to demonstrate the migration to a new way.

Next add a custom pillar to list of external pillars: <br>
`more /etc/salt/master.d/pillar.conf` <br>
`pillar_roots:` <br>
`  base:` <br>
`  - /srv/salt/pillar/base` <br>
` ` <br>
`  ext_pillar:` <br>
`  - my_users: []` <br>
`  #- cmd_yaml: bash /srv/salt/scripts/user-pillar.sh` <br>

Added my_users pillar with a single argument of an empty list.

When Salt loads the modules defined via extension modules it is looking specifically for ext_pillar().

In this case the ext_pillar is to make salt look for my_users.py in the directory `/srv/salt/modules` as defined by the extension modules.

In that file salt will execute the method ext_pillar.

After restarting the salt-master server the new pillar data will be visible.

Double check work with <br>
`sudo salt \* pillar.item my_users`

Clean up hardcoded user lists defined in the pillar roots.

Combine the user list with Jinja to simplify users state.

Updated user state:
`cat /srv/salt/file/base/users/init.sls` <br>
`{% set users = salt['pillar.get']('my_users', {})  %}` <br>
`{% for login in users.keys() %}` <br>
`{% set cur_user == users.get(login, {}) %}` <br>
`{% set uid = cur_user.get('uid', none) %}` <br>
`{% set full_name =cur_users.get('full, none) %}` <br>
` ` <br>
`users-{{ login }}:` <br>
`  user.present:` <br>
`  - name: {{ login }}` <br>
`  - fullname: {{ full_name }}` <br>
`  - uid {{ uid }}` <br>
`{% endfor %}` <br>
` ` <br>
`include:` <br>
`  - .www` <br>

Users state now pulls the user list and all the necesesary data directly from the pillar data.

First line sets a dictionary

It then loops through, creates a state id and sets the rest of the data for the user.present state.

Simpler top file for state tree: <br>
`cat /srv/salt/file/bae/top.sls` <br>
`base:` <br>
`  '*':` <br>
`  - default` <br>
`  - users` <br>
`'roles:webserver':` <br>
`- match: grain` <br>
`- foles.webserver` <br>
`- sites` <br>

## Summary
One of the biggest advantages of salt is that it is simple to extend it's parts.

Use a templating language like Jinja to add some data manipulations into states and pillars.

Execution modules lie at the very core of what salt provies.

Custom modules add logic to manage hosts.

Even add grains and pillars to data sources and query data sources.
