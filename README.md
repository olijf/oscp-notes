# OSCP Repo by Week using the Offsec recommended 12 week plan
I do things in my own tempo, so I might be a bit slower than the recommended 12 weeks. I will be using this repo to track my progress and share my notes and resources.

Open the repo in Obsidian or any other markdown editor to view the notes and resources in a more organized way.

I used pdfcpu to process the PDF files and make them printable (they have a stupid background)
```bash
docker run --rm -it -v "$(pwd)":/app pdfcpu crop '0 100' 'PEN-200 12 week learning plan.pdf' 'PEN-200 12 week learning plan4.pdf'
# 
docker run --rm -it -v "$(pwd)":/app pdfcpu nup "form:A4P,enforce:on,bo:off,ma:0" 'PEN-200 12 week learning plan5.pdf' 2 'PEN-200 12 week learning plan4.pdf'
```



```bash
convert -density 300 PEN-200\ 12\ week\ learning\ plan5.pdf   -colorspace Gray   -normalize   -threshold 65%  -negate PEN-200\ 12\ week\ learning\ plan6.pdf
```

convert is a command-line tool from ImageMagick that can convert and process images and PDFs. In this case, the command is used to convert the PDF file "PEN-200 12 week learning plan5.pdf" into a grayscale image, normalize it, apply a threshold to make it more readable, and then negate the colors to create a new PDF file.
