<p align="center">
    <a href="http://kituranext.org/">
        <img src="https://raw.githubusercontent.com/Kitura-Next/Kitura/master/Sources/Kitura/resources/kitura-bird.svg?sanitize=true" height="100" alt="Kitura">
    </a>
</p>


<p align="center">
    <a href="http://www.kituranext.org/">
    <img src="https://img.shields.io/badge/docs-kitura-1FBCE4.svg" alt="Docs">
    </a>
    <a href="https://travis-ci.org/Kitura-Next/Kitura-CredentialsGitHub">
    <img src="https://travis-ci.org/Kitura-Next/Kitura-CredentialsGitHub.svg?branch=master" alt="Build Status - Master">
    </a>
    <img src="https://img.shields.io/badge/os-macOS-green.svg?style=flat" alt="macOS">
    <img src="https://img.shields.io/badge/os-linux-green.svg?style=flat" alt="Linux">
    <img src="https://img.shields.io/badge/license-Apache2-blue.svg?style=flat" alt="Apache 2">
    <a href="http://swift-at-ibm-slack.mybluemix.net/">
    <img src="http://swift-at-ibm-slack.mybluemix.net/badge.svg" alt="Slack Status">
    </a>
</p>

# Kitura-CredentialsGitHub

Plugin for the Credentials framework that authenticate using GitHub

## Summary
Plugin for [Kitura-Credentials](https://github.com/Kitura-Next/Kitura-Credentials) framework that authenticates using the [GitHub web login with OAuth2](https://developer.github.com/v3/oauth/#web-application-flow) API.

## Table of Contents
* [Swift version](#swift-version)
* [Example of GitHub web login](#example-of-github-web-login)
* [License](#license)

## Swift version
The latest version of Kitura-CredentialsGitHub requires **Swift 4.0** or newer. You can download this version of the Swift binaries by following this [link](https://swift.org/download/). Compatibility with other Swift versions is not guaranteed.

## Example of GitHub web login
This guide assumes basic knowledge of `Kitura` app routing.

First, set up the session middleware:

```swift
import KituraSession

router.all(middleware: Session(secret: "Very very secret..."))
```

Create an instance of `CredentialsGitHub` plugin and register it with `Credentials` framework:

```swift
import Credentials
import CredentialsGitHub

let credentials = Credentials()
let gitCredentials = CredentialsGitHub(clientId: gitClientId, clientSecret: gitClientSecret, callbackUrl: serverUrl + "/login/github/callback", userAgent: "my-kitura-app", options: ["scopes": ["user:email"]])
credentials.register(gitCredentials)
```

**Where:**

- *gitClientId* is the Client ID of your app in your GitHub Developer application settings
- *gitClientSecret* is the Client Secret of your app in your GitHub Developer application settings
- *callbackUrl* is used to tell the GitHub web login page where the user's browser should be redirected when the login is successful. It should be a URL handled by the server you are writing.
- *userAgent* is an optional argument that passes along a User-Agent of your choice on API calls against GitHub. By default, `Kitura-CredentialsGitHub` is set as the User-Agent. [User-Agent is required when invoking GitHub APIs](https://developer.github.com/v3/#user-agent-required).
- *options* is an optional dictionary (`[String: Any]`); the allowable options are listed in `CredentialsGitHubOptions`

Next, specify where to redirect non-authenticated requests:

```swift
credentials.options["failureRedirect"] = "/login/github"
```

Connect `credentials` middleware to handle requests to a protected path on the server, such as `/private`:

```swift
router.all("/private", middleware: credentials)
router.get("/private/data", handler: { request, response, next in
  ...  
  next()
})
```

And call `authenticate` to login with GitHub and to handle the redirect (callback) from the GitHub login web page after a successful login:

```swift
router.get("/login/github", handler: credentials.authenticate(gitCredentials.name))

router.get("/login/github/callback", handler: credentials.authenticate(gitCredentials.name))
```

## License
This library is licensed under Apache 2.0. Full license text is available in [LICENSE](LICENSE.txt).
