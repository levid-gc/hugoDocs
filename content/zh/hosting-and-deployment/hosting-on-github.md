---
title: 在 GitHub 上托管
linktitle: 在 GitHub 上托管
description: 将 Hugo 部署为一个 GitHub Pages 项目或个人/机构站点并使用一个简单 shell 脚本自动化整个过程。
date: 2014-03-21
publishdate: 2014-03-21
lastmod: 2017-03-30
categories: [hosting and deployment]
keywords: [github,git,deployment,hosting]
authors: [Spencer Lyon, Gunnar Morling]
menu:
  docs:
    parent: "hosting-and-deployment"
    weight: 30
weight: 30
sections_weight: 30
draft: false
toc: true
aliases: [/tutorials/github-pages-blog/]
---

GitHub 在 SSL 上为个人，机构，或项目页面提供了免费且快速的静态托管，在 GitHub 仓储中直接通过它的 [GitHub Pages service][]。

## 假设

1. 您拥有 Git 2.8 或更高版本 [安装在您的机器上][installgit]。
2. 您拥有一个 GitHub 账户。GitHub 的 [注册][ghsignup] 是免费的。
3. 您拥有一个即将发布的 Hugo 网站或至少完成了 [Quick Start][] 部分。

## GitHub Pages 类型

GitHub Pages 有两种类型：

- 用户/机构 Pages (`https://<USERNAME|ORGANIZATION>.github.io/`)
- 项目 Pages (`https://<USERNAME|ORGANIZATION>.github.io/<PROJECT>/`)

请参考 [GitHub Pages 文档][ghorgs] 以决定您选择的站点类型，因为这会影响下面使用的方法。

如要创建用户/机构 Pages 站点，请遵循下面的 *GitHub 用户或机构 Pages* 部分。

如要创建项目 Pages 站点，请选择 *项目 Pages* 部分的一个方法。

## GitHub 用户或机构 Pages

[GitHub Pages 文档][ghorgs] 提到，除了项目 pages，您还可以托管用户/机构 pages。下面是用户和机构 GitHub Pages 的关键不同之处：

1. 您必须使用一个 `<USERNAME>.github.io` 来托管您的 **生成的** 内容
2. 将会使用来自 `master` 分支的内容发布您的 GitHub Pages 站点

这是一个相对简单得多的设置，因为您的 Hugo 文件和生成的内容是发布在两个不同的仓储中的。

### 步骤指导

1. 在 GitHub 中创建一个 `<YOUR-PROJECT>`（比如，`blog`）仓储。这个仓储中将会包含 Hugo 的内容和其他的源文件。
2. 创建一个 `<USERNAME>.github.io` GitHub 仓储。这个仓储包含的就是完全渲染好的 Hugo 站点版本。
3. `git clone <YOUR-PROJECT-URL> && cd <YOUR-PROJECT>`
4. 确保您的站点在本地能够正常运行（`hugo server` 或 `hugo server -t <YOURTHEME>`）并且打开浏览器访问 <http://localhost:1313>。
5. 一旦您对结果满意了：
    * 按下 <kbd>Ctrl</kbd>+<kbd>C</kbd> 以关闭服务
    * 运行 `rm -rf public` 来完全删除 `public` 目录
6. `git submodule add -b master git@github.com:<USERNAME>/<USERNAME>.github.io.git public`。这将会创建一个 git [submodule][]。现在当您运行 `hugo` 命令将您的站点编译到 `public` 目录中，那么创建的 `public` 目录将会拥有一个不同的远程源（比如，托管的 GitHub 仓储）。您可以使用下面的脚本来自动化某些这些步骤。

### 将它放进一个脚本

You're almost done. You can also add a `deploy.sh` script to automate the preceding steps for you. You can also make it executable with `chmod +x deploy.sh`.

The following are the contents of the `deploy.sh` script:

```
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..
```


You can then run `./deploy.sh "Your optional commit message"` to send changes to `<USERNAME>.github.io`. Note that you likely will want to commit changes to your `<YOUR-PROJECT>` repository as well.

That's it! Your personal page should be up and running at `https://<USERNAME>.github.io` within a couple minutes.

## GitHub Project Pages

{{% note %}}
Make sure your `baseURL` key-value in your [site configuration](/getting-started/configuration/) reflects the full URL of your GitHub pages repository if you're using the default GH Pages URL (e.g., `<USERNAME>.github.io/<PROJECT>/`) and not a custom domain.
{{% /note %}}

### Deployment of Project Pages from `/docs` folder on `master` branch

[As described in the GitHub Pages documentation][ghpfromdocs], you can deploy from a folder called `docs/` on your master branch. To effectively use this feature with Hugo, you need to change the Hugo publish directory in your [site's][config] `config.toml` and `config.yaml`, respectively:

```
publishDir = "docs"
```
```
publishDir: docs
```

After running `hugo`, push your master branch to the remote repository and choose the `docs/` folder as the website source of your repo. Do the following from within your GitHub project:

1. Go to **Settings** &rarr; **GitHub Pages**
2. From **Source**,  select "master branch /docs folder". If the option isn't enabled, you likely do not have a `docs/` folder in the root of your project.

{{% note %}}
The `docs/` option is the simplest approach but requires you set a publish directory in your site configuration. You cannot currently configure GitHub pages to publish from another directory on master, and not everyone prefers the output site live concomitantly with source files in version control.
{{% /note %}}

### Deployment of Project Pages From Your `gh-pages` branch

You can also tell GitHub pages to treat your `master` branch as the published site or point to a separate `gh-pages` branch. The latter approach is a bit more complex but has some advantages:

* It keeps your source and generated website in different branches and therefore maintains version control history for both.
* Unlike the preceding `docs/` option, it uses the default `public` folder.

#### Preparations for `gh-pages` Branch

These steps only need to be done once. Replace `upstream` with the name of your remote; e.g., `origin`:

##### Add the `public` Folder

First, add the `public` folder to your `.gitignore` file at the project root so that the directory is ignored on the master branch:

```
echo "public" >> .gitignore
```

##### Initialize Your `gh-pages` Branch

You can now initialize your `gh-pages` branch as an empty [orphan branch][]:

```
git checkout --orphan gh-pages
git reset --hard
git commit --allow-empty -m "Initializing gh-pages branch"
git push upstream gh-pages
git checkout master
```

#### Build and Deployment

Now check out the `gh-pages` branch into your `public` folder using git's [worktree feature][]. Essentially, the worktree allows you to have multiple branches of the same local repository to be checked out in different directories:

```
rm -rf public
git worktree add -B gh-pages public upstream/gh-pages
```

Regenerate the site using the `hugo` command and commit the generated files on the `gh-pages` branch:

{{< code file="commit-gh-pages-files.sh">}}
hugo
cd public && git add --all && git commit -m "Publishing to gh-pages" && cd ..
{{< /code >}}

If the changes in your local `gh-pages` branch look alright, push them to the remote repo:

```
git push upstream gh-pages
```

##### Set `gh-pages` as Your Publish Branch

In order to use your `gh-pages` branch as your publishing branch, you'll need to configure the repository within the GitHub UI. This will likely happen automatically once GitHub realizes you've created this branch. You can also set the branch manually from within your GitHub project:

1. Go to **Settings** &rarr; **GitHub Pages**
2. From **Source**,  select "gh-pages branch" and then **Save**. If the option isn't enabled, you likely have not created the branch yet OR you have not pushed the branch from your local machine to the hosted repository on GitHub.

After a short while, you'll see the updated contents on your GitHub Pages site.

#### Put it Into a Script

To automate these steps, you can create a script with the following contents:

{{< code file="publish_to_ghpages.sh" >}}
#!/bin/sh

DIR=$(dirname "$0")

cd $DIR/..

if [[ $(git status -s) ]]
then
    echo "The working directory is dirty. Please commit any pending changes."
    exit 1;
fi

echo "Deleting old publication"
rm -rf public
mkdir public
git worktree prune
rm -rf .git/worktrees/public/

echo "Checking out gh-pages branch into public"
git worktree add -B gh-pages public upstream/gh-pages

echo "Removing existing files"
rm -rf public/*

echo "Generating site"
hugo

echo "Updating gh-pages branch"
cd public && git add --all && git commit -m "Publishing to gh-pages (publish.sh)"
{{< /code >}}

This will abort if there are pending changes in the working directory and also makes sure that all previously existing output files are removed. Adjust the script to taste, e.g. to include the final push to the remote repository if you don't need to take a look at the gh-pages branch before pushing. Or adding `echo yourdomainname.com >> CNAME` if you set up for your gh-pages to use customize domain.

### Deployment of Project Pages from Your `master` Branch

To use `master` as your publishing branch, you'll need your rendered website to live at the root of the GitHub repository. Steps should be similar to that of the `gh-pages` branch, with the exception that you will create your GitHub repository with the `public` directory as the root. Note that this does not provide the same benefits of the `gh-pages` branch in keeping your source and output in separate, but version controlled, branches within the same repo.

You will also need to set `master` as your publishable branch from within the GitHub UI:

1. Go to **Settings** &rarr; **GitHub Pages**
2. From **Source**,  select "master branch" and then **Save**.

## Use a Custom Domain

If you'd like to use a custom domain for your GitHub Pages site, create a file `static/CNAME`. Your custom domain name should be the only contents inside `CNAME`. Since it's inside `static`, the published site will contain the CNAME file at the root of the published site, which is a requirements of GitHub Pages.

Refer to the [official documentation for custom domains][domains] for further information.

[config]: /getting-started/configuration/
[domains]: https://help.github.com/articles/using-a-custom-domain-with-github-pages/
[ghorgs]: https://help.github.com/articles/user-organization-and-project-pages/#user--organization-pages
[ghpfromdocs]: https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/#publishing-your-github-pages-site-from-a-docs-folder-on-your-master-branch
[ghsignup]: https://github.com/join
[GitHub Pages service]: https://help.github.com/articles/what-is-github-pages/
[installgit]: https://git-scm.com/downloads
[orphan branch]: https://git-scm.com/docs/git-checkout/#git-checkout---orphanltnewbranchgt
[Quick Start]: /getting-started/quick-start/
[submodule]: https://github.com/blog/2104-working-with-submodules
[worktree feature]: https://git-scm.com/docs/git-worktree
