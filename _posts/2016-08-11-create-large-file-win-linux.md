---
toc: true 
toc_label: 'Contents' 
toc_icon: 'cog'
title: Creating a large file in Windows / Linux
category: ['Centos', 'Rhel', 'Windows']
tags: ['centos', 'windows']
---

Was working on a monitoring project, need to create a large file to test notifications.
Here is how we can do that. 

Create a file of 500MB in Windows
    
    fsutil file createnew <filename> <size_in_bytes>
    fsutil file createnew output.file.dat 53687091200

Create a file of 256M in Linux

    dd if=<input_file> of=<output_file>  bs=<bytes_at_a_time>  count=<blocks>
    dd if=/dev/zero of=output.file.dat  bs=256M  count=1

Create a file of 512MB in Linux   
    
    dd if=/dev/zero of=output.file.dat  bs=256M  count=2    
    
    