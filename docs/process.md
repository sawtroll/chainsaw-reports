# Description of the process of extracting and managing report files

## Fetching raw HTML with report links

- From site https://kwf-services.de/fpa-gebrauchswertzeichen/
  - Iframe: https://kwf2018.kwf-online.de/serviceszertifikatlist/index_standalone.html
- Perform search
  - Objektgruppe: Elektro-Kettensägen, Motorsägen
  - Zeige maximal: 300
  - (check) Auch archivierte Prüfungen anzeigen
- Click Inspect
- Select element (table), right click: Edit as HTML
- Select all
- Copy
- Paste in text editor and save to table.html

## Extracting unique file list

```bash
cat table.html|grep -Eo 'data[^"]+?pdf'|sort -u > files.txt`
data/resources/aagw/elektrokettensaege/9194_20e.pdf
data/resources/aagw/elektrokettensaege/9195_20.pdf
data/resources/aagw/elektrokettensaege/9195_20e.pdf
data/resources/aagw/motorsaegen/10382_22.pdf
data/resources/aagw/motorsaegen/10382_22e.pdf
data/resources/aagw/motorsaegen/10383_23.pdf
data/resources/aagw/motorsaegen/10383_23e.pdf
data/resources/aagw/motorsaegen/3758_19.pdf
...
```

## Downloading files

`download.sh`
```bash
#!/bin/bash
if [ ${#} != 3 ]; then
        echo "Download files specified in text file <file-url-list> prepending <prepend-url> and downloading files to <destination-directory>"
        echo "Usage: $0 <file-url-list> <prepend-url> <destination-directory>"
        exit 1
fi

destDir="$3"
prepend="$2"
listFile="$1"

mkdir -p "${destdir}"

for file in $(cat "${1}"); do
    url="${prepend}${file}"
    name=${url##*/}
    destFile="${destDir}/${name}"
    echo "Downloading $name to $destFile"
    cmd="curl --progress-bar --create-dirs -o $destFile $url"
    # echo "executing ${cmd}"
    ${cmd}
done
```

```bash
Download:
./download.sh files.txt https://kwf2018.kwf-online.de/serviceszertifikatlist/ ./reports

Downloading 10251_22.pdf to ./files-2023-11-12/10251_22.pdf
################################################################################################################## 100.0%
Downloading 10251_22e.pdf to ./files-2023-11-12/10251_22e.pdf
################################################################################################################## 100.0%
Downloading 10252_22.pdf to ./files-2023-11-12/10252_22.pdf
...
```

## Massage PDF files to match brand and model

* Remove anu usage restrictions (ability to Copy content)  
  `qpdf *.pdf --decrypt --replace-input`
* Check for duplicates against current list (use utility)
* Rename files to match brand and model (plus `-en` suffix for English language versions)
* Add new files to repository file list in [reports](./reports/)
* List the files
  `ls -1p | egrep -v /$` 
* Copy file name listing to spreadsheet [KWF Test Report Markdown generation](https://docs.google.com/spreadsheets/d/1m3kVIHacdpucfIuVUekzyl80FkJsdFMETi23rCuUpgE/edit#gid=331780796)
* Copy Markdown table content from sheet `MarkdownExport` and paste in [README.md](../README.md)
* Commit changes and wait for Github Pages to be built and deployed to https://sawtroll.github.io/chainsaw-reports/
 
