# Update Package Structure
_Working Draft of a planned feature. Subject to change._

Update packages are specific directory structures of files distributed in Phar archives.
This document describes the directories and files that comprise a valid update 
package.

Update packages may contain sufficient patches to update code that is one or several
releases old, depending on how it is constructed.

Besides being a directory tree archive format, Phar supports storage of user-defined 
archive- and file-level metadata. Many opportunities to use phar metadata storage 
(example: name of the application it applies to) are intentionally avoided to improve 
simplicity of archive creation and examination with standard tools. Consequently, update 
packages include a number of small files that capture such information.

## Update package root
The following is a listing of files and directories at the package root.

* `application` - file, required  
  The name of the application this update package applies to, e.g "backdrop" or "drupal"
* `component` - file, optional  
  When the update package contains an update for some sub-component of the application, 
  this file may be used to contain the name of that component. e.g, a Drupal module or
  theme. This information is used by application targeting code to locate the path this
  update applies to. Should be included even when the "core" of a multi-component 
  application is being updated, and only excluded if the application is monolithic.
* `package-version` - file, required  
  Specifies the upgrade package format version. May be left empty or may contain "1.0".
* `version` - file, required  
  The version of the application or component that will result after applying this update
  package.
* `version-order` - file, required  
  This file lists all versions of the application or component that this update
  package supports bringing up-to-date. Because the process of applying the package 
  internally involves the application of patches, the unpacker must know in which order 
  to apply previous versions' patches. 
