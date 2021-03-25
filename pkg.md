# Module pkg


## func pkg_clean

Clean out the local package manager database cache and do anything else to the package manager to try to clean up any
bad states it might be in.

## func pkg_gentoo_canonicalize

Takes as input a package name that may or may not have a category identifier on it.  If it does not have a category
(e.g. app-misc or dev-util), then find the category that contains the specified package.

NOTE: If the results would be ambiguous, fails and indicates that a category is required.

```Groff
ARGUMENTS

   name
         Package name whose category you'd like to find.

```

## func pkg_install

Install some set of packages whose names are specified.  Note that while this function supports several different
package managers, packages may have different names on different systems.

```Groff
ARGUMENTS

   names
         Names of package to install.
```

## func pkg_installed

Returns success if the specified package has been installed on this machine and false if it has not.

```Groff
ARGUMENTS

   name
         Name of package to look for.

```

## func pkg_known

Determine if the package management system locally knows of a package with the specified name.  This won't update the
package database to do its check.  Note that this does _not_ mean the package is installed.  Just that the package
system believes it could install it.

See pkg_installed to check if a package is actually installed.

```Groff
ARGUMENTS

   name
         Name of package to look for.

```

## func pkg_sync

Sync the local package manager database with whatever remote repositories are known so that all packages known to those
repositories are also known locally.

## func pkg_uninstall

Use local package manager to remove any number of specified packages without prompting to ask any questions.

```Groff
ARGUMENTS

   names
         Names of package to install.
```

## func pkg_upgrade

Replace the existing version of the specified package with the newest available package by that name.

```Groff
ARGUMENTS

   name
         Name of the package that should be upgraded to the newest possible version.

```
