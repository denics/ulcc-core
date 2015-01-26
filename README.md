# ULCC deployment of EPrints 3.3 branch #

ULCC EPrints deployment is based on https://github.com/eprints/eprints/tree/3.3 (which is currently the active development branch).

Contributors *must* use the following branching model: http://nvie.com/posts/a-successful-git-branching-model/

## Getting Started - Developers

Assuming your want to work in /opt/eprints3:

````
cd /opt
mkdir eprints3
chown eprints:eprints eprints3
chmod 02775 eprints3
cd eprints3
su eprints
git clone https://github.com/eprintsug/ulcc-core.git .
git checkout develop # the development branch
cp perl_lib/EPrints/SystemSettings.pm.ulcc perl_lib/EPrints/SystemSettings.pm
````

Edit SystemSettings.pm to suit - usually this just means adding the appropriate user and group.

````
git submodule init
git submodule update
````

To create a repository, first fork https://github.com/eprintsug/ulcc-skel, then:

````
cd archives
git clone https://github.com/your/fork
````

The repository will only contain a minimal set of files which you can edit to suit:

````
 tree archives/foo
 archives/foo/
 ├── cfg
 │   ├── cfg.d
 │   │   ├── 10_core.pl # hostname of repository
 │   │   ├── adminemail.pl
 │   │   └── database.pl # db connection details (use 'bin/epadmin config_db foo' to change if preferred)
 │   └── lang
 │       └── en
 │           └── phrases
 │               └── archive_name.xml # repository name
 ├── documents
 ├── html
 └── var
````

Get everything up and running:

````
cd /opt/eprints3
mkdir archives/foo/documents/disk0
bin/epadmin create_db foo
bin/import_subjects foo lib/defaultcfg/subjects
bin/generate_apacheconf --replace --system # follow instructions for adding EPrints to global apache conf
bin/epadmin create_user foo
testdata/bin/import_test_data foo archive username
````

### Enabling plugins ###

In most cases, plugins can be enabled with 2 steps, for example:

````
tools/epm/link_lib bootstrap
tools/epm enable foo bootstrap
````

Some plugins require additional steps - see below.

**RIOXX2 and Recollect**

Both of these expect to find a workflow file archives/foo/cfg/workflows/eprint/default.xml - the ulcc-skel repository does not provide this file so do the following before enabling:

````
mkdir -p archives/blank/cfg/workflows/eprint/
cp lib/defaultcfg/workflows/eprint/default.xml archives/blank/cfg/workflows/eprint/
````

**MePrints**

MePrints expects to find 2 workflow files: archives/foo/cfg/workflows/eprint/default.xml and archives/foo/cfg/workflows/user/default.xml - the ulcc-skel repository does not provide these files so do the following before enabling:

````
mkdir -p archives/blank/cfg/workflows/eprint/
mkdir -p archives/blank/cfg/workflows/user/
cp lib/defaultcfg/workflows/eprint/default.xml archives/blank/cfg/workflows/eprint/
cp lib/defaultcfg/workflows/user/default.xml archives/blank/cfg/workflows/user/
````

### Adding new plugins ###

```
git checkout develop # don't modify the master branch
git submodule add http://github.com/eprintsug/foo lib/epm/foo
git status # should show changes to .gitmodules and lib/epm/foo
git commit -am "Added foo 1.0.0"
git push
```

## Merging upstream changes ##

ulcc-core is a fork of https://github.com/eprints/eprints/tree/3.3 so upstream changes can be merged as follows:

```
git clone git@github.com:eprintsug/ulcc-core .
git checkout develop # don't modify the master branch
git submodule init
git submodule update
git remote add upstream https://github.com/eprints/eprints.git
git fetch upstream 3.3
git merge upstream/3.3
# may need to fix conflicts and commit
git push
```

## Initial setup ##

````
git init
git remote add origin git@github.com:eprintsug/ulcc-core.git
git remote add upstream https://github.com/eprints/eprints.git
git remote -v # sanity check
git pull upstream 3.3
git push origin master
git branch --set-upstream-to=origin/master master
````
