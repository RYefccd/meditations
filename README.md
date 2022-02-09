# Meditations

技术组织文档----沉思录


使用 jupyter-book 中来构建技术共享文档.




## 部署


### 部署至github pages

    这里使用预先构建并推送将构建的页面推送到github的 pages 页面.

    pip install ghp-import
    ghp-import -n -p -f _build/html  # 把 _build/html 目录下构建的网页推送到 github 的 gh-pages 分支. 这样github的pages就会渲染页面.


### 部署至netlify

runtime.txt 是用于netlify构建时使用的环境变量.

参考链接:
https://github.com/netlify/build-image/blob/xenial/included_software.md#languages
https://github.com/netlify/build-image/blob/focal/included_software.md
https://docs.netlify.com/configure-builds/manage-dependencies/#python
https://jupyterbook.org/publish/netlify.html

