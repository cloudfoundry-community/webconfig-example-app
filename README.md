# webconfig-example-app
Example app for web config transform buildpack. For more detailed instructions please see []

## How to use:
*This repository includes solution files specific to Visual Studio. Once downloaded, cd into the web project folder () to run the below commands as is
### 1. Create a service for Spring Cloud Config Server

1. Make sure you have config server available in your CF marketplace. 
    > NOTE: To check if you have this server, run `cf marketplace`. You should see `p.config-server` or `p-config-server` in this list. 

1. Create config server using the CF cli. This will target an example config server repository found [here](https://github.com/mvalliath/webconfig-example-externalfiles)
    ```script
    cf create-service p-config-server standard my_configserver  -c .\config-server.json
    ```
### 2. Push the sample app to CF
1. Publish web project to bin/release/publish (we push this folder contents to CF)

1. Push app to CF
   ```script
   cf push --var env=Development
   ```
1. Observe the logs. web-configtranform buildpack emits logs when it replaces settings. 
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


### 3. How configuration settings externalized in git repository
1. Below settings were extranalized based on environment
   * appSettings - Setting1 and CommonSetting  
   * connectionStrings - MyDB
   * serviceModel - address, binding and bindingConfiguration
   
 1. `CommonSetting` ia common across all environments. So have that setting in [sampleapp.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp.yml).
 1. `Setting1`, `MyDB` and `serviceModel` settings differ by enevironment. So have them in environment specific files.
    * [sampleapp-Development.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp-Development.yml)  
    * [sampleapp-Production.yml](https://github.com/mvalliath/webconfig-example-externalfiles/blob/master/sampleapp-Production.yml)
    
    > Note: file names should match to your app name. In this example filenames start with `sampleapp` since we specified application name as `sampleapp` in manifest.yml
 
 
   
