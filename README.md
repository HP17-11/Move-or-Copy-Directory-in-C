# Move-or-Copy-Directory-in-C
---> dircpmvlist act as a commad to move or copy a directory from specified location to other folder or directory


dircpmvlist.c file contains the main code
description : it is C program "dircpmvlist" that copies or moves a directory tree rooted at a specific path to a specific destination folder along the file types specified in the extension list (or the entire directory if the extension list is not specified).

Synopsis :
dircpmvlist [ source_dir] [destination_dir] [options] <extension list>

--> Both source_dir and destination_dir can be either absolute or relative paths. 
--> If the destination_dir is not present in the home directory hierarchy, it will be newly created.
--> -cp copy the directory rooted at source_dir to destination_dir and it does not delete the directory (and contents) rooted at source_dir
--> -mv move the directory rooted at source_dir to destination_dir and deletes the directory (and contents) rooted at source_dir 
--> extension list: up to 6 file extensions can be provided ( c , pdf, txt etc.)
if extension list is provided, only the files with matching extensions will be moved/copied. If it is not provided, all the files will be moved or copied.
