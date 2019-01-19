GraphicsNode
=======
Just as [MathJax](https://www.mathjax.org/) makes it easier to use the tex math equations on a web page, `GraphicsNode` makes it easy to get the corresponding graphics with scripts such as `metapost`, `graphviz` on `tikz` on a web page.

Installation
------------
- Prerequisites:
  
  - [TeX Live](https://www.tug.org/texlive/), An easy way to get up and running with the TeX document production system
  - [Graphviz](https://www.graphviz.org/), Graph visualization is a way of representing structural information as diagrams of abstract graphs and networks.
  - [OpenResty](http://openresty.org/en/), A full-fledged web platform that integrates the standard Nginx core, LuaJIT
  - [lua-gumbo](https://craigbarnes.gitlab.io/lua-gumbo/), A HTML5 parser and DOM library for Lua
  - [lua-resty-requests](https://github.com/tokers/lua-resty-requests), Yet Another HTTP library for OpenResty

- place `lib/resty` to your lua library path.

Getting Started
---------------
```bash
$ export PATH=/usr/local/openresty/nginx/sbin:$PATH
$ mkdir ~/www
$ cd ~/www
$ mkdir -p conf logs util html/images
$ nginx -p `pwd`/ -c conf/nginx.conf
```

- conf/nginx.conf
```
worker_processes 1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
env PATH;
http {
    lua_shared_dict gxn_cache 10m;
    
    server {
        listen 8080;
        
        resolver 8.8.8.8;
        include mime.types;
        
        location /sample {
            default_type text/html;
            content_by_lua_block {
                require("resty.gxn")()
            }
        }
    }
}
```

- util/gxn.sh
```bash
#!/bin/bash

cd $1

ERROR=0

case $2 in
    mplibcode)
        $5 $3
        ERROR=$?
        ;;
    graphviz)
        $5 -Tsvg $3.gv -o $3.$4
        ERROR=$?
        ;;
    tikzpicture)
        $5 $3
        pdf2svg $3.pdf $3.svg
        ERROR=$?
        ;;
    *)
        echo 'NOT SUPPORTED'
esac

rm -rf *.mp *.mpx *.gv

exit $ERROR
```

- html/sample.html
```html
<html>
<body>

<hr>
<mplibcode>
beginfig(1)
  pair A, B, C;
  A:=(0,0); B:=(1cm,0); C:=(0,1cm);
  draw A--B--C;
endfig;
</mplibcode>

<hr>
<mplibcode src="/source/triangle.mp" width="300"/>

<hr>
<mplibcode src="http://www.cs.ucc.ie/~dongen/mpost/mp/Escher87.mp"/>

<hr>
<tikzpicture src="/source/cylinder.tikz"/>

<hr>
<graphviz cmd="dot">
  digraph G {Hello->World}
</graphviz>

</body>
</html>
```


Copyright (C) 2018-2019 Soojin Nam
