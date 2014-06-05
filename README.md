### NuGet CouchbaseAspNet Extended Package

### Some Links

- NuGet Package available at https://nuget.org/packages/CouchbaseAspNetExtended
- Source code & documentation available at https://github.com/evereq/couchbase-aspnet

### Library Description

This library provides infrastructure support for using [Couchbase Server](http://couchbase.com) and ASP.NET.

## Features:

ASP.NET SessionState Provider

* Port of the [Enyim Memcached Provider](https://github.com/enyim/memcached-providers) to Couchbase Server with some improvements
* Session data compression is supported, starting from version 1.3.0
* Starting from version 1.3.2 NLog logging can be enabled to output basic information for all Save / Load actions (item id and size before and after compression)
* LZ4 fast compression algorithm supported in version 1.3.1 and was default algorithm if compression enabled for 1.3.1 version.
* QuickLZ very fast compression algorithm supported in version 1.3.2 and now is default algorithm if compression enabled (in version 1.3.2).
* LZ4Sharp library used for fastest compression (https://github.com/stangelandcl/LZ4Sharp) and added as project to solution (licensed under BSD License)
* QuickLZ DLL used from http://www.quicklz.com/download.html. Please make sure you read license details for QuickLZ at http://www.quicklz.com/order.html
and purchase commercial license if required.
* Provider write his configuration during start to Windows Event Log (version 1.3.1) and in addition to log file (starting from version 1.3.2).
  Check Application/CouchbaseSessionStateProvider source (using Event Viewer) and Logs/CouchbaseSessionStateProvider.log file
* Starting from version 1.3.0, trivial MVC example was removed from solution.
* Starting from version 1.3.3.3 target framework set to .NET 4.5

## Requirements

* You'll need .NET Framework 4.5 or later to use the precompiled binaries.
* To build the client, you'll need Visual Studio 2012.
* The Nuget package for CouchbaseNetClient (http://nuget.org/packages/CouchbaseNetClient) is referenced by Couchbase.AspNet
* Couchbase Server 1.8 or 2.0
* Important: If you enable QuickLZ compression, please make sure you have QuickLZ dlls in the 'QuickLZC' subfolder inside bin folder of your application.

## Configuring the SessionState provider

Update the sessionState section in Web.config as follows:

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" />
      </providers>
    </sessionState>
        
Configure the Couchbase Client as you normally would:

    <section name="couchbase" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase"/>	

    <couchbase>
        <servers bucket="default" bucketPassword="">
        <add uri="http://127.0.0.1:8091/pools"/>      
        </servers>
    </couchbase>

If you would like to use a custom configuration section, you may do so by specifying a value for the "section" attribute of the provider entry (see below).

    <section name="couchbaseSession" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase"/>    

    <couchbaseSession>
        <servers bucket="sessionState" bucketPassword="">
        <add uri="http://127.0.0.1:8091/pools"/>      
        </servers>
    </couchbaseSession>

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" section="couchbaseSession" />
      </providers>
    </sessionState>

If you would like to use a custom client factory, you may do so by specifying a value in the "factory" attribute of the provider entry. The example below sets it to the default factory, but you can replace this with your own factory class to have full control over the creation and lifecycle of the Couchbase client.

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" factory="Couchbase.AspNet.SessionState.CouchbaseClientFactory" />
      </providers>
    </sessionState>

This session handler also supports the ability to disable exclusive session access for ASP.NET sessions if desired. You can set the value using the "exclusiveAccess" attribute of the provider entry.

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" exclusiveAccess="false" />
      </providers>
    </sessionState>

This session handler also supports the ability to enable session data compression if desired. By default compression is disabled. 
Note: you can't enable / disable sessions compression if your storage already contain some users session data! You need to clean up session storage before change compression mode.
You may want to enable compression of session data on client side (IIS process), if your web servers CPU capacity exceed your network or session storage performance capacity or if you want to bypass Memcached / Couchbase item size limitation of 1Mb / 20Mb correspondantly,
because compression ratio can be big enough you may store more data in sessions compared to case when you do not use session compression.
Note however that store huge amount of data in session storage usually considered as bad practice - try to make sure you keep your session data size relatively small.

You can set the value using the "compress" attribute of the provider entry.

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" compress="true" />
      </providers>
    </sessionState>

It is also possible to select compression algorithm (starting from version 1.3.1): GZip ('gzip'), LZ4 ('lz4'), QuickLZ ('quicklz') are currently supported.
You can change it using 'compressionType' attribute of the provider entry.

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" compress="true" compressionType="quicklz" />
      </providers>
    </sessionState>

Starting from version 1.3.2 logging (using NLog) can be enabled (logging disabled by default):

    <sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" logging="true" />
      </providers>
    </sessionState>

Please make sure you create 'Logs' folder in the root folder of your Web application. Logs by default will be stored in file 'CouchbaseSessionStateProvider.logs' in that folder. If you want to change logs location, you can do following:

    <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" logging="true" loggingFilename="Logs/YourCouchbaseSessionStateProviderLogFile.log" /> 
    
If your application already uses and configure NLog and you don't want to override your application settings, use following configuration:

    <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" logging="true" useExistedLoggingConfig="true" /> 
    
Note that currently, code-based configuration of the CouchbaseClient is not supported.

In code, simply use the Session object as you normally would.

    Session["Message"] = "Couchbase is awesome!";

Be sure to mark any user defined types as Serializable.

    [Serializable]
    public class SessionUser 
    {
        public string Username { get; set; }

        public string Email { get; set; }
    }

## Configuring the OutputCache provider

Update the outputCache section in Web.config as follows:

    <outputCache defaultProvider="CouchbaseCache">
        <providers>
            <add name="CouchbaseCache" type="Couchbase.AspNet.OutputCache.CouchbaseOutputCacheProvider, Couchbase.AspNet" section="couchbase-caching"/>
        </providers>
    </outputCache>

Configure the Couchbase Client as you normally would:

    <section name="couchbase" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase"/>

    <couchbase>
        <servers bucket="default" bucketPassword="">
            <add uri="http://127.0.0.1:8091/pools"/>
        </servers>
    </couchbase>

If you would like to use a custom configuration section, you may do so by specifying a value for the "section" attribute of the provider entry (see below).

    <section name="couchbaseSession" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase"/>

    <couchbaseSession>
        <servers bucket="sessionState" bucketPassword="">
            <add uri="http://127.0.0.1:8091/pools"/>
        </servers>
    </couchbaseSession>

    <outputCache defaultProvider="CouchbaseCache">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" section="couchbaseSession" />
      </providers>
    </outputCache>

Once configured, simply enable output cache as you already do with ASP.NET MVC

    [OutputCache(Duration = 60, VaryByParam="foo")]
    public ActionResult Time(string foo)
    {
        return Content(DateTime.Now.ToString());
    }

or with ASP.NET WebForms

    <%@ OutputCache Duration="60" VaryByParam="foo" %>


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/evereq/couchbaseaspnetextended.nuget/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

## Configuring multiple providers or direct use of Couchbase API.

Current version of Couchbase providers require that each of them work with separate bucket, because each use own instance of Couchbase client and multiple clients can't work with the same bucket
(see http://docs.couchbase.com/couchbase-sdk-net-1.3/index.html#configuring-the-net-client-library for more info)

So in order to use multiple providers (say Cache and Sessions) or use one (or more) providers and direct Couchbase API, you should configure multiple buckets (e.g. "sessions", "cache", "data" etc) in the Couchbase Server and in your application config file.
For example, in your Web.config you may have following config:

    <configSections>
     
	 ....
    
		<sectionGroup name="couchbase">
		  <section name="couchbase-sessions" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase" />
		  <section name="couchbase-cache" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase" />
		  <section name="couchbase-data" type="Couchbase.Configuration.CouchbaseClientSection, Couchbase" />
		</sectionGroup>

	 ....
		
    </configSections>

    <couchbase>
		<couchbase-sessions>
		  <servers bucket="sessions" bucketPassword="">
			<add uri="http://127.0.0.1:8091/pools" />      
		  </servers>    
		</couchbase-sessions>
		<couchbase-cache>
		  <servers bucket="cache" bucketPassword="">
			<add uri="http://127.0.0.1:8091/pools" />      
		  </servers>    
		</couchbase-cache>    
		<couchbase-data>
		  <servers bucket="data" bucketPassword="">
			<add uri="http://127.0.0.1:8091/pools" />      
		  </servers>    
		</couchbase-data>    
    </couchbase>
	
	....
	
	<sessionState customProvider="Couchbase" mode="Custom">
      <providers>
        <add name="Couchbase" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" section="couchbase/couchbase-sessions" />        
      </providers>
    </sessionState>

    <outputCache defaultProvider="CouchbaseCache">
      <providers>
        <add name="CouchbaseCache" type="Couchbase.AspNet.SessionState.CouchbaseSessionStateProvider, Couchbase.AspNet" section="couchbase/couchbase-cache" />
      </providers>
    </outputCache>
	
With this in mind, if you need to use Couchbase API to handle your application data, your manager code could be something like below (it use bucket 'data' from 'couchbase-data' section in Web.config)

    /// <summary>
    /// Manager of single Couchbase client instance for all Data operations
    /// </summary>
    public static class CouchbaseManager
    {
        private readonly static CouchbaseClient _instance;

        static CouchbaseManager()
        {
            var cacheSection = (CouchbaseClientSection)ConfigurationManager.GetSection("couchbase/couchbase-data");
            _instance = new CouchbaseClient(cacheSection);
        }

        public static CouchbaseClient Instance { get { return _instance; } }
    }	
