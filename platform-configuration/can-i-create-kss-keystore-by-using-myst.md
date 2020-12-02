Yes, KSS keystore are created using MyST Studio.

Below are the instructions for creating the kss keystore

# Prerequisites

1. Create plain text files in myst [custom action](https://help.mystsoftware.com/platform-configuration/configure-myst-custom-action#creating-custom-actions-in-myst-studio)

   - name: <KSS certificate name> 
   - location:<kSS certificate name>.cer
   - content: paste the certificate

2. update the certificates names in the below code in the line **certs = ['cert1','cert2']**

   ```
   import os
   
   def myst(cfg):
       """Custom VF script to add load balancer SSL certificates to default KSS truststore"""
       # ##
       # Description
       # ##
       if 'admin.url' in cfg and 'core.fmw.admin.username' in cfg \
           and 'core.fmw.admin.password' in cfg:
           LOG.info('Connecting to Admin Server')
           connect(cfg.getProperty('core.fmw.admin.username'),
                   cfg.getProperty('core.fmw.admin.password'),
                   cfg.getProperty('admin.url'))
       LOG.info('**************************')
       LOG.info('Started adding load balancer SSL certificates to default KSS truststore')
       LOG.info('**************************')
       svc = getOpssService(name='KeyStoreService')
   
       # Prints the current certificates in trust store
       LOG.info('List of exiting certificates in the default KSS trust')
       svc.listKeyStoreAliases(appStripe='system', name='trust', password='', type='TrustedCertificate')
   
       certs = ['cert1','cert2']
       for cert in certs:
           os.system('openssl x509 -in resources/custom/'+cert+'.cer -outform PEM -out resources/custom/'+ cert+'.pem')
           try:
              svc.importKeyStoreCertificate(appStripe='system', name='trust', password='', alias=''+ cert + '', keypassword='', type='TrustedCertificate', filepath= cfg.getProperty('myst.workspace')+ '/resources/custom/'+ cert +'.pem')
           except Exception, e:
              LOG.warn(cert + ' encountered exception imported certificate')
              LOG.warn('Check if ' + cert +' certificate already exists')
              LOG.warn('Skipping import')
       
       # Prints the current certificates in trust store
   
       LOG.info('List of certificates after adding load balancer SSL certificates the default KSS trust')
       svc.listKeyStoreAliases(appStripe='system', name='trust', password='', type='TrustedCertificate')
   
       LOG.info("Performing WLST syncKeystores('system',' 'KSS')")
       syncKeyStores('system', 'KSS')
       LOG.info('**************************')
       LOG.info('Finished adding load balancer SSL certificates to default KSS truststore')
       LOG.info('**************************')
   ```

   

3. Create a `WLST Action` in the myst custom action`import-kss-certificates.py` and paste the updated code in the **content** section

# Import Certificates

Run the **import-kss-certificates** custom action on platform model 

1. Go to **Platform Models** > <Your model> > **Actions** > **control**
2.  select **custom** as Select an action to perform.
3. Now, select  **import-kss-certificates**  and click **execute**



