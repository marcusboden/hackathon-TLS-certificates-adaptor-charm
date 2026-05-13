 ## Charmed Operator Comparative Analysis                                                                                                                                                         
                                                                                                                                                                                               
 ### What They Do                                                                                                                                                                                  
                                                                                                                                                                                               
 All three charms are TLS certificate adaptors: they bridge the legacy (v1/reactive) tls-certificates interface used by older OpenStack charms with the modern tls-certificates interface      
 (v3/v4) used by providers such as lego, vault, and self-signed-certificates.                                                                                                                  
                                                                                                                                                                                               
 ────────────────────────────────────────────────────────────────────────────────                                                                                                              
                                                                                                                                                                                               
 ### Comparison Matrix                                                                                                                                                                             
                                                                                                                                                                                               
 ┌─────────────────────────┬────────────────────────────────────────────────────┬──────────────────────────────────────────────────────┬─────────────────────────────────────────────────────┐ 
 │ Dimension               │ certificate-translator-operator                    │ tls-certificates-adaptor-operator                    │ tls-translator-charm                                │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Lines of charm code     │ ~498                                               │ ~930                                                 │ ~572                                                │ 
 │ (src/)                  │                                                    │                                                      │                                                     │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Interface library       │ tls-certificates v3 (copied into lib/, +2 123 LOC) │ charmlibs.interfaces.tls_certificates v4 (PyPI dep,  │ charmlibs.interfaces.tls_certificates v4 (PyPI dep) │ 
 │                         │                                                    │ no lib dir)                                          │                                                     │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Architecture            │ Delta/event-driven                                 │ Holistic reconciliation                              │ Delta/event-driven                                  │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ State management        │ StoredState (mutable dicts; private keys stored    │ CharmState Pydantic snapshot (immutable, computed    │ StoredState (JSON-serialized dataclasses)           │ 
 │                         │ unencrypted)                                       │ per event)                                           │                                                     │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ CSR/Key generation      │ Reinvented: manual cryptography calls              │ Delegated to library (CertificateRequestAttributes)  │ Uses library classes but then reinvents relation    │ 
 │                         │                                                    │                                                      │ databag writes                                      │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Renewal handling        │ Manual expiring handler with CSR regeneration      │ Handled internally by v4 library                     │ Not implemented                                     │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Client cert synthesis   │ ❌ None                                            │ ✅ Required by ovn-central                           │ ❌ None                                             │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Legacy format support   │ Batch only (cert_requests)                         │ Legacy (common_name/sans) + Batch                    │ Legacy + Batch + Application-scoped                 │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Python target           │ >=3.10                                             │ >=3.10                                               │ Unspecified                                         │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ charmcraft.yaml         │ Consolidated, single base                          │ Consolidated, multi-base, rich metadata              │ Consolidated but minimal, uses legacy plugin: charm │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Testing framework       │ ops.testing.Context (modern)                       │ ops.testing.Context (modern)                         │ ops.testing.Harness (legacy)                        │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Unit test count         │ 1 file, decent coverage                            │ 5 files, extensive coverage                          │ 1 file, 2 tests, minimal                            │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Integration tests       │ Minimal (deploy + wait)                            │ Placeholder only                                     │ None                                                │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Type hints / linting    │ Good                                               │ Excellent (mypy, ruff, bandit)                       │ Mixed, inconsistent                                 │ 
 ├─────────────────────────┼────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼─────────────────────────────────────────────────────┤ 
 │ Peer relation           │ None                                               │ None                                                 │ cluster (defined but unused)                        │ 
 └─────────────────────────┴────────────────────────────────────────────────────┴──────────────────────────────────────────────────────┴─────────────────────────────────────────────────────┘ 
                                                                                                                                                                                               
 ────────────────────────────────────────────────────────────────────────────────                                                                                                              
                                                                                                                                                                                               
 ## Major Technical Differences                                                                                                                                                                   
                                                                                                                                                                                               
 ### 1. Library Adherence & "Reinventing the Wheel"                                                                                                                                            
                                                                                                                                                                                               
 - tls-certificates-adaptor-operator is the only charm that fully delegates to the modern v4 library. It never touches cryptography directly, never manually constructs relation databag       
 entries for the v4 side, and lets TLSCertificatesRequiresV4.sync() handle CSR lifecycle.                                                                                                      
 - certificate-translator-operator copies the v3 library into lib/ (technical debt) and then reinvents CSR generation with raw cryptography.x509 calls. It also manually tracks renewal.       
 - tls-translator-charm imports v4 classes (CertificateRequestAttributes, PrivateKey) but then manually injects JSON into the v4 relation databag (certificate_signing_requests key). This is  
 a fatal flaw: it bypasses the library's sync protocol and will break when the library's wire format changes.                                                                                  
                                                                                                                                                                                               
 ### 2. Architecture                                                                                                                                                                           
                                                                                                                                                                                               
 - tls-certificates-adaptor-operator uses holistic reconciliation: one reconcile() method reads the entire world, computes the desired state, and writes it. This is the Canonical-recommended 
 pattern for stateless bridge charms.                                                                                                                                                          
 - The other two use delta-style event handlers that mutate StoredState incrementally. This is harder to test and prone to state drift.                                                        
                                                                                                                                                                                               
 ### 3. State Management                                                                                                                                                                       
                                                                                                                                                                                               
 - tls-certificates-adaptor-operator uses a Pydantic CharmState snapshot. No StoredState is used anywhere.                                                                                     
 - certificate-translator-operator stores private keys in StoredState, which is an unencrypted, unversioned key-value store. This is a security vulnerability.                                 
 - tls-translator-charm serializes dataclasses into StoredState JSON, which is brittle and loses type safety.                                                                                  
                                                                                                                                                                                               
 ### 4. OpenStack v1 Interface Compatibility                                                                                                                                                   
                                                                                                                                                                                               
 - tls-certificates-adaptor-operator explicitly supports:                                                                                                                                      
     - Legacy single-cert format (common_name + sans)                                                                                                                                          
     - Batch format (cert_requests)                                                                                                                                                            
     - Synthetic client cert generation per relation (client.cert/client.key), which charms like ovn-central require.                                                                          
 - certificate-translator-operator supports only the batch format. It will fail with reactive charms that use the legacy top-level keys.                                                       
 - tls-translator-charm parses both formats, plus client_cert_requests and application_cert_requests, but its delivery logic is brittle and the app-level request deduplication is broken (it  
 merges SANs incorrectly).                                                                                                                                                                     
                                                                                                                                                                                               
 ### 5. Domain / SAN Filtering                                                                                                                                                                 
                                                                                                                                                                                               
 - certificate-translator-operator and tls-translator-charm implement opinionated filtering that rejects .lxd, RFC1918 IPs, etc. For an adaptor charm, this is incorrect: it should forward    
 whatever the legacy requirer asks for.                                                                                                                                                        
 - tls-certificates-adaptor-operator has no filtering — it trusts the requirer and the upstream provider. This is the correct adaptor behavior.                                                
 - tls-translator-charm has a dangerous bug: when the CN is non-public but SANs are public, it replaces the CN with the first public SAN, changing the certificate identity.                   
                                                                                                                                                                                               
 ### 6. CA Bundle Handling                                                                                                                                                                     
                                                                                                                                                                                               
 - tls-certificates-adaptor-operator properly deduplicates the CA and chain into a single ca key (the v1 interface has no chain key) and supports appending extra CA certificates via charm    
 config.                                                                                                                                                                                       
 - tls-translator-charm writes a chain key to the legacy relation, which no legacy consumer reads. Dead code.                                                                                  
 - certificate-translator-operator simply passes through ca and chain with no deduplication or extra config.                                                                                   
                                                                                                                                                                                               
 ────────────────────────────────────────────────────────────────────────────────                                                                                                              
                                                                                                                                                                                               
 Scoring (out of 100)                                                                                                                                                                          
                                                                                                                                                                                               
 ┌──────────────────────────────────┬───────────────────────────────────────┬──────────────────────────────────────┬───────────────────────────────────────────────────┐                       
 │ Pillar                           │ certificate-translator                │ tls-certificates-adaptor             │ tls-translator                                    │                       
 ├──────────────────────────────────┼───────────────────────────────────────┼──────────────────────────────────────┼───────────────────────────────────────────────────┤                       
 │ Library Adherence (25 pts)       │ 12/25 (v3 lib, manual crypto/renewal) │ 24/25 (full v4 delegation)           │ 8/25 (v4 classes but manual databag)              │                       
 ├──────────────────────────────────┼───────────────────────────────────────┼──────────────────────────────────────┼───────────────────────────────────────────────────┤                       
 │ Architectural Modernity (25 pts) │ 8/25 (StoredState, delta-style)       │ 23/25 (holistic reconcile, Pydantic) │ 6/25 (StoredState, delta, Harness)                │                       
 ├──────────────────────────────────┼───────────────────────────────────────┼──────────────────────────────────────┼───────────────────────────────────────────────────┤                       
 │ Code Quality (25 pts)            │ 14/25 (readable, no models/config)    │ 22/25 (excellent types/docs/modules) │ 8/25 (monolithic, brittle filtering, mixed types) │                       
 ├──────────────────────────────────┼───────────────────────────────────────┼──────────────────────────────────────┼───────────────────────────────────────────────────┤                       
 │ Testing Rigor (25 pts)           │ 16/25 (Context used, decent tests)    │ 20/25 (5 test files, comprehensive)  │ 4/25 (2 tests, legacy Harness)                    │                       
 ├──────────────────────────────────┼───────────────────────────────────────┼──────────────────────────────────────┼───────────────────────────────────────────────────┤                       
 │ Total                            │ 50/100                                │ 89/100                               │ 26/100                                            │                       
 └──────────────────────────────────┴───────────────────────────────────────┴──────────────────────────────────────┴───────────────────────────────────────────────────┘                       
                                                                                                                                                                                               
 ────────────────────────────────────────────────────────────────────────────────                                                                                                              
                                                                                                                                                                                               
 Final Verdict                                                                                                                                                                                 
                                                                                                                                                                                               
 ### 🏆 Production Ready: tls-certificates-adaptor-operator                                                                                                                                    
                                                                                                                                                                                               
 Why this wins:                                                                                                                                                                                
 - Zero wheel reinvention: Uses charmlibs.interfaces.tls_certificates v4 as a PyPI dependency. No manual CSR generation, no manual relation databag mutation for the modern side, no copied    
 vendored library.                                                                                                                                                                             
 - Holistic reconciliation: Single reconcile() entry point makes the charm trivial to reason about and unit test.                                                                              
 - No StoredState: Immutable CharmState Pydantic model eliminates an entire class of state-drift bugs.                                                                                         
 - Real OpenStack compatibility: Handles legacy common_name/sans, batch cert_requests, and synthesises the client.cert/client.key pair that ovn-central requires.                              
 - Production-grade tooling: mypy, bandit, ruff, and comprehensive unit tests with modern ops.testing.Context.                                                                                 
 - Multi-base support: charmcraft.yaml targets both 22.04 and 24.04.                                                                                                                           
                                                                                                                                                                                               
 ### Why the others are NOT production ready                                                                                                                                                   
                                                                                                                                                                                               
 - certificate-translator-operator                                                                                                                                                             
     - Security flaw: stores private keys in unencrypted StoredState.                                                                                                                          
     - Incomplete v1 support: does not handle legacy common_name/sans top-level keys.                                                                                                          
     - Reinvented crypto: uses raw cryptography.x509 instead of library helpers.                                                                                                               
     - Manual renewal handling instead of library-managed rotation.                                                                                                                            
 - tls-translator-charm                                                                                                                                                                        
     - Fatal wire-protocol bug: manually constructs the v4 relation databag, bypassing TLSCertificatesRequiresV4.sync(). This will break when the library updates.                             
     - Dangerous CN substitution: replaces non-public CNs with SANs, altering certificate identity.                                                                                            
     - Legacy test framework: still uses ops.testing.Harness instead of Context.                                                                                                               
     - Dead code: writes chain key to v1 relations that do not consume it; defines a cluster peer relation that is never used.                                                                 

