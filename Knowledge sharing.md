We have collected top 3 common/hot issues and arranged in each open source language. We hope these issues will quickly slove or give you quick hits about your facing issues.

- PHP
   + **Q**: What PHP configurations we can modify in Azure App Service Web Apps?    
     **A**: We can change the built-in PHP version from 5.4(default), 5.5, 5.6, and 7.0(preview).  
            We can change the built-in PHP_INI_USER, PHP_INI_PERDIR, PHP_INI_ALL and, PHP_INI_SYSTEM configurations.  
            We can enable extensions in the default PHP runtime.  
            We can even use a custom PHP runtime.  
            We also can enable Composer automation in Azure for manage your PHP packages.  
            Please refer to the offical guide, [Configure PHP in Azure App Service Web Apps](https://azure.microsoft.com/en-us/documentation/articles/web-sites-php-configure/).

      - - -
   + **Q**: How to enable connection to Azure SQL Database in PHP applications on Azure Web Apps?  
     **A**: Be default, the DLL libraries `php_sqlsrv` and `php_pdo_sqlsrv` have been installed in the PHP ext folder on Azure, but haven't been configured in PHP runtime. So we need to configure the extensions manually:
     1. Create a file named `.user.ini` at the root directory of your application on Azure.
     2. Write the following content in `.user.ini`:
         ```
        extension=php_pdo_sqlsrv.dll
        extension=php_sqlsrv.dll
         ```
     3. Restart your application. Check the extension of `sqlsrv` and `pdo_sqlsrv` again.
     
      - - -
    + **Q**: How to generate SAS token of Azure Blob Storage in PHP in version **2015-04-05**?  
      **A**: The code snippet:
      ```php
        function getSASForBlob($accountName,$containerName, $fileName, $resourceType, $permissions, $expiry) {
           /* Create the signature */ 
            $_arraysign = array();
            $_arraysign[] = $permissions;
            $_arraysign[] = '';
            $_arraysign[] = $expiry;
            $_arraysign[] = '/blob/' . $accountName . '/' . $container . '/' . $blob;
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_arraysign[] = "2015-04-05"; //the API version is now required
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_arraysign[] = '';
            $_str2sign = implode("\n", $_arraysign);
            return base64_encode(
                hash_hmac('sha256', urldecode(utf8_encode($_str2sign)), base64_decode($key), true)
            );
        }
        function getBlobDownloadUrl($container,$blob,$accountName,$key){
            /* Create the signed query part */         
            $resourceType='b';
            $permissions='r';
            /*$expTime_utc = new DateTime(null, new DateTimeZone("UTC"));
            $expTime_utc->add(new DateInterval('PT1H'));
            $expiry=$expTime_utc->format('Y-m-d\TH:i:s\Z');*/
            $expiry = date('Y-m-d\TH:i:s\Z', strtotime('+1 day'));
            $_signature=getSASForBlob($accountName,$key,$container, $blob, $resourceType, $permissions, $expiry);
            $_parts = array();
            $_parts[] = (!empty($expiry))?'se=' . urlencode($expiry):'';
            $_parts[] = 'sr=' . $resourceType;
            $_parts[] = (!empty($permissions))?'sp=' . $permissions:'';
            $_parts[] = 'sig=' . urlencode($_signature);
            $_parts[] = "sv=2015-04-05";
            /* Create the signed blob URL */
            $_url = 'https://'
            .$accountName.'.blob.core.windows.net/'
            . $container . '/'
            . $blob . '?'
            . implode('&', $_parts);x
            return $_url;
        }
        ```
        Refer to [Constructing a Service SAS](https://msdn.microsoft.com/en-us/library/azure/dn140255.aspx) for more.
        
     - - -

- Java
      + **Q**: How to generate the authorization header for using REST APIs of Azure Blob Storage.  
        **A**: The details of authentication for the Azure Storage Services shows at https://msdn.microsoft.com/en-us/library/azure/dd179428.aspx, please see the code snippet using Java below.
        ```Java
        private static Base64 base64 = new Base64();  
        private String createAuthorizationHeader(String canonicalizedString)     {  
            Mac mac = Mac.getInstance("HmacSHA256");  
            mac.init(new SecretKeySpec(base64.decode(key), "HmacSHA256"));  
            String authKey = new String(base64.encode(mac.doFinal(canonicalizedString.getBytes("UTF-8"))));  
            String authStr = "SharedKey " + account + ":" + authKey;  
            return authStr;  
        }  
        
        String urlPath = containerName+"/"+ blobName;  
        String storageServiceVersion = "2009-09-19";  

        SimpleDateFormat fmt = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss");  
        fmt.setTimeZone(TimeZone.getTimeZone("GMT"));  
        String date = fmt.format(Calendar.getInstance().getTime()) + " GMT";  
        String blobType = "BlockBlob"; //This is important as there are two types of blob, Page blob and Block blob  

        String canonicalizedHeaders = "x-ms-blob-type:"+blobType+"\nx-ms-date:"+date+"\nx-ms-version:"+storageServiceVersion;  
        String canonicalizedResource = "/"+account+"/"+urlPath;  

        //This will change based on the Authentication scheme you are using. This is for SharedKey, you need to change it if you are using SharedKeyLite
        String stringToSign = requestMethod+"\n\n\n"+blobLength+"\n\n\n\n\n\n\n\n\n"+canonicalizedHeaders+"\n"+canonicalizedResource; 
        String authorizationHeader = createAuthorizationHeader(stringToSign);  
        ```
        
     - - -
     + **Q**: How to authenticate with ARM & ASM using Azure SDK for Java.  
       **A**: Please see the codes below.
       
       Code 1. Authentication with ARM
       ```Java
        // The parameters include clientId, clientSecret, tenantId, subscriptionId and resourceGroupName.
        private static final String clientId = "<client-id>";
        private static final String clientSecret = "<key>";
        private static final String tenantId = "<tenant-id>";
        private static final String subscriptionId = "<subscription-id>";
        private static final String resouceGroupName = "<resource-group-name>";

        // The function for getting the access token via Class AuthenticationResult
        private static AuthenticationResult getAccessTokenFromServicePrincipalCredentials()
                throws ServiceUnavailableException, MalformedURLException, ExecutionException, InterruptedException {
            AuthenticationContext context;
            AuthenticationResult result = null;
            ExecutorService service = null;
            try {
                service = Executors.newFixedThreadPool(1);
                // TODO: add your tenant id
                context = new AuthenticationContext("https://login.windows.net/" + tenantId, false, service);
                // TODO: add your client id and client secret
                ClientCredential cred = new ClientCredential(clientId, clientSecret);
                Future<AuthenticationResult> future = context.acquireToken("https://management.azure.com/", cred, null);
                result = future.get();
            } finally {
                service.shutdown();
            }

            if (result == null) {
                throw new ServiceUnavailableException("authentication result was null");
            }
            return result;
        }

        // The process for getting the list of VMs in a resource group
        Configuration config = ManagementConfiguration.configure(null, 
            new URI("https://management.core.windows.net"),
            subscriptionId,
            getAccessTokenFromServicePrincipalCredentials().getAccessToken());
       ```
        
       Code2. Authentication with ASM
       ```Java
        static String uri = "https://management.core.windows.net/";
        static String subscriptionId = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";
        static String keyStoreLocation = "<cert-file>.jks";
        static String keyStorePassword = "my-cert-password";

        Configuration config = ManagementConfiguration.configure(
            new URI(uri), 
            subscriptionId,
            keyStoreLocation, // the file path to the JKS
            keyStorePassword, // the password for the JKS
            KeyStoreType.jks // flags that I'm using a JKS keystore
        );
       ```
        
     - - -
     + **Q**: How to get started with Event Hubs using Java.  
       **A**: We can get started with Event Hubs by following the three steps below.
       1. Create an Event Hub via Azure classic portal.
       2. Send messages to Event Hubs with `EventHubClient`.
       3. Receive messages from Event Hubs with `EventProcessorHost`.
       
       The classes above `EventHubClient` & `EventProcessorHost` are in the Azure SDK for Java, you can configure the `pom.xml` file of your Maven project to reference them.
       ```XML
        <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>azure-eventhubs</artifactId>
            <version>0.7.2</version>
        </dependency>
       ```
       
       As reference, please see the offical tutorial at https://azure.microsoft.com/en-us/documentation/articles/event-hubs-java-ephjava-getstarted/.
     - - -
     


- Python
     + **Q**: How to get started a web app in Python on Azure Web Apps.  
      **A**: Generally, you can use [Python Tools for Visual Studio](https://azure.microsoft.com/en-us/documentation/articles/web-sites-python-ptvs-django-mysql/PTVS) to create a simple polls web app using one of the PTVS sample templates.  
            Then build a MySQL service hosted on Azure, and configure the web app to this MySQL.  
            Then publish the web app to [Azure App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714) via Visual Studio.  
            Please refer to [Django and MySQL on Azure with Python Tools 2.2 for Visual Studio](https://azure.microsoft.com/en-us/documentation/articles/web-sites-python-ptvs-django-mysql/) for details.

       - - -
     + **Q**: How to implement Azure IOT on Raspberry Pi.  
      **A**: We suggest that you can try to extend Python with Azure IoT SDK for C, please see: https://azure.microsoft.com/en-us/documentation/articles/iot-hub-device-sdk-c-intro/ and https://docs.python.org/2/extending/extending.html.

        The other choice is that using the Azure IoT SDK for NodeJS to create a server as proxy for listening Python push messages and forwarding to Azure IoTHub, please see https://github.com/Azure/azure-iot-sdks/tree/master/node/device. And according to the version of your Respberry Pi, you need to download the suitable nodejs runtime as below from the nodejs offical website https://nodejs.org/en/download/ or via using sudo apt-get install nodejs on the Raspbian OS.  
       * Respberry Pi: ARMv6   
       * Respberry Pi 2: ARMv7  
      
       Otherwise, the simple way for sending messages from device to Azure IoTHub on Respberry PI is that using the Device Messaging REST APIs in Python.
        - - -
     + **Q**: AttributeError: `module` object has no attribute `BlobService`.  
       **A**: As in the latest `azure.storage 0.30.0` , `BlobSrvice` is split into `BlockBlobService`, `AppendBlobService`, `PageBlobService` object, you could use `BlockBlobService` insteading of `BlobService`. There are many articles need to update the content.  
     If you insist on perferring `BlobService`, you could install package `azure.storage 0.20.0`.
     
    - - -

- Node.js
  + **Q**: How to build/deploy/configure a web app in Node.js on Azure Web Apps.  
    **A**: Please refer to [Get started with Node.js web apps in Azure App Service](https://azure.microsoft.com/en-us/documentation/articles/app-service-web-nodejs-get-started/).  
    Additionally, Azure Web Apps uses IISNode to host the Node process inside of IIS. Your Node site is actually given a Named Pipe which receives the incoming requests, not a TCP port like you would use when running locally or hosting yourself. Even if you could open a TCP port, Azure Websites are really only meant for traditional websites and don't have a way to open ports to the outside world. 

    - - -
  + **Q**: How to use `tedious` to connection to Azure SQL.  
    **A**: Use following code snippet:  
    ```javascript
    var Connection = require('tedious').Connection;
    var config = {
        userName: '<yourusername>',
        password: '<yourpassword>',
        server: '<yourserver>.database.windows.net',
        // When you connect to Azure SQL Database, you need these next options.
        options: {encrypt: true, database: '<database>'}
    };
    var connection = new Connection(config);
    connection.on('connect', function(err) {
        // If no error, then good to proceed.
        console.log("Connected");
    });
    ```  
    
    - - -
  + **Q**: Azure webapp node.js app not starting  
    **A**: Azure WebApp for node.js is running via iisnode as a native IIS module that allows hosting in IIS on Windows. Please see the document https://blogs.msdn.microsoft.com/silverlining/2012/06/14/windows-azure-websites-node-js/ to know its features.  
    For debugging the webapp for node.js, I suggest you can refer to the document to know how to do it.  
    If you are using Visual Studio as the IDE for node.js, I recommend installing [NTVS](https://github.com/Microsoft/nodejstools) in VS for debugging and deploying the node.js. Please see the documents below to know how to get started.  
      [Installation for NTVS](https://github.com/Microsoft/nodejstools/wiki/Installation)  
      [Install Node.js and get started with NTVS](https://github.com/Microsoft/nodejstools/wiki/Install-Node.js-and-get-started-with-NTVS)  
    And there is an other tool called [node-inspector](https://github.com/Azure/node-inspector) for inspecting the node.js app on Azure. As reference, you can refer to the doc http://www.ranjithr.com/?p=98.  
    Meanwhile, please check the web.config file in your node webapp via Kudo Console, you can compare your codes with the sample generated by the template for Express from Gallery on Azure portal.
     
  - - -
    
- Rudy 
     + **Q**: How to generate the authorization header for using REST APIs of Azure Blob Storage.  
       **A**:
       
     - - -
     + **Q**: How to generate the authorization header for using REST APIs of Azure Blob Storage.  
       **A**:
        
     - - -
     + **Q**: How to generate the authorization header for using REST APIs of Azure Blob Storage.  
       **A**:
        
     - - -
