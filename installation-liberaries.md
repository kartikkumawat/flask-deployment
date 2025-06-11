## Installation and liberaries problems on latest python3.13 
### weasyprint==65.1
Use gtk3 runtime 
[https://github.com/tschoonj/GTK-for-Windows-Runtime-Environment-Installer/releases](https://github.com/tschoonj/GTK-for-Windows-Runtime-Environment-Installer/releases)

## backports.zoneinfo==0.2.1
This is in-build in python3.13, so no need to install externally

## django-pdfkit==0.3.1 && pdfkit==1.0.0
* django-pdfkit==0.3.1 pdfkit==0.6.0

## pkg_resources==0.0.0
Skip this 

## Pillow==9.5.0
Not support on python3.13
Try this 
```bash
pip install pillow
```

## PyYAML==6.0
Try this 
```bash
pip install PyYAML
```
## mysqlclient==2.1.1
Try this 
```bash
pip install mysqlclient
```
