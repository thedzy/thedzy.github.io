---
layout: default
title: Create a script that will embed and extract a file
parent: repositories
nav_order: 1
has_toc: true
---
[View on my GitHub](https://github.com/thedzy/embed_file_in_a_script){: .btn .fs-5 .mb-10 .mr-5 .mb-md-0 }

# embed_file_in_a_script
Create a script that will embed and extract a file

*Note: The script can later be modified to perform actions against or with the extracted file*

```bash
Usage: embed_file_in_a_script.py [options]
Create a script that will embed a file.  The script can be added on to to handle the file.  Example: A image can be embedded and then we set as the wallpaper

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -o OUTFILE, --outfile=OUTFILE
                        OutFile: The file that will be created, when the new
                        script is run
  -i INFILE, --infile=INFILE
                        InFile: The file that will read the create the script
  -s SCRIPT, --script=SCRIPT
                        IScript: The script that will host the InFile
```
