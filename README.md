[![Build status](https://ci.appveyor.com/api/projects/status/7n41lpcl451xjlat?svg=true)](https://ci.appveyor.com/project/garfieldos/aspnetcore-ipfiltering)

# AspNetCore.IpFiltering

A midleware that allows whitelist or blacklist (Ip filtering) incoming requests. 

It supports: 
* single IP
* IP range IPv4 and IPv6
* wildcard (*)
* configurable caching

Configuration of whitelist and blacklist addresses can be made by: asp.net Core configuration system or by implementing custom `IIpRulesProvider`.

## Get in on NuGet
```
Install-Package AspNetCore.IpFiltering
```

## Usage

## Appsetting based configuration

### Startup.cs file:
```
public class Startup
    {
        // ....

        public void ConfigureServices(IServiceCollection services)
        {
            // ...

            services.AddIpFiltering(_configuration.GetSection("IpFilteringConfiguration"));
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            // ...

            app.UseIpFilteringMiddleware();
            app.UseMvc();
        }
    }
```

### appsettings.yml file:
```
{
    "IpFilteringConfiguration" : {
        "Whitelist": ["*"],
        "Blacklist": [""],
        "IpRulesSource": "Configuration",
        "IpRulesCacheSource" : "Configuration",
        "DefaultIpRuleCacheDuration" : "300",
        "FailureHttpStatusCode": "403",
        "FailureMessage" : "IP address rejected"
    }
}
```
## Custom provider based configuration 

### Startup.cs file:
```
public class Startup
    {
        // ....

        public void ConfigureServices(IServiceCollection services)
        {
            // ...
            
            services.AddTransient<IIpRulesProvider, InMemoryListRulesProvider>();
            services.AddIpFiltering(new WhitelistOptions
            {
                IpListSource = IpListSource.Provider,
                FailureHttpStatusCode = 404
            });
        }

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            // ...

            app.UseIpFilteringMiddleware();
            app.UseMvc();
        }
    }

```

### InMemoryListRulesProvider class:

```
    public class InMemoryListRulesProvider : IIpRulesProvider
    {
        public Task<IpRule[]> GetIpRules()
        {
            return Task.FromResult(new List<IpRule>()
            {
                // blacklist
                new IpRule(IpAddressRangeWithWildcard.Parse("127.0.0.4"),
                    IpRuleType.Blacklist),
                new IpRule(IpAddressRangeWithWildcard.Parse("127.0.0.4"),
                    IpRuleType.Blacklist),
                
                // whitelist
                new IpRule(IpAddressRangeWithWildcard.GetWildcardRange(),IpRuleType.Whitelist)
            }.ToArray());
        }
    }
```

In real implementation you would make a real db call instead of returning static list.

## More samples can be found here: 
[https://github.com/garfieldos/AspNetCore.IpFiltering/tree/master/src/samples](https://github.com/garfieldos/AspNetCore.IpFiltering/tree/master/src/samples)
