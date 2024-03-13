# deCODE proteomics

Mirror started on 2024-03-13 as `gs://gentropy-vault/decode-proteomics2023`.

## 1. Obtain download links
1. Go to the dataset hosted on deCODE: https://www.decode.com/summarydata/.
2. Navigate down to the paper by Grímur Hjörleifsson Eldjarn, Egil Ferkingstad _et al._ titled “Large-scale plasma proteomics comparisons through genetics and disease associations”.
3. Click on the “Summary Data” link.
4. In the form which opens, accept the details, enter your details, pass CAPTCHA, and click on “Send”.
5. Follow the link you receive in your email.
6. Click on the top checkbox to select all files. The text should now read: “Total Files selected: 19361, Total Size: 20 TB”.
7. Click on the “Download files” button.
8. In the window which opens, scroll down to the bottom part and click on “Download text file with links”.

## 2. Transfer the downloaded file with URLs to the vault ingestion instance
```bash
gcloud compute scp urls.txt vault-ingest:~ \
    --project=open-targets-genetics-dev \
    --zone=europe-west1-d
```

At this point, switch to the vault ingestion machine.

## 3. Create the dataset directory
```bash
cd ~/vault
mkdir -p decode-proteomics2023
cd decode-proteomics2023
```

## 4. Prepare the download links and filenames
> [!NOTE] 
> The original files are all in one directory, which is not convenient. Here we separate them based on prefix (& correspondigly, content).

```bash
FOLDERS="GBR_UKB_Africa_OLINK GBR_UKB_Africa_OLINK2 GBR_UKB_OLINK GBR_UKB_OLINK2 GBR_UKB_SAsia_OLINK GBR_UKB_SAsia_OLINK2 Proteomics_PC0 Proteomics_SMP"
for F in $FOLDERS; do
    export F
    # Create the output directory.
    mkdir $F
    # Extract URLs only for this folder.
    grep "${F}_" ~/urls.txt > /tmp/$F.urls.txt
    # Generate output file names, including folders.
    cut -d'=' -f3 /tmp/$F.urls.txt | xargs -I{} echo $F/{} > /tmp/$F.filenames.txt
    # Prepare final file with URLS and filenames.
    paste /tmp/$F.urls.txt /tmp/$F.filenames.txt > /tmp/decode-proteomics2023.$F.final.txt
done
cat /tmp/decode-proteomics2023.*.final.txt > ~/urls-with-filenames.txt
```

## 5. Sync the data
> [!NOTE]  
> deCODE website suggests to use `aria2` for downloading the files. However, the server hosting the files appears to be incompatible with some of aria2 functionality, causing a lot of non-recoverable “Invalid range header” errors: https://github.com/aria2/aria2/issues/1344. Because of this, we use curl for mirroring.
```bash
# Define a function to fetch one file.
function fetch () {
    curl \
        --connect-timeout 600 \
        --max-time 3600 \
        --silent \
        "$1" \
        > "$2"
}
export -f fetch

# Download all files in parallel.
time parallel \
    --bar \
    --eta \
    --jobs 64 \
    --joblog ~/decode-proteomics2023.log \
    --retries 100 \
    --colsep "\t" \
    fetch {1} {2} \
    :::: \
    ~/urls-with-filenames.txt
```

## 6. Download extra files
Download the one extra file named “Read me file” and then run the command:
```bash
gsutil cp \
    proteomics2023README.txt \
    gs://gentropy-vault/decode-proteomics2023/
```
