# Newsletter for EPiServer #
A free module for sending newsletters from your EPiServer CMS or Commerce site. You can send the content from pages based on one or more newsletter pagetypes to multiple recipients, using a cloud service or your own SMTP server.

>This is a port of the old EPiSendMail (BV Network Newsletter) module from EPiCode, rewritten to support EPiServer CMS and Commerce 7.5 and newer (including CMS 8).

## Features ##
 * Create newsletters as pages in EPiServer
 * Uses recipient lists to hold your email addresses
 * Example templates with responsive email design using Zurb Ink (you can use any framework, or hand code your email markup from scratch).
 * Will inline CSS to help make the email look correct in all (major) email clients
 * Configurable sender system (supports SMTP Server, pick up directory, [Mailgun](http://www.mailgun.com) and [SendGrid](http://sendgrid.com)
 * Send newsletters to contacts in Commerce using filtered contact lists

### Additional Features when using Mailgun or SendGrid ###
 * Supports large amount of emails
 * Low cost per email 
	 * Mailgun: first 10.000 emails per month are free
	 * SendGrid: first 12.000 emails per month are free (limited to 400 per day)
 * Supports tracking of delivery, open and clicks (using Mailgun and SendGrid) 
 * Supports the Mailgun campaign feature to get a better overview of your newsletters
 * Variable substitution in the email markup (for adding the recipient email address to unsubscribe links for an example).

## Downloads ##
The installation is done through Visual Studio using the EPiServer Nuget Feed on http://nuget.episerver.com/ 

Main Newsletter Add-on:
```
Install-Package EPiCode.Newsletter
```

Send using SendGrid:
```
Install-Package EPiCode.Newsletter.SendGrid
```

Commerce Integration:
```
Install-Package EPiCode.Newsletter.Commerce
```

MVC Examples for Alloy:
```
Install-Package EPiCode.Newsletter.Examples
```
> **Note!** The examples package has requirements to code and configuration in the Alloy MVC sample. It will most likely fail to compile if you install it in a custom built site. It also uses Bootstrap for the CSS.
   
## Example Newsletter Designs ##
In addition to the Newsletter module itself, you also need a layout and design for the newsletter, that works across as many Email clients as possible. The MVC Examples for Alloy has two example newsletters based on the [Zurb Ink](http://zurb.com/ink/) framework.

**Note!** The unsubscribe link in the example templates uses variable substitution, and requires Mailgun or SendGrid.

## Creating Your Own Newsletter Design ##
The module will send a page in EPiServer CMS if it inherits the `NewsletterBase` base class. You should add additional properties to your own page (inheriting the NewsletterBase class) that you will use as content in your newsletter.

**Note!** If you use content areas in your newsletter, and add pages and blocks to it, make sure you have special renderers (using a tag), since the default page and block renderers in your project will probably not look good in an email. You might also want to restrict what types of content the editor is allowed to add to the content area, to prevent adding content that will not render correctly in the email.

You can have multiple newsletter pagetypes for different designs, just make sure you inherit the `NewsletterBase` base class in order to show the editor view for it.

## Installation ##
1. Select the [EPiServer Nuget Feed](http://nuget.episerver.com/feed/packages.svc/) in Visual Studio Nuget Package Manager Dialog or the Packet Manger Console
1. Install the EPiCode.Newsletter or EPiCode.Newsletter.Examples (for Alloy) package
1. Recompile your site

## Configuration ##
The module supports sending emails through different mail senders (also called a ''sender engine''):
1. Recommended: Mailgun (http://mailgun.com) or SendGrid (http://sendgrid.com)
1. Built-in SMTP support (default)
1. aspnetEmail component from [AdvancedIntellect](http://www.advancedintellect.com/product.aspx?smtp) (license purchase required). **Note!** This sender is not actively supported.

**Note!** We recommend using a cloud sender (Mailgun or SendGrid), as it has the most features and offloads the sending process to the cloud.

The aspnetEmail sender engine is included for backwards compatibility, but will eventually be removed from the module.

### Configuring SMTP ###
When installing the module, the default sender is the built-in SMTP client in .NET. In order to send emails using SMTP, you need to configure the standard `mailSettings` in web.config:

Example using smtp server. There are more settings you can configure for password protected servers, ssl support etc.
```xml
<system.net>
  <mailSettings>
    <smtp deliveryMethod="Network" from="noreply@my.domain.com">
      <network host="your.smtp.server"/>
    </smtp>
  </mailSettings>
</system.net>
```

You can also use a pick up directory if you have a SMTP server on your network that can access the pickup directory. The pickup directory delivery is preferable to network, as sending emails goes a lot faster, and you are less likely to get timeouts in the scheduled job.
```xml
<system.net>
  <mailSettings>
    <smtp deliveryMethod="SpecifiedPickupDirectory" from="noreply@my.domain.com">
      <specifiedPickupDirectory pickupDirectoryLocation="c:\temp\email-pickup-directory" />
    </smtp>        
  </mailSettings>
</system.net>
```
**Note!** The `pickupDirectoryLocation` can be on a network share.

### Configuring Mailgun ###
Add the following keys to the `<appSettings>` section in web.config to use Mailgun:
```xml
<add key="Newsletter.SenderType" value="BVNetwork.EPiSendMail.Library.MailSenderMailgun, BVNetwork.EPiSendMail" />
<add key="Mailgun.ApiKey"    value="your-mailgun-api-key-here" />
<add key="Mailgun.Domain"    value="your-mailgun-domain-here" />
<add key="Mailgun.PublicKey" value="your-mailgun-public-key-here" />
```

The Api keys can be found on the Mailgun Account home page: https://mailgun.com/cp

You also need to configure your mail sending domain for your Mailgun account, which is well described on your Mailgun account page. For testing purposes, you can use the sandbox domain that is created when you register your Mailgun account.

### Configuring SendGrid ###
Add the following keys to the `<appSettings>` section in web.config to use Mailgun:
```xml
<add key="Newsletter.SenderType" value="BVNetwork.EPiSendMail.SendGrid.MailSenderSendGrid, BVNetwork.EPiSendMail.SendGrid" />
```

Add a new connectionString to configure your SendGrid account:
```xml
<connectionStrings>
  ...
  <add name="EPiCode.Newsletter.SendGrid" connectionString="username=sendgridusername;password=sendgridpassword" providerName="Custom" />
</connectionStrings>
```
**Note!** You can create additional SendGrid users that only has access to the API, and not the full administration interface. It is recommended to create a separate account in order to authorize access to the SendGrid API.

## Troubleshooting ##
If you get an error during startup like this: `Cannot add duplicate collection entry of type 'add' with unique key attribute 'name' set to 'ExtensionlessUrlHandler-Integrated-4.0'` please check your web.config and remove any duplicate `ExtensionlessUrlHandler-Integrated-4.0` handlers (under `system.webServer`)

### Missing Assembly Redirects ###
In some cases, there are missing assembly redirects for some of the `System.Net` and `System.Web` assemblies. Make sure you have these in your web.config:
```xml
<assemblyBinding>
  ...
  <dependentAssembly>
    <assemblyIdentity name="System.Web.Http" publicKeyToken="31bf3856ad364e35" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-5.1.0.0" newVersion="5.1.0.0" />
  </dependentAssembly>
  <dependentAssembly>
    <assemblyIdentity name="System.Net.Http.Formatting" publicKeyToken="31bf3856ad364e35" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-5.1.0.0" newVersion="5.1.0.0" />
  </dependentAssembly>
  <dependentAssembly>
    <assemblyIdentity name="System.Web.Http.WebHost" publicKeyToken="31bf3856ad364e35" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-5.1.0.0" newVersion="5.1.0.0" />
  </dependentAssembly>
  <dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="b03f5f7f11d50a3a" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.0.0.0" newVersion="4.0.0.0" />
  </dependentAssembly>
</assemblyBinding>

```
'''Important! ''' You might have never versions of these referenced in your project. Make sure you check the actual versions of the assemblies before you add the redirects.

## Support ##
This module is open source and unsupported. Feel free to [register new issues](https://github.com/BVNetwork/Newsletter/issues/new) though.

## Requirements ##
Runtime:
* EPiServer CMS 7.9 or newer
* .NET 4.5

## Dependencies ##
* PreMailer.Net (1.2.6 or newer)
* Microsoft.AspNet.WebApi (5.1.2 or newer)
* RestSharp (104.4.0 or newer)

## Supports ##
* EPiServer Commerce 7.5 or newer for custom recipient lists

## Contributions by ##
BV Network AS
Departementenes sikkerhets- og serviceorganisasjon (DSS)