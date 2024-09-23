
# Contributing Guide

Thank you for deciding to contribute and help us improve Percona Packages for Valkey documentation!

We welcome contributors from all users and community. By contributing, you agree to the [Percona Community code of conduct](https://github.com/percona/community/blob/main/content/contribute/coc.md).

You can contribute to documentation in the following ways:

1. [Contribute to Valkey documentation](https://github.com/valkey-io/valkey-doc/blob/main/README.md). 
2. **[Contribute to the current documentation](#contribute-to-the-current-documentation)**. Click the **Edit this page** icon next to the page title. This link leads you to the source file of the page on GitHub. There you make changes, create a pull request that we review and add to the doc project. For details how to do it, read on.

## Contribute to the current documentation 

Percona Packages for Valkey documentation is written in [Markdown](https://www.markdownguide.org/basic-syntax/) language, so you can 
[edit it online via GitHub](#edit-documentation-online-vi-github). If you wish to have more control over the doc process, jump to how to [edit documentation locally](#edit-documentation-locally). 

To contribute to the documentation, you should be familiar with the following technologies:

- [MkDocs](https://www.mkdocs.org/getting-started/) documentation generator. We use it to convert source ``.md`` files to .html and PDF documents.
- [git](https://git-scm.com/) and [GitHub](https://guides.github.com/activities/hello-world/)
- [Docker](https://docs.docker.com/get-docker/). It allows you to run MkDocs in a virtual environment instead of installing it and its dependencies on your machine.

The .md files are in the ``docs`` directory. 

### Edit documentation online via GitHub

1. Click the **Edit this page** link on the sidebar. The source ``.md`` file of the page opens in GitHub editor in your browser. If you haven’t worked with the repository before, GitHub creates a [fork](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) of it for you.

2. Edit the page. You can check your changes on the **Preview** tab.

3. Commit your changes.

	 - In the *Commit changes* section, describe your changes.
	 - Select the **Create a new branch for this commit and start a pull request** option
	 - Click **Propose changes**.

4. GitHub creates a branch and a commit for your changes. It loads a new page on which you can open a pull request to Percona. The page shows the base branch - the one you offer your changes for, your commit message and a diff - a visual representation of your changes against the original page.  This allows you to make a last-minute review. When you are ready, click the **Create pull request** button.
5. Someone from our team reviews the pull request and if everything is correct, merges it into the documentation. Then it gets published on the site.

### Edit documentation locally

This option is for users who prefer to work from their computer and / or have the full control over the documentation process.

The steps are the following:

1. Fork this repository
2. Clone the repository on your machine:

```sh
git clone git@github.com:<your_name>/percona-valkey-doc.git
```

3. Change the directory to ``percona-valkey-doc`` and add the remote upstream repository:

```sh
git remote add upstream git@github.com:percona/percona-valkey-doc.git
```

4. Pull the latest changes from upstream

```sh
git fetch upstream
git merge upstream/<branch>
```

Make sure that your local branch and the branch you merge changes from are the same. So if you are on the ``main`` branch, merge changes from ``upstream/main``.

5. Create a separate branch for your changes

```sh
git checkout -b <my_branch>
```

6. Make changes
7. Commit your changes
8. Open a pull request to Percona

### Building the documentation

To verify how your changes look, generate the static site with the documentation. This process is called *building*. 
You can do it in these ways:

- [use Docker](#use-docker)
- [install MkDocs and build locally](#install-mkdocs-and-build-locally)

Learn more about the documentation structure in the [Repository structure](#repository-stucture) section.


#### Use Docker

1. [Get Docker](https://docs.docker.com/get-docker/)
2. We use [this Docker image](https://github.com/Percona-Lab/percona-doc-docker) to build documentation. Run the following command:

```sh
docker run --rm -v $(pwd):/docs perconalab/pmm-doc-md mkdocs build
```

   If Docker can't find the image locally, it first downloads the image, and then runs it to build the documentation.

3. Go to the ``site`` directory and open the ``index.html`` file to see the documentation.
4. To view your changes as you make them, run the following command:

``` sh
docker run --rm -p 8000:8000 -v $(pwd):/docs perconalab/pmm-doc-md mkdocs serve -a 0.0.0.0:8000
```

#### Install MkDocs and build locally

1. Install [pip](https://pip.pypa.io/en/stable/installing/)
2. Install [MkDocs](https://www.mkdocs.org/getting-started/#installation).
3. While in the root directory of the doc project, run the following command to build the documentation:

```sh
mkdocs build 
```

4. Go to the ``site`` directory and open the ``index.html`` file in your web browser to see the documentation.
5. To automatically rebuild the documentation and reload the browser as you make changes, run the following command:

```sh
mkdocs serve 
```


## Repository structure

The repository includes the following directories and files:

- `mkdocs.yml` - the configuration file. It includes general settings and documentation structure.
- `docs`:
  - `*.md` - Source markdown files.
  - `_images` - Images, logos and favicons
  - `css` - Styles
  - `js` - Javascript files
- `_resource`:
   - `.icons` - The folder with the icons used in the documentation
   - overrides - The folder with the Material theme template customization for builds
- `site` - This is where the output HTML files are put after the build