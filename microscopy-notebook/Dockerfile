# Adapted from jupyter/scipy-notebook
# https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile
ARG BASE_CONTAINER=jupyter/minimal-notebook
FROM $BASE_CONTAINER


USER root

# ffmpeg for matplotlib anim & dvipng+cm-super for latex labels;
# curl gnupg for cloud-sdk installation
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends ffmpeg dvipng cm-super curl gnupg

# install Cloud SDK
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | \
    tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && \
    apt-get update -y && apt-get install google-cloud-sdk -y

# cleanup
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


USER ${NB_UID}

# Install Python3 packages
RUN mamba install --quiet --yes \
    'conda-forge::blas=*=openblas' \
    'cython' \
    'h5py' \
    'ipympl'\
    'ipywidgets' \
    'matplotlib-base' \
    'numba' \
    'numexpr' \
    'pandas' \
    'pytables' \
    'scikit-image' \
    'scikit-learn' \
    'scipy' \
    'seaborn' \
    'widgetsnbextension' \
    'hyperspy' \
    'xlrd' \
    'bioconda::snakemake' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME="/home/${NB_USER}/.cache/"

RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions "/home/${NB_USER}"

USER ${NB_UID}

COPY README.md /tmp

WORKDIR "${HOME}"

# set project ID
ARG GCP_PROJECT_ID
RUN gcloud config set project "${GCP_PROJECT_ID}" && \
    gcloud config set pass_credentials_to_gsutil true