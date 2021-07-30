# NexusLIMS Jupyter Environment -- minimal-notebook

## Base image
[jupyter/minimal-notebook](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-minimal-notebook)

## Additional packages
- `gsutil`
- `google-cloud-storage`


## Use `gsutil` for cloud object storage
- [Authenticate](https://cloud.google.com/storage/docs/gsutil_install#creds-gsutil)
    ```bash
    gsutil config
    # copy and paste the URL into a browser window
    # click Allow Access button
    # copy the authorization code into the gsutil prompt
    # enter project ID into the gsutil prompt
    ```
- [List](https://cloud.google.com/storage/docs/gsutil/commands/ls)
    ```bash
    gsutil ls gs://<BUCKET_NAME>
    ```
- [Copy](https://cloud.google.com/storage/docs/gsutil/commands/cp)
    ```bash
    gsutil cp gs://<BUCKET_NAME>/<OBJECT_NAME> . # download object from cloud storage to local directory
    gsutil cp *.txt gs://<BUCKET_NAME> # upload all text files from local directory to a bucket
    ```
- [Help](https://cloud.google.com/storage/docs/gsutil/commands/help)
    ```bash
    gsutil help
    ```
- Documentation: https://cloud.google.com/storage/docs/gsutil
