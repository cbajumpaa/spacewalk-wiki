(For more details on the Multiple Organization Support Phase 2 feature, see [[_Features_MultiOrg2]])
# User Interface Mockups

## Spacewalk Administrator View




The Spacewalk administrator will be the user role authorized to create and remove trusts between organizations. Trusts are bi-directional, so if the spacewalk admin sets up a trust between org a and org b, org a can see org b's shared content AND org b can see org a's shared content.
### Viewing Trusts Across Organizations



For an overview of which organizations have trusts set up and how many they have set up, the current 'Organizations' overview page (/rhn/admin/multiorg/Organizations.do) will be expanded to include a column to indicate trusts. See mockup below.

![Alt](images/orgs-list.png?raw=True)
### Viewing Trusts for One Organization



To view and modify the trusts associated with a particular organization, you click on the entry in the 'trusts' column for that org and it will take you to a new 'Trusts' tab in that organization's details screen:

![Alt](images/org-trust-tab.png?raw=True)
### Confirming Trust Removal



Sometimes you may choose to remove a trust without realizing that the organization you're removing the trust from has systems that depend on that trust relationship being there for content. This screen will list out any organizations whose trust the current organization relies on for system content:

![Alt](images/org-trust-tab-confirm.png?raw=True)
## Organization Administrator View



The org admin at least needs some visibility into which organizations are trusted by his org and which trust his org.

This screen *will not* allow the switching of systems/channels.

Instead, *the Spacewalk administrator will be able to switch off system migration and/or channel sharing bits between channels*'. This simplifies the feature because we don't have to worry about the stars and moon aligning properly for either to work between two org admins (bidirectional constraint). *Need a new mockup for spacewalk admin only screen, and for channels we need a confirm screen similar to the one above ("Confirming Trust Removal")*. Still working these details out in the requirements.

![Alt](images/org-mytrusts.png?raw=True)
### External Organization Details: Details

![Alt](images/org-extorg-details.png?raw=True)

### External Organization Details: Provided Channels

![Alt](images/org-extorg-channelsprovided.png?raw=True)

### External Organization Details: Consumed Channels

![Alt](images/org-extorg-channelsconsumed.png?raw=True)
## Mockup Sources



The sources for these mockups are in the SVG files in the Attachments list below. Note they were created in Inkscape.

