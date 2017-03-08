# rdopkg as packaging CLI framework (formerly: pwnpkg)


## What? Why?

I'm not the first person to discover that `fedpkg` and the underlying `rpkg`
python modules have terrible source code.

Do you think it's OK to write [over 100 lines of hacks, wrappers and
mockings](https://github.com/redhat-openstack/rdopkg/blob/master/rdopkg/actionmods/kojibuild.py)
to achieve something as simple as following?

    from fedpkg import kojibuild

    build_id = kojibuild.new_build()
    kojibuild.watch_build(build_id)

Neither do I and thus I wrote
[rdopkg](https://github.com/redhat-openstack/rdopkg)
from scratch, the Swiss army knife of RPM packaging. You can reuse any of its
parts on a level you chose. You can use the low level functions directly. You
can use higher level wrappers full of white automagic. Or you can unleash full
rdopkg superpowers with transaction-like series of idempotent steps that allow
you to fix the problem in your shell and then just `rdopkg --continue`. Just
find what you need, import it, and use it.

Although minimalist and lightweight in nature, `rdopkg` is now too big with
around 4000 SLOC (organised into modules, not a ball of spaghetti) covering
many problems we encountered in RDO, mostly related to RPM packaging. It
contains quite a few nice tricks I stole from various python projects and some
features I only wished that existed. It's time to properly share all this
awesomenes and make it (re)useful for everyone.

Oh, and also time to rewrite `fedpkg`.


## Status

After trying to work on `pwnpkg` alongside `rdopkg`, I came to a conclusion
it's simply better not to split focus and community into two projects
and thus I started refactoring `rdopkg` to easily create new CLIs.

### What has been DONE to enable `rdopkg` as a framework

 * modular actions system (see *Actions* bellow)
 * factor current code into easily maintainable and pluggable modules
 * refactor entry point/shell logic to enable reusability
 * cleanup - remove obsolete actions and modules

### What REMAINS to be done
 * https://github.com/openstack-packages/rdopkg/issues/105
 * https://github.com/openstack-packages/rdopkg/issues/107


Note that *most of the hard work and disruptive refactoring has been
completed* - **rdopkg is almost ready to build new tools**.


## The Plan

So let me share the grand vision with you. `rdopkg` will provide convenient
yet lightweight framework for building high quality modular CLI tools out of
the box. This includes:

 * minimal [declarative way](https://github.com/openstack-packages/rdopkg/blob/master/rdopkg/actions.py) to describe both CLI and trasaction-ish program flow
 * lightweight modularity with on-demand module import
 * convenient wrappers for interfacing with the system/shell
 * descriptive and [useful errors](https://github.com/openstack-packages/rdopkg/blob/master/rdopkg/exception.py)
 * nice logging with colors out of the box

Functionality shared across different packaging tools will be provided as
`pwnpkg` action modules, not limited to:

 * parsing and manipulating manipulating `.spec` files
 * interacting with `distgit` (patches management, auto rebases)
 * automagically detecting package environment

Above functionality is implemented in `rdopkg` already.

The plan is to create state of the art pluggable CLI framework as well as
a set of reusable modules for packaging tooling for RPM based systems that are
a pleasure to use both from command line and from python. A tool to create
powerful tools that automate all the steps that dosn't really require human
input while enforcing sane minimalist conventions (over configuration) that
will make it easy for packagers to swiftly package anything upstream throws at
them.


## Actions - Lightweight Plugins

`rdopkg` provides action manager that allows single definition for both
command line and programming interfaces. This is done using special
lightweight plugins called **actions** with interface declaration in
`__init__.py` directly mapping to functions in `actions.py`. This allows
modular, transaction-ish, and terminal friendly operation where `rdopkg`
drops to terminal when human interaction is required, lets you fix the
problem using the tools of your preference and then continue the transaction
with `rdopkg --continue`.

Action moduels provide lighweight modularity with **on-demand module
imports** as opposed to usual [IMPORT ALL THE PYTHON
MODULES](https://jruzicka.fedorapeople.org/pkgs/import.jpg) madness which can
take up to a second of hardcore importing for a simple command(!) as seen in
`openstackclient`.

When `rdopkg` action module gets imported, its `__init__.py` only
contains `ACTIONS` structure that describes module endpoints (actions),
which can be atomic or a series of actions that directly map by convention to
functions in `actions.py`. These are loaded on demand when module
functionality is actually required. Action modules can contain
additional python submodules that implement actual features. These ordinary
python modules can be used (imported) directly when desired, but on top of
that, `actions.py` structures this functionality into convenient consumable
chunks.

Thanks to this design, `rdopkg` modules should be reausable by default while
not bloating even with lots of different functionality with various
requirements in one tool.


## Interested?

Do you want to be the hero to rewrite `fedpkg`? Or just create your own
packaging tools? Let me know, I could help.

 * github Issues
 * IRC: `jruzicka` @ `#rdo` on freenode IRC
 * mail: `jruzicka` at `fedorapeople.org`
