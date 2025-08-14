Six Feet Up Usage
=================

The sections below are the original README. This section is specifically for how we use this script at Six Feet Up.

This script accesses Trac values via XMLRPC. This endpoint is restricted so it can only be accessed by scripts running on the Trac server.

Setup:
* Log into the Trac server
* Check out the repo
* create a virtualenv
* `$ source env/bin/activate`
* `$ pip install -r requirements.txt`
* `$ cp migrate.cfg.sfupexample migrate.cfg`

Edit the `migrate.cfg`. There are various values in `[source]` and `[target]` that need to be updated with your project name.

Credentials:
* "Trac Admin" in 1Password for admin username and password for Trac - used in `[source]`:`url`
* Christine's GitHub access token (`token` in migrate.cfg). This means that Christine's user needs to have permission to create issues in the target repo.

Running the script:
* Use a tmux session in case you get disconnected
* When running on a new project, delete the cache and attachements: `$ rm -rf trac_cache archive/*`
* With the virtualenv active, run `$ ./migrate.py`

Post-migration steps:
* The script will output `migrated_issues.csv`. Use this to run https://github.com/hillairet/ciftt for updating the approprite values on the GitHub issues
* The script also saves all the attachments to a folder (`archive/attachments/`), and these need to be uploaded, by adding them as release assets:
  * In GitHub, create a new release/tag in the repo called `ticketmigration`.
  * Clone the repo for the project (where the new issues live)
  * Move the `attachments` folder into the root of the repo. (Do not commit, this is just so that the CLI knows which project we are on)
  * From the root of the repo, use the GitHub CLI to upload the files as release assets: `gh release upload ticketmigration attachments/*`

What
=====

This script migrates milestones, issues/tickets, and wiki pages from Trac to GitHub.

The script has its origin at https://github.com/moimael/trac-to-gitlab,
which then was [extended to suite a specific use case of SVN+Trac to GitLab migration](https://www.gams.com/~stefan/svn2git/).
Next, GitLab specific code was removed, and a migration to GitHub was added.

In its present form, it is used for the migration of SageMath from
https://trac.sagemath.org/ to GitHub. This migration is described in more detail in
https://trac.sagemath.org/ticket/30363

Why
===

[docs/Github-vs-Gitlab-vs-trac.md](docs/Github-vs-Gitlab-vs-trac.md) compares
[Github](https://github.com/) and [Trac](https://trac.sagemath.org/),
focusing on the specific differences that are important to the SageMath
community.

How
====

Migrating a Trac project to GitHub is a relatively complex process involving four steps:

 * Create a new project
 * Migrate the repository
 * Migrate issues and milestones
 * Migrate wiki pages

The script [migrate.py](./migrate.py) takes care of the third and fourth bullet points.

Usage:

  1. Symlink or copy [migrate.cfg.sagetracmigrationarchive](./migrate.cfg.sagetracmigrationarchive) to ```migrate.cfg```
  2. Configure the values
  3. Run (```./migrate.py```).

See [docs/Migration-Trac-to-Github.md](docs/Migration-Trac-to-Github.md) for details of the migration process
and a proposed workflow on GitHub (with transition guide from Trac for developers).

Features
--------

 * Creates a [migration archive](https://github.github.com/enterprise-migrations/#/./2.1-export-archive-format)
   in a subdirectory ``archive/``, containing records for issues (converted from Trac tickets).
 * Creates a markdown file for each converted ticket for easy inspection of the generated migration archive
   in subdirectories of ``wiki/`` like [Issues-33xxx](https://github.com/sagemath/trac_to_gh/tree/main/Issues-33xxx).
 * Ticket title, description, comments, attachments are copied over.
 * Component, issue type, priority, severity, resolution are converted to labels.
 * Selected keywords and milestones can be converted to labels.
 * CC is added to the issue description as "@" mentions.
 * Attribute changes are converted to issue events or issue comments.
 * Creates a file ``minimized_issue_comments.json`` that lists the IDs of issue comments that
   correspond to attribute changes.
 * Links to the cgit server are rewritten as GitHub repository links.
 * Links to Trac tickets and ticket comments are rewritten as GitHub issue links.
 * Links to the Trac wiki are rewritten as GitHub wiki links.
 * Wiki pages including attachments are exported into files in ``wiki/`` that can be
   added to the GitHub project wiki repository.

Missing
-------

 * History on wiki pages is not kept.
 * Edit history of ticket comments is not kept.

Other modes of operation of the script (not used in the SageMath migration)
---------------------------------------------------------------------------

Instead of creating a migration archive, the script can directly add issues to a GitHub project.
See [migrate.cfg.example](./migrate.cfg.example) for a sample configuration for this mode of operation.

 * It needs either a GitHub access token or a username/password pair.
 * All issues and issue comments are attributed to this username, and timestamps are not preserved.
 * Original usernames and timestamps are noted as part of the Issue descriptions and comments.
 * Make sure you test it on a test project prior; if you run the script twice against the same project,
   you will get duplicated issues.
 * Issue text attachments are uploaded as Gist (GitHub doesn't allow to attach files to issues via the GitHub API)
   or all issue attachments are exported to local files


License
=======

LGPL license version 3.0.

Requirements
==============

 * Python 3; various packages, see ```requirements.txt```
 * Trac server on which [XML-RPC plugin](http://trac-hacks.org/wiki/XmlRpcPlugin) enabled
