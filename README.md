Starting over is never easy. Here are some notes...

## Getting started

For starters, clone your repo.

### Installation

Install Mkdocs and dependencies if needed. 

#### Dependencies 
- Python
- Github CLI
- VS Code
  
#### Create a virtual environment (optional)

``` ps
cd \<project dir\>
python -m venv venv
source venv/Scripts/activate
```

On macOS:
``` sh
source venv/bin/activate
```
#### Install with pip (venv)

``` py
pip install mkdocs
pip install mkdocs-material
```
#### Install with python (w/o venv)

On Windows:
``` ps
python -m pip install mkdocs
python -m pip install mkdocs-material
```

``` py
pip install -r requirements.txt
```

This will automatically install compatible versions of all dependencies:
[MkDocs], [Markdown], [Pygments] and [Python Markdown Extensions] + all of the plugins you've added.

#### Serve Locally
(venv)
 ``` py
 mkdocs serve
 ```

Windows 
 ``` ps
 python -m mkdocs serve
 ```

## How to upgrade

Upgrade to the latest version with:

```
pip install --upgrade --force-reinstall mkdocs-material
```

Show the currently installed version with:

```
pip show mkdocs-material
```


## More information

#### Documentation
[MkDocs](https://www.mkdocs.org/) <br />
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
[Installing packages using pip and virtual environments — Python Packaging User Guide](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/)


###### Further Reading 
[GitHub - squidfunk/mkdocs-material: Documentation that simply works](https://github.com/squidfunk/mkdocs-material) <br />
[Publishing Obsidian.md notes with GitLab Pages | GitLab](https://about.gitlab.com/blog/2022/03/15/publishing-obsidian-notes-with-gitlab-pages/)


