- Feature Name: csam_filtering
- Start Date: 2023-09-12
- RFC PR: [LemmyNet/rfcs#0000](https://github.com/LemmyNet/rfcs/pull/0000)
- Lemmy Issue: [LemmyNet/lemmy#0000](https://github.com/LemmyNet/lemmy/issues/0000)

# Summary

This feature wants to implement a way to combat CSAM / unwanted images.

# Motivation

This feature wants to reduce the liability risk of the instance admins, that have to fight against CSAM in many kind and forms,  
it has mental stress for everyone involved too, many admins, many mods just straight up left the platform because of the stress induced by such pictures. 
More about it here https://github.com/iftas-org/resources/blob/main/CSAM-CSE/README.md
# Guide-level explanation

Explain the proposal as if it was already included in Lemmy and you were teaching it to another user. That generally means:

- Explaining the feature largely in terms of examples.
- Explaining how Lemmy users should *think* about the feature, and how it should impact the way they use Lemmy. It should explain the impact as concretely as possible.
- For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.

- As an admin you can add your credentials of different CSAM filtering Services ( For Example [Microsofts PhotoDNA](https://www.microsoft.com/en-us/photodna)) in the Admin Settings and/or activate a offline CSAM scanner ( For example [AI Horde csam scanner](https://github.com/Haidra-Org/horde-safety/blob/main/horde_safety/csam_checker.py), [Thorn Safer [paid]](https://get.safer.io/csam-detection-tool-for-child-safety), [Meta PQD](https://github.com/facebook/ThreatExchange/tree/main/pdq)). ( Only either one CSAM filtering Service or offline csam scanner is required )
- As an example you can use 
- You can change what it should do below the credentials:
    + [X] Remove Image in the storage
    + [X] Dont federate Posts/Comments/DMs before the CSAM check is done
    + [X] Lazy Checking ( This will activate lazy checking the image => Instead of blocking the upload, it "allows" the image through but it will purge it afterwards if it hit a check )
    + [ ] Ban the inflicting user.
      * [ ] Remove Data from the inflicting User
    + [ ] Purge the inflicting user.
    + [ ] Notify Admins over dms [ Local and if possible non Local admins ]

- You can activate it then with a checkbox on top and it will scan all new images and log its actions ( Thumbnails, Embeds, Images in Posts, Images in Comments, Images in private messages )



# Reference-level explanation

This is the technical portion of the RFC. Explain the design in sufficient detail that:

- The picture upload process has to change if this feature is active:
    * For Every upload or image creation ( from local or federated ) it has to go through activated checks before it finally lands in the storage. 
        + The checks has to be done before showing that post/comment/dm.

    * If the checks failed because it returned it IS csam, it should get punished with the activated options in the admin panel.

    * If the checks failed because an error

    * The checks should be scalable that it isnt lagging a large instance behind of images. (Some of the services have "Bulk" image uploads, those should be used if there is such option.)

The section should return to the examples given in the previous section, and explain more fully how the detailed proposal makes those examples work.

In particular explain the following, if applicable:

- Database Changes:
    + Create a table "csam-detection-status with the columns
        - Id (Primary Key)
        - UserID ( string reference to the users table )
        - Status ( string | either one of "running" | "done" )
        - CSAM ( optional boolean - only when the status is "done")
        - entityId ( optional number | referencing to posts/comments/dms - only used when lazy loading is active )
        - entityType ( option string | either one of "post" | "comment" | "dm" - only used when lazy loading is active )

- API Changes:
    + New API for the settings
    + New API for the logs for the admins ( the logs can be defered from the "csam-detection-status" table )

- This feature would be ONLY local. ( It will still check federated posts )

# Drawbacks

None found please let us know if there are any.

# Rationale and alternatives

- What is the impact of not doing this?
    + Legal liability of admins ( and in some way the developers of lemmy )
    + Trauma of the users, mods, admins.
- Could this change be implemented in an external project instead? How much complexity does it add?
    + No, as this should be a core feature of the backend.

# Prior art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Does this feature exist in other social media platforms and what experience have their community had?
    + Every other social media platform has this feature ( or even his own [see Meta PQD]) because of the liability it reduces it has only positive effects.
- Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.
    + https://github.com/iftas-org/resources/blob/main/CSAM-CSE/README.md#detection
    + https://decoded.legal/blog/2022/11/notes-on-operating-fediverse-services-mastodon-pleroma-etc-from-an-english-law-point-of-view
    + https://www.eff.org/deeplinks/2022/12/user-generated-content-and-fediverse-legal-primer
    + https://www.europarl.europa.eu/RegData/etudes/BRIE/2022/738224/EPRS_BRI(2022)738224_EN.pdf
    + https://denise.dreamwidth.org/91757.html
    + https://en.wikipedia.org/wiki/Legality_of_child_pornography


# Unresolved questions

# Future possibilities

Another good extension of this, would be sort of a simple Image Hash filtering. Where admins can add image (hashes) of unwanted "Porn" images and it gets filtered out of any posts/comments/dms, with it repeated assaults can be deminished with this simple trick. 

The most problematic point is that even after purging an post/comment/dm the image(s) stay on the storage and because of that a ticking liability bomb. It should simply be avoided that such content goes to the storage. 