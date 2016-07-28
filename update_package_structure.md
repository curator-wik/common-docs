# Update Package Structure
_Working Draft of an in-progress feature. Subject to change._

Update packages are specific directory structures of files typically distributed in Phar 
archives. This document describes the directories and files that comprise a valid update 
package.

A single update package may contain sufficient patches to update installations that are 
one or several releases old. The number of previous releases a given package supports is
up to its creator.

Besides being a directory tree archive format, Phar supports storage of user-defined 
archive- and file-level metadata. Many opportunities to use phar metadata storage 
(example: name of the application it applies to) are intentionally avoided to improve 
simplicity of archive creation and examination with standard tools. Consequently, update 
packages include a number of small files that capture such information.

## Update package root
The following is a description of files and directories at the package root.

* `application` - file, required  
  The name of the application this update package applies to, e.g "backdrop" or "drupal"
* `component` - file, optional  
  When the update package contains an update for some sub-component of the application, 
  this file may be used to contain the name of that component. e.g, a Drupal module or
  theme. This information is used by application targeting code to locate the destination
  path this update will be applied to. Should be included even when it is the "core" of a
  multi-component application that is being updated. Only exclude this if the application
  is monolithic.
* `package-version` - file, required  
  Specifies the upgrade package format version. May be left empty or may contain "1.0".
* `version` - file, required  
  The version of the application or component that will result after applying this update
  package. Any string is valid as a version identifier as long as it uniquely identifies
  the release.
* `prev-versions-inorder` - file, required  
  This file lists all previous versions of the application or component that this update
  package supports updating from. The versions must be listed in the order they were 
  released, from earliest to latest, one per line. Order is important because the 
  process of applying an entire update package internally involves the application of 
  a succession of patches, and unpacker must know which versions to apply patches from in
  which order.  
  Even if the package only supports updating from a single former release,
  that former release must be listed so that application targeting code can verify that
  the destination can be updated by this package.  
  It is illegal to list the version from the `version` file, as the package cannot
  support an upgrade _from_ that version.
* `files` - directory, optional  
  A collection of complete files and directories to apply verbatim to the destination.
  Existing files are overwritten and new files are created. To cause files to be deleted,
  use `patches`.
* `patches` - directory, optional  
  See patches section below. Do not include patches to files if they are included in the
  `files` directory.
  
An update package that lacks both the `files` and `patches`
directories makes no sense.
### The patches Directory
The patches directory contains (only) subdirectories whose names match versions from
the `version` or `prev-versions-inorder` file. Contents of these directories contain the 
patches that make the destination code version match that of the directory name.
The earliest version listed in `prev-versions-inorder` is therefore illegal as a patches 
subdirectory.

Within each of these subdirectories is the following structure:
* `deleted_files` - file, optional  
  When files were deleted in this release, this file contains the names of those files
  as paths relative to the component root, with '/' as directory separator, one per
  line.  
* For any file `$f` that has been changed in the original source, two files are created in the
  update package (directory structure is mirrored as necessary):
  * `$f.patch` is the patch to apply, in [google-diff-match-patch unidiff-like format.](https://code.google.com/p/google-diff-match-patch/wiki/Unidiff)
  * `$f.meta` is a json encoding of the following metadata:
    * `initial-md5`: base64-encoded md5 hash of the pre-patched file from the former
      release.
    * `resulting-md5`: base64-encoded md5 hash of the file in the official release;
      after applying `$f.patch`, should match the hash of `$f`. 
  
  Hashes in the metadata are useful to (1) determine if the file at the destination has
  been altered from the official release; and (2) to verify the patch routine's
  correctness.
