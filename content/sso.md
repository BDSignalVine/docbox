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

#### Sample Service Provider Metadata File

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

## IDP Metadata

An Identity Provider (IdP), sometimes called an Identity Service Provider or Identity
Assertion Provider, is an online service or website that authenticates users on the Internet
by means of security tokens, one of which is SAML 2.0. Signal Vine needs
the IDP SAML metadata that we get from the IdP that includes the issuer's name, expiration
information, and keys that can be used to validate the SAML authentication response
(assertions) that are received from the IdP. 

## IDP Metadata Mapping

The SAML 2.0 assertion should include all the attributes we need for the user in the service
provider (Signal Vine application). Exactly what is transported is a matter of negotiation
between SV and the operator of the identity provider. The identity provider sends the
attributes as attribute=value pairs. We need to know the name of the SAML 2.0 attribute
and what kind of value it carries so we can map it to user attributes in SV application
(already mapped) 

#### Mapping Attributes to fetch from IDP SAML Response 
```scala
<saml:Attribute Name="uid" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
 <saml:AttributeValue xsi:type="xs:string">test15@mailinator.com</saml:AttributeValue>
</saml:Attribute>
<saml:Attribute Name="telephoneNumber" NameFormat="urn:oasis:names:tc:SAML:2.0:attrnameformat:basic">

 <saml:AttributeValue xsi:type="xs:string">18014374321</saml:AttributeValue>
</saml:Attribute>
<saml:Attribute Name="givenName" NameFormat="urn:oasis:names:tc:SAML:2.0:attrnameformat:basic">

 <saml:AttributeValue xsi:type="xs:string">TUser99991</saml:AttributeValue>
</saml:Attribute>
<saml:Attribute Name="sn" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
 <saml:AttributeValue xsi:type="xs:string">Test88881</saml:AttributeValue>
</saml:Attribute>
<saml:Attribute Name="mail" NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:basic">
 <saml:AttributeValue xsi:type="xs:string">test15@mailinator.com</saml:AttributeValue>
</saml:Attribute>
```