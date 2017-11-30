---
title: Jypyter Notebook Best Practice
---

# Jypyter Notebook Best Practice

## 版本控制

同时提交 py 和 html 文件，方便进行 codereview。

通过 jupyter-notebook 的 FileContentsManager.post_save_hook 实现，添加如下代码到 jupyter-notebook 配置文件：

```python
import os
from subprocess import check_call

def post_save(model, os_path, contents_manager):
    """post-save hook for converting notebooks to .py scripts"""
    if model['type'] != 'notebook':
        return # only do this for notebooks
    d, fname = os.path.split(os_path)
    check_call(['jupyter', 'nbconvert', '--to', 'script', fname], cwd=d)
    check_call(['jupyter', 'nbconvert', '--to', 'html', fname], cwd=d)

c.FileContentsManager.post_save_hook = post_save
```

## 目录结构/文件命名

```shell
- develop # (Lab-notebook style)
 + [ISO 8601 date]-[DS-initials]-[2-4 word description].ipynb
 + 2015-06-28-jw-initial-data-clean.html
 + 2015-06-28-jw-initial-data-clean.ipynb
 + 2015-06-28-jw-initial-data-clean.py
 + 2015-07-02-jw-coal-productivity-factors.html
 + 2015-07-02-jw-coal-productivity-factors.ipynb
 + 2015-07-02-jw-coal-productivity-factors.py
- deliver # (final analysis, code, presentations, etc)
 + Coal-mine-productivity.ipynb
 + Coal-mine-productivity.html
 + Coal-mine-productivity.py
- figures
 + 2015-07-16-jw-production-vs-hours-worked.png
- src # (modules and scripts)
 + init.py
 + load_coal_data.py
 + figures # (figures and plots)
 + production-vs-number-employees.png
 + production-vs-hours-worked.png
- data (backup-separate from version control)
 + coal_prod_cleaned.csv
```

There are many benefits to this workflow and structure. The first and primary one is that they create a historical record of how the analysis progressed. It’s also easily searchable:

* by date (ls 2015-06*.ipynb)
* by author (ls 2015*-jw-*.ipynb)
* by topic (ls *-coal-*.ipynb)

## 数据加载

避免在不同位置多次导入同一份数据。通过 head 函数导入部分数据：

```python
dframe = pd.read_csv('./data/titanic-data.csv')
dframe.head()
```

### Qgrid

[grid](https://github.com/quantopian/qgrid) - SlickGrid in Jupyter Notebooks

![](/images/15077993187069/15079714067211.jpg)

**注：qgrid 包对于 ipywidgets 和 jupyter notebook 的版本存在严格的依赖关系，盲目使用最新版本可能导致 qgrid 不可用**

## ast_node_interactivity

当在同一个 code cell 中同时使用 dataframe 的 head、tail 等多个方法时，Jupyter Notebook 的默认行为是只显示其中的最后一个。如果需要将他们同时显示出来，则需要修改如下配置：

```python
# see value of statements at once
from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"
```

效果如下：

![](/images/15077993187069/15080350603391.jpg)

## 插件

### Jupyter-contrib extensions

Jupyter-contrib extensions is a family of extensions which give Jupyter a lot more functionality, including e.g. jupyter spell-checker and code-formatter.

```bash
pip install https://github.com/ipython-contrib/jupyter_contrib_nbextensions/tarball/master
pip install jupyter_nbextensions_configurator
jupyter contrib nbextension install --user
jupyter nbextensions_configurator enable --user
```

![](/images/15077993187069/15081178978067.jpg)

### Create a presentation

```bash
pip install RISE
jupyter-nbextension install rise --py --sys-prefix
jupyter-nbextension enable rise --py --sys-prefix
```

![](/images/15077993187069/15081186000636.jpg)

## Tips

* 通过 `_` + 数字访问之前的 cell 的输出，例如：

![](/images/15077993187069/15078594496187.jpg)

* 通过 `_!` + 数字访问之前 cell 的输入，例如：

![](/images/15077993187069/15078595418789.jpg)

* put imports at the top of the Notebook
* 通过 `?` + 函数名获取 docstring 中的文档信息

![](/images/15077993187069/15080352795694.jpg)

* [magic commands](http://ipython.readthedocs.io/en/stable/interactive/magics.html) `%`: line command, `%%`: cell command
* 通过 `%env` 控制环境变量而无需重启 jupyter server
* 通过 `%load` 从外部向当前 code cell 中加载代码，可以为路径或 url
* 通过 `%store` 在两个 notebook 之间传递数据
* 通过 `%who type` 获取当前 notebook 中的全局变量，type 可以为 str 等数据类型
* 使用 `%timeit` 或 `%%time` 测试代码的运行耗时
* Suppress the output of final function with a semicolon at the end
* 通过 `!ls` 方式执行 shell 命令或使用 `%%bash` 将解释器变更为 bash。甚至可以以如下方式使用：

![](/images/15077993187069/15081187831139.jpg)


# References

* https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/
* https://svds.com/jupyter-notebook-best-practices-for-data-science/
* http://blog.juliusschulz.de/blog/ultimate-ipython-notebook
* https://www.dataquest.io/blog/how-to-setup-a-data-science-blog/


