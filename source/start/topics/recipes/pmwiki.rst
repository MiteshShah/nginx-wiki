
.. meta::
   :description: A sample NGINX configuration for PmWiki.

.. _recipe-pmwiki:

PmWiki
======

PmWiki is a wiki-based system for collaborative creation and maintenance of
websites.

Requirements
------------

* `php-fpm <http://php-fpm.org/>`__

Recipe
----------

The configuration uses a separated, reusable layout. It adds three files.

.. code-block:: nginx

    server {
        server_name wiki.example.com;
        root /srv/www/pmwiki/example.com/wiki/public;

        index pmwiki.php;

        location ~ ^/(cookbook|local|scripts|wiki.d|wikilib.d) {
            deny all;
        }

        location / {
            try_files $uri $uri/ @pmwiki;
        }

        location @pmwiki {
            rewrite ^/(.*) /pmwiki.php?n=$1;
        }

        ## php configuration using unix sockets.
        location ~ \.php$ {
            include /etc/nginx/fastcgi_params;
            fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
        }


        # cache configuration for most common files
        location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
            # Some basic cache-control for static files to be sent to the browser
            expires max;
            add_header Pragma public;
            add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }

        # drop common log errors
        location = /robots.txt { access_log off; log_not_found off; }
        location = /favicon.ico { access_log off; log_not_found off; }
        location ~ /\. { access_log off; log_not_found off; deny all; }
        location ~ ~$ { access_log off; log_not_found off; deny all; }
    }
