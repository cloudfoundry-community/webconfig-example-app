# webconfig-example-app
Example app for web config transform buildpack. 

For instructions on how to use this buildpack with your own app and external configuration files, please see the buildpack repository (https://github.com/greenhouse-org/web-config-transform-buildpack)

## How to use:
*This repository includes solution files specific to Visual Studio. Once downloaded, cd into the web project folder (WebConfigSampleApp) to run the below commands as is*
### 1. Create a service for Spring Cloud Config Server

* Make sure you have config server available in your CF marketplace. 
    > NOTE: To check if you have this server, run `cf marketplace`. You should see `p.config-server` or `p-config-server` in this list. 

* Create config server using the CF cli. This will target an example config server repository found [here](https://github.com/mvalliath/webconfig-example-externalfiles)
    ```script
    cf create-service p-config-server standard my_configserver  -c .\config-server.json
    ```
### 2. Push the sample app to CF
* Publish web project to bin/release/publish (we push this folder contents to CF)

* Push app to CF. The below command will target the `Development` configuration file in the example config server repository. If you want to target another environment (e.g. `Production`), swap this value out for the `env` argument. 
   ```script
   cf push --var env=Development
   ```
   
* Observe the logs. web-configtranform buildpack emits logs when it replaces settings. 
   ```text
   Downloading hwc_buildpack...
   Downloaded hwc_buildpack
   Cell b8e4aaa7-f146-4777-8702-b3faf909b6c6 creating container for instance e543ac9a-9224-4c7f-a59a-f5357adc13d3
   Cell b8e4aaa7-f146-4777-8702-b3faf909b6c6 successfully created container for instance e543ac9a-9224-4c7f-a59a-f5357adc13d3
   Downloading app package...
   Downloading build artifacts cache...
   Downloaded build artifacts cache (237B)
   Downloaded app package (6.9M)
   Downloaded buildpack `https://github.com/greenhouse-org/web-config-transform-buildpack/releases/download/v1.1.6/Pivotal.Web.Config
   ================================================================================
   =============== WebConfig Transform Buildpack execution started ================
   ================================================================================
   -----> Using Environment: Development
   -----> Config server binding found...
   info: Steeltoe.Extensions.Configuration.ConfigServer.ConfigServerConfigurationProvider[0]
         Fetching config from server at: https://config-f7c2aba4-5dfd-41a6-b5b7-456d6372367f.apps.pcfone.io
   info: Steeltoe.Extensions.Configuration.ConfigServer.ConfigServerConfigurationProvider[0]
         Located environment: sampleapp, Development, (null), 78da03e611009eb7f8c3388b5fdf08d5807f878b, (null)
   -----> Replacing value for matching appSetting key `CommonSetting` in web.config
   -----> Replacing token `#{client:Default_IMyLogService:bindingConfiguration}` in web.config
   -----> Replacing token `#{client:Default_IMyLogService:binding}` in web.config
   -----> Replacing token `#{client:Default_IMyLogService:address}` in web.config
   ================================================================================
   ============== WebConfig Transform Buildpack execution completed ===============
   ================================================================================
   ```

### 3. Observe changes made to the web.Config file!
You can directly view the changes made to the web.Config file by ssh'ing into your app container.
```script
  cf ssh sampleapp  
```
Once ssh'd into the container, cd into the app directory and view the web.Config files. You should now see the replaced values that are defined in the configuration repository. 

``` script
cd app
type web.Config 
```

In the above example, we targeted the `Development` environment. Using this configuration file, your web.Config will now have the development environment configuration values:

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, please visit
  https://go.microsoft.com/fwlink/?LinkId=169433
  -->
<configuration>
  <appSettings>
    <add key="Setting1" value="git dev setting" />
    <add key="CommonSetting" value="common setting from git" />
  </appSettings>
  <connectionStrings>
    <add name="MyDB" connectionString="Data Source=GithubDevSQLServer;Initial Catalog=MyDB;User ID=xxxx;Password=xxxx" />
  </connectionStrings>
  <system.serviceModel>
    <client>
      <endpoint address="git_dev_endpoint" binding="git_dev_binding" bindingConfiguration="git_dev_bindingConfiguration" contract="ServiceProxy.IMyLogService" name="Default_IMyLogService" />
    </client>
  </system.serviceModel>
  <system.web>
    <compilation targetFramework="4.7.2" />
    <httpRuntime targetFramework="4.7.2" />
  </system.web>
  <system.codedom>
    <compilers>
      <compiler language="c#;cs;csharp" extension=".cs" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.CSharpCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=2.0.0.0, Culture=neutral, Pu
blicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:default /nowarn:1659;1699;1701" />
      <compiler language="vb;vbs;visualbasic;vbscript" extension=".vb" type="Microsoft.CodeDom.Providers.DotNetCompilerPlatform.VBCodeProvider, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, Version=2.0.0.0, Culture=
neutral, PublicKeyToken=31bf3856ad364e35" warningLevel="4" compilerOptions="/langversion:default /nowarn:41008 /define:_MYTYPE=\&quot;Web\&quot; /optionInfer+" />
    </compilers>
  </system.codedom>
</configuration>
```

You will see these values change if you target the Production environment. 

###  How configuration settings were externalized in git repository
* Below settings were externalized based on environment
   * appSettings - Setting1 and CommonSetting  
   * connectionStrings - MyDB
   * serviceModel - address, binding and bindingConfiguration
   
 1. `CommonSetting` ia common across all environments. So have that setting in [sampleapp.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp.yml).
 1. `Setting1`, `MyDB` and `serviceModel` settings differ by enevironment. So have them in environment specific files.
    * [sampleapp-Development.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp-Development.yml)  
    * [sampleapp-Production.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp-Production.yml)
    
    > Note: file names should match to your app name. In this example filenames start with `sampleapp` since we specified application name as `sampleapp` in manifest.yml
 
 
   
