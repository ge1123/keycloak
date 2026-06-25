# Federation

## Purpose

Route user federation investigations across LDAP, standalone Kerberos, SSSD,
and Ipatuura without conflating provider families, external system assumptions,
or test surfaces.

## Scope

`federation/ldap`, `federation/kerberos`, `federation/sssd`, and
`federation/ipatuura`.

## Verified Facts

* All four modules register `UserStorageProviderFactory` implementations through
  service-loader files.
* `federation/ldap` owns the LDAP user storage provider, identity-store/query
  layer, LDAP mapper SPI, mapper implementations, admin extension providers for
  test connection/capability checks, and LDAP/Kerberos integration.
* `federation/kerberos` owns the standalone Kerberos user storage provider,
  SPNEGO and username/password Kerberos authenticators, Kerberos config, and a
  read-only user delegate. Its factory says standalone Kerberos is for
  non-LDAP-backed Kerberos.
* `federation/sssd` owns SSSD/PAM-backed user storage, D-Bus access to SSSD,
  PAM authentication, and bundled `org.jvnet.libpam`/JNA bridge code.
* `federation/ipatuura` owns a SCIM/Ipatuura user storage provider that talks to
  remote HTTPS SCIM/domain/credential endpoints through `SimpleHttp`.
* LDAP depends on `keycloak-kerberos-federation`; Kerberos does not depend on
  LDAP.

## Entry Points

* LDAP:
  `LDAPStorageProvider`, `LDAPStorageProviderFactory`,
  `LDAPIdentityStoreRegistry`, `LDAPStorageMapperManager`,
  `LDAPStorageMapperSpi`, mapper factories, `TestLdapConnectionRealmAdminProvider`,
  and `LdapServerCapabilitiesRealmAdminProvider`.
* Kerberos:
  `KerberosFederationProvider`, `KerberosFederationProviderFactory`,
  `SPNEGOAuthenticator`, and `KerberosUsernamePasswordAuthenticator`.
* SSSD:
  `SSSDFederationProvider`, `SSSDFederationProviderFactory`, `Sssd`, and
  `PAMAuthenticator`.
* Ipatuura:
  `IpatuuraUserStorageProvider`, `IpatuuraUserStorageProviderFactory`,
  `Ipatuura`, and `IpatuuraAuthenticator`.

## Flows

* LDAP factory creation flows through config decorators and
  `LDAPIdentityStoreRegistry.getLdapStore`, then creates
  `LDAPStorageProvider`.
* LDAP Kerberos auth can reuse an already-authenticated SPNEGO context through
  `KerberosConstants.AUTHENTICATED_SPNEGO_CONTEXT` so another LDAP/Kerberos
  provider can look up the user without replaying Kerberos.
* SSSD lazily opens a system-bus D-Bus connection, reads user/group data from
  SSSD, authenticates password through PAM, imports users locally, and returns
  read-only delegates.
* Ipatuura logs into a remote HTTPS service using CSRF/session cookies, then
  calls SCIM/domain endpoints and a password-validation endpoint.

## Investigation Strategy

Use `qn_pattern` for this area; `file_pattern` returned no results during
ingest for federation module paths.

```text
search_graph(qn_pattern=".*federation\\.ldap.*LDAPStorageProvider.*")
search_graph(qn_pattern=".*federation\\.kerberos.*KerberosFederationProvider.*")
search_graph(qn_pattern=".*federation\\.sssd.*SSSDFederationProvider.*")
search_graph(qn_pattern=".*federation\\.ipatuura.*IpatuuraUserStorageProvider.*")
```

For tests:

```text
search_graph(qn_pattern=".*federation\\.ldap\\.src\\.test.*", label="Class")
search_graph(qn_pattern=".*testsuite.*federation\\.ldap.*", label="Class")
search_graph(qn_pattern=".*testsuite.*federation\\.kerberos.*", label="Class")
search_graph(qn_pattern=".*testsuite.*sssd.*", label="Class")
```

Verify service-loader files under each module's `src/main/resources/META-INF/services`
before assuming a provider or mapper is active.

## Pitfalls

* LDAP provider avoids passing `CachedUserModel` into lower storage code to
  prevent `StackOverflowError`; preserve that layering rule.
* LDAP Kerberos/SPNEGO replay protection relies on reusing the stored SPNEGO
  context instead of authenticating again.
* SSSD behavior is environment-sensitive: system D-Bus, SSSD, PAM, native/JNA
  pieces, and external users/groups matter.
* Ipatuura assumes HTTPS endpoint shape, CSRF cookies, session cookies, and
  SCIM response schemas; useful local tests likely need HTTP stubbing.
* The graph marked many federation test nodes as `is_test=false`; classify
  tests by file path in this area.

## Related Wiki Pages

* [Persistence And Storage](persistence-storage.md)
* [Runtime Services](services-runtime.md)
* [Component Routing](index.md)

## Open Questions

* Whether Ipatuura is intended as supported product surface or a specialized
  integration; source currently has no local tests.
* Which current CI job, if any, runs SSSD tests given their environment
  requirements.
* Whether graph indexing can be adjusted so `file_pattern` works for federation
  module paths and test node flags are reliable.

