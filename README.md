# Azure Active Directory Single Sign-on for WordPress

A WordPress plugin that allows organizations to use their Azure Active Directory
user accounts to sign in to WordPress. Organizations with Office 365 already have
Azure Active Directory and can use this plugin for all of their users and any onsite
Active Directory linked to Azure Active Directory can also be used.

- Security group membership can be mapped to WordPress user roles.
- Standard WordPress login is still available.

In the typical flow:

1. User attempts to access the admin section of the blog (`wp-admin`). At the sign in page, they are given a link to sign in with their Azure Active Directory organization account (e.g. an Office 365 account).
2. After signing in, the user is redirected back to the blog with a JSON Web Token (JWT), containing a minimal set of claims.
3. The plugin uses these claims to attempt to find a WordPress user with the Azure AD ID that matches the Azure Active Directory user.
4. If one is found, the user is authenticated in WordPress as that user.
5. (Optional) Membership to certain groups in Azure AD can be mapped to roles in WordPress.

## Differences from upstream

The following functional changes have been made to this module to better suit our needs. You may find them useful to and thus choose this repo over its parent.

1. The login page has the Office 365 logo so that it is more obvious to users what they are doing
2. New users are registered using their 'mail' attribute instead of their 'proxyAddresses' attribute, so that they get @yourcompany.com instead of @yourcompany.onmicrosoft.com in their profile
3. Since we use it in conjunction with the Force Login plugin to create an internet-accessible but private site, the user is always redirected to the site's home page after logging in, rather than back to the page they came from. This avoids a weird condition where after logging in the user is taken back to the login page. If you aren't using Force Login this brute force approach might be too harsh for you. If so play with redirect_after_login() in aad_sso_wordpress.php

## Getting Started

The following instructions will get you started. In this case, we will be configuring the plugin to use the user roles configured in WordPress.

### 1. Download the plugin

You can do this with `git` or with the 'Download ZIP' link on the right.

Place the `aad-sso-wordpress` folder in your WordPress' plugin folder. Normally, this is `<yourblog>/wp-content/plugins`.

### 2. Register an Azure Active Directory application

For these steps, you must have an Azure subscription with access to the Azure Active Directory tenant that you would like to use with your blog.

1. Sign in to the [Azure portal](https://manage.windowsazure.com), and navigate to the ACTIVE DIRECTORY section. Choose the directory (tenant) that you would like to use. This should be the directory containing the users and (optionally) groups that will have access to your WordPress blog.
3. Under the APPLICATIONS tab, click ADD to register a new application. Choose 'Add an application my organization is developing', and a recognizable name. Choose values for sign-in URL and App ID URL. The blog's URL is usually a good choice.
4. When the app is created, under the CONFIGURE tab, generate a key and copy the secret value (it will be visible once only, after you save).
5. Add a reply URL with the format: `https://<your blog url>/wp-login.php`.

### 3. Configure the plugin

The plugin can be configured in Settings > AAD Settings.

### 4. (Optional) Set WordPress roles based on Azure AD group membership

The AADSSO plugin can be configured to set different WordPress roles based on the user's membership to a set of user-defined groups. This is a great way to control who has access to the blog, and under what role.

**You will need to enable the permissions for your Azure Active Directory application**. In the 'CONFIGURE' tab, Under the 'permissions to other applications' heading, it needs to have Delegated Permission to "Read Directory Data" and "Enable sign-on and read users' profiles".

Once your application permissions have been updated, you will need to update the plugin's configuration in Settings > AAD Settings:

- Enable role mapping must be checked.
- There are inputs for several common WordPress roles, Administrator, Editor, Author, Contributor, and Subscriber. The inputs expect the Azure Active Directory group object IDs. You can find the group object IDs under the Active Directory 'GROUPS' tab.
	- You will see a listing of any custom groups that exist for that directory.
	- Click on the one you want the ID for.
	- Click on the 'PROPERTIES' tab.
	- At the bottom, you'll see the value you need in the 'OBJECT ID' setting.
- Custom WordPress roles to Activie Directory groups can be added in the Custom role mapping box by placing one mapping per line in the format: `<wp_role> <aad_group_id>`.

### Groups membership-based roles (no default role)

Users are matched by the Azure AD login IDs in WordPress, and WordPress roles are dictated by membership to a given Azure AD group. If the user is not a part of any of these groups, they are assigned the default new user role in WordPress, or you can specify the default role in the plugin's configuration settings.
