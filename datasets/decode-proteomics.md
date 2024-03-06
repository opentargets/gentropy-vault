# deCODE proteomics

Mirror started on 2024-03-05 as `gs://gentropy-vault/decode-proteomics`.

## 1. Obtain download links
1. Go to the dataset hosted on deCODE: https://www.decode.com/summarydata/.
2. Navigate down to the paper by Ferkingstad, E. _et al._ titled _Large-scale integration of the plasma proteome with genetics and disease._
3. Click on the “Summary Data” link.
4. In the form which opens, accept the details, enter your details, pass CAPTCHA, and click on “Send”.
5. Follow the link you receive in your email.
6. Click on the top checkbox to select all files. The text should now read: “Total Files selected: 24271, Total Size: 24 TB”.
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
mkdir -p decode-proteomics/summary-data
cd decode-proteomics/summary-data
```

## 4. Prepare the download links
> [!NOTE]  
> The original list of files from deCODE contains an error: the download for the `proteomics` dataset contains all files for it (all good), _plus_ additional files from the `proteomics2023` dataset but with broken links. So in order to have a successful download, we need to only keep the files from the original `proteomics` dataset with working links.
```bash
# Remove broken links.
head -n 9816 ~/urls.txt > ~/urls.working.txt
# Add filenames.
paste ~/urls.working.txt <(cut -d= -f3 ~/urls.working.txt) > ~/urls-with-filenames.txt
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
    --joblog ~/decode-proteomics.log \
    --retries 100 \
    --colsep "\t" \
    fetch {1} {2} \
    :::: \
    ~/urls-with-filenames.txt
```

## 6. Download extra files
Download the three extra files named “Extra annotation”, “Excluded variants”, and “Read me file” and then run the command:
```bash
gsutil cp \
    proteomics_readme.txt \
    assocvariants.excluded.txt.gz \
    assocvariants.annotated.txt.gz \
    gs://gentropy-vault/decode-proteomics/additional-files/
```
