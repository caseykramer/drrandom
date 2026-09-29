---
url: "/2007/08/17/forcing-redirection-to-https-in-asp-net/"
title: "Forcing redirection to HTTPS in ASP.Net"
date: 2007-08-17T21:02:01.000Z
slug: "forcing-redirection-to-https-in-asp-net"
tags: 
  - "uncategorized"
---

I ran into this issue today when trying to find a quick and dirty script to enforce redirection from HTTP to HTTPS for an intranet site we're enabling for HTTPS (yes, I know SSL on an intranet???  One word - Audit).  I thought this would be easy...I half expected that there would be an IIS setting to handle this for me; I was wrong on both counts.  So what I ended up doing is using an old DNN HttpModule which was set up to allow the user to specify specific tabs in DNN for SSL, and greatly simplifying things to work to my advantage.  I thought I would post it so I wouldn't have to look for it again.

To give a quick overview of what is going on, this is an HTTP Module which looks at a value in the web.config to determine whether or not to enforce HTTPS or not.  If the setting is set to true (or yes), then I'm just grabbing the Url property from the Request object, loading it up in a UriBuilder, and setting the Scheme to "https", and the port to 443 (this may not be necessary, but it was generating an URL with a port of 80 before, which defeats the purpose so I decided to play it safe).  It then feeds the generated URI to Response.Redirect(), and your off.  There is some additional code in there to disable the feature if your on localhost, which is mostly to keep from blowing up your dev box.

Here is the class:

```
using System;
using System.Configuration;
using System.Web;

namespace HTTPSRedirectHandler
{
    /// <summary>
    /// An HttpModule which redirects traffic to HTTPS based on configuration settings.
    /// </summary>
    public class HttpsRedirector : IHttpModule
    {
        HttpApplication _context;

        public HttpsRedirector()
        {
            //
            // TODO: Add constructor logic here
            //
        }

        #region IHttpModule Members

        /// <summary>
        /// Initializes a module and prepares it to handle
        /// requests.
        /// </summary>
        /// <param name="context">An <see cref="T:System.Web.HttpApplication"/> that provides access to the methods, properties, and events common to all application objects within an ASP.NET application</param>
        public void Init(HttpApplication context)
        {
            _context = context;
            _context.BeginRequest += new EventHandler(context_BeginRequest);
        }

        /// <summary>
        /// Disposes of the resources (other than memory) used by the
        /// module that implements <see langword="IHttpModule."/>
        /// </summary>
        public void Dispose()
        {
            _context.BeginRequest -= new EventHandler(context_BeginRequest);
            _context.Dispose();
        }

        #endregion

        /// <summary>
        /// Handles the BeginRequest event of the current Http Context.
        /// </summary>
        private void context_BeginRequest(object sender, EventArgs e)
        {
            bool useSSL = false;
            string result = null;
            if((result = ConfigurationSettings.AppSettings["RequireSSL"]) != null)
            {
                if(result.ToUpper() == "TRUE" || result.ToUpper() == "YES")
                    useSSL = true;
            }
            if (useSSL)
                EnforceSSL();
        }

        /// <summary>
        /// Enforces a redirection to HTTPS if the current connection is using HTTP (port 80).
        /// </summary>
        private void EnforceSSL()
        {
            if(_context.Request.ServerVariables["SERVER_NAME"].ToLower() != "localhost")
            {
                if(_context.Request.ServerVariables["SERVER_PORT"] == "80")
                {
                    UriBuilder uri = new UriBuilder(_context.Request.Url);
                    uri.Scheme = "https";
                    uri.Port = 443;
                    _context.Response.Redirect(uri.ToString());
                }
            }
        }
    }
}

```

[](http://11011.net/software/vspaste) 

Here is the web.config settings you need to use it:

```
<httpModules>
    <add name="SSLRedirect" type="HTTPSRedirectHandler.HttpsRedirector, HTTPSRedirectHandler" /></httpModules>
```

```
<appSettings>    <add key="RequireSSL" value="True" />        
</appSettings>
```
