ARG BASE_CONTAINER=jupyter/minimal-notebook
FROM $BASE_CONTAINER:latest

USER ${NB_UID}

# Install Python3 packages
RUN conda install --quiet --yes \
    'gsutil' \
    'google-cloud-storage' && \
    conda clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

WORKDIR "${HOME}"
