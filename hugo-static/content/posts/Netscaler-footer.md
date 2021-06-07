---
title: "Add custom text to Citrix Gateway Login Page RfWebUI"
date: 2021-06-07T17:56:14+02:00
draft: false
---

Before using nFactor it was easy to add some footer with Informations and Warnings using Rewrite Policy or editing the index.html/tmindex.html to Citrix Gateway or an Authentication vserver. As soon as we are applying an AuthenticationProfile to a Citrix vserver or using Advanced Authentication Policies for an Authentication vserver this does not work anymore. Now this can be achived by creating a custom portal theme and a custom login schema.

The output of this example will look like this:

![Output Example](/devmaryb/media/footer-0.png)

First we need to create a custom Portal Theme derived from RfWebUI. We can do some customizations here like images and background images and colors or we can simply leave it untouched. We'll need it later for the customization of our footer.

![Create Portal Theme](/devmaryb/media/footer-1.png)

Now we create a custom LoginSchema which will hold the place for our custom text after the login Button.

```
<?xml version="1.0" encoding="utf-8"?>
<AuthenticateResponse xmlns="http://citrix.com/authentication/response/1">
<Status>success</Status>
<Result>more-info</Result>
<StateContext/>
<AuthenticationRequirements>
<PostBack>/nf/auth/doAuthentication.do</PostBack>
<CancelPostBack>/Citrix/Authentication/ExplicitForms/CancelAuthenticate</CancelPostBack>
<CancelButtonText>Cancel</CancelButtonText>
<Requirements>
<Requirement>
<Credential>
<ID>login</ID>
<SaveID>Username</SaveID>
<Type>username</Type>
</Credential>
<Label>
<Text>User name</Text>
<Type>plain</Type>
</Label>
<Input>
<Text>
<Secret>false</Secret>
<ReadOnly>false</ReadOnly>
<InitialValue></InitialValue>
<Constraint>.+</Constraint>
</Text>
</Input>
</Requirement>
<Requirement>
<Credential>
<ID>passwd</ID>
<SaveID>Password</SaveID>
<Type>password</Type>
</Credential>
<Label>
<Text>Password:</Text>
<Type>plain</Type>
</Label>
<Input>
<Text>
<Secret>true</Secret>
<ReadOnly>false</ReadOnly>
<InitialValue/>
<Constraint>.+</Constraint>
</Text>
</Input>
</Requirement>
<Requirement>
<Credential>
<ID>loginBtn</ID>
<Type>none</Type>
</Credential>
<Label>
<Type>none</Type>
</Label>
<Input>
<Button>Log On</Button>
</Input>
</Requirement>
<Requirement>
<Credential>
<Type>nsg-custom-cred</Type>
<ID>passwd</ID>
</Credential>
<Label>
<Type>nsg-custom-label</Type>
</Label>
</Requirement>
</Requirements>
</AuthenticationRequirements>
</AuthenticateResponse>
```

Basically this is the LoginSchema for a Username and a Password and we added another Requirement Tag with the Type nsg-custom-cred after the login Button.

I normally create a .xml File in /nsconfig/loginschema for this new LoginSchema. Here I name it SingleAuth-footer3.xml.

Now we will create a Login Schema Profile with the newly created SingleAuth-footer3.xml. The GUI will give us an error that it cannot read property 'type' of undefined. We will ignore this and select the XML anyway.

![Create Login Schema Profile](/devmaryb/media/footer-3.png)

Now lets create a matching Login Schema Policy for this profile.

![Create Login Schema Policy](/devmaryb/media/footer-3.png)

We will bind the newly created portal theme and the LoginSchema to the Authentication vserver that is used in our Authentication Profile for the Citrix Gateway since we are using nFactor this should already be in place.

![Bind LoginSchema](/devmaryb/media/footer-4.png)

![Set PortalTheme](/devmaryb/media/footer-5.png)

For the portal theme set it on the authentication vserver as well as on the vpn/citrix vserver.

The only thing that is missing now is the custom text we would like to see at the bottom of the LoginPage.
This needs to be added in the script.js of the portal theme.
```
/var/netscaler/logon/themes/RfWebUI-footer
```
Add at the bottom of the existing script.js the following:
```
// Custom Label Handler for Self Service Links
CTXS.ExtensionAPI.addCustomAuthLabelHandler({
getLabelTypeName: function () { return "nsg-custom-label"; },
getLabelTypeMarkup: function (requirements) {
return $("<div style=\"color: white; font-size: 30px;\"> This is my personal text in case of any problems call the helpdesk at +1234567890</div>");
},
// Instruction to parse the label as if it was a standard type
parseAsType: function () {
return "plain";
}
});
//Custom Credential Handler for Self Service Links
CTXS.ExtensionAPI.addCustomCredentialHandler({
getCredentialTypeName: function () { return "nsg-custom-cred"; },
getCredentialTypeMarkup: function (requirements) {
return $("<div/>");
},
});
```

References:

<https://docs.citrix.com/en-us/citrix-adc/current-release/aaa-tm/authentication-methods/multi-factor-nfactor-authentication/nfactor-extensibility.html>

<https://www.mycugc.org/blogs/cugc-blogs/2016/12/15/adding-text-links-and-other-elements-to-the-netsca>
