---
category: Technical
section: How-To
title: Configuring Active Directory Federation Services as a SAML identity provider
---

# Configuring Active Directory Federation Services (ADFS) as a SAML identity provider

Using Microsoft Active Directory Federation Services (ADFS) as a SAML identity provider for GitHub Enterprise requires some configuration.

##### Date Created: 2018-10-01

##### Affected products and versions:
  - [ ] GitHub.com Team product
  - [x] GitHub Enterprise (Versions: all )

## Setting up SAML Authentication with GitHub Enterprise
<table>
  <tbody>
     <tr>
      <th align="center"> Setting up SAML Authentication with GitHub Enterprise Cloud</th>
      <th align="center">Setting up SAML Authentication with GitHub Enterprise Server</th>
    </tr>
    <tr>
      <td>1. In the ADFS management console, go to the Identifiers tab, and add https://github.com/orgs/ORG_NAME as your Relying Party Identifier.
<br><img width="255" alt="screen shot 2018-10-11 at 4 40 43 pm" src="https://user-images.githubusercontent.com/18128948/46839813-a8ff3c80-cd74-11e8-93fc-a63718a3ed1e.png"></td>
      <td>1. In the ADFS management console, go to the Identifiers tab, and add `https://[hostname]` as your Relaying Party Identifier.
<br><img width="255" alt="screen shot 2018-11-19 at 3 36 51 pm" src="https://user-images.githubusercontent.com/18128948/48733574-03f83f00-ec11-11e8-87f2-38dee29079df.png"></td>
    </tr>
    <tr>
      <td>2. Under Endpoints, add a new endpoint with these settings:
        <ul>
          <li>Endpoint type: SAML Assertion Consumer</li>
          <li>Binding: POST</li>
          <li>URL: https://github.com/orgs/ORG_NAME/saml/consume</li>
       </td>
      <td>2. Under Endpoints, add a new endpoint with these settings:
        <ul>
          <li>Endpoint type: SAML Assertion Consumer</li>
          <li>Binding: POST</li>
          <li>URL: http(s)://[hostname]/saml/consume</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td>3. Under claims, you'll only need to set a claim for NameID. Please edit the claim, and use "Persistent identifier" for the format.</td>
      <td>3. Under claims, you'll need to set a claim for NameID. Please edit the claim, and use "Persistent identifier" for the format. You may map other SAML attributes as appropriate.</td>
    </tr>
    <tr>
      <td>
       4. On GitHub.com, use these settings for your <a href="https://help.github.com/articles/enabling-and-testing-saml-single-sign-on-for-your-organization/"> organization's SAML configuration</a>:
        <ul>
          <li>Sign on URL: `https://YOUR_ADFS_HOSTNAME/adfs/ls/`</li>
          <li>Issuer: `http://YOUR_ADFS_HOSTNAME/adfs/services/trust`</li>
          <li>Public certificate: You should be able to find your x509 certificate in your<a href="https://myserver.domain.com/FederationMetadata/2007-06/FederationMetadata.xml"> FederationMetadata.xml file</a>, inside of a <X509Certificate> tag. You can copy and paste this string into a tool like <a href="https://www.samltool.com/format_x509cert.php"> x509 formatter </a> to format it properly for GitHub usage</li>
      </td>
      <td> 4. Follow the directions in Configuring SAML settings to configure the GitHub Enterprise side. For ADFS your specific settings should be:
        <ul>
          <li>Sign on URL: `https://[YOUR_ADFS_HOSTNAME]/adfs/ls/`</li>
          <li>Issuer: The entityID value found in FederationMetadata.xml. Usually takes the format `http://[YOUR_ADFS_HOSTNAME]/adfs/services/trust`.</li>
          <li>Public certificate: You should be able to find your x509 certificate in your FederationMetadata.xml file, inside of a <X509Certificate> tag. You can copy and paste this string into a tool like this to format it properly with the headers: https://www.samltool.com/format_x509cert.php</li>
        </ul>   
       </td>
    </tr>
    <tr>
      <td>
     5. **NOTE:** We currently don’t support encrypting the assertion, however the SAML responses are already signed / encapsulated in HTTPS.
      </td>
    </tr>
  </tbody>
</table>


## Managing access to GitHub via a Role or Group in ADFS
GitHub Enterprise Admins can manage the level of access a user has to the GitHub Enterprise instance (example: admin or general user access) and can provision user accounts via SAML SSO through ADFS Roles and/or Groups.

_The following instructions are for GitHub Enterprise. GitHub Business Cloud currently does not support groups and roles as GitHub Enterprise does. With GitHub Business Cloud, you cannot directly map groups to admins or owners._

1. **Create GitHub Groups in ADFS**

Example: We created two sample GitHub groups in ADFS:
   * github-admin
   * github-user
   
<img width="651" alt="creategroups" src="https://user-images.githubusercontent.com/18128948/46839528-4194bd00-cd73-11e8-80dc-5846ed2fc3cb.png">

2. **Create AD User in ADFS**

Example: We created three users in ADFS. These users will allow us to verify the three unique scenarios.
   * Alan Ning-GHAdmin
   * Alan Ning-GHUser
   * Alan Ning-NotGHUser
<img width="656" alt="createusers" src="https://user-images.githubusercontent.com/18128948/46839529-4194bd00-cd73-11e8-8969-c9b9ca75a6b4.png">

3. **Assign Users to GitHub AD Groups**
<img width="657" alt="assignuserstogroups" src="https://user-images.githubusercontent.com/18128948/46839524-40fc2680-cd73-11e8-8b93-71554b444999.png">

4. **Add GitHub as a Trusted Party**
* Next, we need to establish a trust relationship between ADFS and GitHub. We can do this through [*adding a relying party
trust wizard*](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/operations/create-a-relying-party-trust).
* In the first page of the wizard, we enter in the SAML metadata URL of the GitHub server (https://github.com/orgs/ORGANIZATION/saml/metadata.xml). The wizard will fetch the
metadata from the URL and automatically fill in the necessary information.
<img width="450" alt="relyingpartytrustwizard1" src="https://user-images.githubusercontent.com/18128948/46839531-4194bd00-cd73-11e8-8e6c-7fa38c25ab60.png">
<img width="450" alt="relyingpartytrustwizard2" src="https://user-images.githubusercontent.com/18128948/46839533-4194bd00-cd73-11e8-9adb-bb79e58b33fb.png">
<img width="450" alt="relyingpartytrustwizard3" src="https://user-images.githubusercontent.com/18128948/46839534-422d5380-cd73-11e8-8b56-74decd7a63be.png">

* Go ahead and skip the Issuance Authorization Rules step for now and finish the wizard. We will address the Issurance Transform Rules and
Issurance Authorization Rules in the next section.

5. **Create Issurance Transform Rules** 
* The Issurance Transform Rules configure the mapping between the data inside the AD account and the SAML attributes. To
complete our scenarios, we need to set the core attributes expected by GitHub and the admin attribute for users with
admin privilege to GitHub.

**Core Attributes**
* First, we need to create the a claim rule to allow the core attributes to be mapped from LDAP to SAML claim types. For this
specific case, we chose the following mapping:
   * User Principal Name <-> Name ID
   * Email-Addresses <-> emails
   * Display-Name <-> full_name
These three attributes are GitHub's [core SAML attributes](https://help.github.com/enterprise/2.12/admin/guides/user-management/using-saml/#saml-attributes).
<img width="658" alt="coreattributes" src="https://user-images.githubusercontent.com/18128948/46839526-40fc2680-cd73-11e8-98db-90eb0731f23b.png">

**Admin Attribute**
* Next, we need to create an Issurance Transform rule to send the administrator=true attribute for users that are in the
github-admin group.
<img width="657" alt="coreattributes2" src="https://user-images.githubusercontent.com/18128948/46839527-40fc2680-cd73-11e8-9f46-dd439a9cd542.png">

6. **Create Issurance Authorization Rules**

* The Issurance Authorization Rules specifies who can or can not access GitHub.
   
For our scenario, we want to accept users who are in the github-admin and github-user group and reject those who are not in these groups.

<img width="657" alt="claimrule" src="https://user-images.githubusercontent.com/18128948/46839525-40fc2680-cd73-11e8-87b7-862eb8540e11.png">
<img width="450" alt="rule1" src="https://user-images.githubusercontent.com/18128948/46839535-422d5380-cd73-11e8-87b8-368bf12796a8.png">
<img width="450" alt="rule2" src="https://user-images.githubusercontent.com/18128948/46839536-422d5380-cd73-11e8-87af-d3f038351760.png">
