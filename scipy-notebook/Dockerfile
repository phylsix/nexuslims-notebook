ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER:latest

LABEL maintainer="Weinan Si <w.si@euclidtechlabs.com>"

USER ${NB_UID}

# Install Python3 packages
RUN conda install --quiet --yes \
    'gsutil' \
    'google-cloud-storage' && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

COPY README.md "/home/${NB_USER}"

WORKDIR "${HOME}"
