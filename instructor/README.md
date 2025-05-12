# Instructor Guide for ENTER_HoL_NAME

This guide provides detailed instructions on setting up the workshop environment, preparing virtual machines with the necessary prerequisites, and publishing the workshop guide for participants.

## Hands on Lab Environment Setup

_Any details on how to setup HoL environment_

## Publishing the Workshop Guide to GitHub Pages

Two methods are available to publish this automation lab guide to GitHub pages - a GitHub Action (recommended) and manually publishing.

> [!NOTE]  
> Follow the manual steps to test your guide locally (via the `mkdocs serve` command).

### Using GitHub Action

The [publish_mkdocs.yml](../.github/workflows/publish_mkdocs.yml) GitHub action is used to automatically publish the the lab guide to GitHub pages.

The action is triggered on a push to the `main` branch. The action can also be launched manually if required.

### Steps to manually publish the guide

* Create a Python Virtual Environment

   ```bash
   python3 -m venv ~/mkdocs_venv
   source ~/mkdocs_venv/bin/activate
   ```

* Clone the <REPOSITORY_NAME> GitHub repository

  ```bash
  git clone https://github.infra.cloudera.com/GOES/<REPOSITORY_NAME>.git
  ```

* Install Required Dependencies for MkDocs

   ```bash
   cd <REPOSITORY_NAME>/instructor/mkdocs
   pip install -r requirements.txt
   ```

* Run the following command to test your guide locally:

   ```bash
   mkdocs serve
   ```

   Open `http://127.0.0.1:8000` in your browser to view the guide.

* Use the following command to publish the guide to the `origin` repository's GitHub Pages:

   ```bash
   mkdocs gh-deploy -r origin --no-history
   ```

* The lab guide should now be live on your GitHub Pages site. The URL for the site can be found via the `Settings -> Pages` menu of the repository, or go to https://github.infra.cloudera.com/pages/GOES/<REPOSITORY_NAME>/
