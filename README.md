Overview
========
The Wadl normalizer (normalizeWadl.sh and the associated xslts) is a
tool that takes as input a wadl file that is spread out into several
files and has been written with methods split out and reused
(e.g. <method href="#getVersionInfo"/>) and produces a version of the
wadl flattened into a single file and with all of the references
expanded. As part of this process, the tool flattens any xsds included
in the wadl (or in any wadls pulled in to the source wadl) and any
xsds imported into any of those xsds. These flattened xsds are written
out as xsd-1.xsd, xsd-2.xsd, etc. All references to the xsds are
updated.

After producing the flattened wadl and xsds, the tool validates them
against the appropriate w3c schema using xmllint.

The flattened version of the wadl is easier to process, especially if
the source wadl takes advantage of authoring convenience features such
as resource_types. Even tools with bugs and limitations in their
ability to handle wadls like SoapUI can consume a normalized wadl.

In addition to flattening the wadl out, the normalizer also simplifies
other constructs that are known to cause problems for tools like
SoapUI. For example, give a wadl:param like the following where
auth:UserType is an enumeration with three possible values:

    <param name="type"      
          style="query"
          required="false" 
          type="auth:UserType"
          default="CLOUD" />

The normalizer converts the wadl:param to a string:

    <param name="type" 
	  style="query" required="false" default="CLOUD"
          type="xsd:string">
          <option value="CLOUD"/>
          <option value="NAST"/>
          <option value="MOSSO"/>
    </param>

Dependencies
------------
The wadl normalizer is a bash script and requires that xmllint be in
your path. Windows users should use the script with Cygwin. 

These scripts require an xslt 2.0 processor such as Saxon. Saxon
9.1.0.8, released under the Mozilla Public License is provided bundled
with the scripts as a convenience.

Limitations
-----------
Wadls with matrix parameters are not supported if you use "path"
format.

Installation
------------
Clone the git repository a convenient location on your system and
create a symlink to bin/normalizeWadl.sh in a directory that is in
your path (e.g. /usr/bin). 

If you use oXygen XML Editor, create a symlink to the wadl-tools
directory in your oXygen frameworks directory. When placed in the
frameworks directory, wadl-tools adds functionality to oXygen so that
it validates as you type against both the wadl xsd schema and a set of
schematron rules. In addition, wadl-tools adds a transformation
scenario to oXygen to normalize wadls. Note that oXygen is a
commercial tool available from http://www.oxygenxml.com/

Note that the transformation scenario in oXygen currently does not
validate the wadl or xsds as the shell script does.

Usage
-----
Invoke the script as follows:

normalizeWadl.sh -w /path/to/file.wadl

The script creates a normalized version of the wadl in
/path/to/file-normalized.wadl. In addition to the wadl file name, you
can pass in other parameters:

    Usage: normalizeWadl.sh [-?fvx] -w wadlFile

    OPTIONS:
       -w wadlFile: The wadl file to normalize.
       -f Wadl format. path or tree
          path: Format resources in path format, 
                e.g. <resource path='foo/bar'/>
          tree: Format resources in tree format, 
                e.g. <resource path='foo'><resource path='bar'>...
          If you omit the -f switch, the script makes no 
          changes to the structure of the resources.
       -v XSD Version (1.0 and 1.1 supported, 1.1 is the default)
       -x true or false. Flatten xsds (true by default).
       -r keep or omit. Omit resource_type elements (keep by default).

The -f argument controls the format of the path attributes on the
resource elements:

 * Omitting the second argument leaves the structure of the resource
   elements unchanged.

 * Using "path" as the second argument causes the normalizer to
   flatten out the path attributes on resources. So:

    <resource path="nast" id="nastUsers">
            <resource path="{nastId}" id="nastId">

   becomes:

    <resource path="nast/{nastId}" id="nastUsers">

 * Using "tree" as the second arguments causes the normalizer to
   expand any flat path attributes on resource elements. So:

    <resource path="nast/{nastId}" id="nastUsers">

   becomes

    <resource path="nast" id="nastUsers">
       <resource path="{nastId}" id="nastId-d2e381">

For example, if you are in the normalizeWadl directory, running:

    ./bin/normalizeWadl.sh -f path -w samples/resources/admin.wadl

Creates the file samples/resources/normalized/admin.wadl which is a
normalized version of the input file with the path attributes
flattened out.

Typing normalizeWadl.sh -? prints out a brief usage summary. 

References
==========
* Wadl Specification: http://www.w3.org/Submission/wadl
* Cygwin: http://www.cygwin.com/
* Oxygen XML Editor: http://www.oxygenxml.com/
* SoapUI: http://www.soapui.org/

License
=======
These scripts are released under the Apache 2 license. See LICENSE for
more information.

Saxon 9.1.0.8, released under the Mozilla Public License is provided
with the scripts as a convenience.