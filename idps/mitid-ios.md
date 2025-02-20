---
title: IOS MitID integration
layout: home
parent: App MitID integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 2
---

## IOS MitID integration
On iOS two recommended options are available. 

1. SFSafariViewController - Officially supported by MitID.
2. ASWebAuthenticationSession - Not officially supported by MitID. Recommended by Signaturgruppen.

The SFSafariViewController supports running the Safari browser as part of an iOS app, whereas the ASWebAuthenticationSession is a secure webview browser. 

Signaturgruppen recommend the ASWebAuthenticationSession, as this provides a proper termination hook that allows for a better handling of the UX in the app switch flow. Signaturgruppen consider the risk of a future MitID update to cause errors with the ASWebAuthenticationSession functionality to be small.

> MitID does not test and support anything but the SFSafariViewController, but does allow the usage of ASWebAuthenticationSession, there is no official support from MitID, should ASWebAuthenticationSession stop working with MitID on iOS.

## Universal links

To enable the MitID app to return to the app, the app needs to implement a Universal link by hosting an apple-app-site-association file in the root of the relavant domain and register a matching associated domain in the app, see <https://developer.apple.com/ios/universal-links/>.
