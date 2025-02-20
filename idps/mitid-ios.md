---
title: IOS MitID integration
layout: home
parent: App MitID integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 2
---

# IOS

Two options is available for iOS: SFSafariViewController and ASWebAuthenticationSession.

On iOS the SFSafariViewController is part of the app instance and thus when navigating to the MitID app and back to the app using app switch, the SFSafariViewController will be in focus.
The SFSafariViewController do not offer the same control for the app as ASWebAuthenticationSession, but is officially supported by MitID.

ASWebAuthenticationSession is a secure embedded browser, which is setup with a proper termination hook that triggers on the last redirect of the browser flow and returns control to the app.

Signaturgruppen recommends ASWebAuthenticationSession, as this allows for better control and we believe it to be stable across upcoming MitID updates.

## Universal links

To enable the MitID app to return to the app, the app needs to implement a Universal link by hosting an apple-app-site-association file in the root of the relavant domain and register a matching associated domain in the app, see <https://developer.apple.com/ios/universal-links/>.
