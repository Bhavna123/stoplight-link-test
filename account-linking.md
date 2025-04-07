---
stoplight-id: 03oawlb9h34tc
---

# Account Linking

cidaas user management offers a crucial feature known as account linking. When providing multiple different login providers there are cases when user conflicts are arising. In general we are differentiating two cases:

1. **Manual account linking** a user uses different identifiers e.g. registers using email melli.may@gmail.com and melli.may@web.de. The user now notices that both accounts are hers and would like to merge both accounts to one single account
2. **Social account linking** a user has already registered using email as self identity and afterwards tries to login using facebook, google or any other login provider that contains the same userId as for the self identity. In this case a user conflict arises

This feature enables users to link multiple accounts, including social media accounts like Facebook, Amazon, Gmail, etc., as well as personal accounts such as their own cidaas account, to the cidaas database without any hassle.

The primary purpose of account linking is to unify all the user accounts on a single platform, merging all relevant information in one place. By doing so, users can access their own accounts.

<!-- theme: info -->
> ### Understanding the Account Structure and Fieldsetup
> An account has 1:n identities, so cidaas allows the integration of identity providers. Each of these identities has a predefined set of system fields. See more in [field setting](../../docs/user-management/setup/field-settings.md) and recap the user [account structure](../account-structure.md)




<!--
type: tab
title: User-Initiated account linking 
-->

## Manual account linking 

In the manual linking you need to enter your , account or mobile number or user name that you want to link. cidaas will then initiate the linking process, integrating your provided information with the existing account. 


### Technical Integration


API| Description| Link
--|-------|------
Initiate a new user link | Initiates a new user link to link for existing users | [Link to API](https://docs.cidaas.com/docs/cidaas-iam/f204dcc804672-initiate-a-new-user-link)

### Process of Manual account linking   

Click [here](../../user-interfaces/default-user-ui.md) to know more about functionalites of user profile UI.


For manual account linking :

1. Login to your user profile
2. Click on the  **link my ID**, you will find the below screen.


![Link-account-manual](../../../assets/images/usermanagement/link-account-manual.png)

3. Enter the account/mobile/username you want to link in the link area and click on **link**. A the user has to enter a valid username (like email, mobile_number, username etc.) of another existing account (user B). *Prerequisites* the used username needs to be configured as allowed login method.
4. The user will get redirected to login page with username of user B as `login_hint`. In the default hosted pages the username is prefilled based on the `login_hint`
5. Afterwards the user will perform the login with the already registered account, to confirm this one also belongs to him. In the successful login call a header param link id is overhanded.
6. All the user information of the user B are now linked to the user account of user A.

Are two user accounts linked the user or admin can **unlink** this two accounts.

1. All linked accounts are listed in the link/ unlink section in the user information.
2. Once the user or admin would like to unlink one account (user B) he can select unlink for the account.
3. A verification code will be sent to the selected email id (email of user B).
4. Once verification is successfully the accounts are unlinked.
>**Note:** Once an account is unlinked, the unlinked account (user B) loses its identity and henceforth any login with that account information is not possible.

API| Description| Link
--|-------|------
Un-link the account| De-link the account using API | [Link to API]()



<!--
type: tab
title: Social account linking
-->

## Social account linking 

Social account linking refers to the process of integrating your social media accounts such as Google,Facebook, etc. into cidaas. By doing so, you can effortlessly log in using any of your linked social media accounts. 

To link a social media account, you need to have a self-cidaas account or social register account. When you register a new account with the same account with a different provider, it'll be linked automatically.

![social-account-linking](../../../assets/images/usermanagement/social-account-linking.png)



### Process of social account linking  


For social account linking :

1. Open the cidaas login page.
2. Register a new account using social login providers after allowing your instance to register with social login information.

   - You'll need to create a new account either using the SELF Account option or via a Social Account using the [Register API](https://docs.cidaas.com/docs/cidaas-iam/427632e587203-register-a-new-user). Make sure to use the same email address as registered in the social provider you'll use for login.

API | Description | Link 
---------|----------|---------
 Register a new user | To register a new user, you can use this endpoint. It allows to provide all infromation you would like to request from the user. | [Link](https://docs.cidaas.com/docs/cidaas-iam/427632e587203-register-a-new-user)
 

- If the application requires more fields than already provided by the social provider, the user will be redirected to the progressive profiling and the final register Call will be completed from the UI. In this case the API will return `StatusCode 200` and a `redirect_uri` in the body to continue to. 

   - If the application does not require fruther information the user will be redirected to the login page immediately and asked for authentication using the existing user account

3. The next step involves making an authorization call to generate a unique **requestId**. This **requestId** will be used in subsequent steps for **authentication** and **verification** purposes.

API | Description | Link 
---------|----------|---------
 Authorization call | This api can be used to perform the following oauth2/openid flows. | [Link](https://docs.cidaas.com/docs/cidaas-iam/b6d284f55be5e-authorization-request)

4. Choose a social login provider. It's important that the provider returns `email_verified` as false. This is necessary to prompt the user for linking their social account with their existing account.
5. Use the [social login with request ID](../../reference/login-providers.yaml/paths/~1login-srv~1social~1login~1:provider_name~1:requestId) endpoint to initiate the login process with the chosen social provider. After logging in, you'll be redirected to a callback URL (`/login-srv/social/callback/:provider_name`) with a code.

API | Description | Link 
---------|----------|---------
 social login with request ID | This API logs in to the social provider using the request ID(*generated during authz call*). | [Link](https://docs.cidaas.com/docs/cidaas-iam/b6d284f55be5e-authorization-request)
 Get callback URL | This API takes the **callback** URL from the application's with a code. | [Link](t)

 <br>

   6.  The next step is to exchange code for token and user information, here take the code received in the callback URL and exchange it for a token and user information.

       - At this point, a conflict might occur if the user's social account email doesn't match any existing account. In such a case, the user will be redirected to the login page with additional parameters.

7. Provide all options for the user to login i.e., If it's a self account (i.e., not a social account), provide all available login options such as email OTP (One-Time Password) or password.

8. If the user chooses to log in using a password, initiate the password verification process using the `/verification-srv/v2/authenticate/initiate/password` and `/authenticate/password` endpoints. Provide the `requestId` generated earlier in the process.


API | Description | Link 
---------|----------|---------
 Initiate verification | This API is used to initiate the verification of the user depending on the type of verification configured. | [Link](https://docs.cidaas.com/docs/cidaas-iam/2a3ea581bb249-initiate-verification)
 Perform the authentication| This API is used to initiate authentication depending on the type of verification configured.| [Link](https://docs.cidaas.com/docs/cidaas-iam/1aa38936252d6-perform-the-authentication-method)

 9. Retrieve `status_id`, After verifying the password, a `status_id` will be returned in the POST call. This `status_id` is needed for the next step.
10. Finally, use the `/login-srv/verification/login` endpoint to log in. Provide the `link-id` and `status_id` obtained in previous steps.


API | Description | Link 
---------|----------|---------
 Login after authentication | This API completes the login after a user successfully completed the passwordless authentication | [Link](https://docs.cidaas.com/docs/cidaas-iam/e6w9ei65kxnix-login-after-authentication)


>
>**Note:** Admin must configure the social media options on the login page for the user.



Your social account will be automatically linked to the main account if you have logged in with the same ID.


![social-link-flow.png](../../../assets/images/usermanagement/social-account-linking-flows.png "Social account linking flow")


### List of all linked account

![account-list-link](../../../assets/images/usermanagement/linked-account-list.png )


API| Description| Link
--|-------|------
List all linked account | Provides the list of all the linked account to the user using the `identities` scope in the access token| [Link to API](https://docs.cidaas.com/docs/cidaas-iam/2zfvjx3vtq6g6-get-user-info)



<!-- type: tab-end -->




<!-- theme: warning -->
> ### Need Support?
> Please contact us directly on our [support page](https://support.cidaas.com/en/support/home).





