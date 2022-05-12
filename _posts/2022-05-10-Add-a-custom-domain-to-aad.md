---
layout: single
title:  "Add a Custom Domain Name to Azure Active Directory"
date:   2022-05-10 11:00:00 +0000
categories: MEM M365 DEV
permalink: aad-customdomainname
tags:
  - Azure Active Directory
  - Custom Domain Name
toc: true
---
## Why add a custom domain name?
When you first set up your Azure account, your Azure Active Directory tenant comes with an initial domain name: *domainname.onmicrosoft.com.* You are free to use this domain name, but it means that your account usernames become quite long, such as *username.domainname.onmicrosoft.com*.

To better represent your organisation, you can add a custom domain name. This allows you to create user names that are familiar to your users, such as *name@organisation.com*.

## Prerequisites
Before you begin, you need to have registered a domain with an [ICANN-Accredited registrar.](https://www.icann.org/registrar-reports/accredited-list.html)

I use [Google Domains](https://domains.google.com/), but I've also used [Ionos](https://www.ionos.co.uk/) before.

## Add your custom domain name to Azure AD
Once you have acquired the domain which you wish to use, you can add your custom domain name.

1. Sign in to the [Azure portal](https://portal.azure.com) with an account that is the Global administrator for the directory.
2. Search for and select *Azure Active Directory* from any page. Then under **Manage**, select **Custom domain names.**
3. From the top of the page select **Add custom domain**.
    
    ![Navigate.gif](/assets/images/aadcdn/Navigate.gif)
    
4. In the **Custom domain name** field, enter your registered domain name. In this example, *cloudmgmt.co.uk*. Select **Add domain**.
    
    ![addcustomdomain.gif](/assets/images/aadcdn/addcustomdomain.gif)
    

You must use a top level domain extension for this to work properly.
{: .notice--warning}
    
5. The domain is now added but unverified. To verify the domain you will need to configure DNS with a TXT record, via the domain registrar, where you purchased your domain.

## Add your DNS information to the domain registrar

Creating a TXT record with your domain registrar verifies ownership of your domain name.

1. Go to your domain registrar.
2. Create a new TXT record for your domain based on the DNS information provided when you added your custom domain to Azure Active Directory.

    ![DNS.png](/assets/images/aadcdn/DNS.png)

3. Wait at least an hour for the DNS updates to propagate.

## Verify your custom domain in Azure

The propagation from your domain registrar to Azure AD can be instantaneous or it can take a few days.

To verify your custom domain name, follow these steps:

1. Sign in to the Azure portal using an account which is a Global administrator for the directory.
2. Search for and select *Azure Active Directory* from any page. Then under **Manage**, select **Custom domain names.**
3. In **Custom domain names**, you will see your newly added domain name. It should appear showing a status of *unverified*. Select the unverified custom domain.
    
    ![unverified.png](/assets/images/aadcdn/unverified.png)
    
4. On the domain page, select the **Verify**.
    
    ![verify.gif](/assets/images/aadcdn/verify.gif)
    
5. Navigate back to the **Custom domain names** page and you will see your custom domain now has a status of *verified*.
    
    ![verified.png](/assets/images/aadcdn/verified.png)
    
6. To make the custom domain your primary domain, select the custom domain, and select *Make primary*.
    
    ![primary.gif](/assets/images/aadcdn/primary.gif)
    

## Creating a new user

Your custom domain name is now set up. When you go to create a new user in Azure Active Directory, your newly added domain will be available to select as a domain.

   ![newuser.png](/assets/images/aadcdn/newuser.png)