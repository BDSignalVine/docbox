## Overview

This section deals with configuring Single Sign On for use with Signal Vine Messaging Platform

SAML metadata is an XML document which contains information necessary for interaction
with SAML-enabled identity or service providers. Typically one metadata document will be generated for our own service provider and sent to all identity providers we want to enable single sign-on with. Similarly, each identity provider will make it's own metadata available for us to import into our service provider application. 

## SP Metadata

Signal Vine SP Metadata needs to be configured at the IDP end. It can be found at the
following url: `https://v3.signalvine.com/saml/metadata`. 

The Service Provider metadata file contains the following definitions

*  `entityId`: Specify the unique identity of the REST Web application
(Service Provider).

*  `AuthnRequestsSigned`: Specify if the REST Web application (Service Provider) signs
authentication requests.

*  `WantAssertionsSigned`: Specify if the REST Web application requires signed
assertions.

*  `Certificate`: Specify the certificate that must be used by the IdP to
register the Service Provider. This can either be a selfsigned
or a Certificate Authority (CA) signed certificate. 

*  `Binding`: Specify the bindings to be included in the metadata for the
SSO profile. Supported values are POST, Artifact, and
PAOS.
Signal Vine supports POST binding only.


```scala
<?xml version="1.0" encoding="UTF-8"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" entityID="SignalVine">
   <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
      <md:KeyDescriptor use="signing">
         <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:X509Data>
               <ds:X509Certificate>MIICYDCCAcmgAwIBAgIBADANBgkqhkiG9w0BAQ0FADBNMQswCQYDVQQGE
wJ1czELMAkGA1UECAwCVkExFDASBgNVBAoMC1NpZ25hbCBWaW5lMRswGQYDVQQDDBJ3d3cuc2ln
bmFsdmluZS5jb20wHhcNMTYxMjAyMDYyMzM4WhcNMjYxMTMwMDYyMzM4WjBNMQswCQYDVQQ
GEwJ1czELMAkGA1UECAwCVkExFDASBgNVBAoMC1NpZ25hbCBWaW5lMRswGQYDVQQDDBJ3d3cuc
2lnbmFsdmluZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMVPx2alTo1s1RoD00rsG15K
WUla+VJD8nSbQ08hPvrqvKht7xXvTBotxiR59+pRTShggx/Zy/
muKLPbGcwOSqtKlOxM918xm5PtXX7kun+bzIChzwis6ImQpusyZ4JM2hnn/
h68nD9mdGvNhxibaVf+O8SWTG5QWVopgrrBp5jpAgMBAAGjUDBOMB0GA1UdDgQWBBTRF8d7zc7p
LkACv3fl7ZbSS6p1ITAfBgNVHSMEGDAWgBTRF8d7zc7pLkACv3fl7ZbSS6p1ITAMBgNVHRMEBTADAQH
/
MA0GCSqGSIb3DQEBDQUAA4GBAGt5ehX5jspMS8uStNPphlPfzbO1bEGDlFbuwbYjUO8LLZTlWvUvpp
CiHdvHy4kz8vBez1XMHH8yNGoqr+E/
yz718Ky3bHpPHD9IdQd1Tp54AWBDR4lmkkH4CPc7bVHuUbhuFZIONRfjCVPGo/
Avs0K9tr03e1h7t+Rykh4K2eQv</ds:X509Certificate>
            </ds:X509Data>
         </ds:KeyInfo>
      </md:KeyDescriptor>
      <md:KeyDescriptor use="encryption">
         <ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:X509Data>
               <ds:X509Certificate>MIICYDCCAcmgAwIBAgIBADANBgkqhkiG9w0BAQ0FADBNMQswCQYDVQQGE
wJ1czELMAkGA1UECAwCVkExFDASBgNVBAoMC1NpZ25hbCBWaW5lMRswGQYDVQQDDBJ3d3cuc2ln
bmFsdmluZS5jb20wHhcNMTYxMjAyMDYyMzM4WhcNMjYxMTMwMDYyMzM4WjBNMQswCQYDVQQ
GEwJ1czELMAkGA1UECAwCVkExFDASBgNVBAoMC1NpZ25hbCBWaW5lMRswGQYDVQQDDBJ3d3cuc
2lnbmFsdmluZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAMVPx2alTo1s1RoD00rsG15K
WUla+VJD8nSbQ08hPvrqvKht7xXvTBotxiR59+pRTShggx/Zy/
muKLPbGcwOSqtKlOxM918xm5PtXX7kun+bzIChzwis6ImQpusyZ4JM2hnn/
h68nD9mdGvNhxibaVf+O8SWTG5QWVopgrrBp5jpAgMBAAGjUDBOMB0GA1UdDgQWBBTRF8d7zc7p
LkACv3fl7ZbSS6p1ITAfBgNVHSMEGDAWgBTRF8d7zc7pLkACv3fl7ZbSS6p1ITAMBgNVHRMEBTADAQH
/
MA0GCSqGSIb3DQEBDQUAA4GBAGt5ehX5jspMS8uStNPphlPfzbO1bEGDlFbuwbYjUO8LLZTlWvUvpp
CiHdvHy4kz8vBez1XMHH8yNGoqr+E/
yz718Ky3bHpPHD9IdQd1Tp54AWBDR4lmkkH4CPc7bVHuUbhuFZIONRfjCVPGo/
Avs0K9tr03e1h7t+Rykh4K2eQv</ds:X509Certificate>
            </ds:X509Data>
         </ds:KeyInfo>
      </md:KeyDescriptor>
      <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://172.16.0.106:8080/auth/saml-callback" />
      <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</md:NameIDFormat>
      <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTPPOST" Location="http://172.16.0.106:8080/auth/saml-callback" index="1" />
   </md:SPSSODescriptor>
</md:EntityDescriptor>
```