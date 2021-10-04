# NexusLIMS Jupyter Environment -- microscopy-notebook

## General Information
This computing environment is built from a [Docker image](https://github.com/phylsix/nexuslims-notebook/blob/main/microscopy-notebook/Dockerfile),
whose base image is [jupyter/minimal-notebook](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-minimal-notebook)
. On top of that, several scientific Python libraries are
pre-installed including:

- `h5py`
- `matplotlib`
- `numba`
- `numpy`
- `pandas`
- `pytables`
- `scikit-image`
- `scikit-learn`
- `scipy`
- `seaborn`
- `hyperspy`

. Additionally `gcloud` and `gsutil` CLI tools are installed
for GCP operations.

## Operations with GCP
## Setup Project ID and Authentication
### Setup Project ID
NexusLIMS Jupyter environment is running within a GCP project.
To be able to access the resources (storage, CPU, services etc.
) using `gcloud`, **Project ID** must be configured. You will
get it from your admin, and configure `gcloud` with
```bash
gcloud config set project <PROJECT_ID>
```
You only need to do this once as long as you stay with the
same project.

### Authentication
When you first log in the NexusLIMS Jupyter environment,
or has not been authenticated for a long time, you will need to
authenticate yourself with your credentials (you will be
reminded). To authenticate,
```bash
gcloud auth login
```
, then copy paste the URL prompted in the terminal to the
browser address bar, click "allow aceess" and copy paste
the access token back to the terminal to finish the
authentication.

To submit computing jobs to the Compute Engine, additional
authentication to get the Application Default Credential (ADC)
is required:
```bash
gcloud auth application-default login
```
Take similar steps as above to finish the authentication.

## Using `gsutil` to access data file in cloud storage
First, make sure to authenticate yourself following the
instruction above.

The two most common operations are `list` and `copy`:
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

## Using `snakemake` to submit computing jobs on Compute Engine
[`Snakemake`](https://snakemake.github.io/) is an open-sourced
workflow management framework, esp. for reproducible data
analysis. It is pre-installed in the NexusLIMS Jupyer
environment. In the below, an example is shown to submit
computing jobs on Compute Engine using `snakemake`.

Before starting, make sure you authenticate yourself to get
ADC following the instruction above. In addition, make sure you
have the following roles in the configured project:
* Compute Storage Admin(Can potentially be restricted further)
* Compute Viewer
* Service Account User
* Cloud Life Sciences Workflows Runner
* Service Usage Consumer

These are the minimal permission roles suggested from the
[snakemake doc](https://snakemake.readthedocs.io/en/stable/executor_tutorial/google_lifesciences.html#credentials).
Contact your admin for permission granting if you do not have
those roles.

In this example, we are configuring a job to
1. download a `.dm3` data file from cloud storage,
2. read the local file with `hyperspy`,
3. generate a png thumbnail image with `matplotlib` and
    upload to the cloud storage.

First, we make a folder named `test_submission`:
```bash
mkdir test_submission && cd test_submission
```

Next, we prepare three files:
1. `environment.yaml`
    ```yml
    channels:
    - conda-forge
    dependencies:
    - python
    - matplotlib
    - hyperspy
    ```
    This contains the dependent packages of the job.
2. `test.py`
    ```python
    import matplotlib.pyplot as plt
    import hyperspy.api_nogui as hs


    signal = hs.load(snakemake.input[0])
    signal.plot()
    fig = plt.gcf()
    fig.tight_layout()
    fig.savefig(snakemake.output[0])
    ```
    This is the actual script which will be run in the
    conda environment set up using the above `environment.yml`
3. `Snakefile`
    ```python
    from snakemake.remote.GS import RemoteProvider as GSRemoteProvider
    GS = GSRemoteProvider()


    rule step1:
        conda:
            "environment.yaml"
        resources:
            machine_type='n1-standard-1'
        input:
            GS.remote("nexuslims_example-datafiles/MockDataFiles/0.00V.dm3")
        output:
            "thumbnail.png"
        script:
            "test.py"
    ```
    This is the job configuration file which will be parsed by
    `snakemake` for submissions. In the above file, we configured
    a single step named *step1*. It will set up a conda
    environment defined in `environment.yaml`, and will be
    provisioning a machine satisfying the minimal requirement
    of *n1-standard-1* (The list of GCP Compute Engine machine
    type can be found [here](https://cloud.google.com/compute/docs/machine-types)).
    A list of input and output files are given, and they will
    be passed to `test.py` to run.

At last, we submit this job with the following command in terminal:
```bash
snakemake \
    --google-lifesciences \
    --default-remote-prefix nexuslims_example-joboutput \
    --use-conda \
    --google-lifesciences-region us-central1 \
    --preemption-default 1 \
    -j 1
```
By running the above command, we put the output file and logs
in `nexuslims_example-joboutput` bucket, and provision Compute
Engine resource from `us-central1` region, which is the same
region as the cloud storage. We instruct snakemake to use
preemptable VM to save the cost, and limit the job to run on
single core.

After the job is finished, we can inspect the result:
```bash
gsutil ls gs://nexuslims_example-joboutput/thumbnail.png
```
. This concludes the job submission example, feel free to
delete the generated example files locally or in the cloud
storage.

For more usage and configuration of `snakemake`,
please refer to its [documentation](https://snakemake.readthedocs.io/en/stable/).