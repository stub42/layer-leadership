# Leadership Layer for Juju Charms

The Leadership layer is for charm-tools and 'charm build', making it
easier for layered charms to deal with Juju leadership.

This layer will initialize charms.reactive states, allowing you to
write handlers that will be activated by these states. It allows you
to completely avoid writing leader-elected and leader-settings-changed
hooks. As a simple example, these two handlers are all that is required
to make the leader unit generate a password if it is not already set,
and have the shared password stored in a file on all units:

```python
from reactive.leadership import leader_get, leader_set
from charmhelpers.core.host import pwgen


@when('leadership.is_leader')
@when_not('leadership.set.admin_password')
def generate_secret():
    leader_set(admin_password=pwgen())


@when('leadership.changed.admin_password')
def store_secret():
    write_file('/etc/foopass', leader_get('admin_password'))
```


## States

The following states are set appropriately on startup, before any @hook
decorated methods are invoked:

* `leadership.is_leader`

  This state is set when the unit is the leader. The unit will remain
  the leader for the remainder of the hook, but may not be leader in
  future hooks.

* `leadership.set.{varname}`

  This state is set for each leadership setting (ie. the
  `leadership.set.foo` state will be set if the leader has set
  the foo leadership setting to any value). It will remain
  set for the remainder of the hook, unless the unit is the leader
  and calls `reactive.leadership.leader_set()` and resets the value
  to None.

* `leadership.changed.{varname}`

  This state is set for each leadership setting that has changed
  since the last hook. It will remain set for the remainder of the
  hook. It will not be set in the next hook, unless the leader has
  changed the leadership setting yet again.


## Methods

The `reactive.leadership` module exposes the `leader_set()` and
`leader_get()` methods, which match the methods found in the
`charmhelpers.core.hookenv` module. `reactive.leadership.leader_set()`
should be used instead of the charmhelpers function to ensure that
the reactive state is updated when the leadership settings are. If you
do not do this, then you risk handlers waiting on these states to not
be run on the leader (because when the leader changes settings, it 
triggers leader-settings-changed hooks on the follower units but
no hooks on itself).


## Support

This layer is maintained on Launchpad by
Stuart Bishop (stuart.bishop@canonical.com).

Code is available using git at git+ssh://git.launchpad.net/layer-leadership.

Bug reports can be made at https://bugs.launchpad.net/layer-leadership.

Queries and comments can be made on the Juju mailing list, Juju IRC
channels, or at https://answers.launchpad.net/layer-leadership.
