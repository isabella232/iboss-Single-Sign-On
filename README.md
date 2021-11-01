## iboss and ForgeRock SSO SAML Integration


  The iboss cloud service can integrate with a "ForgeRock" identity provider (IdP) to authorize user access from the endpoint using client software (iboss Cloud Connector) to authenticate service access, assign security policies and log traffic.

  You can enable this at a cluster level or on a per-gateway level for multiple IdP's in a single account.

The SAML initiator mode is "SP-Initiator mode."

  This document guides the iboss admin through several common Identity Provider's (IdP) and iboss Cloud (SP) configuration steps for SAML 2.0 auth using iboss Cloud Connectors on the endpoint.

1.
    iboss Cloud Connector redirects the user to iboss (the cloud service provider
    (SP)).
2.
    iboss creates a SAML authentication request token and responds with a redirect
    to ForgeRock (the identity provider (IdP)).
3. The user follows a redirect to ForgeRock to authenticate.
4. User authenticates to ForgeRock
5.
    ForgeRock sends the SAML assertion to iboss via the Assertion Consumer Service
    (ACS) URL.
6.
     iboss validates the SAML assertion and sends the SAML session token
    to the iboss Cloud Connector
7. Session starts.

ce ![mceclip0.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/mceclip0.png)

## Licensing Requirements

Unlimited Package

## Prerequisites and Guidelines
*At least one iboss Cloud account.
*At least one Identity Provider (IdP)
*   Assumes that either the Windows or Mac Cloud Connectors are installed:
       - Windows requires the iboss Desktop App to be deployed.
* Review the [iboss PortList](https://support.ibosscloud.com/hc/en-us?articles/115007869047) article to determine the inbound and outbound ports required by the iboss cloud platform.
* Local breakout for IdP destinations

To begin the configuration, please follow the integration guide below:

## Step 1 - Acquire iboss Gateway SAML SP settings for Identity Provider

  In this step, we will acquire some service provider settings used later when
  configuring the identity provider.

1.
    Open a web browser and navigate to "[https://cloud.iboss.com](https://cloud.iboss.com/)," then sign into your iboss cloud account.
2.
    Navigate to **Users, Groups and Devices** → **User SSO** → **SAML.**

  ![usersgroups_devices.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/usersgroups&devices.png)

  From here, take note of the "SAML Entity ID" and "SP ACS URL" values for the next steps.

  ![SPsettings.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/SPsettings.png)

1.
    Navigate to **Actions** and click **Download Service Provider (SP) Metadata**as
    this will be needed in a later step.

  ![downloadspmetadata.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/downloadspmetadata.png)

## Step 2 - Configure Identity Provider Settings

  In this step, we cover the below identity provider for use with iboss cloud integration.

### Create a Circle of Trust

  The first step requires creating a ForgeRock C ircle of Trust. A circle of trust is an Identity Cloud concept that groups at least one identity provider and at least one service provider who agrees to share authentication information. To learn more, read [Configuring IDPs, SPs, and CoTs.](https://backstage.forgerock.com/docs/idcloud-am/latest/saml2-guide/saml2-providers-and-cots.html)

  ForgeRock Access Management can be hosted on a variety of domains. Sign in to
  your ForgeRock admin account. 

1.
    Navigate to **Native Consoles** > **Access Management.**

  ![nativeconsole.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/nativeconsole.png)

2.
    Navigate to **Applications** > **Federation**> **Circle of Trust**.

  ![applications.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/applications.png)

3. Click **Add Circle of Trust**.

4.
    Provide a name, and then click **Create**.

  ![circleoftrust.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/circleoftrust.png)

5.  Add a **Description** and click **Save Changes**.

  ![circleoftrustsaved.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/circleoftrustsaved.png)

### Create a Hosted Entity Provider

  The following steps will configure ForgeRock as the Hosted Identity Provider. 

1. Browse to **Applications** > **Federation** > **Entity Providers** > **Add Entity Provider** and click **Hosted**.
 
  ![hostedentityprovider.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/hostedentityprovider.png)

2.
    Set the **"ENTITY ID**" for ForgeRock and verify the **Entity Provider Base URL** value. In the Meta Aliases section, provide a URL-friendly value in the Identity Provider Meta Alias. Click **Create**.

  ![EntityID_ForgeRockIDP.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/EntityID_ForgeRockIDP.png)

3.
    Next, a role is created for the IDP-hosted Entity Provider.

  ![EntityProviders_ForgeRockIDP.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/EntityProviders_ForgeRockIDP.png)

### Import and Configure a Remote Entity Provider

  The service provider details have already been configured and can be downloaded from the iboss cloud platform. Refer to step 1 in this document on how to download the xml file _._This iboss XML will be imported into the ForgeRock platform as a Remote Service Provider.

1.
    Click the **Add Entity Provider** and select **Remote**.

  ![addentityproviderremote.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/addentityproviderremote.png)

2.
    On the New Remote Entity Provider page, Drag and drop the iboss **Service Provider (SP) Metadata** XML file into the dotted box. Click **Create**.

3. Next, the Entity Provider will include both the IDP and SP roles.

  ![EntityProvidersList.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/EntityProvidersList.png)

### Generate ForgeRock IDP Metadata

  You will need to export the IDP settings and provide them to the Service Provider.

1.
    Export the XML-based metadata from your hosted provider using the following URL. Make sure to change the URL to point to your ForgeRock instance.

```
https://openam.example.com/openam/saml2/jsp/exportmetadata.jsp?entityid=myHostedProvider&realm=/mySubRealm
```

2.
    Copy and Paste the contents into an editor of your choice and save the file as an XML.

  ![MetadataForgeRock.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/MetadataForgeRock.png)

## Step 3 - Identity Provider Group Mappings

  In this step, we need to map the user Group values from the Identity Provider to the "iboss Gateway SAML SP."

  Configuring the group mapping ensures that the group values can be sent in the assertion to the SP allowing group memberships to be sent and read by the SP for group-based policy assignment.

  First, we will configure a test user that we can use to log in and then configure a user profile attribute to be sent to the Service Provider application.

  Sign in to your ForgeRock Platform admin account "[https://customersubdomainadmin.ForgeRock.com/login](https://customersubdomain-admin.forgerock.com/admin)"

1. Log in to your Identity Cloud Console and browse to  **Identities**  > **Manage** and click **New Realm User** to create a test user.

  ![adduserforidp.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/adduserforidp.png)

2.
    In the New User page, enter user details and click **Save**.

3. Navigate to **Native Console** > **Access Management**.

  ![nativeconsole.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/nativeconsole.png)

4. Navigate to **Applications** > **Federation** > **Entity Providers** and select the Entity provider iboss-gateway-sp to edit the settings.

  ![EntityProvidersSP.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/EntityProvidersSP.png)

5. Navigate to **Assertion Processing** > **Attribute Mapper.**
6. Navigate to **Attribute Map** and add **Group Names** as the **Name Format** and **memberOf** for the **SAML Attribute**. Click **Add**.

  ![EntityID_ibossSP_GroupMapping.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/EntityID_ibossSP_GroupMapping.png)

## Step 4 - Configure iboss Gateway SAML SP settings

1.
    Sign in to your iboss Cloud account "[https://cloud.iboss.com](https://cloud.iboss.com/)"
2. Navigate to "Users, Groups and Devices → User SSO → SAML."

  ![usersgroups_devices.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/usersgroups&devices.png)

3. From here, set the following:

  Paste the "IDP Metadata" from the previous step, "Step 2 - Configure Identity Provider Settings."

  Optional - "SAML Auth Bypass Domains" and "SAML IDP Domains, "it is recommended to send the ForgeRock IDP domains direct using a PAC Zone detailed in Step 5 "Configure PAC Zone":

```
forgerock.com,forgeblocks.com
```

1.
    Set " **IDP Binding Method**" to " **Post**."
2.
    Set " **SAML Auth Method**" to " **Agent-Based**."
3.
    Ensure that " **Use SSL**" is enabled.
4.
    If more than one gateway is present, " **Use Cluster SAML**" should be enabled.
5.
    Finally, click " **Save**."

  ![ibossSAMLsettings.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/ibossSAMLsettings.png)

## Step 5 - Configure PAC Zone

  To prevent redirect loops with IdP auth traffic and to prevent any unnecessary filtering, we alter the PAC Zone to include the required IdP domains within a function to bypass and send direct.

  If clients are behind a firewall, a local breakout should be enabled for direct access to the domains specified:

1.
    Navigate to **Locations and Geomapping**.

  ![locations_geomapping.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/locations&geomapping.png)

2.
    Edit the intended **PAC Zone** or **PAC Zones**.

  ![editpac.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/editpac.png)

3.
    Select the **PAC Settings** tab.

  ![pacsettings.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/pacsettings.png)

4.
    Beneath **Add A Function**, select the **Select a function** dropdown menu and click **Domain and Sub-domain List.**

  ![pacdomainlist.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/pacdomainlist.png)

5. Enter the domains to which ForgeRock is hosted to the **Domain and Sub-domain** list field. Click **Add** to confirm the addition of the new domain bypass list for ForgeRock resources.

```
forgerock.com, forgeblocks.com
```

  ![ForgeRockBypasses.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/ForgeRockBypasses.png)

## Step 6 -Configure iboss Cloud Proxy Settings

  Next, the iboss Cloud proxy needs configuring to accept the user authentication and the registration type.

1. Navigate to **Proxy and Caching** > **Proxy Settings**.

  ![navigatingtoproxy.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/navigatingtoproxy.png)

2. Set " **User Authentication Method**" to " **Local User Credentials + Cloud Connections**."
3.
    Set " **Connector Registration Method**" to " **Standard Registration + SAML**."
4.
    Click " **Save**."

  ![proxysettings.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/proxysettings.png)

## Step 7 - Agent Policy - Enable the iboss Desktop App & SAML Runtime Mode

  Next, we need to create an "Agent Policy" to install and enable the "iboss Desktop App and SAML."

1.
    Navigate to **Connect Devices to Cloud** → **Agent Policies** and Click **Add Agent Policy**.

  ![navigatetoagentpolicies.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/navigatetoagentpolicies.png)

2.
    Give an Agent Policy Name of " **Enable Desktop App & SAML**. Click **Add Agent Policy**.

  ![Add_Agent_Name.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/Add_Agent_Name.png)

3.
    From the " **Dynamic Linking**" tab, select what groups to target/assign this policy to.

  ![openingdynamicsettingstab.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/openingdynamicsettingstab.png)

4.
    From the **Agent Settings** tab, find the **General Settings** area and set the **Runtime Mode** to **SAML**.

  ![samlruntimemode.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/samlruntimemode.png)

  Important - Do not complete this step unless "Steps 5 & 6"  have been completed. Otherwise, the targeted/assigned devices will lose connectivity.

5.
    From the **Agent Settings** tab, find the **Windows Desktop App** area and set the **Enable Windows Desktop App** to enabled.

  ![windowsdesktopapppolicy.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/windowsdesktopapppolicy.png)

6.
    Scroll to the bottom and click **Save**.

  Further "Desktop App" reading can be found here: [Desktop App for the Windows Connector](https://support.ibosscloud.com/hc/en-us?articles/360047892793).


  Once the above Agent Polices have fully replicated within the Cluster, the Cloud
  Connector needs to pull these new Agent Policy settings.

  The Cloud Connector will poll for new Agent policies every hour as default or if a restart, user event change, or network event change occurs. We would recommend restarting the PC and signing in again.

  Once restarted and signed in, we should see the iboss Desktop App in a "Not Connected" state.

## Step 8 - Registering to iboss cloud

  After the above Agent Polices have fully replicated within the Cluster, the Cloud Connector needs to pull these new Agent Policy settings.

  The Cloud Connector will poll for new Agent policies every hour as default or if a restart, user event change, or network event change occurs. We would recommend restarting the PC and signing in again.

1.
    Once restarted and signed in, locate the **iboss Cloud Desktop App**.

  ![ibossdesktopapp.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/ibossdesktopapp.png)

2.
     Click **Connect to Cloud**.

  ![connecttocloud.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/connecttocloud.png)

3.
    This will begin the SAML registration process redirecting the user to the IDP for sign-in.

  ![forgerocksignin.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/forgerocksignin.png)

4.
    A successful SAML authentication will notify the user in the browser.

  ![samlsuccessfullogin.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/samlsuccessfullogin.png)

## Step 9 - Verify Agent Policies have been Assigned

  The Cloud Connector pulls any "Agent Policy" change on a frequency of every hour
  or if a registration is forced, which can be done either via the below:

* Logoff+Login
* IBSA Service Restart
* Device Reboot
* Network Change

  Once the registration occurs, the device assignments can be checked under the "Cloud Connected Devices" report and drill down on an example device.

1.
    Under **Users, Groups and Devices** → **Cloud Connected Devices**.

  ![navigatingtocloudconnecteddevices.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/navigatingtocloudconnecteddevices.png)

2. "Double Click" on an example device:

  ![users_devices.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/users&devices.png)

3.
    Observe the " **Registration Information**" detailing what Agent policies have been assigned since the last Cloud Connector registration:

  ![agentregistratrion.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/agentregistratrion.png)

## Step 10 - SAML Client Cookie Lifetime

  The client-side Cookie generated during a successful SAML assertion has a configurable lifetime.

  This is called "Enable Session Timeout" and is disabled as default to never expire.

  However, it can be set to have a finite expiry time forcing the user to have to reauthenticate.

1.
    Navigate to **Connect Devices to Cloud** -> **Cloud Connectors** -> **Advance Connector Settings**.

  ![advancedcloudconnectorsettings.png](https://raw.githubusercontent.com/ForgeRock/iboss-Single-Sign-On/master/media/advancedcloudconnectorsettings.png)

2.
    From here, you can **Enable Session Timeout**, set a value in **Session Timeout Minutes**, and **Save.**
