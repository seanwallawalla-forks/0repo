0repo
=====

Copyright Thomas Leonard, 2013


Introduction
------------

The 0repo software provides an easy and reliable way to maintain a repository
of 0install software for others to use. It can be run interactively by a single
developer on their laptop to maintain a repository of their own programs, or
installed as a service to allow a group of people to manage a set of programs
together.

Developers place new software releases in an "incoming" directory. 0repo
performs various checks on the new release and, if it's OK, adds it to the
repository. 0repo signs the published feeds with its GPG key.

The generated files may be rsync'd to a plain web host, without the need for
special software on the hosting platform.

Features:

- Can be run automatically as part of a scripted release process (for
  single-developer use).

- Can run as a service to accept contributions from multiple developers (not
  yet implemented).

- Keeps feeds under version control.

- Repositories are always consistent (no missing keys, missing stylesheets,
  invalid URIs, etc).

- Files can be hosted on a standard web host (e.g. Apache).

- Provides a catalogue file listing all published feeds, which can be polled
  automatically by mirror sites (e.g. 0mirror).

- Supports both archives hosted within the repository and archives hosted
  externally.


Setup
-----

If you're setting up a repository for a single developer then you can use use
your existing personal GPG key (if you have one). Otherwise, you should create
a new one:

    $ gpg --gen-key

You can accept the defaults offered. Make sure you specify an email address,
because 0repo uses that as the committer in Git log messages, which require
an email address.

Then run "0repo create DIR KEY" to create the new repository (directory DIR
will be created to hold the files and will be populated with an initial
configuration).

    $ 0repo create ~/repo 'John Smith'
    $ cd ~/repo

Within this directory you will find:

- 0repo-config.py - configuration settings
- feeds	          - directory of (unsigned) feeds, initially empty
- feeds/.git	  - version control Git repository for the feeds
- incoming	  - queue of incoming files to be processed
- public	  - output directory (to be rsync'd to hosting provider)

Edit 0repo-config.py and set the required parameters:

These are required:

- `REPOSITORY_BASE_URL`: The base URL for the feeds
- `ARCHIVES_BASE_URL`: The base URL for the archives

These are optional:

- Command to upload feeds to web hosting (not yet implemented)
- Command to upload archives to archive hosting
- GPG keys of trusted contributors (not yet implemented)

Finally, register this repository so that other tools can find it (you need to do this after setting `REPOSITORY_BASE_URL`):

    $ 0repo register
    Created new entry in /home/me/.config/0install.net/0repo/repositories.json:
    http://example.com/: {"path": "/home/me/repositories/myrepo", "type": "local"}


Importing pre-existing feeds
----------------------------

If you've been managing a set of feeds without 0repo, you can import them into it using the `add` command:

    cd .../my-old-repository
    0repo add *.xml

The feeds will be added to the `feeds` directory, with any signatures removed (the signature will be stored in the Git commit message). 0repo looks at the `uri` attribute in the XML to decide which of the registered repositories to use.


Adding a release
----------------

To add a release, create a local XML file containing just the new version, with a `<feed-for>` giving the target feed. For example, you could do this using [0template](http://0install.net/0template.html):

    $ 0template someprog.xml.template version=1.2
    Writing someprog-1.2.xml

Then, ask 0repo to add the new XML to the repository:

    $ 0repo add someprog-1.2.xml

0repo will use the `<feed-for>` to select the correct repository and will add
it there. If the feed doesn't already exist in the repository, 0repo will
create a new one for it.


Archives
--------

If the archives are to be stored outside of the repository (e.g. an existing
3rd-party release), you can just include the full URL in the XML file.

On the other hand, if you wish to store the release archives in the repository,
use a relative href on the `<archive>` element and place the archive in the
same directory as your XML. e.g.

    <archive extract="someprog-1.2" href="someprog-1.2.tar.gz" size="87942"/>

0repo will upload the archive to the repository's file hosting (using the command
configured in `0repo-config.py`) and insert the full URL into the generated feed.


The generated files
-------------------

After importing feeds or adding new versions, 0repo will generate a set of signed
feeds in the `public` directory, along with a `catalog.xml` file listing all the
programs in the repository, the repository's public GPG key and various stylesheets.

When 0repo generates the signed feeds it will also:

- check that each feed's URI is correct for its location
- add the stylesheet declaration
- for each relative <archive>'s href, check that the archive is known
  and make the URL absolute

The `public` directory can then be transferred to the hosting provider (e.g.
using rsync).


Editing feeds
-------------

You can edit the unsigned feeds under repo/feeds whenever you want. Running
0repo again will regenerate the signed feeds in repo/public (if the source feed
has changed). You should commit your changes with `git commit`.

To remove a feed, `git rm repo/feeds/FEED.xml` and run `0repo` again.


Running a shared repository
---------------------------

This is not yet implemented.

For a shared repository: the release tool generates the archives and
the XML for the new version, signs the XML with the developer's key,
and uploads to a queue (could be e.g. FTP). 0repo downloads the
contents of the queue to its incoming directory, checks the signature
and merges the new XML into the feed. If there's a problem, it emails
the user.

For other edits (e.g. adding a `<package-implementation>` or adding a missing
dependency to an already-released version), the contributor sends a Git pull
request. The repository owner merges the pull request and runs 0repo.


Auditing
--------

When 0repo adds a new release, the Git commit message includes the XML,
including the signature, if any. This makes it possible to tell whether a
malicious update was caused by a compromised 0repo (commit is invalid) or by a
compromised contributor (the malicious XML is correctly signed by that
contributor).

Commits made by 0repo are signed with its GPG key. You can check these
signatures using `git log --show-signature` in the `feeds` directory.


Conditions
----------

This library is free software; you can redistribute it and/or
modify it under the terms of the GNU Lesser General Public
License as published by the Free Software Foundation; either
version 2.1 of the License, or (at your option) any later version.

This library is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public
License along with this library; if not, write to the Free Software
Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307  USA

The feed.xsl and feed.css stylesheets have their own license; see
the file headers for details.


Bug Reports
-----------

Please report any bugs to [the 0install mailing list](http://0install.net/support.html).
