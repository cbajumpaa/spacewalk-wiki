** Initial draft in progress **
# Feature Title

### Reviewed by:

## Overview




The current multiple organization feature as required by [[_Features_MultiOrg2_Requirements]] provides the ability to migrate systems from one organization to another through either the XMLRPC API or a stand-alone tool (migrate-system-profile); however, as a system is migrated, several of the characteristics of the system associated with the originating organization are removed from the system.  This includes characteristics such as the following:

 * entitlements
 * software channel subscriptions
 * system group membership
 * virtual guest associations
 * monitoring probes
 * configuration channels
 * custom data values
 * snapshots

As a result, the system after migration requires some additional configuration to be applied to it, either using the web-UI or XMLRPC APIs.  In order to minimize the amount configuration needed after the migration, it is desirable to have this work done as part of the migration itself by leveraging an Activation Key, similar to what can be done when a system registers in to an organization.
## Requirements



 * Support usage of an optional activation key when performing a system migration.  This must be supported by both:
   * the XMLRPC API org.migrateSystems 
   * the spacewalk-utils migrate-system-profile script
   * *TODO*: Do we need to support using more than 1 activation key during a system migration?
     [[BR]]''I don't know. We'd have to discuss this and the implications -taw''

 * The activation key specified during migration must exist in the destination org.

 * When an activation key is included during the migration:
   * The key can be used to:
     * assign system entitlements, base channel and child channels to the system
     * assign system to system groups
     * subscribe system to configuration channels
     * schedule action to deploy configuration files to the system
     * schedule action to install packages to the system
   * If the activation key's "usage" is not specified (i.e. empty), the usage will be unlimited; otherwise, it will specify the number of times the activation key may be used.
   * Prior to performing the migration, it will be verified that the destination org has enough entitlements (system and software) to support the requested migration.  If not enough entitlements are available, no systems will be migrated and an error will be generated to the user.

 * Universal default - if an activation key is defined as the Universal Default, it will be used for system migration's where no activation key has been included.
   * *TODO*: Do we want/need to support Universal Default for system migrations?  (If so, minor chg needed to Mockup to account for it.)
     [[BR]]''I would assume so. What does it mean if we don't? -taw''
   * *TODO*: If so, should the same Universal Default be used for both system migrations and registrations?  (i.e. should we support 1 Universal Default field or 2 (e.g. Migration Universal Default, Registration Universal Default)?)
     [[BR]]''I would assume so. Need to discuss. -taw''
## Other Feature Impact



 * UI - This feature will use the existing Activation Key UI (i.e. Systems -> Activation Keys) and no impact/changes are currently anticipated to it to support this feature.
 * API - This feature primarily impacts the existing API (system.migrateSystems) and migrate-system-profile tool.  Details on impact provided in the 'Proposed Implementation'.
 * Schema - No impacts identified.
 * Backend - No impacts identified.
## Proposed Implementation



 * Script - system-migrate-profile (package: spacewalk-utils)
  * Update script to support the optional "--activationkey=ACTIVATIONKEY" input
  * Update manpage to document "--activationkey=ACTIVATIONKEY"
  * This script will basically pass the activation key (if provided) to the XMLRPC API that will perform the actual migration of the system to the destination org.

 * API - org.migrateSystems
  * Deprecate the existing org.migrateSystems(string sessionKey, int toOrgId, array[systemId](int))
  * Create a new API: array[serverIdMigrated](int) org.migrateSystems(string sessionKey, int toOrgId, string activationKey, array[systemId](int)), where activation key is optional
  * Leverage existing java objects to retrieve the activation key and associate those items it defines (e.g. base channel, child channels, entitlements, server groups, config channels...etc) with the server.  More details included in the Mockups section below.
    * While investigating the feature, did consider possibly leveraging the backend logic for processing an activation key (e.g. backend/server/rhnServer/server_token.py) in the system migration solution; however, it would likely require the java code to interface with it using XMLRPC as well as need modifications to ensure error handling meets the needs of the new system migration logic.  We do, however, have much of the logic needed to apply the activation key also available in java through various existing hibernate, data source, Manager, Factory and Command objects; therefore, we plan to leverage that logic instead.
### Known Issues



There are several decisions that need to be made before this specification is finalized.  Those decisions are highlighted with *TODO*. 
### Future Enhancements



None currently identified.
## Mockups



This is an API/script-based feature; therefore, no UI mockups are included.

The key changes needed to support applying Activation Keys during a system migration will be to the XMLRPC Java API (system.migrateSystems).  The basic algorithm to be applied will be the following:
   (Note: some of this logic currently exists.)

 * If the user executing the migration is not an org admin, throw exception.  No migration performed.
 * If the destination org does not exist, throw exception.  No migration performed.
 * If an activation key is provided, but does not exist in the destination org, throw exception. No migration performed.
 * If at least 1 of the servers provided does not exist, throw exception.  No migration performed.
 * If at least 1 of the servers provided is in an org that is not trusted by the destination org, throw exception.  No migration performed.
 * If an activation key is provided and --either--
   * it has a non-null Usage defined and that usage is less than the number of servers provided, throw exception.  No migration performed.  --or --
   * it has specified assignment of system or channel entitlements, but the destination org does not have enough to support all systems provided, throw exception.  No migration performed.
 * For each of the servers provided:
   * Migrate the server to the destination org.  This will perform things such as: 
     * remove relationships the server has with the originating org including: system and channel entitlements, channels, system groups, custom data values, virt guests, config channels, reactivation keys, errata & package cache, snapshots and monitoring probes.
     * update org administrator relationships - unassociate the server from org admins in the originating and associate with org admins in the destination org.
     * reassign the server to the destination org
   * If an activation key is provided, apply the characteristics defined by that key to the server.  This will perform things such as:
     * assign system entitlements, base channel and child channels to the system
     * assign system to system groups
     * subscribe system to configuration channels
     * schedule action to deploy configuration files to the system
     * schedule action to install packages to the system
   * Add an event to the server history to record the migration.
   * Add an entry to to rhnSystemMigrations to record the migration.
## Tasks



Break down the work items probably in a table structure. Is there webui work? what about cli? database? etc.

|  Description  |  Estimate  |  Confidence  |
| --- | --- | --- |
|  migrate-system-profile script  |  6h  |  4  |
|  migrate-system-profile manpage  |  1h  |  5  |
|  xmlrpc api : org.migrateSystems  |  32h  |  4  |
## Test Notes



The current migrateSystems API and supporting code (e.g. MigrationManager) is currently tested using junit.  As part of this feature enhancement, additional junits will be added to cover the new logic.

In addition, it will also be beneficial to configure and test a migration involving a large number of systems (e.g. 1K).
## Risks/Concerns



There no significant risks or concerns to be noted at this time.  
