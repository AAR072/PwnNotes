# Tricks
- Verify that image size according to exif is the same as true size
## Corrupted images
- Splice the header from another file
	 - The JFIF (JPEG File Interchange Format) header in hexadecimal is `FF D8 FF E0 00 10 4A 46 49 46 00 01` or `FF D8 FF E0 xx xx 4A 46 49 46 00`
	- Just splice - Copy from the beginning (`FFD8`) 
	- Continue **until just before the first Start of Frame (SOF)** marker (usually `FFC0`, `FFC2` etc.)

That’s typically **a few hundred bytes** for standard EXIF/JFIF files.
# Websites
- https://fotoforensics.com/
	- Basic Analyis