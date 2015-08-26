======================
varnish-vcl-collection
======================

Collection of Varnish VCL files.  The purpose of this collection is to make
it easier to configure varnish for some common setups and behaviors.  Some
of these files require you to define specific ACLs in your main VCL, and also
note that the order of including files could change the behavior of Varnish.

The main goal for this VCL Collection is to provide a simple config with just
a few options that can make Varnish work for the majority of sites. Most of
the "configuration" is done by choosing which files to include and defining
ACLs. We hope to provide a robust configuration that can be used with
relatively little information or control over what is actually running on
the backend.  This is particularly useful for managed web hosts who want
to have customers benefit from the performance of a caching proxy without
limiting what can run on the web backend and without customizing the
configuration for each customer.

See `wordpress-example.vcl` for an example of how to use this collection
of VCLs to configure Varnish for a WordPress site.


CloudFlare
==========

If you use CloudFlare and the backend is running `mod_cloudflare` (for Apache)
or `http_realip_module` (for Nginx), then you should use `lib/cloudflare.vcl`.

CloudFlare uses the HTTP Header `CF-Connecting-IP` to store the original
client IP address.  Note that unlike the `X-Forwarded-For` Header, this just
contains the original IP address, not the entire forward chain.

If you are using a CloudFlare Railgun server, or have any other trusted proxy
servers between Varnish and CloudFlare, you will need to specify them in the
`cloudflare` ACL.

Also note that you will need to make sure that Varnish's IP is configured
as trusted in `mod_cloudflare` or the `http_realip_module` on the backend.

Example usage::

	include "lib/cloudflare.vcl";

	acl cloudflare {
		# put the IP of your Railgun (or proxy) server here
		# "1.2.3.4";
	}


HTTP Purging
============

Include `lib/purge.vcl` to allow purging pages from the cache using the HTTP
PURGE method.  This uses the ban feature of varnish and will make bans take
advantage of Varnish's ban-lurker.  You will need to specify a `purge` ACL
so that only requests coming from your application are allowed to purge.

Example usage::

	include "lib/purge.vcl";

	acl purge {
		# include the IP of your app if it isn't on localhost
		"localhost";
		"127.0.0.1";
	}

There are several different possible behaviors of HTTP purging which can be
controlled with the X-Purge-Method HTTP header.  This config will be smart
and attempt to automatically pick the best method based on the URL if you
don't use an X-Purge-Method header.  See the comments in `lib/purge.vcl` for
details.


Static File Caching
===================

Include `lib/static.vcl` to use a simple set of rules to cache static files.
This will ignore the query string part of request URLs, and discard all
cookies for these requests.  This will also cache static files for 24 hours.

The cache behavior for this vcl can be bypassed by adding `nocache` to the
url.  For example, `http://example.com/foo.jpg?nocache` will always
retrieve the file from the backend instead of serving from the cache.
