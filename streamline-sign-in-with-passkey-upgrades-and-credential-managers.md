# Streamline sign-in with passkey upgrades and credential managers

Presenter: Garett Davidson, Authentication Experience Engineer

Link: https://developer.apple.com/wwdc24/10125

- Automatic passkey upgrades
- Improvements for credential managers
- Passwords app

## Automatic passkey upgrades

- You can automatically create a passkey without upselling or upgrading
- User signs in and then gets a notification that a passkey has automatically been created
- When the user comes back to the app, they enter the username and the passkey takes care of the rest

- Checks for passkey upgrades
  - Is a credential manager available?
  - Does it support auotmatic passkey updates?
  - Is the device currently set up to use passkeys?
  - If web, it checks to make sure it's a non-private browsing tab
  - If satisfied, it passes a request to credential manager, which has its own checks
    - Was the credential manager just used to sign in to an account with this username?
    - If conditions are not met, you get an Error

- Won't lead to a passkey 100% of the time, but if it does, it's a smooth user flow with no upsell

Consider this example:

```swift
// Offering a passkey upsell

func signIn() async throws {
    let userInfo = try await signInWithPassword()
    guard !userInfo.hasPasskey else { return }
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
        relyingPartyIdentifier: "example.com")

    guard try offerPasskeyUpsell() else { return }
    let request = provider.createCredentialRegistrationRequest(
        challenge: try await fetchChallenge(),
        name: userInfo.user,
        userID: userInfo.accountID)

    do {
        let passkey = try await authorizationController.performRequest(request)
        // Save new passkey to the backend
    } catch { … }
}
```

With automatic passkey upgrades, you no longer have to offer an upsell - instead, just try it:

```swift
// Automatic passkey upgrade

func signIn() async throws {
    let userInfo = try await signInWithPassword()
    guard !userInfo.hasPasskey else { return }
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(
        relyingPartyIdentifier: "example.com")

    let request = provider.createCredentialRegistrationRequest(
        challenge: try await fetchChallenge(),
        name: userInfo.user,
        userID: userInfo.accountID,
        requestStyle: .conditional)

    do {
        let passkey = try await authorizationController.performRequest(request)
        // Save new passkey to the backend
    } catch { … }
}
```

On the web, it's very similar

## Improvements for credential managers

- Automatic passkey upgrades
- AutoFill for verification codes
- AutoFill text from editing menu

To add support for these features, there are new keys available for the Info.plist:

```swift
<dict>
	<key>NSExtensionAttributes</key>
	<dict>
		<key>ASCredentialProviderExtensionCapabilities</key>
		<dict>
			<key>ProvidesPasswords</key>
			<true/>
			<key>ProvidesPasskeys</key>
			<true/>
			<key>SupportsConditionalPasskeyRegistration</key>
			<true/>
			<key>ProvidesOneTimeCodes</key>
			<true/>
			<key>ProvidesTextToInsert</key>
			<true/>
		</dict>
	</dict>
</dict>
```

- [ASCredentialProviderExtensionCapabilities docs](https://developer.apple.com/documentation/bundleresources/information_property_list/nsextension/nsextensionattributes/ascredentialproviderextensioncapabilities)

## Passwords app

- New in fall releases
- Website/app displayed in the Passwords app
- You can change how it's displayed
- There are Passkeys and Verification Code sections
- In macOS, use a menu bar item
- Can import from another credential manager


### Update website appearance

- Add an OpenGraph site name
  - `<meta name="og:sitename" content="Shiny">
  - Serve metadata to all user agents from all user-reachable subdomains
  - Use high-resolution website icons so they look great

### Verification code setup

- Add one-tap verification setup
  - Use a button or link to open an `otpauth:` URL
  - When suggesting apps for verification code setup, consider adding "Apple Passwords"

## Resources

- [Passkeys overview](https://developer.apple.com/passkeys/)
- [Connecting to a service with passkeys](https://developer.apple.com/documentation/authenticationservices/connecting_to_a_service_with_passkeys)
