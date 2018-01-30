---
title: Creating Documents Using pandoc
category: ['Linux', 'Centos', 'Documents']
tags: ['centos', 'linux', 'win', 'documents']
---

Pandoc is an opensource utility to create documents from markdown. We can create PDF, Doc, doc, html and other formats. And can be also used to convert html to doc, html to pdf, markdown to pdf and many more.

If you need to convert files from one markup format into another, pandoc is your swiss-army knife. Pandoc can convert documents in markdown, reStructuredText, textile, HTML, DocBook, LaTeX, MediaWiki markup, TWiki markup, OPML, Emacs Org-Mode, Txt2Tags, Microsoft Word docx, LibreOffice ODT, EPUB, or Haddock markup to

- HTML formats: XHTML, HTML5, and HTML slide shows using Slidy, reveal.js, Slideous, S5, or DZSlides.
- Word processor formats: Microsoft Word docx, OpenOffice/LibreOffice ODT, OpenDocument XML
- Ebooks: EPUB version 2 or 3, FictionBook2
- Documentation formats: DocBook, TEI Simple, GNU TexInfo, Groff man pages, Haddock markup
- Page layout formats: InDesign ICML
- Outline formats: OPML
- TeX formats: LaTeX, ConTeXt, LaTeX Beamer slides
- PDF via LaTeX
- Lightweight markup formats: Markdown (including CommonMark), reStructuredText, AsciiDoc, MediaWiki markup, DokuWiki markup, Emacs Org-Mode, Textile
- Custom formats: custom writers can be written in lua.

Pandoc understands a number of useful markdown syntax extensions, including document metadata (title, author, date); footnotes; tables; definition lists; superscript and subscript; strikeout; enhanced ordered lists (start number and numbering style are significant); running example lists; delimited code blocks with syntax highlighting; smart quotes, dashes, and ellipses; markdown inside HTML blocks; and inline LaTeX. If strict markdown compatibility is desired, all of these extensions can be turned off.


Intro from [http://pandoc.org/](http://pandoc.org/)

### Installing `pandoc` on Centos / Ubuntu

On Centos/RHEL

	[ahmed@mylaptop ~]# yum install pandoc
	[ahmed@mylaptop ~]# sudo yum install texlive
					  
					  
On Ubuntu             
					  
	[ahmed@mylaptop ~]# apt-get install pandoc
	[ahmed@mylaptop ~]# apt-get install texlive 	

### Converting markdown to pdf format

Create a sample markdown file.

	[ahmed@mylaptop ~]$ mkdir markdown pdf
	[ahmed@mylaptop ~]$ cat markdown/2016-08-15-firstmarkdownfile.md

	# How To Configure Swappiness
	
	Swappiness is a Linux kernel parameter that controls the relative weight given to swapping out runtime memory, as opposed to dropping pages from the system page cache. Swappiness can be set to values between 0 and 100 inclusive. A low value causes the kernel to avoid swapping, a higher value causes the kernel to try to use swap space. The default value is 60, and for most desktop systems, setting it to 100 may affect the overall performance, whereas setting it lower (even 0) may decrease response latency.
	
	    Value	                Strategy
	    vm.swappiness = 0	    The kernel will swap only to avoid an out of memory condition. 
	                                See the "VM Sysctl documentation".
	    vm.swappiness = 1	    Kernel version 3.5 and over, as well as kernel version 2.6.32-303 
	                                and over: Minimum amount of swapping without disabling it entirely.
	    vm.swappiness = 10	    This value is sometimes recommended to improve performance 
	                                when sufficient memory exists in a system.
	    vm.swappiness = 60	    The default value.
	    vm.swappiness = 100     The kernel will swap aggressively.
	
	With kernel version `3.5` and over, as well as kernel version `2.6.32-303` and over, it is likely better to use `1` for cases where `0` used to be optimal.
	To temporarily set the swappiness in Linux, write the desired value (e.g. 10) to `/proc/sys/vm/swappiness` using the following command, running as root user.   
	
	    #  Set the swappiness value as root
	    echo 10 > /proc/sys/vm/swappiness
	
	    #  Alternatively, run this 
	    sysctl -w vm.swappiness=10
	
	    #  Verify the change
	    cat /proc/sys/vm/swappiness
	    10
	
	    #  Alternatively, verify the change
	    sysctl vm.swappiness
	    vm.swappiness = 10
	    
	To find the current swappiness settings, type:
	
		cat /proc/sys/vm/swappiness
		60
	
	Swapiness can be a value from 0 to 100. 
	
	1. Swappiness near 100 means that the operating system will swap often and usually, too soon. 
	2. Although swap provides extra resources, RAM is much faster than swap space. Any time something is moved from RAM to swap, it slows down.
	
	A swappiness value of 0 means that the operating will only rely on swap when it absolutely needs to. We can adjust the swappiness with the sysctl command:
	
		sysctl vm.swappiness=10
		vm.swappiness=10
	
	If we check the system swappiness again, we can confirm that the setting was applied:
	
		cat /proc/sys/vm/swappiness
		10
	
	To make changes permanent, you can add the setting to the /etc/sysctl.conf file:
	
		sudo nano /etc/sysctl.conf
	
	Add the below line.
	
	#  Search for the vm.swappiness setting.  Uncomment and change it as necessary.
	vm.swappiness=10

This is our sample markdown file above. Now we will convert it to PDF using the below script.

~~~ ruby
[ahmed@mylaptop ~]$ cd pdf
[ahmed@mylaptop ~]$ cat create_pdf_from_md.sh
#!/bin/bash

for markdown_file in `ls ../markdown/201*`; 
do
    input_file=$markdown_file 
    output_file=`echo $markdown_file | cut -d'/' -f3 | cut -d'.' -f1`.pdf

    #echo $input_file
    #echo $output_file

    if [ -f $output_file ];
    then
        echo "File '$output_file' Already Converted to PDF."
    else
        pandoc -f markdown -r markdown -t pdf -w pdf $input_file -t latex -o $output_file -V geometry:"left=2.0cm, right=2.0cm,top=1.0cm, bottom=1.0cm" --latex-engine=xelatex
    fi
done;
~~~

Now lets execute the script.

	[ahmed@mylaptop ~]$ sh create_pdf_from_md.sh
	
We are done. Here is how the file looks like.

![pdf generate demo](https://zubayr.github.io/images/pdf_gen_demo.PNG)

### Converting html to doc format

Creating a document file using `pandoc`

	pandoc -f html -t docx -o chef-server-setup.docx http://zubayr.github.io/chef-server-setup/

This will create a `doc` file from `html` link.
