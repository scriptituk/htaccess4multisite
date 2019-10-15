# htaccess4multisite

Here is a generic .htaccess mod_rewrite solution for hosting multiple websites on a single server using domain aliases (parked domains) so that they function like add-on domains, each in its own subdirectory.

To make it generic and automatic we adopt the convention of naming each subdirectory the same as the mid-level component of the domain name, i.e. the part after any `www.` but before the domain name extensions (ccTLDs & gTLDs). The primary domain files should also reside in a similar subdirectory.

So files for primary domain `mainplace.com` would go under `/mainplace/` in the document root,  
and files for parked domain `otherplace.org.uk` would go under `/otherplace/`, etc.

Here’s the .htaccess boilerplate code:

```
RewriteEngine on
Rewrite Base /

# set env vars
RewriteCond %{HTTP_HOST} ^(www\.)?(.*?)\.(.*)$
RewriteRule ^ - [E=E_SCHEME:,E=E_SCHEME:http,E=E_HOST:,E=E_HOST:%2.%3,E=E_DIR:,E=E_DIR:%2]
RewriteCond %{HTTPS} On [NC]
RewriteRule ^ - [E=E_SCHEME:https]

# permanent redirect with www.
#RewriteCond %{ENV:REDIRECT_STATUS} ^$
#RewriteCond %{HTTP_HOST} !^www\.
#RewriteRule .* %{ENV:E_SCHEME}://www.%{ENV:E_HOST}/$0 [R=301,L]

# permanent redirect without www.
RewriteCond %{ENV:REDIRECT_STATUS} ^$
RewriteCond %{HTTP_HOST} ^www\.
RewriteRule .* %{ENV:E_SCHEME}://%{ENV:E_HOST}/$0 [R=301,L]

# permanent redirect without dir/
RewriteCond %{ENV:REDIRECT_STATUS} ^$
RewriteCond %{ENV:E_DIR}==%{REQUEST_URI} ^(.*?)==/\1/?(.*)$
RewriteRule ^ /%2 [R=301,L]

# redirect to dir/
RewriteCond %{ENV:E_DIR}==%{REQUEST_URI} !^(.*?)==/\1.*$
RewriteRule .* /%{ENV:E_DIR}/$0 [L]

# test if exists
RewriteCond %{REQUEST_FILENAME} -d [OR]
RewriteCond %{REQUEST_FILENAME} -f
RewriteRule ^ - [L]
```

What it does is:
* force a 301 (permanent) redirect to remove the `www.` if present in the request URI
* force a 301 redirect to remove the subdirectory if present somehow in the request URI
* do a 302 (found) redirect to the subdirectory for the website files to fake found status
* stop rewriting and pass through with no substitution if path found

That’s it.
It works by storing domain components in environment variables to save duplicating string literals and using those stored values in the conditions and rules.
The empty `REDIRECT_STATUS` test prevents recursion.

For WordPress sites,
set both the WordPress Address and the Site Address in WP Admin General Settings to the root of the alias domain (or primary domain if the primary is a WordPress site),
then prepend the line `RewriteOptions Inherit` to the .htaccess file in the WordPress directory to inherit the configuration of the parent.
Parent rules are applied after child rules but if you know it's Apache v2.4 use `RewriteOptions InheritBefore` instead.

I’m using this to host multiple sites on one account for under £1/month total, adding free aliases as needed without having to touch the redirection logic.

I will extend it for subdomains one day…

**References**
* [Apache Module mod_rewrite v2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html) – or [v2.2](http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html)
* [Apache mod_rewrite in .htaccess in Depth](http://www.ckollars.org/apache-rewrite-htaccess.html) – excellent guidance by Chuck Kollars

